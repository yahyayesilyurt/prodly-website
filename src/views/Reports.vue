<template>
  <div class="reports-page">
    <h1>Reports</h1>
    <p class="subtitle">Generated insights and exportable summaries</p>

    <!-- Top Actions -->
    <div class="top-bar">
      <div class="date-filters">
        <button
          v-for="r in ranges"
          :key="r.value"
          :class="['range-btn', { active: activeRange === r.value }]"
          @click="activeRange = r.value"
        >
          {{ r.label }}
        </button>
      </div>
      <button class="export-btn" @click="showExportModal = true">
        ⬇ Export Report
      </button>
    </div>

    <!-- KPI Row -->
    <div class="kpi-row">
      <div class="kpi-card" v-for="k in kpis" :key="k.label">
        <div class="kpi-value" :style="{ color: k.color }">{{ k.value }}</div>
        <div class="kpi-label">{{ k.label }}</div>
        <div :class="['kpi-trend', k.trend > 0 ? 'up' : 'down']">
          {{ k.trend > 0 ? "▲" : "▼" }} {{ Math.abs(k.trend) }}% vs last period
        </div>
      </div>
    </div>

    <!-- Charts Section -->
    <div class="charts-grid">
      <!-- Feedback Over Time -->
      <div class="chart-card wide">
        <h3>Feedback Volume Over Time</h3>
        <div class="line-chart">
          <div v-for="(bar, i) in volumeData" :key="i" class="bar-col">
            <div class="bar-stack">
              <div class="bar-seg general" :style="{ flex: bar.general }"></div>
              <div class="bar-seg bug" :style="{ flex: bar.bug }"></div>
              <div class="bar-seg feature" :style="{ flex: bar.feature }"></div>
            </div>
            <span class="bar-label">{{ bar.month }}</span>
          </div>
        </div>
        <div class="legend">
          <span class="leg-item feature">🔵 Feature Requests</span>
          <span class="leg-item bug">🔴 Bug Reports</span>
          <span class="leg-item general">🟡 General</span>
        </div>
      </div>

      <!-- Top Feature Requests -->
      <div class="chart-card">
        <h3>Top Feature Requests</h3>
        <div class="top-list">
          <div v-for="(item, i) in topFeatures" :key="i" class="top-row">
            <span class="rank">#{{ i + 1 }}</span>
            <div class="top-info">
              <div class="top-title">{{ item.title }}</div>
              <div class="bar small">
                <div class="fill blue" :style="{ width: item.pct + '%' }"></div>
              </div>
            </div>
            <span class="top-votes">{{ item.votes }} votes</span>
          </div>
        </div>
      </div>

      <!-- Risk Distribution -->
      <div class="chart-card">
        <h3>Delivery Risk Distribution</h3>
        <div class="donut-wrap">
          <svg viewBox="0 0 120 120" class="donut">
            <circle
              cx="60"
              cy="60"
              r="48"
              fill="none"
              stroke="#f3f4f6"
              stroke-width="20"
            />
            <circle
              cx="60"
              cy="60"
              r="48"
              fill="none"
              stroke="#22c55e"
              stroke-width="20"
              stroke-dasharray="100 201"
              stroke-dashoffset="-0"
              transform="rotate(-90 60 60)"
            />
            <circle
              cx="60"
              cy="60"
              r="48"
              fill="none"
              stroke="#eab308"
              stroke-width="20"
              stroke-dasharray="67 201"
              stroke-dashoffset="-100"
              transform="rotate(-90 60 60)"
            />
            <circle
              cx="60"
              cy="60"
              r="48"
              fill="none"
              stroke="#ef4444"
              stroke-width="20"
              stroke-dasharray="34 201"
              stroke-dashoffset="-167"
              transform="rotate(-90 60 60)"
            />
          </svg>
          <div class="donut-center">
            <span class="donut-total">9</span>
            <span class="donut-sub">features</span>
          </div>
        </div>
        <div class="donut-legend">
          <div class="dl-row">
            <span class="dot green"></span> Low risk <strong>2</strong>
          </div>
          <div class="dl-row">
            <span class="dot yellow"></span> Medium risk <strong>2</strong>
          </div>
          <div class="dl-row">
            <span class="dot red"></span> High risk <strong>2</strong>
          </div>
        </div>
      </div>
    </div>

    <!-- Recent Reports Table -->
    <div class="table-card">
      <div class="table-header">
        <h3>Generated Reports</h3>
        <input
          v-model="tableSearch"
          class="table-search"
          placeholder="🔍 Search..."
        />
      </div>
      <table>
        <thead>
          <tr>
            <th @click="sortBy('name')">Report Name {{ sortIcon("name") }}</th>
            <th @click="sortBy('type')">Type {{ sortIcon("type") }}</th>
            <th @click="sortBy('date')">Generated {{ sortIcon("date") }}</th>
            <th @click="sortBy('status')">Status {{ sortIcon("status") }}</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="r in sortedReports" :key="r.id">
            <td class="report-name">📄 {{ r.name }}</td>
            <td>
              <span :class="['type-badge', r.typeKey]">{{ r.type }}</span>
            </td>
            <td class="gray">{{ r.date }}</td>
            <td>
              <span :class="['status-badge', r.status]">
                {{ r.status === "ready" ? "✅ Ready" : "⏳ Processing" }}
              </span>
            </td>
            <td class="actions-cell">
              <button class="tbl-btn" @click="download(r)">⬇ Download</button>
              <button class="tbl-btn red" @click="deleteReport(r.id)">
                🗑
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Export Modal -->
    <div
      v-if="showExportModal"
      class="modal-overlay"
      @click.self="showExportModal = false"
    >
      <div class="modal">
        <h2>Export Report</h2>
        <p>Choose a format to export the current report.</p>
        <div class="format-options">
          <label v-for="fmt in formats" :key="fmt.value" class="format-option">
            <input type="radio" v-model="selectedFormat" :value="fmt.value" />
            <span class="fmt-icon">{{ fmt.icon }}</span>
            <span>{{ fmt.label }}</span>
          </label>
        </div>
        <div class="modal-actions">
          <button class="btn-cancel" @click="showExportModal = false">
            Cancel
          </button>
          <button class="btn-export" @click="handleExport">
            Export {{ selectedFormat.toUpperCase() }}
          </button>
        </div>
      </div>
    </div>

    <!-- Toast -->
    <div v-if="toast" class="toast">{{ toast }}</div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const activeRange = ref("30d");
