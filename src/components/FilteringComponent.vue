<template>
  <!-- accepts outer class (e.g., flex-1) from parent -->
  <div class="filter-shell" :class="{ collapsed }">
    <!-- When collapsed we keep the same shell but hide the inputs -->
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
    modelValue: {
      type: Object,
      default: () => ({ gender: "", minAge: null, maxAge: null })
    },
    collapsed: { type: Boolean, default: false }
  },
  emits: ["update:modelValue"],
  data() {
    return { local: { ...this.modelValue } };
  },
  watch: {
    local: {
      deep: true,
      handler(v) {
        this.$emit("update:modelValue", v);
      }
    },
    modelValue: {
      deep: true,
      handler(v) {
        this.local = { ...v };
      }
    }
  }
};
</script>

<style scoped>
/* keep in sync with PageSwipe.vue's --filter-h */
:root { --filter-h: 36px; }

.filter-shell {
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  background: #fff;
  padding: 0 12px;
  height: 44px;              /* NEW: match toggle button height */
  display: flex;
  align-items: center;
  overflow: hidden;          /* prevents height change on collapse */
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
.select,
.input {
  height: 28px;                       /* shorter controls inside */
  border: 1px solid #d1d5db;
  border-radius: 8px;
  padding: 0 8px;
  outline: none;
}
.select { min-width: 120px; }
.input  { width: 90px; }
</style>
