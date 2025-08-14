<template>
  <nav class="auth-nav">
    <ul>
      <li v-if="!isLoggedIn" @click="go('login')">log in</li>
      <li v-if="!isLoggedIn" @click="go('signup')">sign up</li>
      <li v-if="isLoggedIn" @click="go('logout')">log out</li>
    </ul>
  </nav>
</template>

<script>
export default {
  name: "NavigationAuth",
  props: { isLoggedIn: { type: Boolean, required: true } },
  methods: {
    go(page) {
      this.$emit("setActivePage", page);
    },
  },
};
</script>

<style scoped>
.auth-nav {
  position: absolute;
  top: 0;
  right: 0;
  padding: 0.8rem 1rem;
}

.auth-nav ul {
  display: flex;
  list-style: none;
  gap: 16px;
  margin: 0;
  padding: 0;
}

.auth-nav li {
  cursor: pointer;
  font-weight: 500;
  color: #8a4b00; /* dark orange-brown text */
  padding: 6px 12px;
  border-radius: 6px;
  transition: all 0.2s ease;
  background: #ffecb3; /* light yellow background */
}

.auth-nav li:hover {
  background: #ffe082; /* slightly darker yellow */
  color: #5d3700;
}
</style>

