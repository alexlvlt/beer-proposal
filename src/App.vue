<template>
  <div class="bg-bubbles">
    <div v-for="n in 6" :key="n" class="bubble" />
  </div>

  <Transition name="win">
    <div v-if="won" class="win-overlay">
      <div class="confetti-row">🎉🍺🎬🎉</div>
      <div class="win-title">ура!! 🎊</div>
      <div class="win-sub">договорились, жду тебя 🥳</div>
      <div class="confetti-row" style="font-size:2rem">🍿🛋️🌙✨</div>
    </div>
  </Transition>

  <main class="scene">
    <span class="emoji-float">🍺🎬</span>
    <p class="question">придёшь ко мне в гости<br />пить пиво и смотреть фильм?</p>

    <div class="buttons-area">
      <button
          ref="yesBtnRef"
          class="btn btn-yes"
          :style="{ transform: `scale(${yesScale})` }"
          @click="sayYes"
      >
        да 🍻
      </button>

      <button
          v-if="!noMoved && !noGone"
          ref="noStaticBtnRef"
          class="btn btn-no"
          @click="sayNo"
      >
        нет
      </button>
    </div>

    <button
        v-if="noMoved && !noGone"
        ref="noBtnRef"
        class="btn btn-no"
        :style="floatingStyle"
        @click="sayNo"
    >
      {{ noLabel }}
    </button>
  </main>
</template>

<script setup>
import { computed, ref, nextTick, onMounted, onBeforeUnmount } from 'vue'

const NO_TEXTS = [
  'ну пожалуйста…',
  'ну серьёзно??',
  'подумай ещё раз',
  'я обижусь 😢',
  'последний шанс',
  'окей, я уточняю',
  'ты уверен(а)?',
  'нет — не вариант 😤',
  'кнопка сломалась, жми ДА',
]

const won = ref(false)
const noCount = ref(0)
const noMoved = ref(false)
const noVisible = ref(false)
const noPos = ref({ x: 0, y: 0 })

const yesBtnRef = ref(null)
const noStaticBtnRef = ref(null)
const noBtnRef = ref(null)

const viewportWidth = ref(0)
const yesBaseWidth = ref(0)

const desiredYesScale = computed(() => 1 + (noMoved.value ? noCount.value + 1 : 0) * 0.2)
const noFloatScale = computed(() => Math.max(1 - noCount.value * 0.07, 0.5))
const noGone = computed(() => noCount.value >= NO_TEXTS.length)
const noLabel = computed(() => NO_TEXTS[Math.min(noCount.value, NO_TEXTS.length - 1)])

const yesScale = computed(() => {
  const maxWidth = Math.max(viewportWidth.value - 32, 120)
  const safeScale = yesBaseWidth.value > 0 ? maxWidth / yesBaseWidth.value : 2.5
  return Math.max(1, Math.min(desiredYesScale.value, safeScale))
})

const floatingStyle = computed(() => ({
  position: 'fixed',
  left: `${noPos.value.x}px`,
  top: `${noPos.value.y}px`,
  transform: `scale(${noFloatScale.value})`,
  transformOrigin: 'top left',
  transition: 'left 0.35s cubic-bezier(.34,1.4,.64,1), top 0.35s cubic-bezier(.34,1.4,.64,1), transform 0.3s ease, opacity 0.15s',
  opacity: noVisible.value ? 1 : 0,
  zIndex: 50,
}))

function getViewport() {
  const vv = window.visualViewport

  if (vv) {
    return {
      width: vv.width,
      height: vv.height,
      offsetLeft: vv.offsetLeft,
      offsetTop: vv.offsetTop,
    }
  }

  return {
    width: window.innerWidth,
    height: window.innerHeight,
    offsetLeft: 0,
    offsetTop: 0,
  }
}

function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max)
}

function updateViewportMetrics() {
  const viewport = getViewport()
  viewportWidth.value = viewport.width

  if (yesBtnRef.value) {
    yesBaseWidth.value = yesBtnRef.value.offsetWidth || 0
  }

  if (noMoved.value && !noGone.value) {
    clampFloatingButtonToViewport()
  }
}

function getFloatingButtonSize() {
  const btn = noBtnRef.value

  if (!btn) {
    return { width: 140, height: 48 }
  }

  return {
    width: btn.offsetWidth * noFloatScale.value,
    height: btn.offsetHeight * noFloatScale.value,
  }
}

function setFloatingStartFromStatic() {
  const source = noStaticBtnRef.value
  const viewport = getViewport()

  if (!source) {
    return
  }

  const rect = source.getBoundingClientRect()

  noPos.value = {
    x: rect.left + viewport.offsetLeft,
    y: rect.top + viewport.offsetTop,
  }
}

function placeRandom() {
  const viewport = getViewport()
  const { width: btnW, height: btnH } = getFloatingButtonSize()
  const margin = 20

  const minX = viewport.offsetLeft + margin
  const maxX = viewport.offsetLeft + viewport.width - btnW - margin
  const minY = viewport.offsetTop + margin
  const maxY = viewport.offsetTop + viewport.height - btnH - margin

  noPos.value = {
    x: clamp(minX + Math.random() * Math.max(maxX - minX, 0), minX, maxX),
    y: clamp(minY + Math.random() * Math.max(maxY - minY, 0), minY, maxY),
  }
}

