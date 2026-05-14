<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Task Board</title>
  <link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Mono:wght@300;400;500&display=swap" rel="stylesheet"/>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg: #0e0e10;
      --surface: #16161a;
      --surface2: #1e1e24;
      --border: #1d1dc8;
      --accent: #13ba24;
      --accent2: #ff6b6b;
      --text: #f0f0f5;
      --muted: #6b6b80;
      --done-text: #44444f;
    }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: 'DM Mono', monospace;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 48px 16px 80px;
    }

    /* Ambient glow */
    body::before {
      content: '';
      position: fixed;
      top: -200px; left: 50%;
      transform: translateX(-50%);
      width: 700px; height: 400px;
      background: radial-gradient(ellipse, rgba(200,240,74,0.07) 0%, transparent 70%);
      pointer-events: none;
      z-index: 0;
    }

    .container {
      width: 100%;
      max-width: 560px;
      position: relative;
      z-index: 1;
    }

    /* Header */
    header {
      margin-bottom: 40px;
    }
    .header-tag {
      font-size: 11px;
      letter-spacing: 0.2em;
      color: var(--accent);
      text-transform: uppercase;
      margin-bottom: 8px;
    }
    h1 {
      font-family: 'Syne', sans-serif;
      font-size: 42px;
      font-weight: 800;
      line-height: 1;
      letter-spacing: -1px;
      color: var(--text);
    }
    h1 span {
      color: var(--accent);
    }
    .stats {
      margin-top: 12px;
      font-size: 12px;
      color: var(--muted);
      display: flex;
      gap: 20px;
    }
    .stats b { color: var(--text); }

    /* Input area */
    .input-area {
      display: flex;
      gap: 10px;
      margin-bottom: 32px;
    }
    .input-wrap {
      flex: 1;
      position: relative;
    }
    .input-wrap input {
      width: 100%;
      background: var(--surface);
      border: 1px solid var(--border);
      color: var(--text);
      font-family: 'DM Mono', monospace;
      font-size: 14px;
      padding: 14px 16px;
      border-radius: 10px;
      outline: none;
      transition: border-color 0.2s, box-shadow 0.2s;
    }
    .input-wrap input::placeholder { color: var(--muted); }
    .input-wrap input:focus {
      border-color: var(--accent);
      box-shadow: 0 0 0 3px rgba(200,240,74,0.08);
    }
    .add-btn {
      background: var(--accent);
      color: #0e0e10;
      border: none;
      font-family: 'Syne', sans-serif;
      font-size: 14px;
      font-weight: 700;
      padding: 0 22px;
      border-radius: 10px;
      cursor: pointer;
      letter-spacing: 0.02em;
      transition: transform 0.15s, background 0.15s;
      flex-shrink: 0;
    }
    .add-btn:hover { background: #d8ff52; transform: translateY(-1px); }
    .add-btn:active { transform: translateY(0px); }

    /* Filters */
    .filters {
      display: flex;
      gap: 6px;
      margin-bottom: 20px;
    }
    .filter-btn {
      background: transparent;
      border: 1px solid var(--border);
      color: var(--muted);
      font-family: 'DM Mono', monospace;
      font-size: 11px;
      padding: 6px 14px;
      border-radius: 20px;
      cursor: pointer;
      transition: all 0.15s;
      letter-spacing: 0.05em;
    }
    .filter-btn:hover { border-color: var(--accent); color: var(--text); }
    .filter-btn.active {
      background: var(--accent);
      border-color: var(--accent);
      color: #0e0e10;
      font-weight: 500;
    }

    /* Task list */
    #task-list { list-style: none; display: flex; flex-direction: column; gap: 8px; }

    .task-item {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 12px;
      display: flex;
      align-items: center;
      gap: 14px;
      padding: 14px 16px;
      animation: slideIn 0.25s ease;
      transition: border-color 0.2s, transform 0.15s;
    }
    .task-item:hover { border-color: #3a3a48; transform: translateX(2px); }
    .task-item.done {
      border-color: transparent;
      background: var(--surface2);
    }

    @keyframes slideIn {
      from { opacity: 0; transform: translateY(-10px); }
      to   { opacity: 1; transform: translateY(0); }
    }
    @keyframes fadeOut {
      from { opacity: 1; transform: scale(1); }
      to   { opacity: 0; transform: scale(0.95); }
    }
    .task-item.removing { animation: fadeOut 0.2s ease forwards; }

    /* Checkbox */
    .check-btn {
      width: 22px; height: 22px;
      border-radius: 6px;
      border: 2px solid var(--border);
      background: transparent;
      cursor: pointer;
      flex-shrink: 0;
      position: relative;
      transition: border-color 0.2s, background 0.2s;
    }
    .check-btn:hover { border-color: var(--accent); }
    .task-item.done .check-btn {
      background: var(--accent);
      border-color: var(--accent);
    }
    .check-btn::after {
      content: '';
      position: absolute;
      top: 50%; left: 50%;
      transform: translate(-50%, -50%) scale(0);
      width: 10px; height: 7px;
      border-left: 2px solid #0e0e10;
      border-bottom: 2px solid #0e0e10;
      rotate: -45deg;
      margin-top: -1px;
      transition: transform 0.15s;
    }
    .task-item.done .check-btn::after { transform: translate(-50%, -50%) scale(1); }

    /* Task text */
    .task-text {
      flex: 1;
      font-size: 14px;
      color: var(--text);
      line-height: 1.4;
      word-break: break-word;
      transition: color 0.2s;
    }
    .task-item.done .task-text {
      color: var(--done-text);
      text-decoration: line-through;
      text-decoration-color: #3a3a48;
    }

    /* Done badge */
    .done-badge {
      font-size: 10px;
      letter-spacing: 0.1em;
      padding: 3px 8px;
      border-radius: 20px;
      background: rgba(200,240,74,0.1);
      color: var(--accent);
      display: none;
      flex-shrink: 0;
    }
    .task-item.done .done-badge { display: block; }

    /* Delete btn */
    .del-btn {
      background: transparent;
      border: none;
      color: var(--muted);
      cursor: pointer;
      font-size: 18px;
      line-height: 1;
      padding: 0 2px;
      flex-shrink: 0;
      transition: color 0.15s, transform 0.15s;
    }
    .del-btn:hover { color: var(--accent2); transform: scale(1.2); }

    /* Empty state */
    .empty {
      text-align: center;
      padding: 60px 0;
      color: var(--muted);
      font-size: 13px;
      display: none;
    }
    .empty-icon { font-size: 36px; margin-bottom: 12px; opacity: 0.4; }
    .empty.visible { display: block; }

    /* Clear done */
    .clear-btn {
      background: transparent;
      border: 1px solid var(--border);
      color: var(--muted);
      font-family: 'DM Mono', monospace;
      font-size: 11px;
      padding: 8px 16px;
      border-radius: 8px;
      cursor: pointer;
      margin-top: 20px;
      width: 100%;
      transition: all 0.15s;
    }
    .clear-btn:hover { border-color: var(--accent2); color: var(--accent2); }

    /* Progress bar */
    .progress-wrap {
      margin-bottom: 28px;
    }
    .progress-label {
      font-size: 11px;
      color: var(--muted);
      margin-bottom: 6px;
      display: flex;
      justify-content: space-between;
    }
    .progress-bar {
      height: 4px;
      background: var(--surface2);
      border-radius: 4px;
      overflow: hidden;
    }
    .progress-fill {
      height: 100%;
      background: var(--accent);
      border-radius: 4px;
      transition: width 0.4s cubic-bezier(.4,0,.2,1);
      width: 0%;
    }
  </style>
</head>
<body>
<div class="container">
  <header>
    <div class="header-tag">// task_board.js</div>
    <h1>Get <span>Things</span><br>Done.</h1>
    <div class="stats">
      <span><b id="total-count">0</b> total</span>
      <span><b id="done-count">0</b> completed</span>
      <span><b id="pending-count">0</b> pending</span>
    </div>
  </header>

  <div class="input-area">
    <div class="input-wrap">
      <input type="text" id="task-input" placeholder="Add a new task..." maxlength="120"/>
    </div>
    <button class="add-btn" onclick="addTask()">+ ADD</button>
  </div>

  <div class="progress-wrap">
    <div class="progress-label">
      <span>Progress</span>
      <span id="progress-pct">0%</span>
    </div>
    <div class="progress-bar"><div class="progress-fill" id="progress-fill"></div></div>
  </div>

  <div class="filters">
    <button class="filter-btn active" onclick="setFilter('all', this)">All</button>
    <button class="filter-btn" onclick="setFilter('pending', this)">Pending</button>
    <button class="filter-btn" onclick="setFilter('done', this)">Completed</button>
  </div>

  <ul id="task-list"></ul>
  <div class="empty" id="empty-state">
    <div class="empty-icon">✦</div>
    <div>No tasks here.</div>
  </div>

  <button class="clear-btn" onclick="clearDone()">Clear completed tasks</button>
</div>

<script>
  let tasks = JSON.parse(localStorage.getItem('tasks_v1') || '[]');
  let filter = 'all';

  function save() {
    localStorage.setItem('tasks_v1', JSON.stringify(tasks));
  }

  function render() {
    const list = document.getElementById('task-list');
    const empty = document.getElementById('empty-state');
    list.innerHTML = '';

    const filtered = tasks.filter(t =>
      filter === 'all' ? true :
      filter === 'pending' ? !t.done :
      t.done
    );

    if (filtered.length === 0) {
      empty.classList.add('visible');
    } else {
      empty.classList.remove('visible');
    }

    filtered.forEach(t => {
      const li = document.createElement('li');
      li.className = 'task-item' + (t.done ? ' done' : '');
      li.dataset.id = t.id;
      li.innerHTML = `
        <button class="check-btn" onclick="toggleDone('${t.id}')" title="Mark complete"></button>
        <span class="task-text">${escHtml(t.text)}</span>
        <span class="done-badge">DONE</span>
        <button class="del-btn" onclick="deleteTask('${t.id}')" title="Delete">×</button>
      `;
      list.appendChild(li);
    });

    updateStats();
  }

  function updateStats() {
    const total = tasks.length;
    const done = tasks.filter(t => t.done).length;
    const pending = total - done;
    document.getElementById('total-count').textContent = total;
    document.getElementById('done-count').textContent = done;
    document.getElementById('pending-count').textContent = pending;

    const pct = total === 0 ? 0 : Math.round((done / total) * 100);
    document.getElementById('progress-fill').style.width = pct + '%';
    document.getElementById('progress-pct').textContent = pct + '%';
  }

  function addTask() {
    const input = document.getElementById('task-input');
    const text = input.value.trim();
    if (!text) { input.focus(); return; }
    tasks.unshift({ id: Date.now().toString(), text, done: false });
    save();
    input.value = '';
    render();
    input.focus();
  }

  function toggleDone(id) {
    const t = tasks.find(t => t.id === id);
    if (t) { t.done = !t.done; save(); render(); }
  }

  function deleteTask(id) {
    const el = document.querySelector(`[data-id="${id}"]`);
    if (el) {
      el.classList.add('removing');
      el.addEventListener('animationend', () => {
        tasks = tasks.filter(t => t.id !== id);
        save(); render();
      }, { once: true });
    }
  }

  function clearDone() {
    tasks = tasks.filter(t => !t.done);
    save(); render();
  }

  function setFilter(f, btn) {
    filter = f;
    document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    render();
  }

  function escHtml(s) {
    return s.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
  }

  document.getElementById('task-input').addEventListener('keydown', e => {
    if (e.key === 'Enter') addTask();
  });

  render();
</script>
</body>
</html>
