# organizerplus
my organizer planner calendar
[santiago_planner.html](https://github.com/user-attachments/files/26198830/santiago_planner.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Santiago — Planner</title>
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #f5f4f1;
    --surface: #ffffff;
    --surface2: #f0efe9;
    --border: #e2e0d8;
    --text: #1a1916;
    --text2: #6b6960;
    --text3: #a8a49c;
    --red: #c0392b; --red-bg: #fdf0ee;
    --yellow: #c67c00; --yellow-bg: #fdf8ee;
    --green: #2d7a4f; --green-bg: #eef8f2;
    --blue: #1a56a0; --blue-bg: #eef3fb;
    --radius: 10px;
    --font: 'DM Sans', sans-serif;
    --mono: 'DM Mono', monospace;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: var(--font); background: var(--bg); color: var(--text); min-height: 100vh; font-size: 14px; overflow: hidden; }

  /* TOPBAR */
  .topbar { position: fixed; top: 0; left: 0; right: 0; z-index: 300; background: var(--surface); border-bottom: 1px solid var(--border); display: flex; align-items: center; padding: 0 20px; height: 52px; gap: 8px; }
  .topbar-title { font-size: 15px; font-weight: 500; letter-spacing: -0.02em; margin-right: 8px; }
  .topbar-dot { width: 6px; height: 6px; border-radius: 50%; background: var(--border); margin: 0 4px; }
  .view-tabs { display: flex; gap: 2px; background: var(--surface2); border-radius: 8px; padding: 3px; }
  .view-tab { padding: 4px 12px; border-radius: 6px; font-size: 12px; font-weight: 500; cursor: pointer; border: none; background: none; color: var(--text2); transition: all 0.15s; white-space: nowrap; font-family: var(--font); }
  .view-tab.active { background: var(--surface); color: var(--text); box-shadow: 0 1px 3px rgba(0,0,0,0.08); }
  .spacer { flex: 1; }
  .topbar-btn { padding: 6px 12px; border-radius: 7px; font-size: 12px; font-weight: 500; cursor: pointer; border: 1px solid var(--border); background: var(--surface); color: var(--text2); font-family: var(--font); transition: all 0.15s; white-space: nowrap; }
  .topbar-btn:hover { background: var(--surface2); color: var(--text); }
  .topbar-btn.primary { background: var(--text); color: #fff; border-color: var(--text); }
  .topbar-btn.primary:hover { opacity: 0.85; }

  /* CANVAS — free-position workspace */
  #canvas {
    position: fixed;
    top: 52px; left: 0; right: 0; bottom: 0;
    overflow: auto;
  }
  #canvas-inner {
    position: relative;
    width: 3000px;
    height: 2000px;
  }

  /* URGENT BAR */
  .urgent-bar { position: fixed; top: 52px; left: 0; right: 0; z-index: 200; background: var(--red-bg); border-bottom: 1px solid #f5c5c0; padding: 7px 20px; display: flex; align-items: center; gap: 10px; flex-wrap: wrap; }
  .urgent-bar-label { font-size: 11px; font-weight: 500; color: var(--red); text-transform: uppercase; letter-spacing: 0.06em; }
  .urgent-chip { font-size: 11px; padding: 3px 8px; border-radius: 5px; background: #fff; border: 1px solid #f5c5c0; color: var(--red); cursor: pointer; }

  /* CLASS GROUP — freely positioned */
  .class-group {
    position: absolute;
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    display: flex;
    flex-direction: column;
    max-height: 600px;
    box-shadow: 0 2px 12px rgba(0,0,0,0.06);
    transition: box-shadow 0.15s;
  }
  .class-group.dragging { box-shadow: 0 12px 40px rgba(0,0,0,0.18); opacity: 0.95; z-index: 500; }
  .class-group.collapsed { max-height: none; }

  .group-header {
    display: flex; align-items: center; gap: 8px;
    padding: 11px 13px 9px;
    border-bottom: 1px solid var(--border);
    cursor: grab;
    user-select: none;
    border-radius: var(--radius) var(--radius) 0 0;
  }
  .group-header:active { cursor: grabbing; }
  .group-color { width: 10px; height: 10px; border-radius: 50%; flex-shrink: 0; }
  .group-name { font-size: 13px; font-weight: 500; flex: 1; }
  .group-count { font-size: 11px; color: var(--text3); font-family: var(--mono); }
  .group-collapse-btn { width: 20px; height: 20px; border-radius: 5px; border: none; background: none; color: var(--text3); cursor: pointer; font-size: 11px; display: flex; align-items: center; justify-content: center; }
  .group-collapse-btn:hover { background: var(--surface2); color: var(--text); }

  .group-tasks { overflow-y: auto; padding: 8px; flex: 1; min-height: 40px; }
  .group-tasks::-webkit-scrollbar { width: 4px; }
  .group-tasks::-webkit-scrollbar-thumb { background: var(--border); border-radius: 4px; }
  .collapsed .group-tasks { display: none; }
  .collapsed .group-footer { display: none; }

  .group-footer { padding: 8px; border-top: 1px solid var(--border); }
  .add-task-btn { width: 100%; padding: 7px; border: 1.5px dashed var(--border); border-radius: 7px; background: none; color: var(--text3); font-size: 12px; font-family: var(--font); cursor: pointer; transition: all 0.15s; }
  .add-task-btn:hover { border-color: var(--text3); color: var(--text2); background: var(--surface2); }

  /* resize handle bottom-right */
  .resize-handle {
    position: absolute; right: 0; bottom: 0;
    width: 16px; height: 16px;
    cursor: se-resize;
    display: flex; align-items: flex-end; justify-content: flex-end;
    padding: 3px;
  }
  .resize-handle::after {
    content: '';
    width: 6px; height: 6px;
    border-right: 2px solid var(--border);
    border-bottom: 2px solid var(--border);
    border-radius: 1px;
  }
  .collapsed .resize-handle { display: none; }

  /* TASK CARD */
  .task-card { background: var(--surface); border: 1px solid var(--border); border-radius: 8px; padding: 9px 11px; margin-bottom: 6px; cursor: default; transition: box-shadow 0.12s, border-color 0.12s; position: relative; }
  .task-card:hover { box-shadow: 0 2px 8px rgba(0,0,0,0.07); border-color: #ccc; }
  .task-card.priority-red { border-left: 3px solid var(--red); background: var(--red-bg); }
  .task-card.priority-yellow { border-left: 3px solid var(--yellow); background: var(--yellow-bg); }
  .task-card.priority-green { border-left: 3px solid var(--green); background: var(--green-bg); }
  .task-card.done-card { opacity: 0.4; }
  .task-card.done-card .task-title { text-decoration: line-through; }
  .task-top { display: flex; align-items: flex-start; gap: 8px; }
  .task-cb { width: 15px; height: 15px; border-radius: 4px; border: 1.5px solid var(--border); flex-shrink: 0; margin-top: 1px; display: flex; align-items: center; justify-content: center; cursor: pointer; transition: all 0.12s; }
  .task-cb.checked { background: var(--text); border-color: var(--text); }
  .task-title { font-size: 13px; line-height: 1.4; flex: 1; }
  .task-meta { display: flex; align-items: center; gap: 6px; margin-top: 6px; flex-wrap: wrap; }
  .task-date { font-size: 11px; color: var(--text3); font-family: var(--mono); }
  .task-type { font-size: 10px; padding: 2px 6px; border-radius: 4px; font-weight: 500; }
  .type-test { background: #fde8e8; color: #b91c1c; }
  .type-hw { background: var(--surface2); color: var(--text2); }
  .type-ec { background: #fef3c7; color: #92400e; }
  .task-actions { display: none; position: absolute; top: 6px; right: 6px; }
  .task-card:hover .task-actions { display: flex; }
  .task-action-btn { width: 22px; height: 22px; border-radius: 5px; border: none; background: var(--surface); box-shadow: 0 1px 3px rgba(0,0,0,0.1); color: var(--text2); cursor: pointer; font-size: 11px; display: flex; align-items: center; justify-content: center; }
  .task-action-btn:hover { background: var(--surface2); }

  /* CALENDAR */
  #calendar-view { display: none; position: fixed; top: 52px; left: 0; right: 0; bottom: 0; overflow-y: auto; padding: 20px; background: var(--bg); z-index: 100; }
  .cal-header { display: flex; align-items: center; gap: 12px; margin-bottom: 20px; }
  .cal-title { font-size: 18px; font-weight: 500; }
  .cal-nav { width: 30px; height: 30px; border-radius: 7px; border: 1px solid var(--border); background: var(--surface); color: var(--text2); cursor: pointer; font-size: 14px; display: flex; align-items: center; justify-content: center; }
  .cal-nav:hover { background: var(--surface2); }
  .cal-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 1px; background: var(--border); border: 1px solid var(--border); border-radius: var(--radius); overflow: hidden; }
  .cal-dow { background: var(--surface2); text-align: center; padding: 8px; font-size: 11px; font-weight: 500; color: var(--text3); text-transform: uppercase; letter-spacing: 0.05em; }
  .cal-cell { background: var(--surface); min-height: 90px; padding: 8px; }
  .cal-cell.other-month { background: var(--surface2); }
  .cal-cell.today { background: #f0f4ff; }
  .cal-day-num { font-size: 12px; font-weight: 500; color: var(--text3); margin-bottom: 4px; font-family: var(--mono); }
  .cal-cell.today .cal-day-num { color: var(--blue); }
  .cal-task-chip { font-size: 10px; padding: 2px 5px; border-radius: 4px; margin-bottom: 2px; cursor: pointer; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; border-left: 2px solid; }

  /* MATRIX */
  #matrix-view { display: none; position: fixed; top: 52px; left: 0; right: 0; bottom: 0; padding: 20px; background: var(--bg); z-index: 100; overflow-y: auto; }
  .matrix-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; min-height: calc(100vh - 110px); }
  .matrix-cell { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 14px; overflow-y: auto; }
  .matrix-label { font-size: 11px; font-weight: 500; text-transform: uppercase; letter-spacing: 0.06em; margin-bottom: 10px; }
  .matrix-cell.q1 { border-top: 3px solid var(--red); }
  .matrix-cell.q2 { border-top: 3px solid var(--yellow); }
  .matrix-cell.q3 { border-top: 3px solid var(--text3); }
  .matrix-cell.q4 { border-top: 3px solid var(--green); }

  /* GRADES */
  #grades-view { display: none; position: fixed; top: 52px; left: 0; right: 0; bottom: 0; padding: 20px; background: var(--bg); z-index: 100; overflow-y: auto; }
  .grades-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 14px; }
  .grade-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 16px; }
  .grade-card-header { display: flex; align-items: center; gap: 10px; margin-bottom: 14px; }
  .grade-dot { width: 10px; height: 10px; border-radius: 50%; }
  .grade-class-name { font-size: 14px; font-weight: 500; }
  .grade-entry { display: flex; align-items: center; gap: 8px; margin-bottom: 6px; }
  .grade-entry-name { flex: 1; font-size: 13px; color: var(--text2); }
  .grade-entry-score { font-family: var(--mono); font-size: 13px; font-weight: 500; }
  .grade-add-btn { margin-top: 10px; width: 100%; padding: 7px; border: 1.5px dashed var(--border); border-radius: 7px; background: none; color: var(--text3); font-size: 12px; font-family: var(--font); cursor: pointer; transition: all 0.15s; }
  .grade-add-btn:hover { border-color: var(--text3); color: var(--text2); }
  .grade-avg { font-size: 13px; color: var(--text3); margin-top: 12px; padding-top: 10px; border-top: 1px solid var(--border); display: flex; justify-content: space-between; }
  .grade-avg span { font-weight: 500; color: var(--text); font-family: var(--mono); }

  /* MODAL */
  .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.25); z-index: 1000; display: flex; align-items: center; justify-content: center; opacity: 0; pointer-events: none; transition: opacity 0.15s; }
  .modal-overlay.open { opacity: 1; pointer-events: all; }
  .modal { background: var(--surface); border-radius: 14px; padding: 24px; width: 420px; max-width: 95vw; box-shadow: 0 20px 60px rgba(0,0,0,0.15); transform: translateY(8px); transition: transform 0.15s; max-height: 90vh; overflow-y: auto; }
  .modal-overlay.open .modal { transform: translateY(0); }
  .modal-title { font-size: 16px; font-weight: 500; margin-bottom: 18px; }
  .form-row { margin-bottom: 14px; }
  .form-label { font-size: 11px; font-weight: 500; color: var(--text2); text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 5px; display: block; }
  .form-input { width: 100%; padding: 9px 12px; border: 1px solid var(--border); border-radius: 8px; font-size: 13px; font-family: var(--font); background: var(--surface); color: var(--text); outline: none; transition: border-color 0.12s; }
  .form-input:focus { border-color: var(--text3); }
  select.form-input { cursor: pointer; }
  .form-row-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 14px; }
  .priority-btns { display: flex; gap: 6px; }
  .priority-btn { flex: 1; padding: 8px; border-radius: 7px; border: 1.5px solid var(--border); background: none; font-size: 12px; font-weight: 500; cursor: pointer; font-family: var(--font); transition: all 0.12s; }
  .priority-btn.p-none.sel { background: var(--surface2); border-color: var(--text3); color: var(--text); }
  .priority-btn.p-green.sel { background: var(--green-bg); border-color: var(--green); color: var(--green); }
  .priority-btn.p-yellow.sel { background: var(--yellow-bg); border-color: var(--yellow); color: var(--yellow); }
  .priority-btn.p-red.sel { background: var(--red-bg); border-color: var(--red); color: var(--red); }
  .modal-footer { display: flex; gap: 8px; margin-top: 20px; justify-content: flex-end; }
  .btn { padding: 8px 16px; border-radius: 8px; font-size: 13px; font-weight: 500; cursor: pointer; border: 1px solid var(--border); background: var(--surface); color: var(--text2); font-family: var(--font); transition: all 0.12s; }
  .btn:hover { background: var(--surface2); }
  .btn.primary { background: var(--text); color: #fff; border-color: var(--text); }
  .btn.primary:hover { opacity: 0.85; }
  .btn.danger { background: var(--red-bg); color: var(--red); border-color: #f5c5c0; }

  /* ADD CLASS TILE */
  .add-group-tile { position: absolute; background: none; border: 2px dashed var(--border); border-radius: var(--radius); width: 200px; height: 80px; display: flex; align-items: center; justify-content: center; cursor: pointer; color: var(--text3); font-size: 13px; font-family: var(--font); transition: all 0.15s; gap: 6px; }
  .add-group-tile:hover { border-color: var(--text3); color: var(--text2); background: var(--surface); }
</style>
</head>
<body>

<!-- TOPBAR -->
<div class="topbar">
  <span class="topbar-title">Planner</span>
  <div class="topbar-dot"></div>
  <div class="view-tabs">
    <button class="view-tab active" onclick="switchView('board')">Board</button>
    <button class="view-tab" onclick="switchView('calendar')">Calendar</button>
    <button class="view-tab" onclick="switchView('matrix')">Matrix</button>
    <button class="view-tab" onclick="switchView('grades')">Grades</button>
  </div>
  <div class="spacer"></div>
  <button class="topbar-btn" onclick="exportICS()">Export .ics</button>
  <button class="topbar-btn primary" onclick="openAddTask()">+ Task</button>
</div>

<!-- URGENT BAR -->
<div class="urgent-bar" id="urgent-bar" style="display:none;">
  <span class="urgent-bar-label">🔴 Urgent</span>
  <span id="urgent-chips"></span>
</div>

<!-- FREE CANVAS -->
<div id="canvas">
  <div id="canvas-inner"></div>
</div>

<!-- CALENDAR -->
<div id="calendar-view">
  <div class="cal-header">
    <button class="cal-nav" onclick="calNav(-1)">‹</button>
    <span class="cal-title" id="cal-title"></span>
    <button class="cal-nav" onclick="calNav(1)">›</button>
  </div>
  <div class="cal-grid" id="cal-grid"></div>
</div>

<!-- MATRIX -->
<div id="matrix-view">
  <p style="font-size:13px;color:var(--text2);margin-bottom:14px;">Assign quadrants when adding or editing a task.</p>
  <div class="matrix-grid">
    <div class="matrix-cell q1"><div class="matrix-label" style="color:var(--red);">⬆ Urgent & Important — Do First</div><div id="m-q1"></div></div>
    <div class="matrix-cell q2"><div class="matrix-label" style="color:var(--yellow);">⬆ Not Urgent, Important — Schedule</div><div id="m-q2"></div></div>
    <div class="matrix-cell q3"><div class="matrix-label" style="color:var(--text3);">⬇ Urgent, Not Important — Delegate</div><div id="m-q3"></div></div>
    <div class="matrix-cell q4"><div class="matrix-label" style="color:var(--green);">⬇ Not Urgent, Not Important — Eliminate</div><div id="m-q4"></div></div>
  </div>
</div>

<!-- GRADES -->
<div id="grades-view">
  <div class="grades-grid" id="grades-grid"></div>
</div>

<!-- TASK MODAL -->
<div class="modal-overlay" id="task-modal">
  <div class="modal">
    <div class="modal-title" id="modal-title-text">New Task</div>
    <div class="form-row"><label class="form-label">Title</label><input class="form-input" id="f-title" placeholder="e.g. Read Ch. 5"></div>
    <div class="form-row-2">
      <div><label class="form-label">Class</label><select class="form-input" id="f-class"></select></div>
      <div><label class="form-label">Type</label><select class="form-input" id="f-type"><option value="hw">Homework</option><option value="test">Test / Quiz</option><option value="ec">Extra Credit</option></select></div>
    </div>
    <div class="form-row"><label class="form-label">Due Date</label><input class="form-input" type="date" id="f-date"></div>
    <div class="form-row">
      <label class="form-label">Priority</label>
      <div class="priority-btns">
        <button class="priority-btn p-none sel" data-p="" onclick="selPriority(this)">None</button>
        <button class="priority-btn p-green" data-p="green" onclick="selPriority(this)">🟢 Low</button>
        <button class="priority-btn p-yellow" data-p="yellow" onclick="selPriority(this)">🟡 Medium</button>
        <button class="priority-btn p-red" data-p="red" onclick="selPriority(this)">🔴 High</button>
      </div>
    </div>
    <div class="form-row"><label class="form-label">Eisenhower Quadrant</label><select class="form-input" id="f-quadrant"><option value="">— Unassigned —</option><option value="q1">Q1 — Urgent & Important</option><option value="q2">Q2 — Not Urgent, Important</option><option value="q3">Q3 — Urgent, Not Important</option><option value="q4">Q4 — Not Urgent, Not Important</option></select></div>
    <div class="form-row"><label class="form-label">Notes</label><input class="form-input" id="f-notes" placeholder="Optional..."></div>
    <div class="modal-footer">
      <button class="btn danger" id="modal-delete-btn" style="display:none;margin-right:auto;" onclick="deleteTask()">Delete</button>
      <button class="btn" onclick="closeModal()">Cancel</button>
      <button class="btn primary" onclick="saveTask()">Save</button>
    </div>
  </div>
</div>

<!-- ADD CLASS MODAL -->
<div class="modal-overlay" id="class-modal">
  <div class="modal">
    <div class="modal-title">Add Class</div>
    <div class="form-row"><label class="form-label">Class name</label><input class="form-input" id="cf-name" placeholder="e.g. AP Bio"></div>
    <div class="form-row"><label class="form-label">Color</label><input type="color" id="cf-color" value="#4f8ef7" style="width:100%;height:36px;border-radius:8px;border:1px solid var(--border);padding:2px;cursor:pointer;"></div>
    <div class="modal-footer">
      <button class="btn" onclick="document.getElementById('class-modal').classList.remove('open')">Cancel</button>
      <button class="btn primary" onclick="saveClass()">Add Class</button>
    </div>
  </div>
</div>

<!-- GRADE MODAL -->
<div class="modal-overlay" id="grade-modal">
  <div class="modal">
    <div class="modal-title" id="grade-modal-title">Add Grade</div>
    <div class="form-row"><label class="form-label">Assignment name</label><input class="form-input" id="gf-name" placeholder="e.g. LAP 4 Test"></div>
    <div class="form-row-2">
      <div><label class="form-label">Score earned</label><input class="form-input" type="number" id="gf-score" placeholder="88"></div>
      <div><label class="form-label">Out of</label><input class="form-input" type="number" id="gf-total" placeholder="100"></div>
    </div>
    <div class="modal-footer">
      <button class="btn" onclick="document.getElementById('grade-modal').classList.remove('open')">Cancel</button>
      <button class="btn primary" onclick="saveGrade()">Save</button>
    </div>
  </div>
</div>

<script>
// ─── STATE ────────────────────────────────────────────────────────────────────
const DEFAULT_POSITIONS = [
  { id:'lang',  name:'AP Lang',    color:'#2563eb', x:20,  y:20,  w:260 },
  { id:'apush', name:'APUSH',      color:'#d97706', x:300, y:20,  w:260 },
  { id:'esp',   name:'AP Español', color:'#dc2626', x:580, y:20,  w:260 },
  { id:'pcal',  name:'Pre-Cal',    color:'#16a34a', x:860, y:20,  w:260 },
  { id:'theo',  name:'Theology',   color:'#7c3aed', x:20,  y:440, w:260 },
  { id:'phys',  name:'AP Physics', color:'#db2777', x:300, y:440, w:260 },
  { id:'robo',  name:'Robotics',   color:'#0891b2', x:580, y:440, w:260 },
];

let state = JSON.parse(localStorage.getItem('planner_v2') || 'null') || {
  classes: DEFAULT_POSITIONS,
  tasks: [],
  collapsed: {},
  grades: {}
};

function save() { localStorage.setItem('planner_v2', JSON.stringify(state)); }

// ─── VIEW ─────────────────────────────────────────────────────────────────────
let currentView = 'board';
function switchView(v) {
  currentView = v;
  const views = ['board','calendar','matrix','grades'];
  views.forEach(id => {
    const el = document.getElementById(id === 'board' ? 'canvas' : id+'-view');
    el.style.display = 'none';
  });
  const target = v === 'board' ? 'canvas' : v+'-view';
  document.getElementById(target).style.display = 'block';
  document.querySelectorAll('.view-tab').forEach((t,i) => t.classList.toggle('active', views[i] === v));
  if (v === 'calendar') renderCalendar();
  if (v === 'matrix') renderMatrix();
  if (v === 'grades') renderGrades();
}

// ─── BOARD ────────────────────────────────────────────────────────────────────
function renderBoard() {
  const inner = document.getElementById('canvas-inner');
  // remove old groups and add-tile
  inner.querySelectorAll('.class-group, .add-group-tile').forEach(el => el.remove());

  // urgent bar
  const redTasks = state.tasks.filter(t => t.priority === 'red' && !t.done);
  const ub = document.getElementById('urgent-bar');
  if (redTasks.length) {
    ub.style.display = 'flex';
    document.getElementById('urgent-chips').innerHTML = redTasks.map(t =>
      `<span class="urgent-chip" onclick="openEditTask('${t.id}')">${t.title}</span>`).join(' ');
    document.getElementById('canvas').style.top = '86px';
  } else {
    ub.style.display = 'none';
    document.getElementById('canvas').style.top = '52px';
  }

  state.classes.forEach(cls => renderGroup(cls, inner));

  // Add class tile
  const tile = document.createElement('button');
  tile.className = 'add-group-tile';
  tile.textContent = '+ Add class';
  tile.style.left = '1200px';
  tile.style.top = '20px';
  tile.onclick = () => document.getElementById('class-modal').classList.add('open');
  inner.appendChild(tile);
}

function renderGroup(cls, container) {
  const tasks = state.tasks.filter(t => t.classId === cls.id);
  const el = document.createElement('div');
  el.className = 'class-group' + (state.collapsed[cls.id] ? ' collapsed' : '');
  el.style.left = (cls.x || 20) + 'px';
  el.style.top = (cls.y || 20) + 'px';
  el.style.width = (cls.w || 260) + 'px';
  el.dataset.classId = cls.id;

  const sortedTasks = [...tasks].sort((a,b) => {
    if (a.done !== b.done) return a.done ? 1 : -1;
    const pd = {red:0,yellow:1,green:2,'':3};
    if ((pd[a.priority||'']) !== (pd[b.priority||''])) return pd[a.priority||''] - pd[b.priority||''];
    return (a.dueDate||'') < (b.dueDate||'') ? -1 : 1;
  });

  el.innerHTML = `
    <div class="group-header">
      <div class="group-color" style="background:${cls.color}"></div>
      <span class="group-name">${cls.name}</span>
      <span class="group-count">${tasks.filter(t=>!t.done).length}</span>
      <button class="group-collapse-btn" onclick="toggleCollapse('${cls.id}',event)">${state.collapsed[cls.id]?'▶':'▼'}</button>
    </div>
    <div class="group-tasks" id="tasks-${cls.id}"></div>
    <div class="group-footer"><button class="add-task-btn" onclick="openAddTask('${cls.id}')">+ Add task</button></div>
    <div class="resize-handle"></div>`;

  container.appendChild(el);

  const tasksEl = el.querySelector(`#tasks-${cls.id}`);
  sortedTasks.forEach(t => tasksEl.appendChild(makeTaskCard(t)));

  // drag to move
  const header = el.querySelector('.group-header');
  initGroupDrag(el, header, cls);

  // resize
  const rh = el.querySelector('.resize-handle');
  initGroupResize(el, rh, cls);
}

function makeTaskCard(t) {
  const div = document.createElement('div');
  div.className = `task-card${t.priority?' priority-'+t.priority:''}${t.done?' done-card':''}`;
  const typeClass = {hw:'type-hw',test:'type-test',ec:'type-ec'}[t.type]||'type-hw';
  const typeLabel = {hw:'HW',test:'TEST',ec:'EC'}[t.type]||'HW';
  const dateStr = t.dueDate ? new Date(t.dueDate+'T12:00:00').toLocaleDateString('en-US',{month:'short',day:'numeric'}) : '';
  div.innerHTML = `
    <div class="task-top">
      <div class="task-cb${t.done?' checked':''}" onclick="toggleDone('${t.id}',event)">
        ${t.done?'<svg width="9" height="7" viewBox="0 0 9 7" fill="none"><polyline points="1,3.5 3.5,6 8,1" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>':''}
      </div>
      <span class="task-title">${t.title}</span>
    </div>
    <div class="task-meta">
      ${dateStr?`<span class="task-date">${dateStr}</span>`:''}
      <span class="task-type ${typeClass}">${typeLabel}</span>
      ${t.notes?`<span class="task-date">📝</span>`:''}
    </div>
    <div class="task-actions">
      <button class="task-action-btn" onclick="openEditTask('${t.id}')">✏️</button>
    </div>`;
  return div;
}

function toggleDone(id, e) {
  e.stopPropagation();
  const t = state.tasks.find(t=>t.id===id);
  if (t) { t.done = !t.done; save(); renderBoard(); }
}

function toggleCollapse(classId, e) {
  e.stopPropagation();
  state.collapsed[classId] = !state.collapsed[classId];
  save(); renderBoard();
}

// ─── FREE DRAG (position anywhere on canvas) ─────────────────────────────────
function initGroupDrag(el, handle, cls) {
  let startMX, startMY, startEX, startEY, dragging = false;

  handle.addEventListener('mousedown', e => {
    if (e.target.classList.contains('group-collapse-btn')) return;
    e.preventDefault();
    dragging = true;
    startMX = e.clientX;
    startMY = e.clientY;
    startEX = cls.x || 0;
    startEY = cls.y || 0;
    el.classList.add('dragging');
    el.style.zIndex = 500;

    const onMove = ev => {
      if (!dragging) return;
      const canvas = document.getElementById('canvas');
      const scrollX = canvas.scrollLeft;
      const scrollY = canvas.scrollTop;
      const nx = Math.max(0, startEX + (ev.clientX - startMX) + scrollX - scrollX);
      const ny = Math.max(0, startEY + (ev.clientY - startMY));
      // account for canvas scroll
      cls.x = Math.max(0, startEX + ev.clientX - startMX);
      cls.y = Math.max(0, startEY + ev.clientY - startMY);
      el.style.left = cls.x + 'px';
      el.style.top = cls.y + 'px';
    };

    const onUp = () => {
      dragging = false;
      el.classList.remove('dragging');
      el.style.zIndex = '';
      save();
      document.removeEventListener('mousemove', onMove);
      document.removeEventListener('mouseup', onUp);
    };

    document.addEventListener('mousemove', onMove);
    document.addEventListener('mouseup', onUp);
  });
}

// ─── RESIZE ───────────────────────────────────────────────────────────────────
function initGroupResize(el, handle, cls) {
  handle.addEventListener('mousedown', e => {
    e.preventDefault();
    e.stopPropagation();
    const startX = e.clientX;
    const startW = cls.w || 260;

    const onMove = ev => {
      cls.w = Math.max(200, Math.min(600, startW + ev.clientX - startX));
      el.style.width = cls.w + 'px';
    };
    const onUp = () => {
      save();
      document.removeEventListener('mousemove', onMove);
      document.removeEventListener('mouseup', onUp);
    };
    document.addEventListener('mousemove', onMove);
    document.addEventListener('mouseup', onUp);
  });
}

// ─── TASK MODAL ───────────────────────────────────────────────────────────────
let editingTaskId = null, selectedPriority = '';

function populateClassSelect(selectedId) {
  document.getElementById('f-class').innerHTML = state.classes.map(c =>
    `<option value="${c.id}"${c.id===selectedId?' selected':''}>${c.name}</option>`).join('');
}

function openAddTask(classId) {
  editingTaskId = null;
  document.getElementById('modal-title-text').textContent = 'New Task';
  document.getElementById('modal-delete-btn').style.display = 'none';
  ['f-title','f-notes'].forEach(id => document.getElementById(id).value = '');
  document.getElementById('f-date').value = '';
  document.getElementById('f-type').value = 'hw';
  document.getElementById('f-quadrant').value = '';
  selectedPriority = '';
  document.querySelectorAll('.priority-btn').forEach(b => b.classList.toggle('sel', b.dataset.p === ''));
  populateClassSelect(classId || state.classes[0]?.id);
  document.getElementById('task-modal').classList.add('open');
  setTimeout(() => document.getElementById('f-title').focus(), 100);
}

function openEditTask(id) {
  const t = state.tasks.find(t=>t.id===id);
  if (!t) return;
  editingTaskId = id;
  document.getElementById('modal-title-text').textContent = 'Edit Task';
  document.getElementById('modal-delete-btn').style.display = 'block';
  document.getElementById('f-title').value = t.title;
  document.getElementById('f-date').value = t.dueDate || '';
  document.getElementById('f-notes').value = t.notes || '';
  document.getElementById('f-type').value = t.type || 'hw';
  document.getElementById('f-quadrant').value = t.quadrant || '';
  selectedPriority = t.priority || '';
  document.querySelectorAll('.priority-btn').forEach(b => b.classList.toggle('sel', b.dataset.p === selectedPriority));
  populateClassSelect(t.classId);
  document.getElementById('task-modal').classList.add('open');
}

function selPriority(btn) {
  selectedPriority = btn.dataset.p;
  document.querySelectorAll('.priority-btn').forEach(b => b.classList.remove('sel'));
  btn.classList.add('sel');
}

function saveTask() {
  const title = document.getElementById('f-title').value.trim();
  if (!title) { document.getElementById('f-title').focus(); return; }
  const data = {
    title, classId: document.getElementById('f-class').value,
    type: document.getElementById('f-type').value,
    dueDate: document.getElementById('f-date').value,
    notes: document.getElementById('f-notes').value.trim(),
    priority: selectedPriority,
    quadrant: document.getElementById('f-quadrant').value,
  };
  if (editingTaskId) {
    Object.assign(state.tasks.find(t=>t.id===editingTaskId), data);
  } else {
    state.tasks.push({ id: Date.now().toString(), done: false, ...data });
  }
  save(); closeModal(); renderAll();
}

function deleteTask() {
  state.tasks = state.tasks.filter(t=>t.id!==editingTaskId);
  save(); closeModal(); renderAll();
}

function closeModal() { document.getElementById('task-modal').classList.remove('open'); }
document.getElementById('task-modal').addEventListener('click', e => { if (e.target===e.currentTarget) closeModal(); });

// ─── ADD CLASS ────────────────────────────────────────────────────────────────
function saveClass() {
  const name = document.getElementById('cf-name').value.trim();
  if (!name) return;
  const color = document.getElementById('cf-color').value;
  const id = Date.now().toString();
  state.classes.push({ id, name, color, x: 20 + Math.random()*400, y: 20 + Math.random()*200, w: 260 });
  if (!state.grades[id]) state.grades[id] = [];
  save();
  document.getElementById('class-modal').classList.remove('open');
  document.getElementById('cf-name').value = '';
  renderAll();
}
document.getElementById('class-modal').addEventListener('click', e => { if (e.target===e.currentTarget) document.getElementById('class-modal').classList.remove('open'); });

// ─── CALENDAR ─────────────────────────────────────────────────────────────────
let calYear = new Date().getFullYear(), calMonth = new Date().getMonth();
function calNav(dir) { calMonth += dir; if (calMonth>11){calMonth=0;calYear++;}if(calMonth<0){calMonth=11;calYear--;} renderCalendar(); }
function renderCalendar() {
  const months=['January','February','March','April','May','June','July','August','September','October','November','December'];
  document.getElementById('cal-title').textContent = months[calMonth]+' '+calYear;
  const grid = document.getElementById('cal-grid');
  grid.innerHTML = ['Sun','Mon','Tue','Wed','Thu','Fri','Sat'].map(d=>`<div class="cal-dow">${d}</div>`).join('');
  const first = new Date(calYear,calMonth,1).getDay();
  const days = new Date(calYear,calMonth+1,0).getDate();
  const today = new Date();
  for (let i=0;i<first;i++) {
    const d = new Date(calYear,calMonth,-first+i+1);
    grid.innerHTML += `<div class="cal-cell other-month"><div class="cal-day-num">${d.getDate()}</div></div>`;
  }
  for (let d=1;d<=days;d++) {
    const ds = `${calYear}-${String(calMonth+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
    const isToday = today.getFullYear()===calYear&&today.getMonth()===calMonth&&today.getDate()===d;
    const cell = document.createElement('div');
    cell.className = 'cal-cell'+(isToday?' today':'');
    cell.innerHTML = `<div class="cal-day-num">${d}</div>`;
    state.tasks.filter(t=>t.dueDate===ds&&!t.done).forEach(t=>{
      const cls = state.classes.find(c=>c.id===t.classId);
      const chip = document.createElement('div');
      chip.className='cal-task-chip';
      chip.style.borderColor=cls?.color||'#888';
      chip.style.background=(cls?.color||'#888')+'18';
      chip.style.color=cls?.color||'#444';
      chip.textContent=t.title; chip.title=t.title;
      chip.onclick=()=>openEditTask(t.id);
      cell.appendChild(chip);
    });
    grid.appendChild(cell);
  }
}

// ─── MATRIX ───────────────────────────────────────────────────────────────────
function renderMatrix() {
  ['q1','q2','q3','q4'].forEach(q => {
    const el = document.getElementById('m-'+q);
    el.innerHTML = '';
    const tasks = state.tasks.filter(t=>t.quadrant===q&&!t.done);
    if (!tasks.length){el.innerHTML=`<div style="font-size:12px;color:var(--text3);padding:8px 0;">No tasks</div>`;return;}
    tasks.forEach(t=>{
      const cls=state.classes.find(c=>c.id===t.classId);
      const chip=document.createElement('div');
      chip.style.cssText=`padding:8px 10px;border-radius:7px;margin-bottom:6px;border:1px solid var(--border);background:var(--surface);cursor:pointer;font-size:13px;display:flex;gap:8px;align-items:center;`;
      chip.innerHTML=`<span style="width:8px;height:8px;border-radius:50%;background:${cls?.color||'#888'};flex-shrink:0;display:inline-block;"></span>${t.title}${t.dueDate?`<span style="margin-left:auto;font-size:11px;color:var(--text3);font-family:var(--mono)">${new Date(t.dueDate+'T12:00:00').toLocaleDateString('en-US',{month:'short',day:'numeric'})}</span>`:''}`;
      chip.onclick=()=>openEditTask(t.id);
      el.appendChild(chip);
    });
  });
}

// ─── GRADES ───────────────────────────────────────────────────────────────────
let gradeTarget = null;
function renderGrades() {
  const grid = document.getElementById('grades-grid');
  grid.innerHTML = '';
  state.classes.forEach(cls=>{
    const entries = state.grades[cls.id]||[];
    const avg = entries.length?Math.round(entries.reduce((s,e)=>s+(e.score/e.total*100),0)/entries.length):null;
    const card = document.createElement('div');
    card.className='grade-card';
    card.innerHTML=`
      <div class="grade-card-header"><div class="grade-dot" style="background:${cls.color}"></div><span class="grade-class-name">${cls.name}</span></div>
      <div>${entries.map((e,i)=>`<div class="grade-entry"><span class="grade-entry-name">${e.name}</span><span class="grade-entry-score" style="color:${e.score/e.total>=.9?'var(--green)':e.score/e.total>=.7?'var(--yellow)':'var(--red)'}">${e.score}/${e.total}</span><button onclick="deleteGrade('${cls.id}',${i})" style="border:none;background:none;color:var(--text3);cursor:pointer;font-size:12px;">✕</button></div>`).join('')}</div>
      ${avg!==null?`<div class="grade-avg">Average <span>${avg}%</span></div>`:''}
      <button class="grade-add-btn" onclick="openGradeModal('${cls.id}','${cls.name}')">+ Add grade</button>`;
    grid.appendChild(card);
  });
}
function openGradeModal(classId,className){
  gradeTarget=classId;
  document.getElementById('grade-modal-title').textContent=`Add Grade — ${className}`;
  ['gf-name','gf-score','gf-total'].forEach(id=>document.getElementById(id).value='');
  document.getElementById('grade-modal').classList.add('open');
  setTimeout(()=>document.getElementById('gf-name').focus(),100);
}
document.getElementById('grade-modal').addEventListener('click',e=>{if(e.target===e.currentTarget)document.getElementById('grade-modal').classList.remove('open');});
function saveGrade(){
  const name=document.getElementById('gf-name').value.trim();
  const score=parseFloat(document.getElementById('gf-score').value);
  const total=parseFloat(document.getElementById('gf-total').value);
  if(!name||isNaN(score)||isNaN(total)||total===0)return;
  if(!state.grades[gradeTarget])state.grades[gradeTarget]=[];
  state.grades[gradeTarget].push({name,score,total});
  save();document.getElementById('grade-modal').classList.remove('open');renderGrades();
}
function deleteGrade(classId,idx){state.grades[classId].splice(idx,1);save();renderGrades();}

// ─── ICS EXPORT ───────────────────────────────────────────────────────────────
function exportICS(){
  const tasks=state.tasks.filter(t=>t.dueDate&&!t.done);
  if(!tasks.length){alert('No tasks with due dates to export.');return;}
  let ics=`BEGIN:VCALENDAR\r\nVERSION:2.0\r\nPRODID:-//Santiago Planner//EN\r\nCALSCALE:GREGORIAN\r\n`;
  tasks.forEach(t=>{
    const cls=state.classes.find(c=>c.id===t.classId);
    const d=t.dueDate.replace(/-/g,'');
    ics+=`BEGIN:VEVENT\r\nUID:${t.id}@planner\r\nDTSTAMP:${new Date().toISOString().replace(/[-:]/g,'').split('.')[0]}Z\r\nDTSTART;VALUE=DATE:${d}\r\nDTEND;VALUE=DATE:${d}\r\nSUMMARY:${(cls?.name||'')+': '+t.title}\r\nDESCRIPTION:${(t.notes||'').replace(/\n/g,'\\n')}\r\nEND:VEVENT\r\n`;
  });
  ics+=`END:VCALENDAR`;
  const a=document.createElement('a');
  a.href=URL.createObjectURL(new Blob([ics],{type:'text/calendar'}));
  a.download='santiago_planner.ics';a.click();
}

// ─── RENDER ALL ───────────────────────────────────────────────────────────────
function renderAll(){
  renderBoard();
  if(currentView==='calendar')renderCalendar();
  if(currentView==='matrix')renderMatrix();
  if(currentView==='grades')renderGrades();
}

document.addEventListener('keydown',e=>{
  if(e.key==='Escape'){closeModal();['class-modal','grade-modal'].forEach(id=>document.getElementById(id).classList.remove('open'));}
  if((e.metaKey||e.ctrlKey)&&e.key==='k'){e.preventDefault();openAddTask();}
});

renderAll();
</script>
</body>
</html>
