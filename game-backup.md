layout: default
title: Game
permalink: /game
---
<!DOCTYPE html>
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
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        let lastTimestamp = 0;
        const targetFps = 60; // Set your target frame rate here
        const player = {
            x: canvas.width / 2,
            y: canvas.height - 60,
            width: 50,
            height: 50,
            speed: 20,
            angle: 0
        };
        const bullets = [];
        const enemy = {
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
        playerImage.src = 'https://github.com/TayKimmy/CSA_Repo/assets/107821010/28c3e277-b292-43f0-bcef-5460b19689b7';
        const enemyImage = new Image();
        enemyImage.src = 'https://github.com/Ant11234/student/assets/40652645/b7e15072-5d72-48f9-ad28-fdb227f2b20d';
        const minibossImage = new Image();
        minibossImage.src = 'https://github.com/Ant11234/student/assets/40652645/b7e15072-5d72-48f9-ad28-fdb227f2b20d';
        playerImage.onload = () => {
            draw();
        };
        function drawPlayer() {
            ctx.save();
            ctx.translate(player.x + player.width / 2, player.y + player.height / 2);
            ctx.rotate(player.angle);
            ctx.drawImage(playerImage, -player.width / 2, -player.height / 2, player.width, player.height);
            ctx.restore();
        }
        function drawEnemy() {
            ctx.drawImage(enemyImage, enemy.x, enemy.y, enemy.width, enemy.height);
        }
        function drawBullets() {
            for (let i = 0; i < bullets.length; i++) {
                ctx.beginPath();
                ctx.rect(bullets[i].x, bullets[i].y, bullets[i].width, bullets[i].height);
                ctx.fillStyle = "orange";
                ctx.fill();
                ctx.closePath();
            }
        }
        function moveBullets() {
            for (let i = 0; i < bullets.length; i++) {
                bullets[i].y -= 5;
                if (bullets[i].y < 0) {
                    bullets.splice(i, 1);
                }
            }
        }
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
                    score += 1;
                }
            }
        }
        function draw(timestamp) {
            if (!lastTimestamp) {
                lastTimestamp = timestamp;
            }
            const deltaTime = timestamp - lastTimestamp;
            if (deltaTime < 1000 / targetFps) {
                requestAnimationFrame(draw);
                return;
            }
            lastTimestamp = timestamp;
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            if (!isGameOver && timeLeft > 0) {
                drawPlayer();
                drawEnemy();
                drawBullets();
                moveBullets();
                checkCollision();
                requestAnimationFrame(draw);
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
        const playerImages = [
            'https://github.com/Ant11234/student/assets/40652645/15b4e3de-f9c4-4c70-adc9-d696cffd6ad7',
            'https://github.com/Ant11234/student/assets/40652645/97854bf1-907b-4a51-9de1-7064b7f296da',
            'https://github.com/Ant11234/student/assets/40652645/2122f367-aea5-4a97-bb5d-ace685929f77'
        ];
        let currentImageIndex = 0;
        let canShoot = true;
        const cooldownTime = 500;
        document.addEventListener("keydown", function (e) {
            if (e.key == "Right" || e.key == "ArrowRight") {
                if (player.x + player.width < canvas.width) {
                    player.x += player.speed;
                    player.angle = Math.PI / 2;
                    currentImageIndex = (currentImageIndex + 1) % playerImages.length;
                    playerImage.src = playerImages[currentImageIndex];
                } else if (e.key == "Left" || e.key == "ArrowLeft") {
                    if (player.x > 0) {
                        player.x -= player.speed;
                        player.angle = -Math.PI / 2;
                        currentImageIndex = (currentImageIndex - 1 + playerImages.length) % playerImages.length;
                        playerImage.src = playerImages[currentImageIndex];
                    }
                } else if (e.key == " " && canShoot == true) {
                    bullets.push({
                        x: player.x + player.width / 2 - 2.5,
                        y: player.y,
                        width: 5,
                        height: 10
                    });
                    playerImage.src = 'https://github.com/TayKimmy/CSA_Repo/assets/107821010/28c3e277-b292-43f0-bcef-5460b19689b7';
                    player.angle = 0;
                    canShoot = false;
                    setTimeout(() => {
                        canShoot = true;
                    }, cooldownTime);
                }
            }
        });
        draw();
    </script>
</body>
</html>
