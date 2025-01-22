<!DOCTYPE html>
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
      max-width: 400px;
      margin: 0 auto;
      padding: 20px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    }
    #game {
      display: grid;
      grid-template-columns: repeat(9, 40px); /* 9 columns */
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
    <div id="message">Welcome to the Chess-Style Tycoon Game!</div>

    <!-- Menu -->
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

    <!-- Game Board -->
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

    // Initialize grid with updated 2-5-2 layout
    for (let i = 0; i < 99; i++) {
      const row = Math.floor(i / 9);
      const col = i % 9;

      const tile = document.createElement("div");
      tile.className = "tile";

      // Leftmost, middle, and rightmost lanes
      if (col < 2) {
        // Leftmost 2 lanes: zigzag with blues
        tile.className += (row + col) % 2 === 0 ? " skyblue" : " blue";
      } else if (col >= 2 && col < 7) {
        // Middle 5 lanes: zigzag with greens
        tile.className += (row + col) % 2 === 0 ? " green" : " darkgreen";
      } else {
        // Rightmost 2 lanes: zigzag with blues
        tile.className += (row + col) % 2 === 0 ? " skyblue" : " blue";
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

        showMessage(`${buildingType.charAt(0).toUpperCase() + buildingType.slice(1)} built successfully!`);
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
    }

    function updateMoney() {
      moneyDisplay.textContent = money;
    }

    window.onload = () => {
      showMessage("Welcome to the Chess-Style Tycoon Game!");
    };
  </script>
</body>
</html>
