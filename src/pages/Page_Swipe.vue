<template>
  <div>
    <h2>Swipe</h2>

    <div v-if="current" style="border:1px solid #ccc; padding:1rem; max-width:420px; margin:0 auto;">
      <img :src="current.profile_pic_url" alt="" style="max-width:100%; height:auto;" />
      <h3>{{ current.name }}, {{ current.age }}</h3>
      <p>{{ current.bio }}</p>
      <div style="display:flex; gap:12px; justify-content:center; margin-top:8px;">
        <button @click="swipe(false)">❌</button>
        <button @click="swipe(true)">❤️</button>
      </div>
    </div>

    <p v-else>No more profiles.</p>
  </div>
</template>

<script>
export default {
  name:"Page_Swipe",
  props:{ user:{ type:Object, required:true } },
  data(){ return { queue:[], index:0, loading:false }; },
  computed:{ current(){ return this.queue[this.index] || null; } },
  methods:{
    async load(){
      this.loading = true;
      const res = await fetch(`http://localhost:3000/api/profiles/for-swipe/${this.user.user_id}`);
      this.queue = await res.json();
      this.index = 0;
      this.loading = false;
    },
    async swipe(is_like){
      if(!this.current) return;

      const payload = {
        swiper_id: this.user.user_id,
        swiped_id: this.current.profile_id,
        is_like
      };
      await fetch("http://localhost:3000/api/swipes",{
        method:"POST",
        headers:{ "Content-Type":"application/json" },
        body: JSON.stringify(payload)
      });
      this.index++;
      if(this.index >= this.queue.length) this.load();
    }
  },
  mounted(){ this.load(); }
};
</script>
