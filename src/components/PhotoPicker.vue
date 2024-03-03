<template>
  <div class="overlay">
    <div class="overlay-items">
      <div :style="{ display: 'flex', justifyContent: 'end' }">
        <button
          @click="() => emits('update:photo', undefined)"
          :style="{ fontSize: '2rem' }"
        >
          <close-svg-icon />
        </button>
      </div>
      <div class="image-list">
        <button
          v-for="imgSrc in imgSrcList"
          @click="() => emits('update:photo', imgSrc)"
        >
          <img :src="imgSrc" class="image" />
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import CloseSvgIcon from "../assets/icons/CloseSvgIcon.vue";
import face0001 from "../assets/images/face0001.jpg";
import face0002 from "../assets/images/face0002.jpg";
import face0003 from "../assets/images/face0003.jpg";

const imgSrcList = [face0001, face0002, face0003];

const emits = defineEmits<{
  (e: "update:photo", value: string | undefined): void;
}>();
</script>

<style scoped>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  align-items: center;
  flex-direction: column;
  justify-content: center;
  z-index: 5;
}

.image-list {
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 5;
}

.image {
  height: 60vh;
}
.overlay-items {
  background-color: white;
  padding: 1rem;
  overflow: auto;
  max-width: 100%;
}
</style>
