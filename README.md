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
        #pauseButtonMobile {
            width: 120px;
            height: 60px;
            font-size: 18px;
            border-radius: 10px;
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
            #controlsTop {
                justify-content: space-around;
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
                <button class="mobileButton" id="rotateButton">↻</button>
                <button id="pauseButtonMobile">Pause</button>
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
        const pauseButton = document.getElementById('pauseButton');
        const pauseButtonMobile = document .getElementById('pauseButtonMobile');
        const dropButton = document.getElementById('dropButton');
        const rotateButton = document.getElementById('rotateButton');
        const leftButton = document.getElementById('leftButton');
        const downButton = document.getElementById('downButton');
        const rightButton = document.getElementById('rightButton');

        let board = [];
        let score = 0;
        let gameRunning = false;
        let gamePaused = false;
        let tetromino = null;
        let nextTetromino = null;

        function createBoard() {
            for (let i = 0; i < 20; i++) {
                board[i] = [];
                for (let j = 0; j < 10; j++) {
                    board[i][j] = 0;
                }
            }
        }

        function randomTetromino() {
            const tetrominos = [
                [[1, 1], [1, 1]],
                [[1, 1, 1, 1]],
                [[1, 0, 0], [1, 1, 1]],
                [[0, 0, 1], [1, 1, 1]],
                [[1, 1], [1, 0], [1, 0]],
                [[0, 1], [0, 1], [1, 1]],
                [[0, 1, 1], [1, 1, 0]]
            ];
            return tetrominos[Math.floor(Math.random() * tetrominos.length)];
        }

        function drawNextBlock() {
            nextContext.clearRect(0, 0, nextCanvas.width, nextCanvas.height);
            for (let i = 0; i < nextTetromino.length; i++) {
                for (let j = 0; j < nextTetromino[i].length; j++) {
                    if (nextTetromino[i][j] === 1) {
                        nextContext.fillStyle = 'blue';
                        nextContext.fillRect(j * 20, i * 20, 20, 20);
                    }
                }
            }
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
            pauseButtonMobile.textContent = 'Pause';
            update();
        }

        function update() {
            // Game logic here
        }

        function togglePause() {
            if (gameRunning) {
                gamePaused = !gamePaused;
                if (gamePaused) {
                    pauseButton.textContent = 'Resume';
                    pauseButtonMobile.textContent = 'Resume';
                } else {
                    pauseButton.textContent = 'Pause';
                    pauseButtonMobile.textContent = 'Pause';
                }
            }
        }

        playButton.addEventListener('click', startGame);
        pauseButton.addEventListener('click', togglePause);
        pauseButtonMobile.addEventListener('click', togglePause);
    </script>
</body>
</html>
