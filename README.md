<!DOCTYPE html>
<html lang="no">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gange-Galskap</title>
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

        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            background-color: var(--bg);
        }

        .game-viewport {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--bg);
            color: white;
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
            font-size: clamp(4rem, 15vw, 8rem); 
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
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
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
            transition: transform 0.1s;
        }

        .btn:active { transform: scale(0.95); }

        .shake { animation: shake 0.4s; }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            25% { transform: translateX(-10px); }
            75% { transform: translateX(10px); }
        }
    </style>
</head>
<body>

<div class="game-viewport">
    
    <div id="start-screen" class="overlay-content">
        <h1 style="color: var(--accent); margin-top: 0;">Gange-Galskap</h1>
        <p>10 raske oppgaver. Velg nivå:</p>
        <div style="margin: 30px 0; font-size: 1.1rem; display: flex; flex-direction: column; gap: 15px; align-items: center;">
            <label style="cursor: pointer;"><input type="radio" name="lvl" value="0"> 1-3 gangen (Enkel)</label>
            <label style="cursor: pointer;"><input type="radio" name="lvl" value="1" checked> 1-5 gangen (Medium)</label>
            <label style="cursor: pointer;"><input type="radio" name="lvl" value="2"> 5-10 gangen (Vanskelig)</label>
        </div>
        <button class="btn" onclick="startApp()">Start Spill</button>
    </div>

    <div id="game-screen" class="hidden" style="text-align:center;">
        <div class="top-stats">
            <div class="stat-item"><span class="stat-label">Klikk</span><span class="stat-val" id="count-clicks">0</span></div>
            <div class="stat-item"><span class="stat-label">Feil</span><span class="stat-val" id="count-errors" style="color:var(--danger)">0</span></div>
            <div class="stat-item"><span class="stat-label">Tid</span><span class="stat-val" id="count-timer">0.0s</span></div>
        </div>
        
        <div class="question" id="q-display">? × ?</div>
        <input type="number" id="ans-input" class="math-input" placeholder="?" autocomplete="off">
    </div>

    <div id="result-screen" class="overlay-content hidden">
        <h2 style="margin-top:0">Ferdig!</h2>
        <p id="res-summary"></p>
        <div id="analysis" style="text-align:left; background:#0f172a; padding:15px; border-radius:10px; margin:20px 0; max-height: 200px; overflow-y: auto;"></div>
        
        <button class="btn" onclick="location.reload()">Ny runde</button>
    </div>

</div>

<script>
    let level = 1, qCount = 0, clicks = 0, errors = 0;
    let currentAns = 0, currentQ = "", startTime = 0;
    let failedTasks = [];
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
        qCount = 0; clicks = 0; errors = 0; failedTasks = [];
        startTime = Date.now();
        nextQuestion();
        setInterval(updateTimer, 100);
    }

    function updateTimer() {
        if(startTime > 0) {
            document.getElementById('count-timer').innerText = ((Date.now() - startTime)/1000).toFixed(1) + "s";
        }
    }

    function nextQuestion() {
        if (qCount >= maxQ) return finish();
        qCount++;
        
        let n1, n2;
        if(level === 0) {
            // Begge tall mellom 1 og 3
            n1 = Math.floor(Math.random() * 3) + 1;
            n2 = Math.floor(Math.random() * 3) + 1;
        } else if(level === 1) {
            // Begge tall mellom 1 og 5
            n1 = Math.floor(Math.random() * 5) + 1;
            n2 = Math.floor(Math.random() * 5) + 1;
        } else {
            // Begge tall mellom 5 og 10
            n1 = Math.floor(Math.random() * 6) + 5;
            n2 = Math.floor(Math.random() * 6) + 5;
        }

        currentAns = n1 * n2;
        currentQ = `${n1} × ${n2}`;
        document.getElementById('q-display').innerText = currentQ;
        ansInp.value = "";
        ansInp.focus();
    }

    ansInp.addEventListener('keydown', (e) => {
        if(!["Enter", "Backspace", "Tab", "Shift", "Alt", "Control"].includes(e.key)) {
            clicks++;
            document.getElementById('count-clicks').innerText = clicks;
        }
    });

    ansInp.addEventListener('input', () => {
        const val = ansInp.value;
        if(val.length >= currentAns.toString().length) {
            if(parseInt(val) === currentAns) {
                setTimeout(nextQuestion, 100);
            } else {
                errors++;
                document.getElementById('count-errors').innerText = errors;
                if(!failedTasks.some(t => t.q === currentQ)) failedTasks.push({q: currentQ, a: currentAns});
                
                ansInp.value = "";
                gameScreen.classList.add('shake');
                setTimeout(() => gameScreen.classList.remove('shake'), 400);
            }
        }
    });

    function finish() {
        const finalTime = document.getElementById('count-timer').innerText;
        startTime = 0;
        gameScreen.classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        document.getElementById('res-summary').innerText = `Tid: ${finalTime} | Klikk: ${clicks} | Feil: ${errors}`;
        
        const ana = document.getElementById('analysis');
        if(failedTasks.length > 0) {
            ana.innerHTML = "<b>Oppgaver du bommet på:</b><br>";
            failedTasks.forEach(t => ana.innerHTML += `<div>❌ ${t.q} = ${t.a}</div>`);
        } else {
            ana.innerHTML = "<b style='color:var(--success)'>Perfekt gjennomført! ⭐</b>";
        }
    }
</script>

</body>
</html>
