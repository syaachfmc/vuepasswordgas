<script setup>
import { ref, onMounted } from "vue";
import Login from "./components/Login.vue";
import { validateSession, logout } from "./api/auth.js";

const token = ref(sessionStorage.getItem("authToken") || "");
const user = ref(null);
const checkingSession = ref(true);
const loading = ref(false);
const errorMessage = ref("");

function handleLoginSuccess(data) {
  token.value = data.token;
  user.value = data.user;
}

async function handleLogout() {
  loading.value = true;

  try {
    if (token.value) {
      await logout(token.value);
    }
  } catch (error) {
    console.error("Logout server gagal:", error);
  } finally {
    sessionStorage.removeItem("authToken");
    token.value = "";
    user.value = null;
    loading.value = false;
  }
}

onMounted(async () => {
  if (!token.value) {
    checkingSession.value = false;
    return;
  }

  try {
    const result = await validateSession(token.value);

    if (result.success) {
      user.value = result.user;
    } else {
      sessionStorage.removeItem("authToken");
      token.value = "";
    }
  } catch (error) {
    errorMessage.value = "Sesi belum bisa diverifikasi.";
    console.error(error);
  } finally {
    checkingSession.value = false;
  }
});
</script>

<template>
  <p v-if="checkingSession">Memeriksa sesi...</p>

  <Login v-else-if="!user" @login-success="handleLoginSuccess" />

  <main v-else class="dashboard">
    <h1>Dashboard</h1>
    <p>Login berhasil.</p>
    <p><strong>Nama:</strong> {{ user.name }}</p>
    <p><strong>Username:</strong> {{ user.username }}</p>
    <p><strong>Role:</strong> {{ user.role }}</p>

    <p v-if="errorMessage">{{ errorMessage }}</p>

    <button @click="handleLogout" :disabled="loading">
      {{ loading ? "Memproses..." : "Logout" }}
    </button>
  </main>
</template>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  color: #1f2937;
}

.dashboard {
  padding: 28px;
}

button {
  padding: 12px 20px;
  border: 0;
  border-radius: 6px;
  background: #2563eb;
  color: white;
  cursor: pointer;
}
</style>