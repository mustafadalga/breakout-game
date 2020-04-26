<template>
  <div id="app">
    <canvas id="gameCanvas" width="1000" height="400"></canvas>
    <div id="options">
      <button @click="start()" ref="startBtn">Başlat</button>
    </div>
  </div>
</template>

<script>
import ballImage from "./assets/img/football-ball.png";
import BrickSoundEffeck from "./assets/sound/2.mp3";

export default {
  name: "Home",
  data() {
    return {
      canvas: null,
      ctx: null,
      ballX: null,
      ballY: null,
      ballSpeedX: 5,
      ballSpeedY: -5,
      ballRadius: 15,
      paddleHeight: 10,
      paddleWidth: 120,
      paddleX: null,
      paddleY: null,
      rightPresses: false,
      leftPresses: false,
      brickRowCount: 3,
      brickColumnCount: 16,
      brickWidth: 50,
      brickHeight: 20,
      brickPadding: 10,
      brickOffsetTop: 30,
      brickOffsetLeft: 30,
      score: 0,
      lives: 3,
      level: 1,
      maxLevel: 5,
      paused: true,
      bricks: [],
      ball: null,
      isGameOver: false,
      brickSound: null,
      brickColors: [
        {
          color1: "#00f260",
          color2: "#0575e6"
        },
        {
          color1: "#00b09b",
          color2: "#96c93d"
        },
        {
          color1: "#22c1c3",
          color2: "#fdbb2d"
        },
        {
          color1: "#ff9966",
          color2: "#ff5e62"
        },
        {
          color1: "#7f00ff",
          color2: "#e100ff"
        },
        {
          color1: "#06beb6",
          color2: "#48b1bf"
        },
        {
          color1: "#000000",
          color2: "#0f9b0f"
        }
      ]
    };
  },
  mounted() {
    this.setSoundFile();
    this.setBallImage();
    this.getCanvasObject();
    this.setPaddleLocation();
    this.setBallLocation();
    this.addEventListener();
    this.initBricks();
    this.draw();
  },

  methods: {
    start() {
      if (!this.isGameOver) {
        if (this.paused) {
          this.$refs["startBtn"].innerText = "Duraklat";
          this.paused = false;
          this.draw();
        } else {
          this.$refs["startBtn"].innerText = "Başlat";
          this.paused = true;
        }
      } else {
        this.$refs["startBtn"].innerText = "Duraklat";
        this.isGameOver = false;
        this.paused = false;
        this.resetGameData();
      }
    },
    resetGameData() {
      this.level = 1;
      this.score = 0;
      this.lives = 3;
      this.ballSpeedX = 5;
      this.ballSpeedY = 5;
      this.setPaddleLocation();
      this.setBallLocation();
      this.initBricks();
      this.draw();
    },
    setSoundFile() {
      this.brickSound = new Audio(BrickSoundEffeck);
    },
    playBrickSound() {
      if (this.brickSound.paused) {
        this.brickSound.play();
      } else {
        this.brickSound.currentTime = 0;
      }
    },
    setBallImage() {
      this.ball = new Image();
      this.ball.src = ballImage;
    },
    generateRandomNumber() {
      return Math.floor(Math.random() * 21) - 10;
    },
    setBallLocation() {
      this.generateRandomNumber();
      this.ballX = this.canvas.width / 2 + this.generateRandomNumber();
      this.ballY = this.canvas.height - 30;
    },
    setPaddleLocation() {
      this.paddleX = (this.canvas.width - this.paddleWidth) / 2;
      this.paddleY = this.canvas.height - this.paddleHeight;
    },
    getCanvasObject() {
      this.canvas = document.getElementById("gameCanvas");
      this.ctx = this.canvas.getContext("2d");
    },
    setBall() {},
    addEventListener() {
      document.addEventListener("keydown", this.keyDownHandler);
      document.addEventListener("keyup", this.keyUpHandler);
      document.addEventListener("mousemove", this.mouseMoveHandler);
    },
    initBricks() {
      for (var c = 0; c < this.brickColumnCount; c++) {
        this.bricks[c] = [];
        for (var r = 0; r < this.brickRowCount; r++) {
          this.bricks[c][r] = { x: 0, y: 0, status: 1 };
        }
      }
    },
    draw() {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height); //Her konum değiştirildiğinden cancas yenile
      this.drawBricks();
      this.drawBall();
      this.drawPaddle();
      this.drawScore();
      this.drawLives();
      this.drawLevel();
      this.collisionDetection();

      //En aşağıya veya yukarıya gittiğinde

      if (this.ballY + this.ballSpeedY < this.ballRadius) {
        this.ballSpeedY = -this.ballSpeedY;
      } else if (
        this.ballY + this.ballSpeedY >
        this.canvas.height - this.ballRadius
      ) {
        //Rakete dokunduğunda

        if (
          this.ballX > this.paddleX &&
          this.ballX < this.paddleX + this.paddleWidth
        ) {
          this.ballSpeedY = -this.ballSpeedY;
        } else {
          this.lives--;
          
          if (this.lives > 0) {
            this.paused = true;
            this.ballSpeedY = -this.ballSpeedY;
            this.setBallLocation();
            this.setPaddleLocation();

            this.drawMessage("#d32f2f", "Kalan Hakkınız:" + this.lives);

            setTimeout(() => {
              this.draw();
              this.paused = true;
            }, 1000);

            setTimeout(() => {
              this.paused = false;
              
              this.draw();
            }, 2000);
          } else {
            this.$refs["startBtn"].innerText = "Yeniden Oyna";
            this.paused = true;
            this.isGameOver = true;
            this.drawMessage("#d32f2f", "Oyunu Kaybettiniz");
          }
        }
      }

      //En sağ veya sola gittiğinde
      if (
        this.ballX + this.ballSpeedX < this.ballRadius ||
        this.ballX + this.ballSpeedX > this.canvas.width - this.ballRadius
      ) {
        this.ballSpeedX = -this.ballSpeedX;
      }

      if (
        this.rightPresses &&
        this.paddleX < this.canvas.width - this.paddleWidth
      ) {
        this.paddleX += 7;
      } else if (this.leftPresses && this.paddleX > 0) {
        this.paddleX -= 7;
      }

      this.ballX += this.ballSpeedX;
      this.ballY += this.ballSpeedY;
      if (!this.paused) {
        requestAnimationFrame(this.draw);
      }
    },
    drawBricks() {
      for (
        var columnIndex = 0;
        columnIndex < this.brickColumnCount;
        columnIndex++
      ) {
        for (var rowIndex = 0; rowIndex < this.brickRowCount; rowIndex++) {
          if (this.bricks[columnIndex][rowIndex].status === 1) {
            var brickX =
              columnIndex * (this.brickWidth + this.brickPadding) +
              this.brickOffsetLeft;
            var brickY =
              rowIndex * (this.brickHeight + this.brickPadding) +
              this.brickOffsetTop;
            this.bricks[columnIndex][rowIndex].x = brickX;
            this.bricks[columnIndex][rowIndex].y = brickY;
            this.ctx.beginPath();
            this.ctx.rect(brickX, brickY, this.brickWidth, this.brickHeight);

            //   this.ctx.fillStyle = this.getRandomColor();
            this.ctx.fillStyle = "#fdbb2d";
            this.ctx.fill();
            this.ctx.closePath();
          }
        }
      }
    },
    getRandomColor() {
      let randomNumber = Math.floor(
        Math.random() * Math.floor(this.brickColors.length)
      );
      let gradientColor = this.ctx.createLinearGradient(0, 0, 0, 150);
      gradientColor.addColorStop(0, this.brickColors[randomNumber].color1);
      gradientColor.addColorStop(1, this.brickColors[randomNumber].color2);
      return gradientColor;
    },
    keyDownHandler(e) {
      if (e.keyCode === 39) {
        this.rightPresses = true;
      } else if (e.keyCode === 37) {
        this.leftPresses = true;
      }
    },

    keyUpHandler(e) {
      if (e.keyCode === 39) {
        this.rightPresses = false;
      } else if (e.keyCode === 37) {
        this.leftPresses = false;
      }
    },
    mouseMoveHandler(e) {
      var relativeX = e.clientX - this.canvas.offsetLeft;

      if (
        relativeX > this.paddleWidth / 2 &&
        relativeX < this.canvas.width - this.paddleWidth / 2
      ) {
        this.paddleX = relativeX - this.paddleWidth / 2;
      }
    },
    drawBall() {
      this.ctx.drawImage(
        this.ball,
        this.ballX,
        this.ballY,
        this.ballRadius,
        this.ballRadius
      );
    },

    drawPaddle() {
      this.ctx.beginPath();
      this.ctx.rect(
        this.paddleX,
        this.paddleY,
        this.paddleWidth,
        this.paddleHeight
      );
      this.ctx.fillStyle = "#0095DD";
      this.ctx.fill();
      this.ctx.closePath();
    },

    collisionDetection() {
      for (
        let columnIndex = 0;
        columnIndex < this.brickColumnCount;
        columnIndex++
      ) {
        for (let rowIndex = 0; rowIndex < this.brickRowCount; rowIndex++) {
          var b = this.bricks[columnIndex][rowIndex];
          if (b.status === 1) {
            if (
              this.ballX > b.x &&
              this.ballX < b.x + this.brickWidth &&
              this.ballY > b.y &&
              this.ballY < b.y + this.brickHeight
            ) {
              this.ballSpeedY = -this.ballSpeedY;
              b.status = 0;
              this.score++;

              this.playBrickSound();

              if (this.score === this.brickRowCount * this.brickColumnCount) {
                if (this.level === this.maxLevel) {
                  alert("You Win,congratilations");
                  document.location.reload();
                } else {
                  this.level++;
                  //start the next level
                  this.brickRowCount++;
                  this.initBricks();
                  this.score = 0;
                  this.ballSpeedX += 1;
                  this.ballSpeedY = -this.ballSpeedY;
                  this.ballSpeedY -= 1;
                  this.setBallLocation();
                  this.setPaddleLocation();
                  this.paused = true;
                  this.drawMessage(
                    "#d32f2f",
                    "Level " +
                      (this.level - 1) +
                      " Tamamlandı.Sonraki Level'e geçiş yapılıyor..."
                  );
                  setTimeout(() => {
                    this.paused = false;
                    this.draw();
                  }, 3000);
                }
              }
            }
          }
        }
      }
    },
    drawMessage(color, message) {
      this.ctx.beginPath();
      /*
      this.ctx.rect(0, 0, 300, 100);
      this.ctx.fillStyle = color;
      this.ctx.fill();
      */
      this.ctx.font = "24px Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText(
        message,
        this.canvas.width / 2 - 75,
        this.canvas.height / 2 - 15
      );
    },
    drawScore() {
      this.ctx.font = "16px Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText("Score:" + this.score, 8, 20);
    },

    drawLives() {
      this.ctx.font = "16px Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText("Lives:" + this.lives, this.canvas.width - 65, 20);
    },
    drawLevel() {
      this.ctx.font = "16px Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText("Level:" + this.level, this.canvas.width / 2 - 30, 20);
    }
  }
};
</script>
<style >
body {
  background: #004d40;
}
* {
  margin: 0;
  padding: 0;
}

#gameCanvas {
  background: #c94b4b; /* fallback for old browsers */
  background: -webkit-linear-gradient(
    to left,
    #4b134f,
    #c94b4b
  ); /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(
    to left,
    #4b134f,
    #c94b4b
  ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */

  display: block;
  margin: 0 auto;
  cursor: pointer;
}
#options {
  margin-top: 1em;
  display: flex;
  justify-content: center;
}
#options button {
  width: 8em;
  height: 3em;
  background: rgb(66, 184, 221);
  border: none;
  border-radius: 1em;
  outline: none;
  cursor: pointer;
}
#options button:not(:first-child) {
  margin-left: 1em;
}
</style>