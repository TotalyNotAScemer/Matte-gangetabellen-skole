<!DOCTYPE html>
<html lang="no">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gange-Galskap: Skoleutgave</title>
    <script src="https://download.playfab.com/PlayFabClientApi.js"></script>
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

        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--bg);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
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

        /* Stats & Header */
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

        /* Game Area */
        .question { font-size: 4.5rem; font-weight: 900; margin: 20px 0; }
        input {
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
        input:focus { border-color: var(--accent); }

        /* Leaderboard Table */
        table { width: 100%; border-collapse: collapse; margin-top: 15px; background: #0f172a; border-radius: 10px; overflow: hidden; }
        th { background: #1e293b; padding: 10px; font-size: 0.8rem; color: var(--accent); }
        td { padding: 10px; border-bottom: 1px solid #1e293b; font-size: 0.9rem; }
        .my-score { background: rgba(56, 189, 248, 0.2); }

        /* Buttons */
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
        .btn:disabled { opacity: 0.5; cursor: not-allowed; }

        /* Analysis */
        .analysis-list { text-align: left; font-size: 0.85rem; max-height: 150px; overflow-y: auto; margin: 10px 0; }
        .fail-item { color: var(--danger); margin-bottom: 5px; }
    </style>
</head>
<body>

<div class="container">
    <div id="start-screen">
        <h2 style="color: var(--accent)">Gange-Galskap 🧠</h2>
        <p>Hvor få klikk trenger du på 10 oppgaver?</p>
        <input type="text" id="player-name" placeholder="Ditt navn..." style="font-size: 1.2rem; width: 80%; margin-bottom: 10px;">
        <div style="margin: 15px 0;">
            <label><input type="radio" name="lvl" value="1" checked> Nivå 1 (1-5)</label>
            <label style="margin-left: 15px;"><input type="radio" name="lvl" value="2"> Nivå 2 (5-10)</label>
        </div>
        <button class="btn" onclick="startApp()">START SPILLET</button>
    </div>

    <div id="game-screen" class="hidden">
        <div class="stats-grid">
            <div class="stat-box"><span class="stat-label">Klikk</span><span class="stat-val" id="count-clicks">0</span></div>
            <div class="stat-box"><span class="stat-label">Feil</span><span class="stat-val" id="count-errors" style="color: var(--danger)">0</span></div>
            <div class="stat-box"><span class="stat-label">Tid</span><span class="stat-val" id="count-timer">0.0s</span></div>
        </div>
        <div class="question" id="q-display">? × ?</div>
        <input type="number" id="ans-input" autocomplete="off">
        <p id="feedback" style="height: 20px; margin-top: 10px; font-weight: bold;"></p>
    </div>

    <div id="result-screen" class="hidden">
        <h2 id="final-status" style="color: var(--gold)">Runde Ferdig!</h2>
        <p>Total score: <span id="res-total" style="font-weight: bold; color: var(--accent);">0</span> klikk</p>
        
        <div class="analysis-list" id="analysis"></div>

        <div id="lb-container">
            <h4 style="margin-bottom: 5px;">🏆 Topp 5 Leaderboard</h4>
            <table id="lb-table">
                <thead><tr><th>#</th><th>Navn</th><th>Klikk</th></tr></thead>
                <tbody id="lb-body"></tbody>
            </table>
        </div>
        <button class="btn" onclick="location.reload()">Spill Igjen</button>
    </div>
</div>

<script>
    // --- KONFIGURASJON ---
    PlayFab.settings.titleId = "DIN_TITLE_ID_HER"; // <--- Sett inn din Title ID her!

    let level = 1, qCount = 0, clicks = 0, errors = 0;
    let currentAns = 0, currentQ = "", startTime = 0, qStartTime = 0;
    let results = [];
    const maxQ = 10;

    const ansInp = document.getElementById('ans-input');

    function startApp() {
        const name = document.getElementById('player-name').value;
        if (name.length < 2) return alert("Skriv et navn først!");
        
        level = parseInt(document.querySelector('input[name="lvl"]:checked').value);
        
        // PlayFab Login
        PlayFabClientSDK.LoginWithCustomID({
            TitleId: PlayFab.settings.titleId,
            CustomId: name,
            CreateAccount: true
        }, (res) => {
            if(res) {
                PlayFabClientSDK.UpdateUserTitleDisplayName({ DisplayName: name }, () => {});
                document.getElementById('start-screen').classList.add('hidden');
                document.getElementById('game-screen').classList.remove('hidden');
                initGame();
            } else { alert("Feil med PlayFab-tilkobling."); }
        });
    }

    function initGame() {
        startTime = Date.now();
        nextQuestion();
        setInterval(() => {
            if(qCount <= maxQ) document.getElementById('count-timer').innerText = ((Date.now() - startTime)/1000).toFixed(1) + "s";
        }, 100);
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
        if(e.key !== "Enter" && e.key !== "Backspace" && e.key !== "Tab") {
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
                document.getElementById('feedback').innerText = "Feil! Prøv igjen.";
                setTimeout(() => document.getElementById('feedback').innerText = "", 800);
            }
        }
    });

    function finish() {
        document.getElementById('game-screen').classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        document.getElementById('res-total').innerText = clicks;

        // Analyse
        const ana = document.getElementById('analysis');
        ana.innerHTML = "<b>Analyse:</b><br>";
        results.forEach(r => {
            if(r.error || r.time > 3) {
                ana.innerHTML += `<div class="fail-item">${r.error ? '❌ Feil på' : '⏳ Treg på'} ${r.q}</div>`;
            }
        });

        // Send til PlayFab
        const statName = (level === 1) ? "GangeGalskapNiva1" : "GangeGalskapNiva2";
        PlayFabClientSDK.UpdatePlayerStatistics({
            Statistics: [{ StatisticName: statName, Value: clicks }]
        }, () => loadLB(statName));
    }

    function loadLB(statName) {
        PlayFabClientSDK.GetLeaderboard({
            StatisticName: statName, StartPosition: 0, MaxResultsCount: 5
        }, (res) => {
            if(res) {
                const body = document.getElementById('lb-body');
                body.innerHTML = "";
                res.data.Leaderboard.forEach(e => {
                    body.innerHTML += `<tr><td>${e.Position+1}</td><td>${e.DisplayName || 'Anonym'}</td><td>${e.StatValue}</td></tr>`;
                });
            }
        });
    }
</script>
</body>
</html>
