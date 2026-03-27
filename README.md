<!DOCTYPE html>
<html lang="no">
<head>
    <meta charset="UTF-8">
    <title>Gange-Galskap: World Class</title>
    <script src="https://download.playfab.com/PlayFabClientApi.js"></script>
    <style>
        :root { --bg: #0f172a; --card: #1e293b; --accent: #38bdf8; --gold: #fbbf24; }
        body { font-family: sans-serif; background: var(--bg); color: white; display: flex; justify-content: center; padding: 20px; }
        .game-container { background: var(--card); padding: 2rem; border-radius: 1rem; width: 100%; max-width: 500px; text-align: center; }
        
        /* Leaderboard Styling */
        #leaderboard-area { margin-top: 20px; background: #0f172a; padding: 15px; border-radius: 10px; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th { color: var(--accent); font-size: 0.8rem; text-transform: uppercase; border-bottom: 1px solid #334155; padding: 5px; }
        td { padding: 8px; border-bottom: 1px solid #1e293b; font-size: 0.9rem; }
        .gold { color: var(--gold); font-weight: bold; }

        input, button { 
            padding: 10px; border-radius: 5px; border: none; margin: 5px; 
        }
        .btn-send { background: var(--accent); color: #0f172a; font-weight: bold; cursor: pointer; }
        .hidden { display: none; }
    </style>
</head>
<body>

<div class="game-container">
    <h2>Gange-Galskap 🏆</h2>
    
    <div id="login-screen">
        <p>Skriv inn ditt spillernavn:</p>
        <input type="text" id="username" placeholder="Navn...">
        <button class="btn-send" onclick="startMedNavn()">Start Spill</button>
    </div>

    <div id="game-screen" class="hidden">
        <h3 id="task">Laster...</h3>
        <p>Klikk/Tast: <span id="clicks">0</span></p>
        <input type="number" id="ans">
        <div id="feedback"></div>
    </div>

    <div id="result-screen" class="hidden">
        <h3 style="color: var(--gold)">Ferdig!</h3>
        <p>Du brukte <span id="final-clicks">0</span> klikk.</p>
        
        <div id="leaderboard-area">
            <th>TOPP 10 - FÆRREST KLIKK</th>
            <table id="lb-table">
                <thead>
                    <tr>
                        <th>Plass</th>
                        <th>Navn</th>
                        <th>Klikk</th>
                    </tr>
                </thead>
                <tbody id="lb-body">
                    </tbody>
            </table>
        </div>
        <button onclick="location.reload()">Prøv igjen</button>
    </div>
</div>

<script>
    // --- PLAYFAB KONFIGURASJON ---
    PlayFab.settings.titleId = "DIN_TITLE_ID_HER"; // BYTT UT DENNE!

    let totalClicks = 0;
    let currentPlayer = "";

    function startMedNavn() {
        currentPlayer = document.getElementById('username').value;
        if (currentPlayer.length < 2) return alert("Navn for kort");

        // Logg inn i PlayFab
        const loginRequest = {
            TitleId: PlayFab.settings.titleId,
            CustomId: currentPlayer,
            CreateAccount: true
        };

        PlayFabClientSDK.LoginWithCustomID(loginRequest, (result, error) => {
            if (result) {
                console.log("Innlogget!");
                // Oppdater skjermnavn i PlayFab (valgfritt, men pent)
                PlayFabClientSDK.UpdateUserTitleDisplayName({ DisplayName: currentPlayer }, () => {});
                
                document.getElementById('login-screen').classList.add('hidden');
                document.getElementById('game-screen').classList.remove('hidden');
                // Her ville din spill-logikk startet
                document.getElementById('task').innerText = "Hva er 5 x 5?"; 
            } else {
                alert("Kunne ikke koble til PlayFab. Sjekk Title ID.");
            }
        });
    }

    // --- SEND SCORE ---
    function sendScore(score) {
        const updateRequest = {
            Statistics: [{
                StatisticName: "GangeGalskapNiva1",
                Value: score
            }]
        };

        PlayFabClientSDK.UpdatePlayerStatistics(updateRequest, (result, error) => {
            if (result) {
                console.log("Poeng lagret!");
                visLeaderboard();
            }
        });
    }

    // --- HENT OG VIS LEADERBOARD ---
    function visLeaderboard() {
        const getRequest = {
            StatisticName: "GangeGalskapNiva1",
            StartPosition: 0,
            MaxResultsCount: 10
        };

        PlayFabClientSDK.GetLeaderboard(getRequest, (result, error) => {
            if (result) {
                const lbBody = document.getElementById('lb-body');
                lbBody.innerHTML = ""; // Tøm tabellen

                result.data.Leaderboard.forEach((entry, index) => {
                    const row = `
                        <tr>
                            <td class="${index < 3 ? 'gold' : ''}">${entry.Position + 1}</td>
                            <td>${entry.DisplayName || entry.PlayFabId}</td>
                            <td>${entry.StatValue}</td>
                        </tr>
                    `;
                    lbBody.innerHTML += row;
                });
            }
        });
    }

    // Demo-funksjon for å fullføre spillet
    document.getElementById('ans').addEventListener('input', (e) => {
        totalClicks++;
        document.getElementById('clicks').innerText = totalClicks;
        
        if (e.target.value == "25") {
            document.getElementById('game-screen').classList.add('hidden');
            document.getElementById('result-screen').classList.remove('hidden');
            document.getElementById('final-clicks').innerText = totalClicks;
            sendScore(totalClicks);
        }
    });
</script>

</body>
</html>
