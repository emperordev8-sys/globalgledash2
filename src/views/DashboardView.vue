<!-- src/views/DashboardView.vue -->
<template>
  <div class="dash">
    <CommunityModal />

    <!-- ════════════════════════════════════
         WELCOME HEADER
         ════════════════════════════════════ -->
    <div class="dash-toolbar" :class="{ 'cascade-play': cascadeReady }">
      <div class="dash-head">
        <h1 class="dash-head__title">Welcome back, Daniel!</h1>
        <p class="dash-head__sub">At-a-glance view of your account, subscription, and recent activity.</p>
      </div>
      <div class="dash-toolbar__actions" data-tour="header-actions">
        <button class="dtb-btn dtb-btn--primary" @click="router.push('/tutorials')">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="10"/><polygon points="10 8 16 12 10 16 10 8"/>
          </svg>
          Watch to learn
        </button>
        <button class="dtb-btn dtb-btn--ghost" @click="router.push('/billing')">Choose plan</button>
        <button class="dtb-btn dtb-btn--tea" @click="router.push('/wallet')">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round">
            <line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/>
          </svg>
          Top up wallet
        </button>
      </div>
    </div>

    <!-- ════════════════════════════════════
         OVERVIEW — stat cards + activity chart
         ════════════════════════════════════ -->
    <div class="dash-overview">

      <div class="dash-stats" data-tour="stats">
        <div
          v-for="(s, i) in stats"
          :key="s.label"
          class="stat-card"
          :class="[
            { 'cascade-play': cascadeReady },
            i === 0 ? 'stat-card--hero' : i === 1 ? 'stat-card--tea' : 'stat-card--plain',
          ]"
        >
          <span class="stat-card__deco" aria-hidden="true"></span>
          <div class="stat-card__top">
            <span class="stat-card__lbl">{{ s.label }}</span>

            <!-- Wallet balance card → "Top up wallet" action -->
            <button v-if="i === 0" class="stat-card__topup" @click.stop="router.push('/wallet')" aria-label="Top up wallet">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M20 7H5a2 2 0 0 1 0-4h13v4"/><path d="M3 5v14a2 2 0 0 0 2 2h15V7"/><path d="M16 14h.01"/>
              </svg>
              Top up wallet
            </button>

            <span v-else class="stat-card__ico">
              <svg v-if="s.icon === 'card'" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <rect x="2" y="5" width="20" height="14" rx="2"/><line x1="2" y1="10" x2="22" y2="10"/>
              </svg>
              <svg v-else width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <rect x="4" y="2" width="16" height="20" rx="2"/><path d="M14.5 8.5h-3.25a1.75 1.75 0 0 0 0 3.5h1.5a1.75 1.75 0 0 1 0 3.5H9.5"/><line x1="12" y1="6.5" x2="12" y2="17.5"/>
              </svg>
            </span>
          </div>
          <span class="stat-card__val">{{ s.value }}</span>
        </div>
      </div>

      <!-- Activity chart -->
      <div class="chart-card" :class="{ 'cascade-play': cascadeReady }">
        <div class="chart-card__hdr">
          <div>
            <h2 class="chart-card__title">Wallet activity</h2>
            <p class="chart-card__sub">Funds in and out · last 7 days</p>
          </div>
          <span class="chart-card__chip">This week</span>
        </div>

        <div class="chart-card__body">
          <svg class="chart" :viewBox="`0 0 600 200`" preserveAspectRatio="none" role="img" aria-label="Wallet activity for the last 7 days">
            <defs>
              <linearGradient id="chartFill" x1="0" y1="0" x2="0" y2="1">
                <stop offset="0%" stop-color="var(--accent)" stop-opacity="0.32"/>
                <stop offset="100%" stop-color="var(--accent)" stop-opacity="0"/>
              </linearGradient>
            </defs>
            <line v-for="n in 4" :key="n" x1="0" x2="600" :y1="(n - 1) * 62 + 14" :y2="(n - 1) * 62 + 14" class="chart__grid"/>
            <path :d="areaPath" fill="url(#chartFill)"/>
            <path :d="linePath" class="chart__line"/>
            <circle v-for="(p, i) in chartPoints" :key="i" :cx="p.x" :cy="p.y" r="4.5" class="chart__dot"/>
          </svg>

          <div v-if="chartEmpty" class="chart__empty">
            <span class="chart__empty-title">No activity yet</span>
            <span class="chart__empty-sub">Top up your wallet and your activity will appear here.</span>
          </div>
        </div>

        <div class="chart-card__axis">
          <span v-for="d in chartDays" :key="d">{{ d }}</span>
        </div>
      </div>
    </div>

    <!-- ════════════════════════════════════
         WORKSPACES
         (class "quick-actions" is kept as the entrance-animation + tour hook)
         ════════════════════════════════════ -->
    <section class="quick-actions workspaces" :class="{ 'cascade-play': cascadeReady }" data-tour="quick-actions">
      <header class="ws-head">
        <h2 class="ws-head__title">Workspaces</h2>
        <p class="ws-head__sub">Jump straight into any area of your toolkit.</p>
      </header>

      <div class="ws-grid">
        <button
          v-for="(w, i) in workspaces"
          :key="w.title"
          class="ws-card"
          :class="{ 'ws-card--featured': i === 0 }"
          @click="router.push(w.route)"
        >
          <span class="ws-card__deco" aria-hidden="true"></span>

          <span class="ws-card__top">
            <span class="ws-card__stack">
              <img
                v-for="(src, k) in w.images"
                :key="k"
                :src="src"
                alt=""
                class="ws-card__img"
                :style="{ zIndex: w.images.length - k }"
              />
            </span>
            <span class="ws-card__count">{{ w.count }} {{ w.count === 1 ? 'tool' : 'tools' }}</span>
          </span>

          <span class="ws-card__body">
            <span class="ws-card__title">{{ w.title }}</span>
            <span class="ws-card__desc">{{ w.desc }}</span>
          </span>

          <span class="ws-card__foot">
            <span class="ws-card__cta">Open workspace</span>
            <span class="ws-card__arrow" aria-hidden="true">
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round">
                <line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/>
              </svg>
            </span>
          </span>
        </button>
      </div>
    </section>

    <!-- ════════════════════════════════════
         RECENT ACTIVITY (empty states)
         ════════════════════════════════════ -->
    <div class="dash-recent" :class="{ 'cascade-play': cascadeReady }" data-tour="recent">
      <div class="recent-card">
        <div class="recent-card__hdr">
          <span class="recent-card__title">Recent wallet transactions</span>
          <button class="recent-card__link" @click="router.push('/wallet')">
            See all
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.3" stroke-linecap="round" stroke-linejoin="round">
              <line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/>
            </svg>
          </button>
        </div>
        <div class="recent-empty">
          <div class="recent-empty__icon">
            <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round">
              <path d="M20 7H5a2 2 0 0 1 0-4h13v4"/><path d="M3 5v14a2 2 0 0 0 2 2h15V7"/><path d="M16 14h.01"/>
            </svg>
          </div>
          <p class="recent-empty__title">No transactions yet</p>
          <p class="recent-empty__sub">Top up your wallet to see activity here.</p>
        </div>
      </div>

      <div class="recent-card">
        <div class="recent-card__hdr">
          <span class="recent-card__title">Recent payments</span>
          <button class="recent-card__link" @click="router.push('/transactions')">
            See all
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.3" stroke-linecap="round" stroke-linejoin="round">
              <line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/>
            </svg>
          </button>
        </div>
        <div class="recent-empty">
          <div class="recent-empty__icon">
            <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round">
              <path d="M4 2v20l2-1 2 1 2-1 2 1 2-1 2 1 2-1 2 1V2l-2 1-2-1-2 1-2-1-2 1-2-1-2 1-2-1z"/><line x1="8" y1="9" x2="16" y2="9"/><line x1="8" y1="13" x2="14" y2="13"/>
            </svg>
          </div>
          <p class="recent-empty__title">No payments yet</p>
          <p class="recent-empty__sub">Choose a plan or fund your wallet.</p>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useBootSplash } from '../composables/useBootSplash.js'
