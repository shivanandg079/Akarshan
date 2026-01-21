<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
    <title>Iron Man: Pixel Flight</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #111;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        canvas {
            background: linear-gradient(#0a192f, #112240);
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
            max-width: 100%;
            max-height: 100%;
        }
        #ui {
            position: absolute;
            color: white;
            text-align: center;
            pointer-events: none;
            user-select: none;
        }
        .hidden { display: none; }
        #gameOver {
            background: rgba(0, 0, 0, 0.8);
            padding: 20px;
            border-radius: 10px;
            pointer-events: auto;
        }
        button {
            padding: 10px 20px;
            font-size: 18px;
            background: #00d4ff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <div id="ui">
        <div id="scoreDisplay" style="font-size: 30px; margin-top: 20px;">Score: 0</div>
        <div id="gameOver" class="hidden">
            <h1>SYSTEM FAILURE</h1>
            <p>Score: <span id="finalScore">0</span></p>
            <p>Best: <span id="highScore">0</span></p>
            <button onclick="resetGame()">REBOOT SYSTEM</button>
        </div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreEl = document.getElementById('scoreDisplay');
        const gameOverEl = document.getElementById('gameOver');
        const finalScoreEl = document.getElementById('finalScore');
        const highScoreEl = document.getElementById('highScore');

        // Game Constants
        const GRAVITY = 0.25;
        const JUMP = -5;
        const PIPE_WIDTH = 60;
        const PIPE_GAP = 160;
        const PIPE_SPACING = 250;

        // Game State
        let player = { x: 50, y: 200, v: 0, w: 50, h: 50 };
        let pipes = [];
        let score = 0;
        let gameActive = true;
        let speedMultiplier = 1;
        let frameCount = 0;

        // Load Asset
        const ironManImg = new Image();
        // Using the data URI of your uploaded image
        ironManImg.src = 'https://i.postimg.cc/8zP6zVp0/image-e61a621.png'; 

        function init() {
            canvas.width = 400;
            canvas.height = 600;
            resetGame();
            loop();
        }

        function resetGame() {
            player.y = canvas.height / 2;
            player.v = 0;
            pipes = [];
            score = 0;
            speedMultiplier = 1;
            gameActive = true;
            gameOverEl.classList.add('hidden');
            scoreEl.classList.remove('hidden');
            // Add first pipe
            spawnPipe(600);
        }

        function spawnPipe(xPos) {
            const minHeight = 50;
            const maxHeight = canvas.height - PIPE_GAP - minHeight;
            const topHeight = Math.floor(Math.random() * (maxHeight - minHeight + 1)) + minHeight;
            pipes.push({ x: xPos, y: topHeight, passed: false });
        }

        // Input Handling
        window.addEventListener('touchstart', (e) => { 
            if(gameActive) player.v = JUMP; 
            e.preventDefault();
        }, {passive: false});
        
        window.addEventListener('mousedown', () => { 
            if(gameActive) player.v = JUMP; 
        });

        function update() {
            if (!gameActive) return;

            // Player Physics
            player.v += GRAVITY;
            player.y += player.v;

            // Floor/Ceiling collisions
            if (player.y + player.h > canvas.height || player.y < 0) {
                endGame();
            }

            // Pipe Logic
            const currentSpeed = 3 * speedMultiplier;
            
            for (let i = pipes.length - 1; i >= 0; i--) {
                let p = pipes[i];
                p.x -= currentSpeed;

                // Collision Detection
                if (
                    player.x < p.x + PIPE_WIDTH &&
                    player.x + player.w - 10 > p.x && // Small padding for hitbox
                    (player.y + 10 < p.y || player.y + player.h - 10 > p.y + PIPE_GAP)
                ) {
                    endGame();
                }

                // Scoring
                if (!p.passed && p.x + PIPE_WIDTH < player.x) {
                    p.passed = true;
                    score++;
                    scoreEl.innerText = `Score: ${score}`;
                    
                    // Difficulty Scaling: 5% faster every 10 points
                    if (score % 10 === 0) {
                        speedMultiplier *= 1.05;
                    }
                }

                // Remove old pipes
                if (p.x + PIPE_WIDTH < 0) pipes.splice(i, 1);
            }

            // Spawn new pipes
            if (pipes.length > 0 && pipes[pipes.length - 1].x < canvas.width - PIPE_SPACING) {
                spawnPipe(canvas.width);
            }
        }

        function draw() {
            // Clear Background
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw Pipes (Neon Tech Style)
            pipes.forEach(p => {
                ctx.fillStyle = "#00d4ff";
                ctx.shadowBlur = 15;
                ctx.shadowColor = "#00d4ff";
                // Top Pipe
                ctx.fillRect(p.x, 0, PIPE_WIDTH, p.y);
                // Bottom Pipe
                ctx.fillRect(p.x, p.y + PIPE_GAP, PIPE_WIDTH, canvas.height);
                ctx.shadowBlur = 0;
            });

            // Draw Iron Man
            ctx.drawImage(ironManImg, player.x, player.y, player.w, player.h);
        }

        function endGame() {
            gameActive = false;
            const high = localStorage.getItem('highScore') || 0;
            if (score > high) {
                localStorage.setItem('highScore', score);
            }
            finalScoreEl.innerText = score;
            highScoreEl.innerText = localStorage.getItem('highScore');
            gameOverEl.classList.remove('hidden');
            scoreEl.classList.add('hidden');
        }

        function loop() {
            update();
            draw();
            requestAnimationFrame(loop);
        }

        ironManImg.onload = init;
    </script>
</body>
</html><img width="1024" height="1024" alt="9036" src="https://github.com/user-attachments/assets/29770ba5-0eb7-463e-8864-9a91530a7b01" />

