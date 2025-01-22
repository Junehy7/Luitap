<!DOCTYPE html>
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
    }
    #game {
      display: grid;
      grid-template-columns: repeat(10, 50px);
      grid-template-rows: repeat(10, 50px);
      gap: 2px;
      margin: 20px auto;
      width: 520px;
    }
    .tile {
      border: 1px solid #ccc;
      background-color: #f9f9f9;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      position: relative;
    }
    .tile img {
      width: 100%;
      height: 100%;
    }
    #menu {
      margin: 20px auto;
    }
    #message {
      margin: 20px;
      padding: 10px;
      border: 1px solid #ccc;
      background-color: #f3f3f3;
      font-size: 1rem;
      color: #333;
    }
  </style>
</head>
<body>
  <!-- Message Area -->
  <div id="message">Welcome to Tycoon Game!</div>

  <!-- HTML structure -->
  <div id="menu">
    <select id="buildingType">
      <option value="factory">Factory ($20)</option>
      <option value="house">House ($15)</option>
    </select>
    <button onclick="build()">Build</button>
    <button onclick="saveGame()">Save Game</button>
    <button onclick="loadGame()">Load Game</button>
    <p>Money: <span id="money">50</span></p>
  </div>
  <div id="game"></div>

  <script>
    // JavaScript logic
    const grid = document.getElementById("game");
    const moneyDisplay = document.getElementById("money");
    const messageDisplay = document.getElementById("message");

    let money = 50;
    const tiles = [];
    const earnings = 5;

    // Function to show messages
    function showMessage(message) {
      messageDisplay.textContent = message;
    }

    // Initialize grid
    for (let i = 0; i < 100; i++) {
      const tile = document.createElement("div");
      tile.className = "tile";
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

        showMessage("Building constructed successfully!");
        updateMoney();
      } else {
        showMessage("Not enough money!");
      }
    }

    function saveGame() {
      const gameState = {
        money,
        tiles: tiles.map(tile => ({
          built: tile.dataset.built,
          level: tile.dataset.level,
          type: tile.dataset.type,
        })),
      };
      localStorage.setItem("tycoonGame", JSON.stringify(gameState));
      showMessage("Game saved successfully!");
    }

    function loadGame() {
      const savedState = localStorage.getItem("tycoonGame");
      if (!savedState) {
        showMessage("No saved game found!");
        return;
      }
      const { money: savedMoney, tiles: savedTiles } = JSON.parse(savedState);
      money = savedMoney;
      updateMoney();
      savedTiles.forEach((tileData, index) => {
        const tile = tiles[index];
        tile.dataset.built = tileData.built;
        tile.dataset.level = tileData.level;
        tile.dataset.type = tileData.type;
        tile.innerHTML = "";
        if (tileData.built === "true") {
          const img = document.createElement("img");
          img.src = tileData.type === "factory" ? "factory.png" : "house.png";
          tile.appendChild(img);
        }
      });
      showMessage("Game loaded successfully!");
    }

    function updateMoney() {
      moneyDisplay.textContent = money;
    }

    // Earnings loop
    setInterval(() => {
      tiles.forEach(tile => {
        if (tile.dataset.built === "true") {
          money += earnings * parseInt(tile.dataset.level);
        }
      });
      updateMoney();
    }, 3000);

    // Random events
    function triggerRandomEvent() {
      const eventType = Math.random() > 0.5 ? "good" : "bad";
      if (eventType === "good") {
        const bonus = Math.floor(Math.random() * 20) + 10;
        money += bonus;
        showMessage(`Good Event! You received a bonus of $${bonus}.`);
      } else {
        const fine = Math.floor(Math.random() * 15) + 5;
        money -= fine;
        if (money < 0) money = 0;
        showMessage(`Bad Event! You were fined $${fine}.`);
      }
      updateMoney();
    }

    setInterval(triggerRandomEvent, 20000);

    // Load game on page load
    window.onload = loadGame;
  </script>
</body>
</html>
