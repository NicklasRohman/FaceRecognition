<template>
  <div>
    <video ref="video" width="720" height="560" autoplay></video>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import * as faceapi from "face-api.js";

defineOptions({
  name: "FaceApi",
});

const video = ref<HTMLVideoElement | null>(null);
const canvas = ref<HTMLCanvasElement | null>(null);

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
    const MODELS_URL = "./public/models";
    await Promise.all([
      faceapi.nets.ssdMobilenetv1.loadFromUri(MODELS_URL),
      faceapi.nets.faceLandmark68Net.loadFromUri(MODELS_URL),
      faceapi.nets.faceRecognitionNet.loadFromUri(MODELS_URL),
      faceapi.nets.faceExpressionNet.loadFromUri(MODELS_URL),
    ]).then((res) => {
      console.log(res);
    });
  } catch (error) {
    console.error("Error loading models: ", error);
  }
};

const detectFaces = async () => {
  try {
    if (video.value && canvas.value) {
      const displaySize = {
        width: video.value.width,
        height: video.value.height,
      };

      faceapi.matchDimensions(canvas.value, displaySize);

      setInterval(async () => {
        if (video.value) {
          const detections = await faceapi
            .detectAllFaces(video.value, new faceapi.TinyFaceDetectorOptions())
            .withFaceLandmarks()
            .withFaceExpressions();
          const resizedDetections = faceapi.resizeResults(
            detections,
            displaySize
          );

          if (canvas.value) {
            canvas.value
              ?.getContext("2d")
              ?.clearRect(0, 0, canvas.value.width, canvas.value.height);
            faceapi.draw.drawDetections(canvas.value, resizedDetections);
            faceapi.draw.drawFaceLandmarks(canvas.value, resizedDetections);
            faceapi.draw.drawFaceExpressions(canvas.value, resizedDetections);
          }
        }
      }, 100);
    }
  } catch (error) {
    console.error("Error detecting faces:", error);
  }
  requestAnimationFrame(detectFaces);
};

onMounted(() => {
  startVideo();
  loadModels().then(() => {
    detectFaces();
  });
});
</script>
