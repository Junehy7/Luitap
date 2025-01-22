<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Chess-Style Tycoon Game</title>
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
      margin: 0 auto;
      padding: 20px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    }
    #game {
      display: grid;
      grid-template-columns: repeat(8, 60px);
      grid-template-rows: repeat(8, 60px);
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
    .tile.light {
      background-color: #f0d9b5;
    }
    .tile.dark {
      background-color: #b58863;
    }
    .tile img {
      width: 100%;
      height: 100%;
    }
    #menu {
      text-align: center;
      margin: 10px auto;
    }
    #menu select, #menu button {
      margin: 5px;
      padding: 5px 10px;
      font-size: 1rem;
    }
    #menu p {
      margin: 5px;
      font-size: 1.2rem;
      font-weight: bold;
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
    <div id="message">Welcome to Chess-Style Tycoon Game!</div>

    <!-- HTML structure -->
    <div id="menu">
      <select id="buildingType">
        <option value="factory">Factory ($20)</option>
        <option value="house">House ($15)</option>
      </select>
      <button onclick="build()">Build</button>
      <button onclick="endTurn()">End Turn</button>
      <p>Money: <span id="money">50</span></p>
      <p>Turn: <span id="turn">1</span></p>
    </div>

    <div id="game"></div>
  </div>

  <script>
    // JavaScript logic
    const grid = document.getElementById("game");
    const moneyDisplay = document.getElementById("money");
    const turnDisplay = document.getElementById("turn");
    const messageDisplay = document.getElementById("message");

    let money = 50;
    let turn = 1;
    const tiles = [];
    const turnEarnings = 10;

    // Function to show messages
    function showMessage(message) {
      messageDisplay.textContent = message;
    }

    // Initialize chessboard grid
    for (let i = 0; i < 64; i++) {
      const tile = document.createElement("div");
      tile.className = "tile " + ((Math.floor(i / 8) + i) % 2 === 0 ? "light" : "dark");
      tile.dataset.built = "false";
      tile.dataset.level = "0";
      tile.onclick = () => selectTile(i);
      grid.appendChild(tile);
      tiles.push(tile);
    }

    let selectedTile = null;

    function selectTile(index) {
      selectedTile = tiles[index];
      if (selectedTile.dataset.built === "true") {
        showMessage(`This tile is built! Level: ${selectedTile.dataset.level}`);
      } else {
        showMessage("This tile is empty. You can build here!");
      }
    }

    function build() {
      if (!selectedTile || selectedTile.dataset.built === "true") {
        showMessage("You cannot build on this tile!");
        return;
      }

      const buildingType = document.getElementById("buildingType").value;
      const cost = buildingType === "factory" ? 20 : 15;

      if (money >= cost) {
        money -= cost;
        selectedTile.dataset.built = "true";
        selectedTile.dataset.level = "1";
        selectedTile.dataset.type = buildingType;

        selectedTile.innerHTML = "";
        const img = document.createElement("img");
        img.src = buildingType === "factory" ? "factory.png" : "house.png";
        selectedTile.appendChild(img);

        // Apply a fine for building a factory
        if (buildingType === "factory") {
          money -= 5; // Fine for pollution
          showMessage("Factory built! Fine applied for pollution (-$5).");
        } else {
          showMessage("House built successfully!");
        }

        updateMoney();
      } else {
        showMessage("Not enough money!");
      }
    }

    function endTurn() {
      // Add earnings for all built tiles
      tiles.forEach(tile => {
        if (tile.dataset.built === "true") {
          const level = parseInt(tile.dataset.level);
          money += level * turnEarnings; // Earnings depend on tile level
        }
      });

      turn++;
      updateMoney();
      turnDisplay.textContent = turn;

      // Randomly upgrade buildings
      tiles.forEach(tile => {
        if (tile.dataset.built === "true" && Math.random() > 0.7) {
          const level = parseInt(tile.dataset.level);
          tile.dataset.level = level + 1;
          showMessage(`A building was upgraded to level ${level + 1}!`);
        }
      });
    }

    function updateMoney() {
      moneyDisplay.textContent = money;
    }

    // Load game on page load
    window.onload = () => {
      showMessage("Welcome to the Chess-Style Turn-Based Tycoon Game!");
    };
  </script>
</body>
</html>
