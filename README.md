<style>
    :root {
        --bg: #0f172a;
        --card: #1e293b;
        --accent: #38bdf8;
        --gold: #fbbf24;
        --danger: #ef4444;
        --text-muted: #94a3b8;
        --success: #4ad66d;
    }

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

    .question { 
        font-size: 8rem; 
        font-weight: 900; 
        margin: 0; 
        line-height: 1.2;
        text-shadow: 0 10px 30px rgba(0,0,0,0.5);
    }

    .math-input {
        background: transparent;
        border: none;
        border-bottom: 4px solid #334155;
        color: var(--gold);
        font-size: 5rem;
        width: 300px;
        text-align: center;
        outline: none;
        font-weight: bold;
    }

    .overlay-content {
        background: var(--card);
        padding: 3rem;
        border-radius: 2rem;
        border: 2px solid #334155;
        max-width: 650px;
        width: 90%;
        text-align: center;
    }

    .btn {
        background: var(--accent);
        color: #0f172a;
        border: none;
        padding: 15px 30px;
        border-radius: 12px;
        font-weight: 900;
        cursor: pointer;
        font-size: 1.2rem;
        text-transform: uppercase;
        margin: 10px;
    }

    .btn-practice { background: var(--gold); }

    .step-indicator {
        color: var(--accent);
        font-weight: bold;
        text-transform: uppercase;
        letter-spacing: 2px;
        margin-bottom: 20px;
        display: block;
    }

    .shake { animation: shake 0.4s; }
    @keyframes shake {
        0%, 100% { transform: translateX(0); }
        25% { transform: translateX(-10px); }
        75% { transform: translateX(10px); }
    }
</style>

<div class="game-viewport">
    
    <div id="start-screen" class="overlay-content">
        <h1 style="color: var(--accent);">Gange-Galskap</h1>
        <p>10 raske oppgaver. Bruk så få tastetrykk som mulig.</p>
        <div style="margin: 30px 0;">
            <label><input type="radio" name="lvl" value="1" checked> 1-5 gangen</label>
            <label style="margin-left: 20px;"><input type="radio" name="lvl" value="2"> 5-10 gangen</label>
        </div>
        <button class="btn" onclick="startApp()">Start Spill</button>
    </div>

    <div id="game-screen" class="hidden" style="text-align:center;">
        <div class="top-stats" id="stats-bar">
            <div class="stat-item"><span class="stat-label">Klikk</span><span class="stat-val" id="count-clicks">0</span></div>
            <div class="stat-item"><span class="stat-label">Feil</span><span class="stat-val" id="count-errors" style="color:var(--danger)">0</span></div>
            <div class="stat-item"><span class="stat-label">Tid</span><span class="stat-val" id="count-timer">0.0s</span></div>
        </div>
        
        <span id="practice-step" class="step-indicator hidden">Steg 1: Se svaret</span>
        <div class="question" id="q-display">? × ?</div>
        <div id="help-text" style="font-size: 2rem; color: var(--success); margin-bottom: 20px;" class="hidden">Svaret er X</div>
        
        <input type="number" id="ans-input" class="math-input" placeholder="?">
        <p id="instruction" style="color: var(--text-muted); margin-top: 20px;"></p>
    </div>

    <div id="result-screen" class="overlay-content hidden">
        <h2 id="res-title">Ferdig!</h2>
        <p id="res-summary"></p>
        <div id="analysis" style="text-align:left; background:#0f172a; padding:15px; border-radius:10px; margin:20px 0;"></div>
        
        <button id="btn-practice" class="btn btn-practice hidden" onclick="startPractice()">Øv på feil (Feilfri læring)</button>
        <button class="btn" onclick="location.reload()">Ny runde</button>
    </div>

</div>

