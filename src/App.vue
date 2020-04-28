<template>
  <div id="app">
    <canvas id="gameCanvas" :width="canvasWidth" :height="canvasHeight"></canvas>
    <div id="options">
      <button
        type="button"
        class="btn btn-info btn-lg btn3d"
        @click="start()"
        ref="startBtn"
      >Başlat {{ brickColumnCount }}</button>
    </div>
    <div class="modal-overlay">
      <div class="modal">
        <h2>Lütfen cihazını çevirin !</h2>
      </div>
    </div>
  </div>
</template>

<script>
import ballImage from "./assets/img/football-ball.png";
import BrickSoundEffect from "./assets/sound/brick-effect.mp3";
export default {
  name: "Home",
  data() {
    return {
      canvas: null,
      canvasWidth: 1200,
      canvasHeight: 400,
      canvasPaddingLeft: 40,
      messageFontSize: "18px",
      ctx: null,
      ballX: null,
      ballY: null,
      ballSpeedX: 5,
      ballSpeedY: -5,
      ballRadius: 15,
      paddleHeight: 15,
      paddleWidth: 120,
      paddleX: null,
      paddleY: null,
      rightPresses: false,
      leftPresses: false,
      brickRowCount: 3,
      brickColumnCount: 20,
      brickHeight: 30,
      brickMaxWidth: 60,
      brickMinWidth: 30,
      brickPadding: 10,
      brickOffsetTop: 30,
      brickOffsetLeft: 10,
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
        },
        {
          color1: "#ff8235",
          color2: "#30e8bf"
        },
        {
          color1: "#f7971e",
          color2: "#ffd200"
        }
      ]
    };
  },
  created() {
    this.checkWindowResize();
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
    checkWindowResize() {
      let innerWidth = window.innerWidth;
      if (innerWidth >= 1100 && innerWidth < 1200) {
        this.updateWindowResizeData(innerWidth, 17, 9, 30);
      } else if (innerWidth >= 1000 && innerWidth < 1100) {
        this.updateWindowResizeData(innerWidth, 16, 8, 25);
      } else if (innerWidth >= 900 && innerWidth < 1000) {
        this.updateWindowResizeData(innerWidth, 15, 5, 20);
      } else if (innerWidth >= 800 && innerWidth < 900) {
        this.updateWindowResizeData(innerWidth, 13, 5, 10);
      } else if (innerWidth >= 700 && innerWidth < 800) {
        this.updateWindowResizeData(innerWidth, 12, 5, 5);
      } else if (innerWidth >= 600 && innerWidth < 700) {
        this.updateWindowResizeData(innerWidth, 10, 5, 5);
      } else if (innerWidth >= 500 && innerWidth < 600) {
        this.updateWindowResizeData(innerWidth, 8, 5, 1);
      } else if (innerWidth >= 400 && innerWidth < 500) {
        this.updateWindowResizeData(innerWidth, 7, 5, 10);
      } else if (innerWidth >= 300 && innerWidth < 400) {
        this.updateWindowResizeData(innerWidth, 6, 5, 10);
        this.messageFontSize = "14px";
      } else if (innerWidth < 300) {
        this.updateWindowResizeData(innerWidth, 5, 5, 10);
        this.messageFontSize = "12px";
      } else {
        this.updateWindowResizeData(1200, 20, 10, 30);
      }
      if (this.ctx !== null) {
        this.setPaddleLocation();
        this.setBallLocation();
        this.initBricks();
        this.draw();
      }
    },
    updateWindowResizeData(
      canvasWidth,
      brickColumnCount,
      brickOffsetLeft,
      canvasPaddingLeft
    ) {
      this.canvasWidth = canvasWidth;
      this.brickColumnCount = brickColumnCount;
      this.brickOffsetLeft = brickOffsetLeft;
      this.canvasPaddingLeft = canvasPaddingLeft;
    },
    start() {
      if (!this.isGameOver) {
        if (this.paused) {
          this.changeButtonText("Duraklat");
          this.paused = false;
          this.draw();
        } else {
          this.changeButtonText("Başlat");
          this.paused = true;
        }
      } else {
        this.changeButtonText("Duraklat");
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
      this.brickSound = new Audio(BrickSoundEffect);
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
    generateRandomBallXNumber() {
      return Math.floor(Math.random() * 21) - 10;
    },
    generateRandomBrickWidth() {
      return Math.floor(
        Math.random() * (this.brickMinWidth - this.brickMaxWidth + 1) +
          this.brickMaxWidth
      );
    },
    setBallLocation() {
      this.ballX = this.canvasWidth / 2 + this.generateRandomBallXNumber();
      this.ballY = this.canvasHeight - 30;
    },
    setPaddleLocation() {
      this.paddleX = (this.canvasWidth - this.paddleWidth) / 2;
      this.paddleY = this.canvasHeight - this.paddleHeight;
    },
    getCanvasObject() {
      this.canvas = document.getElementById("gameCanvas");
      this.ctx = this.canvas.getContext("2d");
    },
    addEventListener() {
      document.addEventListener("keydown", this.keyDownHandler);
      document.addEventListener("keyup", this.keyUpHandler);
      document.addEventListener("mousemove", this.mouseMoveHandler);
      document.addEventListener("touchmove", this.touchMoveHandler);
      window.addEventListener("resize", this.checkWindowResize);
    },
    checkTotalBrickWidth(row, col) {
      let brickRandomWidth = this.generateRandomBrickWidth();
      let total = this.getTotalWidth(row, col);
      if (total === col * this.brickMaxWidth) {
        this.checkTotalBrickWidth(row, col);
      }
      {
        return brickRandomWidth;
      }
    },
    initBricks() {
      for (let row = 0; row < this.brickRowCount; row++) {
        this.bricks[row] = [];
        for (let col = 0; col < this.brickColumnCount; col++) {
          let brickRandomWidth = null;

          if (col >= (this.brickColumnCount * 2) / 3) {
            brickRandomWidth = this.checkTotalBrickWidth(row, col);
          } else {
            brickRandomWidth = this.generateRandomBrickWidth();
          }
          this.bricks[row][col] = {
            x: 0,
            y: 0,
            width: brickRandomWidth,
            status: 1,
            color: this.getRandomColor()
          };
        }
      }
    },
    draw() {
      this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight); //Her konum değiştirildiğinden cancas yenile
      this.drawBricks();
      this.drawBall();
      this.drawPaddle();
      this.drawScore();
      this.drawLives();
      this.drawLevel();
      this.collisionDetection();

      if (this.ballY + this.ballSpeedY < this.ballRadius) {
        this.ballSpeedY = -this.ballSpeedY;
      } else if (
        this.ballY + this.ballSpeedY >
        this.canvasHeight - this.ballRadius
      ) {
        if (
          this.ballX > this.paddleX &&
          this.ballX < this.paddleX + this.paddleWidth
        ) {
          this.ballSpeedY = -this.ballSpeedY;
        } else {
          if (this.lives > 0) {
            this.lives--;
          }
          if (this.lives > 0) {
            this.paused = true;
            this.ballSpeedY = -this.ballSpeedY;
            this.setBallLocation();
            this.setPaddleLocation();

            this.drawMessage(
              "#d32f2f",
              "Kalan Hakkınız:" + this.lives,
              this.canvasWidth / 2 - 60
            );

            setTimeout(() => {
              this.draw();
              this.paused = true;
            }, 1000);
            setTimeout(() => {
              this.paused = false;
              this.draw();
            }, 2000);
          } else {
            this.changeButtonText("Yeniden Oyna");
            setTimeout(() => {
              this.paused = true;
            }, 100);
            this.isGameOver = true;
            this.drawMessage(
              "#d32f2f",
              "Tüm haklarınız tükendi.Oyunu kaybettiniz.",
              this.canvasWidth / 2 - 120
            );
          }
        }
      }
      if (
        this.ballX + this.ballSpeedX < this.ballRadius ||
        this.ballX + this.ballSpeedX > this.canvasWidth - this.ballRadius
      ) {
        this.ballSpeedX = -this.ballSpeedX;
      }

      if (
        this.rightPresses &&
        this.paddleX < this.canvasWidth - this.paddleWidth
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
      for (let row = 0; row < this.brickRowCount; row++) {
        for (let col = 0; col < this.brickColumnCount; col++) {
          if (this.bricks[row][col].status === 1) {
            let brickX = null;
            if (col > 0) {
              brickX =
                this.getTotalWidth(row, col) +
                col * this.brickOffsetLeft +
                this.canvasPaddingLeft;
            } else {
              brickX = this.canvasPaddingLeft;
            }
            let brickY =
              row * (this.brickHeight + this.brickPadding) +
              this.brickOffsetTop;
            this.bricks[row][col].x = brickX;
            this.bricks[row][col].y = brickY;
            this.ctx.beginPath();
            this.ctx.rect(
              brickX,
              brickY,
              this.bricks[row][col].width,
              this.brickHeight
            );
            this.ctx.fillStyle = this.bricks[row][col].color;
            this.ctx.fill();
            this.ctx.closePath();
          }
        }
      }
    },
    getTotalWidth(rowNumber, colNumber) {
      let totalBrickWidth = 0;
      for (let col = 0; col < colNumber; col++) {
        totalBrickWidth += this.bricks[rowNumber][col].width;
      }
      return totalBrickWidth;
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
        relativeX < this.canvasWidth - this.paddleWidth / 2
      ) {
        this.paddleX = relativeX - this.paddleWidth / 2;
      }
    },
    touchMoveHandler(e) {
      var x = e.touches[0].clientX;
      this.paddleX = x;
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
      for (let row = 0; row < this.brickRowCount; row++) {
        for (let col = 0; col < this.brickColumnCount; col++) {
          let brick = this.bricks[row][col];
          if (brick.status === 1) {
            if (
              this.ballX > brick.x &&
              this.ballX < brick.x + brick.width &&
              this.ballY > brick.y &&
              this.ballY < brick.y + this.brickHeight
            ) {
              this.ballSpeedY = -this.ballSpeedY;
              brick.status = 0;
              this.score++;

              this.playBrickSound();

              if (this.score === this.brickRowCount * this.brickColumnCount) {
                if (this.level === this.maxLevel) {
                  this.changeButtonText("Yeniden Oyna");
                  this.isGameOver = true;
                  this.paused = true;
                  this.drawMessage(
                    "#d32f2f",
                    "Oyunu kazandınız.Tebrikler,Tüm seviyeleri başarıyla tamamladınız.",
                    this.canvasWidth / 2 - 230
                  );
                } else {
                  this.increaseLevel();
                  this.increaseBrickRow();
                  this.initBricks();
                  this.score = 0;
                  this.increaseBallSpeed();
                  this.setBallLocation();
                  this.setPaddleLocation();
                  this.paused = true;
                  this.drawMessage(
                    "#d32f2f",
                    "Level " +
                      (this.level - 1) +
                      " Tamamlandı.Sonraki Level'e geçiş yapılıyor...",
                    this.canvasWidth / 2 - 200
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
    changeButtonText(text) {
      this.$refs["startBtn"].innerText = text;
    },
    increaseLevel() {
      this.level++;
    },
    increaseBrickRow() {
      this.brickRowCount++;
    },
    increaseBallSpeed() {
      this.ballSpeedX += 1;
      this.ballSpeedY = -this.ballSpeedY;
      this.ballSpeedY -= 1;
    },
    drawMessage(color, message, LocationX) {
      this.ctx.beginPath();
      this.ctx.font = this.messageFontSize + " Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText(message, LocationX, this.canvasHeight - 50);
    },
    drawScore() {
      this.ctx.font = "16px Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText("Skor:" + this.score, 8, 20);
    },

    drawLives() {
      this.ctx.font = "16px Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText("Hak:" + this.lives, this.canvasWidth - 65, 20);
    },
    drawLevel() {
      this.ctx.font = "16px Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText("Seviye:" + this.level, this.canvasWidth / 2 - 30, 20);
    }
  }
};
</script>
<style>
body {
  background: #004d40;
  
}
* {
  margin: 0;
  padding: 0;
}
#gameCanvas {
  background: #c94b4b;
  background: -webkit-linear-gradient(to left, #4b134f, #c94b4b);
  background: linear-gradient(to left, #4b134f, #c94b4b);

  display: block;
  margin: 0 auto;
  cursor: pointer;
}
#options {
  margin-top: 1em;
  display: flex;
  justify-content: center;
}
.btn {
  display: inline-block;
  font-weight: 400;
  color: #ffffff;
  text-align: center;
  vertical-align: middle;
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  background-color: transparent;
  border: 1px solid transparent;
  padding: 0.375rem 2rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: 0.25rem;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out,
    border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}
.btn3d {
  position: relative;
  top: -6px;
  border: 0;
  transition: all 40ms linear;
  margin-top: 10px;
  margin-bottom: 10px;
  margin-left: 2px;
  margin-right: 2px;
}
.btn3d:active:focus,
.btn3d:focus:hover,
.btn3d:focus {
  -moz-outline-style: none;
  outline: medium none;
}

.btn3d:active,
.btn3d.active {
  top: 2px;
}
.btn3d.btn-info {
  box-shadow: 0 0 0 1px #00a5c3 inset, 0 0 0 2px rgba(255, 255, 255, 0.15) inset,
    0 8px 0 0 #348fd2, 0 8px 8px 1px rgba(0, 0, 0, 0.5);
  background-color: #39b3d7;
}
.btn3d.btn-info:active,
.btn3d.btn-info.active {
  box-shadow: 0 0 0 1px #00a5c3 inset, 0 0 0 1px rgba(255, 255, 255, 0.15) inset,
    0 1px 3px 1px rgba(0, 0, 0, 0.3);
  background-color: #39b3d7;
}
.modal-overlay {
  display: none;
  position: fixed;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  z-index: 1;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
.modal {
  width: 50%;
  height: 25%;
  background: #c94b4b;
  background: -webkit-linear-gradient(to left, #4b134f, #c94b4b);
  background: linear-gradient(to left, #4b134f, #c94b4b);
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  align-items: center;
  justify-content: center;
}
.modal h2 {
  color: #ffffff;
}
@media screen and (orientation: landscape) and (max-device-width: 900px) {
  .modal-overlay {
    display: block;
  }
}
</style>