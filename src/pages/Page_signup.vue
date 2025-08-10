<template>
  <div>
    <h2>Sign up</h2>
    <form @submit.prevent="submit">
      <input v-model="username" placeholder="username" required />
      <input v-model="password" type="password" placeholder="password" required />
      <button type="submit">Create account</button>
    </form>
    <p><a href="#" @click.prevent="$emit('setActivePage','login')">Have an account? Log in</a></p>
    <p v-if="error" style="color:red">{{ error }}</p>
  </div>
</template>

<script>
export default {
  name:"Page_signup",
  data(){ return { username:"", password:"", error:"" }; },
  methods:{
    async submit(){
      this.error = "";
      try{
        const res = await fetch("http://localhost:3000/api/users", {
          method:"POST",
          headers:{ "Content-Type":"application/json" },
          body: JSON.stringify({ username:this.username, password:this.password })
        });
        if(!res.ok) throw new Error("username taken?");
        const user = await res.json(); // {user_id, username}
        this.$emit("loginSuccess", user);
      }catch(e){ this.error = e.message; }
    }
  }
};
</script>
