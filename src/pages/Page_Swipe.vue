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

    <div v-if="current" class="card" :key="current.profile_id">
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
      // IMPORTANT: the keys match the child (minAge/maxAge)
      filters: { gender: "", minAge: null, maxAge: null }
    };
  },
  computed: {
    // bridge for child (same keys)
    filtersForChild: {
      get() {
        return {
          gender: this.filters.gender,
          minAge: this.filters.minAge,
          maxAge: this.filters.maxAge
        };
      },
      set(v) {
        this.filters.gender = v.gender || "";
        this.filters.minAge = v.minAge ?? null;
        this.filters.maxAge = v.maxAge ?? null;
      }
    },

    // Pure client-side filtering
    filteredQueue() {
      const { gender, minAge, maxAge } = this.filters;

      return this.queue.filter(p => {
        // Gender filter
        if (gender === "male"   && !p.gender_male) return false;
        if (gender === "female" &&  p.gender_male) return false;

        // Age filter
        const age = (p.age === null || p.age === undefined) ? null : Number(p.age);
        if (minAge != null && age != null && age < Number(minAge)) return false;
        if (maxAge != null && age != null && age > Number(maxAge)) return false;

        return true;
      });
    },

    current() {
      return this.filteredQueue[this.index] || null;
    }
  },

  watch: {
    // On ANY filter change, jump to the first matching result
    filters: {
      deep: true,
      handler() {
        this.index = 0;
      }
    },
    // If the *underlying* queue changes (first load / after swipe), keep index in bounds
    filteredQueue() {
      if (this.index >= this.filteredQueue.length) this.index = 0;
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
      try {
        // Load ONCE (unfiltered) and normalize; client-side filtering does the rest
        const res = await fetch(`http://localhost:3000/api/profiles/for-swipe/${this.user.user_id}`);
        const raw = await res.json();

        this.queue = Array.isArray(raw)
          ? raw.map(p => ({
              ...p,
              // normalize so filtering is reliable
              age:
                p.age === null || p.age === undefined || p.age === ""
                  ? null
                  : Number(p.age),
              gender_male:
                p.gender_male === true ||
                p.gender_male === 1 ||
                p.gender_male === "1" ||
                p.gender_male === "true",
            }))
          : [];

        this.index = 0;
      } catch (e) {
        console.error("load() failed", e);
        this.queue = [];
      } finally {
        this.loading = false;
      }
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

      // Remove from *raw* queue so it cannot reappear when filters change
      this.queue = this.queue.filter(p => p.profile_id !== swipedId);

      // keep cursor valid for the new filtered view
      if (this.index >= this.filteredQueue.length) this.index = 0;
    }
  },

  mounted() {
    this.load();
  }
};
</script>

<style scoped>
:root { --filter-h: 36px; }

/* Page background — very light gradient */
.page {
  max-width: 840px;
  margin: 24px auto 48px;
  padding: 0 16px 24px;
  background: linear-gradient(180deg, #fffdf8 0%, #fff7ea 100%);
  border-radius: 20px;
  box-shadow: 0 12px 35px rgba(255, 166, 0, 0.08);
  border: 1px solid rgba(255, 171, 0, 0.15);
}

/* Heading */
.header {
  display: flex;
  justify-content: center;
  margin: 6px 0 14px;
}
.title {
  margin: 0;
  font-size: 28px;
  letter-spacing: .3px;
  text-shadow: 0 1px 0 #fff;
}

/* Filter bar row */
.filter-row {
  display: flex;
  gap: 10px;
  align-items: center;
  justify-content: center;
  margin: 0 auto 20px;
  height: 42px;
}
.toggle {
  height: 100%;
  padding: 0 14px;
  border: 1px solid #ffd39a;
  border-radius: 12px;
  background: linear-gradient(180deg, #fff7e3 0%, #ffe7b7 100%);
  cursor: pointer;
  transition: transform .08s ease, box-shadow .12s ease;
  box-shadow: 0 2px 8px rgba(255, 170, 0, 0.18) inset, 0 2px 6px rgba(0,0,0,.04);
}
.toggle:hover { transform: translateY(-1px); }
.toggle:active { transform: translateY(0); }

.flex-1 { flex: 1; max-width: 620px; }

/* Profile card */
.card {
  position: relative;
  border: 1px solid rgba(255, 179, 71, 0.25);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(8px);
  padding: 18px 18px 14px;
  max-width: 680px;
  margin: 0 auto;
  text-align: center;
  box-shadow:
    0 10px 30px rgba(255, 170, 0, 0.10),
    0 1px 0 rgba(255,255,255,0.8) inset;
}

.photo {
  width: 100%;
  max-width: 560px;
  aspect-ratio: 4/3;
  object-fit: cover;
  border-radius: 14px;
  display: block;
  margin: 0 auto;
  box-shadow: 0 12px 30px rgba(0,0,0,.08);
  border: 6px solid #fff;
}

.name { margin: 14px 0 6px; font-size: 22px; }
.meta { margin: 2px 0; color: #1f2937; opacity: .85; }
.bio  { margin-top: 8px; color: #394150; }

.actions {
  display: flex;
  gap: 18px;
  justify-content: center;
  margin-top: 18px;
}
.action {
  width: 64px;
  height: 64px;
  font-size: 24px;
  line-height: 1;
  border-radius: 50%;
  cursor: pointer;
  border: 2px solid #fff;
  background: #fff;
  box-shadow:
    0 6px 18px rgba(0,0,0,.08),
    0 0 0 6px rgba(255, 170, 0, 0.08);
  transition: transform .1s ease, box-shadow .15s ease;
}
.action:hover {
  transform: translateY(-2px);
  box-shadow:
    0 10px 24px rgba(0,0,0,.12),
    0 0 0 8px rgba(255, 170, 0, 0.12);
}
.action:active { transform: translateY(0); }
.action.like { border-color: #ff5a76; color: #ff5a76; }
.action.dislike { border-color: #9aa0a6; color: #9aa0a6; }

.empty {
  text-align: center;
  color: #6b7280;
  margin-top: 32px;
}
</style>
