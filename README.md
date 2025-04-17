# rajkishorflappy
<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Flappy Bird Game</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { text-align: center; font-family: Arial, sans-serif; }
    canvas { background: #70c5ce; display: block; margin: 20px auto; }
    #startBtn, #restartBtn {
      padding: 10px 20px;
      margin: 10px;
      font-size: 18px;
      cursor: pointer;
      border: none;
      border-radius: 8px;
      background-color: #28a745;
      color: white;
    }
    #scoreDisplay {
      font-size: 24px;
      margin-top: 10px;
    }
  </style>
</head>
<body>  <h1>Flappy Bird</h1>
  <button id="startBtn">Start Game</button>
  <button id="restartBtn" style="display:none;">Restart</button>
  <div id="scoreDisplay">Score: 0 | High Score: 0</div>
  <canvas id="gameCanvas" width="400" height="600"></canvas><audio id="flapSound" src="https://www.soundjay.com/button/sounds/button-16.mp3"></audio> <audio id="scoreSound" src="https://www.soundjay.com/button/sounds/button-10.mp3"></audio>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const startBtn = document.getElementById("startBtn");
    const restartBtn = document.getElementById("restartBtn");
    const scoreDisplay = document.getElementById("scoreDisplay");
    const flapSound = document.getElementById("flapSound");
    const scoreSound = document.getElementById("scoreSound");

    let frames = 0;
    let pipes = [];
    let score = 0;
    let highScore = localStorage.getItem("highScore") || 0;
    let gameRunning = false;

    const birdImg = new Image();
    birdImg.src = "https://i.imgur.com/Qa7cVQW.png";

    const bgImg = new Image();
    bgImg.src = "https://i.imgur.com/lnRZ8Wb.png";

    const pipeImg = new Image();
    pipeImg.src = "https://i.imgur.com/h0jN6Vf.png";

    const bird = {
      x: 50,
      y: 150,
      width: 34,
      height: 26,
      speed: 0,
      gravity: 0.25,
      jump: 4.6,
      draw() {
        ctx.drawImage(birdImg, this.x, this.y, this.width, this.height);
      },
      update() {
        this.speed += this.gravity;
        this.y += this.speed;

        if (this.y + this.height >= canvas.height) {
          this.y = canvas.height - this.height;
          gameOver();
        }
      },
      flap() {
        this.speed = -this.jump;
        flapSound.play();
      },
      reset() {
        this.y = 150;
        this.speed = 0;
      }
    };

    function createPipe() {
      const topHeight = Math.floor(Math.random() * (canvas.height / 2));
      pipes.push({
        x: canvas.width,
        height: topHeight,
        passed: false
      });
    }

    function updatePipes() {
      for (let i = 0; i < pipes.length; i++) {
        let p = pipes[i];
        p.x -= 2;

        let gap = 120;
        let bottomY = p.height + gap;
        let pipeWidth = 50;

        ctx.drawImage(pipeImg, p.x, 0, pipeWidth, p.height);
        ctx.drawImage(pipeImg, p.x, bottomY, pipeWidth, canvas.height - bottomY);

        if (
          bird.x < p.x + pipeWidth &&
          bird.x + bird.width > p.x &&
          (bird.y < p.height || bird.y + bird.height > bottomY)
        ) {
          gameOver();
        }

        if (!p.passed && p.x + pipeWidth < bird.x) {
          score++;
          p.passed = true;
          scoreSound.play();
          if (score > highScore) {
            highScore = score;
            localStorage.setItem("highScore", highScore);
          }
          scoreDisplay.innerText = `Score: ${score} | High Score: ${highScore}`;
        }
      }

      if (pipes.length && pipes[0].x + 50 < 0) {
        pipes.shift();
      }

      if (frames % 90 === 0) {
        createPipe();
      }
    }

    function drawBackground() {
      ctx.drawImage(bgImg, 0, 0, canvas.width, canvas.height);
    }

    function draw() {
      drawBackground();
      bird.draw();
      updatePipes();
    }

    function update() {
      bird.update();
    }

    function loop() {
      if (!gameRunning) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      draw();
      update();
      frames++;
      requestAnimationFrame(loop);
    }

    function startGame() {
      gameRunning = true;
      score = 0;
      frames = 0;
      pipes = [];
      bird.reset();
      scoreDisplay.innerText = `Score: ${score} | High Score: ${highScore}`;
      startBtn.style.display = "none";
      restartBtn.style.display = "none";
      loop();
    }

    function gameOver() {
      gameRunning = false;
      restartBtn.style.display = "inline-block";
      alert("Game Over! Your score: " + score);
    }

    startBtn.onclick = startGame;
    restartBtn.onclick = startGame;

    document.addEventListener("keydown", function () {
      if (gameRunning) bird.flap();
    });

    canvas.addEventListener("touchstart", function () {
      if (gameRunning) bird.flap();
    });
  </script></body>
</html>
