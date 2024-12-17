<template>
  <div class="signup-container">
    <div class="signup-card">
      <h1 class="signup-title">สร้างบัญชี</h1>
      <form @submit.prevent="handleSignUp">
        <div class="input-group">
          <label for="email">Email</label>
          <input
            type="email"
            id="email"
            v-model="email"
            placeholder="Enter your email"
            required
          />
        </div>
        <div class="input-group">
          <label for="password">Password</label>
          <input
            type="password"
            id="password"
            v-model="password"
            placeholder="Create a password"
            required
          />
        </div>
        <button class="signup-button" type="submit">Sign Up</button>
      </form>
      <p class="login-link">
        ถ้ามีบัญชีอยู๋แล้วกด <a href="/login">Login</a>
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
const email = ref('')
const password = ref('')
const { $supabase } = useNuxtApp() // เข้าถึง supabase จาก Nuxt.js context

const handleSignUp = async () => {
  const { data, error } = await $supabase.auth.signUp({
    email: email.value,
    password: password.value,
  })

  if (error) {
    alert(error.message)
  } else {
    alert('Sign Up successful! Please check your email to verify your account.')
    window.location.href = '/login'
  }
}
</script>

<style scoped>
.signup-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: linear-gradient(135deg, #20529c, #75caff);
  color: #fff;
}

.signup-card {
  background: #fff;
  color: #333;
  padding: 2rem;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  max-width: 400px;
  width: 100%;
}

.signup-title {
  text-align: center;
  font-size: 1.5rem;
  margin-bottom: 1rem;
}

.input-group {
  margin-bottom: 1rem;
}

.input-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: bold;
}

.input-group input {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.signup-button {
  width: 100%;
  padding: 0.75rem;
  background: linear-gradient(135deg, #20529c, #75caff);
  color: #fff;
  border: none;
  border-radius: 5px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.3s ease;
}

.signup-button:hover {
  background: linear-gradient(135deg, #1d4e8b, #63b3ed);
}

.login-link {
  text-align: center;
  margin-top: 1rem;
}

.login-link a {
  color: #20529c;
  font-weight: bold;
}
</style>
