<script setup>
import { motion, AnimatePresence } from 'motion-v'
import { ref, onMounted, computed } from 'vue'
import StickerOnStick from './components/StickerOnStick.vue'

const quote = ref('')
const calcBox = ref(0)
const width = ref(200)
const height = ref(100)

async function getKanye() {
  quote.value = ''
  const response = await fetch('https://api.kanye.rest/').then((res) => res.json())
  let text = await response.quote
  quote.value = text
}

const getWords = computed(() => {
  return quote.value.split(' ')
})

const resizeObserver = new ResizeObserver((el) => {
  height.value = Math.ceil(el[0].target.offsetHeight) + 1
  width.value = Math.ceil(el[0].target.offsetWidth) + 1
})

onMounted(() => {
  getKanye()
  resizeObserver.observe(calcBox.value)
})
</script>

<template>
  <div class="container">
    <div class="quoteContainer">
      <motion.div
        class="quote glass"
        layout
        :style="{ width: `${width}px`, height: `${height}px` }"
        :transition="{ type: 'spring' }"
        :dragConstraints="{ top: 10, right: 10, bottom: 10, left: 10 }"
        :dragTransition="{ bounceStiffness: 500, bounceDamping: 15 }"
        :dragElastic="0.2"
        :whileDrag="{ cursor: 'grabbing' }"
        drag
      >
        <AnimatePresence mode="popLayout">
          <motion.span
            v-for="(word, idx) in getWords"
            :key="idx"
            class="word"
            :initial="{ opacity: 0, scale: 0 }"
            :animate="{ opacity: 1, scale: 1 }"
            :transition="{ type: 'spring', delay: idx * 0.05 }"
          >
            {{ word }}
          </motion.span>
          <motion.div
            class="author"
            :key="getWords"
            :initial="{ opacity: 0, scale: 0 }"
            :animate="{ opacity: 1, scale: 1 }"
            :transition="{ type: 'spring', delay: getWords.length * 0.05 }"
          >
            ~ Kayne West
          </motion.div>
        </AnimatePresence>
      </motion.div>
      <div
        style="opacity: 0; top: 0; position: absolute; width: auto; height: auto"
        class="quote glass"
        ref="calcBox"
      >
        <div v-for="(word, idx) in getWords" :key="idx" class="word">
          <span>{{ word }}</span>
        </div>
      </div>
    </div>
    <StickerOnStick
      :src="'/src/assets/kanye.png'"
      :alt="'Kanye West'"
      :outlineWidth="4"
      @click="getKanye"
      class="sticker"
    />
  </div>
</template>

<style scoped>
.container {
  display: grid;
  grid-template-rows: 100%;
  grid-template-columns: 3fr 1fr;
  justify-content: center;
  align-items: center;
  max-height: 100vh;
  max-width: 100vw;
  width: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
}

.quoteContainer {
  grid-row: 1;
  grid-column: 1;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  padding: 30px;
  font-family: 'Rouge Script', cursive;
  font-weight: 400;
  font-style: normal;
  flex-wrap: wrap;
}

.quote {
  padding: 20px;
  background: rgba(255, 255, 255, 0.5);
  min-height: 100px;
  min-width: 200px;
  max-width: 50vw;
  color: white;
  font-size: 3rem;
  border-radius: 20px;
  position: relative;
  display: flex;
  flex-wrap: wrap;
  row-gap: 10px;
  column-gap: 10px;
}

.word {
  height: auto;
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.author {
  position: absolute;
  bottom: 0;
  right: 10px;
  font-size: 1.5rem;
}

.sticker {
  right: 0;
  grid-row: 1;
  grid-column: 2;
}

.glass {
  /* From https://css.glass */
  background: rgba(255, 255, 255, 0.11);
  border-radius: 16px;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10.4px);
  -webkit-backdrop-filter: blur(10.4px);
  border: 1px solid rgba(255, 255, 255, 1);
}
</style>