import { useTour } from '../composables/useTour.js'
import CommunityModal from '../components/ui/CommunityModal.vue'

const router = useRouter()

/* ── Cascade entrance: wait for the boot splash to actually start
     revealing the page before playing the card fade-up, so the
     animation is visible instead of finishing hidden behind it ── */
const { booting } = useBootSplash()
const cascadeReady = ref(!booting.value)
watch(booting, (isBooting) => {
  if (!isBooting) cascadeReady.value = true
})

/* ── Auto-run the product tour once, for first-time visitors ── */
const { start: startTour } = useTour()
const TOUR_KEY = 'globalgle_tour_seen'
onMounted(() => {
  let seen = false
  try { seen = localStorage.getItem(TOUR_KEY) === '1' } catch (_) { /* ignore */ }
  if (seen) return
  try { localStorage.setItem(TOUR_KEY, '1') } catch (_) { /* ignore */ }
  // wait for the boot splash + entrance to settle, then greet the user
  const begin = () => setTimeout(() => startTour(), 1100)
  if (booting.value) {
    const unwatch = watch(booting, (b) => { if (!b) { unwatch(); begin() } })
  } else {
    begin()
  }
})

/* ── Stat cards ── */
const stats = [
  { label: 'Wallet balance', value: '₦0.00', icon: 'wallet' },
  { label: 'Active plan', value: 'None', icon: 'card' },
  { label: 'Recent payments', value: '0', icon: 'receipt' },
]

