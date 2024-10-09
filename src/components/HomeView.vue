<template>
  <div class="video-container">
    <video ref="localVideo" autoplay playsinline muted></video>
    <video ref="remoteVideo" autoplay playsinline></video>
    <button @click="startChat">Start Chat</button>
    <div>
      <textarea v-model="offer" placeholder="Paste offer here"></textarea>
      <button @click="createOffer">Create Offer</button>
      <textarea v-model="answer" placeholder="Paste answer here"></textarea>
      <button @click="createAnswer">Create Answer</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const localVideo = ref(null);
const remoteVideo = ref(null);
const offer = ref("");
const answer = ref("");
let localStream = null;
let peerConnection = null;

const startChat = async () => {
  localStream = await navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true,
  });
  localVideo.value.srcObject = localStream;

  peerConnection = new RTCPeerConnection();

  localStream
    .getTracks()
    .forEach((track) => peerConnection.addTrack(track, localStream));

  peerConnection.ontrack = (event) => {
    remoteVideo.value.srcObject = event.streams[0];
  };

  peerConnection.onicecandidate = (event) => {
    if (event.candidate) {
      console.log(
        "New ICE candidate: ",
        JSON.stringify(peerConnection.localDescription)
      );
    }
  };
};

const createOffer = async () => {
  const offerDescription = await peerConnection.createOffer();
  await peerConnection.setLocalDescription(offerDescription);
  offer.value = JSON.stringify(offerDescription);
};

const createAnswer = async () => {
  const offerDescription = JSON.parse(offer.value);
  await peerConnection.setRemoteDescription(offerDescription);

  const answerDescription = await peerConnection.createAnswer();
  await peerConnection.setLocalDescription(answerDescription);
  answer.value = JSON.stringify(answerDescription);
};
</script>

<style>
.video-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}
video {
  width: 100%;
  max-width: 400px;
  border: 1px solid #ccc;
  border-radius: 8px;
}
textarea {
  width: 100%;
  max-width: 400px;
  height: 100px;
  margin: 10px 0;
}
</style>
