
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <title>Game Mobil Modern</title>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet" />
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Poppins', sans-serif; }
    body { background: #f4f6f8; color: #333; text-align: center; padding: 20px; }
    h1 { font-size: 2rem; margin-bottom: 10px; }

    #gameArea {
      width: 100%;
      max-width: 400px;
      height: 600px;
      margin: 20px auto;
      background: url('https://cdn.pixabay.com/photo/2020/06/25/08/15/asphalt-5338876_960_720.jpg') repeat-y;
      background-size: cover;
      position: relative;
      overflow: hidden;
      border-radius: 20px;
      box-shadow: 0 10px 20px rgba(0,0,0,0.15);
      animation: roadMove 4s linear infinite;
    }

    @keyframes roadMove {
      0% { background-position-y: 0; }
      100% { background-position-y: 100%; }
    }

    .car {
      width: 50px;
      height: 100px;
      background: no-repeat center/contain;
      position: absolute;
      bottom: 20px;
      left: 175px;
      transition: left 0.1s ease;
    }

    .obstacle {
      width: 50px;
      height: 100px;
      background: url('https://cdn-icons-png.flaticon.com/512/296/296216.png') no-repeat center/contain;
      position: absolute;
      top: 0;
    }

    #hud {
      display: flex;
      justify-content: space-between;
      max-width: 400px;
      margin: 0 auto;
      font-size: 1rem;
      margin-bottom: 10px;
    }

    #controls {
      margin: 15px 0;
    }

    #controls button, #restartBtn, #themeBtn, #changeCarBtn {
      font-size: 16px;
      padding: 10px 18px;
      margin: 6px;
      border: none;
      border-radius: 12px;
      background-color: #4CAF50;
      color: white;
      cursor: pointer;
      transition: background 0.2s ease;
    }

    #controls button:hover, #restartBtn:hover, #themeBtn:hover, #changeCarBtn:hover {
      background-color: #388E3C;
    }

    #gameOver {
      font-size: 2rem;
      margin-top: 20px;
      color: crimson;
      display: none;
      font-weight: bold;
    }

    @media (max-width: 500px) {
      #gameArea {
        height: 500px;
      }
      .car, .obstacle {
        width: 12vw;
        height: 20vw;
      }
    }
  </style>
</head>
<body>

<h1>🚗 Game Mobil Modern</h1>

<div id="hud">
  <div>Skor: <span id="score">0</span></div>
  <div>Level: <span id="level">1</span></div>
</div>

<div id="gameArea">
  <div id="car" class="car"></div>
</div>

<div id="controls">
  <button id="leftBtn">⬅️</button>
  <button id="rightBtn">➡️</button>
</div>

<div>
  <button id="changeCarBtn">Ganti Mobil 🚘</button>
  <button id="restartBtn">Main Lagi 🔁</button>
  <button id="themeBtn">Ganti Tema 🌙</button>
</div>

<div id="gameOver">GAME OVER</div>

<audio id="crashSound" src="https://www.soundjay.com/transportation/sounds/car-crash-1.mp3" preload="auto"></audio>

<script>
  const car = document.getElementById("car");
  const gameArea = document.getElementById("gameArea");
  const scoreDisplay = document.getElementById("score");
  const levelDisplay = document.getElementById("level");
  const gameOverText = document.getElementById("gameOver");
  const restartBtn = document.getElementById("restartBtn");
  const crashSound = document.getElementById("crashSound");
  const themeBtn = document.getElementById("themeBtn");
  const changeCarBtn = document.getElementById("changeCarBtn");

  const carSkins = [
    'https://cdn-icons-png.flaticon.com/512/744/744465.png',
    'https://cdn-icons-png.flaticon.com/512/743/743922.png',
    'https://cdn-icons-png.flaticon.com/512/679/679720.png'
  ];
  let currentCarSkin = 0;

  let carLeft = 175;
  let score = 0;
  let level = 1;
  let speed = 3;
  let gameRunning = true;
  let obstacleInterval;

  function updateCarSkin() {
    car.style.backgroundImage = `url('${carSkins[currentCarSkin]}')`;
  }

  changeCarBtn.onclick = () => {
    currentCarSkin = (currentCarSkin + 1) % carSkins.length;
    updateCarSkin();
  };

  document.addEventListener("keydown", e => {
    if (!gameRunning) return;
    if (e.key === "ArrowLeft" && carLeft > 0) carLeft -= 25;
    if (e.key === "ArrowRight" && carLeft < 350) carLeft += 25;
    car.style.left = carLeft + "px";
  });

  document.getElementById("leftBtn").onclick = () => {
    if (carLeft > 0) carLeft -= 25;
    car.style.left = carLeft + "px";
  };

  document.getElementById("rightBtn").onclick = () => {
    if (carLeft < 350) carLeft += 25;
    car.style.left = carLeft + "px";
  };

  function createObstacle() {
    if (!gameRunning) return;
    const obs = document.createElement("div");
    obs.classList.add("obstacle");
    obs.style.left = Math.floor(Math.random() * 8) * 50 + "px";
    gameArea.appendChild(obs);

    let top = 0;
    const fall = setInterval(() => {
      if (!gameRunning) {
        clearInterval(fall);
        obs.remove();
        return;
      }

      top += speed;
      obs.style.top = top + "px";

      if (
        top + 100 >= 500 &&
        parseInt(obs.style.left) === carLeft
      ) {
        crashSound.play();
        gameOver();
        clearInterval(fall);
      }

      if (top > 600) {
        obs.remove();
        score += 10;
        scoreDisplay.textContent = score;

        if (score % 50 === 0) {
          level++;
          speed++;
          levelDisplay.textContent = level;
        }

        clearInterval(fall);
      }
    }, 20);

    obstacleInterval = setTimeout(createObstacle, 1000);
  }

  function gameOver() {
    gameRunning = false;
    gameOverText.style.display = "block";
    restartBtn.style.display = "inline-block";
    clearTimeout(obstacleInterval);
  }

  function startGame() {
    carLeft = 175;
    car.style.left = carLeft + "px";
    score = 0;
    level = 1;
    speed = 3;
    gameRunning = true;
    scoreDisplay.textContent = score;
    levelDisplay.textContent = level;
    gameOverText.style.display = "none";
    restartBtn.style.display = "none";
    document.querySelectorAll(".obstacle").forEach(el => el.remove());
    createObstacle();
    updateCarSkin();
  }

  restartBtn.onclick = startGame;

  let dark = false;
  themeBtn.onclick = () => {
    dark = !dark;
    document.body.style.background = dark ? "#121212" : "#f4f6f8";
    document.body.style.color = dark ? "#fff" : "#333";
    themeBtn.textContent = dark ? "Ganti Tema ☀️" : "Ganti Tema 🌙";
  };

  startGame();
</script>

</body>
</html>
