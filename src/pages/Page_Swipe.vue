<template>
  <div>
    <h2>Swipe</h2>

    <div v-if="current" style="border:1px solid #ccc; padding:1rem; max-width:480px; margin:0 auto;">
      <!-- Photo -->
      <img :src="current.profile_pic_url" alt="" style="max-width:100%; height:auto;" />

      <!-- Name -->
      <h3 style="margin-top:12px;">{{ current.name }}</h3>

      <!-- Age + Gender + Nationality -->
      <p style="margin:4px 0;">
        <strong>
          {{ current.age }}<span v-if="current.age">, </span>
          {{ genderLabel(current.gender_male) }}
          <span v-if="current.nationallity">, from {{ current.nationallity }}</span>
        </strong>
      </p>


      <!-- Height + Weight -->
      <p style="margin:4px 0;">
        <span v-if="current.height !== undefined && current.height !== null">
          {{ fmtHeight(current.height) }}
        </span>
        <span v-if="current.height !== undefined && current.height !== null && current.weight !== undefined && current.weight !== null"> · </span>
        <span v-if="current.weight !== undefined && current.weight !== null">
          {{ fmtWeight(current.weight) }}
        </span>
      </p>

      <!-- Bio -->
      <p style="margin-top:6px;">
        <span>{{ current.bio }}</span>
      </p>

      <!-- Actions -->
      <div style="display:flex; gap:12px; justify-content:center; margin-top:10px;">
        <button @click="swipe(false)">❌</button>
        <button @click="swipe(true)">❤️</button>
      </div>
    </div>

    <p v-else>No more profiles.</p>
  </div>
</template>

<script>
export default {
  name: "Page_Swipe",
  props: { user: { type: Object, required: true } },
  data() {
    return { queue: [], index: 0, loading: false };
  },
  computed: {
    current() {
      return this.queue[this.index] || null;
    },
  },
  methods: {
    genderLabel(val) {
      // DB stores boolean; handle 0/1 just in case
      const b = val === true || val === 1 || val === "true";
      return b ? "Male" : "Female";
    },
    fmtAge(a) {
      return a != null ? `${a} years` : "";
    },
    fmtHeight(h) {
      // meters with 2 decimals
      const n = Number(h);
      return Number.isFinite(n) ? `${n.toFixed(2)} m` : "";
    },
    fmtWeight(w) {
      // kg with 1 decimal
      const n = Number(w);
      return Number.isFinite(n) ? `${n.toFixed(1)} kg` : "";
    },

    async load() {
      this.loading = true;
      const res = await fetch(`http://localhost:3000/api/profiles/for-swipe/${this.user.user_id}`);
      this.queue = await res.json();
      this.index = 0;
      this.loading = false;
    },
    async swipe(is_like) {
      if (!this.current) return;
      const payload = {
        swiper_id: this.user.user_id,
        swiped_id: this.current.profile_id,
        is_like,
      };
      await fetch("http://localhost:3000/api/swipes", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload),
      });
      this.index++;
      if (this.index >= this.queue.length) this.load();
    },
  },
  mounted() {
    this.load();
  },
};
</script>
