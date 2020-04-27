<template>
  <div id="app">
    <canvas id="gameCanvas" :width="canvasWidth" :height="canvasHeight"></canvas>
    <div id="options">
      <button type="button" class="btn btn-info btn-lg btn3d" @click="start()" ref="startBtn">Başlat</button>
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
      canvasWidth: 1000,
      canvasHeight: 400,
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
      brickColumnCount: 16,
      brickHeight: 25,
      minBrickWidth: 30,
      maxBrickWidth: 60,
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
        Math.random() * (this.maxBrickWidth - this.minBrickWidth + 1) +
          this.minBrickWidth
      );
    },
    setBallLocation() {
      this.ballX = this.canvasWidth / 2 + this.generateRandomBallXNumber();
      this.ballY = this.canvas.height - 30;
    },
    setPaddleLocation() {
      this.paddleX = (this.canvasWidth - this.paddleWidth) / 2;
      this.paddleY = this.canvas.height - this.paddleHeight;
    },
    getCanvasObject() {
      this.canvas = document.getElementById("gameCanvas");
      this.ctx = this.canvas.getContext("2d");
    },
 
    addEventListener() {
      document.addEventListener("keydown", this.keyDownHandler);
      document.addEventListener("keyup", this.keyUpHandler);
      document.addEventListener("mousemove", this.mouseMoveHandler);
      window.addEventListener("resize", this.checkResize);
    },
    initBricks() {
      for (let row = 0; row < this.brickRowCount; row++) {
        this.bricks[row] = [];
        for (let col = 0; col < this.brickColumnCount; col++) {
          this.bricks[row][col] = {
            x: 0,
            y: 0,
            width: this.generateRandomBrickWidth(),
            status: 1,
            color: this.getRandomColor()
          };
        }
      }

      /*
      for (var c = 0; c < this.brickColumnCount; c++) {
        this.bricks[c] = [];
        for (var r = 0; r < this.brickRowCount; r++) {
          this.bricks[c][r] = {
            x: 0,
            y: 0,
            width: this.generateRandomBrickWidth(),
            status: 1,
            color: this.getRandomColor()
          };
        }
      }
      */
    },
    draw() {
      this.ctx.clearRect(0, 0, this.canvasWidth, this.canvas.height); //Her konum değiştirildiğinden cancas yenile
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
            this.$refs["startBtn"].innerText = "Yeniden Oyna";
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

      //En sağ veya sola gittiğinde
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
            let brickX=null
            if(col>0){
               brickX =this.getTotalWidth(row, col) + col * this.brickOffsetLeft+(this.brickOffsetLeft*2);
            }else{
               brickX =this.brickOffsetLeft*2;
            }
            let brickY =  row * (this.brickHeight + this.brickPadding) + this.brickOffsetTop;
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
    checkResize() {
      let innerWidth = window.innerWidth;
      console.log(innerWidth);
      if (innerWidth >= 1000) {
        this.canvasWidth = 1000;
      } else {
        this.canvasWidth = innerWidth;
        this.minBrickWidth = 15;
        this.maxBrickWidth = 30;
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
                  this.$refs["startBtn"].innerText = "Yeniden Oyna";
                  this.isGameOver = true;
                  this.paused = true;
                  this.drawMessage(
                    "#d32f2f",
                    "Oyunu kazandınız.Tebrikler,Tüm seviyeleri başarıyla tamamladınız.",
                    this.canvasWidth / 2 - 230
                  );
                } else {
                  this.level++;
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

    drawMessage(color, message, LocationX) {
      this.ctx.beginPath();
      this.ctx.font = "18px Arial";
      this.ctx.fillStyle = "#FFFFFF";
      this.ctx.fillText(message, LocationX, this.canvas.height - 50);
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
</style>