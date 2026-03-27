<style>
    :root {
        --bg: #0f172a;
        --card: #1e293b;
        --accent: #38bdf8;
        --success: #4ad66d;
        --gold: #fbbf24;
        --danger: #ef4444;
        --text-muted: #94a3b8;
    }

    .game-wrapper {
        font-family: 'Segoe UI', system-ui, sans-serif;
        background-color: var(--bg);
        color: white;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 400px;
        padding: 20px;
    }

    .container {
        background: var(--card);
        padding: 2rem;
        border-radius: 1.5rem;
        box-shadow: 0 20px 50px rgba(0,0,0,0.5);
        width: 100%;
        max-width: 500px;
        text-align: center;
        border: 1px solid #334155;
    }

    .hidden { display: none; }

    .stats-grid {
        display: grid;
        grid-template-columns: 1fr 1fr 1fr;
        gap: 10px;
        margin-bottom: 20px;
    }

    .stat-box {
        background: #0f172a;
        padding: 10px;
        border-radius: 12px;
        border: 1px solid #334155;
    }

    .stat-label { font-size: 0.65rem; color: var(--text-muted); text-transform: uppercase; }
    .stat-val { display: block; font-size: 1.1rem; font-weight: bold; color: var(--accent); }

    .question { font-size: 4.5rem; font-weight: 900; margin: 20px 0; min-height: 100px; }
    
    .math-input {
        background: #0f172a;
        border: 3px solid #334155;
        color: var(--gold);
        font-size: 2.5rem;
        width: 150px;
        text-align: center;
        padding: 10px;
        border-radius: 1rem;
        outline: none;
    }
    .math-input:focus { border-color: var(--accent); }

    .btn {
        background: var(--accent);
        color: #0f172a;
        border: none;
        padding: 15px 25px;
        border-radius: 10px;
        font-weight: bold;
        cursor: pointer;
        width: 100%;
        font-size: 1rem;
        margin-top: 10px;
    }

    .analysis-list { 
        text-align: left; 
        font-size: 0.85rem; 
        max-height: 200px; 
        overflow-y: auto; 
        margin: 15px 0; 
        padding: 15px; 
        background: #0f172a; 
        border-radius: 8px; 
    }
    .fail-item { color: var(--danger); margin-bottom: 5px; border-left: 3px solid var(--danger); padding-left: 8px; }
    .slow-item { color: var(--gold); margin-bottom: 5px; border-left: 3px solid var(--gold); padding-left: 8px; }
</style>

<div class="game-wrapper">
    <div class="container">
        <div id="start-screen">
            <h2 style="color: var(--accent)">Gange-Galskap 🧠</h2>
            <p style="color: var(--text-muted)">Mål: Fullfør 10 oppgaver med færrest mulig klikk.</p>
            
            <div style="margin: 30px 0; background: #0f172a; padding: 20px; border-radius: 12px;">
                <span style="display:block; font-size: 0.8rem; color: var(--text-muted); margin-bottom: 15px;">VELG NIVÅ:</span>
                <label style="cursor:pointer;"><input type="radio" name="lvl" value="1" checked> Nivå 1 (1-5)</label>
                <label style="margin-left: 20px; cursor:pointer;"><input type="radio" name="lvl" value="2"> Nivå 2 (5-10)</label>
            </div>
            
            <button class="btn" onclick="startApp()">START TRENING</button>
        </div>

        <div id="game-screen" class="hidden">
            <div class="stats-grid">
                <div class="stat-box"><span class="stat-label">Klikk/Tast</span><span class="stat-val" id="count-clicks">0</span></div>
                <div class="stat-box"><span class="stat-label">Feil</span><span class="stat-val" id="count-errors" style="color: var(--danger)">0</span></div>
                <div class="stat-box"><span class="stat-label">Tid</span><span class="stat-val" id="count-timer">0.0s</span></div>
            </div>
            <div class="question" id="q-display">? × ?</div>
            <input type="number" id="ans-input" class="math-input" autocomplete="off">
            <p id="feedback" style="height: 20px; margin-top: 15px; font-weight: bold;"></p>
        </div>

        <div id="result-screen" class="hidden">
            <h2 style="color: var(--gold)">Runde Ferdig!</h2>
            <div class="stats-grid">
                <div class="stat-box"><span class="stat-label">Total Klikk</span><span class="stat-val" id="res-clicks">0</span></div>
                <div class="stat-box"><span class="stat-label">Total Tid</span><span class="stat-val" id="res-time">0s</span></div>
                <div class="stat-box"><span class="stat-label">Feil</span><span class="stat-val" id="res-errors">0</span></div>
            </div>
            
            <div class="analysis-list" id="analysis"></div>

            <button class="btn" onclick="location.reload()">PRØV IGJEN</button>
        </div>
    </div>
