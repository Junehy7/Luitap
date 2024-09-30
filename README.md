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
        canvas {
            background-color: #FFFFFF;
            display: block;
            border: 2px solid #fff;
        }
        #info {
            margin-left: 20px;
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
        #player-input {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-bottom: 20px;
        }
        #ranking {
            font-size: 18px;
            margin-left: 20px;
        }
        #ranking table {
            border-collapse: collapse;
            width: 100%;
            margin-top: 10px;
        }
        #ranking table th, #ranking table td {
            border: 1px solid white;
            padding: 8px;
            text-align: center;
        }
    </style>
</head>
<body>
    <div id="player-input">
        <label for="playerName">Enter your name: </label>
        <input type="text" id="playerName" placeholder="Your Name">
        <button id="startGame">Start Game</button>
    </div>
    <div>
        <canvas id="tetris" width="320" height="640"></canvas>
    </div>
    <div id="info">
        <div id="score">Score: 0</div>
        <div id="next">Next Block:</div>
        <canvas id="nextBlock" width="160" height="128"></canvas>
    </div>
    <div id="ranking">
        <div>Top 5 Players</div>
        <table>
            <thead>
                <tr><th>Rank</th><th>Name</th><th>Score</th></tr>
            </thead>
            <tbody id="rankList">
                <tr><td>1</td><td>-</td><td>0</td></tr>
                <tr><td>2</td><td>-</td><td>0</td></tr>
                <tr><td>3</td><td>-</td><td>0</td></tr>
                <tr><td>4</td><td>-</td><td>0</td></tr>
                <tr><td>5</td><td>-</td><td>0</td></tr>
            </tbody>
        </table>
    </div>

    <script>
        const playerInputDiv = document.getElementById('player-input');
        const playerNameInput = document.getElementById('playerName');
        const startGameBtn = document.getElementById('startGame');
        let playerName = '';
        let isPaused = false;
        let gameOver = false;
        let score = 0;

        startGameBtn.addEventListener('click', () => {
            playerName = playerNameInput.value.trim();
            if (playerName) {
                playerInputDiv.style.display = 'none'; // Hide name input
                startGame();
            } else {
                alert("Please enter your name.");
            }
        });

        const canvas = document.getElementById('tetris');
        const context = canvas.getContext('2d');
        const nextCanvas = document.getElementById('nextBlock');
        const nextContext = nextCanvas.getContext('2d');
        const scoreElement = document.getElementById('score');

        const grid = 32;

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
                    if (tetromino.y === 0) {
                        endGame();
                        return;
                    }
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

           
