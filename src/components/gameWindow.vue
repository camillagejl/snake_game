<template>
  <div id="gameWindow">

    <Menu
        v-if="screen === 'menu'"
        @playGame="this.startGame"
        @viewHighscore="this.screen = 'highscore'"
        :gameIsOver="gameIsOver"
        :points="points"
        :time="time"
    ></Menu>

    <HighscoreList
        v-if="screen === 'highscore'"
        @viewMenu="this.screen = 'menu'"
        :highScore="highScore"
    ></HighscoreList>

    <div
        class="game"
        v-if="screen === 'game'"
    >
      <div class="gameInfo">
        <Timer
            :time="time"
        ></Timer>

        <Points
            :points="points"
        ></Points>
      </div>

      <SnakePart
          v-for="(part, index) in snake"
          :key="index"
          :defaultSize="defaultSnakeSize"
          :snakeProps="part"
          :style="'position: absolute; top: ' + part.top + '; left: ' + part.left"
          class="snake_part"
          :id="'snake_part_' + index"
      ></SnakePart>

      <Apple
          :style="'position: absolute; top: ' + apple.top + '; left: ' + apple.left"
      ></Apple>
    </div>

  </div>
</template>

<script>
import SnakePart from "@/components/snakePart";
import Apple from "@/components/apple";
import Points from "@/components/points";
import Timer from "@/components/timer";
import Menu from "@/components/menu";
import HighscoreList from "@/components/highscoreList";

