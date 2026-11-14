<!-- src/components/ui/TourOverlay.vue -->
<template>
  <Teleport to="body">
    <Transition name="tour-fade">
      <div v-if="active" class="tour-root">

        <!-- Centered step: full dimmer, no spotlight -->
        <div v-if="mode === 'center'" class="tour-dim" />

        <!-- Spotlight step: transparent hole over the target (its shadow dims everything else) -->
        <div
          v-else-if="box"
          class="tour-spot"
          :style="{
            top: box.top + 'px',
            left: box.left + 'px',
            width: box.width + 'px',
            height: box.height + 'px',
          }"
        />

        <!-- Tooltip / card -->
        <div
          class="tour-card"
          :class="{ 'tour-card--center': mode === 'center' }"
          :style="mode === 'center' ? null : cardStyle"
          ref="cardEl"
          role="dialog"
          aria-live="polite"
        >
          <!-- Segmented progress -->
          <div class="tour-progress" aria-hidden="true">
            <span
              v-for="n in steps.length"
              :key="n"
              class="tour-progress__seg"
              :class="{ 'is-done': n - 1 < stepIndex, 'is-current': n - 1 === stepIndex }"
            />
          </div>

          <button class="tour-card__close" @click="stop" aria-label="Skip tour">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round">
              <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
            </svg>
          </button>

          <div v-if="isWelcome" class="tour-card__logo">
            <img :src="logoSrc" alt="Globalgle" />
          </div>

          <span class="tour-card__eyebrow">
            {{ isWelcome ? 'Guided tour' : `Step ${stepIndex + 1} of ${steps.length}` }}
          </span>
          <h3 class="tour-card__title">{{ current.title }}</h3>
          <p class="tour-card__body">{{ current.body }}</p>

          <div class="tour-card__foot">
            <button class="tour-skip" @click="stop">Skip tour</button>
            <div class="tour-card__nav">
              <button v-if="stepIndex > 0" class="tour-btn tour-btn--ghost" @click="prev">
                <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.6" stroke-linecap="round" stroke-linejoin="round">
                  <line x1="19" y1="12" x2="5" y2="12"/><polyline points="12 19 5 12 12 5"/>
                </svg>
                Back
              </button>
              <button class="tour-btn tour-btn--primary" @click="next">
                {{ isLast ? 'Finish' : isWelcome ? 'Start tour' : 'Next' }}
                <svg v-if="!isLast" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.6" stroke-linecap="round" stroke-linejoin="round">
                  <line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { ref, computed, watch, nextTick, onBeforeUnmount } from 'vue'
import { useTour } from '../../composables/useTour.js'
import logo1 from '../../assets/logo1.jpg'

const { active, stepIndex, steps, stop, next, prev } = useTour()
const logoSrc = logo1

const box = ref(null) // spotlight rect {top,left,width,height}
const mode = ref('center') // 'center' | 'spot'
const cardEl = ref(null)
const cardPos = ref({ top: 0, left: 0 })

const current = computed(() => steps.value[stepIndex.value] ?? { title: '', body: '' })
const isWelcome = computed(() => stepIndex.value === 0 && mode.value === 'center')
const isLast = computed(() => stepIndex.value === steps.value.length - 1)

const PAD = 8 // spotlight padding around target

function findEl() {
  const step = steps.value[stepIndex.value]
  if (!step || step.center || !step.target) return null
  return document.querySelector(step.target)
}

/* Resolve current step: centered, spotlight, or auto-skip when target missing. */
function resolve() {
  if (!active.value) return
  const step = steps.value[stepIndex.value]
  if (!step) { stop(); return }

  if (step.center || !step.target) {
    mode.value = 'center'
    box.value = null
    return
  }

  const el = document.querySelector(step.target)
  if (!el) {
    if (stepIndex.value < steps.value.length - 1) { next(); return }
    stop(); return
  }
  mode.value = 'spot'
  el.scrollIntoView({ block: 'center', inline: 'nearest', behavior: 'smooth' })
  requestAnimationFrame(() => setTimeout(measure, 170))
}

