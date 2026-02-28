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

    // Sinkronkan ukuran canvas dengan video setelah metadata video dimuat
    videoRef.value.addEventListener('loadedmetadata', () => {
        canvasRef.value.width = videoRef.value.videoWidth;
        canvasRef.value.height = videoRef.value.videoHeight;
    });

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
        <video ref="videoRef" autoplay muted playsinline></video>
        <canvas ref="canvasRef"></canvas>
    </div>
</template>

<style scoped>
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

.container {
    position: relative;
    width: 100%;
    max-width: 100vw;
    overflow: hidden;
}

video {
    display: block;
    width: 100%;
    height: auto;
}

canvas {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
</style>