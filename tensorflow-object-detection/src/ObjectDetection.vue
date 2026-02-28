<script setup>
import { ref, onMounted } from 'vue';
import * as tf from '@tensorflow/tfjs';
import * as cocoSsd from '@tensorflow-models/coco-ssd';

const videoRef = ref(null);
const canvasRef = ref(null);
let model = null;

// 1. Inisialisasi Kamera & Model
onMounted(async () => {
  const stream = await navigator.mediaDevices.getUserMedia({
    audio: false,
    video: { facingMode: 'environment' }
  });
  
  videoRef.value.srcObject = stream;
  
  // Load model COCO-SSD
  model = await cocoSsd.load();
  
  // Mulai deteksi
  predictInternal();
});

// 2. Loop Deteksi
const predictInternal = async () => {
  const predictions = await model.detect(videoRef.value);
  renderPredictions(predictions);
  
  // Menggunakan requestAnimationFrame agar performa mulus
  requestAnimationFrame(predictInternal);
};

// 3. Gambar Hasil ke Canvas
const renderPredictions = (predictions) => {
  const ctx = canvasRef.value.getContext('2d');
  ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
  
  const font = "16px sans-serif";
  ctx.font = font;
  ctx.textBaseline = "top";

  predictions.forEach(prediction => {
    const x = prediction.bbox[0];
    const y = prediction.bbox[1];
    const width = prediction.bbox[2];
    const height = prediction.bbox[3];

    // Gambar Kotak
    ctx.strokeStyle = "#00FFFF";
    ctx.lineWidth = 4;
    ctx.strokeRect(x, y, width, height);

    // Gambar Label
    ctx.fillStyle = "#00FFFF";
    const textWidth = ctx.measureText(prediction.class).width;
    const textHeight = parseInt(font, 10);
    ctx.fillRect(x, y, textWidth + 4, textHeight + 4);

    ctx.fillStyle = "#000000";
    ctx.fillText(prediction.class, x, y);
  });
};
</script>

<template>
  <div class="container">
    <video 
      ref="videoRef" 
      autoplay 
      muted 
      width="640" 
      height="480"
    ></video>
    <canvas 
      ref="canvasRef" 
      width="640" 
      height="480"
    ></canvas>
  </div>
</template>

<style scoped>
.container { position: relative; }
video, canvas {
  position: absolute;
  left: 0;
  top: 0;
}
</style>