<template>
  <div>
    <h1>Sign Up</h1>
    <form @submit.prevent="register">
      <input v-model="user.email" placeholder="Email" type="email" required>
      <input v-model="user.name" placeholder="Name" type="text" required>
      <input v-model="user.password" placeholder="Password" type="password" required>
      <button type="submit">Register</button>
    </form>
  </div>
</template>

<script>
export default {
  name: "sad",
  data() {
    return {
      user: {
        email: '',
        name: '',
        password: '',
      },
      message: ''
    };
  },
  methods: {
    async register() {
      try {
        console.log("hi")
        console.log(this.user.password)

        const response = await this.$axios.post('http://127.0.0.1:8081/signup', this.user);  // Assuming baseURL is set
        this.message = response.data.message;
        console.log("Registration successful:", response.data);
      } catch (error) {
        console.error("Error occurred:", error);
        if (error.response) {
          // The request was made and the server responded with a status code
          // that falls out of the range of 2xx
          this.message = error.response.data.message || "Server error";
          console.log(error.response.status);
          console.log(error.response.headers);
        } else if (error.request) {
          // The request was made but no response was received
          this.message = "No response from server";
          console.log(error.request);
        } else {
          // Something happened in setting up the request that triggered an Error
          this.message = "Error in sending request";
          console.log('Error', error.message);
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
