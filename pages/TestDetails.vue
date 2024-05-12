<template>
  <div>
    <h1>Test Details</h1>
    <p>Test ID: {{ testId }}</p>
  </div>
</template>

<script>
export default {
  computed: {
    testId() {
      return this.$route.params.testId;
    }
  },
  mounted() {
    console.log("Test ID:", this.testId);  // This will now fetch from the URL directly
  }
}
</script>

<style scoped>
/* Your styles here */
</style>
