<template>
  <button @click="this.$emit('playGame')">Play game!</button>
  <button @click="this.$emit('viewHighscore')">View highscore</button>
</template>

<script>
export default {
  name: "menu"
}
</script>

<style scoped>

</style>