<script>
    let level = 1, qCount = 0, clicks = 0, errors = 0;
    let currentAns = 0, currentQ = "", startTime = 0, qStartTime = 0;
    let results = [], failedTasks = [];
    let isPractice = false, practiceIndex = 0, practiceStep = 1;
    const maxQ = 10;

    const ansInp = document.getElementById('ans-input');

    function startApp() {
        level = parseInt(document.querySelector('input[name="lvl"]:checked').value);
        document.getElementById('start-screen').classList.add('hidden');
        document.getElementById('game-screen').classList.remove('hidden');
        isPractice = false;
        initGame();
    }

    function initGame() {
        qCount = 0; clicks = 0; errors = 0; results = [];
        startTime = Date.now();
        nextQuestion();
        setInterval(updateTimer, 100);
    }

    function updateTimer() {
        if(startTime > 0 && !isPractice) {
            document.getElementById('count-timer').innerText = ((Date.now() - startTime)/1000).toFixed(1) + "s";
        }
    }

    function nextQuestion() {
        if (qCount >= maxQ) return finish();
        qCount++;
        let n1 = (level === 1) ? Math.floor(Math.random()*5)+1 : Math.floor(Math.random()*6)+5;
        let n2 = Math.floor(Math.random()*10)+1;
        currentAns = n1 * n2;
        currentQ = `${n1} × ${n2}`;
        document.getElementById('q-display').innerText = currentQ;
        ansInp.value = "";
        ansInp.focus();
        qStartTime = Date.now();
    }

    ansInp.addEventListener('keydown', (e) => {
        if(!["Enter", "Backspace", "Tab"].includes(e.key) && !isPractice) {
            clicks++;
            document.getElementById('count-clicks').innerText = clicks;
        }
        if(e.key === "Enter" && isPractice && practiceStep === 1) {
            goToStep2();
        }
    });

    ansInp.addEventListener('input', () => {
        if(isPractice) handlePracticeInput();
        else handleGameInput();
    });

    function handleGameInput() {
        if(ansInp.value.length >= currentAns.toString().length) {
            if(parseInt(ansInp.value) === currentAns) {
                results.push({ q: currentQ, a: currentAns, error: false });
                nextQuestion();
            } else {
                errors++;
                document.getElementById('count-errors').innerText = errors;
                if(!failedTasks.some(t => t.q === currentQ)) failedTasks.push({q: currentQ, a: currentAns});
                ansInp.value = "";
                document.getElementById('game-screen').classList.add('shake');
                setTimeout(() => document.getElementById('game-screen').classList.remove('shake'), 400);
            }
        }
    }

    function finish() {
        startTime = 0;
        document.getElementById('game-screen').classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        document.getElementById('res-summary').innerText = `Du brukte ${clicks} klikk på ${maxQ} oppgaver.`;
        
        const ana = document.getElementById('analysis');
        ana.innerHTML = failedTasks.length > 0 ? "<b>Oppgaver du bør øve på:</b><br>" : "<b>Perfekt! Ingen feil.</b>";
        failedTasks.forEach(t => ana.innerHTML += `<div>❌ ${t.q} = ${t.a}</div>`);
        
        if(failedTasks.length > 0) document.getElementById('btn-practice').classList.remove('hidden');
    }

    // FEILFRI LÆRING LOGIKK
    function startPractice() {
        isPractice = true;
        practiceIndex = 0;
        document.getElementById('result-screen').classList.add('hidden');
        document.getElementById('game-screen').classList.remove('hidden');
        document.getElementById('stats-bar').classList.add('hidden');
        document.getElementById('practice-step').classList.remove('hidden');
        runPracticeStep();
    }

    function runPracticeStep() {
        let task = failedTasks[practiceIndex];
        currentAns = task.a;
        document.getElementById('q-display').innerText = task.q;
        ansInp.value = "";
        ansInp.focus();

        if(practiceStep === 1) {
            document.getElementById('practice-step').innerText = "Steg 1: Se og lær";
            document.getElementById('help-text').classList.remove('hidden');
            document.getElementById('help-text').innerText = `Svaret er ${task.a}`;
            document.getElementById('instruction').innerText = "Trykk på Enter for å gå videre";
            ansInp.style.display = "none";
        } else if(practiceStep === 2) {
            document.getElementById('practice-step').innerText = "Steg 2: Avskrift";
            document.getElementById('help-text').innerText = `Skriv: ${task.a}`;
            ansInp.style.display = "inline-block";
            document.getElementById('instruction').innerText = "";
        } else {
            document.getElementById('practice-step').innerText = "Steg 3: Prøv selv";
            document.getElementById('help-text').classList.add('hidden');
            ansInp.style.display = "inline-block";
        }
    }

    function goToStep2() {
        practiceStep = 2;
        runPracticeStep();
    }

    function handlePracticeInput() {
        if(ansInp.value.length >= currentAns.toString().length) {
            if(parseInt(ansInp.value) === currentAns) {
                if(practiceStep === 2) {
                    practiceStep = 3;
                    setTimeout(runPracticeStep, 300);
                } else if(practiceStep === 3) {
                    practiceIndex++;
                    if(practiceIndex < failedTasks.length) {
                        practiceStep = 1;
                        setTimeout(runPracticeStep, 300);
                    } else {
                        alert("Bra jobba! Du har fullført øvingsrunden.");
                        location.reload();
                    }
                }
            } else {
                ansInp.value = ""; // I feilfri læring tvinger vi riktig svar med en gang
            }
        }
    }
</script>
