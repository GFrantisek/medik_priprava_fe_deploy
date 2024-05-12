<template>
  <div>
    <form @submit.prevent="submitForm">
      <div>
        <label>Username</label>
        <input type="text" v-model="register.username" placeholder="Enter your username" />
      </div>
      <div>
        <label>Email</label>
        <input type="text" v-model="register.email" placeholder="Enter your email" />
      </div>
      <div>
        <label>Password</label>
        <input type="password" v-model="register.password" placeholder="Create a password" />
      </div>
      <div>
        <label>Confirm Password</label>
        <input type="password" v-model="register.password2" placeholder="Confirm your password" />
      </div>
      <div>
        <button type="submit">Submit</button>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  name: "register",

  middleware: ['auth'],
  auth: 'guest',

  data() {
    return {
      register: {
        username: '',
        email: '',
        password: '',
        password2: ''
      }
    }
  },
  methods: {
    async submitForm() {
      try {
        let response = await this.$axios.$post('auth/register', this.register)
        await this.$router.replace({name: 'auth-login'})
      } catch (err) {
        console.log(err)
      }
    }
  }
}
</script>

<style scoped>
div {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh; /* Full height of the viewport */
  background-color: #f4f3f8; /* Light purple background */
}

form {
  padding: 20px;
  border-radius: 10px;
  background: white;
  box-shadow: 0 4px 8px rgba(117, 117, 163, 0.25); /* Subtle shadow */
  width: 300px; /* Fixed width */
}

label {
  margin-bottom: 5px;
  font-size: 1rem;
  color: #5c5470; /* Darker purple for text */
}

input[type="text"],
input[type="password"] {
  padding: 10px;
  margin-bottom: 15px;
  border: 1px solid #dcdce6; /* Light purple border */
  border-radius: 5px;
  width: 100%; /* Full width */
  box-sizing: border-box; /* Includes padding and border in the element's width */
}

button {
  background-color: #6a5acd; /* Slightly vibrant purple */
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #5a4ebc; /* Darker purple on hover */
}
</style>
