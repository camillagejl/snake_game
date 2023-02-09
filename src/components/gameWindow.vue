<template>
  <div id="gameWindow">
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
</template>

<script>
import SnakePart from "@/components/snakePart";
import Apple from "@/components/apple";

export default {
  name: "gameWindow",
  components: {Apple, SnakePart},
  data() {
    return {
      defaultSnakeSize: 15,
      appleSize: 30,
      snakeLength: 100,
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
      ]
    }
  },
  mounted() {
    // Listening for key presses, and reacts if it's an arrow/WASD.
    document.addEventListener('keydown', (event) => {
      if (event.key === ' ') {
        cancelAnimationFrame(this.animation);
      }

      this.arrows.forEach(arrow => {
        arrow.keys.forEach(arrowKey => {
          if (event.key === arrowKey) {
            this.changeDirection(arrow.direction)
          }
        })
      })
    })
    this.createPart();
    this.createApple();
  },
  methods: {
    createPart(direction) {

      // The new part will be added with the standard size (i.e. it starts out a ball)
      let newPart = {
        top: 100,
        left: 100,
        direction: direction,
        height: this.defaultSnakeSize,
        width: this.defaultSnakeSize
      };

      // Finds the right arrow/direction, then compares to the previous direction of the snake to see if it should be
      // able to mov in the new direction.
      this.arrows.forEach(arrow => {
        if (arrow.direction === direction)
          arrow.positionNewSnake.forEach(positionNewSnake => {
            // Checks if the old direction is in the array of the new direction's 'previous directions', where it
            // should work.
            if (this.snake[0].direction === positionNewSnake.previousDirection) {

              // Finds the absolute position of the current front of the snake, so a new part can be added to this
              // place.
              let gameWindowPosition = document.querySelector('#gameWindow').getBoundingClientRect();
              let currenSnakePosition = document.querySelector('#snake_part_0').getBoundingClientRect();

              // The arrows array contains information about where to place the new part depending on direction.
              newPart.top = currenSnakePosition[positionNewSnake.top] - gameWindowPosition.top - 5;
              newPart.left = currenSnakePosition[positionNewSnake.left] - gameWindowPosition.left - 5;

              // If the new part is placed at the bottom or the right of the previous part, it will be places next to it
              // instead of on top of it. To avoid this, we retract the snake's default width from the position.
              if (positionNewSnake.top === 'bottom') {
                newPart.top = newPart.top - this.defaultSnakeSize
              }
              if (positionNewSnake.left === 'right') {
                newPart.left = newPart.left - this.defaultSnakeSize
              }

              // Adds new part to the beginning of the Snake array, so the current nake in front will always be at [0].
              this.snake.unshift(newPart);

              // Cancels the current animation, so the current front of the snake won't move/grow.
              cancelAnimationFrame(this.animation)

              // Starts growing the new snake front
              this.animate();
            }
          })
      })
    },
    createApple() {
      this.apple = {
        top: Math.floor(Math.random() * 720),
        left: Math.floor(Math.random() * 720)
      }


      let appleArea = document.querySelector('.apple').getBoundingClientRect();
      console.log(appleArea)

    },
    changeDirection(direction) {
      this.createPart(direction)
    },
    animate() {
      this.animation = requestAnimationFrame(this.animate)
      const snakeEnd = this.snake.length - 1;

      if (this.snake[snakeEnd].width <= 15 && this.snake[snakeEnd].height <= 15) {
        this.snake.pop();
      }

      // If there's more than one snake part, the front part should grow and the back part should shrink.
      if (this.snake.length > 1) {
        //  Growing the front snake
        if (this.snake[0].direction === 'up' || this.snake[0].direction === 'down') {
          this.snake[0].height = this.snake[0].height + 1
        }
        if (this.snake[0].direction === 'left' || this.snake[0].direction === 'right') {
          this.snake[0].width = this.snake[0].width + 1
        }

        // If the direction is up or left, the snake needs to move as well, as it will otherwise grow in the wrong
        // direction.
        if (this.snake[0].direction === 'up') this.snake[0].top = this.snake[0].top - 1
        if (this.snake[0].direction === 'left') this.snake[0].left = this.snake[0].left - 1

        // Shrinking the back snake
        if (this.snake[snakeEnd].direction === 'left' || this.snake[snakeEnd].direction === 'right') {
          this.snake[snakeEnd].width = this.snake[snakeEnd].width - 1
        }
        if (this.snake[snakeEnd].direction === 'up' || this.snake[snakeEnd].direction === 'down') {
          this.snake[snakeEnd].height = this.snake[snakeEnd].height - 1
        }

        // If the direction is down or right, the snake needs to move as well, as it will otherwise shrink in the wrong
        // direction.
        if (this.snake[snakeEnd].direction === 'down') this.snake[snakeEnd].top = this.snake[snakeEnd].top + 1
        if (this.snake[snakeEnd].direction === 'right') this.snake[snakeEnd].left = this.snake[snakeEnd].left + 1

      }

      // If there's only one snake part, it will simply move.
      else {
        if (this.snake[0].direction === 'up') this.snake[0].top = this.snake[0].top - 1
        if (this.snake[0].direction === 'left') this.snake[0].left = this.snake[0].left - 1
        if (this.snake[0].direction === 'down') this.snake[0].top = this.snake[0].top + 1
        if (this.snake[0].direction === 'right') this.snake[0].left = this.snake[0].left + 1
      }

      let gameWindowPosition = document.querySelector('#gameWindow').getBoundingClientRect();

      // let snakeFront = '';
      // if (this.snake[0].direction === 'up') snakeFront = 'top';
      // if (this.snake[0].direction === 'left') snakeFront = 'left';
      // if (this.snake[0].direction === 'down') snakeFront = 'bottom';
      // if (this.snake[0].direction === 'right') snakeFront = 'right';

      let frontSnakeArea = document.querySelector('#snake_part_0').getBoundingClientRect();

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

      let appleArea = {
        top: this.apple.top,
        left: this.apple.left,
        right: this.apple.left + this.appleSize,
        bottom: this.apple.top + this.appleSize
      }

      console.log(
          [
            snakeFront.top + this.defaultSnakeSize > appleArea.top,
            snakeFront.top < appleArea.bottom,
            snakeFront.left + this.defaultSnakeSize > appleArea.left,
            snakeFront.left > appleArea.right
          ]
      )

      if (snakeFront.top + this.defaultSnakeSize > appleArea.top &&
          snakeFront.top < appleArea.bottom &&
          snakeFront.left + this.defaultSnakeSize > appleArea.left
          && snakeFront.left < appleArea.right) {
        console.log("pplee!!")
      }
    },
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
</style>