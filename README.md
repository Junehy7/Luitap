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
            flex-direction: column;
        }
        #game-container {
            display: flex;
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
        #controls {
            display: flex;
            justify-content: center;
            margin-top: 20px;
        }
        .control-button {
            background-color: #fff;
            color: #000;
            border: none;
            padding: 10px;
            margin: 5px;
            font-size: 18px;
            cursor: pointer;
        }
        #rank-board {
            margin-left: 20px;
            font-family: Arial, sans-serif;
        }
        #rank-board h2 {
            font-size: 24px;
            margin-bottom: 10px;
        }
        #rank-board ol {
            padding-left: 20px;
        }
    </style>
</head>
<body>
    <div>
        <input type="text" id="playerName" placeholder="Enter your name">
        <button onclick="startGame()">Play</button>
    </div>
    <div id="game-container">
        <div>
            <canvas id="tetris" width="320" height="640"></canvas>
        </div>
        <div id="info">
            <div id="score">Score: 0</div>
            <div id="next">Next Block:</div>
            <canvas id="nextBlock" width="160" height="128"></canvas>
        </div>
        <div id="rank-board">
            <h2>Top 5 Players</h2>
            <ol id="rankList">
                <!-- Player scores will be listed here -->
            </ol>
        </div>
    </div>
    <div id="controls">
        <button class="control-button" onclick="moveLeft()">Left</button>
        <button class="control-button" onclick="moveRight()">Right</button>
        <button class="control-button" onclick="rotate()">Rotate</button>
        <button class="control-button" onclick="moveDown()">Down</button>
        <button class="control-button" onclick="hardDrop()">Drop</button>
    </div>

    <script>
        const canvas = document.getElementById('tetris');
        const context = canvas.getContext('2d');
        const nextCanvas = document.getElementById('nextBlock');
        const nextContext = nextCanvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const rankList = document.getElementById('rankList');

        const grid = 32;
        let score = 0;
        let playerName = '';
        let isPaused = false;
        let gameOver = false;

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
                        gameOver = true;
                        updateRankBoard();
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

            requestAnimationFrame(update);
        }

        function drawNextBlock() {
            nextContext.clearRect(0, 0, nextCanvas.width, nextCanvas.height);
            nextTetromino.x = 1; // Center the block in next block canvas
            nextTetromino.y = 1;
            drawTetromino(nextTetromino, nextContext);
        }

        function hardDrop() {
            while (!isColliding(tetromino)) {
                tetromino.y++;
            }
            tetromino.y--;
            mergeTetromino(tetromino);
            clearRows();
            if (tetromino.y === 0) {
                gameOver = true;
                updateRankBoard();
                return;
            }
            tetromino
                    function hardDrop() {
            while (!isColliding(tetromino)) {
                tetromino.y++;
            }
            tetromino.y--;
            mergeTetromino(tetromino);
            clearRows();
            if (tetromino.y === 0) {
                gameOver = true;
                updateRankBoard();
                return;
            }
            tetromino = nextTetromino;
            nextTetromino = randomTetromino();
            drawNextBlock();
        }

        function updateRankBoard() {
            const playerScore = { name: playerName, score: score };
            let scores = JSON.parse(localStorage.getItem('scores')) || [];
            scores.push(playerScore);
            scores.sort((a, b) => b.score - a.score);
            scores = scores.slice(0, 5);
            localStorage.setItem('scores', JSON.stringify(scores));

            rankList.innerHTML = '';
            scores.forEach(player => {
                const li = document.createElement('li');
                li.textContent = `${player.name}: ${player.score}`;
                rankList.appendChild(li);
            });
        }

        function startGame() {
            playerName = document.getElementById('playerName').value;
            if (!playerName) {
                alert('Please enter your name to start the game.');
                return;
            }
            score = 0;
            scoreElement.textContent = 'Score: ' + score;
            gameOver = false;
            isPaused = false;
            tetromino = randomTetromino();
            nextTetromino = randomTetromino();
            drawNextBlock();
            update();
        }

        function moveLeft() {
            tetromino.x--;
            if (isColliding(tetromino)) {
                tetromino.x++;
            }
        }

        function moveRight() {
            tetromino.x++;
            if (isColliding(tetromino)) {
                tetromino.x--;
            }
        }

        function moveDown() {
            tetromino.y++;
            if (isColliding(tetromino)) {
                tetromino.y--;
            }
        }

        function rotate() {
            tetromino = rotate(tetromino);
        }

        document.addEventListener('keydown', event => {
            if (event.key === 'ArrowLeft') {
                moveLeft();
            } else if (event.key === 'ArrowRight') {
                moveRight();
            } else if (event.key === 'ArrowDown') {
                moveDown();
            } else if (event.key === 'ArrowUp') {
                rotate();
            } else if (event.key === ' ') {
                hardDrop();
            } else if (event.key === 'p') {
                isPaused = !isPaused;
                if (!isPaused) {
                    update();
                }
            }
        });

        function moveLeft() {
            tetromino.x--;
            if (isColliding(tetromino)) {
                tetromino.x++;
            }
        }

        function moveRight() {
            tetromino.x++;
            if (isColliding(tetromino)) {
                tetromino.x--;
            }
        }

        function moveDown() {
            tetromino.y++;
            if (isColliding(tetromino)) {
                tetromino.y--;
            }
        }

        function rotate() {
            tetromino = rotate(tetromino);
        }

        document.addEventListener('keydown', event => {
            if (event.key === 'ArrowLeft') {
                moveLeft();
            } else if (event.key === 'ArrowRight') {
                moveRight();
            } else if (event.key === 'ArrowDown') {
                moveDown();
            } else if (event.key === 'ArrowUp') {
                rotate();
            } else if (event.key === ' ') {
                hardDrop();
            } else if (event.key === 'p') {
                isPaused = !isPaused;
                if (!isPaused) {
                    update();
                }
            }
        });

        update();
        drawNextBlock();
    </script>
</body>
</html>
