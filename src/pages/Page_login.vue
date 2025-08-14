<template>
  <div class="auth-wrap">
    <div class="card">
      <h2 class="title">Welcome back</h2>

      <form class="form" @submit.prevent="submit">
        <label class="label">Username</label>
        <input v-model="username" placeholder="username" required />

        <label class="label">Password</label>
        <input v-model="password" type="password" placeholder="password" required />

        <button type="submit" class="btn">Log in</button>
      </form>

      <p class="foot">
        No account?
        <a href="#" @click.prevent="$emit('setActivePage','signup')">Sign up</a>
      </p>

      <p v-if="error" class="error">{{ error }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "Page_login",
  data() {
    return { username: "", password: "", error: "" };
  },
  methods: {
    async submit() {
      this.error = "";
      try {
        const res = await fetch("http://localhost:3000/api/users/login", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ username: this.username, password: this.password }),
        });
        if (!res.ok) throw new Error("Invalid credentials");
        const user = await res.json(); // { user_id, username }
        this.$emit("loginSuccess", user);
      } catch (e) {
        this.error = e.message || "Login failed";
      }
    },
  },
};
</script>

<style scoped>
.auth-wrap {
  min-height: calc(100vh - 80px);
  display: grid;
  place-items: start center;
  padding-top: 40px;
  background: linear-gradient(180deg, #fffaf3 0%, #fff3e0 100%);
}

.card {
  width: min(400px, 92vw);
  margin: 0 auto;
  padding: 28px 24px 22px;
  border: 1px solid #f1e3c9;
  border-radius: 16px;
  background: #ffffff;
  box-shadow: 0 12px 28px rgba(255, 163, 26, 0.08);
}

.title {
  margin: 0 0 14px;
  text-align: center;
}

.form {
  /* this ensures inputs and button share the same width */
  display: grid;
  gap: 10px;
  max-width: 300px;
  margin: 0 auto;
}

.label {
  font-size: 14px;
  color: #374151;
}

input {
  width: 100%;
  height: 44px;
  padding: 0 12px;
  border: 1px solid #e5e7eb;
  border-radius: 10px;
  outline: none;
  box-sizing: border-box;
}

input:focus {
  border-color: #f59e0b;
  box-shadow: 0 0 0 3px rgba(245, 158, 11, 0.15);
}

.btn {
  width: 100%;
  height: 46px;
  border: 0;
  border-radius: 12px;
  cursor: pointer;
  color: #fff;
  font-weight: 700;
  letter-spacing: 0.2px;
  background: linear-gradient(90deg, #f59e0b, #ffb347);
  box-shadow: 0 6px 16px rgba(245, 158, 11, 0.25);
}

.btn:hover {
  filter: brightness(0.98);
}

.foot {
  margin: 12px 0 0;
  text-align: center;
  color: #6b7280;
}

.foot a {
  color: #d97706;
  font-weight: 600;
  text-decoration: none;
}

.foot a:hover {
  text-decoration: underline;
}

.error {
  margin-top: 8px;
  text-align: center;
  color: #dc2626;
}
</style>
