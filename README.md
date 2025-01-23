<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Symmetrical Chess Board</title>
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
      background-color: #808080; /* Dark grey (switched position) */
    }
    .tile.darkgrey {
      background-color: #d3d3d3; /* Light grey (switched position) */
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
    <div id="message">Welcome to the Symmetrical Chess-Style Board!</div>

    <!-- Game Board -->
    <div id="game"></div>
  </div>

  <script>
    const grid = document.getElementById("game");

    // Function to assign tile colors
    function getTileColor(row, col) {
      // Top-left corner
      if (row < 3 && col < 3) {
        const index = row * 3 + col;
        return ["blue", "skyblue", "blue", "skyblue", "darkgreen", "green", "blue", "green", "darkgreen"][index];
      }

      // Top-right corner (mirror horizontally)
      if (row < 3 && col > 7) {
        const index = row * 3 + (10 - col);
        return ["blue", "skyblue", "blue", "skyblue", "darkgreen", "green", "blue", "green", "darkgreen"][index];
      }

      // Bottom-left corner (mirror vertically)
      if (row > 7 && col < 3) {
        const index = (10 - row) * 3 + col;
        return ["blue", "skyblue", "blue", "skyblue", "darkgreen", "green", "blue", "green", "darkgreen"][index];
      }

      // Bottom-right corner (mirror diagonally)
      if (row > 7 && col > 7) {
        const index = (10 - row) * 3 + (10 - col);
        return ["blue", "skyblue", "blue", "skyblue", "darkgreen", "green", "blue", "green", "darkgreen"][index];
      }

      // Center 3x3 grid (switched light grey and dark grey)
      if (row >= 4 && row <= 6 && col >= 4 && col <= 6) {
        return (row + col) % 2 === 0 ? "darkgrey" : "lightgrey";
      }

      // Remaining board (classic chessboard style)
      return (row + col) % 2 === 0 ? "brown" : "lightbrown";
    }

    // Initialize grid with the specified layout
    for (let i = 0; i < 121; i++) {
      const row = Math.floor(i / 11);
      const col = i % 11;

      const tile = document.createElement("div");
      tile.className = `tile ${getTileColor(row, col)}`;
      grid.appendChild(tile);
    }
  </script>
</body>
</html>
