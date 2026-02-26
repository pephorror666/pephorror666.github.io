<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Retro Surf Game</title>
  <link href="https://fonts.googleapis.com/css?family=Press+Start+2P" rel="stylesheet">
  <link href="https://unpkg.com/nes.css@latest/css/nes.min.css" rel="stylesheet" />
  <style>
    body {
      background-color: #87CEEB;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      font-family: 'Press Start 2P', cursive;
    }

    #game-container {
      width: 800px;
      height: 400px;
      position: relative;
      overflow: hidden;
      background: linear-gradient(to bottom, #4169E1 0%, #4169E1 70%, #00BFFF 80%, #00BFFF 100%);
    }

    #player {
      position: absolute;
      bottom: 20px;
      left: 50px;
      font-size: 40px;
      z-index: 2;
    }

    .enemy {
      position: absolute;
      bottom: 20px;
      font-size: 30px;
      z-index: 1;
    }

    #score {
      position: absolute;
      top: 10px;
      right: 10px;
      color: white;
    }

    #message {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      text-align: center;
      color: white;
      font-size: 24px;
    }

    #sun {
      position: absolute;
      font-size: 40px;
    }

    #waves {
      position: absolute;
      bottom: 0;
      left: 0;
      width: 200%;
      height: 40%;
      background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1440 320"><path fill="%2300BFFF" fill-opacity="1" d="M0,128L48,138.7C96,149,192,171,288,181.3C384,192,480,192,576,181.3C672,171,768,149,864,149.3C960,149,1056,171,1152,176C1248,181,1344,171,1392,165.3L1440,160L1440,320L1392,320C1344,320,1248,320,1152,320C1056,320,960,320,864,320C768,320,672,320,576,320C480,320,384,320,288,320C192,320,96,320,48,320L0,320Z"></path></svg>');
      background-repeat: repeat-x;
      animation: wave 10s linear infinite;
    }

    @keyframes wave {
      0% {
        transform: translateX(0);
      }

      100% {
        transform: translateX(-50%);
      }
    }

    .poo {
      position: absolute;
      font-size: 20px;
      z-index: 1;
    }

    .coin {
      position: absolute;
      font-size: 30px;
      z-index: 1;
      animation: blink 0.5s infinite;
    }

    @keyframes blink {
      0%,
      100% {
        opacity: 1;
      }

      50% {
        opacity: 0;
      }
    }
  </style>
</head>

