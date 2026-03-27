<style>
    :root {
        --bg: #0f172a;
        --card: #1e293b;
        --accent: #38bdf8;
        --gold: #fbbf24;
        --danger: #ef4444;
        --text-muted: #94a3b8;
    }

    /* Fyller hele skjermen */
    .game-viewport {
        font-family: 'Segoe UI', system-ui, sans-serif;
        background-color: var(--bg);
        color: white;
        margin: 0;
        height: 100vh;
        width: 100vw;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        overflow: hidden;
        position: fixed;
        top: 0;
        left: 0;
    }

    .hidden { display: none !important; }

    /* Stats i toppen */
    .top-stats {
        position: absolute;
        top: 40px;
        display: flex;
        gap: 40px;
        background: rgba(30, 41, 59, 0.5);
        padding: 15px 40px;
        border-radius: 50px;
        border: 1px solid #334155;
    }

    .stat-item { text-align: center; }
    .stat-label { font-size: 0.7rem; color: var(--text-muted); text-transform: uppercase; display: block; }
    .stat-val { font-size: 1.5rem; font-weight: bold; color: var(--accent); }

    /* Selve mattestykket */
    .main-content {
        text-align: center;
        width: 100%;
        max-width: 800px;
    }

    .question { 
        font-size: 10rem; 
        font-weight: 900; 
        margin: 0; 
        line-height: 1;
        letter-spacing: -5px;
        text-shadow: 0 10px 30px rgba(0,0,0,0.5);
    }

    .input-wrapper { margin-top: 40px; }
    
    .math-input {
        background: transparent;
        border: none;
        border-bottom: 4px solid #334155;
        color: var(--gold);
        font-size: 6rem;
        width: 300px;
        text-align: center;
        outline: none;
        transition: border-color 0.3s;
        font-weight: bold;
    }
    .math-input:focus { border-color: var(--accent); }

    /* Start og Slutt skjermer */
    .overlay-content {
        background: var(--card);
        padding: 4rem;
        border-radius: 3rem;
        border: 2px solid #334155;
        box-shadow: 0 0 100px rgba(0,0,0,0.8);
        max-width: 600px;
        width: 90%;
    }

    .btn {
        background: var(--accent);
        color: #0f172a;
        border: none;
        padding: 20px 50px;
        border-radius: 15px;
        font-weight: 900;
        cursor: pointer;
        font-size: 1.5rem;
        text-transform: uppercase;
        transition: transform 0.2s, background 0.2s;
    }
    .btn:hover { transform: scale(1.05); background: #7dd3fc; }

    .analysis-list { 
        text-align: left; 
        background: #0f172a;
        padding: 20px;
        border-radius: 15px;
        margin: 20px 0;
        max-height: 200px;
        overflow-y: auto;
    }

    /* Animasjon ved feil */
    .shake { animation: shake 0.4s; }
    @keyframes shake {
        0%, 100% { transform: translateX(0); }
        25% { transform: translateX(-10px); }
        75% { transform: translateX(10px); }
    }
</style>

<div class="game-viewport">
    
    <div id="start-screen" class="overlay-content">
        <h1 style="font-size: 3rem; margin-top: 0; color: var(--accent);">Gange-Galskap</h1>
        <p style="font-size: 1.2rem; color: var(--text-muted);">Svar på 10 stykker så raskt som mulig med færrest mulig tastetrykk.</p>
        
        <div style="margin: 40px 0;">
            <p style="margin-bottom: 20px; font-weight: bold;">VELG UTFORDRING:</p>
            <label style="font-size: 1.5rem; cursor:pointer;"><input type="radio" name="lvl" value="1" checked> 1-5 gangen</label>
            <br><br>
            <label style="font-size: 1.5rem; cursor:pointer;"><input type="radio" name="lvl" value="2"> 5-10 gangen</label>
        </div>
        
        <button class="btn" onclick="startApp()">START SPILLET</button>
    </div>

    <div id="game-screen" class="main-content hidden">
        <div class="top-stats">
            <div class="stat-item"><span class="stat-label">Klikk</span><span class="stat-val" id="count-clicks">0</span></div>
            <div class="stat-item"><span class="stat-label">Feil</span><span class="stat-val" id="count-errors" style="color:var(--danger)">0</span></div>
            <div class="stat-item"><span class="stat-label">Tid</span><span class="stat-val" id="count-timer">0.0s</span></div>
        </div>
        
        <div class="question" id="q-display">? × ?</div>
        
        <div class="input-wrapper">
            <input type="number" id="ans-input" class="math-input" placeholder="?">
        </div>
    </div>

    <div id="result-screen" class="overlay-content hidden">
        <h2 style="font-size: 2.5rem; color: var(--gold); margin: 0;">Ferdig!</h2>
        <div style="display: flex; justify-content: space-around; margin: 30px 0;">
            <div><span class="stat-label">Klikk</span><span class="stat-val" id="res-clicks" style="font-size: 2rem;">0</span></div>
            <div><span class="stat-label">Tid</span><span class="stat-val" id="res-time" style="font-size: 2rem;">0s</span></div>
        </div>
        
        <div class="analysis-list" id="analysis"></div>

        <button class="btn" onclick="location.reload()">PRØV IGJEN</button>
    </div>

</div>

<script>
    let level = 1, qCount = 0, clicks = 0, errors = 0;
    let currentAns = 0, currentQ = "", startTime = 0, qStartTime = 0;
    let results = [];
    const maxQ = 10;

    const ansInp = document.getElementById('ans-input');
    const gameScreen = document.getElementById('game-screen');

    function startApp() {
        level = parseInt(document.querySelector('input[name="lvl"]:checked').value);
        document.getElementById('start-screen').classList.add('hidden');
        gameScreen.classList.remove('hidden');
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
        if(!["Enter", "Backspace", "Tab", "Shift"].includes(e.key)) {
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
                
                // Rist skjermen ved feil
                gameScreen.classList.add('shake');
                setTimeout(() => gameScreen.classList.remove('shake'), 400);
            }
        }
    });

    function finish() {
        const totalTime = ((Date.now() - startTime) / 1000).toFixed(1);
        startTime = 0; 
        gameScreen.classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        
        document.getElementById('res-clicks').innerText = clicks;
        document.getElementById('res-time').innerText = totalTime + "s";

        const ana = document.getElementById('analysis');
        ana.innerHTML = "<b style='color:var(--accent)'>Analyse:</b><br><br>";
        
        let hardTasks = results.filter(r => r.error || r.time > 2.5);
        if(hardTasks.length === 0) {
            ana.innerHTML += "🌟 Alt satt perfekt!";
        } else {
            hardTasks.forEach(r => {
                ana.innerHTML += `<div style="color:${r.error ? 'var(--danger)' : 'var(--gold)'}; margin-bottom:5px;">
                    ${r.error ? '❌ Feil på' : '⏳ Litt treg på'} ${r.q} = ${eval(r.q.replace('×','*'))}
                </div>`;
            });
        }
    }
</script>
