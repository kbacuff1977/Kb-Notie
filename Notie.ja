// notie.js - drop into GitHub Pages and call via loader bookmarklet
(function(){
  if (window.__notie_loaded) return;
  window.__notie_loaded = true;

  const STORAGE_KEY = 'notie_data_v1';

  // --------- Utilities ----------
  const qs = sel => document.querySelector(sel);
  const qsa = sel => Array.from(document.querySelectorAll(sel));
  const el = (tag, props = {}, children = []) => {
    const e = document.createElement(tag);
    Object.assign(e, props);
    (children || []).forEach(c => e.appendChild(c));
    return e;
  };
  const fmtTime = (d = new Date()) => {
    // 12-hour hh:mm AM/PM
    return d.toLocaleTimeString([], {hour: '2-digit', minute: '2-digit', hour12: true});
  };
  const uid = () => 'n_' + Math.random().toString(36).slice(2,9);

  // --------- Storage ----------
  function loadData(){
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (!raw) return { notes: [], folders: [], settings: {} };
      return JSON.parse(raw);
    } catch(e) {
      return { notes: [], folders: [], settings: {} };
    }
  }
  function saveData(data){
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
  }

  // --------- Default templates ----------
  const TEMPLATES = {
    "Blank": { title: "", content: "" },
    "Journaling": {
      title: "",
      content:
`Mood:
What happened today:
What I learned:
Gratitude:
Next actions:`
    },
    "College Notes": {
      title: "",
      content:
`Course:
Date:
Topic:
Key points:
Definitions:
Examples:
Questions/Follow-up:`
    },
    "Professional / Business": {
      title: "",
      content:
`Meeting / Project:
Date:
Attendees:
Agenda:
Decisions:
Action items (owner -> due date):
Notes / Follow-up:`
    }
  };

  // --------- Core UI build ----------
  function buildUI(){
    if(document.getElementById('notie-root')) return;

    // container
    const root = el('div', { id: 'notie-root' });
    // basic CSS via style element
    const style = el('style');
    style.textContent = `
#notie-root * { box-sizing: border-box; }
#notie-panel {
  position: fixed;
  top: 80px;
  left: 40px;
  width: 420px;
  height: 560px;
  background: rgba(28,28,30,0.6);
  backdrop-filter: blur(10px);
  color: #fff;
  border: 2px solid #ffd24d;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.6);
  z-index: 2147483647;
  display:flex;
  flex-direction:column;
  font-family: "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}
#notie-bar {
  display:flex;
  align-items:center;
  justify-content:space-between;
  padding:10px 12px;
  background: linear-gradient(90deg,#4d4d4d,#6b6b6b);
  border-top-left-radius:10px;
  border-top-right-radius:10px;
  cursor: move;
}
#notie-title { font-weight:700; font-size:16px; }
#notie-close { cursor:pointer; font-size:16px; padding:6px; border-radius:6px; background:transparent; }
#notie-body { padding:8px; display:flex; flex-direction:column; gap:8px; flex:1; overflow:hidden; }
#notie-controls { display:flex; gap:6px; align-items:center; flex-wrap:wrap; }
.notie-btn { background: linear-gradient(90deg,#5b5b5b,#7b7b7b); color:#fff; border:none; padding:8px 10px; border-radius:8px; cursor:pointer; font-size:14px; }
.notie-btn:hover{ filter:brightness(1.08); }
#notie-filter-row { display:flex; gap:6px; align-items:center; }
#notie-folders, #notie-tags { padding:6px; border-radius:8px; background:rgba(0,0,0,0.2); color:#fff; border:1px solid rgba(255,255,255,0.04); }
#notie-list { flex:1; overflow:auto; padding-right:8px; }
.notie-note { background: rgba(40,40,40,0.55); padding:10px; margin-bottom:8px; border-radius:8px; display:flex; flex-direction:column; gap:6px; }
.notie-note-title { font-weight:700; font-size:15px; }
.notie-note-meta { font-size:12px; color:#d0d0d0; display:flex; gap:8px; align-items:center; flex-wrap:wrap; }
.notie-note-content { white-space:pre-wrap; color:#eee; }
.notie-note-actions { display:flex; gap:6px; justify-content:flex-end; }
.small-input { padding:6px; border-radius:6px; border:1px solid rgba(255,255,255,0.06); background:rgba(0,0,0,0.18); color:#fff; }
#notie-editor { display:flex; flex-direction:column; gap:6px; border-radius:8px; padding:8px; background: rgba(0,0,0,0.12); }
#notie-editor input, #notie-editor textarea { width:100%; padding:8px; border-radius:6px; border:1px solid rgba(0,0,0,0.06); background:rgba(255,255,255,0.03); color:#fff; resize:vertical; }
#notie-footer { display:flex; gap:8px; justify-content:space-between; align-items:center; padding:8px 12px; }
#notie-empty { color:#ddd; font-size:14px; text-align:center; margin-top:20px; }
@media (max-width:480px){
  #notie-panel { left:16px; top:60px; width:90%; height:70vh; }
}
    `;
    document.head.appendChild(style);

    // panel
    const panel = el('div',{ id:'notie-panel' });

    // title bar
    const bar = el('div',{ id:'notie-bar' });
    const titleEl = el('div',{ id:'notie-title', innerText:'📝 Notie' });
    const controlsRight = el('div', { style: 'display:flex; gap:8px; align-items:center;' });
    const closeBtn = el('div',{ id:'notie-close', innerText:'✖', title:'Close Notie' });
    controlsRight.appendChild(closeBtn);
    bar.appendChild(titleEl);
    bar.appendChild(controlsRight);

    // body
    const body = el('div',{ id:'notie-body' });

    // controls row: add, template select, new folder, search, filter
    const controls = el('div',{ id:'notie-controls' });
    const addBtn = el('button',{ className:'notie-btn', innerText: '+ New' });
    const templateSelect = el('select',{ className:'small-input' });
    Object.keys(TEMPLATES).forEach(k=>{
      const o = document.createElement('option'); o.value = k; o.innerText = k; templateSelect.appendChild(o);
    });
    const newFolderBtn = el('button',{ className:'notie-btn', innerText: 'New Folder' });
    const folderFilter = el('select',{ id:'notie-folders', className:'small-input' });
    const tagFilter = el('input',{ id:'notie-tags', className:'small-input', placeholder:'Filter tag' });

    controls.appendChild(addBtn);
    controls.appendChild(templateSelect);
    controls.appendChild(newFolderBtn);
    controls.appendChild(folderFilter);
    controls.appendChild(tagFilter);

    // search row
    const filterRow = el('div',{ id:'notie-filter-row' });
    const searchInput = el('input',{ className:'small-input', placeholder:'Search notes...' });
    const sortLabel = el('div',{ style:'font-size:12px;color:#ddd', innerText:'Newest' });
    filterRow.appendChild(searchInput);
    filterRow.appendChild(sortLabel);

    // list
    const listWrap = el('div',{ id:'notie-list' });

    // editor area (create / edit)
    const editor = el('div',{ id:'notie-editor' });
    const titleInput = el('input',{ placeholder:'Title (optional)' });
    const tagsInput = el('input',{ placeholder:'Tags (comma separated)' });
    const folderInput = el('input',{ placeholder:'Folder (optional)' });
    const contentArea = el('textarea',{ rows:6, placeholder:'Write your note here...' });
    const templateHint = el('div',{ style:'font-size:12px;color:#ccc' , innerText:'Choose template above before clicking + New to prefill.'});

    editor.appendChild(titleInput);
    editor.appendChild(tagsInput);
    editor.appendChild(folderInput);
    editor.appendChild(contentArea);
    editor.appendChild(templateHint);

    // footer with save/cancel and quick templates
    const footer = el('div',{ id:'notie-footer' });
    const leftFooter = el('div',{},[]);
    const rightFooter = el('div',{},[]);
    const saveBtn = el('button',{ className:'notie-btn', innerText:'Save' });
    const cancelBtn = el('button',{ className:'notie-btn', innerText:'Cancel' });
    const quickTemplateLabel = el('div',{ style:'font-size:12px;color:#ddd', innerText:'Quick templates:'});

    // quick templates dropdown
    const quickSelect = el('select',{ className:'small-input' });
    const quickEmptyOpt = el('option',{ value:'', innerText:'(none)' });
    quickSelect.appendChild(quickEmptyOpt);
    Object.keys(TEMPLATES).forEach(k=>{
      const o = document.createElement('option'); o.value = k; o.innerText = k; quickSelect.appendChild(o);
    });

    leftFooter.appendChild(quickTemplateLabel);
    leftFooter.appendChild(quickSelect);
    rightFooter.appendChild(cancelBtn);
    rightFooter.appendChild(saveBtn);
    footer.appendChild(leftFooter);
    footer.appendChild(rightFooter);

    // assemble
    body.appendChild(controls);
    body.appendChild(filterRow);
    body.appendChild(listWrap);
    body.appendChild(editor);
    body.appendChild(footer);

    panel.appendChild(bar);
    panel.appendChild(body);
    root.appendChild(panel);
    document.body.appendChild(root);

    // --------- App state ----------
    let data = loadData();
    if(!data.notes) data.notes = [];
    if(!data.folders) data.folders = [];
    // helper to ensure folder list contains root
    if(!data.folders.includes('Inbox')) data.folders.unshift('Inbox');

    let editingId = null; // null => new note

    // --------- UI behaviors ----------
    function refreshFolderOptions(){
      // folder filter dropdown (first option 'All')
      folderFilter.innerHTML = '';
      const allOpt = document.createElement('option'); allOpt.value=''; allOpt.innerText='All folders'; folderFilter.appendChild(allOpt);
      data.folders.forEach(f => {
        const o = document.createElement('option'); o.value=f; o.innerText=f; folderFilter.appendChild(o);
      });
      // also update folderInput datalist-like suggestions via placeholder (simple)
    }

    function renderList(){
      listWrap.innerHTML = '';
      // apply filters: search, folder, tag
      const q = searchInput.value.trim().toLowerCase();
      const folderF = folderFilter.value;
      const tagQ = tagFilter.value.trim().toLowerCase();

      // sort newest first by createdAt descending
      const notes = (data.notes || []).slice().sort((a,b)=>b.createdAt - a.createdAt);

      const filtered = notes.filter(n=>{
        if(folderF && n.folder !== folderF) return false;
        if(tagQ){
          const has = (n.tags || []).some(t=>t.toLowerCase().includes(tagQ));
          if(!has) return false;
        }
        if(q){
          const hay = (n.title||'') + ' ' + (n.content||'') + ' ' + ((n.tags||[]).join(' ')) + ' ' + (n.folder||'');
          if(!hay.toLowerCase().includes(q)) return false;
        }
        return true;
      });

      if(filtered.length === 0){
        const empty = el('div',{ id:'notie-empty', innerText:'No notes found — create one with + New' });
        listWrap.appendChild(empty);
        return;
      }

      filtered.forEach(n=>{
        const item = el('div',{ className:'notie-note' });
        const title = el('div',{ className:'notie-note-title', innerText: n.title || '(No title)' });
        const meta = el('div',{ className:'notie-note-meta' });
        const time = el('div',{ innerText: n.time + ' • ' + (n.createdDate||'') });
        meta.appendChild(time);
        if(n.folder) meta.appendChild(el('div',{ innerText: 'Folder: ' + n.folder }));
        if(n.tags && n.tags.length) meta.appendChild(el('div',{ innerText: 'Tags: ' + n.tags.join(', ') }));
        const content = el('div',{ className:'notie-note-content', innerText: n.content || '' });

        const actions = el('div',{ className:'notie-note-actions' });
        const editBtn = el('button',{ className:'notie-btn', innerText:'Edit' });
        const delBtn = el('button',{ className:'notie-btn', innerText:'Delete' });
        const moveBtn = el('button',{ className:'notie-btn', innerText:'Move' });

        editBtn.onclick = ()=> openEditor(n.id);
        delBtn.onclick = ()=> {
          if(confirm('Delete this note?')) {
            data.notes = data.notes.filter(x=>x.id!==n.id);
            saveData(data);
            renderList();
          }
        };
        moveBtn.onclick = ()=>{
          const f = prompt('Move to folder:', n.folder || (data.folders[0]||'Inbox'));
          if(f!==null){
            n.folder = (f||'');
            if(f && !data.folders.includes(f)) data.folders.push(f);
            saveData(data);
            refreshFolderOptions();
            renderList();
          }
        };

        actions.appendChild(moveBtn);
        actions.appendChild(editBtn);
        actions.appendChild(delBtn);

        item.appendChild(title);
        item.appendChild(meta);
        item.appendChild(content);
        item.appendChild(actions);

        listWrap.appendChild(item);
      });
    }

    function openEditor(id){
      editingId = id || null;
      if(editingId){
        const n = data.notes.find(x=>x.id===id);
        titleInput.value = n.title || '';
        tagsInput.value = (n.tags || []).join(', ');
        folderInput.value = n.folder || '';
        contentArea.value = n.content || '';
      } else {
        titleInput.value = '';
        tagsInput.value = '';
        folderInput.value = '';
        const tpl = TEMPLATES[templateSelect.value] || { title:'', content:'' };
        titleInput.value = tpl.title || '';
        contentArea.value = tpl.content || '';
      }
      // focus content for quick typing
      contentArea.focus();
    }

    function saveNote(){
      const title = titleInput.value.trim();
      const content = contentArea.value.trim();
      const tags = tagsInput.value.split(',').map(s=>s.trim()).filter(Boolean);
      const folder = folderInput.value.trim() || '';
      if(!content && !title){
        alert('Please enter a title or some content for the note.');
        return;
      }
      const now = new Date();
      const noteObj = {
        id: editingId || uid(),
        title: title,
        content: content,
        tags: tags,
        folder: folder || '',
        time: fmtTime(now),
        createdAt: editingId ? (data.notes.find(n=>n.id===editingId).createdAt) : now.getTime(),
        createdDate: now.toLocaleDateString()
      };

      if(editingId){
        // replace in data
        data.notes = data.notes.map(n=> n.id===editingId ? Object.assign({}, n, noteObj) : n);
      } else {
        data.notes.push(noteObj);
      }
      // add folder to folders list
      if(folder && !data.folders.includes(folder)) data.folders.push(folder);
      saveData(data);
      refreshFolderOptions();
      renderList();
      // clear editor
      editingId = null;
      titleInput.value = '';
      tagsInput.value = '';
      folderInput.value = '';
      contentArea.value = '';
    }

    // --------- Initial wiring ----------
    refreshFolderOptions();
    renderList();

    // add button
    addBtn.onclick = ()=> openEditor(null);

    // new folder
    newFolderBtn.onclick = ()=>{
      const f = prompt('New folder name:');
      if(f){
        if(!data.folders.includes(f)) data.folders.push(f);
        saveData(data);
        refreshFolderOptions();
      }
    };

    // save / cancel
    saveBtn.onclick = saveNote;
    cancelBtn.onclick = ()=> { editingId = null; titleInput.value=''; tagsInput.value=''; folderInput.value=''; contentArea.value=''; };

    // quick template apply
    quickSelect.onchange = ()=>{
      const t = quickSelect.value;
      if(t && TEMPLATES[t]){
        const tpl = TEMPLATES[t];
        titleInput.value = tpl.title || '';
        contentArea.value = tpl.content || '';
      }
    };

    // search/filter
    searchInput.addEventListener('input', debounce(renderList, 150));
    folderFilter.addEventListener('change', renderList);
    tagFilter.addEventListener('input', debounce(renderList, 150));

    // template select (prefill when creating new)
    templateSelect.addEventListener('change', ()=>{ /* no-op, used when New clicked */ });

    // open editor on first load if no notes
    if((data.notes||[]).length === 0){
      openEditor(null);
    }

    // close panel
    closeBtn.onclick = ()=> {
      const rootElem = document.getElementById('notie-root');
      if(rootElem) rootElem.remove();
      window.__notie_loaded = false;
    };

    // --------- Dragging (desktop + touch) ----------
    let dragging=false, dx=0, dy=0;
    function dragStart(e){
      dragging=true;
      const point = (e.touches && e.touches[0]) || e;
      dx = point.clientX - panel.offsetLeft;
      dy = point.clientY - panel.offsetTop;
      e.preventDefault();
    }
    function dragMove(e){
      if(!dragging) return;
      const point = (e.touches && e.touches[0]) || e;
      panel.style.left = (point.clientX - dx) + 'px';
      panel.style.top = (point.clientY - dy) + 'px';
      panel.style.right = '';
    }
    function dragEnd(){ dragging=false; }
    bar.addEventListener('mousedown', dragStart);
    bar.addEventListener('touchstart', dragStart, {passive:false});
    document.addEventListener('mousemove', dragMove);
    document.addEventListener('touchmove', dragMove, {passive:false});
    document.addEventListener('mouseup', dragEnd);
    document.addEventListener('touchend', dragEnd);

    // attach panel variable (for drag functions)
    const panelEl = panel;
    panelEl.style.left = panel.style.left || '40px';
    panelEl.style.top = panel.style.top || '80px';

    // small helper functions
    function debounce(fn,wait){ let t; return function(...a){ clearTimeout(t); t=setTimeout(()=>fn.apply(this,a), wait); }; }

    // finally: store references used by drag functions
    Object.defineProperty(window, '__notie_internal_panel', { value: panelEl, writable:false });

    // keep data in scope
    window.__notie_data = data;
    // ensure UI shows current state
    refreshFolderOptions();
    renderList();

  } // end buildUI

  // run
  buildUI();

})();
