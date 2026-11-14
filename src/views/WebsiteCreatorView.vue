<!-- src/views/WebsiteCreatorView.vue -->
<template>
  <div class="wc">

    <!-- Header -->
    <div class="wc-head">
      <div class="wc-head__text">
        <h1 class="wc-head__title">Website Creator</h1>
        <p class="wc-head__sub">Launch and manage the sites you offer your own customers.</p>
      </div>
      <div class="wc-head__actions">
        <button class="wc-btn wc-btn--moss" @click="router.push('/tools/buy-domain')">
          <span class="wc-btn__ico">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/>
            </svg>
          </span>
          Buy Domain
        </button>
        <button class="wc-btn wc-btn--tea" @click="router.push('/tools/buy-domain#mailboxes')">
          <span class="wc-btn__ico">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
              <rect x="2" y="4" width="20" height="16" rx="2"/><path d="m22 7-10 5L2 7"/>
            </svg>
          </span>
          Buy Webmail
        </button>
      </div>
    </div>

    <!-- Stats -->
    <div class="wc-stats">
      <div class="wc-stat">
        <span class="wc-stat__num">12</span>
        <span class="wc-stat__lbl">Total sites</span>
      </div>
      <div class="wc-stat wc-stat--live">
        <span class="wc-stat__num">1</span>
        <span class="wc-stat__lbl">Live sites</span>
        <span class="wc-stat__dot" />
      </div>
      <div class="wc-stat wc-stat--pending">
        <span class="wc-stat__num">11</span>
        <span class="wc-stat__lbl">Pending setup</span>
      </div>
    </div>

    <!-- Cards grid -->
    <div class="wc-grid">
      <button v-for="tile in tiles" :key="tile.title" class="wc-card" @click="open(tile)">
        <!-- Badge -->
        <span class="wc-card__badge" :class="`wc-card__badge--${tile.badge.toLowerCase()}`">
          {{ tile.badge }}
        </span>

        <!-- Tile image -->
        <span class="wc-card__thumb">
          <img v-if="tile.img" :src="tile.img" :alt="tile.title" class="wc-card__pic" />
          <svg v-else width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.9"
            stroke-linecap="round" stroke-linejoin="round" v-html="tile.icon" />
        </span>

        <!-- Name row -->
        <span class="wc-card__name">
          {{ tile.title }}
          <span v-if="tile.isNew" class="wc-card__new-chip">NEW</span>
          <svg class="wc-card__lock" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="11" width="18" height="11" rx="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/>
          </svg>
        </span>

        <!-- Upgrade pill -->
        <span class="wc-card__upgrade">UPGRADE</span>
      </button>
    </div>

  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'
import tileWebsiteWidgets from '../assets/tile-website-widgets.png'
import tileSupportPages from '../assets/tile-support-pages.png'
import tileTransactionsTrackers from '../assets/tile-transactions-trackers.png'

const router = useRouter()

const PIC = import.meta.glob('../assets/1000880*.png', { eager: true, import: 'default' })
const pic = (n) => PIC[`../assets/1000880${n}.png`]

const G = {
  broker: '<polyline points="22 7 13.5 15.5 8.5 10.5 2 17"/><polyline points="16 7 22 7 22 13"/>',
  bank: '<line x1="3" y1="21" x2="21" y2="21"/><line x1="3" y1="10" x2="21" y2="10"/><polyline points="5 6 12 3 19 6"/><line x1="4" y1="10" x2="4" y2="21"/><line x1="20" y1="10" x2="20" y2="21"/>',
  gift: '<polyline points="20 12 20 22 4 22 4 12"/><rect x="2" y="7" width="20" height="5"/><line x1="12" y1="22" x2="12" y2="7"/><path d="M12 7H7.5a2.5 2.5 0 0 1 0-5C11 2 12 7 12 7z"/><path d="M12 7h4.5a2.5 2.5 0 0 0 0-5C13 2 12 7 12 7z"/>',
  chart: '<line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/>',
  star: '<polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/>',
  cart: '<circle cx="9" cy="21" r="1"/><circle cx="20" cy="21" r="1"/><path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"/>',
  widgets: '<rect x="3" y="3" width="18" height="18" rx="2"/><line x1="3" y1="9" x2="21" y2="9"/><line x1="9" y1="21" x2="9" y2="9"/>',
  file: '<path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/>',
  pin: '<path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/>',
  activity: '<polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/>',
  wallet: '<path d="M20 7H5a2 2 0 0 1 0-4h13v4"/><path d="M3 5v14a2 2 0 0 0 2 2h15V7"/><path d="M16 14h.01"/>',
}

