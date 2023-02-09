<template>
  <svg
      class="snake"
      :width="snakeProps.width"
      :height="snakeProps.height"
  >
    <rect
        :width="snakeProps.width"
        :height="snakeProps.height"
        :rx="defaultSize/1.3"
    />
  </svg>
</template>

<script>
export default {
  name: "snakePart",
  props: {
    defaultSize: Number,
    snakeProps: Object
  }
}
</script>

<style scoped>

</style>