const showExportModal = ref(false);
const selectedFormat = ref("pdf");
const tableSearch = ref("");
const toast = ref("");
const sortKey = ref("date");
const sortDir = ref(-1);

const ranges = [
  { label: "Last 7 days", value: "7d" },
  { label: "Last 30 days", value: "30d" },
  { label: "Last 90 days", value: "90d" },
];

const formats = [
  { label: "PDF Document", value: "pdf", icon: "📕" },
  { label: "CSV Spreadsheet", value: "csv", icon: "📗" },
  { label: "JSON Data", value: "json", icon: "📘" },
];

const kpis = [
  { label: "Total Feedback", value: "392", trend: 12, color: "#2563eb" },
  { label: "Avg Demand Score", value: "0.78", trend: 5, color: "#22c55e" },
  { label: "High Risk Features", value: "2", trend: -2, color: "#ef4444" },
  { label: "Roadmap Success %", value: "82%", trend: 8, color: "#eab308" },
];

const volumeData = [
  { month: "Oct", feature: 28, bug: 12, general: 20 },
  { month: "Nov", feature: 35, bug: 18, general: 25 },
  { month: "Dec", feature: 22, bug: 9, general: 18 },
  { month: "Jan", feature: 42, bug: 21, general: 30 },
  { month: "Feb", feature: 38, bug: 15, general: 22 },
  { month: "Mar", feature: 50, bug: 24, general: 35 },
];

const topFeatures = [
  { title: "Dark mode support", votes: 47, pct: 90 },
  { title: "Export to PDF / CSV", votes: 38, pct: 73 },
  { title: "Slack integration", votes: 31, pct: 60 },
  { title: "Custom dashboards", votes: 25, pct: 48 },
  { title: "Mobile app", votes: 19, pct: 37 },
];

const reports = ref([
  {
    id: 1,
    name: "Q1 2026 Feedback Summary",
    type: "Feedback",
    typeKey: "feedback",
    date: "2026-03-28",
    status: "ready",
  },
  {
    id: 2,
    name: "Roadmap Feasibility Report",
    type: "Simulation",
    typeKey: "simulation",
    date: "2026-03-25",
    status: "ready",
  },
  {
    id: 3,
    name: "Risk Analysis — March",
    type: "Risk",
    typeKey: "risk",
    date: "2026-03-20",
    status: "ready",
  },
  {
    id: 4,
    name: "Customer Demand Deep Dive",
    type: "Demand",
    typeKey: "demand",
    date: "2026-03-15",
    status: "processing",
  },
]);

function sortBy(key) {
  if (sortKey.value === key) sortDir.value *= -1;
  else {
    sortKey.value = key;
    sortDir.value = 1;
  }
}

