<template>
  <div>
    <h2>Your Profile</h2>
    <form @submit.prevent="save" style="max-width:520px; margin:0 auto; text-align:left;">
      <label>Name</label>
      <input v-model="form.name" required style="width:100%;" />

      <label>Age</label>
      <input v-model.number="form.age" type="number" required style="width:100%;" />

      <label>Nationality</label>
      <input v-model="form.nationallity" style="width:100%;" />

      <label>Bio</label>
      <textarea v-model="form.bio" style="width:100%;"></textarea>

      <label>Photo URL</label>
      <input v-model="form.profile_pic_url" style="width:100%;" />

      <div style="margin-top:10px;">
        <button type="submit">Save</button>
      </div>
    </form>
    <p v-if="saved" style="color:green;">Saved!</p>
  </div>
</template>

<script>
export default {
  name:"Page_Profile",
  props:{ user:{ type:Object, required:true } },
  data(){
    return {
      saved:false,
      form:{
        userId: this.user.user_id,
        name:"",
        gender_male:true,
        age:18,
        nationallity:"",
        height:1.7,
        weight:70,
        bio:"",
        profile_pic_url:"",
      }
    };
  },
  async mounted(){
    const res = await fetch(`http://localhost:3000/api/profiles/${this.user.user_id}`);
    if(res.ok){
      const p = await res.json();
      Object.assign(this.form, { ...p, userId: this.user.user_id });
    }
  },
  methods:{
    async save(){
      this.saved = false;
      const res = await fetch("http://localhost:3000/api/profiles",{
        method:"POST",
        headers:{ "Content-Type":"application/json" },
        body: JSON.stringify(this.form)
      });
      if(res.ok) this.saved = true;
    }
  }
};
</script>
