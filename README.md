<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BrookMatch - Realtime Matchfinder</title>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>
    <style>
        /* Modern Dark Gamer Theme */
        body {
            font-family: 'Segoe UI', sans-serif;
            background-color: #121212;
            color: #ffffff;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }
        h1 { color: #00ff88; margin-bottom: 5px; }
        .subtitle { color: #aaa; margin-bottom: 30px; font-size: 14px; }
        .box-container {
            background-color: #1e1e1e;
            padding: 25px;
            border-radius: 12px;
            width: 100%;
            max-width: 400px;
            box-shadow: 0 4px 15px rgba(0, 255, 136, 0.1);
            box-sizing: border-box;
            text-align: center;
        }
        input {
            width: 100%;
            padding: 12px;
            margin: 10px 0;
            border: 1px solid #333;
            border-radius: 6px;
            background-color: #2a2a2a;
            color: white;
            box-sizing: border-box;
        }
        button {
            width: 100%;
            padding: 12px;
            background-color: #00ff88;
            color: #121212;
            border: none;
            border-radius: 6px;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            transition: 0.3s;
            margin-top: 10px;
        }
        button:hover { background-color: #00cc6a; }
        #google-btn {
            background-color: #4285F4;
            color: white;
        }
        #google-btn:hover { background-color: #357ae8; }
        .pano {
            width: 100%;
            max-width: 400px;
            margin-top: 30px;
        }
        .takim-karti {
            background-color: #1e1e1e;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 12px;
            border-left: 5px solid #00ff88;
            display: flex;
            justify-content: space-between;
            align-items: center;
            width: 100%;
            box-sizing: border-box;
        }
        .takim-bilgi { text-align: left; }
        .takim-bilgi h3 { margin: 0 0 5px 0; color: #fff; font-size: 18px; }
        .takim-bilgi p { margin: 0; font-size: 13px; color: #aaa; line-height: 1.5; }
        .mac-istek-btn {
            background-color: #ff3366;
            color: white;
            width: auto;
            padding: 8px 15px;
            font-size: 13px;
            margin-top: 0;
        }
        .mac-istek-btn:hover { background-color: #cc2952; }
        .cikis-btn {
            background-color: #333;
            color: #ccc;
            font-size: 12px;
            padding: 8px;
            width: auto;
            margin-top: 40px;
        }
    </style>
</head>
<body>
    <h1>BrookMatch</h1>
    <div class="subtitle">Realtime Brookhaven Matchfinder</div>
    <div id="login-ekrani" class="box-container">
        <h2>System Login</h2>
        <p style="color: #888; font-size: 14px; margin-bottom: 20px;">Connect your Google account to see all global matches.</p>
        <button id="google-btn">Login with Google</button>
    </div>
    <div id="mac-ekrani" style="display: none; width: 100%; max-width: 400px;">
        <div class="box-container" style="max-width: 100%;">
            <h3>Create a Match Post</h3>
            <input type="text" id="takimAdi" placeholder="Team Name (e.g. Pink Barça)">
            <input type="text" id="robloxUser" placeholder="Roblox Username">
            <input type="text" id="tiktokUser" placeholder="TikTok Username (without @)">
            <button onclick="ilanEkle()">Post to Board</button>
        </div>
        <div class="pano" id="panoListesi">
            <h2>Active Match Seekers</h2>
        </div>
        <center>
            <button id="cikis-btn" class="cikis-btn">Logout</button>
        </center>
    </div>
    <script>
        // Your Firebase Configuration
        const firebaseConfig = {
            apiKey: "AIzaSyCsyqB-MzS72ipbRUSjX1oJ5_fvxplwI2w",
            authDomain: "bh-mac-bulucu.firebaseapp.com",
            databaseURL: "https://bh-mac-bulucu-default-rtdb.europe-west1.firebasedatabase.app",
            projectId: "bh-mac-bulucu",
            storageBucket: "bh-mac-bulucu.firebasestorage.app",
            messagingSenderId: "277878296468",
            appId: "1:277878296468:web:9a185c12334458b0d74981",
            measurementId: "G-EF67NNBS81"
        };
        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const database = firebase.database();
        const provider = new firebase.auth.GoogleAuthProvider();
        // Auth State Observer
        auth.onAuthStateChanged((user) => {
            if (user) {
                document.getElementById("login-ekrani").style.display = "none";
                document.getElementById("mac-ekrani").style.display = "block";
                panoyuCanliDinle(); 
            } else {
                document.getElementById("login-ekrani").style.display = "block";
                document.getElementById("mac-ekrani").style.display = "none";
            }
        });
        // Google Login Button
        document.getElementById("google-btn").onclick = function() {
            auth.signInWithRedirect(provider).catch((error) => {
                alert("Login Error: " + error.message);
            });
        };
        // Logout Button
        document.getElementById("cikis-btn").onclick = function() {
            auth.signOut();
        };
        // Add Post to Database
        function ilanEkle() {
            let takim = document.getElementById("takimAdi").value;
            let roblox = document.getElementById("robloxUser").value;
            let tiktok = document.getElementById("tiktokUser").value;
            if (takim === "" || roblox === "" || tiktok === "") {
                alert("Please fill in all fields!");
                return;
            }
            database.ref('ilanlar').push({
                takim: takim,
                roblox: roblox,
                tiktok: tiktok,
                zaman: Date.now(),
                kullaniciUid: auth.currentUser.uid
            }).then(() => {
                document.getElementById("takimAdi").value = "";
                document.getElementById("robloxUser").value = "";
                document.getElementById("tiktokUser").value = "";
            }).catch((error) => {
                alert("Error: " + error.message);
            });
        }
        // Listen to Realtime Board
        function panoyuCanliDinle() {
            database.ref('ilanlar').orderByChild('zaman').on('value', (snapshot) => {
                let panoListesiDiv = document.getElementById("panoListesi");
                panoListesiDiv.innerHTML = "<h2>Active Match Seekers</h2>";
                let ilanlar = [];
                snapshot.forEach((childSnapshot) => {
                    ilanlar.push(childSnapshot.val());
                });ilanlar.reverse(); // Newest first      if (ilanlar.length === 0) {
                    panoListesiDiv.innerHTML += "<p style='color: #666; text-align: center;'>No active matches right now. Be the first to post!</p>";
                    return;
                }
                ilanlar.forEach((ilan) => {
                    panoListesiDiv.innerHTML += `
                        <div class="takim-karti">
                            <div class="takim-bilgi">
                                <h3>${ilan.takim}</h3>
                                <p>Name: ${ilan.roblox}<br>Tiktok: @${ilan.tiktok}</p>
                            </div>
                            <button class="mac-istek-btn" onclick="macTeklifEt('${ilan.takim}', '${ilan.tiktok}')">Request Match</button>
                        </div>
                    `;
                });
            });
        }

        // Match Request Alert
        function macTeklifEt(takim, tiktok) {
            alert(`Request sent! You can DM @${tiktok} on TikTok and say you are coming from BrookMatch for the ${takim} match.`);
        }
    </script>

</body>
</html>
