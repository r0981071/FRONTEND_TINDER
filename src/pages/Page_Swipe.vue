<template>
  <div>
    <div style="display:flex; align-items:center; justify-content:space-between; max-width:480px; margin:0 auto 8px;">
      <h2 style="margin:0;">Swipe</h2>
      <button @click="showFilters = !showFilters" style="padding:6px 10px; border:1px solid #ccc; border-radius:8px; cursor:pointer;">
        {{ showFilters ? 'Hide filters' : 'Filter' }}
      </button>
    </div>

    <!-- Filtering panel (toggleable). Hiding it DOES NOT clear selections -->
    <div v-show="showFilters" style="border:1px solid #e5e7eb; border-radius:10px; padding:12px; max-width:480px; margin:0 auto 16px;">
      <div style="display:grid; grid-template-columns: 1fr 1fr; gap:12px;">
        <!-- Gender -->
        <div>
          <label style="display:block; font-weight:600; margin-bottom:6px;">Gender</label>
          <select v-model="filters.gender" style="width:100%; height:36px; border:1px solid #d1d5db; border-radius:8px; padding:0 8px;">
            <option value="">Any</option>
            <option value="male">Male</option>
            <option value="female">Female</option>
          </select>
        </div>

        <!-- Age range -->
        <div>
          <label style="display:block; font-weight:600; margin-bottom:6px;">Age</label>
          <div style="display:grid; grid-template-columns: 1fr 1fr; gap:8px;">
            <input
              v-model.number="filters.ageMin"
              type="number"
              min="18"
              placeholder="min"
              style="height:36px; border:1px solid #d1d5db; border-radius:8px; padding:0 8px;"
            />
            <input
              v-model.number="filters.ageMax"
              type="number"
              min="18"
              placeholder="max"
              style="height:36px; border:1px solid #d1d5db; border-radius:8px; padding:0 8px;"
            />
          </div>
          <small style="color:#6b7280;">Leave blank for no limit.</small>
        </div>
      </div>
    </div>

    <div v-if="current" style="border:1px solid #ccc; padding:1rem; max-width:480px; margin:0 auto;">
      <!-- Photo -->
      <img :src="current.profile_pic_url" alt="" style="width:300px; height:300px;" />

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

    <p v-else style="text-align:center; color:#6b7280;">No more profiles.</p>
  </div>
</template>

<script>
export default {
  name: "Page_Swipe",
  props: { user: { type: Object, required: true } },
  data() {
    return {
      queue: [],           // raw list from server (unfiltered)
      index: 0,            // index within filteredQueue
      loading: false,
      showFilters: false,  // toggle panel
      filters: {
        gender: "",        // "", "male", "female"
        ageMin: null,      // number | null
        ageMax: null       // number | null
      }
    };
  },
  computed: {
    // Apply filters client-side (no backend change required)
    filteredQueue() {
      const { gender, ageMin, ageMax } = this.filters;
      return this.queue.filter(p => {
        // gender filter
        if (gender === "male" && !(p.gender_male === true || p.gender_male === 1 || p.gender_male === "true")) return false;
        if (gender === "female" && (p.gender_male === true || p.gender_male === 1 || p.gender_male === "true")) return false;

        // age filter
        const age = Number(p.age);
        if (Number.isFinite(ageMin) && ageMin !== null && age < ageMin) return false;
        if (Number.isFinite(ageMax) && ageMax !== null && age > ageMax) return false;

        return true;
      });
    },
    current() {
      return this.filteredQueue[this.index] || null;
    },
  },
  watch: {
    // If filters change, keep the current pointer valid
    filters: {
      deep: true,
      handler() {
        if (this.index >= this.filteredQueue.length) this.index = 0;
      }
    }
  },
  methods: {
    genderLabel(val) {
      const b = val === true || val === 1 || val === "true";
      return b ? "Male" : "Female";
    },
    fmtHeight(h) {
      const n = Number(h);
      return Number.isFinite(n) ? `${n.toFixed(2)} m` : "";
    },
    fmtWeight(w) {
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

      // keep a reference before we change arrays
      const swipedId = this.current.profile_id;

      // record on the server
      const payload = {
        swiper_id: this.user.user_id,
        swiped_id: swipedId,
        is_like,
      };
      await fetch("http://localhost:3000/api/swipes", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload),
      });

      // Remove the swiped profile from the *raw* queue so it can't reappear after filters change
      this.queue = this.queue.filter(p => p.profile_id !== swipedId);

      // Move forward in the filtered cursor (stay in bounds)
      if (this.index >= this.filteredQueue.length) {
        this.index = 0;
      }
    },
  },
  mounted() {
    this.load();
  },
};
</script>
