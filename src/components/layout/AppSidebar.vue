<!-- src/components/layout/AppSidebar.vue -->
<template>

  <!-- Backdrop (mobile only) -->
  <Transition name="backdrop-fade">
    <div v-if="sidebarOpen" class="sidebar-backdrop" @click="closeSidebar"/>
  </Transition>

  <aside class="sidebar" :class="{ 'sidebar--open': sidebarOpen, 'sidebar--desktop-closed': !desktopOpen }">

    <!-- Logo -->
    <div class="sidebar__logo">
      <div class="sidebar__logo-icon">
        <img :src="logoSrc" alt="Globalgle logo" class="logo-img" />
      </div>
      <span class="sidebar__logo-name">Globalgle</span>

      <!-- Toggle / close button -->
      <button class="sidebar__toggle" @click="handleBadgeClick" aria-label="Toggle sidebar">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round">
          <polyline points="11 17 6 12 11 7"/>
          <polyline points="18 17 13 12 18 7"/>
        </svg>
      </button>
    </div>

    <!-- Scrollable nav area -->
    <div class="sidebar__scroll" ref="scrollEl">

      <div
        v-for="group in groups"
        :key="group.label"
        class="sidebar__group"
        :data-tour="group.tour"
      >
        <span class="sidebar__group-label">{{ group.label }}</span>
        <nav class="sidebar__nav">
          <template v-for="item in group.items" :key="item.label">

            <!-- Plain action (coming soon) -->
            <button v-if="item.soon" class="nav-item" @click="openSoon(item.label)">
              <span class="nav-item__icon">
                <svg width="19" height="19" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" v-html="item.icon" />
              </span>
              <span class="nav-item__label">{{ item.label }}</span>
            </button>

            <!-- Route link -->
            <router-link
              v-else
              :to="item.to"
              class="nav-item"
              :class="{ 'nav-item--active': isActive(item) }"
              @click="closeSidebar"
            >
              <span class="nav-item__icon">
                <svg width="19" height="19" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" v-html="item.icon" />
              </span>
              <span class="nav-item__label">{{ item.label }}</span>
              <span v-if="item.badge" class="nav-item__meta">
                <span class="nav-item__badge">{{ item.badge }}</span>
              </span>
            </router-link>

          </template>
        </nav>
      </div>

      <!-- Upgrade CTA -->
      <div class="sidebar__upgrade" data-tour="upgrade">
        <div class="upgrade__header">
          <span class="upgrade__icon">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="currentColor">
              <path d="M2 7l4.8 3.6L12 3.5l5.2 7.1L22 7l-1.8 11.5a1 1 0 0 1-1 .85H4.8a1 1 0 0 1-1-.85L2 7z"/>
            </svg>
          </span>
          <p class="upgrade__title">Upgrade Pro!</p>
        </div>
        <p class="upgrade__desc">Upgrade to Pro and elevate your experience today</p>
        <button class="upgrade__btn" @click.stop="goUpgrade">
          <svg width="11" height="11" viewBox="0 0 24 24" fill="currentColor"><path d="M13 2 3 14h7l-1 8 10-12h-7l1-8z"/></svg>
          Upgrade
        </button>
      </div>

      <!-- Logout button -->
      <button class="sidebar__logout" @click="logout">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/>
          <polyline points="16 17 21 12 16 7"/>
          <line x1="21" y1="12" x2="9" y2="12"/>
        </svg>
        <span class="sidebar__logout-text">Log Out</span>
      </button>

    </div><!-- end sidebar__scroll -->

  </aside>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import logo1 from '../../assets/logo1.jpg'
import { useSidebar } from '../../composables/useSidebar.js'
import { useToast } from '../../composables/useToast.js'

const { sidebarOpen, desktopOpen, closeSidebar, toggleDesktopSidebar } = useSidebar()
const { info } = useToast()
const router = useRouter()
const route = useRoute()
const logoSrc = logo1

