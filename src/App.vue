<script setup>
import { ref as vueRef } from 'vue';
import app from '../firebase_config.js';
import { getDatabase, ref, push, set } from "firebase/database";
import ConfettiExplosion from "vue-confetti-explosion";


const db = getDatabase(app);
const query = vueRef('');
const visible = vueRef(false);
const slowDown = vueRef(false);
const yt_regex = '^(https?://)?(www\.)?(youtube\.com|youtu\.?be)/.+$';
const id_regex = '(?<=v=|v\/|vi=|vi\/|youtu\.be\/|\/v\/|embed\/)[^#\&\?]*';

const clientId = localStorage.getItem('clientId') || generateUUID();
let timestamp = parseInt(localStorage.getItem('timestamp') || 1691769448);
localStorage.setItem('timestamp', timestamp);
localStorage.setItem('clientId', clientId);

function generateUUID() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        const r = (Math.random() * 16) | 0,
            v = c === 'x' ? r : (r & 0x3) | 0x8;
        return v.toString(16);
    });
}

const explode = () => {
  visible.value = true;
  setTimeout(() => {
    visible.value = false;
  }, 3000);
}

const addURL = (e) => {
  e.preventDefault();
  if (query.value.trim() !== '' && query.value.match(yt_regex)) {
    const id = query.value.trim().match(id_regex)[0];
    console.log(timestamp + 60000, Date.now())
    if (id.length !== 11 || timestamp + 60000 > Date.now()) {
      console.log(timestamp + 60000, Date.now())
      slowDown.value = id.length == 11 ? true : false
      console.log(slowDown.value)
      query.value = '';
      return;
    }
    explode();
    slowDown.value = false;
    const queueRef = ref(db, 'queue');
    let newEntryRef = push(queueRef);
    const obj = {
      clientId,
      timestamps: Date.now(),
      youtubeId: id,
      status: 'pending'
    }
    console.log(obj)
    set(newEntryRef, obj);
    timestamp = Date.now();
    localStorage.setItem('timestamp',timestamp);
  } else {
    console.warn("URL is empty or not in the format wanted.");
  }
  query.value = '';
}
</script>


<template>
  <h1>GROVER VISUALIZER</h1>
  <div class="conf">
    <ConfettiExplosion v-if="visible" />
  </div>
  <div class="warning" v-if="slowDown">
    <div class="warning-txt">
      <p >🤠🤠Slow Down Cowboy🤠🤠</p>
      <span>(or I'll violently murder you in your sleep!!)</span>
    </div>
    <!-- add gif called fortnite-dance -->
    <img width="200" height="200" src="./assets/fortnite-dance.gif"/>
    <img width="200" height="200" src="./assets/fortnite-dance2.gif"/>
    <img width="200" height="200" src="./assets/fortnite-dance3.gif"/>
  </div>
  <form @submit="addURL">
    <label for="query">Enter a YouTube URL:</label>
    <div class="form-container">
      <input v-model="query" type="text" name="query" id="query" placeholder="Enter a youtube URL">
      <input type="submit" value="Add To Queue">
    </div>
  </form>
</template>

<style scoped>

.conf {
  position: absolute;
  top : 55%;
  left: 50%;
  transform: translate(-50%, -50%);
}

label {
  font-size: 1.5rem;
  margin: 5px;
}

p {
  color: red;
  font-size: 1.75rem;
  margin: 5px;
}

form {
  margin: 5px;
  display: flex;
  flex-direction: column;
}

input {
  padding: 10px;
}
</style>