/* ── Wallet activity chart ──
   Driven by real values (₦ per day, oldest → newest). All zeros until the
   account has activity, in which case an empty-state hint is shown. */
const chartDays = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
const activity = ref([0, 0, 0, 0, 0, 0, 0])
const chartEmpty = computed(() => activity.value.every(v => v === 0))

const CW = 600, CH = 200, PAD = 14
const chartPoints = computed(() => {
  const max = Math.max(...activity.value, 1)
  const step = (CW - PAD * 2) / (activity.value.length - 1)
  return activity.value.map((v, i) => ({
    x: PAD + i * step,
    y: CH - PAD - (v / max) * (CH - PAD * 2),
  }))
})
// Smooth line through the points (cubic bezier with mid-point control handles)
const linePath = computed(() => {
  const p = chartPoints.value
  return p.reduce((d, pt, i) => {
    if (i === 0) return `M ${pt.x} ${pt.y}`
    const prev = p[i - 1]
    const cx = (prev.x + pt.x) / 2
    return `${d} C ${cx} ${prev.y}, ${cx} ${pt.y}, ${pt.x} ${pt.y}`
  }, '')
})
const areaPath = computed(() => {
  const p = chartPoints.value
  return `${linePath.value} L ${p[p.length - 1].x} ${CH} L ${p[0].x} ${CH} Z`
})

/* ── Workspaces ──
   Tool artwork lives in src/assets — one overlapping stack per workspace. */
const PIC = import.meta.glob('../assets/1000880*.png', { eager: true, import: 'default' })
const pic = (n) => PIC[`../assets/1000880${n}.png`]

