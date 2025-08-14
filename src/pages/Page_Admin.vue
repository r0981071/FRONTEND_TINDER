<template>
  <div class="admin">
    <h1 class="admin-title">Administrator Profile</h1>
    <h2 class="section-title">All Users</h2>

    <div v-if="loading" class="muted">Loading…</div>
    <div v-else-if="users.length === 0" class="muted">
      No users found (besides admin).
    </div>

    <div class="grid">
      <div v-for="u in users" :key="u.user_id" class="card">
        <img
          class="avatar"
          :src="u.profile?.profile_pic_url || placeholder"
          alt=""
          @error="onImgError($event)"
        />
        <div class="info">
          <div class="name">{{ u.profile?.name || '—' }}</div>
          <div class="sub">@{{ u.username }} (id: {{ u.user_id }})</div>
        </div>

        <div class="actions">
          <input
            v-model="pwd[u.user_id]"
            placeholder="new password"
            class="pwd"
            @keyup.enter="changePassword(u.user_id)"
          />
          <button class="btn" @click="changePassword(u.user_id, pwd[u.user_id])">Change PW</button>
          <button class="btn danger" @click="remove(u.user_id)">Delete account</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const ADMIN_HEADER = { "x-admin-user": "admin" }; // matches your isAdmin() middleware

export default {
  name: "Page_Admin",
  data() {
    return {
      users: [],
      loading: false,
      pwd: {}, // { [userId]: 'newpass' }
      placeholder:
        "https://dummyimage.com/120x120/eeeeee/999999.jpg&text=user",
    };
  },
  methods: {
    async load() {
      this.loading = true;
      try {
        const res = await fetch("http://localhost:3000/api/admin/users", {
          headers: ADMIN_HEADER,
        });
        this.users = res.ok ? await res.json() : [];
      } catch (e) {
        console.error(e);
        this.users = [];
      } finally {
        this.loading = false;
      }
    },
    onImgError(e) {
      e.target.src = this.placeholder;
    },
    async changePassword(userId, newPassword) {
        const np = (newPassword ?? '').trim();
        if (!np) {
            alert('Type a new password first.');
            return;
        }

        try {
            const res = await fetch(`http://localhost:3000/api/admin/users/${userId}/password`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'x-admin-user': 'admin'
            },
            body: JSON.stringify({ new_password: np })
            });

            if (!res.ok) {
            const err = await res.json().catch(() => ({}));
            throw new Error(err.error || 'Failed to update password');
            }

            alert('Password updated successfully!');
        } catch (e) {
            alert(e.message || 'An error occurred');
        }
        },
    async remove(userId) {
      if (!confirm("Delete this user and all related data?")) return;
      try {
        const res = await fetch(
          `http://localhost:3000/api/admin/users/${userId}?username=admin`,
          { method: "DELETE", headers: ADMIN_HEADER }
        );
        if (!res.ok) throw new Error(await res.text());
        this.users = this.users.filter((u) => u.user_id !== userId);
      } catch (e) {
        console.error(e);
        alert("Delete failed.");
      }
    },
  },
  mounted() {
    this.load();
  },
};
</script>

<style scoped>
.admin {
  max-width: 900px;
  margin: 24px auto;
  padding: 20px 16px;
  background: #f9fafb; /* light background */
  border-radius: 12px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.06);
}
.admin-title {
  font-size: 28px;
  font-weight: 800;
  color: #1f2937;
  margin-bottom: 16px;
  border-bottom: 2px solid #e5e7eb;
  padding-bottom: 6px;
}
.section-title {
  font-size: 20px;
  font-weight: 700;
  margin: 12px 0;
  color: #374151;
}
.muted { color: #6b7280; margin: 12px 0; }

.grid { display: grid; gap: 12px; }
.card {
  display: grid;
  grid-template-columns: 120px 1fr auto;
  gap: 12px;
  align-items: center;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  background: #fff;
  padding: 12px;
}
.avatar { width: 120px; height: 120px; object-fit: cover; border-radius: 10px; }
.info .name { font-weight: 700; }
.info .sub { color: #6b7280; font-size: 14px; }

.actions { display: grid; gap: 8px; grid-auto-flow: row; justify-items: end; }
.pwd {
  height: 36px; padding: 0 10px;
  border: 1px solid #d1d5db; border-radius: 8px;
  width: 160px;
}
.btn {
  height: 36px; padding: 0 12px; border-radius: 8px; cursor: pointer;
  border: 1px solid #2563eb; background: #2563eb; color:#fff;
}
.btn:hover { background: #1d4ed8; }
.btn.danger { border-color:#ef4444; background:#ef4444; }
.btn.danger:hover { background:#dc2626; }
</style>