function measure() {
  if (!active.value || mode.value !== 'spot') return
  const el = findEl()
  if (!el) { resolve(); return }
  const r = el.getBoundingClientRect()
  box.value = {
    top: Math.max(r.top - PAD, 6),
    left: Math.max(r.left - PAD, 6),
    width: Math.min(r.width + PAD * 2, window.innerWidth - 12),
    height: r.height + PAD * 2,
  }
  nextTick(placeCard)
}

function placeCard() {
  const b = box.value
  if (!b) return
  const vw = window.innerWidth
  const vh = window.innerHeight
  const card = cardEl.value
  const cw = card?.offsetWidth || 340
  const ch = card?.offsetHeight || 200
  const gap = 16

  let top = b.top + b.height + gap
  if (top + ch > vh - 10) top = b.top - ch - gap // flip above
  if (top < 10) top = 10
  if (top + ch > vh - 10) top = Math.max(10, vh - ch - 10)

  let left = b.left + b.width / 2 - cw / 2
  left = Math.min(Math.max(left, 10), vw - cw - 10)

  cardPos.value = { top, left }
}

const cardStyle = computed(() => ({
  top: cardPos.value.top + 'px',
  left: cardPos.value.left + 'px',
}))

function onViewportChange() {
  if (active.value && mode.value === 'spot') measure()
}

watch([active, stepIndex], async () => {
  if (active.value) { await nextTick(); resolve() }
  else box.value = null
})

watch(active, (on) => {
  if (on) {
    window.addEventListener('resize', onViewportChange)
    window.addEventListener('scroll', onViewportChange, true)
    document.addEventListener('keydown', onKey)
    document.body.style.overflow = 'hidden'
  } else {
    window.removeEventListener('resize', onViewportChange)
    window.removeEventListener('scroll', onViewportChange, true)
    document.removeEventListener('keydown', onKey)
    document.body.style.overflow = ''
  }
})

function onKey(e) {
  if (!active.value) return
  if (e.key === 'Escape') stop()
  else if (e.key === 'ArrowRight' || e.key === 'Enter') next()
  else if (e.key === 'ArrowLeft') prev()
}

onBeforeUnmount(() => {
  window.removeEventListener('resize', onViewportChange)
  window.removeEventListener('scroll', onViewportChange, true)
  document.removeEventListener('keydown', onKey)
  document.body.style.overflow = ''
})
</script>

<style scoped>
.tour-root { position: fixed; inset: 0; z-index: 4000; }

/* Full dimmer for the centered welcome step (moss-tinted, softly blurred) */
.tour-dim {
  position: fixed; inset: 0;
  background: rgba(24, 32, 8, 0.55);
  backdrop-filter: blur(5px);
  -webkit-backdrop-filter: blur(5px);
}

/* Spotlight: transparent box whose huge shadow dims everything else,
   outlined in Tea Green with a soft moss glow */
.tour-spot {
  position: fixed;
  border-radius: 18px;
  outline: 2px solid var(--tea);
  outline-offset: 3px;
  box-shadow:
    0 0 0 9999px rgba(24, 32, 8, 0.55),
    0 0 0 6px rgba(199, 213, 159, 0.22);
  transition: top 0.28s cubic-bezier(0.4,0,0.2,1), left 0.28s cubic-bezier(0.4,0,0.2,1),
              width 0.28s cubic-bezier(0.4,0,0.2,1), height 0.28s cubic-bezier(0.4,0,0.2,1);
  pointer-events: none;
}