/* ── Icons (24×24, stroke) ── */
const I = {
  home: '<path d="M3 9.5L12 3l9 6.5V20a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1V9.5z"/><polyline points="9 21 9 12 15 12 15 21"/>',
  site: '<rect x="2" y="4" width="20" height="16" rx="2"/><line x1="2" y1="9" x2="22" y2="9"/><line x1="5" y1="6.5" x2="5.01" y2="6.5"/><line x1="8" y1="6.5" x2="8.01" y2="6.5"/>',
  tool: '<path d="M14.7 6.3a1 1 0 0 0 0 1.4l1.6 1.6a1 1 0 0 0 1.4 0l3.77-3.77a6 6 0 0 1-7.94 7.94l-6.91 6.91a2.12 2.12 0 0 1-3-3l6.91-6.91a6 6 0 0 1 7.94-7.94l-3.76 3.76z"/>',
  call: '<path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07A19.5 19.5 0 0 1 4.69 12 19.79 19.79 0 0 1 1.63 3.38 2 2 0 0 1 3.6 1.2h3a2 2 0 0 1 2 1.72c.127.96.361 1.903.7 2.81a2 2 0 0 1-.45 2.11L7.91 8.96a16 16 0 0 0 6.13 6.13l1.27-.91a2 2 0 0 1 2.11-.45c.907.339 1.85.573 2.81.7A2 2 0 0 1 22 16.92z"/>',
  mail: '<path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/>',
  spark: '<path d="M12 3l1.9 4.9L19 9.8l-4.9 1.9L12 17l-1.9-5.3L5 9.8l5.1-1.9z"/><path d="M19 15l.6 1.6L21 17l-1.4.4L19 19l-.6-1.6L17 17l1.4-.4z"/>',
  wallet: '<rect x="2" y="5" width="20" height="14" rx="2"/><path d="M16 12h2"/><path d="M2 10h20"/>',
  doc: '<path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="9" y1="13" x2="15" y2="13"/><line x1="9" y1="17" x2="12" y2="17"/>',
  shield: '<path d="M12 2l8 4v6c0 5-3.5 8-8 10-4.5-2-8-5-8-10V6l8-4z"/><polyline points="9 12 11 14 15 10"/>',
  gear: '<circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 2.83-2.83l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/>',
  help: '<circle cx="12" cy="12" r="10"/><path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"/><line x1="12" y1="17" x2="12.01" y2="17"/>',
  flag: '<path d="M4 15s1-1 4-1 5 2 8 2 4-1 4-1V3s-1 1-4 1-5-2-8-2-4 1-4 1z"/><line x1="4" y1="22" x2="4" y2="15"/>',
}

/* ── Navigation (same routes as before) ── */
const groups = [
  {
    label: 'Go Menu',
    items: [
      { label: 'Dashboard', to: '/', icon: I.home, exact: true },
    ],
  },
  {
    label: 'Tools',
    tour: 'sidebar-nav',
    items: [
      { label: 'Website Creator', to: '/website-creator', icon: I.site },
      { label: 'Digital', to: '/digital', icon: I.tool },
      { label: 'Call', to: '/tools/call', icon: I.call, also: ['/tools/call', '/tools/voice-notes'] },
      { label: 'Mailing', to: '/customers', icon: I.mail },
      { label: 'Generator', to: '/tools', icon: I.spark, exact: true },
    ],
  },
  {
    label: 'System',
    items: [
      { label: 'My Wallet', to: '/wallet', icon: I.wallet, badge: '₦0.00' },
      { label: 'Billing & Plans', to: '/billing', icon: I.doc },
    ],
  },
  {
    label: 'Other',
    items: [
      { label: 'Start Reselling', to: '/reselling', icon: I.shield },
      { label: 'Terms', icon: I.doc, soon: true },
      { label: 'Settings', to: '/settings', icon: I.gear },
      { label: 'Support', to: '/help', icon: I.help },
      { label: 'Complaint', to: '/complaint', icon: I.flag },
    ],
  },
]

function isActive(item) {
  const p = route.path
  if (item.also) return item.also.some(a => p === a || p.startsWith(a))
  return p === item.to
}

function openSoon(name) {
  closeSidebar()
  info(`${name} coming soon`, 'This section is not available yet — check back shortly.')
}

function handleBadgeClick() {
  if (window.innerWidth >= 769) toggleDesktopSidebar()
  else closeSidebar()
}

/* Reset sidebar scroll to top on every route change */
const scrollEl = ref(null)
watch(() => route.path, () => {
  if (scrollEl.value) scrollEl.value.scrollTop = 0
})

function goUpgrade() {
  closeSidebar()
  router.push('/billing')
}

function logout() {
  closeSidebar()
  localStorage.removeItem('globalgle_auth')
  router.push('/auth')
}
</script>

<style scoped>
/* ══════════════════════════════════════════════
   SHELL — floating Dark Moss panel
   ══════════════════════════════════════════════ */
