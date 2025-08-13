<template>
  <!-- Show normal UI only if there are matches -->
  <div v-if="hasMatches" style="display:grid; grid-template-columns: 220px 1fr; gap:16px; max-width:960px; margin:0 auto;">
    <!-- LEFT: matches list -->
    <aside>
      <h3>Your Matches</h3>
      <ul style="list-style:none; padding:0;">
        <li v-for="m in matches" :key="m.user_id" style="margin-bottom:8px;">
          <button
            @click="open(m.user_id)"
            style="width:100%; text-align:left; padding:8px; border:1px solid #ddd; background:#fff; cursor:pointer;">
            {{ m.name || m.username || ('User #' + m.user_id) }}
          </button>
        </li>
      </ul>
    </aside>

    <!-- RIGHT: chat -->
    <section v-if="activeId" class="chat">
      <div class="thread" ref="thread">
        <div
          v-for="(msg, i) in messages"
          :key="msg.message_id || i"
          class="row"
          :class="{ mine: msg.sender_id === user.user_id, theirs: msg.sender_id !== user.user_id }"
        >
          <div class="bubble">
            <p class="text">{{ msg.content }}</p>
          </div>
        </div>
      </div>

      <form @submit.prevent="send" class="composer">
        <input v-model="text" placeholder="Type a message..." />
        <button type="submit">Send</button>
      </form>
    </section>

    <section v-else style="display:flex; align-items:center; justify-content:center;">
      <p>Select a match to chat</p>
    </section>
  </div>

  <!-- Empty state -->
  <div v-else class="no-matches">You have no matches yet.</div>
</template>

<script>
export default {
  name: "Page_Matches",
  props: { user: { type: Object, required: true } },
  data() {
    return {
      matches: [],
      activeId: null,
      conversationId: null,
      messages: [],
      text: ""
    };
  },
  computed: {
    hasMatches() {
      return Array.isArray(this.matches) && this.matches.length > 0;
    }
  },
  methods: {
    async loadMatches() {
      try {
        const res = await fetch(`http://localhost:3000/api/matches/list/${this.user.user_id}`);
        this.matches = res.ok ? await res.json() : [];
        if (!this.hasMatches) {
          this.activeId = null;   // ensure chat area doesn't show
          this.messages = [];
        }
      } catch (e) {
        console.error("loadMatches failed", e);
        this.matches = [];
        this.activeId = null;
        this.messages = [];
      }
    },

    async open(otherId) {
      try {
        this.activeId = otherId;
        this.messages = [];
        this.conversationId = null;

        const res = await fetch(
          `http://localhost:3000/api/conversations/with/${this.user.user_id}/${otherId}`
        );
        if (!res.ok) {
          console.error("open() failed", await res.text());
          return;
        }

        const data = await res.json();
        this.conversationId = data.conversation_id;
        this.messages = data.messages || [];

        this.$nextTick(() => this.scrollToBottom(true));
      } catch (e) {
        console.error("open failed", e);
      }
    },

    async send() {
      const text = this.text.trim();
      if (!text || !this.activeId) return;

      try {
        const payload = {
          sender_id: this.user.user_id,
          receiver_id: this.activeId,
          content: text
        };

        const res = await fetch("http://localhost:3000/api/messages", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(payload)
        });

        if (!res.ok) {
          console.error("send() failed", await res.text());
          return;
        }

        const msg = await res.json();
        this.messages.push(msg);
        this.text = "";
        this.$nextTick(() => this.scrollToBottom(true));
      } catch (e) {
        console.error("send failed", e);
      }
    },

    scrollToBottom(force = false) {
      const el = this.$refs.thread;
      if (!el) return;
      const gap = el.scrollHeight - el.scrollTop - el.clientHeight;
      const nearBottom = gap < 80;
      if (force || nearBottom) {
        requestAnimationFrame(() => {
          el.scrollTop = el.scrollHeight;
        });
      }
    }
  },

  mounted() {
    this.loadMatches();
  }
};
</script>

<style scoped>
.chat {
  display: grid;
  grid-template-rows: 1fr auto;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  height: 65vh;
  max-height: 65vh;
  overflow: hidden;
}
.thread {
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-height: 100%;
}
.row { display: flex; width: 100%; }
.row.mine { justify-content: flex-end; }
.row.theirs { justify-content: flex-start; }
.bubble {
  max-width: 70%;
  padding: 10px 14px;
  border-radius: 16px;
  word-break: break-word;
  white-space: pre-wrap;
  box-shadow: 0 1px 0 rgba(0,0,0,0.04);
}
.row.mine .bubble { background: #dbeafe; color: #0b3b7e; border-top-right-radius: 6px; }
.row.theirs .bubble { background: #f3f4f6; color: #111827; border-top-left-radius: 6px; }
.composer {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 8px;
  padding: 10px;
  border-top: 1px solid #e5e7eb;
}
.composer input {
  height: 40px; padding: 0 12px;
  border: 1px solid #d1d5db; border-radius: 10px; outline: none;
}
.composer input:focus { border-color: #7aa7ff; }
.composer button {
  height: 40px; padding: 0 16px;
  border: 1px solid #2563eb; background: #2563eb; color: #fff;
  border-radius: 10px; cursor: pointer;
}
.composer button:hover { background: #1d4ed8; }

/* empty state */
.no-matches {
  display:flex;
  align-items:center;
  justify-content:center;
  min-height: 50vh;
  color:#6b7280;
  font-size: 18px;
}
</style>
