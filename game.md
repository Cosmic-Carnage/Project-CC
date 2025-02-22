---
layout: default
title: Game
permalink: /game
---
<html>
<head>
    <title>Cosmic Carnage</title>
    <style>
        /* Styling for the canvas */
        canvas {
            background-color: black;
            display: block;
            margin: 0 auto;
        }
        /* Styling for the page body */
        body {
            font-family: Arial, sans-serif;
            background-color: #f2f2f2;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
        }
        /* Center-aligned heading */
        h1 {
            text-align: center;
        }
        /* Styling for the "Create Player" form */
        #createPlayerForm {
            background-color: gray;
            border-radius: 5px;
            padding: 20px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            display: block;
        }
        /* Bold styling for labels */
        label {
            font-weight: bold;
        }
        /* Styling for text input */
        input[type="text"] {
            width: 100%;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            color: black;
        }
        /* Styling for the submit button */
        input[type="submit"] {
            background-color: gray;
            color: black;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        /* Center-aligned message text */
        #message {
            text-align: center;
        }
        canvas, table, #playAgainButton {
            display: none;
        }
        #playAgainButton {
            display: none;
            background-color: #662d91; /* Set the button background color to purple */
            color: white; /* Set the text color to white */
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold; /* Make the text bold */
        }
        #createPlayerForm {
            background-color: #662d91; /* Set the background color to purple */
            border-radius: 5px;
            padding: 20px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            display: block;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="600" height="425"></canvas>
    <h1>Scoreboard</h1>
    <table>
        <thead>
            <tr>
                <th>User</th>
                <th>Score</th>
            </tr>
        </thead>
        <tbody id="player-list">
            <!-- Player data will be inserted here -->
        </tbody>
    </table>
    <h1>Create Player</h1>
    <form id="createPlayerForm">
        <label for="user">Username:</label>
        <input type="text" id="user" name="user" required>
        <br>
        <input type="submit" value="Start Game">
    </form>
    <p id="message"></p>
    <!-- Add the "Play Again" button here -->
    <button id="playAgainButton" style="display: none;">Play Again</button>
    <script>
        // Fetch player data from our backend API
        function fetchPlayerData() {
            fetch('http://172.31.132.100:8086/api/players/')
                .then(response => response.json())
                .then(data => {
                    // Player-list tbody element
                    const playerList = document.getElementById('player-list');
                    // Clear any existing data
                    playerList.innerHTML = '';
                    // Loop through the player data and create table rows
                    data.forEach(player => {
                        const row = document.createElement('tr');
                        row.innerHTML = `
                            <td>${player.user}</td>
                            <td>${player.score}</td>
                        `;
                        playerList.appendChild(row);
                    });
                })
                .catch(error => {
                    console.error('Error fetching player data:', error);
                });
        }
        // Fetch and display player data
        fetchPlayerData();
        function startGame() {
            canvas.style.display = 'block';
            document.querySelector('h1').style.display = 'block';
            document.querySelector('table').style.display = 'block';
            // Clear the canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            // Reset game variables
            isGameOver = false;
            score = 0;
            timeLeft = 30;
            // Reset player position
            player.x = canvas.width / 2;
            player.y = canvas.height - 40;
            // Add any other variable resets here
            // Start the game loop
            draw();
        }
        // Display the "Create Player" form when the Start Game button is clicked
        document.getElementById("createPlayerForm").addEventListener("submit", function (e) {
            e.preventDefault();
            const user = document.getElementById("user").value;
            // Display a success message
            document.getElementById("createPlayerForm").style.display = "none"; // Hide the form
            startGame(); // Call your game start function here
        });
        // Initialize the canvas and game parameters
        const canvas = document.getElementById('gameCanvas'); // create canvas element
        const ctx = canvas.getContext('2d'); // 2d rendering of canvas
        const player = { // define player properties
            x: canvas.width / 2,
            y: canvas.height - 40,
            width: 40,
            height: 40,
            speed: 15,
            angle: 0
        };
        const bullets = []; // create an array to store bullets
        const enemy = { // define enemy properties
            x: canvas.width / 2,
            y: 0,
            width: 40,
            height: 40,
            speed: 2
        };
        let isGameOver = false;
        let score = 0;
        let timeLeft = 30;
        const playerImage = new Image();
        playerImage.src = 'https://github.com/TayKimmy/CSA_Repo/assets/107821010/28c3e277-b292-43f0-bcef-5460b19689b7'; // making the player a spaceship image
        const enemyImage = new Image();
        enemyImage.src = 'https://github.com/Ant11234/student/assets/40652645/b7e15072-5d72-48f9-ad28-fdb227f2b20d'; // making the enemy a ufo image
        const minibossImage = new Image();
        minibossImage.src = 'https://github.com/Ant11234/student/assets/40652645/3bf0b840-7b21-428d-858b-3c668db352f6'; // making a miniboss
        playerImage.onload = () => {
            draw(); // execute draw() when the player's image is loaded
        };
        // draw the player's spaceship
        function drawPlayer() {
            ctx.save(); // Save the canvas state
            ctx.translate(player.x + player.width / 2, player.y + player.height / 2); // Translate to the center of the player
            ctx.rotate(player.angle); // Rotate the canvas based on player's angle
            ctx.drawImage(playerImage, -player.width / 2, -player.height / 2, player.width, player.height); // Draw the player image
            ctx.restore();
        }
        // draw the enemy's spaceship
        function drawEnemy() {
            ctx.drawImage(enemyImage, enemy.x, enemy.y, enemy.width, enemy.height);
        }
        // draw bullets on the canvas
        function drawBullets() {
            for (let i = 0; i < bullets.length; i++) {
                ctx.beginPath();
                ctx.rect(bullets[i].x, bullets[i].y, bullets[i].width, bullets[i].height);
                ctx.fillStyle = "orange";
                ctx.fill();
                ctx.closePath();
            }
        }
        // move bullets upward
        function moveBullets() {
            for (let i = 0; i < bullets.length; i++) {
                bullets[i].y -= 5;
                if (bullets[i].y < 0) {
                    bullets.splice(i, 1);
                }
            }
        }
        // check for collisions between player, bullets, and enemy
        function checkCollision() {
            if (
                player.x < enemy.x + enemy.width &&
                player.x + player.width > enemy.x &&
                player.y < enemy.y + enemy.height &&
                player.y + player.height > enemy.y
            ) {
                isGameOver = true;
            }
            for (let i = 0; i < bullets.length; i++) {
                if (
                    bullets[i].x < enemy.x + enemy.width &&
                    bullets[i].x + bullets[i].width > enemy.x &&
                    bullets[i].y < enemy.y + enemy.height &&
                    bullets[i].y + bullets[i].height > enemy.y
                ) {
                    enemy.x = Math.random() * (canvas.width - enemy.width);
                    enemy.y = 10;
                    bullets.splice(i, 1);
                    score += 10; // increment the score when an enemy is hit
                }
            }
        }
        const miniBoss = {
            x: 0,
            y: 0,
            width: 100,
            height: 100,
            speed: 15,
        };
        // Function to spawn the mini-boss
        function spawnMiniBoss() {
            miniBoss.x = Math.random() * (canvas.width - miniBoss.width);
            miniBoss.y = Math.random() * (canvas.height - miniBoss.height);
        }
        // Draw the mini-boss
        function drawMiniBoss() {
            ctx.drawImage(minibossImage, miniBoss.x, miniBoss.y, miniBoss.width, miniBoss.height);
        }
        // Update the mini-boss
        function updateMiniBoss() {
            miniBoss.x += (Math.random() - 0.5) * miniBoss.speed;
            miniBoss.y += (Math.random() - 0.5) * miniBoss.speed;
            miniBoss.x = Math.max(0, Math.min(canvas.width - miniBoss.width, miniBoss.x));
            miniBoss.y = Math.max(0, Math.min(canvas.height - miniBoss.height, miniBoss.y));
        }
        setInterval(() => {
            minibossSpeed += 1; // Increase miniboss speed by 1 (you can adjust the increment as needed)
        }, 5000);
        // Check for collisions with the mini-boss
        function checkMiniBossCollision() {
            if (
                player.x < miniBoss.x + miniBoss.width &&
                player.x + player.width > miniBoss.x &&
                player.y < miniBoss.y + miniBoss.height &&
                player.y + player.height > miniBoss.y
            ) {
                isGameOver = true;
            }
            for (let i = 0; i < bullets.length; i++) {
                if (
                    bullets[i].x < miniBoss.x + miniBoss.width &&
                    bullets[i].x + bullets[i].width > miniBoss.x &&
                    bullets[i].y < miniBoss.y + miniBoss.height &&
                    bullets[i].y + bullets[i].height > miniBoss.y
                ) {
                    spawnMiniBoss(); // Respawn the mini-boss
                    bullets.splice(i, 1);
                    score += 20; // Increment the score when mini-boss is hit
                }
            }
        }
        const playAgainButton = document.getElementById("playAgainButton");
        playAgainButton.addEventListener("click", function () {
            // Reset game variables
            isGameOver = false;
            score = 0;
            timeLeft = 30;
            player.x = canvas.width / 2;
            player.y = canvas.height - 40;
            // Hide the button and show the "Create Player" form
            playAgainButton.style.display = "none";
            document.getElementById("createPlayerForm").style.display = "block";
            // Clear the canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            // Call your game start function again
            startGame();
        });
        spawnMiniBoss();
        // draw on the canvas
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height); // clear the canvas
            if (!isGameOver && timeLeft > 0) { // if it is not game over and time is left
                drawPlayer();
                drawEnemy();
                drawMiniBoss(); // Draw the mini-boss
                updateMiniBoss();
                drawBullets();
                moveBullets();
                checkCollision();
                checkMiniBossCollision();
                requestAnimationFrame(draw);
                // display the score and time on the canvas
                ctx.font = "20px Arial";
                ctx.fillStyle = "white";
                ctx.fillText("Score: " + score, 10, 30);
                ctx.fillText("Time Left: " + timeLeft + "s", 10, 60);
            } else if (!isGameOver && timeLeft === 0) { 
                isGameOver = true;
                ctx.font = "30px Arial";
                ctx.fillStyle = "red";
                ctx.fillText("Time's Up!", canvas.width / 2 - 80, canvas.height / 2);
                ctx.fillText("Score: " + score, canvas.width / 2 - 60, canvas.height / 2 + 40);
            } else {
                ctx.font = "30px Arial";
                ctx.fillStyle = "red";
                ctx.fillText("Game Over", canvas.width / 2 - 80, canvas.height / 2);
                ctx.fillText("Score: " + score, canvas.width / 2 - 60, canvas.height / 2 + 40);
            } if (isGameOver) {
                // Display game over message and score
                ctx.font = "30px Arial";
                ctx.fillStyle = "red";
                ctx.fillText("Game Over", canvas.width / 2 - 80, canvas.height / 2);
                ctx.fillText("Score: " + score, canvas.width / 2 - 60, canvas.height / 2 + 40);
                // Show the "Play Again" button
                console.log("Setting button to block");
                playAgainButton.style.display = "block";
                console.log("Button style: " + playAgainButton.style.display);
            }
        }
        const playerImages = [
            'https://github.com/Ant11234/student/assets/40652645/15b4e3de-f9c4-4c70-adc9-d696cffd6ad7',
            'https://github.com/Ant11234/student/assets/40652645/97854bf1-907b-4a51-9de1-7064b7f296da',
            'https://github.com/Ant11234/student/assets/40652645/2122f367-aea5-4a97-bb5d-ace685929f77'
        ];
        // function to handle user input (keyboard presses)
        function keyDownHandler(e) {
            if (e.key == "Right" || e.key == "ArrowRight") { // if right key is pushed
                if (player.x + player.width < canvas.width) { // if player is not on the very far right
                    player.x += player.speed; // moving with defined speed in the right direction
                    player.angle = Math.PI / 2;
                }
            } else if (e.key == "Left" || e.key == "ArrowLeft") { // if left key is pushed
                if (player.x > 0) { // if player is not on the very far left
                    player.x -= player.speed; // moving with defined speed in left direction
                    player.angle = -Math.PI / 2;
                    // Switch to the previous spaceship image (cycling through the array)
                }
            } else if (e.key == " ") { // if space is pushed
                bullets.push({ // show bullets
                    x: player.x + player.width / 2 - 2.5, // from the middle of the player's icon
                    y: player.y, // from player's height
                    width: 5,
                    height: 10
                });
            }
        }
        // add keydown event listener to the document
        document.addEventListener("keydown", keyDownHandler, false);
        // Initial call to the draw() function
        draw();
        // Set up a timer interval to decrement timeLeft
        const timerInterval = setInterval(function() {
            if (!isGameOver && timeLeft > 0) { // if time and game is not over
                timeLeft--; // decrease time left
            } else {
                clearInterval(timerInterval); // clear interval if game is over
            }
        }, 1000); // 100
    </script>
</body>
</html>
