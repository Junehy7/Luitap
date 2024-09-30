<!DOCTYPE html>
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
        }
        .game-container {
            display: flex;
        }
        canvas {
            background-color: #FFFFFF;
            display: block;
            border: 2px solid #fff;
            margin-right: 20px; /* Move canvas left by adding margin to the right */
        }
        #info {
            font-family: Arial, sans-serif;
        }
        #score {
            font-size: 24px;
            margin-bottom: 10px;
        }
        #next {
            font-size: 18px;
        }
        #next canvas {
            margin-top: 10px;
        }
        #player-name {
            font-size: 20px;
            margin-bottom: 10px;
        }
        #scoreboard {
            margin-left: 20px;
            font-size: 18px;
        }
        #scoreboard ul {
            list-style-type: none;
            padding: 0;
        }
        #scoreboard li {
            margin-bottom: 5px;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <div>
            <input type="text" id="player-name" placeholder="Enter your name" />
            <canvas id="tetris" width="320" height="640"></canvas>
        </div>
        <div id="info">
            <div id="score">Score: 0</div>
            <div id="next">Next Block:</div>
            <canvas id="nextBlock" width="160" height="128"></canvas>
        </div>
        <div id="scoreboard">
            <h3>Scoreboard</h3>
            <ul id="topScores">
                <li>No players yet</li>
            </ul>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('tetris');
        const context = canvas.getContext('2d');
        const nextCanvas = document.getElementById('nextBlock');
        const nextContext = nextCanvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const playerNameInput = document.getElementById('player-name');
        const topScoresElement = document.getElementById('topScores');

        const grid = 32;
        let score = 0;
        let isPaused = false;
        let gameOver = false;
        let playerName = '';

        const tetrominoes = [
            { shape: [[1, 1, 1, 1]], color: 'cyan' }, // I
            { shape: [[1, 1], [1, 1]], color: 'yellow' }, // O
            { shape: [[0, 1, 0], [1, 1, 1]], color: 'purple' }, // T
            { shape: [[1, 1, 0], [0, 1, 1]], color: 'green' }, // S
            { shape: [[0, 1, 1], [1, 1, 0]], color: 'red' }, // Z
            { shape: [[1, 0, 0], [1, 1, 1]], color: 'orange' }, // L
            { shape: [[0, 0, 1], [1, 1, 1]], color: 'blue' } // J
        ];

        function randomTetromino() {
            const rand = Math.floor(Math.random() * tetrominoes.length);
            return { ...tetrominoes[rand], x: 3, y: 0 };
        }

        const board = Array.from({ length: 20 }, () => Array(10).fill(0));
        let tetromino = randomTetromino();
        let nextTetromino = randomTetromino();

        function drawTetromino(tetromino, context) {
            context.fillStyle = tetromino.color;
            tetromino.shape.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value) {
                        context.fillRect((tetromino.x + x) * grid, (tetromino.y + y) * grid, grid, grid);
                        context.strokeStyle = '#000';
                        context.strokeRect((tetromino.x + x) * grid, (tetromino.y + y) * grid, grid, grid);
                    }
                });
            });
        }

        function isColliding(tetromino) {
            return tetromino.shape.some((row, dy) => {
                return row.some((value, dx) => {
                    let newX = tetromino.x + dx;
                    let newY = tetromino.y + dy;
                    return value && (newX < 0 || newX >= 10 || newY >= 20 || board[newY] && board[newY][newX]);
                });
            });
        }

        function mergeTetromino(tetromino) {
            tetromino.shape.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value) {
                        board[tetromino.y + y][tetromino.x + x] = 1;
                    }
                });
            });
        }

        function clearRows() {
            let rowsCleared = 0;
            for (let y = board.length - 1; y >= 0; --y) {
                if (board[y].every(value => value > 0)) {
                    board.splice(y, 1);
                    board.unshift(Array(10).fill(0));
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
            if (isPaused || gameOver) return;

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
                    nextTetromino = randomTetromino();
                    drawNextBlock();
                }
                dropCounter = 0;
            }

            context.clearRect(0, 0, canvas.width, canvas.height);
            drawTetromino(tetromino, context);

            board.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value) {
                        context.fillStyle = 'gray';
                        context.fillRect(x * grid, y * grid, grid, grid);
                        context.strokeStyle = '#000';
                        context.strokeRect(x * grid, y * grid, grid, grid);
                    }
                });
            });

            requestAnimationFrame(update);
        }

        function drawNextBlock() {
            nextContext.clearRect(0, 0, nextCanvas.width, nextCanvas.height);
            if (!gameOver) {
                nextTetromino.x = 1; // Center the block in next block canvas
                nextTetromino.y = 1;
                drawTetromino(nextTetromino, nextContext);
            }
        }

        function hardDrop() {
            while (!isColliding(tetromino)) {
                tetromino.y++;
            }
            tetromino.y--;
            mergeTetromino(tetromino);
            clearRows();
            tetromino = nextTetromino;
            nextTetromino = randomTetromino();
            drawNextBlock();
        }

        function pauseGame() {
            isPaused = !isPaused;
        }

        document.addEventListener('keydown', event => {
            if (gameOver) return;

            if (event.key === 'p') {
                pauseGame();
            } else if (event.key === 'ArrowLeft') {
                tetromino
