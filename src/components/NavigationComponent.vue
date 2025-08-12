<!-- <template>
  <nav style="display:flex; justify-content:center;">
    <ul style="display:flex; list-style:none; gap:20px;">
      <li v-for="(item, i) in items" :key="i"
          style="cursor:pointer;"
          @click="$emit('setActivePage', item)">
        {{ item }}
      </li>
    </ul>
  </nav>
</template>

<script>
export default {
  name: "NavigationComponent",
  data() {
    return { items: ["swipe", "profile", "messages"] };//change messages to matches
  },
};
</script> -->

<template>
  <nav style="display:flex; justify-content:center;">
    <ul style="display:flex; list-style:none; gap:20px;">
      <li v-for="(item, i) in items" :key="i"
          style="cursor:pointer;"
          @click="$emit('setActivePage', item)">
        {{ item }}
      </li>
    </ul>
  </nav>
</template>

<script>
export default {
  name: "NavigationComponent",
  data() {
    return { items: ["swipe", "profile", "matches"] }; // <- changed "messages" -> "matches"
  },
};
</script>