const workspaces = [
  {
    title: 'Website Creator',
    desc: 'Launch and manage the sites you offer your customers.',
    count: 13,
    images: [pic('289'), pic('290'), pic('291'), pic('294')],
    route: '/website-creator',
  },
  {
    title: 'Digital Tools',
    desc: 'SMS, eSIM, wallet funding and account services for your business.',
    count: 6,
    images: [pic('301'), pic('302'), pic('304'), pic('305')],
    route: '/digital',
  },
  {
    title: 'Call Tools',
    desc: 'Place calls with your own caller ID, voice and language.',
    count: 2,
    images: [pic('307'), pic('308')],
    route: '/tools/call',
  },
  {
    title: 'Mailing Tools',
    desc: 'Branded emails, billing notices and bank mailers in one place.',
    count: 3,
    images: [pic('310'), pic('309'), pic('312')],
    route: '/customers',
  },
  {
    title: 'Generator',
    desc: 'Generate branded receipts, trackers and documents on demand.',
    count: 3,
    images: [pic('314'), pic('313'), pic('315')],
    route: '/tools',
  },
]
</script>

<style scoped>
.dash { display: flex; flex-direction: column; gap: 22px; }

/* ══════════════════════════════════
   ENTRANCE — cascading fade-up
   ══════════════════════════════════ */
