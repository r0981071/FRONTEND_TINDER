<template>
  <nav class="main-nav">
    <ul>
      <li
        v-for="(item, i) in items"
        :key="i"
        @click="$emit('setActivePage', item)"
      >
        {{ item }}
      </li>
    </ul>
  </nav>
</template>

<script>
export default {
  name: "NavigationComponent",
  data() {
    return { items: ["swipe", "profile", "matches"] };
  },
};
</script>

<style scoped>
.main-nav {
  background: linear-gradient(90deg, #ffb74d, #fca04b);
  padding: 0.8rem 2rem;
  border-radius: 0 0 12px 12px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.08);
}
.main-nav ul {
  display: flex;
  list-style: none;
  gap: 24px;
  justify-content: center;
  margin: 0;
  padding: 0;
}
.main-nav li {
  cursor: pointer;
  font-weight: 600;
  color: white;
  transition: all 0.2s ease;
  text-transform: capitalize;
}
.main-nav li:hover {
  color: #fff3e0;
  transform: scale(1.05);
}
</style>


