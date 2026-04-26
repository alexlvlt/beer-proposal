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
          :style="yesBtnStyle"
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
import { computed, ref, nextTick } from 'vue'

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

const won      = ref(false)
const noCount  = ref(0)
const noPos    = ref({ x: null, y: null })
const noBtnRef = ref(null)

// Явные пиксельные значения вместо CSS-переменной — надёжнее на Safari/iOS
const yesFontSize = computed(() => Math.min(18 * (1 + noCount.value * 0.22), 18 * 4.5))
const yesPadV     = computed(() => Math.min(14 * (1 + noCount.value * 0.22), 14 * 4.5))
const yesPadH     = computed(() => Math.min(36 * (1 + noCount.value * 0.22), 36 * 4.5))

const yesBtnStyle = computed(() => ({
  fontSize: `${yesFontSize.value}px`,
  padding:  `${yesPadV.value}px ${yesPadH.value}px`,
}))

const noScale = computed(() => Math.max(1 - noCount.value * 0.06, 0.55))
const noGone  = computed(() => noCount.value >= NO_TEXTS.length)
const noLabel = computed(() => NO_TEXTS[Math.min(noCount.value, NO_TEXTS.length - 1)])

const noStyle = computed(() => {
  const base = {
    fontSize: `${16 * noScale.value}px`,
    padding:  `${12 * noScale.value}px ${28 * noScale.value}px`,
  }
  if (noPos.value.x !== null) {
    return {
      ...base,
      position: 'fixed',
      left: `${noPos.value.x}px`,
      top:  `${noPos.value.y}px`,
      transition: 'left 0.3s cubic-bezier(.34,1.4,.64,1), top 0.3s cubic-bezier(.34,1.4,.64,1), font-size 0.3s, padding 0.3s',
      zIndex: 50,
    }
  }
  return base
})

async function sayNo() {
  noCount.value++

  // Ждём nextTick чтобы получить актуальный размер кнопки после shrink
  await nextTick()

  const margin = 16
  const w = window.innerWidth
  const h = window.innerHeight

  // Учитываем реальный размер кнопки — она не выйдет за край
  const btnW = noBtnRef.value?.offsetWidth  ?? 80
  const btnH = noBtnRef.value?.offsetHeight ?? 40

  noPos.value = {
    x: margin + Math.random() * (w - btnW - margin * 2),
    y: margin + Math.random() * (h - btnH - margin * 2),
  }
}

function sayYes() {
  won.value = true
}
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
  position: absolute;
  border-radius: 50%;
  opacity: 0.12;
  animation: float linear infinite;
  bottom: -100px;
}
.bubble:nth-child(1) { width:80px;  height:80px;  left:10%; background:#f7c948; animation-duration:14s; animation-delay:0s;  }
.bubble:nth-child(2) { width:40px;  height:40px;  left:25%; background:#a78bfa; animation-duration:18s; animation-delay:2s;  }
.bubble:nth-child(3) { width:120px; height:120px; left:50%; background:#f7c948; animation-duration:22s; animation-delay:4s;  }
.bubble:nth-child(4) { width:60px;  height:60px;  left:75%; background:#a78bfa; animation-duration:16s; animation-delay:1s;  }
.bubble:nth-child(5) { width:90px;  height:90px;  left:88%; background:#f7c948; animation-duration:20s; animation-delay:6s;  }
.bubble:nth-child(6) { width:30px;  height:30px;  left:5%;  background:#a78bfa; animation-duration:12s; animation-delay:3s;  }

@keyframes float {
  0%   { transform: translateY(0) rotate(0deg);    opacity: 0.12; }
  100% { transform: translateY(-110vh) rotate(720deg); opacity: 0; }
}

.scene {
  position: relative; z-index: 1;
  height: 100vh;
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  padding: 24px;
  text-align: center;
  gap: 48px;
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
  font-weight: 700;
  line-height: 1.4;
  max-width: 600px;
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
  flex-wrap: wrap;
  min-height: 80px;
}

.btn {
  font-family: 'Unbounded', sans-serif;
  font-weight: 700;
  border: none;
  border-radius: 999px;
  cursor: pointer;
  line-height: 1.2;
  white-space: nowrap;
}

.btn-yes {
  background: var(--yes);
  color: #1a1a2e;
  box-shadow: 0 4px 24px rgba(247,201,72,0.35);
  /* убрали transform из transition — он конфликтовал с размером на Safari */
  transition: font-size 0.35s cubic-bezier(.34,1.56,.64,1),
  padding   0.35s cubic-bezier(.34,1.56,.64,1),
  box-shadow 0.2s ease;
}
.btn-yes:hover { box-shadow: 0 8px 36px rgba(247,201,72,0.6); }
.btn-yes:active { opacity: 0.85; }

.btn-no {
  background: var(--no);
  color: #a0a0c0;
  border: 1px solid rgba(255,255,255,0.08);
  box-shadow: 0 2px 10px rgba(0,0,0,0.3);
  transition: font-size 0.3s, padding 0.3s, background 0.2s;
}
.btn-no:hover { background: #3a3a4e; }
.btn-no:active { opacity: 0.85; }

.win-overlay {
  position: fixed; inset: 0; z-index: 100;
  background: rgba(10,10,26,0.92);
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  gap: 24px;
}
.win-title {
  font-family: 'Unbounded', sans-serif;
  font-size: clamp(28px, 6vw, 56px);
  font-weight: 900;
  color: var(--yes);
  text-shadow: 0 0 40px rgba(247,201,72,0.7);
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