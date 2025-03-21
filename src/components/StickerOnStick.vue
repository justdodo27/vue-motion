<script setup>
import { ref, watch, onMounted, computed } from 'vue'
import { motion } from 'motion-v'

const props = defineProps({
  src: String,
  alt: String,
  outlineWidth: {
    type: Number,
    default: 5, // Default outline thickness
  },
})

// Refs for elements
const imgRef = ref(null)
const canvasRef = ref(null)
const size = ref({ width: 100, height: 100 })

// Function to draw the sticker outline
function drawSticker() {
  const img = imgRef.value
  const canvas = canvasRef.value
  if (!img || !canvas) return

  const ctx = canvas.getContext('2d')

  // Ensure image is loaded before proceeding
  if (!img.complete) {
    img.onload = drawSticker
    return
  }

  // Set canvas size
  canvas.width = img.width + props.outlineWidth * 2
  canvas.height = img.height + props.outlineWidth * 32
  size.value.width = canvas.width
  size.value.height = canvas.height

  // Draw brown stick rectangle at the bottom
  ctx.fillStyle = '#964B00' // brown color
  ctx.fillRect(canvas.width * 0.55, canvas.height * 0.75, canvas.width * 0.03, img.height * 0.3)

  const outlineWidth = props.outlineWidth

  // Create an offscreen canvas for the white mask
  const offCanvas = document.createElement('canvas')
  offCanvas.width = img.width
  offCanvas.height = img.height
  const offCtx = offCanvas.getContext('2d')

  // Draw the original image on the offscreen canvas
  offCtx.drawImage(img, 0, 0)

  // Use 'source-in' to keep only the nontransparent parts and make them white
  offCtx.globalCompositeOperation = 'source-in'
  offCtx.fillStyle = 'white'
  offCtx.fillRect(0, 0, offCanvas.width, offCanvas.height)

  // Draw the white outline with an offset around the image
  for (let dx = -outlineWidth; dx <= outlineWidth; dx++) {
    for (let dy = -outlineWidth; dy <= outlineWidth; dy++) {
      if (dx === 0 && dy === 0) continue
      ctx.drawImage(offCanvas, dx + outlineWidth, dy + outlineWidth)
    }
  }

  // Draw the original image on top
  ctx.drawImage(img, outlineWidth, outlineWidth)
}

// Watch for changes in `src` and redraw when it updates
watch(
  () => props.src,
  () => {
    drawSticker()
  },
)

// Call drawSticker when the component is mounted and the image is ready
onMounted(() => {
  if (imgRef.value && imgRef.value.complete) {
    drawSticker()
  } else {
    imgRef.value.onload = drawSticker
  }
})

const getSize = computed(() => {
  return size.value
})
</script>

<template>
  <motion.div
    class="stickerContainer"
    :style="{ width: `${getSize.width}px`, height: `${getSize.height}px` }"
    :whilePress="{ scale: 0.9, rotate: 3 }"
    :dragConstraints="{ top: 10, right: 10, bottom: 10, left: 10 }"
    :dragTransition="{ bounceStiffness: 500, bounceDamping: 15 }"
    :dragElastic="0.2"
    :whileDrag="{ cursor: 'grabbing' }"
    drag
  >
    <canvas ref="canvasRef"></canvas>
    <img ref="imgRef" :src="src" :alt="alt" hidden />
  </motion.div>
</template>

<style>
.stickerContainer {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  max-width: 90svw;
  max-height: 100%;
}

canvas {
  z-index: 1000;
  position: absolute;
  max-width: 100%;
}
</style>
