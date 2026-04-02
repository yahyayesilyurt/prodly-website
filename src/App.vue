<template>
  <div class="app-wrapper">
    <header class="header">
      <div class="brand">
        <svg
          class="logo-svg"
          width="70"
          height="88"
          viewBox="0 0 540 670"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
          aria-label="Prodly logo"
        >
          <!-- lines -->
          <g stroke-linecap="round" stroke-linejoin="round" stroke-width="10">
            <!-- left stem -->
            <line x1="75" y1="67" x2="75" y2="202" stroke="#3F799C" />
            <line x1="75" y1="202" x2="75" y2="428" stroke="#1F4E82" />
            <line x1="75" y1="428" x2="75" y2="608" stroke="#79AFAE" />

            <!-- top -->
            <line x1="75" y1="67" x2="293" y2="67" stroke="#3F799C" />
            <line x1="75" y1="67" x2="197" y2="196" stroke="#79AFAE" />
            <line x1="75" y1="202" x2="197" y2="196" stroke="#79AFAE" />
            <line x1="75" y1="202" x2="197" y2="338" stroke="#1F4E82" />
            <line x1="75" y1="428" x2="197" y2="338" stroke="#1F4E82" />
            <line x1="75" y1="428" x2="197" y2="196" stroke="#79AFAE" />
            <line x1="197" y1="196" x2="197" y2="338" stroke="#79AFAE" />
            <line x1="197" y1="196" x2="293" y2="67" stroke="#79AFAE" />

            <!-- right/top bubble -->
            <line x1="293" y1="67" x2="424" y2="118" stroke="#3F799C" />
            <line x1="424" y1="118" x2="469" y2="254" stroke="#3F799C" />
            <line x1="469" y1="254" x2="439" y2="385" stroke="#79AFAE" />

            <line x1="293" y1="67" x2="321" y2="196" stroke="#79AFAE" />
            <line x1="197" y1="196" x2="321" y2="196" stroke="#79AFAE" />
            <line x1="321" y1="196" x2="469" y2="254" stroke="#1F4E82" />
            <line x1="321" y1="196" x2="439" y2="385" stroke="#1F4E82" />
            <line x1="321" y1="196" x2="403" y2="338" stroke="#1F4E82" />
            <line x1="293" y1="338" x2="469" y2="254" stroke="#79AFAE" />
            <line x1="293" y1="338" x2="439" y2="385" stroke="#79AFAE" />
            <line x1="403" y1="338" x2="469" y2="254" stroke="#79AFAE" />
            <line x1="403" y1="338" x2="439" y2="385" stroke="#79AFAE" />
            <line x1="424" y1="118" x2="403" y2="338" stroke="#1F4E82" />

            <!-- middle -->
            <line x1="197" y1="338" x2="293" y2="338" stroke="#79AFAE" />
            <line x1="197" y1="338" x2="294" y2="429" stroke="#1F4E82" />
            <line x1="293" y1="338" x2="294" y2="429" stroke="#79AFAE" />
            <line x1="294" y1="429" x2="439" y2="385" stroke="#1F4E82" />
            <line x1="75" y1="428" x2="138" y2="459" stroke="#79AFAE" />
            <line x1="138" y1="459" x2="294" y2="429" stroke="#1F4E82" />

            <!-- leg -->
            <line x1="75" y1="428" x2="197" y2="608" stroke="#1F4E82" />
            <line x1="75" y1="608" x2="138" y2="459" stroke="#79AFAE" />
            <line x1="138" y1="459" x2="197" y2="608" stroke="#1F4E82" />
            <line x1="138" y1="459" x2="197" y2="459" stroke="#3F799C" />
            <line x1="197" y1="459" x2="197" y2="608" stroke="#1F4E82" />
            <line x1="75" y1="608" x2="197" y2="608" stroke="#79AFAE" />
          </g>

          <!-- nodes -->
          <circle cx="75" cy="67" r="28" fill="#3F799C" />
          <circle cx="75" cy="202" r="28" fill="#1F4E82" />
          <circle cx="75" cy="428" r="28" fill="#1F4E82" />
          <circle cx="75" cy="608" r="28" fill="#79AFAE" />

          <circle cx="197" cy="196" r="28" fill="#79AFAE" />
          <circle cx="197" cy="338" r="28" fill="#1F4E82" />
          <circle cx="138" cy="459" r="28" fill="#79AFAE" />
          <circle cx="197" cy="608" r="28" fill="#1F4E82" />

          <circle cx="293" cy="67" r="28" fill="#79AFAE" />
          <circle cx="321" cy="196" r="28" fill="#3F799C" />
          <circle cx="293" cy="338" r="28" fill="#79AFAE" />
          <circle cx="294" cy="429" r="28" fill="#1F4E82" />

          <circle cx="424" cy="118" r="28" fill="#3F799C" />
          <circle cx="469" cy="254" r="28" fill="#3F799C" />
          <circle cx="403" cy="338" r="28" fill="#79AFAE" />
          <circle cx="439" cy="385" r="28" fill="#79AFAE" />
        </svg>
        <strong>Prodly</strong>
      </div>
      <span class="tagline"> Your <em>AI</em> Decision Intelligence </span>
    </header>

    <div class="layout">
      <Sidebar :activePage="currentPage" @navigate="goTo" />
      <main class="main">
        <HomePage v-if="currentPage === 'home'" @navigate="goTo" />
        <Dashboard
          v-else-if="currentPage === 'dashboard'"
          @runSimulation="goTo('roadmap')"
        />
        <RoadmapSimulation
          v-else-if="currentPage === 'roadmap'"
          @navigate="goTo"
        />
        <FeedbackAnalysis v-else-if="currentPage === 'feedback'" />
        <ProductMemory v-else-if="currentPage === 'memory'" />
        <Reports v-else-if="currentPage === 'reports'" />
      </main>
    </div>

    <footer v-if="currentPage === 'home'" class="global-footer">
      <div class="footer-brand">
        <strong>✦ Prodly</strong>
        <span>Your AI Decision Intelligence</span>
      </div>
      <p class="footer-copy">&copy; 2026 Prodly Inc. All rights reserved.</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import HomePage from "./views/HomePage.vue";
