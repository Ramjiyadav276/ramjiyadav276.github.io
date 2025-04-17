<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flappy Bird Clone</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #70c5ce;
            font-family: Arial, sans-serif;
            overflow: hidden;
        }
        
        #game-container {
            position: relative;
            width: 400px;
            height: 600px;
            overflow: hidden;
            background-image: url('https://iili.io/30DvYRj.png');
            background-size: cover;
            border: 4px solid #333;
            border-radius: 8px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
        }
        
        #bird {
            position: absolute;
            width: 100px;
            height: 70px;
            background-image: url('https://iili.io/30DvJVI.png');
            background-size: contain;
            background-repeat: no-repeat;
            z-index: 10;
            transition: transform 0.1s ease;
        }
        
        .pipe {
            position: absolute;
            width: 70px;
            background-image: url('https://iili.io/30mdrua.png');
            background-size: cover;
            background-repeat: no-repeat;
            z-index: 5;
        }
        
        .pipe-top {
            top: 0;
            /* Removed transform: rotate(180deg) to use same red pipe image orientation as bottom */
        }
        
        .pipe-bottom {
            bottom: 0;
        }
        
        #score-display {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 36px;
            font-weight: bold;
            color: white;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            z-index: 20;
        }
        
        #game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.8);
            color: white;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            display: none;
            z-index: 30;
        }
        
        #restart-btn {
            background-color: #ff6b6b;
            color: white;
            border: none;
            padding: 10px 20px;
            font-size: 18px;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 15px;
        }
        
        #restart-btn:hover {
            background-color: #ff5252;
        }
        
        #ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 60px;
            background-color: #deb887;
            border-top: 3px solid #8b4513;
            z-index: 7;
        }

        #start-message {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 15px;
            border-radius: 10px;
            text-align: center;
            z-index: 25;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <div id="bird"></div>
        <div id="ground"></div>
        <div id="score-display">0</div>
        <div id="game-over">
            <h2>Game Over!</h2>
            <p>Your score: <span id="final-score">0</span></p>
            <button id="restart-btn">Play Again</button>
        </div>
        <div id="start-message">
            <h2>Flappy Bird</h2>
            <p>Press SPACE or CLICK to start</p>
        </div>
    </div>

    <!-- Audio element for score sound -->
    <audio id="score-sound" src="https://assets.mixkit.co/sfx/preview/mixkit-achievement-bell-600.mp3" preload="auto"></audio>

    <script>
        // Game variables
        const bird = document.getElementById('bird');
        const gameContainer = document.getElementById('game-container');
        const scoreDisplay = document.getElementById('score-display');
        const gameOverScreen = document.getElementById('game-over');
        const finalScoreDisplay = document.getElementById('final-score');
        const restartBtn = document.getElementById('restart-btn');
        const ground = document.getElementById('ground');
        const startMessage = document.getElementById('start-message');
        const scoreSound = document.getElementById('score-sound');
        
        let birdY = 300;
        let birdVelocity = 0;
        let gravity = 0.2;
        let jumpForce = -5.5;
        let gameRunning = false;
        let score = 0;
        let pipes = [];
        let pipeSpeed = 2.0;
        let pipeGap = 150; // Reduced from 200 to 150 (smaller gap between pipes)
        let pipeFrequency = 1800;
        let lastPipeTime = 0;
        let animationId;
        let gameStarted = false;
        let initialDelay = 3000;
        let gameStartTime = 0;
        
        // Initialize game
        function initGame() {
            birdY = 300;
            birdVelocity = 0;
            score = 0;
            pipes = [];
            scoreDisplay.textContent = '0';
            gameOverScreen.style.display = 'none';
            startMessage.style.display = 'none';
            
            // Clear existing pipes
            document.querySelectorAll('.pipe').forEach(pipe => pipe.remove());
            
            // Position bird
            bird.style.top = birdY + 'px';
            bird.style.left = '80px';
            bird.style.transform = 'rotate(0deg)';
            
            gameRunning = true;
            gameStarted = true;
            gameStartTime = Date.now(); // Record start time for delay
            
            // Start game loop
            gameLoop();
        }
        
        // Game loop
        function gameLoop() {
            if (!gameRunning) return;
            
            // Update bird position
            birdVelocity += gravity;
            birdY += birdVelocity;
            bird.style.top = birdY + 'px';
            
            // Rotate bird based on velocity
            let rotation = Math.min(Math.max(birdVelocity * 5, -25), 90);
            bird.style.transform = `rotate(${rotation}deg)`;
            
            // Check for collisions with ceiling or ground
            if (birdY <= 0) {
                console.log("Game Over: Hit ceiling");
                gameOver();
                return;
            }
            if (birdY + 70 >= gameContainer.offsetHeight - ground.offsetHeight) {
                console.log("Game Over: Hit ground");
                gameOver();
                return;
            }
            
            // Generate pipes after initial delay
            const currentTime = Date.now();
            if (currentTime >= gameStartTime + initialDelay && currentTime - lastPipeTime > pipeFrequency) {
                createPipe();
                lastPipeTime = currentTime;
            }
            
            // Move pipes and check for collisions
            movePipes();
            
            // Continue the game loop
            animationId = requestAnimationFrame(gameLoop);
        }
        
        // Create a new pipe
        function createPipe() {
            const minHeight = 50;
            const maxHeight = gameContainer.offsetHeight - pipeGap - ground.offsetHeight - minHeight;
            const pipeHeight = Math.floor(Math.random() * (maxHeight - minHeight)) + minHeight;
            
            // Top pipe
            const topPipe = document.createElement('div');
            topPipe.className = 'pipe pipe-top';
            topPipe.style.height = pipeHeight + 'px';
            topPipe.style.left = gameContainer.offsetWidth + 'px';
            gameContainer.appendChild(topPipe);
            
            // Bottom pipe
            const bottomPipe = document.createElement('div');
            bottomPipe.className = 'pipe pipe-bottom';
            bottomPipe.style.height = (gameContainer.offsetHeight - pipeHeight - pipeGap - ground.offsetHeight) + 'px';
            bottomPipe.style.left = gameContainer.offsetWidth + 'px';
            bottomPipe.style.bottom = ground.offsetHeight + 'px';
            gameContainer.appendChild(bottomPipe);
            
            pipes.push({
                top: topPipe,
                bottom: bottomPipe,
                passed: false
            });
        }
        
        // Move all pipes
        function movePipes() {
            for (let i = 0; i < pipes.length; i++) {
                const pipe = pipes[i];
                const currentLeft = parseInt(pipe.top.style.left);
                const newLeft = currentLeft - pipeSpeed;
                
                pipe.top.style.left = newLeft + 'px';
                pipe.bottom.style.left = newLeft + 'px';
                
                // Check if bird passed the pipe
                if (!pipe.passed && newLeft < 80 - 70) {
                    pipe.passed = true;
                    score++;
                    scoreDisplay.textContent = score;
                    
                    // Play score sound
                    scoreSound.currentTime = 0;
                    scoreSound.play();
                    
                    // Increase difficulty slightly
                    if (score % 5 === 0) {
                        pipeSpeed += 0.1;
                        pipeFrequency = Math.max(1500, pipeFrequency - 30);
                    }
                }
                
                // Check for collisions with pipe body
                const birdLeft = 80;
                const birdRight = 80 + 95; // Slightly reduced for precision
                const birdTop = birdY;
                const birdBottom = birdY + 65; // Slightly reduced for precision
                const pipeLeft = newLeft + 6; // Increased buffer for strict collision
                const pipeRight = newLeft + 70 - 6; // Increased buffer for strict collision
                const topPipeBottom = parseInt(pipe.top.style.height);
                const bottomPipeTop = gameContainer.offsetHeight - parseInt(pipe.bottom.style.height) - ground.offsetHeight;
                
                // Only log when bird is near pipe (within 20px horizontally)
                if (Math.abs(newLeft - 80) < 20) {
                    if (
                        (birdRight > pipeLeft && birdLeft < pipeRight) && 
                        (birdTop < topPipeBottom - 5 || birdBottom > bottomPipeTop + 5)
                    ) {
                        console.log("Game Over: Hit pipe body", {
                            birdLeft, birdRight, birdTop, birdBottom,
                            pipeLeft, pipeRight, topPipeBottom, bottomPipeTop,
                            pipeX: newLeft,
                            overlapX: Math.min(birdRight, pipeRight) - Math.max(birdLeft, pipeLeft),
                            overlapY: birdTop < topPipeBottom - 5 ? topPipeBottom - birdTop : birdBottom - bottomPipeTop
                        });
                        gameOver();
                        return;
                    } else {
                        console.log("No collision (near pipe)", {
                            birdY, birdTop, birdBottom,
                            pipeX: newLeft, pipeLeft, pipeRight,
                            topPipeBottom, bottomPipeTop,
                            distanceX: Math.min(Math.abs(birdLeft - pipeRight), Math.abs(birdRight - pipeLeft)),
                            distanceY: Math.min(Math.abs(birdTop - topPipeBottom), Math.abs(birdBottom - bottomPipeTop))
                        });
                    }
                }
                
                // Remove pipes that are off screen
                if (newLeft < -70) {
                    pipe.top.remove();
                    pipe.bottom.remove();
                    pipes.splice(i, 1);
                    i--;
                }
            }
        }
        
        // Game over
        function gameOver() {
            gameRunning = false;
            cancelAnimationFrame(animationId);
            finalScoreDisplay.textContent = score;
            gameOverScreen.style.display = 'block';
        }
        
        // Jump function
        function jump() {
            if (!gameRunning) {
                initGame();
                return;
            }
            
            birdVelocity = jumpForce;
            bird.style.transform = 'rotate(-25deg)';
        }
        
        // Event listeners
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space' || e.key === ' ' || e.key === 'ArrowUp') {
                jump();
                e.preventDefault();
            }
        });
        
        gameContainer.addEventListener('click', jump);
        restartBtn.addEventListener('click', initGame);
        
        // Initialize bird position
        bird.style.top = birdY + 'px';
        bird.style.left = '80px';
    </script>
</body>
</html>
