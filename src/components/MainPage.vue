<template>
  <loading-screen v-if="isImageProceeding" />
  <photo-picker
    v-if="openModal"
    @update:photo="(val:string|undefined)=>{
    if (val)
      imageSrc = val
    openModal = false
  }"
  />
  <div
    :style="{
      padding: '2rem',
      display: 'flex',
      flexDirection: 'column',
      gap: '1rem',
    }"
  >
    <div
      :style="{
        display: 'flex',
        flexDirection: 'row',
        gap: '1rem',
        position: 'sticky',
        top: '0',
        zIndex: '3',
      }"
    >
      <div>
        <color-picker
          :color="blushColor"
          @update:color="
            (val) => {
              blushColor = val;
            }
          "
        />
        <vue-slider
          v-model="blushOpacity"
          :min="0"
          :max="1"
          :interval="0.01"
          tooltip="none"
        />
      </div>
      <label class="custom-file-input">
        <input
          type="file"
          @change="handleFileChange"
          style="display: none"
          accept="image/*"
        />
        <upload-svg-icon />
      </label>
      <button
        class="custom-file-input"
        @click="
          () => {
            openModal = true;
          }
        "
      >
        <photo-svg-icon />
      </button>
    </div>
    <div class="container">
      <img :src="imageSrc" class="image" id="stockImg" @load="getLandmarks" />
      <canvas
        v-show="!isImageProceeding"
        ref="canvas"
        class="canvas"
        :style="{ opacity: blushOpacity }"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import * as faceapi from "face-api.js";
import VueSlider from "vue-slider-component";
import ColorPicker from "./ColorPicker.vue";
import LoadingScreen from "./LoadingScreen.vue";
import PhotoPicker from "./PhotoPicker.vue";

import UploadSvgIcon from "../assets/icons/UploadSvgIcon.vue";
import PhotoSvgIcon from "../assets/icons/PhotoSvgIcon.vue";

import face0001 from "../assets/images/face0001.jpg";

const imageSrc = ref<string | undefined>();

const leftCheekCenter = ref<{ x: number; y: number } | undefined>();
const rightCheekCenter = ref<{ x: number; y: number } | undefined>();

const blushScale = ref<number | undefined>();
const blushOpacity = ref<number>(0.5);
const blushColor = ref<string>("#fa0a0a");

const isImageProceeding = ref(false);
const openModal = ref(false);

const loadModals = async () => {
  await faceapi.nets.faceLandmark68Net.loadFromUri("/models");
  await faceapi.nets.tinyFaceDetector.loadFromUri("/models");
  await faceapi.nets.faceLandmark68TinyNet.loadFromUri("/models");
  
  imageSrc.value = face0001
};

onMounted(loadModals);

const getLandmarks = async () => {
  isImageProceeding.value = true;
  const img = document.getElementById("stockImg") as HTMLImageElement;
  const detections = await faceapi
    .detectAllFaces(img, new faceapi.TinyFaceDetectorOptions())
    .withFaceLandmarks(true);
  if (detections.length > 0) {
    const landmarks = detections[0].landmarks;

    const mouthLeft = landmarks.getMouth()[0];
    const mouthRight = landmarks.getMouth()[6];
    const eyeLeft = landmarks.getLeftEye()[0];
    const eyeRight = landmarks.getRightEye()[3];

    leftCheekCenter.value = {
      x: (mouthLeft.x + eyeLeft.x) / 2,
      y: (mouthLeft.y + eyeLeft.y) / 2,
    };

    rightCheekCenter.value = {
      x: (mouthRight.x + eyeRight.x) / 2,
      y: (mouthRight.y + eyeRight.y) / 2,
    };

    blushScale.value =
      Math.sqrt(
        Math.pow(rightCheekCenter.value!.x - leftCheekCenter.value!.x, 2) +
          Math.pow(rightCheekCenter.value!.y - leftCheekCenter.value!.y, 2)
      ) / 9;
  } else {
    leftCheekCenter.value = undefined;
    rightCheekCenter.value = undefined;
  }
  isImageProceeding.value = false;
};

const drawBlush = () => {
  isImageProceeding.value = true;

  const canvas = document.querySelector("canvas") as HTMLCanvasElement;
  const img = document.getElementById("stockImg") as HTMLImageElement;
  canvas.width = img.width;
  canvas.height = img.height;
  const ctx = canvas.getContext("2d");

  if (!leftCheekCenter.value && !rightCheekCenter.value) {
    ctx?.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
    isImageProceeding.value = false;
    return;
  }

  if (ctx) {
    ctx.beginPath();
    ctx.arc(
      leftCheekCenter.value!.x,
      leftCheekCenter.value!.y,
      blushScale.value || 0,
      0,
      2 * Math.PI
    );
    ctx.arc(
      rightCheekCenter.value!.x,
      rightCheekCenter.value!.y,
      blushScale.value || 0,
      0,
      2 * Math.PI
    );
    ctx.fillStyle = blushColor.value;
    ctx.fill();
    ctx.closePath();
  }
  isImageProceeding.value = false;
};

watch([leftCheekCenter, rightCheekCenter, blushColor], drawBlush);

const handleFileChange = (event: Event) => {
  const input = event.target as HTMLInputElement;
  const file = input.files?.[0];

  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      imageSrc.value = e.target?.result as string;
    };
    reader.readAsDataURL(file);
  }
};
</script>

<style scoped>
.container {
  display: flex;
  position: relative;
}
.image {
  order: 1;
  position: absolute;
}
.canvas {
  order: 2;
  z-index: 2;
  filter: blur(10px);
}
.custom-file-input {
  background-color: #bbb2dd;
  color: #fff;
  cursor: pointer;
  border-radius: 5px;
  font-size: 2.5rem;
  display: flex;
  align-items: center;
  width: 70px;
  justify-content: center;
}
.custom-file-input:hover {
  background-color: #afa2dd;
}
</style>
