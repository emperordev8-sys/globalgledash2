<!-- src/components/ui/BootSplash.vue -->
<template>
  <Transition name="boot-fade">
    <div v-if="visible" class="boot">
      <div class="boot__center">
        <div class="boot__logo-wrap">
          <img :src="logoSrc" alt="Globalgle" class="boot__logo" />
        </div>
        <span class="boot__name">Globalgle</span>

        <div class="boot__loader">
          <div class="boot__meta">
            <span class="boot__label">Loading</span>
            <span class="boot__pct">{{ pct }}%</span>
          </div>
          <div class="boot__bar" role="progressbar" aria-valuemin="0" aria-valuemax="100" :aria-valuenow="pct">
            <div class="boot__fill" :style="{ width: pct + '%' }"></div>
          </div>
        </div>
      </div>
    </div>
  </Transition>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useBootSplash } from '../../composables/useBootSplash.js'
import logo1 from '../../assets/logo1.jpg'

const { finishBooting } = useBootSplash()
const logoSrc = logo1

const visible = ref(true)
const pct = ref(0)

onMounted(() => {
  const duration = 4200
  const start = performance.now()

  function tick(now) {
    const linear = Math.min((now - start) / duration, 1)
    // ease-out so it feels fast then settles — more premium than linear
    const eased = 1 - Math.pow(1 - linear, 2)
    pct.value = Math.round(eased * 100)
    if (linear < 1) {
      requestAnimationFrame(tick)
    } else {
      setTimeout(() => {
        visible.value = false
        finishBooting()
      }, 320)
    }
  }
  requestAnimationFrame(tick)
})
</script>

<style scoped>
.boot {
  position: fixed;
  inset: 0;
  z-index: 100000;
  display: flex;
  align-items: center;
  justify-content: center;
  /* Tea canvas with soft moss / tea glows */
  background:
    radial-gradient(60% 50% at 50% 38%, rgba(199, 213, 159, 0.55) 0%, transparent 70%),
    radial-gradient(40% 40% at 100% 100%, rgba(64, 83, 26, 0.10) 0%, transparent 70%),
    var(--canvas);
}

.boot__center {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
}

/* ── Logo — small circle with a tea ring ── */
.boot__logo-wrap {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  padding: 3px;
  background: var(--tea);
  animation: boot-rise 0.6s cubic-bezier(0.16, 1, 0.3, 1) both;
}
.boot__logo {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  object-fit: cover;
  display: block;
}

.boot__name {
  font-size: 1.05rem;
  font-weight: 800;
  letter-spacing: -0.02em;
  color: var(--moss);
  animation: boot-rise 0.6s cubic-bezier(0.16, 1, 0.3, 1) 0.1s both;
}

/* ── Bar loader ── */
.boot__loader {
  width: 260px;
  max-width: 72vw;
  margin-top: 8px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  animation: boot-rise 0.6s cubic-bezier(0.16, 1, 0.3, 1) 0.18s both;
}
.boot__meta {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
}
.boot__label {
  font-size: 0.68rem;
  font-weight: 700;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: var(--t3);
}
.boot__pct {
  font-size: 0.8rem;
  font-weight: 800;
  color: var(--moss);
  font-variant-numeric: tabular-nums;
}

.boot__bar {
  position: relative;
  height: 10px;
  border-radius: 999px;
  background: rgba(64, 83, 26, 0.12);
  box-shadow: inset 0 1px 2px rgba(64, 83, 26, 0.15);
  overflow: hidden;
}
.boot__fill {
  position: relative;
  height: 100%;
  border-radius: 999px;
  background: linear-gradient(90deg, #2f3e12 0%, #40531a 45%, #6b8730 100%);
  transition: width 0.18s ease-out;
  overflow: hidden;
}
.boot__fill::after {
  content: '';
  position: absolute; inset: 0;
  background: linear-gradient(100deg, transparent 0%, rgba(199, 213, 159, 0.65) 50%, transparent 100%);
  transform: translateX(-100%);
  animation: boot-sheen 1.1s ease-in-out infinite;
}

@keyframes boot-rise {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes boot-sheen {
  0% { transform: translateX(-100%); }
  60%, 100% { transform: translateX(100%); }
}

@media (prefers-reduced-motion: reduce) {
  .boot__fill::after { animation: none; }
}

.boot-fade-enter-active { transition: opacity 0.2s ease; }
.boot-fade-leave-active { transition: opacity 0.45s ease; }
.boot-fade-enter-from,
.boot-fade-leave-to { opacity: 0; }
</style>
