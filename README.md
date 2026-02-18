# steuer-tracker




Actual complete functional code 

<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Steuer-Tracker Pro</title>
    
    <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2280%22>⚖️</text></svg>">
    
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>

    <style>
        :root {
            --primary: #4a90e2; --bg: #f4f6f8; --card-bg: #ffffff;
            --text: #333; --danger: #e74c3c; --success: #27ae60;
            --accent: #f39c12; --gold: #f1c40f;
        }

        body { font-family: sans-serif; background-color: var(--bg); color: var(--text); margin: 0; padding-bottom: 80px; }
        header { background-color: var(--primary); color: white; padding: 15px; text-align: center; border-radius: 0 0 20px 20px; }
        
        .info-stack { display: flex; flex-direction: column; gap: 10px; padding: 15px; }
        .info-box { background: white; padding: 12px; border-radius: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); text-align: center; }
        .info-box h4 { margin: 0 0 5px 0; font-size: 10px; color: var(--primary); text-transform: uppercase; }
        .info-box p { margin: 0; font-weight: bold; font-size: 14px; }

        .dashboard-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; padding: 0 15px 15px 15px; }
        .subject-card { background: white; padding: 10px; border-radius: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .progress-bar { background: #eee; height: 6px; border-radius: 3px; margin: 8px 0; overflow: hidden; }
        .progress-fill { height: 100%; transition: width 0.5s; }
        .gold-fill { background: linear-gradient(90deg, #f1c40f, #f39c12) !important; box-shadow: 0 0 5px var(--gold); }

        .stats-grid { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 8px; margin-bottom: 15px; }
        .stat-tile { background: #f8f9fa; padding: 8px; border-radius: 8px; text-align: center; font-size: 12px; }

        .bottom-nav { position: fixed; bottom: 0; width: 100%; background: white; display: flex; justify-content: space-around; padding: 10px 0; border-top: 1px solid #ddd; z-index: 100; }
        .nav-item { color: #888; font-size: 10px; cursor: pointer; text-align: center; flex: 1; }
        .nav-item.active { color: var(--primary); font-weight: bold; }
        
        .tracker-input { margin: 15px; padding: 15px; background: white; border-radius: 12px; }
        input, select, textarea { width: 100%; padding: 10px; margin: 5px 0; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; font-size: 16px; }
        .btn { width: 100%; padding: 12px; border: none; border-radius: 6px; font-weight: bold; cursor: pointer; margin-top: 5px; }
        .hidden { display: none; }

        .log-item, .note-item { background: white; padding: 10px; border-radius: 8px; margin-bottom: 10px; border-left: 4px solid var(--primary); display: flex; justify-content: space-between; align-items: center; }
        .action-btns { display: flex; gap: 10px; }
        .btn-icon { background: none; border: none; font-size: 18px; cursor: pointer; }
    </style>
</head>
<body>

    <header>
        <h2 style="margin:0;">Steuer-Tracker Pro</h2>
        <div id="streak-display" style="font-size:12px; margin-top:5px;">🔥 Streak: 0 Tage</div>
    </header>

    <div id="view-dash">
        <div class="info-stack">
            <div class="info-box"><h4>⏳ Tage bis zum 04.04.2026</h4><p id="countdown">Berechne...</p></div>
            <div class="info-box">
                <h4>📈 Gesamtfortschritt</h4>
                <p id="total-label">0h / 0h</p>
                <div class="progress-bar" style="height:10px;"><div id="total-fill" class="progress-fill" style="background:var(--primary);"></div></div>
            </div>
            <div class="info-box"><h4>✨ Motivation</h4><p id="quote" style="font-style:italic; font-weight:normal; font-size:12px;">Lade...</p></div>
        </div>
        <div class="dashboard-grid" id="main-grid"></div>
        <div class="tracker-input">
            <h4 style="margin:0 0 10px 0;">Neues Fach ➕</h4>
            <input type="text" id="new-sub-name" placeholder="z.B. UmwStR">
            <button class="btn" style="background:var(--primary); color:white;" onclick="addNewSubject()">Hinzufügen</button>
        </div>
    </div>

    <div id="view-timer" class="hidden">
        <div class="tracker-input">
            <div class="stats-grid">
                <div class="stat-tile"><b id="s-today">0h</b><br>Heute</div>
                <div class="stat-tile"><b id="s-sub">0h</b><br>Fach</div>
                <div class="stat-tile"><b id="s-rest">0h</b><br>Rest</div>
            </div>
            <select id="timer-sub" onchange="updateTimerStats()"></select>
            <div id="timer-display" style="font-size:48px; text-align:center; font-weight:bold; margin:20px 0;">25:00</div>
            <button class="btn" style="background:var(--primary); color:white;" onclick="startTimer(25)">25 Min Fokus</button>
            <button class="btn" style="background:var(--danger); color:white;" onclick="resetTimer()">Reset</button>
            <hr>
            <input type="number" id="manual-h" placeholder="Stunden eintragen" step="0.1">
            <button id="btn-save-manual" class="btn" style="background:#eee;" onclick="saveManual()">Eintragen</button>
        </div>
        <div style="padding: 0 15px;">
            <h4>Letzte Buchungen bearbeiten</h4>
            <div id="log-list"></div>
        </div>
    </div>

    <div id="view-notes" class="hidden">
        <div class="tracker-input">
            <h4 id="note-edit-title">Neue Notiz 📝</h4>
            <input type="text" id="n-title" placeholder="Thema / Titel">
            <textarea id="n-body" placeholder="Wichtige Merkposten..."></textarea>
            <button id="btn-save-note" class="btn" style="background:var(--primary); color:white;" onclick="addNote()">Speichern</button>
            <button id="btn-cancel-note" class="btn hidden" style="background:#ccc;" onclick="cancelNoteEdit()">Abbrechen</button>
        </div>
        <div id="notes-list" style="padding:15px;"></div>
    </div>

    <div id="view-charts" class="hidden">
        <div class="info-box" style="margin:15px;"><canvas id="chart1"></canvas></div>
        <div class="info-box" style="margin:15px;"><canvas id="chart2"></canvas></div>
    </div>

    <div id="view-settings" class="hidden">
        <div class="tracker-input" id="goal-editor"></div>
        <div class="tracker-input">
            <button class="btn" style="background:var(--danger); color:white;" onclick="if(confirm('Alle Daten löschen?')) { localStorage.clear(); location.reload(); }">Komplett-Reset</button>
        </div>
    </div>

    <nav class="bottom-nav">
        <div class="nav-item active" onclick="show('dash')">🏠<br>Home</div>
        <div class="nav-item" onclick="show('timer')">⏱️<br>Timer</div>
        <div class="nav-item" onclick="show('notes')">📝<br>Notizen</div>
        <div class="nav-item" onclick="show('charts')">📊<br>Analyse</div>
        <div class="nav-item" onclick="show('settings')">⚙️<br>Settings</div>
    </nav>

    <script>
        const colors = { "ESt": "#27ae60", "USt": "#e67e22", "BilSt": "#3498db", "AO": "#9b59b6", "BewR": "#e74c3c", "GesR": "#2c3e50", "PrivR": "#e91e63" };
        const quotes = ["Kopf hoch, Paragrafen beißen nicht! ⚖️", "Jede Minute zählt.", "Fokus an, Welt aus. ✨"];
        
        let data = JSON.parse(localStorage.getItem('st_data')) || {
            "ESt": {h:0, g:10, t:0}, "USt": {h:0, g:8, t:0}, "BilSt": {h:0, g:10, t:0}, "AO": {h:0, g:5, t:0}
        };
        let notes = JSON.parse(localStorage.getItem('st_notes')) || [];
        let logs = JSON.parse(localStorage.getItem('st_logs')) || [];
        let tInt, editNoteId = null;

        function init() {
            updateCD(); updateQuote(); renderDash(); updateDrops(); renderNotes(); renderGoalEdit(); renderLogs();
            Chart.register(ChartDataLabels);
        }

        function show(v) {
            ['dash','timer','notes','charts','settings'].forEach(id => document.getElementById('view-'+id).classList.add('hidden'));
            document.getElementById('view-'+v).classList.remove('hidden');
            document.querySelectorAll('.nav-item').forEach((n,i) => n.classList.toggle('active', ['dash','timer','notes','charts','settings'][i] === v));
            if(v === 'charts') renderCharts();
            if(v === 'timer') renderLogs();
        }

        function updateCD() {
            const diff = new Date("2026-04-04") - new Date();
            document.getElementById('countdown').innerText = `Noch ${Math.ceil(diff/86400000)} Tage`;
        }

        function updateQuote() { document.getElementById('quote').innerText = quotes[Math.floor(Math.random()*quotes.length)]; }

        function renderDash() {
            const grid = document.getElementById('main-grid'); grid.innerHTML = '';
            let sH=0, sG=0;
            Object.entries(data).forEach(([n, d]) => {
                sH+=d.h; sG+=d.g; const p = (d.h/d.g)*100;
                grid.innerHTML += `<div class="subject-card"><b>${n}</b><div class="progress-bar"><div class="progress-fill ${p>=100?'gold-fill':''}" style="width:${Math.min(p,100)}%; background:${colors[n]||'#888'}"></div></div><small>${d.h.toFixed(1)}/${d.g}h</small></div>`;
            });
            document.getElementById('total-label').innerText = `${sH.toFixed(1)}h / ${sG}h`;
            document.getElementById('total-fill').style.width = Math.min((sH/sG)*100, 100)+'%';
            localStorage.setItem('st_data', JSON.stringify(data));
        }

        function addTime(s, h) { 
            data[s].h += h; data[s].t += h; 
            logs.unshift({id: Date.now(), sub: s, h: h, date: new Date().toLocaleDateString()});
            localStorage.setItem('st_logs', JSON.stringify(logs));
            renderDash(); renderLogs(); updateTimerStats();
        }

        function saveManual() { 
            const s=document.getElementById('timer-sub').value, h=parseFloat(document.getElementById('manual-h').value); 
            if(h>0) { addTime(s,h); document.getElementById('manual-h').value=''; } 
        }

        function editLog(id) {
            const log = logs.find(l => l.id === id);
            document.getElementById('timer-sub').value = log.sub;
            document.getElementById('manual-h').value = log.h;
            deleteLog(id, true); // Stilles Löschen zum Bearbeiten
            document.getElementById('btn-save-manual').innerText = "Änderung speichern";
        }

        function deleteLog(id, silent = false) {
            if(!silent && !confirm('Löschen?')) return;
            const idx = logs.findIndex(l => l.id === id);
            const log = logs[idx];
            data[log.sub].h = Math.max(0, data[log.sub].h - log.h);
            logs.splice(idx, 1);
            localStorage.setItem('st_logs', JSON.stringify(logs));
            renderDash(); renderLogs(); updateTimerStats();
        }

        function renderLogs() {
            document.getElementById('log-list').innerHTML = logs.slice(0,8).map(l => `
                <div class="log-item">
                    <span><b>${l.sub}</b>: ${l.h.toFixed(1)}h</span>
                    <div class="action-btns">
                        <button class="btn-icon" onclick="editLog(${l.id})">✏️</button>
                        <button class="btn-icon" onclick="deleteLog(${l.id})">🗑️</button>
                    </div>
                </div>`).join('');
        }

        function addNote() {
            const t=document.getElementById('n-title').value, b=document.getElementById('n-body').value;
            if(!t || !b) return;
            if(editNoteId) {
                const idx = notes.findIndex(n => n.id === editNoteId);
                notes[idx] = {id: editNoteId, t, b};
                editNoteId = null;
                document.getElementById('note-edit-title').innerText = "Neue Notiz 📝";
                document.getElementById('btn-cancel-note').classList.add('hidden');
            } else {
                notes.unshift({id: Date.now(), t, b});
            }
            localStorage.setItem('st_notes', JSON.stringify(notes));
            renderNotes();
            document.getElementById('n-title').value=''; document.getElementById('n-body').value='';
        }

        function editNote(id) {
            const n = notes.find(note => note.id === id);
            document.getElementById('n-title').value = n.t;
            document.getElementById('n-body').value = n.b;
            editNoteId = id;
            document.getElementById('note-edit-title').innerText = "Notiz bearbeiten ✍️";
            document.getElementById('btn-cancel-note').classList.remove('hidden');
            window.scrollTo(0,0);
        }

        function cancelNoteEdit() {
            editNoteId = null;
            document.getElementById('n-title').value=''; document.getElementById('n-body').value='';
            document.getElementById('note-edit-title').innerText = "Neue Notiz 📝";
            document.getElementById('btn-cancel-note').classList.add('hidden');
        }

        function deleteNote(id) {
            if(confirm('Notiz löschen?')) {
                notes = notes.filter(n => n.id !== id);
                localStorage.setItem('st_notes', JSON.stringify(notes));
                renderNotes();
            }
        }

        function renderNotes() { 
            document.getElementById('notes-list').innerHTML = notes.map(n=>`
                <div class="note-item">
                    <div style="width:75%"><b>${n.t}</b><br><small>${n.b}</small></div>
                    <div class="action-btns">
                        <button class="btn-icon" onclick="editNote(${n.id})">✏️</button>
                        <button class="btn-icon" onclick="deleteNote(${n.id})">🗑️</button>
                    </div>
                </div>`).join(''); 
        }

        function startTimer(m) {
            clearInterval(tInt); let s = m*60;
            tInt = setInterval(() => {
                s--; let min = Math.floor(s/60), sec = s%60;
                document.getElementById('timer-display').innerText = `${min}:${sec<10?'0':''}${sec}`;
                if(s<=0) { clearInterval(tInt); addTime(document.getElementById('timer-sub').value, m/60); alert("Zeit um!"); }
            }, 1000);
        }
        function resetTimer() { clearInterval(tInt); document.getElementById('timer-display').innerText = "25:00"; }
        function updateTimerStats() {
            const s = document.getElementById('timer-sub').value;
            document.getElementById('s-sub').innerText = data[s].h.toFixed(1)+'h';
            document.getElementById('s-rest').innerText = Math.max(0, data[s].g - data[s].h).toFixed(1)+'h';
        }
        function renderCharts() {
            const lbs = Object.keys(data), vals = lbs.map(l=>data[l].h), cls = lbs.map(l=>colors[l] || '#888');
            new Chart(document.getElementById('chart1'), { type:'pie', data:{labels:lbs, datasets:[{data:vals, backgroundColor:cls}]}, options:{plugins:{datalabels:{color:'#fff'}}}});
        }
        function renderGoalEdit() {
            const e = document.getElementById('goal-editor'); e.innerHTML = '<h4>Ziele & Fächer</h4>';
            Object.entries(data).forEach(([n,d]) => { 
                e.innerHTML += `<div style="display:flex; justify-content:space-between; margin-bottom:10px;">
                    <span>${n}</span><input type="number" style="width:60px;" value="${d.g}" onchange="data['${n}'].g=parseFloat(this.value); renderDash();">
                </div>`; 
            });
        }
        function updateDrops() { document.getElementById('timer-sub').innerHTML = Object.keys(data).map(s=>`<option value="${s}">${s}</option>`).join(''); }
        function addNewSubject() { 
            const n=document.getElementById('new-sub-name').value; 
            if(n && !data[n]){ data[n]={h:0,g:10,t:0}; document.getElementById('new-sub-name').value=''; renderDash(); updateDrops(); renderGoalEdit(); } 
        }
        init();
    </script>
</body>
</html>
