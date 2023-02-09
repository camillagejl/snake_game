<template>
  <svg
      fill="darkred"
      class="apple"
      :width="appleSize"
      :height="appleSize"
  >
    <rect
        :width="appleSize"
        :height="appleSize"
        :rx="appleSize/2"
    />
  </svg>
</template>

<script>
export default {
  name: "apple",
  data() {
    return {
      appleSize: 30
    }
  }
}
</script>

<style scoped>

</style>