<template>
    <GameWindow></GameWindow>
</template>

<script>

import GameWindow from "@/components/gameWindow";

export default {
  name: 'App',
  components: {GameWindow},
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
