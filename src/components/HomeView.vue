<template>
  <div id="app">
    <div class="videoContainer">
      <div>
        Local
        <video ref="localVideo" autoplay playsinline></video>
      </div>
      <div>
        Remote
        <video ref="remoteVideo" autoplay playsinline></video>
      </div>
    </div>
    <div class="buttonWrapper">
      <div class="buttonContainer">
        <p><b>| offer側の操作</b></p>
        <button @click="createOffer">① Offerを生成</button>
        <button @click="setAnswer">④ Answerを登録</button>
        <button @click="copyIceCandidate">⑤ ICE経路情報をコピー</button>
      </div>
      <div class="buttonContainer">
        <p><b>| answer側の操作</b></p>
        <button @click="setOffer">② Offerを登録</button>
        <button @click="createAnswer">③ Answerを生成</button>
        <button @click="setIceCandidate">⑥ ICE経路情報を登録</button>
      </div>
    </div>
    <div class="infoContainer">
      <textarea v-model="input" placeholder="手動接続用の入力欄"></textarea>
      <textarea ref="textContainer"></textarea>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const localVideo = ref(null);
const remoteVideo = ref(null);
const input = ref("");
const textContainer = ref("");
let localStream = null;
let peerConnection = null;
let iceCandidates = [];

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
      iceCandidates.push(event.candidate);
      console.log("New ICE candidate: ", JSON.stringify(event.candidate));
    }
  };
};

const copyToClipboard = (data, message = { success: "", fail: "" }) => {
  navigator.clipboard
    .writeText(data)
    .then(() => (textContainer.value = message.success))
    .catch((e) => (textContainer.value = message.fail + data));
};

const createOffer = () => {
  peerConnection
    .createOffer()
    .then((offer) => peerConnection.setLocalDescription(offer))
    .then(() => {
      console.log(peerConnection.localDescription);
      copyToClipboard(JSON.stringify(peerConnection.localDescription), {
        success:
          "SDP Offerをクリップボードにコピーしました。\nもう1つウィンドウで入力欄に貼り付け「② Offerを登録」を押してください。",
        fail: "● 以下の文字列をコピーし、もう1つウィンドウで入力欄に貼り付け「② Offerを登録」を押してください。\n\n",
      });
    });
};

const setAnswer = async () => {
  const answer = input.value;
  try {
    await peerConnection.setRemoteDescription(JSON.parse(answer)).then(() => {
      textContainer.value = "Answerを登録しました。";
    });
  } catch (e) {
    textContainer.value =
      "Answerの登録に失敗しました。\n正しくSDP Answerがコピー・ペーストできているか確認してください。";
  }
};

const copyIceCandidate = () => {
  copyToClipboard(JSON.stringify(iceCandidates), {
    success:
      "ICE Candidateをクリップボードにコピーしました。\nもう1つウィンドウで入力欄に貼り付け「⑥ ICE経路情報を登録」を押してください。",
    fail: "● 以下の文字列をコピーし、もう1つウィンドウで入力欄に貼り付け「⑥ ICE経路情報を登録」を押してください。\n\n",
  });
};

const createAnswer = async () => {
  try {
    await peerConnection
      .createAnswer()
      .then((answer) => peerConnection.setLocalDescription(answer))
      .then(() => {
        console.log(peerConnection.localDescription);
        copyToClipboard(JSON.stringify(peerConnection.localDescription), {
          success:
            "SDP Answerをクリップボードにコピーしました。\nもう1つウィンドウで入力欄に貼り付け「④ Answerを登録」を押してください。",
          fail: "● 以下の文字列をコピーし、もう1つウィンドウで入力欄に貼り付け「④ Answerを登録」を押してください。\n\n",
        });
      });
  } catch (e) {
    console.error(e);
    textContainer.value =
      "Answerの生成に失敗しました。\n正しくOfferが登録できているか確認してください。";
  }
};

const setOffer = async () => {
  const offer = input.value;
  try {
    await peerConnection.setRemoteDescription(JSON.parse(offer)).then(() => {
      textContainer.value = "Offerを登録しました。";
    });
  } catch (e) {
    textContainer.value =
      "Offerの登録に失敗しました。\n正しくSDP Offerがコピー・ペーストできているか確認してください。";
  }
};

const setIceCandidate = () => {
  const candidatesStr = input.value;
  try {
    const senderCandidates = JSON.parse(candidatesStr);
    senderCandidates.forEach(async (candidate) => {
      if (candidate === null) return;
      await peerConnection.addIceCandidate(candidate);
      console.log("ICE candidate added: ", candidate);
    });
    textContainer.value = "ICE経路情報を登録しました。";
  } catch (e) {
    textContainer.value =
      "ICE経路情報の登録に失敗しました。\n正しくICE経路情報がコピー・ペーストできているか確認してください。";
  }
};

startChat();
</script>

<style>
.videoContainer {
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
.buttonWrapper {
  display: flex;
  justify-content: space-around;
  width: 100%;
}
.buttonContainer {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.infoContainer {
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
