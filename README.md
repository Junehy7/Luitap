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
            min-height: 100vh;
            background-color: #9ACD32;
            margin: 0;
            color: white;
            font-family: Arial, sans-serif;
        }
        #gameContainer {
            display: flex;
            flex-direction: column;
            align-items: center;
            max-width: 100%;
            padding: 10px;
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
            width: 140px;
        }
        #score, #next {
            font-size: 16px;
            margin-bottom: 10px;
        }
        #nextBlock {
            margin-top: 5px;
        }
        #playButton, #pauseButton {
            margin: 10px;
            padding: 10px 20px;
            font-size: 18px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        #playButton:hover, #pauseButton:hover {
            background-color: #45a049;
        }
        #mobileControls {
            display: none;
            margin-top: 20px;
            width: 100%;
            max-width: 300px;
        }
        .mobileButton {
            font-size: 24px;
            margin: 5px;
            padding: 15px;
            background-color: rgba(255, 255, 255, 0.7);
            border: none;
            border-radius: 50%;
            width: 60px;
            height: 60px;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: manipulation;
        }
        #controlsTop, #controlsBottom {
            display: flex;
            justify-content: space-between;
        }
        #controlsBottom {
            margin-top: 10px;
        }
        @media (max-width: 768px) {
            body {
                background-color: #9ACD32;
            }
            #gameArea {
                flex-direction: column;
                align-items: center;
            }
            #info {
                margin-left: 0;
                margin-top: 10px;
            }
            #mobileControls {
                display: flex;
                flex-direction: column;
            }
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <button id="playButton">Play</button>
        <div id="gameArea">
            <div>
                <canvas id="tetris" width="240" height="400"></canvas>
            </div>
            <div id="info">
                <div id="score">Score: 0</div>
                <div id="next">Next Block:</div>
                <canvas id="nextBlock" width="140" height="140"></canvas>
            </div>
        </div>
        <div id="mobileControls">
            <div id="controlsTop">
                <button class="mobileButton" id="dropButton">⤓</button>
                <button class="mobileButton" id="pauseButton">II</button>
                <button class="mobileButton" id="rotateButton">↻</button>
            </div>
            <div id="controlsBottom">
                <button class="mobileButton" id="leftButton">←</button>
                <button class="mobileButton" id="downButton">↓</button>
                <button class="mobileButton" id="rightButton">→</button>
            </div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('tetris');
        const context = canvas.getContext('2d');
        const nextCanvas = document.getElementById('nextBlock');
        const nextContext = nextCanvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const playButton = document.getElementById('playButton');
        playButton.addEventListener('click', startGame);
        const pauseButton = document.getElementById('pauseButton');
        const playerNameInput = document.getElementById('playerName');

        const grid = 20;
        let score = 0;
        let gameRunning = false;
        let gamePaused = false;

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

        function drawNextBlock() {
            nextContext.clearRect(0, 0, nextCanvas.width, nextCanvas.height);
            const blockWidth = nextTetromino.shape[0].length;
            const blockHeight = nextTetromino.shape.length;
            const offsetX = Math.floor((nextCanvas.width / grid - blockWidth) / 2) - 1;
            const offsetY = Math.floor((nextCanvas.height / grid - blockHeight) / 2);
            drawTetromino(nextTetromino, nextContext, offsetX, offsetY);
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
            const offsetX = (nextCanvas.width / grid - nextTetromino.shape[0].length) / 2;
            const offsetY = (nextCanvas.height / grid - nextTetromino.shape.length) / 2;
            drawTetromino(nextTetromino, nextContext, offsetX, offsetY);
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
            alert('Game Over! Your score: ' + score);
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
            pauseButton.textContent = 'Pause';
            update();
        }

        function togglePause() {
            if (!gameRunning) return;

            gamePaused = !gamePaused;
            if (gamePaused) {
                pauseButton.textContent = 'Continue';
            } else {
                pauseButton.textContent = 'Pause';
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
        pauseButton.addEventListener('click', togglePause);

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
