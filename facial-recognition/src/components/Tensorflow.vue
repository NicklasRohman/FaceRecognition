<template>
  <div>
    <video ref="video" autoplay></video>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import * as faceDetection from "@tensorflow-models/face-detection";
import "@mediapipe/face_detection";

export default defineComponent({
  name: "FaceRecognition",
  setup() {
    const video = ref<HTMLVideoElement | null>(null);
    const canvas = ref<HTMLCanvasElement | null>(null);
    let detector: faceDetection.FaceDetector | null = null;

    const initializeDetector = async () => {
      try {
        const model = faceDetection.SupportedModels.MediaPipeFaceDetector;
        const detectorConfig: faceDetection.MediaPipeFaceDetectorMediaPipeModelConfig =
          {
            runtime: "mediapipe" as const,
            solutionPath:
              "https://cdn.jsdelivr.net/npm/@mediapipe/face_detection@0.4.1646425229/face_detection.min.js",
          };
        detector = await faceDetection.createDetector(model, detectorConfig);
      } catch (error) {
        console.error("Error initializing detector:", error);
      }
    };

    const startVideoStream = async () => {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({
          video: true,
        });
        if (video.value) {
          video.value.srcObject = stream;
        }
      } catch (error) {
        console.error("Error starting video stream:", error);
      }
    };

    const detectFaces = async () => {
      try {
        if (video.value && detector) {
          const faces = await detector.estimateFaces(video.value);
          drawFaces(faces);
        }
      } catch (error) {
        console.error("Error detecting faces:", error);
      }
      requestAnimationFrame(detectFaces);
    };

    const drawFaces = (faces: faceDetection.Face[]) => {
      if (canvas.value) {
        const ctx = canvas.value.getContext("2d");
        if (ctx) {
          ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
          faces.forEach((face) => {
            ctx.beginPath();
            ctx.lineWidth = 2;
            ctx.strokeStyle = "red";
            ctx.rect(
              face.box.xMin,
              face.box.yMin,
              face.box.width,
              face.box.height
            );
            ctx.stroke();
          });
        }
      }
    };

    onMounted(async () => {
      if (video.value && canvas.value) {
        await initializeDetector();
        await startVideoStream();
        detectFaces();
      }
    });

    return { video, canvas };
  },
});
</script>

<style scoped>
video,
canvas {
  width: 100%;
  height: auto;
}
</style>