function sortIcon(key) {
  if (sortKey.value !== key) return "⇅";
  return sortDir.value === 1 ? "▲" : "▼";
}

const sortedReports = computed(() => {
  return [...reports.value]
    .filter((r) =>
      r.name.toLowerCase().includes(tableSearch.value.toLowerCase()),
    )
    .sort((a, b) => {
      const av = a[sortKey.value],
        bv = b[sortKey.value];
      return av < bv ? -sortDir.value : av > bv ? sortDir.value : 0;
    });
});

function deleteReport(id) {
  reports.value = reports.value.filter((r) => r.id !== id);
  showToast("Report deleted.");
}

function download(r) {
  showToast(`Downloading "${r.name}"...`);
}

function handleExport() {
  showExportModal.value = false;
  showToast(`Report exported as ${selectedFormat.value.toUpperCase()} ✓`);
}

function showToast(msg) {
  toast.value = msg;
  setTimeout(() => {
    toast.value = "";
  }, 2800);
}
</script>

<style scoped>
.reports-page {
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
  margin-bottom: 24px;
}

/* Top Bar */
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.date-filters {
  display: flex;
  gap: 8px;
}

.range-btn {
  padding: 7px 16px;
  border-radius: 99px;
  border: 1px solid var(--border);
  background: #fff;
  font-size: 13px;
  color: var(--gray);
  cursor: pointer;
  transition: all 0.2s;
}
.range-btn.active {
  background: var(--blue);
  color: #fff;
  border-color: var(--blue);
}

.export-btn {
  padding: 9px 20px;
  background: var(--blue);
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s;
}
.export-btn:hover {
  background: #1d4ed8;
}

/* KPI */
.kpi-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
  margin-bottom: 24px;
}

.kpi-card {
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px 22px;
}

.kpi-value {
  font-size: 28px;
  font-weight: 700;
  margin-bottom: 4px;
}
.kpi-label {
  font-size: 12px;
  color: var(--gray);
  margin-bottom: 8px;
}
.kpi-trend {
  font-size: 11px;
  font-weight: 600;
}
.kpi-trend.up {
  color: var(--green);
}
.kpi-trend.down {
  color: var(--red);
}

/* Charts Grid */
.charts-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 24px;
}

.chart-card {
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 24px;
}

.chart-card.wide {
  grid-column: 1 / -1;
}

h3 {
  font-size: 14px;
  font-weight: 600;
  margin-bottom: 20px;
  color: #374151;
}

/* Bar Chart */
.line-chart {
  display: flex;
  align-items: flex-end;
  gap: 10px;
  height: 140px;
  margin-bottom: 16px;
}

.bar-col {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  flex: 1;
  height: 100%;
}

.bar-stack {
  display: flex;
  flex-direction: column-reverse;
  flex: 1;
  width: 70%;
  gap: 2px;
  border-radius: 6px;
  overflow: hidden;
}

.bar-seg {
  width: 100%;
  min-height: 4px;
}
.bar-seg.feature {
  background: var(--blue);
}
.bar-seg.bug {
  background: var(--red);
}
.bar-seg.general {
  background: var(--yellow);
}

.bar-label {
  font-size: 11px;
  color: var(--gray);
}

.legend {
  display: flex;
  gap: 20px;
  margin-top: 12px;
}
.leg-item {
  font-size: 12px;
  color: var(--gray);
}

