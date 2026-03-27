// 1. Konfigurasjon
PlayFab.settings.titleId = "DIN_TITLE_ID_HER";

// 2. Logg inn (bruker en tilfeldig ID for enkelhets skyld i demo)
function loginToPlayFab(username) {
    var loginRequest = {
        TitleId: PlayFab.settings.titleId,
        CustomId: username, // Her bør du egentlig bruke en unik ID
        CreateAccount: true
    };
    PlayFabClientSDK.LoginWithCustomID(loginRequest, (result, error) => {
        if (result) console.log("Innlogget!");
    });
}

// 3. Send poengsum (Klikk)
function sendScoreToPlayFab(scoreValue) {
    var updateRequest = {
        Statistics: [{
            StatisticName: "GangeGalskapNiva1",
            Value: scoreValue
        }]
    };
    
    PlayFabClientSDK.UpdatePlayerStatistics(updateRequest, (result, error) => {
        if (result) {
            console.log("Score sendt!");
            showLeaderboard(); // Oppdater lista når sendt
        } else {
            console.error("Feil ved sending:", error);
        }
    });
}

// 4. Hent topplisten
function showLeaderboard() {
    var getRequest = {
        StatisticName: "GangeGalskapNiva1",
        StartPosition: 0,
        MaxResultsCount: 10
    };

    PlayFabClientSDK.GetLeaderboard(getRequest, (result, error) => {
        if (result) {
            console.log("Toppliste:", result.data.Leaderboard);
            // Her kan du generere en HTML-tabell for å vise lista
        }
    });
}
