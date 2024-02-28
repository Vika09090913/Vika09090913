<!DOCTYPE html>
<html lang="uk">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #3498db; /* синій фон поза чорною рамкою */
        }

        canvas {
    display: block;
    margin: auto;
    border: 2px solid #000; /* чорна рамка для контуру гри */
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #ffffff; /* білий відтінок зеленого в середині чорної рамки */
}

    </style>
    <title>Весняна казка</title>
</head>

<body>
    <canvas id="gameCanvas" width="1000" height="600"></canvas>
    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        class Player {
            constructor() {
                this.width = 50;
                this.height = 50;
                this.x = canvas.width / 2 - this.width / 2;
                this.y = canvas.height / 2 - this.height / 2;
                this.color = "#00FF00";
                this.speed = 5;
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x, this.y, this.width, this.height);
            }

            update() {
                if (keys.up && this.y > 0) {
                    this.y -= this.speed;
                }
                if (keys.down && this.y < canvas.height - this.height) {
                    this.y += this.speed;
                }
                if (keys.left && this.x > 0) {
                    this.x -= this.speed;
                }
                if (keys.right && this.x < canvas.width - this.width) {
                    this.x += this.speed;
                }
            }
        }

        class Flower {
            constructor(x, y, size, color) {
                this.x = x;
                this.y = y;
                this.size = size;
                this.color = color || "#FF8C00"; // темніший відтінок жовтого за замовчуванням
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x + this.size / 2, this.y + this.size / 2, this.size / 2, 0, 2 * Math.PI);
                ctx.fill();
            }

            update() {
                this.x -= 1;
                this.y += 1;

                if (this.x + this.size < 0 || this.y > canvas.height) {
                    this.x = canvas.width;
                    this.y = Math.floor(Math.random() * (canvas.height - this.size));
                    this.color = getRandomSpringColor();
                }
            }
        }

        class Obstacle {
            constructor(x, y, size, color) {
                this.x = x;
                this.y = y;
                this.size = size;
                this.color = color || "#FF8C00"; // темніший відтінок жовтого за замовчуванням
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x, this.y, this.size, this.size);
            }

            update() {
                this.x -= 1;
                this.y += 1;

                if (this.x + this.size < 0 || this.y > canvas.height) {
                    this.x = canvas.width;
                    this.y = Math.floor(Math.random() * (canvas.height - this.size));
                    this.color = getRandomSpringColor();
                }
            }
        }

        class Sun {
            constructor() {
                this.radius = 30;
                this.x = canvas.width - 50;
                this.y = 50;
                this.color = "#FFD700"; /* жовтий колір для сонця */
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, 2 * Math.PI);
                ctx.fill();

                /* Промені сонця */
                ctx.strokeStyle = this.color;
                ctx.lineWidth = 2;
                for (let i = 0; i < 12; i++) {
                    const angle = (i / 12) * (2 * Math.PI);
                    const startX = this.x + Math.cos(angle) * this.radius;
                    const startY = this.y + Math.sin(angle) * this.radius;
                    const endX = this.x + Math.cos(angle) * (this.radius + 20);
                    const endY = this.y + Math.sin(angle) * (this.radius + 20);
                    ctx.beginPath();
                    ctx.moveTo(startX, startY);
                    ctx.lineTo(endX, endY);
                    ctx.stroke();
                }
            }
        }

        const player = new Player();
        const flowers = [];
        const obstacles = [];
        const sun = new Sun();
        const keys = {
            up: false,
            down: false,
            left: false,
            right: false
        };
        let score = 0;
        const maxScore = 5;
        let timer = 60;
        let lastTime = 0;
        let gameEnded = false;

        document.addEventListener("keydown", (event) => {
            handleKeyPress(event.key, true);
        });

        document.addEventListener("keyup", (event) => {
            handleKeyPress(event.key, false);
        });

        function handleKeyPress(key, value) {
            switch (key) {
                case "ArrowUp":
                    keys.up = value;
                    break;
                case "ArrowDown":
                    keys.down = value;
                    break;
                case "ArrowLeft":
                    keys.left = value;
                    break;
                case "ArrowRight":
                    keys.right = value;
                    break;
            }
        }

        function drawScore() {
            ctx.fillStyle = "#000000";
            ctx.font = "24px Arial";
            ctx.fillText("Score: " + score, 10, 30);
        }

        function drawTimer() {
            ctx.fillStyle = "#000000";
            ctx.font = "24px Arial";
            ctx.fillText("Time: " + timer + "s", canvas.width - 120, 30);
        }

        function drawGameOver() {
            ctx.fillStyle = "#3C3C3C"; /* темний колір для "Game Over" */
            ctx.font = "40px Arial";
            ctx.fillText("Game Over", canvas.width / 2 - 120, canvas.height / 2 - 20);
        }

        function drawWinner() {
            ctx.fillStyle = "#3C3C3C"; /* темний колір для "Winner!" */
            ctx.font = "40px Arial";
            ctx.fillText("Winner!", canvas.width / 2 - 80, canvas.height / 2 - 20);
        }

        function gameLoop(timestamp) {
            if (gameEnded) return;

            player.update();
            flowers.forEach(flower => flower.update());
            obstacles.forEach(obstacle => obstacle.update());

            if (timestamp - lastTime >= 1000) {
                lastTime = timestamp;
                if (timer > 0) {
                    timer--;
                } else {
                    gameEnded = true;
                    drawGameOver();
                    return;
                }
            }

            for (const flower of flowers) {
                if (
                    player.x < flower.x + flower.size &&
                    player.x + player.width > flower.x &&
                    player.y < flower.y + flower.size &&
                    player.y + player.height > flower.y
                ) {
                    score++;
                    flowers.splice(flowers.indexOf(flower), 1);
                    flowers.push(new Flower(
                        canvas.width - Math.floor(Math.random() * 200),
                        Math.floor(Math.random() * (canvas.height - 50)),
                        Math.floor(Math.random() * 30) + 20,
                        getRandomSpringColor()
                    ));
                }
            }

            if (score >= maxScore) {
                gameEnded = true;
                drawWinner();
                return;
            }

            for (const obstacle of obstacles) {
                if (
                    player.x < obstacle.x + obstacle.size &&
                    player.x + player.width > obstacle.x &&
                    player.y < obstacle.y + obstacle.size &&
                    player.y + player.height > obstacle.y
                ) {
                    gameEnded = true;
                    drawGameOver();
                    return;
                }
            }

            for (const obstacle of obstacles) {
                if (obstacle.x + obstacle.size < 0) {
                    obstacles.splice(obstacles.indexOf(obstacle), 1);
                    obstacles.push(new Obstacle(
                        canvas.width - Math.floor(Math.random() * 200),
                        Math.floor(Math.random() * (canvas.height - 50)),
                        Math.floor(Math.random() * 30) + 20,
                        getRandomSpringColor()
                    ));
                    score--;
                }
            }

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            player.draw();
            flowers.forEach(flower => flower.draw());
            obstacles.forEach(obstacle => obstacle.draw());
            sun.draw();
            drawGrass();
            drawScore();
            drawTimer();

            requestAnimationFrame(gameLoop);
        }

        for (let i = 0; i < 5; i++) {
            flowers.push(new Flower(
                canvas.width - Math.floor(Math.random() * 200),
                Math.floor(Math.random() * (canvas.height - 50)),
                Math.floor(Math.random() * 30) + 20,
                getRandomSpringColor()
            ));
        }

        for (let i = 0; i < 5; i++) {
            obstacles.push(new Obstacle(
                canvas.width - Math.floor(Math.random() * 200),
                Math.floor(Math.random() * (canvas.height - 50)),
                Math.floor(Math.random() * 30) + 20,
                getRandomSpringColor()
            ));
        }

        gameLoop();

        function getRandomSpringColor() {
            const springColors = ["#FF69B4", "#00CED1", "#FFFF00", "#00FF00"];
            return springColors[Math.floor(Math.random() * springColors.length)];
        }

        function drawGrass() {
            ctx.fillStyle = "#388E3C"; /* трав'янистий колір для трави */
            ctx.fillRect(0, canvas.height - 20, canvas.width, 20);
        }
    </script>
</body>

</html>