import Sidebar from "./components/Sidebar.vue";
import Dashboard from "./views/Dashboard.vue";
import RoadmapSimulation from "./views/RoadmapSimulation.vue";
import FeedbackAnalysis from "./views/FeedbackAnalysis.vue";
import ProductMemory from "./views/ProductMemory.vue";
import Reports from "./views/Reports.vue";

const currentPage = ref("home");

function goTo(page) {
  currentPage.value = page;
}

const titles = {
  feedback: "Feedback Analysis",
  reports: "Reports",
  memory: "Product Memory",
};

const pageTitle = computed(() => titles[currentPage.value] || "");
</script>

<style>
.app-wrapper {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.layout {
  flex: 1;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 32px;
  background: #fff;
  border-bottom: 1px solid var(--border);
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 20px;
  font-weight: 700;
  color: #111;
  letter-spacing: -0.3px;
}
.logo {
  color: var(--blue);
  font-size: 22px;
}
.tagline {
  font-size: 13px;
  font-weight: 500;
  letter-spacing: 0.4px;
  color: #6b7280;
  display: flex;
  align-items: center;
  gap: 6px;
}

.tagline em {
  font-style: normal;
  font-weight: 700;
  background: linear-gradient(90deg, #2563eb, #3a9a7a);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.layout {
  display: flex;
  flex: 1;
}

.main {
  flex: 1;
  background: #f8fafc;
}

.placeholder {
  padding: 40px;
}
.placeholder h2 {
  font-size: 24px;
  margin-bottom: 8px;
}
.placeholder p {
  color: var(--gray);
}

.global-footer {
  background: #0f172a;
  padding: 32px 60px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.footer-brand {
  display: flex;
  align-items: center;
  gap: 12px;
  color: #fff;
  font-size: 16px;
}

.footer-brand span {
  font-size: 13px;
  color: #64748b;
}

.footer-copy {
  font-size: 12px;
  color: #475569;
}

/* Tablet */
@media (max-width: 1024px) {
  .global-footer {
    padding: 24px 32px;
  }
}

/* Mobile */
@media (max-width: 768px) {
  .header {
    padding: 14px 16px;
    position: sticky;
    top: 0;
    z-index: 50;
  }

  .tagline {
    display: none;
  }

  .layout {
    flex-direction: column;
  }

  .global-footer {
    flex-direction: column;
    gap: 10px;
    text-align: center;
    padding: 24px 16px;
  }
}

@media (max-width: 1024px) {
  .global-footer {
    display: none;
  }
}


/* ===== Polished overrides ===== */

.app-wrapper {
  background: #f5f7fb;
}

.header {
  position: sticky;
  top: 0;
  z-index: 40;
  padding: 16px 28px;
  background: rgba(255, 255, 255, 0.92);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(226, 232, 240, 0.95);
  box-shadow: 0 6px 18px rgba(15, 23, 42, 0.03);
}

.brand {
  gap: 12px;
  font-size: 22px;
  color: #0f172a;
}

.logo-svg {
  width: 42px;
}

.tagline {
  font-size: 13px;
  padding: 8px 12px;
  border-radius: 999px;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
}

.main {
  background:
    radial-gradient(circle at top left, rgba(37, 99, 235, 0.04), transparent 20%),
    #f8fafc;
}

.global-footer {
  padding: 28px 40px;
  border-top: 1px solid rgba(255,255,255,0.06);
}

.footer-brand strong {
  letter-spacing: -0.3px;
}

@media (max-width: 768px) {
  .header {
    padding: 14px 16px;
  }

  .brand {
    font-size: 18px;
  }

  .logo-svg {
    width: 34px;
  }

  .global-footer {
    padding: 22px 16px;
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
  }
}

</style>
