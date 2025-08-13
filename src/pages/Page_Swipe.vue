<template>
  <div class="page">
    <div class="header">
      <h2 class="title">Swipe</h2>
    </div>

    <!-- Button on the LEFT, touching the filter component on the RIGHT -->
    <div class="filter-row">
      <button class="toggle" @click="showFilters = !showFilters">
        {{ showFilters ? 'Hide filters' : 'Filters' }}
      </button>

      <!-- The shell stays; contents hide/show via :collapsed -->
      <FilteringComponent
        v-model="filtersForChild"
        :collapsed="!showFilters"
        class="flex-1"
      />
    </div>

    <div v-if="current" class="card">
      <img :src="current.profile_pic_url" alt="" class="photo" />

      <h3 class="name">{{ current.name }}</h3>

      <p class="meta">
        <strong>
          {{ current.age }}<span v-if="current.age">, </span>
          {{ genderLabel(current.gender_male) }}
          <span v-if="current.nationallity">, from {{ current.nationallity }}</span>
        </strong>
      </p>

      <p class="meta">
        <span v-if="current.height !== undefined && current.height !== null">
          {{ fmtHeight(current.height) }}
        </span>
        <span
          v-if="current.height !== undefined && current.height !== null && current.weight !== undefined && current.weight !== null"
        > · </span>
        <span v-if="current.weight !== undefined && current.weight !== null">
          {{ fmtWeight(current.weight) }}
        </span>
      </p>

      <p class="bio">{{ current.bio }}</p>

      <div class="actions">
        <button class="action dislike" @click="swipe(false)">❌</button>
        <button class="action like" @click="swipe(true)">❤️</button>
      </div>
    </div>

    <p v-else class="empty">No more profiles.</p>
  </div>
</template>

<script>
import FilteringComponent from "@/components/FilteringComponent.vue";

export default {
  name: "Page_Swipe",
  components: { FilteringComponent },
  props: { user: { type: Object, required: true } },
  data() {
    return {
      queue: [],
      index: 0,
      loading: false,
      showFilters: true,
      filters: { gender: "", ageMin: null, ageMax: null }
    };
  },
  computed: {
    // bridge for child (same keys)
    filtersForChild: {
      get() {
        return {
          gender: this.filters.gender,
          minAge: this.filters.ageMin,
          maxAge: this.filters.ageMax
        };
      },
      set(v) {
        this.filters.gender = v.gender || "";
        this.filters.ageMin = v.minAge ?? null;
        this.filters.ageMax = v.maxAge ?? null;
      }
    },
    filteredQueue() {
      const { gender, ageMin, ageMax } = this.filters;
      return this.queue.filter(p => {
        if (gender === "male" && !(p.gender_male === true || p.gender_male === 1 || p.gender_male === "true")) return false;
        if (gender === "female" &&  (p.gender_male === true || p.gender_male === 1 || p.gender_male === "true")) return false;

        const age = Number(p.age);
        if (Number.isFinite(ageMin) && ageMin !== null && age < ageMin) return false;
        if (Number.isFinite(ageMax) && ageMax !== null && age > ageMax) return false;

        return true;
      });
    },
    current() {
      return this.filteredQueue[this.index] || null;
    }
  },
  watch: {
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
      const swipedId = this.current.profile_id;

      await fetch("http://localhost:3000/api/swipes", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          swiper_id: this.user.user_id,
          swiped_id: swipedId,
          is_like
        }),
      });

      this.queue = this.queue.filter(p => p.profile_id !== swipedId);
      if (this.index >= this.filteredQueue.length) this.index = 0;
    }
  },
  mounted() {
    this.load();
  }
};
</script>

<style scoped>
/* shared height for button + filter component */
:root { --filter-h: 36px; } /* tweak here for shorter/taller */

.page {
  max-width: 720px;
  margin: 24px auto;
  padding: 0 16px;
}
.header {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
}
.title { margin: 0; }

.filter-row {
  display: flex;
  gap: 8px;
  align-items: center;
  justify-content: center;
  margin: 0 auto 18px;
  height: 44px; /* NEW: fixes height so button & filter match */
}
.toggle {
  height: 100%; /* matches parent height */
  padding: 0 14px;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  background: #fff;
  cursor: pointer;
}

/* lets the filter component grow to the right side */
.flex-1 { flex: 1; max-width: 560px; }

.card {
  border: 1px solid #e5e7eb;
  border-radius: 16px;
  background: #fff;
  padding: 16px;
  max-width: 560px;
  margin: 0 auto;
  box-shadow: 0 1px 3px rgba(0,0,0,0.04);
  text-align: center;
}
.photo {
  width: 100%;
  max-width: 480px;
  aspect-ratio: 4/3;
  object-fit: cover;
  border-radius: 12px;
  margin: 0 auto;
  display: block;
}
.name { margin: 12px 0 6px; }
.meta { margin: 4px 0; color: #111827; }
.bio  { margin-top: 6px; color: #374151; }

.actions {
  display: flex;
  gap: 16px;
  justify-content: center;
  margin-top: 14px;
}
.action {
  width: 56px;
  height: 56px;
  font-size: 22px;
  line-height: 1;
  border-radius: 50%;
  cursor: pointer;
  border: 1px solid #e5e7eb;
  background: #fff;
}
.action.like    { border-color: #ef4444; }
.action.dislike { border-color: #6b7280; }

.empty {
  text-align: center;
  color: #6b7280;
  margin-top: 24px;
}
</style>
