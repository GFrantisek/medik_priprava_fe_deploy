<template>
  <div class="login-container">
    <h2>Login to Your Account</h2>
    <form @submit.prevent="submitForm" class="login-form">
      <div class="form-group">
        <label for="email">Email</label>
        <input type="text" id="email" v-model="login.email" placeholder="Enter your email" />
      </div>
      <div class="form-group">
        <label for="password">Password</label>
        <input type="password" id="password" v-model="login.password" placeholder="Enter your password" />
      </div>
      <div class="form-actions">
        <button type="submit" class="submit-btn">Login</button>
        <button @click="navigateToRegister" type="button" class="register-btn">Register</button>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  name: "login",
  middleware: ['auth'],
  auth: 'guest',

  data() {
    return {
      login: {
        email: '',
        password: ''
      }
    }
  },
  methods: {
    async submitForm() {
      try {
        let response = await this.$auth.loginWith('local', {data: this.login})
        this.$router.replace({name: 'auth-user'})
      } catch (err) {
        console.error("Login failed:", err);
      }
    },
    navigateToRegister() {
      this.$router.push({name: 'auth-register'});
    }
  }
}
</script>

<style scoped>
.login-container {
  max-width: 400px;
  margin: 50px auto;
  padding: 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  background-color: #fff;
}

.login-form h2 {
  text-align: center;
  color: #333;
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

.submit-btn, .register-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  color: #fff;
  cursor: pointer;
  margin-right: 10px;
}

.submit-btn {
  background-color: #5C67F2;
}

.register-btn {
  background-color: #f76c6c;
}

.submit-btn:hover {
  background-color: #4a54e1;
}

.register-btn:hover {
  background-color: #f55555;
}
</style>
