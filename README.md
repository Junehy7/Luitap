Cute Tetris
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
        <button id="playButton">Play</button>
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
            margin-bottom: 10px;
            padding: 5px 10px;
            font-size: 16px;
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
        // ... (keep the existing JavaScript code)

        // Update the drawNextBlock function to center the blocks
        function drawNextBlock() {
            nextContext.clearRect(0, 0, nextCanvas.width, nextCanvas.height);
            const blockWidth = nextTetromino.shape[0].length;
            const blockHeight = nextTetromino.shape.length;
            const offsetX = Math.floor((nextCanvas.width / grid - blockWidth) / 2) - 1;
            const offsetY = Math.floor((nextCanvas.height / grid - blockHeight) / 2);
            drawTetromino(nextTetromino, nextContext, offsetX, offsetY);
        }

        // ... (keep the rest of the JavaScript code)
    </script>
</body>
</html>
