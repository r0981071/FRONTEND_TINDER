<template>
  <nav style="position:absolute; top:0; right:0; padding:1rem;">
    <ul style="display:flex; list-style:none; gap:20px;">
      <li v-if="!isLoggedIn" @click="go('login')" style="cursor:pointer;">log in</li>
      <li v-if="!isLoggedIn" @click="go('signup')" style="cursor:pointer;">sign up</li>
      <li v-if="isLoggedIn"  @click="go('logout')" style="cursor:pointer;">log out</li>
    </ul>
  </nav>
</template>

<script>
export default {
  name: "NavigationAuth",
  props: { isLoggedIn: { type: Boolean, required: true } },
  methods: { go(page){ this.$emit("setActivePage", page); } }
};
</script>
