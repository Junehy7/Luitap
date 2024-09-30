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
            flex-direction: column;
            height: 100vh;
            background-color: #9ACD32;
            margin: 0;
            color: white;
            font-family: Arial, sans-serif;
        }
        #game-container {
            display: flex;
        }
        canvas {
            background-color: #FFFFFF;
            display: block;
            border: 2px solid #fff;
            width: 256px; /* Reduced size */
            height: 512px; /* Reduced size */
        }
        #info {
            margin-left: 20px;
            text-align: left;
        }
        #score, #top-scores {
            font-size: 18px;
            margin-bottom: 10px;
        }
        #next {
            font-size: 16px;
        }
        #next canvas {
            margin-top: 10px;
            width: 128px; /* Reduced size */
            height: 102px; /* Reduced size */
        }
        #player-input {
            margin-bottom: 20px;
            text-align: center;
        }
        #controls {
            margin-top: 10px;
            display: flex;
            justify-content: center;
        }
        .control-button {
            background-color: #fff;
            border: none;
            padding: 10px;
            margin: 5px;
            cursor: pointer;
        }
        #top-scores {
            font-size: 16px;
            display: none; /* Hidden until the game ends */
        }
    </style>
</head>
<body>
    <div id="player-input">
        <input type="text" id="playerName" placeholder="Enter your name" />
        <button id="playButton">Play</button>
    </div>
    <div id="game-container">
        <div>
            <canvas id="tetris" width="256" height="512"></canvas>
        </div>
        <div id="info">
            <div id="score">Score: 0</div>
            <div id="next">Next Block:</div>
            <canvas id="nextBlock" width="128" height="102"></canvas>
            <div id="top-scores">
                <h3>Top Scores</h3>
                <ul id="scoreList"></ul>
            </div>
        </div>
    </div>
    <div id="controls">
        <button class="control-button" id="leftButton">Left</button>
        <button class="control-button" id="rightButton">Right</button>
        <button class="control-button" id="downButton">Down</button>
        <button class="control-button" id="rotateButton">Rotate</button>
        <button class="control-button" id="dropButton">Hard Drop</button>
    </div>

    <script>
        const canvas = document.getElementById('tetris');
        const context = canvas.getContext('2d');
        const nextCanvas = document.getElementById('nextBlock');
        const nextContext = nextCanvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const scoreListElement = document.getElementById('scoreList');
        const playButton = document.getElementById('playButton');
        const playerNameInput = document.getElementById('playerName');
        const topScoresElement = document.getElementById('top-scores');

        const grid = 32;
        let score = 0;
        let isPaused = false;
        let gameEnded = false;
        let playerName = '';
        let scores = [];

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

        const board = Array.from({ length: 16 }, () => Array(8).fill(0)); // Adjusted size for the board
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
                    return value && (newX < 0 || newX >= 8 || newY >= 16 || (board[newY] && board[newY][newX]));
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
                    board.unshift(Array(8).fill(0));
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
            if (isPaused || gameEnded) return; // Exit if paused or game has ended
            
            const deltaTime = time - lastTime;
            lastTime = time;
            dropCounter += deltaTime;

            if (dropCounter > dropInterval) {
                tetromino.y++;
                if (isColliding(tetromino)) {
                    tetromino.y--;
                    mergeTetromino(tetromino);
                    clearRows();
                    if (tetromino.y === 0) {
                        gameEnded = true; // Game over if block reaches the top
                        topScoresElement.style.display = 'block'; // Show scores
                        addScore(); // Add score to leaderboard
                        return; // Stop updating
                    }
                    tetromino = nextTetromino;
                    nextTetromino = randomTetromino();
                    drawNextBlock();
                }
                dropCounter = 0;
            }

            context.clearRect(0, 0