function clampFloatingButtonToViewport() {
  const viewport = getViewport()
  const { width: btnW, height: btnH } = getFloatingButtonSize()
  const margin = 20

  const minX = viewport.offsetLeft + margin
  const maxX = viewport.offsetLeft + viewport.width - btnW - margin
  const minY = viewport.offsetTop + margin
  const maxY = viewport.offsetTop + viewport.height - btnH - margin

  noPos.value = {
    x: clamp(noPos.value.x, minX, maxX),
    y: clamp(noPos.value.y, minY, maxY),
  }
}

async function sayNo() {
  if (!noMoved.value) {
    setFloatingStartFromStatic()
    noMoved.value = true
    noVisible.value = true

    await nextTick()
    await new Promise(resolve => requestAnimationFrame(resolve))

    placeRandom()
    return
  }

  noCount.value++
  await nextTick()

  if (noGone.value) {
    return
  }

  placeRandom()
}

function sayYes() {
  won.value = true
}

onMounted(async () => {
  await nextTick()
  updateViewportMetrics()

  window.addEventListener('resize', updateViewportMetrics)
  window.addEventListener('orientationchange', updateViewportMetrics)
  window.visualViewport?.addEventListener('resize', updateViewportMetrics)
  window.visualViewport?.addEventListener('scroll', updateViewportMetrics)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', updateViewportMetrics)
  window.removeEventListener('orientationchange', updateViewportMetrics)
  window.visualViewport?.removeEventListener('resize', updateViewportMetrics)
  window.visualViewport?.removeEventListener('scroll', updateViewportMetrics)
})
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Unbounded:wght@400;700;900&family=Nunito:wght@400;600&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --yes: #f7c948;
  --no: #2e2e3a;
  --bg: #0f0f1a;
  --text: #f0eaff;
}

html, body {
  height: 100%;
  background: var(--bg);
  color: var(--text);
  font-family: 'Nunito', sans-serif;
  overflow: hidden;
}

.bg-bubbles {
  position: fixed; inset: 0; z-index: 0; overflow: hidden; pointer-events: none;
}
.bubble {
  position: absolute; border-radius: 50%; opacity: 0.12;
  animation: float linear infinite; bottom: -100px;
}
.bubble:nth-child(1) { width:80px;  height:80px;  left:10%; background:#f7c948; animation-duration:14s; animation-delay:0s; }
.bubble:nth-child(2) { width:40px;  height:40px;  left:25%; background:#a78bfa; animation-duration:18s; animation-delay:2s; }
.bubble:nth-child(3) { width:120px; height:120px; left:50%; background:#f7c948; animation-duration:22s; animation-delay:4s; }
.bubble:nth-child(4) { width:60px;  height:60px;  left:75%; background:#a78bfa; animation-duration:16s; animation-delay:1s; }
.bubble:nth-child(5) { width:90px;  height:90px;  left:88%; background:#f7c948; animation-duration:20s; animation-delay:6s; }
.bubble:nth-child(6) { width:30px;  height:30px;  left:5%;  background:#a78bfa; animation-duration:12s; animation-delay:3s; }

@keyframes float {
  0%   { transform: translateY(0) rotate(0deg); opacity: 0.12; }
  100% { transform: translateY(-110vh) rotate(720deg); opacity: 0; }
}

.scene {
  position: relative; z-index: 1;
  min-height: 100vh;
  min-height: 100dvh;
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  padding: 24px; text-align: center; gap: 48px;
}

.emoji-float {
  font-size: clamp(48px, 10vw, 80px);
  animation: bob 3s ease-in-out infinite;
}
@keyframes bob {
  0%, 100% { transform: translateY(0); }
  50%       { transform: translateY(-10px); }
}

.question {
  font-family: 'Unbounded', sans-serif;
  font-size: clamp(18px, 4vw, 32px);
  font-weight: 700; line-height: 1.4; max-width: 600px;
  background: linear-gradient(135deg, #f0eaff 30%, #f7c948 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.buttons-area {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  height: 60px;
  overflow: visible;
}

.btn {
  font-family: 'Unbounded', sans-serif;
  font-weight: 700; border: none;
  border-radius: 999px; cursor: pointer;
  line-height: 1.2; white-space: nowrap;
}

.btn-yes {
  font-size: 18px;
  padding: 14px 36px;
  background: var(--yes);
  color: #1a1a2e;
  box-shadow: 0 4px 24px rgba(247,201,72,0.35);
  transform-origin: center center;
  transition: transform 0.4s cubic-bezier(.34,1.56,.64,1);
}

.btn-no {
  font-size: 16px;
  padding: 12px 28px;
  background: var(--no);
  color: #a0a0c0;
  border: 1px solid rgba(255,255,255,0.08);
  box-shadow: 0 2px 10px rgba(0,0,0,0.3);
}

.win-overlay {
  position: fixed; inset: 0; z-index: 100;
  background: rgba(10,10,26,0.92);
  display: flex; flex-direction: column;
  align-items: center; justify-content: center; gap: 24px;
}
.win-title {
  font-family: 'Unbounded', sans-serif;
  font-size: clamp(28px, 6vw, 56px); font-weight: 900;
  color: var(--yes); text-shadow: 0 0 40px rgba(247,201,72,0.7);
}
.win-sub { font-size: clamp(16px, 3vw, 22px); color: #c0b8e8; }
.confetti-row { font-size: clamp(32px, 6vw, 54px); }

.win-enter-active { animation: winPop 0.4s cubic-bezier(.34,1.56,.64,1); }
.win-leave-active { animation: winPop 0.3s reverse ease-in; }
@keyframes winPop {
  from { opacity: 0; transform: scale(0.85); }
  to   { opacity: 1; transform: scale(1); }
}
</style>
