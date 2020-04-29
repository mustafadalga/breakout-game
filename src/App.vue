<template>
  <div id="app">
    <canvas id="gameCanvas" :width="canvasWidth" :height="canvasHeight"></canvas>

    <div id="options">
      <button type="button" class="btn btn-info btn-lg btn3d" @click="start()" ref="startBtn">Başlat</button>
    </div>
    <div id="warning" v-if="warning">{{warning}}</div>
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
      canvasPaddingLeft: 15,
      warning: null,
      ctx: null,
      ballX: null,
      ballY: null,
      ballSpeedX: 3,
      ballSpeedY: -3,
      ballRadius: 15,
      paddleHeight: 15,
      paddleWidth: 120,
      paddleX: null,
      paddleY: null,
      rightPresses: false,
      leftPresses: false,
      brickRowCount: 3,
      brickColumnCount: 17,
      brickHeight: 30,
      brickWidth: 60,
      brickPadding: 10,
      brickOffsetTop: 30,
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
    setSoundFile() {
      this.brickSound = new Audio(BrickSoundEffect);
    },
    setBallImage() {
      this.ball = new Image();
      this.ball.src = ballImage;
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
      var relativeX = Math.floor(e.touches[0].clientX);
      if (
        relativeX > 0 &&
        relativeX < this.canvasWidth - this.paddleWidth / 2
      ) {
        this.paddleX = relativeX;
      }
    },
    initBricks() {
      for (let row = 0; row < this.brickRowCount; row++) {
        this.bricks[row] = [];
        for (let col = 0; col < this.brickColumnCount; col++) {
          this.bricks[row][col] = {
            x: 0,
            y: 0,
            status: 1,
            color: this.getRandomColor()
          };
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
    draw() {
      this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight);
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
          this.decreaseLives();
          if (this.lives > 0) {
            this.changePauseStatus(true);
            this.ballSpeedY = -this.ballSpeedY;
            this.setBallLocation();
            this.setPaddleLocation();

            this.showWarning("Kalan hakkınız:" + this.lives);
            setTimeout(() => {
              this.draw();
              this.changePauseStatus(true);
            }, 1000);
            setTimeout(() => {
              this.changePauseStatus(false);
              this.showWarning(null);
              this.draw();
            }, 2000);
          } else {
            this.changeButtonText("Yeniden Oyna");
            this.changePauseStatus(true);
            this.isGameOver = true;
            this.showWarning("Tüm haklarınız tükendi.Oyunu kaybettiniz.");
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
            let brickX =
              col * (this.brickWidth + this.brickPadding) +
              this.canvasPaddingLeft;
            let brickY =
              row * (this.brickHeight + this.brickPadding) +
              this.brickOffsetTop;
            this.bricks[row][col].x = brickX;
            this.bricks[row][col].y = brickY;
            this.ctx.beginPath();
            this.ctx.rect(brickX, brickY, this.brickWidth, this.brickHeight);
            this.ctx.fillStyle = this.bricks[row][col].color;
            this.ctx.fill();
            this.ctx.closePath();
          }
        }
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
      this.ctx.fillStyle = "#00a5c3";
      this.ctx.fill();
      this.ctx.closePath();
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
    },
    collisionDetection() {
      for (let row = 0; row < this.brickRowCount; row++) {
        for (let col = 0; col < this.brickColumnCount; col++) {
          let brick = this.bricks[row][col];
          if (brick.status === 1) {
            if (
              this.ballX > brick.x &&
              this.ballX < brick.x + this.brickWidth &&
              this.ballY > brick.y &&
              this.ballY < brick.y + this.brickHeight
            ) {
              this.ballSpeedY = -this.ballSpeedY;
              brick.status = 0;
              this.increaseScore();
              this.playBrickSound();

              if (this.score === 1) {
                if (this.level === this.maxLevel) {
                  this.changeButtonText("Yeniden Oyna");
                  this.isGameOver = true;
                  this.changePauseStatus(true);
                  this.showWarning(
                    "Tebrikler! Tüm seviyeleri başarıyla tamamladınız."
                  );
                } else {
                  this.increaseLevel();
                  this.increaseBrickRow();
                  this.initBricks();
                  this.score = 0;
                  this.increaseBallSpeed();
                  this.setBallLocation();
                  this.setPaddleLocation();
                  this.changePauseStatus(true);
                  this.showWarning(
                    "Level " +
                      (this.level - 1) +
                      " Tamamlandı.Sonraki level'e geçiş yapılıyor..."
                  );
                  setTimeout(() => {
                    this.changePauseStatus(false);
                    this.showWarning(null);
                    this.draw();
                  }, 3000);
                }
              }
            }
          }
        }
      }
    },
    start() {
      if (!this.isGameOver) {
        if (this.paused) {
          this.changeButtonText("Duraklat");
          this.changePauseStatus(false);
          this.draw();
        } else {
          this.changeButtonText("Başlat");
          this.changePauseStatus(true);
        }
      } else {
        this.changeButtonText("Duraklat");
        this.isGameOver = false;
        this.changePauseStatus(false);
        this.resetGameData();
      }
    },
    resetGameData() {
      this.level = 1;
      this.score = 0;
      this.lives = 3;
      this.ballSpeedX = 3;
      this.ballSpeedY = -3;
      this.setPaddleLocation();
      this.setBallLocation();
      this.initBricks();
      this.draw();
    },
    playBrickSound() {
      if (this.brickSound.paused) {
        this.brickSound.play();
      } else {
        this.brickSound.currentTime = 0;
      }
    },
    generateRandomBallXNumber() {
      return Math.floor(Math.random() * 21) - 10;
    },

    changeButtonText(text) {
      this.$refs["startBtn"].innerText = text;
    },
    decreaseLives() {
      if (this.lives > 0) {
        this.lives--;
      }
    },
    changePauseStatus(status) {
      this.paused = status;
    },
    increaseScore() {
      this.score++;
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
    showWarning(warning) {
      this.warning = warning;
    },

    checkWindowResize() {
      let innerWidth = window.innerWidth;
      if (innerWidth >= 1150 && innerWidth < 1200) {
        this.updateWindowResizeData(innerWidth, 16, 10, 25);
      } else if (innerWidth >= 1100 && innerWidth < 1150) {
        this.updateWindowResizeData(innerWidth, 16, 10, 0);
      } else if (innerWidth >= 1050 && innerWidth < 1100) {
        this.updateWindowResizeData(innerWidth, 15, 10, 10);
      } else if (innerWidth >= 1000 && innerWidth < 1050) {
        this.updateWindowResizeData(innerWidth, 15, 7, 0);
      } else if (innerWidth >= 950 && innerWidth < 1000) {
        this.updateWindowResizeData(innerWidth, 14, 8, 0);
      } else if (innerWidth >= 900 && innerWidth < 950) {
        this.updateWindowResizeData(innerWidth, 14, 5, 0);
      } else if (innerWidth >= 850 && innerWidth < 900) {
        this.updateWindowResizeData(innerWidth, 13, 5, 10);
      } else if (innerWidth >= 800 && innerWidth < 850) {
        this.updateWindowResizeData(innerWidth, 13, 3, 0);
      } else if (innerWidth >= 750 && innerWidth < 800) {
        this.updateWindowResizeData(innerWidth, 12, 3, 0);
      } else if (innerWidth >= 700 && innerWidth < 750) {
        this.updateWindowResizeData(innerWidth, 11, 3, 5);
      } else if (innerWidth >= 650 && innerWidth < 700) {
        this.updateWindowResizeData(innerWidth, 10, 3, 10);
      } else if (innerWidth >= 600 && innerWidth < 650) {
        this.updateWindowResizeData(innerWidth, 10, 3, 0);
      } else if (innerWidth >= 550 && innerWidth < 600) {
        this.updateWindowResizeData(innerWidth, 9, 2, 0);
      } else if (innerWidth >= 500 && innerWidth < 550) {
        this.updateWindowResizeData(innerWidth, 8, 2, 0);
      } else if (innerWidth >= 450 && innerWidth < 500) {
        this.updateWindowResizeData(innerWidth, 8, 2, 0);
      } else if (innerWidth >= 400 && innerWidth < 450) {
        this.updateWindowResizeData(innerWidth, 7, 2, 0);
      } else if (innerWidth >= 350 && innerWidth < 400) {
        this.updateWindowResizeData(innerWidth, 6, 2, 0);
      } else if (innerWidth >= 300 && innerWidth < 350) {
        this.updateWindowResizeData(innerWidth, 5, 2, 0);
      } else if (innerWidth < 300) {
        this.updateWindowResizeData(innerWidth, 4, 2, 0);
      } else {
        this.updateWindowResizeData(1200, 17, 10, 5);
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
      brickPadding,
      canvasPaddingLeft
    ) {
      this.canvasWidth = canvasWidth;
      this.brickColumnCount = brickColumnCount;
      this.brickPadding = brickPadding;
      this.canvasPaddingLeft = canvasPaddingLeft;
    }
  }
};
</script>
<style>
body {
  background: #c94b4b;
  background: -webkit-linear-gradient(to left, #4b134f, #c94b4b);
  background: linear-gradient(to left, #4b134f, #c94b4b);
}
* {
  margin: 0;
  padding: 0;
}
#gameCanvas {
  background: transparent;
  display: block;
  margin: 0 auto;
  cursor: pointer;
  border-bottom: 10px solid #ffffff;
}
#warning {
  height: 32px;
  width: 300px;
  margin-left: auto;
  margin-right: auto;
  border: 1px solid #ffffff;
  margin-top: 1em;
  background: rgba(255, 255, 255, 0.2);
  font-size: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ffffff;
  padding: 0.5em;
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