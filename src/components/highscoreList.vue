<template>
  <div class="scoreboard">
  Highscore

  <div
      v-for="(score, index) in sortedScores"
      :key="index"
      class="score"
  >
    <span>
    {{ index + 1 }}. Name: {{ score.name }} <br>
      </span>
    <span>
    Score: {{ score.points }}
      </span>
  </div>

  <button @click="this.$emit('viewMenu')">Back to menu</button></div>

</template>

<script>
export default {
  name: "highscoreList",
  props: {
    highScore: Array
  },
  computed: {
    sortedScores() {
      let highScore = this.highScore;
      return highScore.sort((a, b) => b.points - a.points).slice(0, 10)
    }
  }
}
</script>

<style scoped>

.scoreboard {
  max-width: 400px;
  margin: 50px auto;
}

.score {
  padding: 10px 0;
  display: grid;
  grid-template-columns: 1fr 1fr;
}

</style>