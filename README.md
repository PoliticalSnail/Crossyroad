<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Crossy Road Clone</title>
    <style>
        body {
            text-align: center;
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
        }
        canvas {
            background: lightgreen;
            display: block;
            margin: auto;
            border: 2px solid black;
        }
        #cookiePopup, #leaderboardPopup {
            display: none;
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            padding: 15px;
            border: 1px solid black;
            box-shadow: 2px 2px 10px rgba(0,0,0,0.2);
        }
        h1 { color: #333; }
        #leaderboard {
            list-style: none;
            padding: 0;
            max-width: 300px;
            margin: auto;
            background: white;
            border-radius: 5px;
            padding: 10px;
            box-shadow: 2px 2px 10px rgba(0,0,0,0.1);
        }
        #leaderboard li {
            padding: 5px;
            border-bottom: 1px solid #ddd;
        }
    </style>
</head>
<body>
    <h1>Crossy Road Clone</h1>
    <p>Score: <span id="score">0</span></p>
    <p>Top Score: <span id="topScore">0</span></p>
    <canvas id="gameCanvas" width="400" height="500"></canvas>

    <div id="cookiePopup">
        <p>This site uses cookies to save your top score. Do you accept?</p>
        <button onclick="acceptCookies()">Accept</button>
        <button onclick="declineCookies()">Decline</button>
    </div>

    <div id="leaderboardPopup">
        <p>Game Over! Enter a 3-letter name:</p>
        <input type="text" id="playerName" maxlength="3" />
        <button onclick="submitScore()">Submit</button>
    </div>

    <h2>Leaderboard</h2>
    <ul id="leaderboard"></ul>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const scoreDisplay = document.getElementById("score");
        const topScoreDisplay = document.getElementById("topScore");
        const cookiePopup = document.getElementById("cookiePopup");
        const leaderboardPopup = document.getElementById("leaderboardPopup");
        const playerNameInput = document.getElementById("playerName");
        const leaderboardList = document.getElementById("leaderboard");

        let score = 0;
        let topScore = 0;
        let cookiesAccepted = false;
        let gameRunning = true;
        let carSpawnInterval;
        const player = { x: 180, y: 450, width: 20, height: 20, color: "blue" };
        const cars = [];
        let carSpeed = 3;
        const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbxb82qnnHVZlaywKSdojKhjdvoB62J6SjJnIWs184qpPQ8nKfF5HXt_MBgbDng9YYBOWA/exec"; 

        function setCookie(name, value, days) {
            document.cookie = `${name}=${JSON.stringify(value)};max-age=${days * 86400};path=/`;
        }

        function getCookie(name) {
            const cookies = document.cookie.split('; ');
            for (let cookie of cookies) {
                let [key, value] = cookie.split('=');
                if (key === name) return JSON.parse(value);
            }
            return null;
        }

        function checkCookies() {
            if (getCookie("cookiesAccepted")) {
                cookiesAccepted = true;
                topScore = getCookie("topScore") || 0;
                topScoreDisplay.textContent = topScore;
            } else {
                cookiePopup.style.display = "block";
            }
        }

        function acceptCookies() {
            cookiesAccepted = true;
            setCookie("cookiesAccepted", true, 365);
            cookiePopup.style.display = "none";
        }

        function declineCookies() {
            cookiePopup.style.display = "none";
        }

        function createCar() {
            const lane = Math.floor(Math.random() * 5) * 80;
            cars.push({ x: lane, y: 0, width: 40, height: 20, color: "red" });
        }

        function resetGame() {
            score = 0;
            carSpeed = 3;
            cars.length = 0;
            player.x = 180;
            player.y = 450;
            gameRunning = true;
            leaderboardPopup.style.display = "none";
            if (carSpawnInterval) clearInterval(carSpawnInterval);
            carSpawnInterval = setInterval(createCar, 1000);
            update();
        }

        function gameOver() {
            gameRunning = false;
            clearInterval(carSpawnInterval);
            leaderboardPopup.style.display = "block";
        }

       function submitScore() {
    const name = playerNameInput.value.toUpperCase();
    if (name.length !== 3) {
        alert("Enter a 3-letter name.");
        return;
    }

    const scoreData = { name: name, score: score };

    fetch("https://script.google.com/macros/s/AKfycbxb82qnnHVZlaywKSdojKhjdvoB62J6SjJnIWs184qpPQ8nKfF5HXt_MBgbDng9YYBOWA/exec", {
        method: "POST",
        body: JSON.stringify(scoreData),
        headers: { "Content-Type": "application/json" },
    })
    .then(response => response.text())
    .then(data => {
        console.log("Server response:", data); // Debugging response from Google Apps Script
        if (data.trim() === "Success") {
            alert("Score submitted!");
            updateLeaderboard(); // Refresh leaderboard
            setTimeout(() => {
                resetGame(); // Ensure game restarts after leaderboard updates
            }, 1000); // 1 second delay before resetting game
        } else {
            alert("Error submitting score. Check console.");
        }
    })
    .catch(error => {
        console.error("Error:", error);
        alert("Failed to submit score. Check console.");
    });
}



   function updateLeaderboard() {
    fetch("https://script.google.com/macros/s/AKfycbxb82qnnHVZlaywKSdojKhjdvoB62J6SjJnIWs184qpPQ8nKfF5HXt_MBgbDng9YYBOWA/exec")
    .then(response => response.text()) // Get the response as plain text
    .then(data => {
        if (data.trim() === "Error") {
            console.error("Server returned an error:", data);
            alert("There was an issue fetching the leaderboard. Please try again later.");
            return;
        }

        try {
            const jsonData = JSON.parse(data); // Try to parse the response as JSON
            console.log("Fetched leaderboard data:", jsonData);

            leaderboardList.innerHTML = ""; // Clear the existing leaderboard
            let scores = jsonData.slice(0, 5); // Show top 5 scores (data already sorted by Apps Script)

            scores.forEach(entry => {
                const li = document.createElement("li");
                li.textContent = `${entry.name}: ${entry.score}`;
                leaderboardList.appendChild(li);
            });
        } catch (error) {
            console.error("Error parsing JSON response:", error);
            console.log("Response received:", data); // Log the raw response to debug
            alert("Failed to fetch leaderboard. Check console for more details.");
        }
    })
    .catch(error => {
        console.error("Error fetching leaderboard:", error);
        alert("Failed to fetch leaderboard. Check console for more details.");
    });
}









        function update() {
            if (!gameRunning) return;
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = player.color;
            ctx.fillRect(player.x, player.y, player.width, player.height);
            for (let i = cars.length - 1; i >= 0; i--) {
                let car = cars[i];
                car.y += carSpeed;
                ctx.fillStyle = car.color;
                ctx.fillRect(car.x, car.y, car.width, car.height);
                if (player.x < car.x + car.width && player.x + player.width > car.x && player.y < car.y + car.height && player.y + player.height > car.y) {
                    gameOver();
                    return;
                }
                if (car.y > canvas.height) {
                    score++;
                    cars.splice(i, 1);
                }
            }
            scoreDisplay.textContent = score;
            requestAnimationFrame(update);
        }

        document.addEventListener("keydown", (e) => {
            if (gameRunning) {
                if (e.key === "ArrowUp") player.y -= 20;
                if (e.key === "ArrowDown") player.y += 20;
                if (e.key === "ArrowLeft") player.x -= 20;
                if (e.key === "ArrowRight") player.x += 20;
            }
        });

        checkCookies();
        resetGame();
        updateLeaderboard();
    </script>
</body>
</html>	
