<template>
  <div class="dashboard">
    <h1>Dashboard</h1>
    <p class="subtitle">
      Analyze user feedback and simulate roadmap feasibility
    </p>

    <div class="cards-grid">
      <!-- User Feedback Analysis -->
      <div class="card">
        <h3>User Feedback Analysis</h3>
        <div class="big-number">
          <span class="number">392</span>
          <span class="label">Total Feedback</span>
        </div>
        <div class="row-stats">
          <span>📌 124 Feature Requests</span>
          <span>🐛 53 Reports</span>
        </div>
        <button class="btn-secondary" @click="showCharts = !showCharts">
          📊 Charts
        </button>
        <div v-if="showCharts" class="mini-chart">
          <div class="bar-wrap">
            <span>Feature Requests</span>
            <div class="bar">
              <div class="fill blue" style="width: 62%"></div>
            </div>
          </div>
          <div class="bar-wrap">
            <span>Bug Reports</span>
            <div class="bar">
              <div class="fill yellow" style="width: 27%"></div>
            </div>
          </div>
          <div class="bar-wrap">
            <span>Other</span>
            <div class="bar">
              <div class="fill green" style="width: 11%"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Customer Demand Score -->
      <div class="card">
        <h3>Customer Demand Score</h3>
        <div class="demand-score">
          <span class="score-value">0.78</span>
          <div class="progress-bar">
            <div class="fill blue" :style="{ width: demandWidth }"></div>
          </div>
          <span class="demand-label">↑ demand</span>
        </div>
      </div>

      <!-- Delivery Risk Estimate -->
      <div class="card">
        <h3>Delivery Risk Estimate</h3>
        <div class="risk-list">
          <div class="risk-item">
            <span class="dot green"></span>
            <span>Low risk: <strong>2 features</strong></span>
          </div>
          <div class="risk-item">
            <span class="dot yellow"></span>
            <span>Medium risk: <strong>2 features</strong></span>
          </div>
          <div class="risk-item">
            <span class="dot red"></span>
            <span>High risk: <strong>2 features</strong></span>
          </div>
        </div>
      </div>

      <!-- Roadmap Simulation -->
      <div class="card">
        <h3>Roadmap Simulation</h3>
        <button class="btn-primary" @click="$emit('runSimulation')">
          ▶ Run Simulation
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

defineEmits(["runSimulation"]);

const showCharts = ref(false);
const demandRaw = ref(0.78);
const demandWidth = computed(() => demandRaw.value * 100 + "%");
</script>

<style scoped>
.dashboard {
  padding: 32px 40px;
}

h1 {
  font-size: 28px;
  font-weight: 700;
  margin-bottom: 4px;
}

.subtitle {
  color: var(--gray);
  font-size: 14px;
  margin-bottom: 28px;
}

.cards-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.card {
  background: #fff;
  border-radius: 12px;
  padding: 24px;
  border: 1px solid var(--border);
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
}

h3 {
  font-size: 14px;
  font-weight: 600;
  color: #374151;
  margin-bottom: 16px;
}

.big-number {
  display: flex;
  align-items: baseline;
  gap: 8px;
  margin-bottom: 12px;
}

.number {
  font-size: 40px;
  font-weight: 700;
  color: #111;
}
.label {
  font-size: 13px;
  color: var(--gray);
}

.row-stats {
  display: flex;
  gap: 20px;
  font-size: 13px;
  color: var(--gray);
  margin-bottom: 16px;
}

.btn-secondary {
  background: var(--blue);
  color: white;
  border: none;
  padding: 8px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  transition: opacity 0.2s;
}
.btn-secondary:hover {
  opacity: 0.85;
}

.mini-chart {
  margin-top: 14px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.bar-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 12px;
  color: var(--gray);
}
.bar-wrap span {
  width: 110px;
}

.bar {
  flex: 1;
  height: 8px;
  background: #f3f4f6;
  border-radius: 99px;
  overflow: hidden;
}

.fill {
  height: 100%;
  border-radius: 99px;
  transition: width 0.5s ease;
}

.fill.blue {
  background: var(--blue);
}
.fill.yellow {
  background: var(--yellow);
}
.fill.green {
  background: var(--green);
}

/* Demand */
.demand-score {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.score-value {
  font-size: 32px;
  font-weight: 700;
}
.demand-label {
  font-size: 13px;
  color: var(--gray);
}
.progress-bar {
  height: 10px;
  background: #f3f4f6;
  border-radius: 99px;
  overflow: hidden;
}
.progress-bar .fill {
  width: 65%;
}

/* Risk */
.risk-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.risk-item {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 14px;
}
.dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  display: inline-block;
}
.dot.green {
  background: var(--green);
}
.dot.yellow {
  background: var(--yellow);
}
.dot.red {
  background: var(--red);
}

/* Sim */
.sim-bars {
  display: flex;
  flex-direction: column;
  gap: 14px;
  margin-bottom: 20px;
}
.sim-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 13px;
}
.sim-row > span:first-child {
  width: 70px;
}
.sim-row > span:last-child {
  width: 36px;
  text-align: right;
  font-weight: 600;
}
.sim-row .bar {
  flex: 1;
  height: 8px;
  background: #f3f4f6;
  border-radius: 99px;
  overflow: hidden;
}

.btn-primary {
  background: var(--blue);
  color: white;
  border: none;
  padding: 10px 24px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  width: 100%;
  transition: background 0.2s;
}
.btn-primary:hover {
  background: #1d4ed8;
}

@media (max-width: 1024px) {
  .dashboard {
    padding: 24px;
  }
  .cards-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 768px) {
  .dashboard {
    padding: 20px 16px 100px;
  }
  h1 {
    font-size: 22px;
  }
  .number {
    font-size: 32px;
  }
}
</style>
