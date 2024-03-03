<template>
  <div
    :style="{ display: 'flex', gap: '1rem', fontSize: '2.5rem' }"
  >
    <button
      v-for="color in colors"
      class="circle"
      :style="{ backgroundColor: color, color: 'white' }"
      @click="changeColor(color)"
    >
      <color-svg-icon />
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import ColorSvgIcon from "../assets/icons/ColorSvgIcon.vue";

const emits = defineEmits<{
  (e: "update:color", value: string): void;
}>();

const props = defineProps<{
  color: string;
}>();

const pickedColor = ref(props.color);

const colors = [
  "#ff1ca0",
  "#bd5390",
  "#c7245d",
  "#fa0a5e",
  "#fa0a0a",
  "#d13d3d",
];

const changeColor = (val: string) => {
  pickedColor.value = val;
  emits("update:color", val);
};
</script>

<style scoped>
.circle {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.5);
}
</style>
