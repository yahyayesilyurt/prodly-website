<template>
  <div class="app-wrapper">
    <header class="header">
      <div class="brand">
        <svg
          width="48"
          height="48"
          viewBox="0 0 60 65"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <g stroke-width="1.8" stroke-linecap="round">
            <g stroke="#1A363E">
              <line x1="15" y1="10" x2="15" y2="25" />
              <line x1="15" y1="25" x2="15" y2="40" />
              <line x1="15" y1="40" x2="15" y2="55" />

              <line x1="15" y1="10" x2="30" y2="10" />
              <line x1="15" y1="40" x2="30" y2="40" />
            </g>

            <g stroke="#40A09B">
              <line x1="30" y1="10" x2="42" y2="16" />
              <line x1="42" y1="34" x2="30" y2="40" />
            </g>

            <g stroke="#48C0E4">
              <line x1="42" y1="16" x2="45" y2="25" />
              <line x1="45" y1="25" x2="42" y2="34" />
            </g>

            <g stroke="#40A09B" opacity="0.6">
              <line x1="15" y1="10" x2="25" y2="20" />
              <line x1="15" y1="25" x2="25" y2="20" />
              <line x1="30" y1="10" x2="25" y2="20" />
              <line x1="42" y1="16" x2="25" y2="20" />

              <line x1="15" y1="25" x2="30" y2="28" />
              <line x1="15" y1="40" x2="30" y2="28" />
              <line x1="30" y1="40" x2="30" y2="28" />
              <line x1="42" y1="34" x2="30" y2="28" />
            </g>

            <g stroke="#48C0E4" opacity="0.6">
              <line x1="25" y1="20" x2="30" y2="28" />
              <line x1="45" y1="25" x2="30" y2="28" />
            </g>
          </g>

          <circle cx="25" cy="20" r="3.5" fill="#40A09B" />
          <circle cx="30" cy="28" r="4" fill="#1A363E" />

          <circle cx="30" cy="10" r="4" fill="#40A09B" />
          <circle cx="42" cy="16" r="4.5" fill="#48C0E4" />
          <circle cx="45" cy="25" r="4" fill="#48C0E4" />
          <circle cx="42" cy="34" r="4.5" fill="#40A09B" />
          <circle cx="30" cy="40" r="4" fill="#1A363E" />

          <circle cx="15" cy="10" r="4.5" fill="#1A363E" />
          <circle cx="15" cy="25" r="4.5" fill="#1A363E" />
          <circle cx="15" cy="40" r="4.5" fill="#1A363E" />
          <circle cx="15" cy="55" r="4.5" fill="#40A09B" />
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
import Reports from "./views/Reports.vue";

const currentPage = ref("home");

function goTo(page) {
  currentPage.value = page;
}

const titles = {
  feedback: "Feedback Analysis",
  reports: "Reports",
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
</style>
