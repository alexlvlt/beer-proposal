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

    <div ref="buttonsAreaRef" class="buttons-area">
      <button
          ref="yesBtnRef"
          class="btn btn-yes"
          :style="{ transform: `scale(${yesScale})` }"
          @click="sayYes"
      >
        да 🍻
      </button>

      <button
          v-if="!noGone"
          ref="noBtnRef"
          class="btn btn-no"
          :style="noStyle"
          @click="sayNo"
      >
        {{ noLabel }}
      </button>
    </div>
  </main>
</template>

<script setup>
import { computed, ref, onMounted, onBeforeUnmount, nextTick } from 'vue'

const NO_TEXTS = [
  'нет',
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
const noBtnRef = ref(null)
const yesBtnRef = ref(null)
const buttonsAreaRef = ref(null)

const noPos = ref({ x: 0, y: 0 })
const noReady = ref(false)

const yesScale = computed(() => Math.min(1 + noCount.value * 0.3, 4))
const noScale = computed(() => Math.max(1 - noCount.value * 0.06, 0.55))
const noGone = computed(() => noCount.value >= NO_TEXTS.length)
const noLabel = computed(() => NO_TEXTS[Math.min(noCount.value, NO_TEXTS.length - 1)])

const noStyle = computed(() => ({
  position: 'fixed',
  left: `${noPos.value.x}px`,
  top: `${noPos.value.y}px`,
  transform: `scale(${noScale.value})`,
  transformOrigin: 'center center',
  opacity: noReady.value ? 1 : 0,
  transition:
      noCount.value === 0
          ? 'opacity 0.2s ease'
          : 'left 0.35s cubic-bezier(.34,1.4,.64,1), top 0.35s cubic-bezier(.34,1.4,.64,1), transform 0.3s ease, opacity 0.2s ease',
}))

function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max)
}

function getViewportSize() {
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

function getScaledSize() {
  const btn = noBtnRef.value
  if (!btn) return { width: 140, height: 48 }

  const rect = btn.getBoundingClientRect()
  const scale = noScale.value || 1

  return {
    width: rect.width * scale,
    height: rect.height * scale,
  }
}

function placeNoInitially() {
  const yesBtn = yesBtnRef.value
  const noBtn = noBtnRef.value
  const area = buttonsAreaRef.value
  if (!yesBtn || !noBtn || !area) return

  const yesRect = yesBtn.getBoundingClientRect()
  const areaRect = area.getBoundingClientRect()
  const { width: btnW, height: btnH } = getScaledSize()
  const { width: vpW, height: vpH, offsetLeft, offsetTop } = getViewportSize()

  const gap = 16
  const idealX = yesRect.right + gap
  const idealY = yesRect.top + yesRect.height / 2 - btnH / 2

  noPos.value = {
    x: clamp(idealX + offsetLeft, 12 + offsetLeft, vpW - btnW - 12 + offsetLeft),
    y: clamp(
        idealY + offsetTop,
        Math.max(12 + offsetTop, areaRect.top + offsetTop),
        Math.min(vpH - btnH - 12 + offsetTop, areaRect.bottom - btnH + offsetTop)
    ),
  }
}

function moveNoRandomly() {
  const area = buttonsAreaRef.value
  const noBtn = noBtnRef.value
  if (!area || !noBtn) return

  const areaRect = area.getBoundingClientRect()
  const { width: btnW, height: btnH } = getScaledSize()
  const { width: vpW, height: vpH, offsetLeft, offsetTop } = getViewportSize()
  const margin = 12

  const minX = Math.max(margin + offsetLeft, areaRect.left + offsetLeft)
  const maxX = Math.min(vpW - btnW - margin + offsetLeft, areaRect.right - btnW + offsetLeft)

  const minY = Math.max(margin + offsetTop, areaRect.top + offsetTop)
  const maxY = Math.min(vpH - btnH - margin + offsetTop, areaRect.bottom - btnH + offsetTop)

  noPos.value = {
    x: clamp(minX + Math.random() * Math.max(maxX - minX, 0), minX, maxX),
    y: clamp(minY + Math.random() * Math.max(maxY - minY, 0), minY, maxY),
  }
}

async function updateInitialPosition() {
  await nextTick()
  placeNoInitially()
  noReady.value = true
}

async function sayNo() {
  noCount.value++
  await nextTick()

  if (noGone.value) return
  moveNoRandomly()
}

function sayYes() {
  won.value = true
}

function handleViewportChange() {
  if (won.value || noGone.value) return

  if (noCount.value === 0) {
    placeNoInitially()
  } else {
    moveNoRandomly()
  }
}

onMounted(async () => {
  await updateInitialPosition()

  window.addEventListener('resize', handleViewportChange)
  window.addEventListener('orientationchange', handleViewportChange)
  window.visualViewport?.addEventListener('resize', handleViewportChange)
  window.visualViewport?.addEventListener('scroll', handleViewportChange)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleViewportChange)
  window.removeEventListener('orientationchange', handleViewportChange)
  window.visualViewport?.removeEventListener('resize', handleViewportChange)
  window.visualViewport?.removeEventListener('scroll', handleViewportChange)
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
  50% { transform: translateY(-10px); }
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
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 120px;
  width: min(100%, 520px);
}

.btn {
  font-family: 'Unbounded', sans-serif;
  font-weight: 700;
  border: none;
  border-radius: 999px;
  cursor: pointer;
  line-height: 1.2;
  white-space: nowrap;
  -webkit-tap-highlight-color: transparent;
  touch-action: manipulation;
}

.btn-yes {
  width: 144px;
  height: 52px;
  font-size: 18px;
  padding: 0 24px;
  background: var(--yes);
  color: #1a1a2e;
  box-shadow: 0 4px 24px rgba(247,201,72,0.35);
  transition: transform 0.4s cubic-bezier(.34,1.56,.64,1), box-shadow 0.2s ease;
  transform-origin: center;
}
.btn-yes:active { filter: brightness(0.9); }

.btn-no {
  width: 220px;
  height: 48px;
  font-size: 16px;
  padding: 0 18px;
  background: var(--no);
  color: #a0a0c0;
  border: 1px solid rgba(255,255,255,0.08);
  box-shadow: 0 2px 10px rgba(0,0,0,0.3);
  will-change: left, top, transform;
}
.btn-no:active { filter: brightness(1.2); }

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
  to { opacity: 1; transform: scale(1); }
}

@media (max-width: 480px) {
  .buttons-area {
    min-height: 132px;
    width: min(100%, 340px);
  }

  .btn-yes {
    width: 136px;
    height: 50px;
    font-size: 17px;
  }

  .btn-no {
    width: 196px;
    height: 46px;
    font-size: 15px;
  }
}
</style>
