<!DOCTYPE html>
<html lang="no">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gange-Galskap: 1FEBAF Edition</title>
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
        .question { font-size: 4.5rem; font-weight: 900; margin: 20px 0; min-height: 100px; }
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
        th { background: #1e293b; padding: 10px; font-size: 0.8rem; color: var(--accent); text-align: left; }
        td { padding: 10px; border-bottom: 1px solid #1e293b; font-size: 0.9rem; text-align: left; }
        .gold-rank { color: var(--gold); font-weight: bold; }

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
            transition: transform 0.1s;
        }
        .btn:active { transform: scale(0.98); }

        /* Analysis */
        .analysis-list { text-align: left; font-size: 0.85rem; max-height: 120px; overflow-y: auto; margin: 15px 0; padding: 10px; background: #0f172a; border-radius: 8px; }
        .fail-item { color: var(--danger); margin-bottom: 5px; }
        .slow-item { color: var(--gold); margin-bottom: 5px; }
    </style>
</head>
<body>

<div class="container">
    <div id="start-screen">
        <h2 style="color: var(--accent); margin-bottom: 5px;">Gange-Galskap 🧠</h2>
        <p style="color: var(--text-muted); font-size: 0.9rem; margin-bottom: 20px;">Konkurrer om færrest klikk!</p>
        
        <input type="text" id="player-name" placeholder="Ditt navn..." style="font-size: 1.2rem; width: 85%; margin-bottom: 15px;">
        
        <div style="margin: 20px 0; background: #0f172a; padding: 15px; border-radius: 12px; border: 1px solid #334155;">
            <span style="display:block; font-size: 0.8rem; color: var(--text-muted); margin-bottom: 10px;">VELG NIVÅ:</span>
            <label style="cursor:pointer;"><input type="radio" name="lvl" value="1" checked> 1-5 gangen</label>
            <label style="margin-left: 20px; cursor:pointer;"><input type="radio" name="lvl" value="2"> 5-10 gangen</label>
        </div>
        
        <button class="btn" onclick="startApp()">LOGG INN & START</button>
    </div>

    <div id="game-screen" class="hidden">
        <div class="stats-grid">
            <div class="stat-box"><span class="stat-label">Klikk</span><span class="stat-val" id="count-clicks">0</span></div>
            <div class="stat-box"><span class="stat-label">Feil</span><span class="stat-val" id="count-errors" style="color: var(--danger)">0</span></div>
            <div class="stat-box"><span class="stat-label">Tid</span><span class="stat-val" id="count-timer">0.0s</span></div>
        </div>
        <div class="question" id="q-display">? × ?</div>
        <input type="number" id="ans-input" autocomplete="off" placeholder="?">
        <p id="feedback" style="height: 20px; margin-top: 15px; font-weight: bold;"></p>
    </div>

    <div id="result-screen" class="hidden">
        <h2 style="color: var(--gold); margin-bottom: 5px;">Runde Ferdig! 🏆</h2>
        <p>Din score: <span id="res-total" style="font-weight: bold; color: var(--accent); font-size: 1.5rem;">0</span> klikk</p>
        
        <div class="analysis-list" id="analysis"></div>

        <div id="lb-container">
            <h4 style="margin: 20px 0 10px 0; text-align: left; color: var(--accent);">TOPP 5 GLOBALT</h4>
            <table id="lb-table">
                <thead><tr><th>#</th><th>Navn</th><th>Klikk</th></tr></thead>
                <tbody id="lb-body">
                    <tr><td colspan="3" style="text-align:center;">Laster liste...</td></tr>
                </tbody>
            </table>
        </div>
        <button class="btn" style="background: #334155; color: white;" onclick="location.reload()">PRØV IGJEN</button>
    </div>
</div>

<script>
    // --- PLAYFAB KONFIGURASJON ---
    PlayFab.settings.titleId = "1FEBAF"; 

    let level = 1, qCount = 0, clicks = 0, errors = 0;
    let currentAns = 0, currentQ = "", startTime = 0, qStartTime = 0;
    let results = [];
    const maxQ = 10;

    const ansInp = document.getElementById('ans-input');

    function startApp() {
        const name = document.getElementById('player-name').value.trim();
        if (name.length < 2) return alert("Skriv navnet ditt (minst 2 tegn)");
        
        level = parseInt(document.querySelector('input[name="lvl"]:checked').value);
        
        // PlayFab Login
        PlayFabClientSDK.LoginWithCustomID({
            TitleId: PlayFab.settings.titleId,
            CustomId: name,
            CreateAccount: true
        }, (res, err) => {
            if(res) {
                // Oppdater Display Name slik at det vises i leaderboardet
                PlayFabClientSDK.UpdateUserTitleDisplayName({ DisplayName: name }, () => {});
                
                document.getElementById('start-screen').classList.add('hidden');
                document.getElementById('game-screen').classList.remove('hidden');
                initGame();
            } else { 
                console.error(err);
                alert("Kunne ikke koble til PlayFab. Sjekk Title ID."); 
            }
        });
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
        
        let n1, n2;
        if (level === 1) {
            n1 = Math.floor(Math.random() * 5) + 1; // 1-5
        } else {
            n1 = Math.floor(Math.random() * 6) + 5; // 5-10
        }
        n2 = Math.floor(Math.random() * 10) + 1;
        
        currentAns = n1 * n2;
        currentQ = `${n1} × ${n2}`;
        document.getElementById('q-display').innerText = currentQ;
        ansInp.value = "";
        ansInp.focus();
        qStartTime = Date.now();
    }

    // Tell hvert tastetrykk (klikk/forsøk)
    ansInp.addEventListener('keydown', (e) => {
        // Vi teller ikke "Enter", "Backspace" eller "Tab" som klikk-forsøk
        if(e.key !== "Enter" && e.key !== "Backspace" && e.key !== "Tab" && e.key !== "Shift") {
            clicks++;
            document.getElementById('count-clicks').innerText = clicks;
        }
    });

    ansInp.addEventListener('input', () => {
        const val = parseInt(ansInp.value);
        if(ansInp.value.length >= currentAns.toString().length) {
            if(val === currentAns) {
                results.push({ q: currentQ, time: (Date.now()-qStartTime)/1000, error: false });
                nextQuestion();
            } else {
                errors++;
                document.getElementById('count-errors').innerText = errors;
                results.push({ q: currentQ, time: 0, error: true });
                ansInp.value = "";
                document.getElementById('feedback').innerText = "Feil! Prøv igjen.";
                document.getElementById('feedback').style.color = "var(--danger)";
                setTimeout(() => document.getElementById('feedback').innerText = "", 800);
            }
        }
    });

    function finish() {
        startTime = 0; // Stopp timer
        document.getElementById('game-screen').classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        document.getElementById('res-total').innerText = clicks;

        // Bygg analyse-liste
        const ana = document.getElementById('analysis');
        ana.innerHTML = "<b style='color:var(--accent)'>Treningsrapport:</b><br>";
        let hardTasks = results.filter(r => r.error || r.time > 3);
        
        if(hardTasks.length === 0) {
            ana.innerHTML += "🌟 Perfekt! Du er en mattemester.";
        } else {
            hardTasks.forEach(r => {
                ana.innerHTML += `<div class="${r.error ? 'fail-item' : 'slow-item'}">${r.error ? '❌ Feil på' : '⏳ Treg på'} ${r.q}</div>`;
            });
        }

        // Send score til PlayFab
        const statName = (level === 1) ? "GangeGalskapNiva1" : "GangeGalskapNiva2";
        PlayFabClientSDK.UpdatePlayerStatistics({
            Statistics: [{ StatisticName: statName, Value: clicks }]
        }, (res, err) => {
            if(res) {
                console.log("Score lagret");
                loadLB(statName);
            } else {
                console.error("Kunne ikke lagre score. Har du skrudd på 'Allow client to post player statistics' i PlayFab?");
                document.getElementById('lb-body').innerHTML = "<tr><td colspan='3'>Kunne ikke lagre score. Sjekk PlayFab-innstillinger.</td></tr>";
            }
        });
    }

    function loadLB(statName) {
        PlayFabClientSDK.GetLeaderboard({
            StatisticName: statName,
            StartPosition: 0,
            MaxResultsCount: 5
        }, (res, err) => {
            if(res) {
                const body = document.getElementById('lb-body');
                body.innerHTML = "";
                res.data.Leaderboard.forEach(e => {
                    const isGold = e.Position === 0;
                    body.innerHTML += `
                        <tr>
                            <td class="${isGold ? 'gold-rank' : ''}">${e.Position + 1}</td>
                            <td>${e.DisplayName || 'Anonym'}</td>
                            <td style="font-weight:bold; color:var(--accent)">${e.StatValue}</td>
                        </tr>`;
                });
            }
        });
    }
</script>
</body>
</html>