const tiles = [
  { title: 'Broker', badge: 'POPULAR', route: '/broker', icon: G.broker, img: pic('290') },
  { title: 'Online Banking', badge: 'POPULAR', route: '/online-banking', icon: G.bank, img: pic('291') },
  { title: 'Tesla Giveaway', badge: 'READY', route: '/tesla-giveaway', icon: G.gift, img: pic('294') },
  { title: 'Tesla Investment', badge: 'NEW', route: '/tesla-investment', icon: G.chart, img: pic('293') },
  { title: 'Celebrity', badge: 'NEW', route: '/celebrity', icon: G.star, img: pic('292') },
  { title: 'Ecommerce Site', badge: 'NEW', route: '/ecommerce', icon: G.cart, img: pic('295') },
  { title: 'Website Widgets', badge: 'NEW', isNew: true, route: '/website-widgets', icon: G.widgets, img: tileWebsiteWidgets },
  { title: 'Support Pages', badge: 'READY', route: '/support-pages', icon: G.file, img: tileSupportPages },
  { title: 'Tracking Pages', badge: 'READY', route: '/tracking-pages', icon: G.pin, img: pic('299') },
  { title: 'Transactions Trackers', badge: 'READY', route: '/tools/transactions-tracker', icon: G.activity, img: tileTransactionsTrackers },
  { title: 'Wallets', badge: 'NEW', route: '/wallets', icon: G.wallet, img: pic('300') },
]

function open(tile) {
  if (tile.route) router.push(tile.route)
  else router.push('/billing')
}
</script>

<style scoped>
.wc {
  display: flex;
  flex-direction: column;
  gap: 22px;
}

/* ── Header ── */
.wc-head {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
}
.wc-head__text { display: flex; flex-direction: column; gap: 5px; min-width: 0; }
.wc-head__title {
  margin: 0;
  font-size: 1.55rem;
  font-weight: 800;
  color: var(--moss);
  letter-spacing: -0.03em;
}
.wc-head__sub {
  margin: 0;
  font-size: 0.84rem;
  color: var(--t2);
  font-weight: 500;
}
.wc-head__actions { display: flex; align-items: center; gap: 10px; flex-wrap: wrap; }

/* Buttons */
.wc-btn {
  position: relative; overflow: hidden; isolation: isolate;
  display: inline-flex; align-items: center; gap: 8px;
  height: 42px; padding: 0 20px 0 14px; border-radius: 999px;
  border: none; cursor: pointer;
  font-family: var(--font); font-size: 0.84rem; font-weight: 800; letter-spacing: -0.02em;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}
.wc-btn:hover { transform: translateY(-2px); }
.wc-btn:active { transform: translateY(0); }
.wc-btn--moss {
  background: var(--moss);
  color: var(--tea);
  box-shadow: 0 6px 20px rgba(64, 83, 26, 0.28);
}
.wc-btn--moss:hover { box-shadow: 0 10px 28px rgba(64, 83, 26, 0.36); }
.wc-btn--tea {
  background: var(--tea);
  color: var(--moss-deep);
  box-shadow: 0 6px 20px rgba(199, 213, 159, 0.4);
}
.wc-btn--tea:hover { box-shadow: 0 10px 28px rgba(199, 213, 159, 0.5); }
.wc-btn__ico {
  display: flex; align-items: center; justify-content: center;
  width: 26px; height: 26px; border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  flex-shrink: 0;
}
.wc-btn--tea .wc-btn__ico { background: rgba(64, 83, 26, 0.12); }

/* shine sweep */
.wc-btn::after {
  content: ''; position: absolute; top: 0; bottom: 0; left: -75%;
  width: 45%;
  background: linear-gradient(100deg, transparent 0%, rgba(255,255,255,0.14) 35%, rgba(255,255,255,0.55) 50%, rgba(255,255,255,0.14) 65%, transparent 100%);
  transform: skewX(-22deg); pointer-events: none; z-index: 2;
  animation: wc-shine 2.6s ease-in-out infinite;
}
.wc-head__actions .wc-btn:nth-child(2)::after { animation-delay: 0.35s; }
@keyframes wc-shine { 0% { left: -75%; } 40% { left: 125%; } 100% { left: 125%; } }
@media (prefers-reduced-motion: reduce) { .wc-btn::after { animation: none; display: none; } }

/* ── Stats ── */
.wc-stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }

.wc-stat {
  position: relative;
  background: rgba(255,255,255,0.19);
  backdrop-filter: blur(23px);
  -webkit-backdrop-filter: blur(23px);
  border: 1px solid rgba(255,255,255,0.3);
  box-shadow:
    0 8px 32px rgba(0,0,0,0.07),
    inset 0 1px 0 rgba(255,255,255,0.5),
    inset 0 -1px 0 rgba(255,255,255,0.1),
    inset 0 0 6px 3px rgba(255,255,255,0.3);
  border-radius: 20px;
  padding: 20px 22px;
  display: flex; flex-direction: column; gap: 4px;
  overflow: hidden;
}
.wc-stat--live::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0; height: 3px;
  background: linear-gradient(90deg, var(--moss-soft) 0%, var(--moss) 100%);
  border-radius: 20px 20px 0 0;
}
.wc-stat--pending::before {
  content: '';
  position: absolute; top: 0; left: 0; right: 0; height: 3px;
  background: linear-gradient(90deg, var(--tea-soft) 0%, var(--tea) 100%);
  border-radius: 20px 20px 0 0;
}

.wc-stat__num {
  font-size: 1.9rem;
  font-weight: 800;
  color: var(--moss);
  letter-spacing: -0.04em;
  line-height: 1;
}
.wc-stat--pending .wc-stat__num { color: var(--t2); }

