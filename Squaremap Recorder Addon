// ==UserScript==
// @name         Squaremap Multi-Structure Markers (Hash-Reader Mode)
// @namespace    https://mn755.local/monuments
// @version      2.1
// @description  Mark multiple Minecraft structures by reading X,Y,Z from the URL hash; export coords with type/label
// @match        https://map.loverfella.com/*
// @run-at       document-idle
// @all-frames   false
// @grant        none
// ==/UserScript==

(() => {
  'use strict';

  // ---------- Storage & helpers ----------
  const LS_KEY = 'squaremap_structures_v21';
  const load = () => { try { return JSON.parse(localStorage.getItem(LS_KEY) || '[]'); } catch { return []; } };
  const save = (arr) => localStorage.setItem(LS_KEY, JSON.stringify(arr));
  const copyText = async (t) => {
    try { await navigator.clipboard.writeText(t); }
    catch { const ta=document.createElement('textarea'); ta.value=t; document.body.appendChild(ta); ta.select(); document.execCommand('copy'); ta.remove(); }
  };
  function parseHashXYZ(hash) {
    // Expected: #world;maptype;X,Y,Z;zoom
    if (!hash || hash[0] !== '#') return null;
    const parts = hash.slice(1).split(';');
    if (parts.length < 3) return null;
    const coords = parts[2].split(',');
    if (coords.length < 3) return null;
    const x = Number(coords[0]);
    const y = Number(coords[1]);
    const z = Number(coords[2]);
    if ([x,y,z].some(n => Number.isNaN(n))) return null;
    return { x, y, z, world: parts[0], maptype: parts[1], zoom: Number(parts[3] || 0) };
  }

  // ---------- UI ----------
  const style = document.createElement('style');
  style.textContent = `
    .sm2-ui{position:fixed;top:12px;left:12px;z-index:2147483647;background:#0b1220bf;color:#fff;
      font:12px/1.2 system-ui,Segoe UI,Roboto,Arial;padding:10px;border:1px solid #28324a;border-radius:10px;backdrop-filter:saturate(140%) blur(4px)}
    .sm2-ui .row{display:flex;gap:6px;align-items:center;margin-top:6px}
    .sm2-ui label{font-size:12px;opacity:.9}
    .sm2-ui input,.sm2-ui select,.sm2-ui button{background:#0c1325;color:#e9eeff;border:1px solid #37496e;border-radius:7px;padding:6px 8px;font-size:12px}
    .sm2-ui button{background:#1b2440;border-color:#40507a;cursor:pointer}
    .sm2-ui button:hover{background:#24315a}
    .sm2-ui .txt{width:280px;height:120px;margin-top:6px;background:#0c1325;color:#dfe7ff;border:1px solid #37496e;border-radius:8px;padding:6px;font:12px/1.35 ui-monospace,Menlo,Consolas,monospace}
    .sm2-ui .muted{opacity:.85}
    .sm2-ui .tag{background:#0d172e;color:#aee;border:1px solid #3a5b7a;border-radius:6px;padding:1px 6px;margin-left:8px}
    .sm2-cross{position:fixed;left:50vw;top:50vh;pointer-events:none;z-index:2147483646}
    .sm2-cross:before,.sm2-cross:after{content:"";position:absolute;background:#ffffff90}
    .sm2-cross:before{width:1px;height:28px;left:0;top:-14px}
    .sm2-cross:after{height:1px;width:28px;left:-14px;top:0}
  `;
  document.head.appendChild(style);

  const ui = document.createElement('div');
  ui.className = 'sm2-ui';
  ui.innerHTML = `
    <div style="display:flex;align-items:center;gap:6px">
      <b>Squaremap Markers</b>
      <span class="tag" id="sm2-status">hash mode</span>
    </div>

    <div class="row">
      <label>Structure</label>
      <select id="sm2-type">
        <option value="monument">Ocean Monument</option>
        <option value="village">Village</option>
        <option value="outpost">Pillager Outpost</option>
        <option value="mansion">Woodland Mansion</option>
        <option value="stronghold">Stronghold</option>
        <option value="bastion">Bastion Remnant</option>
        <option value="fortress">Nether Fortress</option>
        <option value="ancient_city">Ancient City</option>
        <option value="shipwreck">Shipwreck</option>
        <option value="ruined_portal">Ruined Portal</option>
        <option value="ocean_ruin">Ocean Ruins</option>
        <option value="desert_temple">Desert Temple</option>
        <option value="jungle_temple">Jungle Temple</option>
        <option value="custom">Custom…</option>
      </select>
      <input id="sm2-custom" type="text" placeholder="custom type" style="display:none;width:130px"/>
    </div>

    <div class="row">
      <label>Label</label>
      <input id="sm2-label" type="text" placeholder="optional note (e.g., west of spawn)" style="flex:1"/>
    </div>

    <div class="row">
      <button id="sm2-add" title="Hotkey: M">Add from center</button>
      <button id="sm2-export">Export JSON</button>
      <button id="sm2-import">Import JSON</button>
      <button id="sm2-clear">Clear</button>
    </div>

    <div class="row">
      <button id="sm2-copy">Copy Coords</button>
      <select id="sm2-format" title="Copy format">
        <option value="xz">X Z type</option>
        <option value="csv">x,z,type,label</option>
        <option value="json">JSON [{x,z,type,label}]</option>
      </select>
      <span class="muted">Center the target, then add.</span>
    </div>

    <textarea id="sm2-out" class="txt" placeholder="Output preview (also copied)"></textarea>
  `;
  document.body.appendChild(ui);

  // crosshair for centering
  const cross = document.createElement('div');
  cross.className = 'sm2-cross';
  document.body.appendChild(cross);

  const $ = (sel, el=ui) => el.querySelector(sel);
  const statusEl = $('#sm2-status');
  const typeSel  = $('#sm2-type');
  const typeCustom = $('#sm2-custom');
  const labelIn  = $('#sm2-label');
  const addBtn   = $('#sm2-add');
  const expBtn   = $('#sm2-export');
  const impBtn   = $('#sm2-import');
  const clrBtn   = $('#sm2-clear');
  const copyBtn  = $('#sm2-copy');
  const fmtSel   = $('#sm2-format');
  const out      = $('#sm2-out');

  let markers = load(); // [{x,z,type,label,world,maptype,zoom,ts}]

  // type selector behavior
  typeSel.addEventListener('change', () => {
    const isCustom = typeSel.value === 'custom';
    typeCustom.style.display = isCustom ? 'inline-block' : 'none';
    if (isCustom) typeCustom.focus();
  });

  function currentType() {
    if (typeSel.value === 'custom') {
      const t = (typeCustom.value || '').trim();
      return t ? t : 'custom';
    }
    return typeSel.value;
  }

  function addFromHash() {
    const p = parseHashXYZ(location.hash);
    if (!p) { statusEl.textContent = 'no coords in hash'; return; }
    const t = currentType();
    const label = labelIn.value.trim() || `${t.replace(/_/g,' ')} @ ${Math.round(p.x)}, ${Math.round(p.z)}`;
    const pt = { x:p.x, z:p.z, type:t, label, world:p.world, maptype:p.maptype, zoom:p.zoom, ts:Date.now() };
    markers.push(pt);
    save(markers);
    statusEl.textContent = `added ${t} at ${Math.round(p.x)}, ${Math.round(p.z)}`;
    // leave label in place for adding a batch of same-type; clear if you hate clutter:
    // labelIn.value = '';
    preview(); // update out box with latest format
  }

  function exportJSON() {
    const data = JSON.stringify(markers, null, 2);
    out.value = data;
    copyText(data);
    statusEl.textContent = 'exported JSON';
  }

  function importJSON() {
    const raw = prompt('Paste markers JSON:'); if (!raw) return;
    try {
      const arr = JSON.parse(raw);
      if (!Array.isArray(arr)) throw 0;
      // normalize keys just in case
      markers = arr.map(o => ({
        x: Number(o.x), z: Number(o.z),
        type: String(o.type || 'structure'),
        label: o.label ? String(o.label) : `${String(o.type||'structure')} @ ${Math.round(o.x)}, ${Math.round(o.z)}`,
        world: o.world || '', maptype: o.maptype || '', zoom: Number(o.zoom||0), ts: Number(o.ts || Date.now())
      })).filter(m => Number.isFinite(m.x) && Number.isFinite(m.z));
      save(markers);
      statusEl.textContent = `imported ${markers.length}`;
      preview();
    } catch { statusEl.textContent = 'import failed'; }
  }

  function clearAll() {
    if (!confirm('Delete all local markers?')) return;
    markers = [];
    save(markers);
    out.value = '';
    statusEl.textContent = 'cleared';
  }

  function preview() {
    if (!markers.length) { out.value = ''; return; }
    const R = n => Math.round(Number(n));
    const fmt = fmtSel.value;
    let text = '';
    if (fmt === 'csv') {
      text = ['x,z,type,label'].concat(markers.map(m => `${R(m.x)},${R(m.z)},${m.type},${JSON.stringify(m.label)}`)).join('\n');
    } else if (fmt === 'json') {
      text = JSON.stringify(markers.map(m => ({ x:R(m.x), z:R(m.z), type:m.type, label:m.label })), null, 2);
    } else { // xz
      text = markers.map(m => `${R(m.x)} ${R(m.z)} ${m.type}`).join('\n');
    }
    out.value = text;
  }

  function copyOut() {
    preview();
    if (!out.value.trim()) { statusEl.textContent = 'nothing to copy'; return; }
    copyText(out.value);
    statusEl.textContent = 'copied';
  }

  // wire up
  addBtn.addEventListener('click', addFromHash);
  expBtn.addEventListener('click', exportJSON);
  impBtn.addEventListener('click', importJSON);
  clrBtn.addEventListener('click', clearAll);
  fmtSel.addEventListener('change', preview);
  copyBtn.addEventListener('click', copyOut);

  // hotkey: M to add from center
  document.addEventListener('keydown', e => {
    if (e.key.toLowerCase() === 'm') addFromHash();
  }, true);

  // live status of center coords
  function updateStatus() {
    const p = parseHashXYZ(location.hash);
    statusEl.textContent = p ? `x:${Math.round(p.x)} z:${Math.round(p.z)} (zoom ${p.zoom||0})` : 'hash mode';
  }
  window.addEventListener('hashchange', () => { updateStatus(); });
  setInterval(updateStatus, 800);
  updateStatus();

  // show existing preview
  preview();
})();