@keyframes dashFadeUp {
  from { opacity: 0; transform: translateY(26px) scale(0.98); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}

.dash-toolbar,
.dash-stats .stat-card,
.chart-card,
.quick-actions,
.dash-recent { opacity: 0; }

.dash-toolbar.cascade-play,
.dash-stats .stat-card.cascade-play,
.chart-card.cascade-play,
.quick-actions.cascade-play,
.dash-recent.cascade-play {
  animation: dashFadeUp 0.7s cubic-bezier(0.16, 1, 0.3, 1) both;
}

.dash-toolbar.cascade-play { animation-delay: 0.05s; }
.dash-stats .stat-card.cascade-play:nth-child(1) { animation-delay: 0.12s; }
.dash-stats .stat-card.cascade-play:nth-child(2) { animation-delay: 0.20s; }
.dash-stats .stat-card.cascade-play:nth-child(3) { animation-delay: 0.28s; }
.chart-card.cascade-play { animation-delay: 0.22s; }
.quick-actions.cascade-play { animation-delay: 0.36s; }
.dash-recent.cascade-play { animation-delay: 0.46s; }

@media (prefers-reduced-motion: reduce) {
  .dash-toolbar, .dash-stats .stat-card, .chart-card, .quick-actions, .dash-recent {
    opacity: 1; animation: none;
  }
}

/* ══════════════════════════════════
   WELCOME HEADER
   ══════════════════════════════════ */
.dash-toolbar {
  display: flex; align-items: flex-start; justify-content: space-between;
  gap: 16px; flex-wrap: wrap;
}
.dash-head { display: flex; flex-direction: column; gap: 6px; min-width: 0; }
.dash-head__title {
  margin: 0;
  font-size: 1.9rem; font-weight: 800; line-height: 1.15;
  letter-spacing: -0.04em;
  color: var(--accent);
}
.dash-head__sub { margin: 0; font-size: 0.84rem; color: var(--t2); font-weight: 500; }

.dash-toolbar__actions { display: flex; align-items: center; gap: 10px; flex-wrap: wrap; }

.dtb-btn {
  display: inline-flex; align-items: center; justify-content: center; gap: 7px;
  height: 42px; padding: 0 20px; border-radius: 999px;
  font-family: inherit; font-size: 0.8rem; font-weight: 800; letter-spacing: -0.02em;
  cursor: pointer; white-space: nowrap;
  border: 1px solid transparent;
  transition: background 0.18s ease, transform 0.12s ease, border-color 0.18s ease;
}
.dtb-btn:active { transform: scale(0.97); }

/* Moss — primary */
.dtb-btn--primary {
  background: var(--moss); color: var(--tea);
  box-shadow: 0 8px 20px rgba(64, 83, 26, 0.28);
}
.dtb-btn--primary:hover { background: var(--moss-soft); }

/* Tea — secondary highlight */
.dtb-btn--tea {
  background: var(--tea); color: var(--moss-deep);
  box-shadow: 0 8px 20px rgba(169, 188, 120, 0.35);
}
.dtb-btn--tea:hover { background: #d3e0ad; }

.dtb-btn--ghost {
  background: var(--bg); color: var(--t1);
  border-color: var(--border);
}
.dtb-btn--ghost:hover { border-color: var(--accent-border); background: var(--accent-dim); }

/* ══════════════════════════════════
   OVERVIEW GRID
   ══════════════════════════════════ */
.dash-overview {
  display: grid;
  grid-template-columns: minmax(280px, 0.78fr) minmax(0, 1.6fr);
  gap: 18px;
  align-items: stretch;
}

.dash-stats { display: flex; flex-direction: column; gap: 14px; }

.stat-card {
  position: relative; overflow: hidden;
  display: flex; flex-direction: column; justify-content: space-between; gap: 10px;
  padding: 18px 20px;
  border-radius: 24px;
  background: var(--bg);
  border: 1px solid var(--border);
  box-shadow: var(--shadow-card);
}
.stat-card--hero { flex: 1.35; min-height: 150px; }
.stat-card--tea, .stat-card--plain { flex: 1; min-height: 96px; }

/* Decorative soft circle (echoes the rounded shapes in the reference) */
.stat-card__deco {
  position: absolute; right: -34px; bottom: -44px;
  width: 150px; height: 150px; border-radius: 50%;
  pointer-events: none;
}

/* Moss hero */
.stat-card--hero {
  background:
    radial-gradient(120% 90% at 100% 0%, rgba(199, 213, 159, 0.28) 0%, transparent 58%),
    linear-gradient(150deg, #566e24 0%, #40531a 55%, #2f3e12 100%);
  border-color: rgba(199, 213, 159, 0.22);
  box-shadow: 0 16px 36px rgba(64, 83, 26, 0.30);
}
.stat-card--hero .stat-card__deco { background: rgba(199, 213, 159, 0.16); }
.stat-card--hero .stat-card__lbl { color: rgba(244, 248, 232, 0.78); }
.stat-card--hero .stat-card__val { color: #fff; font-size: 2.2rem; }

/* Tea card */
.stat-card--tea {
  background: var(--tea);
  border-color: rgba(64, 83, 26, 0.12);
  box-shadow: 0 12px 28px rgba(169, 188, 120, 0.32);
}
.stat-card--tea .stat-card__deco { background: rgba(255, 255, 255, 0.34); }
.stat-card--tea .stat-card__lbl { color: rgba(47, 62, 18, 0.72); }
.stat-card--tea .stat-card__val { color: var(--moss-deep); }
.stat-card--tea .stat-card__ico { background: rgba(64, 83, 26, 0.14); border-color: rgba(64, 83, 26, 0.22); color: var(--moss); }

/* Plain card */
.stat-card--plain .stat-card__deco { background: var(--accent-dim); }

.stat-card__top, .stat-card__val { position: relative; z-index: 1; }
.stat-card__top { display: flex; align-items: flex-start; justify-content: space-between; gap: 10px; }
.stat-card__lbl { font-size: 0.8rem; font-weight: 600; color: var(--t3); }
.stat-card__ico {
  width: 32px; height: 32px; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  border-radius: 10px;
  background: var(--accent-dim);
  border: 1px solid var(--accent-border);
  color: var(--accent);
}
.stat-card__val {
  font-size: 1.55rem; font-weight: 800; color: var(--t1);
  letter-spacing: -0.04em; line-height: 1.1;
}

/* "Top up wallet" chip on the hero */
.stat-card__topup {
  display: inline-flex; align-items: center; gap: 6px; flex-shrink: 0;
  padding: 7px 13px; border-radius: 999px; cursor: pointer;
  font-family: inherit; font-size: 0.72rem; font-weight: 800;
  background: var(--tea); color: var(--moss-deep);
  border: 0;
  transition: background 0.18s ease, transform 0.12s ease;
}
.stat-card__topup:hover { background: #d8e4b6; }
.stat-card__topup:active { transform: scale(0.97); }

/* ══════════════════════════════════
   ACTIVITY CHART CARD
   ══════════════════════════════════ */
.chart-card {
  display: flex; flex-direction: column;
  padding: 20px 22px 16px;
  border-radius: 24px;
  background: var(--bg);
  border: 1px solid var(--border);
  box-shadow: var(--shadow-card);
  min-width: 0;
}
.chart-card__hdr { display: flex; align-items: flex-start; justify-content: space-between; gap: 12px; }
.chart-card__title { margin: 0; font-size: 1rem; font-weight: 800; color: var(--accent); letter-spacing: -0.03em; }
.chart-card__sub { margin: 3px 0 0; font-size: 0.74rem; color: var(--t3); font-weight: 500; }
.chart-card__chip {
  padding: 6px 14px; border-radius: 999px;
  font-size: 0.72rem; font-weight: 800;
  background: var(--moss); color: var(--tea);
}
:root:not([data-theme="light"]) .chart-card__chip { background: var(--tea); color: var(--moss-deep); }

.chart-card__body { position: relative; flex: 1; min-height: 170px; margin-top: 14px; }
.chart { position: absolute; inset: 0; width: 100%; height: 100%; display: block; overflow: visible; }
.chart__grid { stroke: var(--border); stroke-width: 1; stroke-dasharray: 3 5; vector-effect: non-scaling-stroke; }
.chart__line {
  fill: none; stroke: var(--accent); stroke-width: 3;
  stroke-linecap: round; stroke-linejoin: round; vector-effect: non-scaling-stroke;
}
.chart__dot { fill: var(--bg); stroke: var(--accent); stroke-width: 2.5; vector-effect: non-scaling-stroke; }

.chart__empty {
  position: absolute; left: 50%; top: 42%; transform: translate(-50%, -50%);
  display: flex; flex-direction: column; align-items: center; gap: 4px;
  padding: 12px 18px; border-radius: 16px; text-align: center;
  background: var(--bg); border: 1px solid var(--border);
  box-shadow: var(--shadow-card);
  max-width: 88%;
}
.chart__empty-title { font-size: 0.86rem; font-weight: 800; color: var(--t1); }
.chart__empty-sub { font-size: 0.72rem; color: var(--t3); font-weight: 500; }

.chart-card__axis {
  display: flex; justify-content: space-between;
  padding: 10px 14px 0;
  font-size: 0.68rem; font-weight: 600; color: var(--t3);
}

/* ══════════════════════════════════
   WORKSPACES
   ══════════════════════════════════ */
.quick-actions { display: flex; flex-direction: column; gap: 16px; margin-top: 4px; }

.ws-head { display: flex; flex-direction: column; gap: 4px; }
.ws-head__title { margin: 0; font-size: 1.15rem; font-weight: 800; color: var(--accent); letter-spacing: -0.035em; }
.ws-head__sub { margin: 0; font-size: 0.8rem; font-weight: 500; color: var(--t3); }

/* 6-col grid: three cards on the first row, two wider on the second */
.ws-grid { display: grid; grid-template-columns: repeat(6, 1fr); gap: 16px; }

.ws-card {
  grid-column: span 2;
  position: relative; overflow: hidden;
  display: flex; flex-direction: column; justify-content: space-between; gap: 22px;
  min-height: 210px;
  padding: 20px 22px;
  border-radius: 26px;
  background: var(--bg);
  border: 1px solid var(--border);
  box-shadow: var(--shadow-card);
  text-align: left; cursor: pointer;
  font-family: inherit; color: var(--t1);
  transition: transform 0.18s ease, border-color 0.2s ease, box-shadow 0.2s ease;
}
.ws-card:nth-child(n+4) { grid-column: span 3; min-height: 170px; }
.ws-card:hover { transform: translateY(-4px); border-color: var(--accent-border); box-shadow: var(--shadow-float); }
.ws-card:active { transform: translateY(-1px); }

/* soft decorative circle */
.ws-card__deco {
  position: absolute; right: -40px; bottom: -56px;
  width: 190px; height: 190px; border-radius: 50%;
  background: var(--tea); opacity: 0.45;
  transition: transform 0.35s ease, opacity 0.35s ease;
  pointer-events: none;
}
.ws-card:hover .ws-card__deco { transform: scale(1.18); opacity: 0.7; }

.ws-card__top, .ws-card__body, .ws-card__foot { position: relative; z-index: 1; }

.ws-card__top { display: flex; align-items: center; justify-content: space-between; gap: 10px; }
/* Overlapping tool-artwork stack */
.ws-card { --ring: var(--bg); }
.ws-card--featured { --ring: #4a5f1e; }
.ws-card__stack { display: flex; align-items: center; }
.ws-card__img {
  position: relative;
  width: 52px; height: 52px; flex-shrink: 0;
  border-radius: 16px;
  object-fit: cover; display: block;
  background: #0b0f08;
  box-shadow: 0 0 0 3px var(--ring);
  transition: transform 0.25s ease;
}
.ws-card__img + .ws-card__img { margin-left: -16px; }
.ws-card:hover .ws-card__img { transform: translateY(-2px); }
.ws-card:hover .ws-card__img:nth-child(2) { transition-delay: 0.04s; }
.ws-card:hover .ws-card__img:nth-child(3) { transition-delay: 0.08s; }
.ws-card:hover .ws-card__img:nth-child(4) { transition-delay: 0.12s; }
/* Glass chip — navbar recipe (tea-tinted so it reads on white cards) */
.ws-card__count {
  position: relative;
  padding: 6px 14px; border-radius: 999px;
  font-size: 0.68rem; font-weight: 800; letter-spacing: 0.02em;
  color: var(--moss-deep);
  background: rgba(199, 213, 159, 0.34);
  backdrop-filter: blur(23px);
  -webkit-backdrop-filter: blur(23px);
  border: 1px solid rgba(255, 255, 255, 0.5);
  box-shadow:
    0 6px 20px rgba(64, 83, 26, 0.10),
    inset 0 1px 0 rgba(255, 255, 255, 0.6),
    inset 0 -1px 0 rgba(255, 255, 255, 0.15),
    inset 0 0 6px 3px rgba(255, 255, 255, 0.3);
}
.ws-card__count::before {
  content: ''; position: absolute; pointer-events: none;
  top: 0; left: 12px; right: 12px; height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.9), transparent);
}

.ws-card__body { display: flex; flex-direction: column; gap: 6px; }
.ws-card__title { font-size: 1.05rem; font-weight: 800; letter-spacing: -0.035em; color: var(--t1); }
.ws-card__desc { font-size: 0.78rem; font-weight: 500; line-height: 1.55; color: var(--t3); max-width: 34ch; }

.ws-card__foot { display: flex; align-items: center; justify-content: space-between; }
.ws-card__cta { font-size: 0.76rem; font-weight: 800; color: var(--accent); letter-spacing: -0.02em; }
.ws-card__arrow {
  width: 36px; height: 36px; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  border-radius: 50%;
  background: var(--moss); color: var(--tea);
  transition: transform 0.2s ease, background 0.2s ease;
}
.ws-card:hover .ws-card__arrow { transform: translateX(3px); background: var(--moss-soft); }

/* Featured (Website Creator) — moss */
.ws-card--featured {
  background:
    radial-gradient(120% 90% at 100% 0%, rgba(199, 213, 159, 0.28) 0%, transparent 58%),
    linear-gradient(150deg, #566e24 0%, #40531a 55%, #2f3e12 100%);
  border-color: rgba(199, 213, 159, 0.22);
  box-shadow: 0 16px 36px rgba(64, 83, 26, 0.30);
}
.ws-card--featured .ws-card__deco { background: rgba(199, 213, 159, 0.16); opacity: 1; }
.ws-card--featured .ws-card__count {
  background: rgba(255, 255, 255, 0.19);
  border-color: rgba(255, 255, 255, 0.3);
  color: var(--tea);
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.5),
    inset 0 -1px 0 rgba(255, 255, 255, 0.1),
    inset 0 0 6px 3px rgba(255, 255, 255, 0.3);
}
.ws-card--featured .ws-card__title { color: #fff; }
.ws-card--featured .ws-card__desc { color: rgba(244, 248, 232, 0.72); }
.ws-card--featured .ws-card__cta { color: var(--tea); }
.ws-card--featured .ws-card__arrow { background: var(--tea); color: var(--moss-deep); }
.ws-card--featured:hover .ws-card__arrow { background: #d8e4b6; }

/* ══════════════════════════════════
   RECENT ACTIVITY (empty states)
   ══════════════════════════════════ */
.dash-recent { display: grid; grid-template-columns: 1fr 1fr; gap: 18px; margin-top: 4px; }

.recent-card {
  background: var(--bg);
  border: 1px solid var(--border); border-radius: 24px;
  box-shadow: var(--shadow-card);
  display: flex; flex-direction: column;
}
.recent-card__hdr {
  display: flex; align-items: center; justify-content: space-between;
  padding: 16px 20px; border-bottom: 1px solid var(--border-soft);
}
.recent-card__title { font-size: 0.9rem; font-weight: 800; color: var(--accent); letter-spacing: -0.03em; }
.recent-card__link {
  display: inline-flex; align-items: center; gap: 5px;
  padding: 5px 12px; border-radius: 999px; cursor: pointer;
  background: var(--tea); border: 0;
  font-family: inherit; font-size: 0.72rem; font-weight: 800; color: var(--moss-deep);
  transition: background 0.18s ease;
}
.recent-card__link:hover { background: #d3e0ad; }

.recent-empty {
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  gap: 8px; text-align: center;
  padding: 40px 20px; min-height: 180px;
}
.recent-empty__icon {
  width: 50px; height: 50px; margin-bottom: 4px;
  display: flex; align-items: center; justify-content: center;
  border-radius: 16px;
  background: var(--accent-dim); color: var(--accent);
}
.recent-empty__title { margin: 0; font-size: 0.95rem; font-weight: 800; color: var(--t1); }
.recent-empty__sub { margin: 0; font-size: 0.8rem; color: var(--t3); font-weight: 500; }

/* ══════════════════════════════════
   RESPONSIVE
   ══════════════════════════════════ */
@media (max-width: 1100px) {
  .ws-grid { grid-template-columns: repeat(2, 1fr); }
  .ws-card, .ws-card:nth-child(n+4) { grid-column: span 1; min-height: 180px; }
  .ws-card:last-child { grid-column: 1 / -1; }
}
@media (max-width: 1024px) {
  .dash-overview { grid-template-columns: 1fr; }
  .dash-stats { flex-direction: row; }
  .stat-card { flex: 1 1 0; min-width: 0; }
  .stat-card--hero { flex: 1.3 1 0; }
}
@media (max-width: 760px) {
  .dash-recent { grid-template-columns: 1fr; }
}
@media (max-width: 640px) {
  .dash-head__title { font-size: 1.5rem; }
  .dash-toolbar { flex-direction: column; align-items: stretch; }
  .dash-toolbar__actions { display: grid; grid-template-columns: 1fr 1fr; }
  .dash-toolbar__actions .dtb-btn:first-child { grid-column: 1 / -1; }

  /* Stat cards become a swipeable slider on mobile */
  .dash-stats {
    display: flex; flex-direction: row;
    gap: 12px;
    overflow-x: auto;
    scroll-snap-type: x mandatory;
    -webkit-overflow-scrolling: touch;
    padding-bottom: 4px;
    scrollbar-width: none;
  }
  .dash-stats::-webkit-scrollbar { display: none; }
  .dash-stats .stat-card { flex: 0 0 82%; scroll-snap-align: start; }
}
@media (max-width: 420px) {
  .ws-grid { grid-template-columns: 1fr; }
}
</style>