.wc-stat__lbl {
  font-size: 0.7rem;
  font-weight: 700;
  letter-spacing: 0.07em;
  text-transform: uppercase;
  color: var(--t3);
}

.wc-stat__dot {
  position: absolute; top: 20px; right: 20px;
  width: 8px; height: 8px; border-radius: 50%;
  background: var(--moss);
  box-shadow: 0 0 0 3px rgba(64, 83, 26, 0.18);
  animation: wc-pulse 2.2s ease-out infinite;
}
@keyframes wc-pulse {
  0% { box-shadow: 0 0 0 3px rgba(64, 83, 26, 0.18); }
  60% { box-shadow: 0 0 0 9px rgba(64, 83, 26, 0); }
  100% { box-shadow: 0 0 0 3px rgba(64, 83, 26, 0); }
}

/* ── Cards grid ── */
.wc-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

.wc-card {
  position: relative;
  display: flex; flex-direction: column; align-items: flex-start; gap: 12px;
  background: rgba(255,255,255,0.19);
  backdrop-filter: blur(23px);
  -webkit-backdrop-filter: blur(23px);
  border: 1px solid rgba(255,255,255,0.3);
  box-shadow:
    0 8px 32px rgba(0,0,0,0.06),
    inset 0 1px 0 rgba(255,255,255,0.5),
    inset 0 -1px 0 rgba(255,255,255,0.1),
    inset 0 0 6px 3px rgba(255,255,255,0.3);
  border-radius: 22px;
  padding: 18px; cursor: pointer; text-align: left; font-family: var(--font);
  transition: border-color 0.2s ease, transform 0.18s ease, box-shadow 0.18s ease;
}
.wc-card:hover {
  border-color: rgba(199, 213, 159, 0.6);
  transform: translateY(-4px);
  box-shadow:
    0 16px 48px rgba(64, 83, 26, 0.14),
    inset 0 1px 0 rgba(255,255,255,0.5),
    inset 0 0 6px 3px rgba(255,255,255,0.3);
}
.wc-card:active { transform: translateY(-1px); }

/* Badge */
.wc-card__badge {
  align-self: flex-start;
  font-size: 0.6rem; font-weight: 800; letter-spacing: 0.08em; text-transform: uppercase;
  padding: 3px 10px; border-radius: 999px;
}
.wc-card__badge--popular {
  color: var(--moss-deep);
  background: var(--tea);
  border: 1px solid rgba(199, 213, 159, 0.5);
}
.wc-card__badge--ready {
  color: var(--moss);
  background: rgba(199, 213, 159, 0.3);
  border: 1px solid rgba(199, 213, 159, 0.5);
}
.wc-card__badge--new {
  color: var(--on-accent, #fff);
  background: var(--moss);
  border: 1px solid var(--moss-soft);
}

/* Thumbnail */
.wc-card__thumb {
  width: 52px; height: 52px;
  border-radius: 14px;
  overflow: hidden;
  flex-shrink: 0;
  background: var(--moss-soft);
  display: flex; align-items: center; justify-content: center;
  color: var(--moss);
  box-shadow: 0 4px 12px rgba(64, 83, 26, 0.15);
}
.wc-card__pic { width: 100%; height: 100%; object-fit: cover; display: block; }

/* Name row */
.wc-card__name {
  display: flex; align-items: center; gap: 7px; flex-wrap: wrap;
  font-size: 0.9rem; font-weight: 800; color: var(--t1);
  letter-spacing: -0.02em; line-height: 1.3;
}
.wc-card__new-chip {
  font-size: 0.55rem; font-weight: 800; letter-spacing: 0.04em; text-transform: uppercase;
  color: var(--tea); background: var(--moss); padding: 2px 7px; border-radius: 999px;
}
.wc-card__lock { color: var(--t3); opacity: 0.6; flex-shrink: 0; }

/* Upgrade pill */
.wc-card__upgrade {
  margin-top: auto;
  font-size: 0.65rem; font-weight: 800; letter-spacing: 0.08em; text-transform: uppercase;
  color: var(--moss-deep);
  background: rgba(199, 213, 159, 0.35);
  border: 1px solid rgba(199, 213, 159, 0.5);
  padding: 6px 14px; border-radius: 999px;
  transition: background 0.18s ease, color 0.18s ease, border-color 0.18s ease;
}
.wc-card:hover .wc-card__upgrade {
  background: var(--moss);
  color: var(--tea);
  border-color: var(--moss-soft);
}

/* ── Responsive ── */
@media (max-width: 1100px) { .wc-grid { grid-template-columns: repeat(3, 1fr); } }
@media (max-width: 760px) {
  .wc-grid { grid-template-columns: repeat(2, 1fr); }
  .wc-stats { grid-template-columns: 1fr; }
}
@media (max-width: 560px) {
  .wc-head { flex-direction: column; align-items: stretch; }
  .wc-head__actions { display: grid; grid-template-columns: 1fr 1fr; }
}
@media (max-width: 400px) { .wc-grid { grid-template-columns: 1fr; } }
</style>
