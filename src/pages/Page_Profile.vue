<template>
  <div class="profile-page">
    <div class="card">
      <h2 class="title">Your Profile</h2>

      <form @submit.prevent="save" class="form">
        <!-- Name -->
        <label class="field">
          <span class="label">Name</span>
          <input v-model="form.name" required class="control" />
        </label>

        <!-- Gender -->
        <label class="field">
          <span class="label">Gender</span>
          <select v-model="form.gender_male" class="control">
            <option disabled value="">Select gender</option>
            <option :value="true">Male</option>
            <option :value="false">Female</option>
          </select>
        </label>

        <!-- Age -->
        <label class="field">
          <span class="label">Age</span>
          <input v-model.number="form.age" type="number" min="18" step="1" class="control" />
        </label>

        <!-- Nationality -->
        <label class="field">
          <span class="label">Nationality</span>
          <input v-model="form.nationallity" class="control" />
        </label>

        <!-- Height -->
        <label class="field">
          <span class="label">Height (meters)</span>
          <input v-model.number="form.height" type="number" step="0.01" class="control" />
        </label>

        <!-- Weight -->
        <label class="field">
          <span class="label">Weight (kg)</span>
          <input v-model.number="form.weight" type="number" step="0.1" class="control" />
        </label>

        <!-- Bio -->
        <label class="field">
          <span class="label">Bio</span>
          <textarea v-model="form.bio" rows="3" class="control"></textarea>
        </label>

        <!-- File picker -->
        <label class="field">
          <span class="label">Profile picture</span>
          <input type="file" accept="image/png, image/jpeg" @change="onFileChange" class="control file" />
        </label>

        <!-- Preview -->
        <div v-if="form.profile_pic_url && !imgError" class="preview">
          <img
            :src="form.profile_pic_url"
            alt="preview"
            @error="imgError = true"
            @load="imgError = false"
          />
          <small class="hint">Preview</small>
        </div>

        <!-- Actions -->
        <div class="actions">
          <button type="submit" class="btn primary">Save</button>
          <span v-if="saved" class="saved">Saved!</span>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: "Page_Profile",
  props: { user: { type: Object, required: true } },
  data() {
    return {
      saved: false,
      imgError: false,
      form: {
        profile_id: this.user.user_id,
        name: "",
        gender_male: "",
        age: null,
        nationallity: "",
        height: null,
        weight: null,
        bio: "",
        profile_pic_url: "",
      },
    };
  },
  async mounted() {
    const res = await fetch(`http://localhost:3000/api/profiles/${this.user.user_id}`);
    if (res.ok) {
      const p = await res.json();
      this.form = { profile_id: this.user.user_id, ...p };
    }
  },
  methods: {
    async onFileChange(e) {
      const file = e.target.files?.[0];
      if (!file) return;

      const fd = new FormData();
      fd.append("file", file);

      try {
        const res = await fetch("http://localhost:3000/api/uploads", {
          method: "POST",
          body: fd,
        });
        if (!res.ok) throw new Error(await res.text());
        const { url } = await res.json();
        this.form.profile_pic_url = url;
        this.imgError = false;
      } catch (err) {
        console.error("Upload failed:", err);
        alert("Upload failed. Please try another image.");
      }
    },

    async save() {
      this.saved = false;
      const payload = { ...this.form };

      const res = await fetch("http://localhost:3000/api/profiles", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload),
      });

      if (res.ok) {
        this.saved = true;
      } else {
        console.error("Profile save failed", await res.text());
        alert("Could not save profile.");
      }
    },
  },
};
</script>

<style scoped>
/* page container with subtle vertical gradient */
.profile-page {
  min-height: calc(100vh - 64px);
  background: linear-gradient(180deg, #fffaf2 0%, #fff 100%);
  padding: 24px 16px 48px;
  display: grid;
  place-items: start center;
}

/* card */
.card {
  width: 100%;
  max-width: 680px;
  background: #fff;
  border: 1px solid #f1f5f9;
  border-radius: 16px;
  padding: 22px;
  box-shadow: 0 12px 30px rgba(255, 170, 56, 0.08);
}

/* heading */
.title {
  text-align: center;
  margin: 4px 0 14px;
}

/* form grid */
.form {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px 16px;
}

/* stack wide fields full width */
.field:nth-child(1),
.field:nth-child(7),
.field:nth-child(8),
.preview,
.actions {
  grid-column: 1 / -1;
}

/* field block */
.field {
  display: grid;
  gap: 6px;
}
.label {
  font-size: 13px;
  color: #475569;
}

/* controls */
.control {
  height: 42px;
  padding: 0 12px;
  border: 1px solid #dfe3ea;
  border-radius: 10px;
  outline: none;
  background: #fff;
  width: 100%;
  box-sizing: border-box;
}
.control:focus {
  border-color: #f59e0b;
  box-shadow: 0 0 0 3px rgba(245, 158, 11, 0.15);
}
textarea.control {
  height: 88px;
  padding: 10px 12px;
  resize: vertical;
}
.file {
  padding: 8px 10px;
  height: auto;
}

/* preview */
.preview {
  display: grid;
  justify-items: center;
  gap: 8px;
  padding-top: 4px;
}
.preview img {
  width: 180px;
  height: 180px;
  object-fit: cover;
  border-radius: 14px;
  border: 1px solid #eee;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.04);
}
.hint {
  color: #6b7280;
}

/* actions */
.actions {
  display: flex;
  align-items: center;
  gap: 10px;
}
.btn {
  height: 42px;
  padding: 0 16px;
  border-radius: 10px;
  border: 1px solid transparent;
  cursor: pointer;
}
.btn.primary {
  background: linear-gradient(135deg, #ffb347 0%, #ff8c42 100%);
  border-color: #ffa73a;
  color: #fff;
  font-weight: 600;
  box-shadow: 0 10px 24px rgba(255, 167, 58, 0.25);
}
.btn.primary:hover {
  filter: brightness(1.03);
}
.saved {
  color: #16a34a;
  font-weight: 600;
}
</style>
  