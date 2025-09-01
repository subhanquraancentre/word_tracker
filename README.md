# word_tracker<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Word Tracker — Weekly & Daily</title>
<style>
  body{font-family:Arial,Helvetica,sans-serif;background:#f3f5f7;margin:0;padding:18px;color:#222}
  .wrap{max-width:980px;margin:0 auto;background:#fff;padding:18px;border-radius:10px;box-shadow:0 6px 20px rgba(0,0,0,0.06)}
  h1{margin:0 0 8px;font-size:20px}
  .subtitle{color:#555;font-size:13px;margin-bottom:14px}
  .cards{display:grid;grid-template-columns:repeat(2,1fr);gap:12px}
  @media(max-width:720px){.cards{grid-template-columns:1fr}}
  .card{background:#fbfcff;border-radius:10px;padding:12px;border:1px solid #e9eef6}
  .wordTitle{display:flex;justify-content:space-between;align-items:center;font-weight:700;margin-bottom:6px}
  .count{font-size:36px;color:#0b3b82;font-weight:700;margin:8px 0}
  .controls{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
  button{cursor:pointer;border:0;border-radius:8px;padding:10px 12px;font-weight:700}
  .add{background:#0b5cff;color:#fff}
  .sub{background:#f0f4ff}
  .reset{background:#ff6b6b;color:#fff}
  input[type=number]{width:90px;padding:8px;border-radius:8px;border:1px solid #d6dbe8}
  .top{display:flex;justify-content:space-between;align-items:center;gap:8px;flex-wrap:wrap;margin-bottom:12px}
  .top .left{display:flex;flex-direction:column}
  .top .right{display:flex;gap:8px;align-items:center}
  .summary{margin-top:14px;background:#fff;padding:12px;border-radius:10px;border:1px solid #eef2f8}
  table{width:100%;border-collapse:collapse;margin-top:10px}
  table th, table td{border:1px solid #e6ebf2;padding:6px;text-align:center}
  .small{padding:8px 10px;border-radius:8px;background:#fff;border:1px solid #d6dbe8}
</style>
</head>
<body>
  <div class="wrap">
    <div class="top">
      <div class="left">
        <h1>Word Tracker — Weekly & Daily</h1>
        <div class="subtitle">Words: Jazakallah, Mashaallah, Alhamdulillah, Inshallah. Click buttons to record pronunciations. Data saved in your browser.</div>
      </div>
      <div class="right">
        <label>Week: </label>
        <select id="weekSelect" class="small"></select>
        <button id="showWeek" class="small">Show</button>
        <button id="exportWeek" class="small">Export Week CSV</button>
        <button id="exportAll" class="small">Export All CSV</button>
        <button id="resetAll" class="small" style="background:#ff6b6b;color:#fff">Reset All</button>
      </div>
    </div>

    <div class="cards" id="cards">
      <!-- cards injected by JS -->
    </div>

    <div class="summary" id="summary">
      <strong>Weekly Summary</strong>
      <div id="summaryContent" style="margin-top:8px">Select a week to view summary.</div>
    </div>

    <div class="summary" style="margin-top:12px">
      <strong>Daily Chart (This week)</strong>
      <div id="dailyChart"></div>
    </div>

  </div>

<script>
(function(){
  // Words config (English + Urdu label)
  const words = [
    {key:'jazakallah', label:'Jazakallah — جزاک اللہ'},
    {key:'mashaallah', label:'Mashaallah — ماشاء اللہ'},
    {key:'alhamdulillah', label:'Alhamdulillah — الحمدللہ'},
    {key:'inshallah', label:'Inshallah — انشاء اللہ'}
  ];

  const STORAGE_KEY = 'word-tracker-weekly-v2';

  // ISO week key helper (YYYY-W##)
  function getISOWeekKey(date = new Date()){
    const d = new Date(Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()));
    d.setUTCDate(d.getUTCDate() + 4 - (d.getUTCDay()||7));
    const yearStart = new Date(Date.UTC(d.getUTCFullYear(),0,1));
    const weekNo = Math.ceil((((d - yearStart)/86400000) + 1)/7);
    return d.getUTCFullYear() + '-W' + String(weekNo).padStart(2,'0');
  }

  // Format date YYYY-MM-DD
  function fmtDate(date){ return date.toISOString().split('T')[0]; }

  // Storage helpers
  function loadData(){ try{ return JSON.parse(localStorage.getItem(STORAGE_KEY) || '{}'); }catch(e){ return {}; } }
  function saveData(d){ localStorage.setItem(STORAGE_KEY, JSON.stringify(d)); }

  // Ensure structure: data.days[YYYY-MM-DD] -> {word:count}
  function ensureDay(data, day){ if(!data[day]){ data[day] = {}; words.forEach(w=> data[day][w.key]=0); } }

  // Build UI cards
  const cardsEl = document.getElementById('cards');
  function buildCards(){
    cardsEl.innerHTML = '';
    words.forEach(w=>{
      const div = document.createElement('div'); div.className='card';
      div.innerHTML = `
        <div class="wordTitle"><div>${w.label}</div><div style="font-size:12px;color:#666">This week's count</div></div>
        <div class="count" data-key="${w.key}">0</div>
        <div class="controls">
          <button class="add" data-action="inc" data-key="${w.key}">+ Add</button>
          <button class="sub" data-action="dec" data-key="${w.key}">− Subtract</button>
          <input type="number" class="step small" data-key="${w.key}" value="1" min="1" />
          <button class="reset" data-action="reset" data-key="${w.key}">Reset</button>
        </div>
        <div style="margin-top:8px;font-size:12px;color:#555">Enter amount then press + to add that many counts</div>
      `;
      cardsEl.appendChild(div);
    });
    attachCardEvents();
  }

  // Attach button events
  function attachCardEvents(){
    cardsEl.querySelectorAll('button').forEach(btn=>{
      btn.addEventListener('click', (e)=>{
        const action = btn.getAttribute('data-action');
        const key = btn.getAttribute('data-key');
        const stepInput = cardsEl.querySelector('input.step[data-key="'+key+'"]');
        const step = Math.max(1, parseInt(stepInput.value || '1', 10));
        const weekKey = document.getElementById('weekSelect').value || getISOWeekKey();
        const data = loadData();
        // store by weekKey -> days -> date -> word counts
        if(!data[weekKey]) data[weekKey] = {};
        // current day: use today's date (local)
        const today = new Date();
        const dayStr = fmtDate(today);
        if(!data[weekKey][dayStr]) { data[weekKey][dayStr] = {}; words.forEach(w=> data[weekKey][dayStr][w.key]=0); }
        if(action === 'inc') data[weekKey][dayStr][key] = (data[weekKey][dayStr][key]||0) + step;
        if(action === 'dec') data[weekKey][dayStr][key] = Math.max(0, (data[weekKey][dayStr][key]||0) - step);
        if(action === 'reset') data[weekKey][dayStr][key] = 0;
        saveData(data);
        renderWeek(weekKey);
      });
    });
  }

  // Populate week select with existing weeks and current week on top
  function populateWeekSelect(){
    const sel = document.getElementById('weekSelect');
    const data = loadData();
    const keys = Object.keys(data);
    const current = getISOWeekKey();
    const unique = Array.from(new Set([current, ...keys])).sort().reverse();
    sel.innerHTML = '';
    unique.forEach(k=>{
      const opt = document.createElement('option'); opt.value = k; opt.textContent = k + (k===current ? ' (current)' : '');
      sel.appendChild(opt);
    });
    if(!sel.value) sel.value = current;
  }

  // Render summary and daily chart for a week
  function renderWeek(weekKey){
    const data = loadData();
    // find dates that belong to this ISO week (approx): we will consider days stored under this weekKey in storage
    const wk = data[weekKey] || {};
    // Collect sorted days (if no days yet, create today)
    const days = Object.keys(wk).sort();
    // If none, ensure current day exists
    if(days.length === 0){
      const today = fmtDate(new Date());
      data[weekKey] = data[weekKey] || {};
      data[weekKey][today] = {};
      words.forEach(w=> data[weekKey][today][w.key]=0);
      saveData(data);
    }
    // Recompute days list
    const dayList = Object.keys(data[weekKey]).sort();
    // Weekly totals
    const totals = {};
    words.forEach(w=> totals[w.key]=0);
    dayList.forEach(d=>{
      words.forEach(w => totals[w.key] += (data[weekKey][d][w.key]||0));
    });

    // Update card counts (show sum across week)
    words.forEach(w=>{
      const el = document.querySelector('.count[data-key="'+w.key+'"]');
      if(el) el.textContent = totals[w.key] || 0;
    });

    // Summary text
    const summaryDiv = document.getElementById('summaryContent');
    let html = '<div style="margin-top:6px">';
    words.forEach(w=>{
      html += <div style="margin-bottom:6px"><strong>${w.label.split(' — ')[0]}:</strong> ${totals[w.key] || 0}</div>;
    });
    const grand = Object.values(totals).reduce((s,n)=>s+n,0);
    html += <div style="margin-top:8px"><strong>Total:</strong> ${grand}</div>;
    html += '</div>';
    summaryDiv.innerHTML = html;

    // Daily chart (table) — show each date in week stored (if missing days within week, show 0)
    // We'll try to show 7 days: find one date in week then compute 7-day window around it.
    // Simpler: if dayList available, show sorted dayList; otherwise show current week days
    let table = '<table><tr><th>Date</th>';
    words.forEach(w=> table += <th>${w.label.split(' — ')[0]}</th>);
    table += '</tr>';

    // To present a consistent week view: build 7-day window from today - (todayWeekday) to +6
    const today = new Date();
    const dayOfWeek = today.getDay(); // Sunday=0
    const start = new Date(today); start.setDate(today.getDate() - dayOfWeek); // Sunday
    for(let i=0;i<7;i++){
      const d = new Date(start); d.setDate(start.getDate() + i);
      const ds = fmtDate(d);
      table += <tr><td>${ds}</td>;
      words.forEach(w=>{
        const val = (data[weekKey] && data[weekKey][ds] && data[weekKey][ds][w.key]) ? data[weekKey][ds][w.key] : 0;
        table += <td>${val}</td>;
      });
      table += '</tr>';
    }
    table += '</table>';
    document.getElementById('dailyChart').innerHTML = table;

    // repopulate weeks select in case new week created
    populateWeekSelect();
  }

  // Export CSV for week or all
  function exportWeekCSV(weekKey){
    const data = loadData();
    const wk = data[weekKey] || {};
    const rows = [['Date','Word','Count']];
    Object.keys(wk).sort().forEach(d=>{
      words.forEach(w=>{
        rows.push([d, w.label.split(' — ')[0], wk[d][w.key] || 0]);
      });
    });
    downloadCSV(rows, 'week-' + weekKey + '-tracker.csv');
  }
  function exportAllCSV(){
    const data = loadData();
    const rows = [['Week','Date','Word','Count']];
    Object.keys(data).sort().forEach(weekKey=>{
      const wk = data[weekKey];
      Object.keys(wk).sort().forEach(d=>{
        words.forEach(w=>{
          rows.push([weekKey, d, w.label.split(' — ')[0], wk[d][w.key]||0]);
        });
      });
    });
    downloadCSV(rows, 'all-weeks-word-tracker.csv');
  }
  function downloadCSV(rows, filename){
    const csv = rows.map(r => r.map(c => '"' + String(c).replace(/"/g,'""') + '"').join(',')).join('\\n');
    const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a'); a.href = url; a.download = filename; document.body.appendChild(a); a.click(); a.remove();
    setTimeout(()=> URL.revokeObjectURL(url), 500);
  }

  // Reset entire storage
  function resetAll(){
    if(!confirm('Reset all saved data? This will delete all weeks.')) return;
    localStorage.removeItem(STORAGE_KEY);
    populateWeekSelect();
    renderWeek(document.getElementById('weekSelect').value || getISOWeekKey());
  }

  // Initialize page
  function init(){
    buildCards();
    populateWeekSelect();
    // ensure current week exists in storage
    const data = loadData();
    const curr = getISOWeekKey();
    if(!data[curr]) data[curr] = {};
    // ensure today's day exists under current week
    const todayStr = fmtDate(new Date());
    if(!data[curr][todayStr]){ data[curr][todayStr] = {}; words.forEach(w=> data[curr][todayStr][w.key]=0); }
    saveData(data);
    renderWeek(curr);

    // Hook buttons
    document.getElementById('showWeek').addEventListener('click', ()=> {
      renderWeek(document.getElementById('weekSelect').value || getISOWeekKey());
    });
    document.getElementById('exportWeek').addEventListener('click', ()=> {
      exportWeekCSV(document.getElementById('weekSelect').value || getISOWeekKey());
    });
    document.getElementById('exportAll').addEventListener('click', exportAllCSV);
    document.getElementById('resetAll').addEventListener('click', resetAll);
  }

  // run
  init();

})();
</script>
</body>
</html>
