<template>
  <div class="container">
    <div>SWIPE DETECTOR 5000</div>
    <div>{{ anglePresentation }}</div>
    <div class="direction-info">{{ angleDesc }}</div>
  </div>
</template>

<script setup lang="ts">
import {computed, onBeforeMount, onBeforeUnmount, ref} from "vue";

const angle = ref<number>(0);
const isMousePressed = ref<boolean>(false);
const startX = ref<number | null>(null);
const startY = ref<number | null>(null);

const angleDesc = computed(() => {
  if (!isMousePressed.value) {
    return 'IDLE'
  }

  if (angle.value >= 45 && angle.value < 135) {
    return 'UP';
  }

  if (angle.value >= 135 && angle.value < 225) {
    return 'RIGHT';
  }

  if (angle.value >= 225 && angle.value < 315) {
    return 'DOWN';
  }

  if (angle.value >= 315 || angle.value < 45) {
    return 'LEFT';
  }

  throw new Error(`Unexpected values: angle: ${angle.value}, isMousePressed: ${isMousePressed.value}`);
});

const anglePresentation = computed((): string => {
  return new Intl.NumberFormat(['en-US'], { maximumFractionDigits: 2 }).format(angle.value);
});

const getSwipeAngle = (e: MouseEvent): number | null => {
  if (!startX.value || !startY.value) {
    return null;
  }

  const endX: number = e.screenX + e.movementX;
  const endY: number = e.screenY + e.movementY;

  const deltaX: number = endX - startX.value;
  const deltaY: number = endY - startY.value;

  return Math.atan2(deltaY, deltaX) * 180 / Math.PI + 180;
}

const onMouseDown = (e: MouseEvent) => {
  isMousePressed.value = true;
  startX.value = e.screenX;
  startY.value = e.screenY;
};

const onMouseMove = (e: MouseEvent) => {
  if (!isMousePressed.value) return;

  const _angle: number | null = getSwipeAngle(e);

  if (_angle === 0 || _angle) {
    angle.value = _angle;
  }
};

const onMouseUp = () => {
  isMousePressed.value = false;
  // clean-up
  angle.value = 0;
  startX.value = null;
  startY.value = null;
};

onBeforeMount(() => {
  document.addEventListener('mousedown', onMouseDown);
  document.addEventListener('mousemove', onMouseMove);
  document.addEventListener('mouseup', onMouseUp);
});

onBeforeUnmount(() => {
  document.removeEventListener('mousedown', onMouseDown);
  document.removeEventListener('mousemove', onMouseMove);
  document.removeEventListener('mouseup', onMouseUp);
});
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.container div {
  user-select: none
}

.direction-info {
  font-weight: 700;
}
</style>
