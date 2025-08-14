<template>
  <div v-if="hasMatches" class="layout">
  <!-- LEFT: matches list -->
  <aside class="sidebar">
    <h3 class="side-title">Your Matches</h3>
    <ul class="side-list">
      <li v-for="m in matches" :key="m.user_id">
        <button class="match-btn" @click="open(m.user_id)">
          {{ m.name || m.username || ('User #' + m.user_id) }}
        </button>
      </li>
    </ul>
  </aside>


  <!-- RIGHT: chat (unchanged inside) -->
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

  <section v-else class="empty-chat">
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
/* CHAT COLUMN fills available height; thread grows and scrolls; composer sticks bottom */
.chat {
  display: grid;
  grid-template-rows: 1fr auto;   /* thread (flexible) + composer (fixed) */
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  background: #fff;
  height: 100%;                   /* fill full column height */
  min-height: 0;                  /* allow inner scroll */
  overflow: hidden;               /* clip inside to the card */
}

.thread {
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-height: 0;                  /* allow scrolling */
}

.composer {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 8px;
  padding: 10px;
  border-top: 1px solid #e5e7eb;
  background: #fff;
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
/* Page background */
:host, :root, body {
  background: linear-gradient(180deg, #fff7ec 0%, #fff 40%, #fff 100%);
}

/* Two-column layout that fills the width; left fixed, right grows */
.matches-layout {
  display: grid;
  grid-template-columns: 280px minmax(0, 1fr);
  gap: 20px;
  max-width: 1200px;
  margin: 24px auto;
  padding: 0 16px;
  box-sizing: border-box;
}

/* LEFT: Matches list card */
.matches-pane {
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 14px;
  padding: 14px;
  height: 70vh;                 /* nice tall panel */
  overflow: hidden;             /* contain the scroll area */
  display: flex;
  flex-direction: column;
}

.matches-pane h3 {
  margin: 0 0 10px;
  font-size: 18px;
}

.matches-pane ul {
  list-style: none;
  padding: 0;
  margin: 0;
  overflow-y: auto;             /* scroll only the list */
  scrollbar-width: thin;
}

.matches-pane li + li { margin-top: 8px; }

.matches-pane button {
  width: 100%;
  text-align: left;
  padding: 10px 12px;
  border: 1px solid #e5e7eb;
  background: #fff;
  border-radius: 10px;
  cursor: pointer;
  transition: background .15s ease, border-color .15s ease, transform .06s ease;
}

.matches-pane button:hover {
  background: #fafafa;
  border-color: #d7dbdf;
}

.matches-pane button:active {
  transform: translateY(1px);
}

/* RIGHT SIDE takes remaining space — your .chat styles already handle the rest */
.empty-chat {
  display: grid;
  place-items: center;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  background: #fff;
  color: #6b7280;
  height: 100%;
}

/* Full-width layout: left list + chat filling the rest */
.layout {
  width: 100%;
  max-width: none;
  margin: 0;
  padding: 16px 24px;
  display: grid;
  grid-template-columns: 260px minmax(0, 1fr); /* left fixed, right flexible */
  gap: 20px;
  box-sizing: border-box;
  background: linear-gradient(180deg, #ffffff, #fffaf1 60%, #fff7e6);
}

/* Sidebar styles (left column) */
.sidebar {
  display: flex;
  flex-direction: column;
  height: 100%;
  min-height: 0;             /* IMPORTANT: enables inner scroll areas */
}

.side-title { margin: 0 0 10px; font-weight: 700; }

.side-list {
  list-style: none;
  padding: 8px;
  margin: 0;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  background: #fff;
  flex: 1 1 auto;            /* take the remaining height */
  min-height: 0;             /* allow scrolling */
  overflow: auto;            /* list scrolls, not the whole page */
}

.side-list li { margin-bottom: 8px; }
.side-list li:last-child { margin-bottom: 0; }

.match-btn {
  width: 100%;
  text-align: left;
  padding: 10px 12px;
  border: 1px solid transparent;
  border-radius: 10px;
  background: #f9fafb;
  cursor: pointer;
  transition: background .15s, border-color .15s;
}
.match-btn:hover { background:#f3f4f6; border-color:#e5e7eb; }

/* Keep your existing .chat, .thread, .row, .bubble, .composer styles as-is */
/* OUTER LAYOUT: full width + full viewport height (minus navbar if any) */
.layout {
  display: grid;
  grid-template-columns: 260px minmax(0, 1fr); /* left fixed, right grows */
  gap: 20px;
  width: 100%;
  height: calc(100vh - 64px); /* adjust 64px to your navbar height; try 0 if none */
  box-sizing: border-box;
  padding: 16px 24px;
  background: linear-gradient(180deg, #ffffff, #fffaf1 60%, #fff7e6);
}

</style>
