<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Fixed Chess-Style Board</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      text-align: center;
      background-color: #ececec;
    }
    #container {
      max-width: 500px;
      margin: 20px auto;
      padding: 10px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    }
    #game {
      display: grid;
      grid-template-columns: repeat(11, 40px); /* 11 columns */
      grid-template-rows: repeat(11, 40px); /* 11 rows */
      gap: 0;
      margin: 20px auto;
      border: 2px solid #333;
    }
    .tile {
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      position: relative;
    }
    .tile.brown {
      background-color: #a0522d; /* Dark brown */
    }
    .tile.lightbrown {
      background-color: #d2b48c; /* Light brown */
    }
    .tile.skyblue {
      background-color: #87ceeb; /* Skyblue */
    }
    .tile.blue {
      background-color: #4682b4; /* Blue */
    }
    .tile.green {
      background-color: #98fb98; /* Light green */
    }
    .tile.darkgreen {
      background-color: #006400; /* Dark green */
    }
    .tile.lightgrey {
      background-color: #d3d3d3; /* Light grey */
    }
    .tile.darkgrey {
      background-color: #808080; /* Dark grey */
    }
    #message {
      margin: 10px auto;
      padding: 10px;
      border: 1px solid #ccc;
      background-color: #f3f3f3;
      font-size: 1rem;
      color: #333;
      border-radius: 4px;
    }
  </style>
</head>
<body>
  <div id="container">
    <!-- Message Area -->
    <div id="message">Welcome to the Custom Chess-Style Board!</div>

    <!-- Game Board -->
    <div id="game"></div>
  </div>

  <script>
    const grid = document.getElementById("game");

    // Initialize grid with the specified layout
    for (let i = 0; i < 121; i++) {
      const row = Math.floor(i / 11); // Current row
      const col = i % 11; // Current column

      const tile = document.createElement("div");
      tile.className = "tile";

      // Four corners (top-left, top-right, bottom-left, bottom-right)
      if ((row < 3 && col < 3) || (row < 3 && col > 7) || (row > 7 && col < 3) || (row > 7 && col > 7)) {
        // Top-left corner pattern
        if (row < 3 && col < 3) {
          if ((row === 0 && col === 0) || (row === 0 && col === 2) || (row === 2 && col === 0)) tile.className += " blue";
          else if ((row === 0 && col === 1) || (row === 1 && col === 0)) tile.className += " skyblue";
          else if ((row === 1 && col === 1) || (row === 2 && col === 2)) tile.className += " darkgreen";
          else if ((row === 1 && col === 2) || (row === 2 && col === 1)) tile.className += " green";
        }
        // Symmetrical corners
        else if ((row < 3 && col > 7) || (row > 7 && col < 3) || (row > 7 && col > 7)) {
          const symmetryOffset = (col > 7 ? -8 : 0) + (row > 7 ? -88 : 0);
          const symIndex = i + symmetryOffset;
          if ((symIndex === 0) || (symIndex === 2) || (symIndex === 6)) tile.className += " blue";
          else if (symIndex === 1 || symIndex === 3) tile.className += " skyblue";
          else if (symIndex === 4 || symIndex === 8) tile.className += " darkgreen";
          else if (symIndex === 5 || symIndex === 7) tile.className += " green";
        }
      }
      // Center 9 blocks (3x3 grid)
      else if (row >= 4 && row <= 6 && col >= 4 && col <= 6) {
        tile.className += (row + col) % 2 === 0 ? " lightgrey" : " darkgrey";
      }
      // Rest of the board: classic chessboard style
      else {
        tile.className += (row + col) % 2 === 0 ? " brown" : " lightbrown";
      }

      grid.appendChild(tile);
    }
  </script>
</body>
</html>
