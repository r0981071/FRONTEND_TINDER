<template>
  <div class="filter-shell" :class="{ collapsed }">
    <div class="content" v-show="!collapsed">
      <label class="lab">Gender</label>
      <select v-model="local.gender" class="select">
        <option value="">Any</option>
        <option value="male">Male</option>
        <option value="female">Female</option>
      </select>

      <label class="lab">Min age</label>
      <input
        type="number"
        min="18"
        placeholder="min"
        v-model.number="local.minAge"
        class="input"
      />

      <label class="lab">Max age</label>
      <input
        type="number"
        min="18"
        placeholder="max"
        v-model.number="local.maxAge"
        class="input"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: "FilteringComponent",
  props: {
    /* Vue 3 v-model */
    modelValue: {
      type: Object,
      default: () => ({ gender: "", minAge: null, maxAge: null })
    },
    /* Vue 2 v-model */
    value: {
      type: Object,
      default: () => ({ gender: "", minAge: null, maxAge: null })
    },
    collapsed: { type: Boolean, default: false }
  },
  /* Vue 3 emits (ignored in Vue 2, harmless) */
  emits: ["update:modelValue", "input"],
  data() {
    return {
      // prefer modelValue (Vue 3); fallback to value (Vue 2)
      local: { ...(this.modelValue ?? this.value) }
    };
  },
  watch: {
    // keep local in sync if parent changes from either prop
    modelValue: {
      deep: true,
      handler(v) { if (v) this.local = { ...v }; }
    },
    value: {
      deep: true,
      handler(v) { if (v && this.modelValue === undefined) this.local = { ...v }; }
    },
    // push changes up for BOTH APIs so parent always receives updates
    local: {
      deep: true,
      handler(v) {
        this.$emit("update:modelValue", v); // Vue 3
        this.$emit("input", v);             // Vue 2
      }
    }
  }
};
</script>

<style scoped>
:root { --filter-h: 36px; }

.filter-shell {
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  background: #fff;
  padding: 0 12px;
  height: 44px;
  display: flex;
  align-items: center;
  overflow: hidden;
  box-sizing: border-box;
}
.content {
  display: flex;
  gap: 10px;
  align-items: center;
  width: 100%;
}
.lab {
  font-size: 13px;
  color: #374151;
  user-select: none;
}
.select, .input {
  height: 28px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  padding: 0 8px;
  outline: none;
}
.select { min-width: 120px; }
.input  { width: 90px; }
</style>
