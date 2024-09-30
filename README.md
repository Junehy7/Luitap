<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tetris Game</title>
    <style>
        /* Styles omitted for brevity */
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
            <!-- Mobile controls omitted for brevity -->
        </div>
    </div>

    <script>
        // Script code omitted for brevity, insert the original JavaScript here
        const canvas = document.getElementById('tetris');
        const context = canvas.getContext('2d');
        const nextCanvas = document.getElementById('nextBlock');
        const nextContext = nextCanvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const playButton = document.getElementById('playButton');
        const pauseButton = document.getElementById('pauseButton');

        // Other game variables omitted for brevity...

        function startGame() {
            console.log("Starting Game...");  // Debug log
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

        playButton.addEventListener('click', startGame);
        pauseButton.addEventListener('click', togglePause);

        // Draw and update functions omitted for brevity...
    </script>
</body>
</html>
