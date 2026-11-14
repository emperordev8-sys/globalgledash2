<!-- src/components/ui/TourLauncher.vue -->
<template>
  <Teleport to="body">
    <Transition name="fab-pop">
      <button
        v-if="!active"
        class="tour-fab"
        @click="launch"
        aria-label="Take the tour"
      >
        <span class="tour-fab__pulse" />
        <span class="tour-fab__badge">
          <svg width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="10"/>
            <polygon points="16.24 7.76 14.12 14.12 7.76 16.24 9.88 9.88 16.24 7.76"/>
          </svg>
        </span>
        <span class="tour-fab__text">
          <span class="tour-fab__label">Take the tour</span>
          <span class="tour-fab__sub">{{ stepCount }} quick steps</span>
        </span>
      </button>
    </Transition>
  </Teleport>
</template>

<script setup>
import { useRouter, useRoute } from 'vue-router'
import { useTour, defaultTourSteps } from '../../composables/useTour.js'

const router = useRouter()
const route = useRoute()
const { active, start } = useTour()
const stepCount = defaultTourSteps.length

function launch() {
  const onDash = route.path === '/'
  if (!onDash) router.push('/')
  // let the dashboard render before the tour measures its targets
  setTimeout(() => start(), onDash ? 50 : 450)
}
</script>

<style scoped>
/* Floating glass pill — same frosted recipe as the navbar capsule,
   outlined in Tea Green with Dark Moss content */
.tour-fab {
  position: fixed;
  right: 22px;
  bottom: 22px;
  z-index: 3000;
  display: inline-flex;
  align-items: center;
  gap: 11px;
  height: 56px;
  padding: 0 22px 0 8px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.19);
  backdrop-filter: blur(23px);
  -webkit-backdrop-filter: blur(23px);
  border: 1.5px solid var(--tea);
  box-shadow:
    0 8px 32px rgba(64, 83, 26, 0.16),
    inset 0 1px 0 rgba(255, 255, 255, 0.5),
    inset 0 -1px 0 rgba(255, 255, 255, 0.1),
    inset 0 0 6px 3px rgba(255, 255, 255, 0.3);
  color: var(--moss);
  font-family: var(--font);
  cursor: pointer;
  transition: transform 0.18s ease, background 0.2s ease, border-color 0.2s ease;
}
.tour-fab:hover {
  transform: translateY(-2px);
  background: rgba(255, 255, 255, 0.34);
  border-color: var(--moss-soft);
}
.tour-fab:active { transform: translateY(0); }

/* Moss icon badge */
.tour-fab__badge {
  position: relative; z-index: 2;
  width: 40px; height: 40px; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  border-radius: 50%;
  background: linear-gradient(145deg, var(--moss-soft) 0%, var(--moss) 60%, var(--moss-deep) 100%);
  color: var(--tea);
}
.tour-fab:hover .tour-fab__badge svg { animation: fab-spin 0.7s ease; }
@keyframes fab-spin { from { transform: rotate(0); } to { transform: rotate(360deg); } }

.tour-fab__text {
  position: relative; z-index: 2;
  display: flex; flex-direction: column; align-items: flex-start; gap: 1px;
  line-height: 1.15;
}
.tour-fab__label { white-space: nowrap; font-size: 0.86rem; font-weight: 800; letter-spacing: -0.02em; color: var(--moss-deep); }
.tour-fab__sub { white-space: nowrap; font-size: 0.66rem; font-weight: 600; color: var(--t2); }

/* Soft attention ring in Moss */
.tour-fab__pulse {
  position: absolute;
  inset: -1.5px;
  border-radius: 999px;
  border: 2px solid var(--moss);
  z-index: -1;
  pointer-events: none;
  animation: fab-pulse 2.8s ease-out infinite;
}
@keyframes fab-pulse {
  0% { transform: scale(1); opacity: 0.4; }
  70% { transform: scale(1.14, 1.3); opacity: 0; }
  100% { transform: scale(1.14, 1.3); opacity: 0; }
}

.fab-pop-enter-active { transition: transform 0.28s cubic-bezier(0.34, 1.56, 0.64, 1), opacity 0.2s ease; }
.fab-pop-leave-active { transition: transform 0.18s ease, opacity 0.18s ease; }
.fab-pop-enter-from, .fab-pop-leave-to { opacity: 0; transform: scale(0.8) translateY(8px); }

/* Small screens: tuck above the bottom gutter, drop the sub-label */
@media (max-width: 768px) {
  .tour-fab {
    right: 14px;
    bottom: 18px;
    height: 50px;
    padding: 0 18px 0 6px;
  }
  .tour-fab__badge { width: 38px; height: 38px; }
  .tour-fab__sub { display: none; }
}

@media (prefers-reduced-motion: reduce) {
  .tour-fab__pulse { animation: none; display: none; }
  .tour-fab:hover .tour-fab__badge svg { animation: none; }
}
</style>