/* Top List */
.top-list {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.top-row {
  display: flex;
  align-items: center;
  gap: 12px;
}

.rank {
  font-size: 13px;
  font-weight: 700;
  color: var(--gray);
  width: 24px;
}

.top-info {
  flex: 1;
}

.top-title {
  font-size: 13px;
  font-weight: 500;
  margin-bottom: 5px;
}

.bar.small {
  height: 6px;
  background: #f3f4f6;
  border-radius: 99px;
  overflow: hidden;
}

.fill {
  height: 100%;
  border-radius: 99px;
  background: var(--blue);
  transition: width 0.5s;
}

.top-votes {
  font-size: 12px;
  color: var(--gray);
  white-space: nowrap;
}

/* Donut */
.donut-wrap {
  position: relative;
  width: 130px;
  margin: 0 auto 20px;
}

.donut {
  width: 130px;
  height: 130px;
}

.donut-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.donut-total {
  font-size: 22px;
  font-weight: 700;
  display: block;
}
.donut-sub {
  font-size: 11px;
  color: var(--gray);
}

.donut-legend {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.dl-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  color: var(--gray);
}

.dl-row strong {
  margin-left: auto;
  color: #111;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  display: inline-block;
  flex-shrink: 0;
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

/* Table */
.table-card {
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 24px;
  overflow: hidden;
}

.table-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.table-search {
  padding: 7px 14px;
  border: 1px solid var(--border);
  border-radius: 99px;
  font-size: 13px;
  outline: none;
  width: 200px;
}
.table-search:focus {
  border-color: var(--blue);
}

table {
  width: 100%;
  border-collapse: collapse;
}

th {
  font-size: 12px;
  font-weight: 600;
  color: var(--gray);
  text-align: left;
  padding: 10px 14px;
  border-bottom: 1px solid var(--border);
  cursor: pointer;
  user-select: none;
  white-space: nowrap;
}
th:hover {
  color: var(--blue);
}

td {
  font-size: 13px;
  padding: 12px 14px;
  border-bottom: 1px solid #f3f4f6;
  vertical-align: middle;
}

.report-name {
  font-weight: 500;
}
.gray {
  color: var(--gray);
}

.type-badge {
  padding: 3px 10px;
  border-radius: 99px;
  font-size: 11px;
  font-weight: 600;
}
.type-badge.feedback {
  background: var(--blue-light);
  color: var(--blue);
}
.type-badge.simulation {
  background: var(--green-light);
  color: #166534;
}
.type-badge.risk {
  background: #fee2e2;
  color: var(--red);
}
.type-badge.demand {
  background: var(--yellow-light);
  color: #92400e;
}

.status-badge {
  font-size: 12px;
  font-weight: 500;
}

.actions-cell {
  display: flex;
  gap: 8px;
  align-items: center;
}

.tbl-btn {
  padding: 5px 12px;
  border-radius: 6px;
  border: 1px solid var(--border);
  background: #f9fafb;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s;
}
.tbl-btn:hover {
  background: var(--blue-light);
  color: var(--blue);
  border-color: var(--blue);
}
.tbl-btn.red:hover {
  background: #fee2e2;
  color: var(--red);
  border-color: var(--red);
}

/* Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.35);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.modal {
  background: #fff;
  border-radius: 16px;
  padding: 32px;
  width: 360px;
  box-shadow: 0 8px 40px rgba(0, 0, 0, 0.15);
}

.modal h2 {
  font-size: 20px;
  font-weight: 700;
  margin-bottom: 8px;
}
.modal p {
  color: var(--gray);
  font-size: 14px;
  margin-bottom: 20px;
}

.format-options {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 24px;
}

.format-option {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  border: 1px solid var(--border);
  border-radius: 10px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}
.format-option:has(input:checked) {
  border-color: var(--blue);
  background: var(--blue-light);
  color: var(--blue);
}
.format-option input {
  accent-color: var(--blue);
}
.fmt-icon {
  font-size: 18px;
}

.modal-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.btn-cancel {
  padding: 9px 18px;
  border: 1px solid var(--border);
  background: #fff;
  border-radius: 8px;
  font-size: 13px;
  cursor: pointer;
}
.btn-export {
  padding: 9px 18px;
  background: var(--blue);
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
}
.btn-export:hover {
  background: #1d4ed8;
}

/* Toast */
.toast {
  position: fixed;
  bottom: 28px;
  left: 50%;
  transform: translateX(-50%);
  background: #1a1a2e;
  color: #fff;
  padding: 12px 24px;
  border-radius: 99px;
  font-size: 13px;
  font-weight: 500;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  animation: fadeIn 0.3s ease;
  z-index: 200;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

@media (max-width: 1024px) {
  .reports-page {
    padding: 24px;
  }
  .kpi-row {
    grid-template-columns: repeat(2, 1fr);
  }
  .charts-grid {
    grid-template-columns: 1fr;
  }
  .chart-card.wide {
    grid-column: auto;
  }
}

@media (max-width: 768px) {
  .reports-page {
    padding: 20px 16px 100px;
  }

  h1 {
    font-size: 22px;
  }

  .top-bar {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .date-filters {
    flex-wrap: wrap;
  }

  .export-btn {
    width: 100%;
    text-align: center;
  }

  .kpi-row {
    grid-template-columns: 1fr 1fr;
    gap: 10px;
  }
  .kpi-value {
    font-size: 22px;
  }

  table {
    font-size: 12px;
  }
  th,
  td {
    padding: 10px 8px;
  }

  .table-header {
    flex-direction: column;
    gap: 10px;
  }
  .table-search {
    width: 100%;
  }

  th:nth-child(2),
  td:nth-child(2),
  th:nth-child(4),
  td:nth-child(4) {
    display: none;
  }

  .modal {
    width: calc(100vw - 40px);
    padding: 24px 20px;
  }
}
</style>
