<script setup>
import { ref } from "vue";
import { mqtt_v5 as u8mqtt } from "u8-mqtt";

const messages = ref([]);
const input = ref(null)

const my_mqtt = u8mqtt({
  log_conn(event) {
    switch (event) {
      case 'on_disconnect':
        console.log('disconnected')
        break
      case 'on_live':
        console.log('async queue started')
        break
      case 'on_ready':
        console.log('connection handshake complete')
        break
      default:
    }
  }
}).with_websock("ws://localhost:8080").with_autoreconnect()

async function init() {
  await my_mqtt.connect();

  my_mqtt.subscribe_topic("u8-mqtt/demo-simple/:topic", (pkt, params, ctx) => {
    messages.value.push(pkt.json());
  });
}

init();

async function publishMessage(message) {
  await my_mqtt.json_send("u8-mqtt/demo-simple/live", {
    note: message,
    live: new Date().toISOString(),
  });

  input.value.value = ''
}
</script>

<template>
  <div>
    <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <label for="prompt"> Enter message and press ENTER </label><br />
  <input
    id="prompt"
    type="text"
    @change="publishMessage($event.target.value)"
    ref="input"
  />
  <ul>
    <li v-for="message in messages" :key="message.live">
      {{ message.note }}
    </li>
  </ul>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
