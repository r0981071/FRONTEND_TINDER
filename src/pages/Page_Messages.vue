<template>
  <div style="display:grid; grid-template-columns: 220px 1fr; gap:16px; max-width:960px; margin:0 auto;">
    <aside>
      <h3>Your Matches</h3>
      <ul style="list-style:none; padding:0;">
        <li v-for="m in matches" :key="m.user_id" style="margin-bottom:8px;">
          <button @click="open(m.user_id)" style="width:100%;">{{ m.name || m.username }}</button>
        </li>
      </ul>
    </aside>

    <section v-if="activeId">
      <div style="border:1px solid #ddd; height:55vh; overflow:auto; padding:8px; text-align:left;">
        <div v-for="msg in messages" :key="msg.message_id" style="margin:6px 0;">
          <b>{{ msg.sender_id === user.user_id ? 'You' : 'Them' }}:</b> {{ msg.content }}
        </div>
      </div>
      <form @submit.prevent="send" style="display:flex; gap:8px; margin-top:8px;">
        <input v-model="text" placeholder="Type a message..." style="flex:1;" />
        <button type="submit">Send</button>
      </form>
    </section>

    <section v-else style="display:flex; align-items:center; justify-content:center;">
      <p>Select a match to chat</p>
    </section>
  </div>
</template>

<script>
export default {
  name:"Page_Messages",
  props:{ user:{ type:Object, required:true } },
  data(){
    return {
      matches: [],
      activeId: null,
      conversation: null,
      messages: [],
      text: ""
    };
  },
  methods:{
    async loadMatches(){
      const res = await fetch(`http://localhost:3000/api/matches/${this.user.user_id}`);
      this.matches = res.ok ? await res.json() : [];
    },
    async open(otherId){
      this.activeId = otherId;
      const res = await fetch(`http://localhost:3000/api/conversations/with/${this.user.user_id}/${otherId}`);
      const data = await res.json(); // { conversation, messages }
      this.conversation = data.conversation;
      this.messages = data.messages || [];
    },
    async send(){
      if(!this.text.trim() || !this.conversation) return;
      const payload = {
        conversationId: this.conversation.conversation_id,
        senderId: this.user.user_id,
        receiverId: this.activeId,
        content: this.text.trim(),
      };
      const res = await fetch("http://localhost:3000/api/messages",{
        method:"POST",
        headers:{ "Content-Type":"application/json" },
        body: JSON.stringify(payload)
      });
      if(res.ok){
        const msg = await res.json();
        this.messages.push(msg);
        this.text = "";
      }
    }
  },
  mounted(){ this.loadMatches(); }
};
</script>
