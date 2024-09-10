<script setup lang="ts">
const canvas = useTemplateRef("canvas");
const { width, height } = useWindowSize()

let ctx: CanvasRenderingContext2D | null;

let cx = 0;
let cy = window.innerHeight / 2;

let px = 0;
let py = window.innerHeight / 2;

const step = 5

const draw = (px: number, py: number, cx: number, cy: number) => {
  if (!ctx) return;

  ctx.beginPath()
  ctx.moveTo(px, py)

  ctx.lineTo(cx, cy)
  ctx.stroke()
}

const history = ref<{ x: number, y: number }[]>([])
const startOffset = ref(0)

const part = computed(() => history.value.slice(startOffset.value, startOffset.value + width.value))
const canSwipe = computed(() => history.value.length * step > width.value)
const drawOffset = computed(() => startOffset.value * step)

const redraw = () => {
  ctx?.clearRect(0, 0, width.value, height.value)

  for (let index = 1; index < part.value.length; index++) {
    const curr = part.value[index];
    const prev = part.value[index - 1];

    draw(prev.x - drawOffset.value, prev.y, curr.x - drawOffset.value, curr.y)
  }
}

let offsetX = 0
const { distanceX } = usePointerSwipe(canvas, {
  threshold: 0,
  onSwipe: () => {
    if (!canSwipe.value) return

    const r = distanceX.value + offsetX
    startOffset.value = Math.max(0, Math.min((history.value.length * step - width.value) / step, r))

    redraw()
  },
  onSwipeEnd: () => {
    offsetX = Math.max(0, startOffset.value)
  }
})

useEventListener('resize', () => redraw())

const handleWheel = (event: WheelEvent) => {
  const delta = event.deltaY
  cx += step

  if (delta > 0) {
    cy -= step
  } else {
    cy += step
  }

  draw(px - drawOffset.value, py, cx - drawOffset.value, cy)

  history.value.push({
    x: cx,
    y: cy
  })

  px = cx
  py = cy
};

onMounted(() => {
  if (!canvas.value) return;
  ctx = canvas.value.getContext("2d");
});
</script>

<template>
  <canvas
    ref="canvas"
    @wheel="handleWheel"
    :width="width"
    :height="height"
    class="main"
    :class="{ 'cursor': history.length * step > width }"
  />
  <p class="count">{{ startOffset }}</p>
</template>

<style>
html,
body,
#__nuxt,
p {
  margin: 0px;
  padding: 0px;
  width: 100%;
  height: 100%;
}

* {
  box-sizing: border-box;
}

.count {
  position: absolute;
  top: 0;
  left: 0;
  height: max-content;
}

.main {
  width: 100%;
  height: 100%;
  display: block;
}

.cursor {
  cursor: grab;
}
</style>
