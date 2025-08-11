<template>
  <div>
    <h2>Your Profile</h2>

    <form @submit.prevent="save" style="max-width:520px; margin:0 auto; text-align:left; display:grid; gap:10px;">
      <label>Name
        <input v-model="form.name" required style="width:100%;" />
      </label>

      <label>Gender
        <select v-model="form.gender_male" style="width:100%;">
          <option disabled value="">Select gender</option>
          <option :value="true">Male</option>
          <option :value="false">Female</option>
        </select>
      </label>

      <label>Age
        <input v-model.number="form.age" type="number" min="18" step="1" style="width:100%;" />
      </label>

      <label>Nationality
        <input v-model="form.nationallity" style="width:100%;" />
      </label>

      <label>Height (meters)
        <input v-model.number="form.height" type="number" step="0.01" style="width:100%;" />
      </label>

      <label>Weight (kg)
        <input v-model.number="form.weight" type="number" step="0.1" style="width:100%;" />
      </label>

      <label>Bio
        <textarea v-model="form.bio" rows="3" style="width:100%;"></textarea>
      </label>

      <!-- File picker (no text box) -->
      <label>Profile picture
        <input type="file" accept="image/png, image/jpeg" @change="onFileChange" />
      </label>

      <!-- Preview -->
      <div v-if="form.profile_pic_url && !imgError" style="margin-top:4px;">
        <small>Preview:</small><br />
        <img
          :src="form.profile_pic_url"
          alt="preview"
          style="max-width:100%; border:1px solid #eee; padding:4px;"
          @error="imgError = true"
          @load="imgError = false"
        />
      </div>

      <div style="margin-top:8px;">
        <button type="submit">Save</button>
        <span v-if="saved" style="color:green; margin-left:8px;">Saved!</span>
      </div>
    </form>
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
        profile_id: this.user.user_id, // matches backend field
        name: "",
        gender_male: "",
        age: null,
        nationallity: "",
        height: null,
        weight: null,
        bio: "",
        profile_pic_url: "", // gets set automatically after upload
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
