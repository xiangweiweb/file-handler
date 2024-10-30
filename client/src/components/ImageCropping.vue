<!-- 
 https://juejin.cn/post/7391160093565976616?searchId=202410252117510918920329E9C7866C2A

 Cropper.js
 -->
<script setup>
import { ref, onMounted } from 'vue'

const fileInput = ref();
const sourceImgCanvas = ref();
const targetImgCanvas = ref();

let sourceImgCanvasCtx = null;
let targetImgCanvasCtx = null;

onMounted(() => {
    sourceImgCanvasCtx = sourceImgCanvas.value.getContext('2d');
    targetImgCanvasCtx = targetImgCanvas.value.getContext('2d');
});

const image = new Image();

// 将上传的图片用canvas绘制出来
const changeFile = (event) => {
    const file = event.target.files[0];
    image.src = URL.createObjectURL(file);
    image.onload = () => {
        console.log(image);
        const { width, height } = image;
        sourceImgCanvas.value.width = width;
        sourceImgCanvas.value.height = height;
        sourceImgCanvasCtx.drawImage(image, 0, 0, width, height);

        // 同时裁剪原图中坐标0,0，width和height都为100的照片
        // const croppingImageData = sourceImgCanvasCtx.getImageData(0,0,100,100);
        // targetImgCanvasCtx.putImageData(croppingImageData, 0, 0);
    }
};

let startX = 0;
let startY = 0;
let endX = 0;
let endY = 0;
let isDragging = false;

const drawSelectRect = () => {
    sourceImgCanvasCtx.clearRect(0, 0, sourceImgCanvas.value.width, sourceImgCanvas.value.height);
    sourceImgCanvasCtx.drawImage(image, 0 ,0 );
    if(isDragging) {
        sourceImgCanvasCtx.strokeStyle = 'red';
        sourceImgCanvasCtx.lineWidth = 2;
        sourceImgCanvasCtx.strokeRect(startX, startY, endX - startX, endY - startY);
    }
};

const clip = () => {
    const cropWidth = endX - startX;
    const cropHeight = endY - startY;
    console.log(`clip endX ${endX}, endY ${endY}; startX ${startX}, endY ${startY}; width ${cropWidth}, height ${cropHeight}`);
    const imageData = sourceImgCanvasCtx.getImageData(startX, startY, cropWidth, cropHeight);

    targetImgCanvas.value.width = cropWidth;
    targetImgCanvas.value.height = cropHeight;
    targetImgCanvasCtx.putImageData(imageData, 0, 0);
}

const canvasMousedown = (event) => {
    startX = event.offsetX;
    startY = event.offsetY;
    isDragging = true;
};
const canvasMousemove = (event) => {
    if(isDragging) {
        endX = event.offsetX;
        endY = event.offsetY;
        drawSelectRect();
    }
};
const canvasMouseup = (event) => {
    isDragging = false;
    endX = event.offsetX;
    endY = event.offsetY;
    console.log(`mouseup endX ${endX}, endY ${endY}`);
    drawSelectRect();
    clip();
};

</script>

<template>
    <input type="file" ref="fileInput" @change="changeFile"/><br/>
    <canvas ref="sourceImgCanvas" 
        @mousedown="canvasMousedown"
        @mousemove="canvasMousemove"
        @mouseup="canvasMouseup">
    </canvas>
    <canvas ref="targetImgCanvas"></canvas>
</template>

<style>

canvas {
    border: 1px solid black;
    margin: 10px;
}

</style>