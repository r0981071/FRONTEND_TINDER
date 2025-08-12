<template>
  <div class="filter-card">
    <div class="row">
      <h4 class="title">Gender</h4>
      <div class="choices">
        <label><input type="radio" name="g" value=""        v-model="local.gender" /> Any</label>
        <label><input type="radio" name="g" value="male"    v-model="local.gender" /> Male</label>
        <label><input type="radio" name="g" value="female"  v-model="local.gender" /> Female</label>
      </div>
    </div>

    <div class="row">
      <h4 class="title">Age</h4>
      <div class="age">
        <input type="number" min="18" placeholder="Min" v-model.number="local.minAge" />
        <span>—</span>
        <input type="number" min="18" placeholder="Max" v-model.number="local.maxAge" />
      </div>
      <small class="hint">Leave blank for no limit.</small>
    </div>

    <div class="actions">
      <button @click="apply">Apply</button>
      <button class="ghost" @click="clear">Clear</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "Filtering_Component",
  props: {
    modelValue: {
      type: Object,
      default: () => ({ gender: "", minAge: null, maxAge: null })
    }
  },
  emits: ["update:modelValue", "apply"],
  data() {
    return { local: { ...this.modelValue } };
  },
  watch: {
    local: {
      deep: true,
      handler(v) {
        // keep parent in sync so hiding doesn't wipe choices
        this.$emit("update:modelValue", v);
      }
    },
    modelValue: {
      deep: true,
      handler(v) {
        this.local = { ...v };
      }
    }
  },
  methods: {
    apply() {
      this.$emit("apply", { ...this.local });
    },
    clear() {
      this.local = { gender: "", minAge: null, maxAge: null };
      this.$emit("apply", { ...this.local });
    }
  }
};
</script>

<style scoped>
.filter-card {
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 12px;
  background: #fff;
  display: grid;
  gap: 12px;
}
.row { display: grid; gap: 8px; }
.title { margin: 0; font-size: 14px; color: #374151; }
.choices { display: flex; gap: 14px; }
.age { display: flex; align-items: center; gap: 8px; }
.age input {
  width: 100px; height: 36px; padding: 0 8px;
  border: 1px solid #d1d5db; border-radius: 8px; outline: none;
}
.hint { color: #6b7280; }
.actions { display: flex; gap: 8px; }
.actions button {
  height: 36px; padding: 0 12px; border-radius: 8px; cursor: pointer;
  background: #2563eb; border: 1px solid #2563eb; color: #fff;
}
.actions .ghost { background: #fff; color: #111827; border-color: #d1d5db; }
</style>
