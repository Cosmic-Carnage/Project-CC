---
layout: default
title: Game
permalink: /game
---
<html>
<head>
    <title>Cosmic Carnage</title>
    <style>
        canvas {
        background-color: black;
        display: block;
        margin: 0 auto;
    }
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
        h1 {
            text-align: center;
        }
        #createPlayerForm {
            background-color: gray;
            border-radius: 5px;
            padding: 20px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
        }
        label {
            font-weight: bold;
        }
        input[type="text"],
        input[type="number"] {
            width: 100%;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            color: black;
        }
        input[type="submit"] {
            background-color: gray;
            color: black;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="600" height="425"></canvas>
<html>
<head>
    <title>Player List</title>
</head>
<body>
    <h1>Scoreboard</h1>
    <table>
        <thead>
            <tr>
                <th>User</th>
                <th>Score</th>
            </tr>
        </thead>
    <h1>Create Player</h1>
    <form id="createPlayerForm">
        <label for="user">Username:</label>
        <input type="text" id="user" name="user" required>
        <br>
        <label for="score">Score:</label>
        <input type="number" id="score" name="score" required>
        <br>
        <input type="submit" value="Submit">
    </form>
        <tbody id="player-list">
            <!-- Player data will be inserted here -->
        </tbody>
    </table>
</body>
</html>
    <script>
        // fetch player data from our backend API
        function fetchPlayerData() {
            fetch('https://cosmic-backend.stu.nighthawkcodingsociety.com/api/leaderboard/')
                .then(response => response.json())
                .then(data => {
                    // player-list tbody element
                    const playerList = document.getElementById('player-list');
                    // clear any existing data
                    playerList.innerHTML = '';
                    // loop through the player data and create table rows
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
        fetchPlayerData();
        document.getElementById("createPlayerForm").addEventListener("submit", function (e) {
            e.preventDefault();
            const id = 161;
            const user = document.getElementById("user").value;
            const score = parseInt(document.getElementById("score").value);
            // Create a player object
            const playerData = {
                user: user,
                score: score
            };
            fetch('https://cosmic-backend.stu.nighthawkcodingsociety.com/api/leaderboard/addScore/' + (id+1) + '/' + score, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(playerData),
            })
            .then(response => {
                if (response.status === 201) {
                    return response.json(); // Successfully created a player
                } else {
                    return response.json().then(errorData => {
                        throw new Error(errorData.message);
                    });
                }
            })
            .then(player => {
                // Display a success message or redirect to another page
                document.getElementById("message").innerHTML = `Player ${player.user} created with score ${player.score}`;
            })
            .catch(error => {
                // Display an error message
                document.getElementById("message").innerHTML = `Error: ${error.message}`;
            });
        });
        const canvas = document.getElementById('gameCanvas'); // create canvas element
        const ctx = canvas.getContext('2d'); // 2d rendering of canvas
        const player = { // define player properties
            x: canvas.width / 2,
            y: canvas.height - 60,
            width: 50,
            height: 50,
            speed: 20,
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
        minibossImage.src = 'https://github.com/Ant11234/student/assets/40652645/b7e15072-5d72-48f9-ad28-fdb227f2b20d'; // making a second enemy
        playerImage.onload = () => {
            draw(); // execute draw() when the player's image is loaded
        };
        // draw the player's spaceship
        function drawPlayer() {
            ctx.save(); // Save the current context state
            ctx.translate(player.x + player.width / 2, player.y + player.height / 2); // Move the context origin to the center of the player
            ctx.rotate(player.angle); // Rotate the context based on the player's angle
            ctx.drawImage(playerImage, -player.width / 2, -player.height / 2, player.width, player.height); // Draw the player centered
            ctx.restore(); // Restore the context to the previous state
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
                    score += 1; // increment the score when an enemy is hit
                }
            }
        }
        // draw on the canvas
        function draw() {
             ctx.clearRect(0, 0, canvas.width, canvas.height); // clear the canvas
            if (!isGameOver && timeLeft > 0) { // if it is not game over and time is left
                drawPlayer();
                drawEnemy();
                drawBullets();
                moveBullets();
                checkCollision();
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
            }
        }
        // Define an array to store spaceship images
        const playerImages = [
            'https://github.com/Ant11234/student/assets/40652645/15b4e3de-f9c4-4c70-adc9-d696cffd6ad7',
            'https://github.com/Ant11234/student/assets/40652645/97854bf1-907b-4a51-9de1-7064b7f296da',
            'https://github.com/Ant11234/student/assets/40652645/2122f367-aea5-4a97-bb5d-ace685929f77'
        ];
        let currentImageIndex = 0; // Index to keep track of the current spaceship image
        // In the keyDownHandler function, update the player image based on movement direction
        let canShoot = true;
        const cooldownTime = 500; // Adjust the cooldown time as needed (in milliseconds)
        document.addEventListener("keydown", function (e) {
            if (e.key == "Right" || e.key == "ArrowRight") { // if right key is pushed
                if (player.x + player.width < canvas.width) { // if player is not on the very far right
                    player.x += player.speed;
                    player.angle = Math.PI / 2;
                    currentImageIndex = (currentImageIndex + 1) % playerImages.length;
                    playerImage.src = playerImages[currentImageIndex];
                }
            } else if (e.key == "Left" || e.key == "ArrowLeft") { // if left key is pushed
                if (player.x > 0) { // if player is not on the very far left
                    player.x -= player.speed;
                    player.angle = -Math.PI / 2;
                    // Switch to the previous spaceship image (cycling through the array)
                    currentImageIndex = (currentImageIndex - 1 + playerImages.length) % playerImages.length;
                    playerImage.src = playerImages[currentImageIndex];
                }
            } else if (e.key == " " && canShoot == true) { // if space is pushed
                bullets.push({ // show bullets
                    x: player.x + player.width / 2 - 2.5, // from the middle of the player's icon
                    y: player.y, // from player's height
                    width: 5,
                    height: 10
                });
                playerImage.src = 'https://github.com/TayKimmy/CSA_Repo/assets/107821010/28c3e277-b292-43f0-bcef-5460b19689b7';
                player.angle = 0;
                canShoot = false; // Prevent shooting until the cooldown period is over
                setTimeout(() => {
                     canShoot = true;
                }, cooldownTime);
            }
        });
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
        }, 1000); // 1000 ms - making sure it is every 1 second
    </script>
</body>
</html>

<!-- Enemy flying design -->

<!-- https://github.com/Ant11234/student/assets/40652645/3bf0b840-7b21-428d-858b-3c668db352f6 -->

