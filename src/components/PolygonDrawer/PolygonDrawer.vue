<script lang="ts" setup>
import { onBeforeUnmount, onMounted, ref } from 'vue';

interface Point {
  x: number
  y: number
}

const canvas = ref<HTMLCanvasElement | null>(null)
const points = ref<Point[]>([])
const isCompleted = ref<boolean>(false)

function onMouseDown(event: MouseEvent) {
  if (!canvas.value)
    return

  let ctx = canvas.value.getContext("2d")

  if (!ctx)
    return

  const rect = canvas.value.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;

  ctx.fillStyle = 'red';
  ctx.strokeStyle = 'green';

  ctx.beginPath();
  ctx.arc(x, y, 10, 0, Math.PI * 2, true);
  ctx.fill();
  ctx.closePath()

  points.value.push({ x, y })

  const hasPreviousPoints: boolean = points.value.length > 1

  if (!hasPreviousPoints)
    return

  const point1 = points.value[points.value.length - 2]
  const point2 = points.value[points.value.length - 1]

  ctx.beginPath();
  ctx.moveTo(point1.x, point1.y);
  ctx.lineTo(point2.x, point2.y);
  ctx.stroke();
}

function addCanvasEvents(): void {
  if (!canvas.value)
    return
  
  canvas.value.addEventListener('mousedown', onMouseDown)
}

function removeCanvasEvents(): void {
  if (!canvas.value)
    return
  
  canvas.value.removeEventListener('mousedown', onMouseDown)
}

function onComplete(): void {
  if (!canvas.value)
    return

  let ctx = canvas.value.getContext("2d")

  if (!ctx)
    return
  
  isCompleted.value = true

  ctx.beginPath();

  points.value.forEach((point1) => {
    points.value.forEach((point2) => {
      if (!ctx)
        return
      
        ctx.lineTo(point1.x, point1.y);
        ctx.lineTo(point2.x, point2.y);
        ctx.fill()
    })
  })

  ctx.fill();
}

function onClear(): void {
  if (!canvas.value)
    return

  let ctx = canvas.value.getContext("2d")

  if (!ctx)
    return
  
  points.value = []
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  isCompleted.value = false
}

onMounted(() => {
  addCanvasEvents()
});

onBeforeUnmount(() => {
  removeCanvasEvents()
})
</script>

<template>
  <div
    class="ss-polygon-drawer"
    :class="{
      'ss-polygon-drawer--completed': isCompleted
    }"
  >
    <canvas 
      ref="canvas"
      class="ss-polygon-drawer__canvas"
      width="800"
      height="600"
    >
    </canvas>
    <div class="ss-polygon-drawer__actions">
      <button @click="onClear">Clear</button>
      <button @click="onComplete">Complete</button>
    </div>
  </div>
</template>

<style>
.ss-polygon-drawer__canvas {
  width: 800px;
  height: 600px;
  border: 1px solid gray;
  cursor: crosshair;
}

.ss-polygon-drawer__actions {
  display: flex;
  gap: 16px;
  justify-content: center;
}

.ss-polygon-drawer--completed .ss-polygon-drawer__canvas {
  pointer-events: none;
}
</style>