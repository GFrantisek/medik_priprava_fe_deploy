<template>
  <div class="register-container">
    <div class="help">
      <h2>Create Your Account</h2>
    </div>
    <form @submit.prevent="submitForm" class="register-form">
      <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" v-model="register.username" placeholder="Enter your username" />
      </div>
      <div class="form-group">
        <label for="email">Email</label>
        <input type="text" id="email" v-model="register.email" placeholder="Enter your email" />
      </div>
      <div class="form-group">
        <label for="password">Password</label>
        <input type="password" id="password" v-model="register.password" placeholder="Create a password" />
      </div>
      <div class="form-group">
        <label for="confirm-password">Confirm Password</label>
        <input type="password" id="confirm-password" v-model="register.password2" placeholder="Confirm your password" />
      </div>
      <div class="form-actions">
        <button type="submit" class="submit-btn">Register</button>
        <button @click="navigateToLogin" type="button" class="back-btn">Back to Login</button>
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
        let response = await this.$axios.$post('auth/register', this.register);
        this.$router.replace({name: 'auth-login'});
      } catch (err) {
        console.error("Registration failed:", err);
      }
    },
    navigateToLogin() {
      this.$router.push({name: 'auth-login'});
    }
  }
}
</script>

<style scoped>
.register-container {
  max-width: 400px;
  margin: 50px auto;
  padding: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  background-color: #fff;
}

.register-form h2 {
  text-align: center;
  color: #333;
  font-size: 24px;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  color: #666;
  margin-bottom: 5px;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-actions {
  text-align: center;
  margin-top: 20px;
}

.submit-btn, .back-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  color: #fff;
  background-color: #5C67F2;
  cursor: pointer;
  margin-right: 10px;
}

.back-btn {
  background-color: #f76c6c; /* Different color to distinguish from the submit button */
}

.submit-btn:hover {
  background-color: #4a54e1;
}

.back-btn:hover {
  background-color: #f55555;
}

.help{
  display: flex;
  flex-direction: column;
  align-items: center; /* Centers content horizontally */

  padding-bottom: 10px;
}
</style>
