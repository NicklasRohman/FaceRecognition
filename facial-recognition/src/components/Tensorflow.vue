<template>
  <div>This is tensorflow</div>
  <div>
    <video ref="video" autoplay></video>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import * as faceLandmarksDetection from "@tensorflow-models/face-landmarks-detection";

export default defineComponent({
  name: "Tensorflow",
  setup() {
    const video = ref<HTMLVideoElement | null>(null);
    const canvas = ref<HTMLCanvasElement | null>(null);

    onMounted(async () => {
      if (video.value && canvas.value) {
        const model = faceLandmarksDetection.SupportedModels.MediaPipeFaceMesh;

        const detectorConfig: faceLandmarksDetection.MediaPipeFaceMeshMediaPipeModelConfig =
          {
            runtime: "mediapipe" as const,
            solutionPath: "https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh",
            refineLandmarks: true,
          };
        const detector = await faceLandmarksDetection.createDetector(
          model,
          detectorConfig
        );

        // Start video stream
        navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
          if (video.value) {
            video.value.srcObject = stream;
          }
        });

        // Detect faces
        const detectFaces = async () => {
          if (video.value) {
            const predictions = await detector.estimateFaces(video.value);

            // Draw predictions on canvas
            if (canvas.value) {
              const ctx = canvas.value.getContext("2d");
              if (ctx) {
                ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
                predictions.forEach((prediction: any) => {
                  ctx.beginPath();
                  ctx.lineWidth = 2;
                  ctx.strokeStyle = "red";
                  ctx.rect(
                    prediction.boundingBox.topLeft[0],
                    prediction.boundingBox.topLeft[1],
                    prediction.boundingBox.bottomRight[0] -
                      prediction.boundingBox.topLeft[0],
                    prediction.boundingBox.bottomRight[1] -
                      prediction.boundingBox.topLeft[1]
                  );
                  ctx.stroke();
                });
              }
            }
          }
          requestAnimationFrame(detectFaces);
        };

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
