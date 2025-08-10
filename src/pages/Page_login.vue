<template>
  <div>
    <h2>Login</h2>
    <form @submit.prevent="submit">
      <input v-model="username" placeholder="username" required />
      <input v-model="password" type="password" placeholder="password" required />
      <button type="submit">Log in</button>
    </form>
    <p><a href="#" @click.prevent="$emit('setActivePage','signup')">No account? Sign up</a></p>
    <p v-if="error" style="color:red">{{ error }}</p>
  </div>
</template>

<script>
export default {
  name: "Page_login",
  data(){ return { username:"", password:"", error:"" }; },
  methods:{
    async submit(){
      this.error = "";
      try{
        const res = await fetch("http://localhost:3000/api/users/login", {
          method:"POST",
          headers:{ "Content-Type":"application/json" },
          body: JSON.stringify({ username:this.username, password:this.password })
        });
        if(!res.ok) throw new Error("invalid credentials");
        const user = await res.json(); // expect {user_id, username}
        this.$emit("loginSuccess", user);
      }catch(e){ this.error = e.message; }
    }
  }
};
</script>
