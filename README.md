<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tycoon Game</title>
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
    #menu {
      margin-bottom: 20px;
    }
    #game {
      display: grid;
      grid-template-columns: repeat(11, 40px);
      grid-template-rows: repeat(11, 40px);
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
    <div id="menu">
      <select id="buildingType">
        <option value="factory">Factory ($20)</option>
        <option value="house">House ($15)</option>
      </select>
      <button onclick="build()">Build</button>
      <p>Money: <span id="money">50</span></p>
    </div>

    <!-- Game Board -->
    <div id="game"></div>

    <div id="message">Click on a tile to start building!</div>
  </div>

  <script>
    const grid = document.getElementById("game");
    const moneyDisplay = document.getElementById("money");
    const message = document.getElementById("message");

    let money = 50;
    const tiles = [];
    let selectedTile = null;

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

      // Center 3x3 grid (switched dark grey and light grey)
      if (row >= 4 && row <= 6 && col >= 4 && col <= 6) {
        return (row + col) % 2 === 0 ? "darkgrey" : "lightgrey"; // Switched!
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
      tile.dataset.built = "false";
      tile.dataset.level = "0";
      tile.onclick = () => selectTile(i);
      grid.appendChild(tile);
      tiles.push(tile);
    }

    function selectTile(index) {
      selectedTile = tiles[index];
      if (selectedTile.dataset.built === "true") {
        message.textContent = `This tile is already built! Level: ${selectedTile.dataset.level}`;
      } else {
        message.textContent = "This tile is empty. You can build here!";
      }
    }

    function build() {
      if (!selectedTile || selectedTile.dataset.built === "true") return;

      const buildingType = document.getElementById("buildingType").value;
      const cost = buildingType === "factory" ? 20 : 15;

      if (money >= cost) {
        money -= cost;
        selectedTile.dataset.built = "true";
        selectedTile.dataset.level = "1";
        selectedTile.dataset.type = buildingType;

        // Add an image to the tile
        selectedTile.innerHTML = "";
        const img = document.createElement("img");
        img.src = buildingType === "factory" ? "factory.png" : "house.png";
        img.style.width = "100%";
        img.style.height = "100%";
        selectedTile.appendChild(img);

        updateMoney();
        message.textContent = `Built a ${buildingType}!`;
      } else {
        message.textContent = "Not enough money!";
      }
    }

    function updateMoney() {
      moneyDisplay.textContent = money;
    }
  </script>
</body>
</html>
