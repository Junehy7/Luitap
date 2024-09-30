<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tetris Game</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #9ACD32;
            margin: 0;
            color: white;
            font-family: Arial, sans-serif;
        }
        #gameContainer {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #gameArea {
            display: flex;
            justify-content: center;
            align-items: flex-start;
        }
        canvas {
            background-color: #FFFFFF;
            display: block;
            border: 2px solid #fff;
        }
        #info {
            margin-left: 10px;
            width: 120px;
        }
        #score, #next {
            font-size: 16px;
            margin-bottom: 10px;
        }
        #nextBlock {
            margin-top: 5px;
        }
        #leaderboard {
            margin-left: 10px;
            width: 150px;
        }
        #leaderboard h3 {
            margin-top: 0;
        }
        #nameInput {
            margin-bottom: 10px;
        }
        #playButton {
            margin-bottom: 10px;
        }
        #mobileControls {
            display: none;
            margin-top: 10px;
        }
        .mobileButton {
            font-size: 24px;
            margin: 5px;
            padding: 10px;
            background-color: rgba(255, 255, 255, 0.5);
            border: none;
            border-radius: 5px;
        }
        @media (max-width: 600px) {
            #gameArea {
                flex-direction: column;
                align-items: center;
            }
            #info, #leaderboard {
                margin-left: 0;
                margin-top: 10px;
            }
            #mobileControls {
                display: block;
            }
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div id="nameInput">
            <input type="text" id="playerName" placeholder="Enter your name">
            <button id="playButton">Play</button>
        </div>
        <div id="gameArea">
            <div>
                <canvas id="tetris" width="240" height="400"></canvas>
            </div>
            <div id="info">
                <div id="score">Score: 0</div>
                <div id="next">Next Block:</div>
                <canvas id="nextBlock" width="80" height="80"></canvas>
            </div>
            <div id="leaderboard">
                <h3>Top 5 Players</h3>
                <ol id="leaderboardList"></ol>
            </div>
        </div>
        <div id="mobileControls">
            <button class="mobileButton" id="leftButton">←</button>
            <button class="mobileButton" id="rightButton">→</button>
            <button class="mobileButton" id="rotateButton">↻</button>
            <button class="mobileButton" id="downButton">↓</button>
            <button class="mobileButton" id="dropButton">Drop</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('tetris');
        const context = canvas.getContext('2d');
        const nextCanvas = document.getElementById('nextBlock');
        const nextContext = nextCanvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const playButton = document.getElementById('playButton');
        const playerNameInput = document.getElementById('playerName');
        const leaderboardList = document.getElementById('leaderboardList');

        const grid = 20;
        let score = 0;
        let gameRunning = false;
        let gamePaused = false;
        let leaderboard = [];

        const tetrominoes = [
            { shape: [[1, 1, 1, 1]], color: 'cyan' },
            { shape: [[1, 1], [1, 1]], color: 'yellow' },
            { shape: [[0, 1, 0], [1, 1, 1]], color: 'purple' },
            { shape: [[1, 1, 0], [0, 1, 1]], color: 'green' },
            { shape: [[0, 1, 1], [1, 1, 0]], color: 'red' },
            { shape: [[1, 0, 0], [1, 1, 1]], color: 'orange' },
            { shape: [[0, 0, 1], [1, 1, 1]], color: 'blue' }
        ];

        function randomTetromino() {
            const rand = Math.floor(Math.random() * tetrominoes.length);
            return { ...tetrominoes[rand], x: 3, y: 0 };
        }

        const board = Array.from({ length: 20 }, () => Array(12).fill(0));
        let tetromino = randomTetromino();
        let nextTetromino = randomTetromino();

        function drawTetromino(tetromino, context, offsetX = 0, offsetY = 0) {
            context.fillStyle = tetromino.color;
            tetromino.shape.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value) {
                        context.fillRect((tetromino.x + x + offsetX) * grid, (tetromino.y + y + offsetY) * grid, grid - 1, grid - 1);
                        context.strokeStyle = '#000';
                        context.strokeRect((tetromino.x + x + offsetX) * grid, (tetromino.y + y + offsetY) * grid, grid - 1, grid - 1);
                    }
                });
            });
        }

        function isColliding(tetromino) {
            return tetromino.shape.some((row, dy) => {
                return row.some((value, dx) => {
                    let newX = tetromino.x + dx;
                    let newY = tetromino.y + dy;
                    return value && (newX < 0 || newX >= 12 || newY >= 20 || board[newY] && board[newY][newX]);
                });
            });
        }

        function mergeTetromino(tetromino) {
            tetromino.shape.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value) {
                        board[tetromino.y + y][tetromino.x + x] = tetromino.color;
                    }
                });
            });
        }

        function clearRows() {
            let rowsCleared = 0;
            for (let y = board.length - 1; y >= 0; --y) {
                if (board[y].every(value => value !== 0)) {
                    board.splice(y, 1);
                    board.unshift(Array(12).fill(0));
                    rowsCleared++;
                }
            }
            if (rowsCleared > 0) {
                score += rowsCleared * 100;
                scoreElement.textContent = 'Score: ' + score;
            }
        }

        function rotate(tetromino) {
            const rotatedShape = tetromino.shape[0].map((_, i) => tetromino.shape.map(row => row[i])).reverse();
            const newTetromino = { ...tetromino, shape: rotatedShape };
            return isColliding(newTetromino) ? tetromino : newTetromino;
        }

        let dropCounter = 0;
        let dropInterval = 1000;
        let lastTime = 0;

        function update(time = 0) {
            if (!gameRunning || gamePaused) return;

            const deltaTime = time - lastTime;
            lastTime = time;
            dropCounter += deltaTime;

            if (dropCounter > dropInterval) {
                tetromino.y++;
                if (isColliding(tetromino)) {
                    tetromino.y--;
                    mergeTetromino(tetromino);
                    clearRows();
                    tetromino = nextTetromino;
                    if (isColliding(tetromino)) {
                        gameOver();
                        return;
                    }
                    nextTetromino = randomTetromino();
                    drawNextBlock();
                }
                dropCounter = 0;
            }

            draw();
            requestAnimationFrame(update);
        }

        function draw() {
            context.clearRect(0, 0, canvas.width, canvas.height);

            board.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value) {
                        context.fillStyle = value;
                        context.fillRect(x * grid, y * grid, grid - 1, grid - 1);
                        context.strokeStyle = '#000';
                        context.strokeRect(x * grid, y * grid, grid - 1, grid - 1);
                    }
                });
            });

            drawTetromino(tetromino, context);
        }

        function drawNextBlock() {
            nextContext.clearRect(0, 0, nextCanvas.width, nextCanvas.height);
            drawTetromino(nextTetromino, nextContext, -2, -1);
        }

        function hardDrop() {
            while (!isColliding(tetromino)) {
                tetromino.y++;
            }
            tetromino.y--;
            mergeTetromino(tetromino);
            clearRows();
            tetromino = nextTetromino;
            if (isColliding(tetromino)) {
                gameOver();
                return;
            }
            nextTetromino = randomTetromino();
            drawNextBlock();
        }

        function gameOver() {
            gameRunning = false;
            updateLeaderboard();
            alert('Game Over! Your score: ' + score);
        }

        function updateLeaderboard() {
            const playerName = playerNameInput.value || 'Anonymous';
            leaderboard.push({ name: playerName, score: score });
            leaderboard.sort((a, b) => b.score - a.score);
            leaderboard = leaderboard.slice(0, 5);
            displayLeaderboard();
        }

        function displayLeaderboard() {
            leaderboardList.innerHTML = '';
            leaderboard.forEach(entry => {
                const li = document.createElement('li');
                li.textContent = `${entry.name}: ${entry.score}`;
                leaderboardList.appendChild(li);
            });
        }

        function startGame() {
            if (gameRunning) return;

            board.forEach(row => row.fill(0));
            score = 0;
            scoreElement.textContent = 'Score: 0';
            tetromino = randomTetromino();
            nextTetromino = randomTetromino();
            drawNextBlock();
            gameRunning = true;
            gamePaused = false;
            update();
        }

        function togglePause() {
            if (!gameRunning) return;

            gamePaused = !gamePaused;
            if (!gamePaused) {
                update();
            }
        }

        document.addEventListener('keydown', event => {
            if (!gameRunning || gamePaused) return;

            if (event.key === 'ArrowLeft') {
                tetromino.x--;
                if (isColliding(tetromino)) {
                    tetromino.x++;
                }
            } else if (event.key === 'ArrowRight') {
                tetromino.x++;
                if (isColliding(tetromino)) {
                    tetromino.x--;
                }
            } else if (event.key === 'ArrowDown') {
                tetromino.y++;
                if (isColliding(tetromino)) {
                    tetromino.y--;
                }
            } else if (event.key === 'ArrowUp') {
                tetromino = rotate(tetromino);
            } else if (event.key === ' ') {
                hardDrop();
            } else if (event.key === 'p') {
                togglePause();
            }
        });

        playButton.addEventListener('click', startGame);

        // Mobile controls
        document.getElementById('leftButton').addEventListener('click', () => {
            if (!gameRunning || gamePaused) return;
            tetromino.x--;
            if (isColliding(tetromino)) {
                tetromino.x++;
            }
        });

        document.getElementById('rightButton').addEventListener('click', () => {
            if (!gameRunning || gamePaused) return;
            tetromino.x++;
            if (isColliding(tetromino)) {
                tetromino.x--;
            }
        });

        document.getElementById('rotateButton').addEventListener('click', () => {
            if (!gameRunning || gamePaused) return;
            tetromino = rotate(tetromino);
        });

        document.getElementById('downButton').addEventListener('click', () => {
            if (!gameRunning || gamePaused) return;
            tetromino.y++;
            if (isColliding(tetromino)) {
                tetromino.y--;
            }
        });

        document.getElementById('dropButton').addEventListener('click', () => {
            if (!gameRunning || gamePaused) return;
            hardDrop();
        });

        draw();
    </script>
</body>
</html>
