<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Custom Tycoon Game</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
      text-align: center;
      background-color: #ececec;
    }
    #container {
      max-width: 400px;
      margin: 0 auto;
      padding: 20px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    }
    #game {
      display: grid;
      grid-template-columns: repeat(9, 40px); /* 9 horizontal tiles */
      grid-template-rows: repeat(11, 40px); /* 11 vertical tiles */
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
      background-color: #a0522d; /* Brown for non-middle lanes */
    }
    .tile.skyblue {
      background-color: #87ceeb;
    }
    .tile.blue {
      background-color: #4682b4;
    }
    .tile.green {
      background-color: #98fb98;
    }
    .tile.darkgreen {
      background-color: #006400;
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
    <div id="message">Welcome to the Customized Tycoon Game!</div>

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
    const grid = document.getElementById("game");
    const moneyDisplay = document.getElementById("money");
    const turnDisplay = document.getElementById("turn");
    const messageDisplay = document.getElementById("message");

    let money = 50;
    let turn = 1;
    const tiles = [];
    const turnEarnings = 10;

    function showMessage(message) {
      messageDisplay.textContent = message;
    }

    // Initialize grid with zig-zag colors
    for (let i = 0; i < 99; i++) {
      const row = Math.floor(i / 9);
      const col = i % 9;

      const tile = document.createElement("div");
      tile.className = "tile";

      // Middle 3 horizontal lanes
      if (row >= 4 && row <= 6) {
        if ((row + col) % 2 === 0) {
          tile.className += row === 4 ? " skyblue" : row === 6 ? " blue" : " green"; // Zigzag with blues and greens
        } else {
          tile.className += row === 4 ? " blue" : row === 6 ? " darkgreen" : " darkgreen"; // Alternate colors
        }
      } else {
        tile.className += " brown"; // Non-middle rows
      }

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

        // Random pollution fine
        if (buildingType === "factory") {
          const pollutionFine = Math.floor(Math.random() * 10) + 5;
          money -= pollutionFine;
          showMessage(`Factory built! Random pollution fine applied (-$${pollutionFine}).`);
        } else {
          showMessage("House built successfully!");
        }

        updateMoney();
      } else {
        showMessage("Not enough money!");
      }
    }

    function endTurn() {
      tiles.forEach(tile => {
        if (tile.dataset.built === "true") {
          const level = parseInt(tile.dataset.level);
          money += level * turnEarnings;
        }
      });

      turn++;
      updateMoney();
      turnDisplay.textContent = turn;
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

    window.onload = () => {
      showMessage("Welcome to the Customized Turn-Based Tycoon Game!");
    };
  </script>
</body>
</html>
