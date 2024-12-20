<template>
  <div>
    <video ref="video" width="720" height="560" autoplay></video>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import * as faceapi from "face-api.js";

defineOptions({
  name: "FaceApi",
});

const video = ref<HTMLVideoElement | null>(null);
const MODELS_URL = "/models";

const startVideo = () => {
  if (video.value != null) {
    navigator.mediaDevices
      .getUserMedia({ video: {} })
      .then((stream) => {
        (video.value as HTMLVideoElement).srcObject = stream;
      })
      .catch((err) => console.error(err));
  }
};

const loadModels = async () => {
  try {
    await faceapi.nets.tinyFaceDetector.loadFromUri(MODELS_URL);
    await faceapi.nets.faceLandmark68Net.loadFromUri(MODELS_URL);
    await faceapi.nets.faceRecognitionNet.loadFromUri(MODELS_URL);
    await faceapi.nets.faceExpressionNet.loadFromUri(MODELS_URL);
  } catch (error) {
    console.error("Error loading models: ", error);
  }
};

onMounted(() => {
  startVideo();
  loadModels();
});
</script>
