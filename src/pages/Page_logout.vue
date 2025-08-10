<template>
  <div>
    <h2>Logging out…</h2>
  </div>
</template>

<script>
export default {
  name:"Page_logout",
  mounted(){
    this.$emit("logOut");
    this.$emit("setActivePage","login");
  }
};
</script>
