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
  </div>
</template>

<script>
import SnakePart from "@/components/snakePart";

export default {
  name: "gameWindow",
  components: {SnakePart},
  data() {
    return {
      defaultSnakeSize: 15,
      snake: [
        {
          top: 100,
          left: 100,
          direction: 'right',
          height: 15,
          width: 100
        }
      ],
      direction: '',
      arrows: [
        {
          direction: 'up',
          keys: ['ArrowUp', 'W', 'w'],
          effectAtDirection: ['left', 'right']
        },
        {
          direction: 'left',
          keys: ['ArrowLeft', 'A', 'a'],
          effectAtDirection: ['up', 'down']
        },
        {
          direction: 'down',
          keys: ['ArrowDown', 'S', 's'],
          effectAtDirection: ['left', 'right']
        },
        {
          direction: 'right',
          keys: ['ArrowRight', 'D', 'd'],
          effectAtDirection: ['up', 'down']
        }
      ]
    }
  },
  mounted() {
    document.addEventListener('keydown', (event) => {
      this.arrows.forEach(arrow => {
        arrow.keys.forEach(arrowKey => {
          if (event.key === arrowKey) {
            this.direction = arrow.direction
            console.log('Direction:', this.direction)
          }
        })
      })
    })
    this.createPart();
  },
  methods: {
    createPart() {
      console.log('creating part');
    },
    onArrowDown() {
      console.log('changing direction');
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
</style>