/* ── Card ── */
.tour-card {
  position: fixed;
  width: min(360px, calc(100vw - 20px));
  background: var(--bg, #fff);
  border: 1px solid var(--tea);
  border-radius: 24px;
  padding: 22px 22px 18px;
  box-shadow: 0 24px 60px rgba(24, 32, 8, 0.32);
  animation: tour-pop 0.26s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.tour-card--center {
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  width: min(400px, calc(100vw - 24px));
  padding-top: 26px;
  text-align: center;
  animation: tour-pop-center 0.26s cubic-bezier(0.34, 1.56, 0.64, 1);
}
@keyframes tour-pop {
  from { opacity: 0; transform: scale(0.94) translateY(6px); }
  to { opacity: 1; transform: scale(1) translateY(0); }
}
@keyframes tour-pop-center {
  from { opacity: 0; transform: translate(-50%, -50%) scale(0.94); }
  to { opacity: 1; transform: translate(-50%, -50%) scale(1); }
}

/* Segmented progress */
.tour-progress { display: flex; gap: 5px; margin-bottom: 18px; padding-right: 34px; }
.tour-card--center .tour-progress { padding-right: 0; }
.tour-progress__seg {
  flex: 1; height: 4px; border-radius: 999px;
  background: var(--tea-soft);
  transition: background 0.3s ease;
}
.tour-progress__seg.is-done { background: var(--moss-soft); }
.tour-progress__seg.is-current { background: var(--moss); }

.tour-card__close {
  position: absolute; top: 14px; right: 14px;
  width: 28px; height: 28px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  background: var(--tea-soft); border: none;
  color: var(--moss); cursor: pointer;
  transition: background 0.18s, color 0.18s;
}
.tour-card__close:hover { background: var(--moss); color: var(--tea); }

/* Welcome logo */
.tour-card__logo {
  width: 58px; height: 58px; margin: 0 auto 14px;
  border-radius: 50%; padding: 3px; background: var(--tea);
}
.tour-card__logo img { width: 100%; height: 100%; border-radius: 50%; object-fit: cover; display: block; }

.tour-card__eyebrow {
  display: inline-block;
  padding: 4px 11px; border-radius: 999px;
  font-size: 0.64rem; font-weight: 800; letter-spacing: 0.1em; text-transform: uppercase;
  color: var(--moss-deep); background: var(--tea);
}

.tour-card__title {
  margin: 12px 0 8px;
  font-size: 1.12rem; font-weight: 800; color: var(--moss);
  letter-spacing: -0.035em; line-height: 1.3;
}
.tour-card__body {
  margin: 0; font-size: 0.82rem; line-height: 1.65; color: var(--t2, #555);
}

.tour-card__foot {
  display: flex; align-items: center; justify-content: space-between;
  gap: 12px; margin-top: 20px; padding-top: 14px;
  border-top: 1px solid var(--border-soft);
}
.tour-card--center .tour-card__foot { flex-direction: row-reverse; }
.tour-card__nav { display: flex; align-items: center; gap: 8px; }

.tour-skip {
  background: none; border: none; cursor: pointer; padding: 6px 4px;
  font-family: inherit; font-size: 0.76rem; font-weight: 700; color: var(--t3);
  transition: color 0.18s ease;
}
.tour-skip:hover { color: var(--moss); }

.tour-btn {
  display: inline-flex; align-items: center; gap: 6px;
  height: 38px; padding: 0 18px; border-radius: 999px;
  font-family: inherit; font-size: 0.8rem; font-weight: 800; letter-spacing: -0.02em;
  cursor: pointer; border: 1px solid transparent;
  transition: background 0.18s ease, transform 0.12s ease, color 0.18s ease, border-color 0.18s ease;
}
.tour-btn:active { transform: scale(0.96); }
.tour-btn--ghost { background: var(--bg); border-color: var(--border); color: var(--t1); }
.tour-btn--ghost:hover { border-color: var(--accent-border); background: var(--accent-dim); }
.tour-btn--primary {
  background: var(--moss); color: var(--tea);
  box-shadow: 0 8px 18px rgba(64, 83, 26, 0.28);
}
.tour-btn--primary:hover { background: var(--moss-soft); }

.tour-fade-enter-active, .tour-fade-leave-active { transition: opacity 0.24s ease; }
.tour-fade-enter-from, .tour-fade-leave-to { opacity: 0; }
</style>
