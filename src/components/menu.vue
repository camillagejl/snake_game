<template>
  <div class="menuContainer">
    <div class="menu">

      <div v-if="gameIsOver">Game over! You got {{ points }} points in {{ time }} seconds.</div>

      <label for="name">Your name</label>
      <input type="text" id="name" name="name" required v-model="playerName">

      <button @click="this.$emit('playGame', playerName)">Play game!</button>

      <button @click="this.$emit('viewHighscore')">View highscore</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "menu",
  data() {
    return {
      playerName: 'Player'
    }
  },
  props: {
    gameIsOver: Boolean,
    points: Number,
    time: Number
  }
}
</script>

<style scoped>
.menuContainer {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.menu {
  max-width: 400px;
  display: flex;
  flex-direction: column;
}
</style>