export default {
  name: "gameWindow",
  components: {HighscoreList, Menu, Timer, Points, Apple, SnakePart},
  data() {
    return {
      screen: 'menu',
      playerName: '',
      gameIsOver: false,
      defaultSnakeSize: 15,
      appleSize: 30,
      speed: 5,
      snakeLength: 100,
      points: 0,
      timer: false,
      time: 0,
      animation: '',
      snake: [
        {
          top: 100,
          left: 100,
          direction: 'right',
          height: 15,
          width: 100
        }
      ],
      apple: {},
      arrows: [
        {
          direction: 'up',
          keys: ['ArrowUp', 'W', 'w'],
          positionNewSnake: [
            {
              previousDirection: 'left',
              top: 'top',
              left: 'left',
            },
            {
              previousDirection: 'right',
              top: 'top',
              left: 'right'
            }
          ]
        },
        {
          direction: 'left',
          keys: ['ArrowLeft', 'A', 'a'],
          positionNewSnake: [
            {
              previousDirection: 'up',
              top: 'top',
              left: 'left'
            },
            {
              previousDirection: 'down',
              top: 'bottom',
              left: 'left'
            }
          ]
        },
        {
          direction: 'down',
          keys: ['ArrowDown', 'S', 's'],
          positionNewSnake: [
            {
              previousDirection: 'left',
              top: 'top',
              left: 'left'
            },
            {
              previousDirection: 'right',
              top: 'top',
              left: 'right'
            }
          ]
        },
        {
          direction: 'right',
          keys: ['ArrowRight', 'D', 'd'],
          positionNewSnake: [
            {
              previousDirection: 'up',
              top: 'top',
              left: 'left'
            },
            {
              previousDirection: 'down',
              top: 'bottom',
              left: 'left'
            }
          ]
        }
      ],
      highScore: [
        {
          name: 'Troels',
          points: 200
        },
        {
          name: 'Camilla',
          points: 120
        },
        {
          name: 'Bitten',
          points: 10090
        }
      ]
    }
  },
  mounted() {

    // Listening for key presses, and reacts if it's an arrow/WASD.
    document.addEventListener('keydown', (event) => {
      this.arrows.forEach(arrow => {
        arrow.keys.forEach(arrowKey => {
          if (event.key === arrowKey) {

            // Changes the direction depending on the new direction.
            this.changeDirection(arrow.direction)
          }
        })
      })

      // Set space key to pause animation for testing
      // if (event.key === ' ') {
      //   cancelAnimationFrame(this.animation);
      // }
    })

    // Creates initial apple
    this.createApple();
  },
  methods: {
    startGame(name) {
      this.screen = 'game';

      // Resets game to starting point
      this.snake = [
        {
          top: 100,
          left: 100,
          direction: 'right',
          height: 15,
          width: 100
        }
      ]
      this.timer = false;
      this.points = 0;
      this.time = 0;
      this.playerName = name
    },

    changeDirection(direction) {
      // Compares the key pressed to the arrows array.
      this.arrows.forEach(arrow => {
        if (arrow.direction === direction) {

          // Checks on the arrow whether the new direction is compatible with the previous direction.
          arrow.positionNewSnake.forEach(positionNewSnake => {
            if (this.snake[0].direction === positionNewSnake.previousDirection) {

              // The first time the snake starts moving, the timer will start
              if (!this.timer) this.startTimer();

              // Creates the new part, that will now be the front.
              this.createPart(arrow.direction, positionNewSnake);

              // Cancels the current animation, so the current/previous front of the snake won't move/grow.
              cancelAnimationFrame(this.animation)

              // Starts growing the new snake front
              this.animate();
            }
          })
        }
      })
    },

    createPart(direction, positionNewSnake) {
      // The new part will be added with the standard size (i.e. it starts out a ball)
      let newPart = {
        top: 100,
        left: 100,
        direction: direction,
        height: this.defaultSnakeSize,
        width: this.defaultSnakeSize
      };

      // Finds the absolute position of the current front of the snake, so a new part can be added to this
      // place. the gameWindow is needed to subtract the margins.
      let gameWindowPosition = document.querySelector('#gameWindow').getBoundingClientRect();
      let currentSnakePosition = document.querySelector('#snake_part_0').getBoundingClientRect();

      // The arrows array contains information about where to place the new part depending on direction.
      newPart.top = currentSnakePosition[positionNewSnake.top] - gameWindowPosition.top - 5;
      newPart.left = currentSnakePosition[positionNewSnake.left] - gameWindowPosition.left - 5;

      // If the new part is placed at the bottom or the right of the previous part, it will be places next to it
      // instead of on top of it. To avoid this, we retract the snake's default width from the position.
      if (positionNewSnake.top === 'bottom') {
        newPart.top -= this.defaultSnakeSize
      }
      if (positionNewSnake.left === 'right') {
        newPart.left -= this.defaultSnakeSize
      }

      // Adds new part to the beginning of the Snake array, so the snake currently in front will always be at [0].
      this.snake.unshift(newPart);
    },

    startTimer() {
      this.timer = window.setInterval(() => {
        this.time += 1
        console.log(this.time)
      }, 1000)
    },

    createApple() {
      // Positions apple somewhere in the game, retracting apple size so it doesn't go outside right and bottom borders.
      this.apple = {
        top: Math.floor(Math.random() * (750 - this.appleSize)),
        left: Math.floor(Math.random() * (750 - this.appleSize))
      }
    },

    animate() {
      this.animation = requestAnimationFrame(this.animate)

      // snakeEnd is the back end of the snake, last in the array..
      let snakeEnd = this.snake.length - 1;

      // If the end of the snake is back to default size, it should disappear so the next part can animate.
      if (this.snake[snakeEnd].width <= this.defaultSnakeSize && this.snake[snakeEnd].height <= this.defaultSnakeSize) {
        this.snake.pop();
      }

      // Updates snakeEnd in case the end was popped but there's still more than one part in the array.
      snakeEnd = this.snake.length - 1;

      // If there's more than one snake part, the front part should grow and the back part should shrink.
      if (this.snake.length > 1) this.moveMultipleParts(snakeEnd)

      // If there's only one snake part, it will simply move.
      else this.moveSinglePart();

      // Collision detection
      this.collisionDetection()

    },

    moveMultipleParts(snakeEnd) {
      //  Growing the front snake
      if (this.snake[0].direction === 'up' || this.snake[0].direction === 'down') {
        this.snake[0].height += this.speed
      }
      if (this.snake[0].direction === 'left' || this.snake[0].direction === 'right') {
        this.snake[0].width += this.speed
      }

      // If the direction is up or left, the snake needs to move as well, as it will otherwise grow in the wrong
      // direction.
      if (this.snake[0].direction === 'up') this.snake[0].top -= this.speed
      if (this.snake[0].direction === 'left') this.snake[0].left -= this.speed

      // Shrinking the back snake
      if (this.snake[snakeEnd].direction === 'left' || this.snake[snakeEnd].direction === 'right') {
        this.snake[snakeEnd].width -= this.speed
      }
      if (this.snake[snakeEnd].direction === 'up' || this.snake[snakeEnd].direction === 'down') {
        this.snake[snakeEnd].height -= this.speed
      }

      // If the direction is down or right, the snake needs to move as well, as it will otherwise shrink in the wrong
      // direction.
      if (this.snake[snakeEnd].direction === 'down') this.snake[snakeEnd].top += this.speed
      if (this.snake[snakeEnd].direction === 'right') this.snake[snakeEnd].left += this.speed
    },

    moveSinglePart() {

      if (this.snake[0].direction === 'up') this.snake[0].top -= this.speed
      if (this.snake[0].direction === 'left') this.snake[0].left -= this.speed
      if (this.snake[0].direction === 'down') this.snake[0].top += this.speed
      if (this.snake[0].direction === 'right') this.snake[0].left += this.speed
    },

    collisionDetection() {
      let gameWindowPosition = document.querySelector('#gameWindow').getBoundingClientRect();
      let frontSnakeArea = document.querySelector('#snake_part_0').getBoundingClientRect();

      // Define where the moving front part of the snake is, as this is what can collide.
      let snakeFront = {top: 0, left: 0};
      if (this.snake[0].direction === 'up' || this.snake[0].direction === 'left') {
        snakeFront = {top: frontSnakeArea.top, left: frontSnakeArea.left}
      }

      if (this.snake[0].direction === 'down') {
        snakeFront = {top: frontSnakeArea.top + frontSnakeArea.height, left: frontSnakeArea.left}
      }

      if (this.snake[0].direction === 'right') {
        snakeFront = {top: frontSnakeArea.top, left: frontSnakeArea.left + frontSnakeArea.width}
      }

      snakeFront = {
        top: snakeFront.top - gameWindowPosition.top - 5,
        left: snakeFront.left - gameWindowPosition.left - 5
      }

      // Detect collision with apple
      this.appleDetection(snakeFront);

      let gameOver = false;
      // Detect collision with snake
      if (this.snakeDetection(snakeFront) === 'gameOver') gameOver = true;

      // Detect collision with borders
      if (this.wallDetection(snakeFront) === 'gameOver') gameOver = true;

      if (gameOver) this.gameOver();
    },

    appleDetection(snakeFront) {
      let appleArea = {
        top: this.apple.top,
        left: this.apple.left,
        bottom: this.apple.top + this.appleSize,
        right: this.apple.left + this.appleSize,
      }

      if (snakeFront.top + this.defaultSnakeSize > appleArea.top &&
          snakeFront.top < appleArea.bottom &&
          snakeFront.left + this.defaultSnakeSize > appleArea.left &&
          snakeFront.left < appleArea.right
      ) {
        this.points = this.points + 10
        this.createApple();
      }
    },

    wallDetection(snakeFront) {
      if (snakeFront.top <= 0 || snakeFront.left <= 0 ||
          snakeFront.top >= 750 || snakeFront.left >= 750
      ) {
        return'gameOver';
      }
      return '';
    },

    snakeDetection(snakeFront) {
      this.snake.forEach(part => {
        let snakeArea = {
          top: part.top,
          left: part.left,
          bottom: part.top + part.height,
          right: part.left + part.width
        }

        if (snakeFront.top > snakeArea.top &&
            snakeFront.top < snakeArea.bottom &&
            snakeFront.left > snakeArea.left &&
            snakeFront.left < snakeArea.right
        ) {
          return 'gameOver';
        }
      })
      return '';
    },

    gameOver() {
      cancelAnimationFrame(this.animation)

      // Creates score for score board
      let score = {
        name: this.playerName,
        points: this.points
      }

      // Shows the menu, now with game over text
      this.gameIsOver = true;
      this.highScore.push(score)
      this.screen = 'menu'
      window.clearInterval(this.timer)
    }
  }
}
</script>

<style scoped>
#gameWindow {
  border: 5px solid black;
  height: 750px;
  width: 750px;
  position: relative;
}

.gameInfo {
  display: flex;
  justify-content: space-between;
  padding: 10px;
}

</style>