.sidebar {
  --sb-text: rgba(244, 248, 232, 0.76);
  --sb-text-strong: #f4f8e8;
  --sb-line: rgba(199, 213, 159, 0.16);

  width: 248px;
  position: fixed;
  top: 14px; left: 14px; bottom: 14px;
  z-index: 100;
  display: flex;
  flex-direction: column;
  border-radius: 26px;
  background:
    radial-gradient(120% 60% at 100% 0%, rgba(199, 213, 159, 0.16) 0%, transparent 55%),
    linear-gradient(180deg, #4a5f1e 0%, #40531a 45%, #34440f 100%);
  box-shadow: 0 18px 50px rgba(40, 52, 16, 0.28), inset 0 1px 0 rgba(255, 255, 255, 0.08);
  overflow: hidden;
  transition: transform 0.32s cubic-bezier(0.4, 0, 0.2, 1), width 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* ── Logo ── */
.sidebar__logo {
  display: flex; align-items: center; gap: 10px;
  padding: 20px 18px 16px;
  flex-shrink: 0;
}
.sidebar__logo-icon {
  width: 38px; height: 38px; flex-shrink: 0;
  border-radius: 12px; overflow: hidden;
  background: #fff;
  box-shadow: 0 0 0 2px rgba(199, 213, 159, 0.5);
}
.logo-img { width: 100%; height: 100%; object-fit: cover; display: block; }
.sidebar__logo-name {
  flex: 1; min-width: 0;
  font-size: 1.12rem; font-weight: 800;
  color: var(--sb-text-strong);
  letter-spacing: -0.03em;
}
.sidebar__toggle {
  width: 28px; height: 28px; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  border-radius: 9px; border: 1px solid var(--sb-line);
  background: rgba(255, 255, 255, 0.06);
  color: var(--sb-text); cursor: pointer;
  transition: background 0.18s ease, color 0.18s ease;
}
.sidebar__toggle:hover { background: rgba(255, 255, 255, 0.14); color: #fff; }

/* ── Scroll area ── */
.sidebar__scroll {
  flex: 1; min-height: 0;
  overflow-y: auto; overflow-x: hidden;
  padding: 0 12px 14px;
  -webkit-overflow-scrolling: touch;
}
.sidebar__scroll::-webkit-scrollbar { width: 3px; }
.sidebar__scroll::-webkit-scrollbar-thumb { background: rgba(199, 213, 159, 0.3); border-radius: 999px; }

/* ── Groups ── */
.sidebar__group { padding: 14px 0 2px; }
.sidebar__group-label {
  display: block;
  font-size: 0.62rem; font-weight: 700;
  letter-spacing: 0.14em; text-transform: uppercase;
  color: rgba(199, 213, 159, 0.62);
  padding: 0 12px; margin-bottom: 8px;
}
.sidebar__nav { display: flex; flex-direction: column; gap: 3px; }

/* ── Items ── */
.nav-item {
  position: relative;
  display: flex; align-items: center; gap: 12px;
  width: 100%;
  padding: 10px 12px;
  border-radius: 14px;
  border: 0;
  background: transparent;
  color: var(--sb-text);
  font-family: inherit; text-align: left; text-decoration: none;
  cursor: pointer;
  transition: background 0.18s ease, color 0.18s ease, transform 0.12s ease;
}
.nav-item:hover { background: rgba(255, 255, 255, 0.09); color: #fff; }
.nav-item:active { transform: scale(0.985); }

.nav-item__icon {
  width: 22px; height: 22px; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
}
.nav-item__label {
  flex: 1; min-width: 0;
  font-size: 0.86rem; font-weight: 600; letter-spacing: -0.02em;
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}

/* Active → Tea Green pill, Moss text */
.nav-item--active,
.nav-item--active:hover {
  background: var(--tea);
  color: var(--moss-deep);
  box-shadow: 0 6px 16px rgba(20, 28, 6, 0.28);
}
.nav-item--active .nav-item__label { font-weight: 800; }

.nav-item__meta { margin-left: auto; display: flex; align-items: center; flex-shrink: 0; }
.nav-item__badge {
  font-size: 0.62rem; font-weight: 700;
  padding: 2px 8px; border-radius: 999px;
  color: var(--tea);
  background: rgba(199, 213, 159, 0.14);
  border: 1px solid rgba(199, 213, 159, 0.3);
}
.nav-item--active .nav-item__badge {
  color: var(--moss-deep);
  background: rgba(64, 83, 26, 0.12);
  border-color: rgba(64, 83, 26, 0.28);
}

/* ── Upgrade card ── */
.sidebar__upgrade {
  position: relative; overflow: hidden;
  margin: 18px 2px 12px;
  padding: 16px 15px 15px;
  border-radius: 20px;
  background:
    radial-gradient(120% 90% at 100% 0%, rgba(199, 213, 159, 0.42) 0%, transparent 60%),
    linear-gradient(160deg, #567028 0%, #2f3e12 100%);
  border: 1px solid rgba(199, 213, 159, 0.28);
}
.upgrade__header { display: flex; align-items: center; gap: 9px; margin-bottom: 7px; }
.upgrade__icon {
  width: 26px; height: 26px; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  border-radius: 9px; background: var(--tea); color: var(--moss-deep);
}
.upgrade__title { margin: 0; font-size: 0.95rem; font-weight: 800; color: #fff; letter-spacing: -0.02em; }
.upgrade__desc {
  margin: 0 0 13px;
  font-size: 0.74rem; line-height: 1.5; font-weight: 500;
  color: rgba(244, 248, 232, 0.72);
}
.upgrade__btn {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 8px 18px; border: 0; border-radius: 999px; cursor: pointer;
  font-family: inherit; font-size: 0.76rem; font-weight: 800;
  background: var(--tea); color: var(--moss-deep);
  transition: transform 0.16s ease, background 0.16s ease;
}
.upgrade__btn:hover { background: #d8e4b6; transform: translateY(-1px); }
.upgrade__btn:active { transform: translateY(0); }

/* ── Logout ── */
/* Glass pill — same frosted recipe as the navbar capsule */
.sidebar__logout {
  position: relative;
  display: flex; align-items: center; justify-content: center; gap: 8px;
  width: 100%; padding: 12px 0;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.19);
  backdrop-filter: blur(23px);
  -webkit-backdrop-filter: blur(23px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.5),
    inset 0 -1px 0 rgba(255, 255, 255, 0.1),
    inset 0 0 6px 3px rgba(255, 255, 255, 0.3);
  color: #fff;
  font-family: inherit; font-size: 0.82rem; font-weight: 800;
  cursor: pointer;
  transition: background 0.18s ease, transform 0.12s ease;
}
/* Light-catch highlights (top + left edge), inset to follow the curve */
.sidebar__logout::before,
.sidebar__logout::after { content: ''; position: absolute; pointer-events: none; }
.sidebar__logout::before {
  top: 0; left: 22px; right: 22px; height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.8), transparent);
}
.sidebar__logout::after {
  top: 12px; bottom: 12px; left: 0; width: 1px;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.8), transparent, rgba(255, 255, 255, 0.3));
}
.sidebar__logout:hover { background: rgba(255, 255, 255, 0.28); }
.sidebar__logout:active { transform: scale(0.98); }

/* ══════════════════════════════════════════════
   DESKTOP COLLAPSE → slim icon rail
   ══════════════════════════════════════════════ */
@media (min-width: 769px) {
  .sidebar--desktop-closed { width: 76px; }
  .sidebar--desktop-closed .sidebar__logo-name,
  .sidebar--desktop-closed .sidebar__toggle,
  .sidebar--desktop-closed .sidebar__group-label,
  .sidebar--desktop-closed .nav-item__label,
  .sidebar--desktop-closed .nav-item__meta,
  .sidebar--desktop-closed .sidebar__upgrade,
  .sidebar--desktop-closed .sidebar__logout-text { display: none; }

  .sidebar--desktop-closed .sidebar__logo { justify-content: center; padding: 18px 0 12px; }
  .sidebar--desktop-closed .sidebar__scroll { padding: 0 10px 14px; }
  .sidebar--desktop-closed .sidebar__nav { align-items: center; }
  .sidebar--desktop-closed .nav-item { justify-content: center; width: 44px; height: 44px; padding: 0; }
  .sidebar--desktop-closed .sidebar__logout { width: 44px; height: 44px; padding: 0; margin: 12px auto 0; }
}

/* ── Tablet ── */
@media (min-width: 769px) and (max-width: 1366px) {
  .sidebar:not(.sidebar--desktop-closed) { width: 232px; }
  .nav-item { padding: 9px 11px; }
  .nav-item__label { font-size: 0.82rem; }
}

/* ── Short viewports ── */
@media (max-height: 820px) and (min-width: 769px) {
  .sidebar__group { padding-top: 10px; }
  .nav-item { padding: 8px 12px; }
}
@media (max-height: 700px) and (min-width: 769px) {
  .sidebar__upgrade { display: none; }
}

/* ══════════════════════════════════════════════
   MOBILE — slide-in drawer
   ══════════════════════════════════════════════ */
.sidebar-backdrop {
  position: fixed; inset: 0; z-index: 99;
  background: rgba(20, 28, 6, 0.55);
}
.backdrop-fade-enter-active, .backdrop-fade-leave-active { transition: opacity 0.28s ease; }
.backdrop-fade-enter-from, .backdrop-fade-leave-to { opacity: 0; }

@media (max-width: 768px) {
  .sidebar {
    top: 0; left: 0; bottom: 0;
    width: 280px;
    border-radius: 0 26px 26px 0;
    transform: translateX(-105%);
  }
  .sidebar--open { transform: translateX(0); }
  .sidebar__toggle { display: none; }
  .nav-item__label { font-size: 0.95rem; }
}

@supports not (height: 100dvh) {
  .sidebar { height: -webkit-fill-available; }
}
</style>
