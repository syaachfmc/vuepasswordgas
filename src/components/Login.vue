
<script setup>
import { ref } from "vue";
import { login } from "../api/auth.js";

const emit = defineEmits(["login-success"]);

const username = ref("");
const password = ref("");
const loading = ref(false);
const errorMessage = ref("");

async function handleLogin() {
  errorMessage.value = "";
  loading.value = true;

  try {
    const result = await login(
      username.value,
      password.value
    );

    if (!result.success) {
      errorMessage.value = result.message;
      return;
    }

    sessionStorage.setItem("authToken", result.token);

    emit("login-success", {
      token: result.token,
      user: result.user
    });
  } catch (error) {
    console.error(error);
    errorMessage.value = "Gagal menghubungi server.";
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <main class="container">
    <section class="card">
      <h1>Login</h1>
      <p>Masukkan username dan password.</p>

      <form @submit.prevent="handleLogin">
        <label for="username">Username</label>
        <input
          id="username"
          v-model.trim="username"
          autocomplete="username"
          required
        />

        <label for="password">Password</label>
        <input
          id="password"
          v-model="password"
          type="password"
          autocomplete="current-password"
          required
        />

        <p v-if="errorMessage" class="error">
          {{ errorMessage }}
        </p>

        <button type="submit" :disabled="loading">
          {{ loading ? "Memproses..." : "Login" }}
        </button>
      </form>
    </section>
  </main>
</template>

<style scoped>
.container {
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 20px;
  background: #f3f4f6;
  font-family: Arial, sans-serif;
}

.card {
  width: 100%;
  max-width: 400px;
  padding: 28px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 18px #00000012;
}

h1 {
  margin-top: 0;
}

label {
  display: block;
  margin: 16px 0 6px;
}

input {
  box-sizing: border-box;
  width: 100%;
  padding: 11px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 16px;
}

button {
  width: 100%;
  margin-top: 20px;
  padding: 12px;
  border: 0;
  border-radius: 6px;
  background: #2563eb;
  color: white;
  font-size: 16px;
  cursor: pointer;
}

button:disabled {
  opacity: 0.6;
  cursor: wait;
}

.error {
  color: #dc2626;
}
</style>