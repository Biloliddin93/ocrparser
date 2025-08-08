<template>
  <div class="camera-wrapper">
    <!-- Video preview -->
    <video ref="video" autoplay playsinline class="camera-feed"></video>

    <!-- ROI overlay -->
    <canvas ref="overlay" class="overlay"></canvas>

    <!-- Capture button -->
    <button @click="capture">📸 Suratga olish</button>

    <!-- Cropped results -->
    <div class="results">
      <div
          v-for="(img, idx) in croppedImages"
          :key="idx"
          class="crop-preview"
      >
        <img :src="img" alt="Cropped ROI" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";

const video = ref(null);
const overlay = ref(null);
const croppedImages = ref([]);

// ROI lar — nisbiy (proportsional) formatda
// 0-1 oralig'ida: x, y, w, h => videoWidth * x, videoHeight * y ...
const rois = [
  { x: 0.33, y: 0.13, w: 0.33, h: 0.070, name: "familiya" },
  { x: 0.33, y: 0.22, w: 0.33, h: 0.070, name: "ism" },
  { x: 0.33, y: 0.30, w: 0.33, h: 0.070, name: "otasining ismi" },
  { x: 0.33, y: 0.39, w: 0.24, h: 0.070, name: "tugilgan sana" },
  { x: 0.33, y: 0.49, w: 0.24, h: 0.070, name: "berilgan sana" },
  { x: 0.33, y: 0.575, w: 0.24, h: 0.070, name: "amal qilish muddati" },
  { x: 0.82, y: 0.39, w: 0.15, h: 0.065, name: "jinsi" },
  { x: 0.69, y: 0.47, w: 0.27, h: 0.070, name: "fuqaroligi" },
  { x: 0.69, y: 0.56, w: 0.27, h: 0.070, name: "karta raqami" },
  { x: 0.05, y: 0.13, w: 0.27, h: 0.35, name: "foto" },
  { x: 0.13, y: 0.51, w: 0.16, h: 0.1, name: "imzo" }
];

onMounted(() => {
  startCamera();
});

function startCamera() {
  navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
    video.value.srcObject = stream;
    video.value.onloadedmetadata = () => {
      drawOverlay();
    };
  });
}

function drawOverlay() {
  const ctx = overlay.value.getContext("2d");

  function render() {
    const rect = video.value.getBoundingClientRect();
    overlay.value.width = rect.width;
    overlay.value.height = rect.height;

    ctx.clearRect(0, 0, overlay.value.width, overlay.value.height);
    ctx.strokeStyle = "red";
    ctx.lineWidth = 2;

    rois.forEach((roi) => {
      ctx.strokeRect(
          roi.x * rect.width,
          roi.y * rect.height,
          roi.w * rect.width,
          roi.h * rect.height
      );
    });

    requestAnimationFrame(render);
  }

  render();
}

function capture() {
  const tempCanvas = document.createElement("canvas");
  tempCanvas.width = video.value.videoWidth;
  tempCanvas.height = video.value.videoHeight;

  const ctx = tempCanvas.getContext("2d");
  ctx.drawImage(video.value, 0, 0);

  croppedImages.value = rois.map((roi) => {
    const cropCanvas = document.createElement("canvas");
    cropCanvas.width = roi.w * tempCanvas.width;
    cropCanvas.height = roi.h * tempCanvas.height;

    const cropCtx = cropCanvas.getContext("2d");
    cropCtx.drawImage(
        tempCanvas,
        roi.x * tempCanvas.width,
        roi.y * tempCanvas.height,
        roi.w * tempCanvas.width,
        roi.h * tempCanvas.height,
        0,
        0,
        roi.w * tempCanvas.width,
        roi.h * tempCanvas.height
    );

    return cropCanvas.toDataURL("image/png");
  });
}
</script>

<style>
.camera-wrapper {
  position: relative;
  width: 100%;
  max-width: 800px;
}
.camera-feed {
  width: 100%;
  height: auto;
}
.overlay {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
}
.results {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 10px;
}
.crop-preview img {
  border: 1px solid #ccc;
  height: 60px;
}
</style>