</div>

<script>
    let level = 1, qCount = 0, clicks = 0, errors = 0;
    let currentAns = 0, currentQ = "", startTime = 0, qStartTime = 0;
    let results = [];
    const maxQ = 10;

    const ansInp = document.getElementById('ans-input');

    function startApp() {
        level = parseInt(document.querySelector('input[name="lvl"]:checked').value);
        document.getElementById('start-screen').classList.add('hidden');
        document.getElementById('game-screen').classList.remove('hidden');
        initGame();
    }

    function initGame() {
        startTime = Date.now();
        nextQuestion();
        setInterval(() => {
            if(qCount <= maxQ && startTime > 0) {
                document.getElementById('count-timer').innerText = ((Date.now() - startTime)/1000).toFixed(1) + "s";
            }
        }, 100);
    }

    function nextQuestion() {
        if (qCount >= maxQ) return finish();
        qCount++;
        
        let n1 = (level === 1) ? Math.floor(Math.random() * 5) + 1 : Math.floor(Math.random() * 6) + 5;
        let n2 = Math.floor(Math.random() * 10) + 1;
        
        currentAns = n1 * n2;
        currentQ = `${n1} × ${n2}`;
        document.getElementById('q-display').innerText = currentQ;
        ansInp.value = "";
        ansInp.focus();
        qStartTime = Date.now();
    }

    ansInp.addEventListener('keydown', (e) => {
        if(e.key !== "Enter" && e.key !== "Backspace" && e.key !== "Tab" && e.key !== "Shift") {
            clicks++;
            document.getElementById('count-clicks').innerText = clicks;
        }
    });

    ansInp.addEventListener('input', () => {
        if(ansInp.value.length >= currentAns.toString().length) {
            if(parseInt(ansInp.value) === currentAns) {
                results.push({ q: currentQ, time: (Date.now()-qStartTime)/1000, error: false });
                nextQuestion();
            } else {
                errors++;
                document.getElementById('count-errors').innerText = errors;
                results.push({ q: currentQ, time: 0, error: true });
                ansInp.value = "";
                document.getElementById('feedback').innerText = "Feil!";
                document.getElementById('feedback').style.color = "#ef4444";
                setTimeout(() => document.getElementById('feedback').innerText = "", 600);
            }
        }
    });

    function finish() {
        const totalTime = ((Date.now() - startTime) / 1000).toFixed(1);
        startTime = 0; 
        document.getElementById('game-screen').classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        
        document.getElementById('res-clicks').innerText = clicks;
        document.getElementById('res-time').innerText = totalTime + "s";
        document.getElementById('res-errors').innerText = errors;

        const ana = document.getElementById('analysis');
        ana.innerHTML = "<b>Oppgaver å øve mer på:</b><br><br>";
        
        let hardTasks = results.filter(r => r.error || r.time > 3);
        
        if(hardTasks.length === 0) {
            ana.innerHTML += "✨ Ingen! Du svarte raskt og riktig på alt.";
        } else {
            hardTasks.forEach(r => {
                ana.innerHTML += `<div class="${r.error ? 'fail-item' : 'slow-item'}">
                    ${r.error ? '❌ BOM:' : '⏳ TREG:'} ${r.q} = ${eval(r.q.replace('×','*'))}
                </div>`;
            });
        }
    }
</script>
