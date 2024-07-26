<script setup>
import { io } from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

// Import the logo image
import logo from './logo.png';

const socket = io('http://localhost:3000');

const messages = ref([]);
const messageText = ref('');
const joined = ref(false);
const name = ref('');
const typingDisplay = ref('');

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) => {
    messages.value = response;
  });

  socket.on('message', (message) => {
    messages.value.push(message);
  });

  socket.on('typing', ({ name, isTyping }) => {
    typingDisplay.value = isTyping ? `${name} is typing...` : '';
  });
});

const join = () => {
  socket.emit('join', { name: name.value }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  if (messageText.value.trim()) {
    socket.emit('createMessage', { text: messageText.value }, () => {
      messageText.value = '';
    });
  }
};

let timeout;
const emitTyping = () => {
  socket.emit('typing', { isTyping: true });
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    socket.emit('typing', { isTyping: false });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <div v-if="!joined" class="join-form">
      <img :src="logo" alt="Chat App Logo" class="logo"/>
      <h2>Welcome, Type your username</h2>
      <form @submit.prevent="join">
        <div class="input-group">
          <input id="name" v-model="name" placeholder="Enter your name" />
        </div>
        <div class="button-group">
          <button type="submit" class="btn">Join Chat</button>
        </div>
      </form>
    </div>

    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages" :key="message.id" class="message" :class="{ 'message-sent': message.name === name, 'message-received': message.name !== name }">
          <strong>👤 {{ message.name }}</strong>: {{ message.text }}
        </div>
      </div>

      <div v-if="typingDisplay" class="typing-indicator">{{ typingDisplay }}</div>

      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <input
            v-model="messageText"
            @input="emitTyping"
            placeholder="Type a message..."
          />
          <button type="submit" class="btn">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import './assets/base.css';

.chat {
  display: flex;
  flex-direction: column;
  height: 100vh;
  max-width: 600px;
  margin: 0 auto;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  background: #f9f9f9;
}

.join-form {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100%;
  text-align: center;
}

.logo {
  width: 150px; /* Adjust width as needed */
  margin-bottom: 20px;
}

h2 {
  margin-bottom: 20px;
  font-size: 20px;
  color: black;
}

.input-group, .button-group {
  margin: 10px 0;
  display: flex;
  justify-content: center;
  width: 100%;
}

input {
  padding: 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
  width: 80%; /* Adjust width as needed */
}

input:focus {
  outline: none;
  border-color: #007bff;
}

button {
  background-color: #25D366; /* WhatsApp green */
  color: #fff;
  cursor: pointer;
  padding: 10px 20px;
  border-radius: 8px; /* Added radius */
  border: none;
  width: 150px; /* Increased width */
}

button:hover {
  background-color: #128C7E; /* Darker green on hover */
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messages-container {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
  border-bottom: 1px solid #ddd;
  display: flex;
  flex-direction: column;
}

.message {
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 10px;
  max-width: 75%;
  word-wrap: break-word;
  display: flex;
  align-items: center;
}

.message-sent {
  background-color: #007bff;
  color: #fff;
  align-self: flex-end;
  margin-left: auto;
  width: 100%;
  text-align: right;
}

.message-received {
  background-color: #e1ffc7;
  color: #000;
  align-self: flex-start;
  width: 100%;
  text-align: left;
}

.typing-indicator {
  padding: 5px;
  color: #888;
  border-top: 1px solid #ddd;
}

.message-input {
  padding: 10px;
  background-color: #fff;
  border-top: 1px solid #ddd;
}

.message-input input {
  width: calc(100% - 70px);
  display: inline-block;
  vertical-align: top;
}

.message-input button {
  width: 60px;
  display: inline-block;
  vertical-align: top;
}
</style>
