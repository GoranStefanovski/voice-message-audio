<template>
  <div>
    <button @click="startRecording(1)" :disabled="isRecording1">Start Recording Version 1</button>
    <button @click="stopRecording(1)" :disabled="!isRecording1">Stop Recording Version 1</button>
    <br><br>
    <button @click="startRecording(2)" :disabled="isRecording2">Start Recording Version 2</button>
    <button @click="stopRecording(2)" :disabled="!isRecording2">Stop Recording Version 2</button>
    <br><br>
    <button v-if="recordChunks1.length > 0 && recordChunks2.length > 0" @click="combineRecordings">Combine Recordings</button>
    <br><br>
    <button v-if="mp3TmpUrl && isCombined" @click="playCombinedRecording">Play Combined Recording</button>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';

let mediaRecorder;
let recordChunks1 = reactive([]);
let recordChunks2 = reactive([]);
let isRecording1 = ref(false);
let isRecording2 = ref(false);
let mp3TmpUrl = ref(null);
let combinedAudio = ref(null);
let isCombined = ref(false);

const saveChunkToRecording = (event, version) => {
  if (version === 1) {
    recordChunks1.push(event.data);
  } else if (version === 2) {
    recordChunks2.push(event.data);
  }
};

const saveRecording = async () => {
  const allChunks = [...recordChunks1, ...recordChunks2];
  const combinedBlob = new Blob(allChunks, { type: 'audio/mp3' });
  mp3TmpUrl.value = URL.createObjectURL(combinedBlob);
};

async function stopRecording(version) {
  if (mediaRecorder) {
    mediaRecorder.stop();
    mediaRecorder.stream.getTracks().forEach(track => {
      track.stop();
    });

    if (version === 1) {
      isRecording1.value = false;
    } else if (version === 2) {
      isRecording2.value = false;
    }
  }
}

async function startRecording(version) {
  isCombined = false;
  if (version === 1) {
    recordChunks1 = [];
  } else if (version === 2) {
    recordChunks2 = [];
  }

  try {
    navigator.mediaDevices.getUserMedia({
      audio: true 
    }).then(stream => {
      if (version === 1) {
        isRecording1.value = true;
      } else if (version === 2) {
        isRecording2.value = true;
      }

      mediaRecorder = new MediaRecorder(stream);

      mediaRecorder.ondataavailable = (event) => saveChunkToRecording(event, version);
      mediaRecorder.onstop = saveRecording;
      mediaRecorder.start();

    });

  } catch (error) {
    console.error('Error starting the record: ', error);
  }
}

function combineRecordings() {
  const allChunks = [...recordChunks1, ...recordChunks2];
  const combinedBlob = new Blob(allChunks, { type: 'audio/mp3' });
  mp3TmpUrl.value = URL.createObjectURL(combinedBlob);
  isCombined = true;
}

function playCombinedRecording() {
  if (mp3TmpUrl.value) {
    combinedAudio.value = new Audio(mp3TmpUrl.value);
    combinedAudio.value.play();
  }
}
</script>