<body>
  <div id="game-container" class="nes-container is-rounded">
    <div id="waves"></div>
    <div id="player">🏄</div>
    <div id="score">SCORE: 0</div>
    <div id="message">
      <span style="color: #AB47BC; margin-right: 5px;">S</span>
      <span style="color: #FFEB3B; margin-right: 5px;">U</span>
      <span style="color: #FFA726; margin-right: 5px;">R</span>
      <span style="color: #D32F2F; margin-right: 5px;">F</span>
      <span style="color: #BA68C8; margin-right: 5px;">E</span>
      <span style="color: #E57373; margin-right: 5px;">R</span>
      <br>
      <span style="color: #ffffff; margin-right: 5px;">press any key</span>
    </div>
    <div id="sun">🌙</div>
  </div>

  <!-- Audio elements for sounds -->
  <audio id="killSound" src="https://cdn.freesound.org/previews/735/735804_15950203-lq.mp3"></audio>
  <audio id="coinSound" src="https://cdn.freesound.org/previews/350/350875_5450487-hq.mp3"></audio>
  <audio id="gameOverSound" src="https://cdn.freesound.org/previews/475/475347_7724198-lq.mp3"></audio>
  <audio id="jumpSound" src="https://assets.mixkit.co/sfx/preview/mixkit-retro-game-emergency-alarm-1000.mp3"></audio>
  <audio id="gameMusic" src="https://midis101.com/play-midi/23413-beach-boys-surfin-usa.mp3"></audio>

  <script>
    const player = document.getElementById('player');
    const gameContainer = document.getElementById('game-container');
    const scoreElement = document.getElementById('score');
    const messageElement = document.getElementById('message');
    const sunElement = document.getElementById('sun');
    const killSound = document.getElementById('killSound');
    const coinSound = document.getElementById('coinSound');
    const gameOverSound = document.getElementById('gameOverSound');
    const jumpSound = document.getElementById('jumpSound');
    const gameMusic = document.getElementById('gameMusic');
    let playerX = 50;
    let playerY = gameContainer.offsetHeight - 60;
    let playerVelocityY = 0;
    let playerVelocityX = 0;
    let isJumping = false;
    let score = 0;
    let highScore = localStorage.getItem('highScore') || 0;
    let gravity = 0.5;
    let jumpStrength = 10.8;
    let gameIsOver = false;
    let gameStarted = false;
    let sunX = gameContainer.offsetWidth;
    let enemyInterval;
    let coinInterval;

    function updatePlayerPosition() {
      playerX += playerVelocityX;
      playerY -= playerVelocityY;
      playerVelocityY -= gravity;

      if (playerY > gameContainer.offsetHeight - 60) {
        playerY = gameContainer.offsetHeight - 60;
        playerVelocityY = 0;
        isJumping = false;
      }

      if (playerX < 0) playerX = 0;
      if (playerX > gameContainer.offsetWidth - 50) playerX = gameContainer.offsetWidth - 50;

      player.style.left = playerX + 'px';
      player.style.bottom = (gameContainer.offsetHeight - playerY - 40) + 'px';
    }

    function updateSunPosition() {
      sunX -= 0.5;
      if (sunX < -40) {
        sunX = gameContainer.offsetWidth;
      }
      sunElement.style.left = sunX + 'px';
      sunElement.style.top = (gameContainer.offsetHeight * 0.2) + 'px';
    }

    function jump() {
      if (!isJumping) {
        isJumping = true;
        playerVelocityY = jumpStrength;
        jumpSound.play(); // Play jump sound
      }
    }

    function createEnemy() {
      let type;
      if (score >= 120 && Math.random() < 0.05) {
        type = 'eagle'; //New enemy
      } else if (score >= 80 && Math.random() < 0.05) {
        type = 'dolphin';
      } else if (score >= 40 && Math.random() < 0.1) {
        type = 'shark';
      } else if (score >= 20 && Math.random() < 0.3) {
        type = 'crab';
      } else {
        type = 'lobster';
      }

      const enemy = document.createElement('div');
      enemy.className = 'enemy';
      enemy.textContent = type === 'shark' ? '🦈' : type === 'crab' ? '🦀' : type === 'dolphin' ? '🐬' : type === 'lobster' ? '🦞' : type === 'eagle' ? '🦅' : '';
      const startFromLeft = Math.random() < 0.5;
      enemy.style.left = startFromLeft ? '-30px' : gameContainer.offsetWidth + 'px';

      // Eagle specific properties
      let eagleY = 50 + Math.random() * (gameContainer.offsetHeight * 0.3); //random height
      if (type === 'eagle') {
        enemy.style.top = eagleY + 'px';
      } else {
        enemy.style.bottom = '20px'; // other enemies stay on the bottom
      }

      gameContainer.appendChild(enemy);

      let enemyX = startFromLeft ? -30 : gameContainer.offsetWidth;
      let direction = startFromLeft ? 1 : -1;
      let speed = type === 'shark' ? 4 : type === 'dolphin' ? 3 : type === 'crab' ? 3.5 : type === 'eagle' ? 2.5 : 2;
      let isJumping = false;
      let jumpTimer = 0;
      let dolphinVelocityY = 0; // Vertical velocity for dolphin jump
      const dolphinJumpStrength = 8; // Initial jump strength
      const dolphinGravity = 0.4; // Gravity for dolphin jump
      let dolphinY = 20; // Initial bottom position

      const moveInterval = setInterval(() => {
        if (gameIsOver) {
          clearInterval(moveInterval);
          return;
        }

        if (type === 'crab') {
          // Define a turning point for the crab, before it reaches the edge
          const turnMargin = 50; // Adjust this value to control how far from the edge the crab turns

          if (direction === 1 && enemyX >= (gameContainer.offsetWidth - turnMargin)) {
            // If crab is moving right and close to the right edge, turn around
            direction = -1;
          } else if (direction === -1 && enemyX <= turnMargin) {
            // If crab is moving left and close to the left edge, turn around
            direction = 1;
          }
        }

        if (type === 'dolphin') {
          jumpTimer++;

          // Dolphin jump logic
          if (jumpTimer >= 100 && !isJumping) {
            isJumping = true;
            dolphinVelocityY = dolphinJumpStrength;
          }

          if (isJumping) {
            dolphinY += dolphinVelocityY;
            dolphinVelocityY -= dolphinGravity; // Apply gravity

            if (dolphinY <= 20) {
              dolphinY = 20;
              dolphinVelocityY = 0;
              isJumping = false;
              jumpTimer = 0;
            }

            enemy.style.bottom = dolphinY + 'px';
          }
        }

        if (type === 'eagle') {
          //Poo dropping logic
          if (Math.random() < 0.015) { //adjust the value
            dropPoo(enemyX, eagleY);
          }
        }

        enemyX += direction * speed;
        enemy.style.left = enemyX + 'px';

        // Añade esta sección para invertir el emoji
        if (direction === 1) {
          enemy.style.transform = 'scaleX(-1)'; // Normal cuando va hacia la derecha
        } else {
          enemy.style.transform = 'scaleX(1)'; // Invertido cuando va hacia la izquierda
        }

        if ((startFromLeft && enemyX > gameContainer.offsetWidth) || (!startFromLeft && enemyX < -30)) {
          clearInterval(moveInterval);
          enemy.remove();
        }

        checkCollision(enemy, enemyX, moveInterval, type, isJumping);
      }, 20);
    }

    function checkCollision(enemy, enemyX, moveInterval, type, isJumping) {
      const playerRect = player.getBoundingClientRect();
      const enemyRect = enemy.getBoundingClientRect();
      const playerCenterX = playerRect.left + playerRect.width / 2;
      const playerCenterY = playerRect.top + playerRect.height / 2;
      const enemyCenterX = enemyRect.left + enemyRect.width / 2;
      const enemyCenterY = enemyRect.top + enemyRect.height / 2;

      const distance = Math.sqrt(
        Math.pow(playerCenterX - enemyCenterX, 2) +
        Math.pow(playerCenterY - enemyCenterY, 2)
      );

      const collisionThreshold = (playerRect.width + enemyRect.width) * 0.325; // 35% closer...
      if (distance < collisionThreshold) {
        if (type === 'shark' || (type === 'dolphin' && isJumping) || (playerVelocityY >= 0 || playerRect.bottom >= enemyRect.bottom)) {
          // Player collided with shark, jumping dolphin, or from the side/below
          gameOver();
          clearInterval(moveInterval);
        } else {
          // Player is falling and hits the enemy from above
          killEnemy(enemy, enemyX, type);
          let points = 0;
          if (type === 'lobster') points = 5;
          if (type === 'crab') points = 7;
          if (type === 'dolphin') points = 10;
          score += points;
          clearInterval(moveInterval);
          playerVelocityY = jumpStrength * 0.7; // Bounce effect
          killSound.play(); // Play kill sound
        }
        updateScore();
      }
    }

    function killEnemy(enemy, enemyX, type) {
      enemy.textContent = '🔥';
      setTimeout(() => {
        enemy.textContent = '💀';
        let skullY = 20;
        const floatInterval = setInterval(() => {
          skullY += 2;
          enemy.style.bottom = skullY + 'px';
          if (skullY > gameContainer.offsetHeight) {
            clearInterval(floatInterval);
            enemy.remove();
          }
        }, 50);
      }, 500);
    }

    function updateScore() {
      scoreElement.textContent = `SCORE: ${score}`;
    }

    function gameOver() {
      if (!gameIsOver) {
        gameIsOver = true;
        if (score > highScore) {
          highScore = score;
          localStorage.setItem('highScore', highScore);
          messageElement.textContent = "GAME OVER. Congrats! You got the high score!";
        } else {
          messageElement.textContent = `GAME OVER. High Score: ${highScore}`;
        }
        messageElement.style.display = 'block';
        gameOverSound.play(); // Play game over sound
        gameMusic.pause(); //Añade esta línea
        gameMusic.currentTime = 0;
      }
    }

    function startGame() {
      gameStarted = true;
      messageElement.style.display = 'none';
      gameMusic.play();
      enemyInterval = setInterval(createEnemy, 1500);
      coinInterval = setInterval(createCoin, 15000); // Create a coin every 15 seconds
      gameLoop();
    }

    function resetGame() {
      clearInterval(enemyInterval);
      clearInterval(coinInterval);
      gameIsOver = false;
      gameStarted = false;
      score = 0;
      playerX = 50;
      playerY = gameContainer.offsetHeight - 60;
      playerVelocityY = 0;
      playerVelocityX = 0;
      isJumping = false;
      updateScore();
      messageElement.style.display = 'none';
      document.querySelectorAll('.enemy').forEach(enemy => enemy.remove());
      document.querySelectorAll('.poo').forEach(poo => poo.remove());
      document.querySelectorAll('.coin').forEach(coin => coin.remove());
    }


    document.addEventListener('keydown', (e) => {
      if (gameIsOver) {
        resetGame();
        startGame();
      } else if (!gameStarted) {
        startGame();
      } else {
        if (e.key === 'ArrowLeft') {
          playerVelocityX = -5;
          player.style.transform = 'scaleX(-1)';
        }
        if (e.key === 'ArrowRight') {
          playerVelocityX = 5;
          player.style.transform = 'scaleX(1)';
        }
        if (e.key === 'ArrowUp') jump();
      }
    });

    document.addEventListener('keyup', (e) => {
      if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') playerVelocityX = 0;
    });

    function gameLoop() {
      if (!gameIsOver) {
        updatePlayerPosition();
        updateSunPosition();
        requestAnimationFrame(gameLoop);
      }
    }

    //Eagle Poo
    function dropPoo(eagleX, eagleY) {
      const poo = document.createElement('div');
      poo.className = 'poo';
      poo.textContent = '💩';
      poo.style.left = eagleX + 'px';
      poo.style.top = eagleY + 30 + 'px'; // Adjusted position
      gameContainer.appendChild(poo);

      let pooY = eagleY + 30;
      const pooSpeed = 3;

      const pooInterval = setInterval(() => {
        if (gameIsOver) {
          clearInterval(pooInterval);
          poo.remove();
          return;
        }

        pooY += pooSpeed; //Falling down
        poo.style.top = pooY + 'px';

        //Collision
        const playerRect = player.getBoundingClientRect();
        const pooRect = poo.getBoundingClientRect();

        if (
          pooRect.left < playerRect.right &&
          pooRect.right > playerRect.left &&
          pooRect.top < playerRect.bottom &&
          pooRect.bottom > playerRect.top
        ) {
          //Collision detected
          gameOver();
          clearInterval(pooInterval);
          poo.remove();
        }

        if (pooY > gameContainer.offsetHeight) {
          clearInterval(pooInterval);
          poo.remove();
        }
      }, 20);
    }

    // Coin functionality
    function createCoin() {
      if (gameIsOver) return;

      const coin = document.createElement('div');
      coin.className = 'coin';
      coin.textContent = '🪙';
      const coinX = Math.random() * (gameContainer.offsetWidth - 30);
      const coinY = gameContainer.offsetHeight - 60; // Same height as the player
      coin.style.left = coinX + 'px';
      coin.style.bottom = '20px';
      gameContainer.appendChild(coin);

      // Remove the coin after 3 seconds
      setTimeout(() => {
        if (coin.parentElement) {
          coin.remove();
        }
      }, 3000);

      // Check for collision with the player
      const coinInterval = setInterval(() => {
        if (gameIsOver) {
          clearInterval(coinInterval);
          coin.remove();
          return;
        }

        const playerRect = player.getBoundingClientRect();
        const coinRect = coin.getBoundingClientRect();

        if (
          coinRect.left < playerRect.right &&
          coinRect.right > playerRect.left &&
          coinRect.top < playerRect.bottom &&
          coinRect.bottom > playerRect.top
        ) {
          // Collision detected
          score += 25;
          updateScore();
          clearInterval(coinInterval);
          coin.remove();
          coinSound.play(); // Play coin sound
        }
      }, 20);
    }
  </script>
</body>

</html>
