<template>
  <div class="messages-page">
    <!-- LEFT: matches list -->
    <aside class="matches">
      <h2>Your Matches</h2>
      <ul>
        <li
          v-for="m in matches"
          :key="m.other_user_id"
          :class="{ active: selected && selected.other_user_id === m.other_user_id }"
          @click="openThread(m)"
        >
          <span class="avatar">{{ m.other_name?.[0] || "?" }}</span>
          <span class="name">{{ m.other_name || ("User " + m.other_user_id) }}</span>
        </li>
      </ul>
    </aside>

    <!-- RIGHT: chat thread -->
    <section class="chat">
      <div v-if="!selected" class="empty">
        Pick a match to start chatting
      </div>

      <div v-else class="thread" ref="thread">
        <div
          v-for="msg in messages"
          :key="msg.message_id || (msg.sender_id + '-' + msg.content + '-' + Math.random())"
          class="row"
          :class="{ mine: msg.sender_id === meId, theirs: msg.sender_id !== meId }"
        >
          <div class="bubble">
            <p class="text">{{ msg.content }}</p>
          </div>
        </div>
      </div>

      <div class="composer" v-if="selected">
        <input
          v-model="draft"
          type="text"
          placeholder="Type a message..."
          @keyup.enter="send"
        />
        <button @click="send">Send</button>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  name: "Page_Messages",
  data() {
    return {
      meId: Number(localStorage.getItem("user_id")), // assumes you store this
      matches: [],
      selected: null,        // { other_user_id, other_name }
      messages: [],
      draft: "",
      api: "http://localhost:3000", // adjust if different
      loadingThread: false,
    };
  },
  mounted() {
    this.fetchMatches();
  },
  methods: {
    async fetchMatches() {
      try {
        const res = await fetch(`${this.api}/matches/${this.meId}`);
        const raw = await res.json(); // [{ match_id, user1_id, user2_id }]
        // decorate with the "other user" id; name optional if you have an endpoint
        this.matches = raw.map((m) => {
          const other = m.user1_id === this.meId ? m.user2_id : m.user1_id;
          return { other_user_id: other, other_name: m.other_name || null };
        });
      } catch (e) {
        console.error(e);
      }
    },
    async openThread(match) {
      this.selected = match;
      this.loadingThread = true;
      try {
        const res = await fetch(`${this.api}/conversations/with/${this.meId}/${match.other_user_id}`);
        const data = await res.json(); // { conversation_id, messages: [...] }
        this.messages = data.messages || [];
        this.$nextTick(this.scrollToBottom);
      } catch (e) {
        console.error(e);
      } finally {
        this.loadingThread = false;
      }
    },
    async send() {
      const content = this.draft.trim();
      if (!content || !this.selected) return;
      const payload = {
        sender_id: this.meId,
        receiver_id: this.selected.other_user_id,
        content,
      };
      try {
        const res = await fetch(`${this.api}/messages`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload),
        });
        if (!res.ok) {
          const err = await res.json().catch(() => ({}));
          throw new Error(err.error || "Failed to send");
        }
        const saved = await res.json();
        this.messages.push(saved);     // add to thread
        this.draft = "";
        this.$nextTick(this.scrollToBottom);
      } catch (e) {
        console.error(e);
        alert(e.message || "Could not send message");
      }
    },
    scrollToBottom() {
      const el = this.$refs.thread;
      if (el) el.scrollTop = el.scrollHeight;
    },
  },
};
</script>

<style scoped>
:host, :root, body {
  background: linear-gradient(180deg, #fff7ec 0%, #fff 40%, #fff 100%);
}

/* layout */
.messages-page {
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 24px;
  max-width: 1200px;
  margin: 24px auto;
  padding: 0 16px;
  box-sizing: border-box;
}

/* matches list */
.matches h2 {
  margin: 0 0 12px;
  font-size: 20px;
}
.matches ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
.matches li {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 10px;
  cursor: pointer;
  margin-bottom: 10px;
  transition: background 0.15s ease;
}
.matches li:hover { background: #fafafa; }
.matches li.active {
  border-color: #7aa7ff;
  background: #f2f6ff;
}
.matches .avatar {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  display: grid;
  place-items: center;
  background: #e5e7eb;
  font-weight: 600;
}
.matches .name {
  font-size: 15px;
}

/* chat pane */
.chat {
  display: grid;
  grid-template-rows: 1fr auto;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  min-height: 440px;
}
.empty {
  display: grid;
  place-items: center;
  color: #666;
}

/* message list */
.thread {
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.thread::-webkit-scrollbar { width: 8px; }
.thread::-webkit-scrollbar-thumb { background: #d4d4d8; border-radius: 8px; }

/* each row aligns left/right */
.row {
  display: flex;
  width: 100%;
}
.row.mine { justify-content: flex-end; }
.row.theirs { justify-content: flex-start; }

/* chat bubble */
.bubble {
  max-width: 70%;
  padding: 10px 14px;
  border-radius: 16px;
  word-break: break-word;
  white-space: pre-wrap;
  line-height: 1.25;
  box-shadow: 0 1px 0 rgba(0,0,0,0.04);
}
.row.mine .bubble {
  background: #dbeafe;        /* light blue */
  color: #0b3b7e;
  border-top-right-radius: 6px;
}
.row.theirs .bubble {
  background: #f3f4f6;        /* light gray */
  color: #111827;
  border-top-left-radius: 6px;
}

/* composer */
.composer {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 8px;
  padding: 10px;
  border-top: 1px solid #e5e7eb;
}
.composer input {
  height: 40px;
  padding: 0 12px;
  border: 1px solid #d1d5db;
  border-radius: 10px;
  outline: none;
}
.composer input:focus { border-color: #7aa7ff; }
.composer button {
  height: 40px;
  padding: 0 16px;
  border: 1px solid #2563eb;
  background: #2563eb;
  color: #fff;
  border-radius: 10px;
  cursor: pointer;
}
.composer button:hover { background: #1d4ed8; }
</style>
