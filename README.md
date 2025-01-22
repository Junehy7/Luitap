<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
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
    position: relative;
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

#pauseButton {
    flex-grow: 1;
    width: auto;
    padding: 10px 30px;
    font-size: 18px;
}

#statusMessage {
    margin-top: 20px;
    background-color: rgba(0, 0, 0, 0.8);
    color: white;
    padding: 10px;
    border-radius: 5px;
    display: none;
    text-align: center;
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
    <div id="statusMessage"></div> <!-- Status message area -->
    <button id="playButton">Play</button>
    <div id="gameArea">
        <div>
            <canvas id="tetris" width="240" height="400"></canvas>
        </div>
        <div id="info">
            <div id="score">Score: 0</div>
            <div id="next">Next Block:</div>
            <canvas id="nextBlock" width="140" height="70"></canvas>
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
    const pauseButton = document.getElementById('pauseButton');
    const statusMessage = document.getElementById('statusMessage'); // New message area

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

    function displayMessage(message) {
        statusMessage.textContent = message;
        statusMessage.style.display = 'block';
        setTimeout(() => {
            statusMessage.style.display = 'none';
        }, 3000);
    }

    function isColliding(tetromino) {
        return tetromino.shape.some((row, dy) => {
            return row.some((value, dx) => {
                const newX = tetromino.x + dx;
                const newY = tetromino.y + dy;
                return value && (newX < 0 || newX >= 12 || newY >= 20 || board[newY] && board[newY][newX]);
            });
        });
    }

    function gameOver() {
        gameRunning = false;
        displayMessage('Game Over! Your score: ' + score);
    }

    function togglePause() {
        if (!gameRunning) return;
        gamePaused = !gamePaused;
        if (gamePaused) {
            displayMessage('Game Paused');
            pauseButton.textContent = 'Continue';
        } else {
            displayMessage('Game Resumed');
            pauseButton.textContent = 'Pause';
            update();
        }
    }

    playButton.addEventListener('click', () => {
        if (!gameRunning) {
            score = 0;
            gameRunning = true;
            gamePaused = false;
            pauseButton.textContent = 'Pause';
            update();
        }
    });

    pauseButton.addEventListener('click', togglePause);

    // (Other functions remain unchanged)
</script>
</body>
</html>
