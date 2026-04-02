<template>
  <div class="sim-page">

    <!-- Header -->
    <div class="page-header">
      <div>
        <h1>AI Roadmap Simulation</h1>
        <p class="subtitle">
          Monte Carlo probabilistic engine · {{ simCount.toLocaleString() }} simulations run
        </p>
      </div>
      <div class="header-actions">
        <button class="btn-back" @click="$emit('navigate', 'dashboard')">← Dashboard</button>
        <button class="btn-run" :class="{ running: simRunning }" @click="runSimulation">
          <span v-if="!simRunning">▶ Run Simulation</span>
          <span v-else class="running-label">
            <span class="spin">⟳</span> Running {{ simProgress }}%
          </span>
        </button>
      </div>
    </div>

    <!-- Summary KPIs -->
    <div class="kpi-row">
      <div class="kpi-card" v-for="k in kpis" :key="k.label">
        <div class="kpi-top">
          <span class="kpi-icon">{{ k.icon }}</span>
          <span :class="['kpi-risk', k.risk]">{{ k.riskLabel }}</span>
        </div>
        <div class="kpi-val">{{ k.value }}</div>
        <div class="kpi-label">{{ k.label }}</div>
      </div>
    </div>

    <!-- Main two-col grid -->
    <div class="main-grid">

      <!-- LEFT: Risk-Based Roadmap (Gantt) -->
      <div class="card gantt-card">
        <div class="card-header-row">
          <div>
            <h3>Risk-Based Roadmap</h3>
            <p class="card-sub">Feature delivery timeline with probabilistic risk overlay</p>
          </div>
          <div class="risk-legend">
            <span class="leg green">Low Risk</span>
            <span class="leg yellow">Moderate</span>
            <span class="leg red">High Risk</span>
          </div>
        </div>

        <!-- Gantt: CSS Grid approach — labels col + chart col perfectly aligned -->
        <div class="gantt-wrap">

          <!-- Month header row -->
          <div class="gantt-head-row">
            <div class="gantt-head-spacer"></div>
            <div class="gantt-head-months">
              <div class="month" v-for="m in months" :key="m">{{ m }}</div>
            </div>
          </div>

          <!-- Feature rows -->
          <div class="gantt-feature-row" v-for="f in features" :key="f.id">
            <!-- Label -->
            <div class="gantt-label">
              <div class="feature-name">{{ f.name }}</div>
              <span :class="['risk-badge', f.risk]">{{ f.riskLabel }}</span>
            </div>
            <!-- Chart area -->
            <div class="gantt-bar-area">
              <!-- grid lines -->
              <div class="grid-col" v-for="(m, mi) in months" :key="m"
                :style="{ left: (mi / months.length * 100) + '%', width: (1 / months.length * 100) + '%' }">
              </div>
              <!-- feature bar -->
              <div
                class="gantt-bar"
                :class="[f.risk, { shimmer: simRunning }]"
                :style="{
                  left: (f.start / totalWeeks * 100) + '%',
                  width: (f.duration / totalWeeks * 100) + '%'
                }"
              >
                <span class="bar-label">{{ f.name }}</span>
                <span class="bar-prob">{{ f.prob }}%</span>
              </div>
              <!-- uncertainty range -->
              <div class="uncertainty-range"
                :style="{
                  left: (f.earlyEnd / totalWeeks * 100) + '%',
                  width: ((f.lateEnd - f.earlyEnd) / totalWeeks * 100) + '%'
                }">
              </div>
            </div>
          </div>

        </div>

        <!-- AI Recommendation box -->
        <div class="ai-rec">
          <div class="ai-rec-icon">🤖</div>
          <div class="ai-rec-body">
            <div class="ai-rec-title">AI Recommendation</div>
            <div class="ai-rec-text">
              <strong>Auth Redesign</strong> and <strong>Dashboard v2</strong> show highest delivery confidence.
              Delay <strong>ML Pipeline</strong> by 2 weeks to reduce cascade risk from API Gateway dependency.
            </div>
          </div>
        </div>
      </div>

      <!-- RIGHT: Monte Carlo Bell Curve -->
      <div class="card monte-card">
        <div>
          <h3>Delivery Probability Distribution</h3>
          <p class="card-sub">Monte Carlo simulation · {{ simCount.toLocaleString() }} iterations</p>
        </div>

        <!-- Feature selector -->
        <div class="feature-tabs">
          <button
            v-for="f in features"
            :key="f.id"
            :class="['feat-tab', { active: selectedFeature === f.id }, f.risk]"
            @click="selectedFeature = f.id"
          >
            {{ f.shortName }}
          </button>
        </div>

        <!-- Bell curve SVG -->
        <div class="bell-wrap">
          <svg viewBox="0 0 340 160" class="bell-svg" preserveAspectRatio="none">
            <!-- grid lines -->
            <line x1="0" y1="140" x2="340" y2="140" stroke="#f1f5f9" stroke-width="1"/>
            <line x1="85" y1="10" x2="85" y2="140" stroke="#f1f5f9" stroke-width="1" stroke-dasharray="4,3"/>
            <line x1="170" y1="10" x2="170" y2="140" stroke="#f1f5f9" stroke-width="1" stroke-dasharray="4,3"/>
            <line x1="255" y1="10" x2="255" y2="140" stroke="#f1f5f9" stroke-width="1" stroke-dasharray="4,3"/>

            <!-- filled bell area -->
            <path :d="bellPath" :fill="bellFill" opacity="0.15"/>
            <!-- bell curve line -->
            <path :d="bellPath" fill="none" :stroke="bellStroke" stroke-width="2.5" stroke-linecap="round"/>

            <!-- Early marker -->
            <line x1="68" y1="20" x2="68" y2="140" stroke="#22c55e" stroke-width="1.5" stroke-dasharray="4,3"/>
            <circle cx="68" cy="20" r="3" fill="#22c55e"/>

            <!-- Most Likely marker -->
            <line :x1="bellPeak" y1="10" :x2="bellPeak" y2="140" :stroke="bellStroke" stroke-width="2"/>
            <circle :cx="bellPeak" cy="10" r="4" :fill="bellStroke"/>

            <!-- Late marker -->
            <line x1="272" y1="30" x2="272" y2="140" stroke="#ef4444" stroke-width="1.5" stroke-dasharray="4,3"/>
            <circle cx="272" cy="30" r="3" fill="#ef4444"/>

            <!-- P50 shaded region -->
            <rect x="130" y="10" width="80" height="130" :fill="bellStroke" opacity="0.07"/>
          </svg>

          <!-- X axis labels -->
          <div class="bell-x-labels">
            <span class="x-label early">Early<br/><strong>{{ activeFeat.early }}</strong></span>
            <span class="x-label likely">Most Likely<br/><strong>{{ activeFeat.likely }}</strong></span>
            <span class="x-label late">Late<br/><strong>{{ activeFeat.late }}</strong></span>
          </div>
        </div>

        <!-- Stats row -->
        <div class="stats-row">
          <div class="stat-item">
            <div class="stat-val green">{{ activeFeat.early }}</div>
            <div class="stat-lbl">P10 · Optimistic</div>
          </div>
          <div class="stat-item center">
            <div class="stat-val" :style="{ color: bellStroke }">{{ activeFeat.likely }}</div>
            <div class="stat-lbl">P50 · Most Likely</div>
          </div>
          <div class="stat-item right">
            <div class="stat-val red">{{ activeFeat.late }}</div>
            <div class="stat-lbl">P90 · Conservative</div>
          </div>
        </div>

        <!-- Risk breakdown -->
        <div class="risk-breakdown">
          <div class="rb-header">
            <span>Success Probability</span>
            <span class="rb-pct" :style="{ color: bellStroke }">{{ activeFeat.prob }}%</span>
          </div>
          <div class="rb-bar">
            <div class="rb-fill" :style="{ width: activeFeat.prob + '%', background: bellStroke }"></div>
          </div>
          <div class="rb-sub">Based on team velocity, scope complexity, and historical delivery data</div>
        </div>

        <!-- Confidence intervals -->
        <div class="ci-list">
          <div class="ci-row" v-for="ci in activeFeat.cis" :key="ci.label">
            <span class="ci-label">{{ ci.label }}</span>
            <div class="ci-bar-wrap">
              <div class="ci-bar">
                <div class="ci-fill" :style="{ left: ci.left + '%', width: ci.width + '%', background: bellStroke + '40' }"></div>
              </div>
            </div>
            <span class="ci-range">{{ ci.range }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Bottom: Feature risk detail table -->
    <div class="card detail-table-card">
      <div class="card-header-row">
        <div>
          <h3>Full Feature Risk Analysis</h3>
          <p class="card-sub">All features ranked by delivery confidence</p>
        </div>
        <div class="table-actions">
          <span class="sim-badge">{{ simCount.toLocaleString() }} simulations</span>
        </div>
      </div>

      <table class="risk-table">
        <thead>
          <tr>
            <th>Feature</th>
            <th>Risk</th>
            <th>Success Prob.</th>
            <th>Most Likely</th>
            <th>Delay Risk</th>
            <th>Dependencies</th>
            <th>AI Signal</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="f in features"
            :key="f.id"
            :class="{ 'row-selected': selectedFeature === f.id }"
            @click="selectedFeature = f.id"
          >
            <td class="feat-cell">
              <span class="feat-dot" :class="f.risk"></span>
              <strong>{{ f.name }}</strong>
            </td>
            <td><span :class="['risk-pill', f.risk]">{{ f.riskLabel }}</span></td>
            <td>
              <div class="prob-cell">
                <div class="prob-mini-bar">
                  <div class="prob-mini-fill" :style="{ width: f.prob + '%', background: f.barColor }"></div>
                </div>
                <span :style="{ color: f.barColor }">{{ f.prob }}%</span>
              </div>
            </td>
            <td class="mono">{{ f.likely }}</td>
            <td>
              <span :class="['delay-tag', f.delayRisk]">{{ f.delayLabel }}</span>
            </td>
            <td class="deps-cell">
              <span class="dep-tag" v-for="d in f.deps" :key="d">{{ d }}</span>
            </td>
            <td class="signal-cell">{{ f.signal }}</td>
          </tr>
        </tbody>
      </table>
    </div>

  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

defineEmits(['navigate'])

const simRunning = ref(false)
const simProgress = ref(0)
const simCount = ref(10000)
const selectedFeature = ref(1)

function runSimulation() {
  if (simRunning.value) return
  simRunning.value = true
  simProgress.value = 0
  const interval = setInterval(() => {
    simProgress.value += Math.floor(Math.random() * 18) + 5
    if (simProgress.value >= 100) {
      simProgress.value = 100
      simCount.value = simCount.value + Math.floor(Math.random() * 2000) + 500
      clearInterval(interval)
      setTimeout(() => { simRunning.value = false }, 400)
    }
  }, 120)
}

const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug']
const totalWeeks = 32

const features = [
  {
    id: 1, shortName: 'Auth',
    name: 'Auth Redesign',
    risk: 'green', riskLabel: 'Low Risk',
    prob: 82, barColor: '#22c55e',
    start: 0, duration: 6, earlyEnd: 5, lateEnd: 8,
    likely: 'Feb 14', early: 'Feb 7', late: 'Mar 1',
    delayRisk: 'low', delayLabel: '< 5 days',
    deps: ['Design', 'Backend'],
    signal: '✅ High confidence',
    cis: [
      { label: 'P25–P75', left: 30, width: 40, range: 'Feb 10 – Feb 20' },
      { label: 'P10–P90', left: 15, width: 70, range: 'Feb 7 – Mar 1' },
    ]
  },
  {
    id: 2, shortName: 'Dashboard',
    name: 'Dashboard v2',
    risk: 'green', riskLabel: 'Low Risk',
    prob: 76, barColor: '#22c55e',
    start: 4, duration: 8, earlyEnd: 11, lateEnd: 14,
    likely: 'Mar 28', early: 'Mar 21', late: 'Apr 11',
    delayRisk: 'low', delayLabel: '< 7 days',
    deps: ['Auth', 'API'],
    signal: '✅ On track',
    cis: [
      { label: 'P25–P75', left: 28, width: 44, range: 'Mar 24 – Apr 4' },
      { label: 'P10–P90', left: 12, width: 75, range: 'Mar 21 – Apr 11' },
    ]
  },
  {
    id: 3, shortName: 'Notifs',
    name: 'Notification Center',
    risk: 'yellow', riskLabel: 'Moderate',
    prob: 61, barColor: '#eab308',
    start: 8, duration: 6, earlyEnd: 13, lateEnd: 17,
    likely: 'Apr 18', early: 'Apr 11', late: 'May 9',
    delayRisk: 'medium', delayLabel: '1–2 weeks',
    deps: ['Dashboard', 'Infra'],
    signal: '⚠️ Monitor scope',
    cis: [
      { label: 'P25–P75', left: 25, width: 50, range: 'Apr 14 – May 2' },
      { label: 'P10–P90', left: 8, width: 82, range: 'Apr 11 – May 9' },
    ]
  },
  {
    id: 4, shortName: 'API GW',
    name: 'API Gateway',
    risk: 'yellow', riskLabel: 'Moderate',
    prob: 55, barColor: '#eab308',
    start: 6, duration: 10, earlyEnd: 14, lateEnd: 19,
    likely: 'May 2', early: 'Apr 18', late: 'May 23',
    delayRisk: 'medium', delayLabel: '2–3 weeks',
    deps: ['Infra', 'Backend'],
    signal: '⚠️ High complexity',
    cis: [
      { label: 'P25–P75', left: 22, width: 56, range: 'Apr 25 – May 16' },
      { label: 'P10–P90', left: 5, width: 88, range: 'Apr 18 – May 23' },
    ]
  },
  {
    id: 5, shortName: 'ML',
    name: 'ML Pipeline',
    risk: 'red', riskLabel: 'High Risk',
    prob: 38, barColor: '#ef4444',
    start: 14, duration: 12, earlyEnd: 22, lateEnd: 30,
    likely: 'Jun 27', early: 'Jun 6', late: 'Aug 1',
    delayRisk: 'high', delayLabel: '4–6 weeks',
    deps: ['API GW', 'Data', 'ML Infra'],
    signal: '🔴 High risk – defer',
    cis: [
      { label: 'P25–P75', left: 18, width: 64, range: 'Jun 20 – Jul 18' },
      { label: 'P10–P90', left: 3, width: 94, range: 'Jun 6 – Aug 1' },
    ]
  },
  {
    id: 6, shortName: 'Export',
    name: 'Export Engine',
    risk: 'red', riskLabel: 'High Risk',
    prob: 44, barColor: '#ef4444',
    start: 18, duration: 9, earlyEnd: 24, lateEnd: 30,
    likely: 'Jul 11', early: 'Jun 27', late: 'Aug 1',
    delayRisk: 'high', delayLabel: '3–5 weeks',
    deps: ['Dashboard', 'ML', 'Billing'],
    signal: '🔴 Blocker detected',
    cis: [
      { label: 'P25–P75', left: 20, width: 60, range: 'Jul 4 – Jul 25' },
      { label: 'P10–P90', left: 5, width: 90, range: 'Jun 27 – Aug 1' },
    ]
  },
]

const kpis = computed(() => [
  { icon: '✅', value: '2', label: 'Low Risk Features', risk: 'green', riskLabel: 'On Track' },
  { icon: '⚠️', value: '2', label: 'Moderate Risk Features', risk: 'yellow', riskLabel: 'Monitor' },
  { icon: '🔴', value: '2', label: 'High Risk Features', risk: 'red', riskLabel: 'Action Needed' },
  { icon: '📅', value: 'Aug 1', label: 'P90 Completion Date', risk: 'neutral', riskLabel: 'Conservative' },
])

const activeFeat = computed(() => features.find(f => f.id === selectedFeature.value) || features[0])

// Bell curve colors
const bellStroke = computed(() => {
  const risk = activeFeat.value.risk
  return risk === 'green' ? '#22c55e' : risk === 'yellow' ? '#eab308' : '#ef4444'
})
const bellFill = computed(() => bellStroke.value)

// Bell peak X position based on prob
const bellPeak = computed(() => {
  // Map prob 38–82 → x 130–210
  const p = activeFeat.value.prob
  return 100 + ((p - 38) / (82 - 38)) * 140
})

// SVG bell curve path
const bellPath = computed(() => {
  const peak = bellPeak.value
  const spread = activeFeat.value.risk === 'red' ? 100 : activeFeat.value.risk === 'yellow' ? 80 : 60
  const pts = []
  for (let x = 0; x <= 340; x += 4) {
    const z = (x - peak) / spread
    const y = 140 - 128 * Math.exp(-0.5 * z * z)
    pts.push(`${x},${y}`)
  }
  const line = 'M ' + pts.join(' L ')
  // Close path for fill
  return line + ` L 340,140 L 0,140 Z`
})
</script>

<style scoped>
.sim-page {
  padding: 32px 40px;
  background: #f8fafc;
  min-height: 100%;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

/* ── Header ── */
.page-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
}

h1 {
  font-size: 28px;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.5px;
  margin-bottom: 4px;
}

.subtitle {
  font-size: 13px;
  color: #94a3b8;
}

.header-actions {
  display: flex;
  gap: 10px;
  align-items: center;
  flex-shrink: 0;
}

.btn-back {
  background: #fff;
  border: 1px solid #e2e8f0;
  color: #64748b;
  padding: 9px 18px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.2s;
}
.btn-back:hover { background: #f8fafc; color: #111; }

.btn-run {
  background: #2563eb;
  color: #fff;
  border: none;
  padding: 10px 22px;
  border-radius: 10px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.2s;
  min-width: 160px;
}
.btn-run:hover { background: #1d4ed8; }
.btn-run.running { background: #7c3aed; cursor: not-allowed; }

.running-label {
  display: flex;
  align-items: center;
  gap: 8px;
  justify-content: center;
}

.spin {
  display: inline-block;
  animation: spin 0.8s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* ── KPI Row ── */
.kpi-row {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 14px;
}

.kpi-card {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 14px;
  padding: 18px 20px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.04);
}

.kpi-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.kpi-icon { font-size: 20px; }

.kpi-risk {
  font-size: 10px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 99px;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}
.kpi-risk.green { background: #f0fdf4; color: #16a34a; }
.kpi-risk.yellow { background: #fefce8; color: #ca8a04; }
.kpi-risk.red { background: #fef2f2; color: #dc2626; }
.kpi-risk.neutral { background: #f1f5f9; color: #64748b; }

.kpi-val {
  font-size: 28px;
  font-weight: 800;
  color: #0f172a;
  letter-spacing: -1px;
  line-height: 1;
  margin-bottom: 4px;
}
.kpi-label { font-size: 12px; color: #94a3b8; }

/* ── Cards ── */
.card {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.04);
}

.card-header-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 6px;
  gap: 12px;
}

h3 {
  font-size: 15px;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 2px;
}

.card-sub {
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 20px;
}

/* ── Main Grid ── */
.main-grid {
  display: grid;
  grid-template-columns: 1.4fr 1fr;
  gap: 20px;
}

/* ── Risk Legend ── */
.risk-legend {
  display: flex;
  gap: 12px;
  align-items: center;
  font-size: 12px;
  flex-shrink: 0;
}

.leg {
  display: flex;
  align-items: center;
  gap: 5px;
  color: #64748b;
}
.leg::before {
  content: '';
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 3px;
}
.leg.green::before { background: #22c55e; }
.leg.yellow::before { background: #eab308; }
.leg.red::before { background: #ef4444; }

/* ── Gantt ── */
/* ── Gantt: flat-row layout ── */
.gantt-wrap {
  display: flex;
  flex-direction: column;
  margin-bottom: 16px;
  border: 1px solid #f1f5f9;
  border-radius: 10px;
  overflow: hidden;
}

/* Header row: spacer + month labels */
.gantt-head-row {
  display: flex;
  background: #f8fafc;
  border-bottom: 1px solid #f1f5f9;
}

.gantt-head-spacer {
  width: 155px;
  flex-shrink: 0;
  border-right: 1px solid #f1f5f9;
}

.gantt-head-months {
  flex: 1;
  display: flex;
}

.month {
  flex: 1;
  height: 30px;
  font-size: 11px;
  color: #94a3b8;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  border-right: 1px solid #f1f5f9;
}
.month:last-child { border-right: none; }

/* Each feature row: label + bar area side by side */
.gantt-feature-row {
  display: flex;
  border-bottom: 1px solid #f8fafc;
  height: 48px;
}
.gantt-feature-row:last-child { border-bottom: none; }

.gantt-label {
  width: 155px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  gap: 3px;
  padding: 0 10px 0 12px;
  border-right: 1px solid #f1f5f9;
  background: #fff;
}

.feature-name {
  font-size: 12px;
  font-weight: 600;
  color: #0f172a;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 130px;
}

.risk-badge {
  font-size: 9px;
  font-weight: 700;
  padding: 1px 6px;
  border-radius: 99px;
  display: inline-block;
  white-space: nowrap;
}
.risk-badge.green { background: #f0fdf4; color: #16a34a; }
.risk-badge.yellow { background: #fefce8; color: #ca8a04; }
.risk-badge.red { background: #fef2f2; color: #dc2626; }

/* Chart area: same width as gantt-head-months */
.gantt-bar-area {
  flex: 1;
  position: relative;
  overflow: hidden;
  background: #fff;
}

.grid-col {
  position: absolute;
  top: 0;
  bottom: 0;
  border-right: 1px solid #f8fafc;
  pointer-events: none;
}

.gantt-bar {
  position: absolute;
  top: 10px;
  height: 28px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 8px;
  font-size: 11px;
  font-weight: 600;
  color: #fff;
  z-index: 2;
  white-space: nowrap;
  overflow: hidden;
  min-width: 28px;
  cursor: pointer;
  transition: opacity 0.2s, filter 0.2s;
}
.gantt-bar:hover { filter: brightness(1.08); }
.gantt-bar.green { background: #22c55e; }
.gantt-bar.yellow { background: #eab308; }
.gantt-bar.red { background: #ef4444; }
.gantt-bar.shimmer { animation: shimmer 1s ease infinite alternate; }

@keyframes shimmer {
  from { opacity: 1; }
  to { opacity: 0.55; }
}

.bar-label { font-size: 10px; opacity: 0.92; overflow: hidden; }
.bar-prob { font-size: 10px; font-weight: 800; flex-shrink: 0; margin-left: 4px; }

.uncertainty-range {
  position: absolute;
  bottom: 6px;
  height: 3px;
  background: #cbd5e1;
  border-radius: 99px;
  z-index: 1;
  opacity: 0.5;
}

/* ── AI Rec ── */
.ai-rec {
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  border-radius: 12px;
  padding: 14px 16px;
  display: flex;
  gap: 12px;
  align-items: flex-start;
}
.ai-rec-icon { font-size: 20px; flex-shrink: 0; }
.ai-rec-title { font-size: 12px; font-weight: 700; color: #1e40af; margin-bottom: 4px; }
.ai-rec-text { font-size: 12px; color: #1e3a8a; line-height: 1.5; }

/* ── Monte Carlo ── */
.feature-tabs {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  margin-bottom: 16px;
}

.feat-tab {
  padding: 5px 12px;
  border-radius: 99px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  border: 1.5px solid transparent;
  transition: all 0.15s;
  background: #f8fafc;
  color: #64748b;
}
.feat-tab.active.green { background: #f0fdf4; color: #16a34a; border-color: #22c55e; }
.feat-tab.active.yellow { background: #fefce8; color: #ca8a04; border-color: #eab308; }
.feat-tab.active.red { background: #fef2f2; color: #dc2626; border-color: #ef4444; }
.feat-tab:not(.active):hover { background: #f1f5f9; }

.bell-wrap {
  background: #f8fafc;
  border-radius: 12px;
  padding: 16px;
  margin-bottom: 16px;
  border: 1px solid #f1f5f9;
}

.bell-svg {
  width: 100%;
  height: 160px;
  display: block;
}

.bell-x-labels {
  display: flex;
  justify-content: space-between;
  padding: 0 4px;
  margin-top: 8px;
}

.x-label {
  font-size: 11px;
  color: #64748b;
  line-height: 1.4;
}
.x-label strong { display: block; font-size: 12px; color: #0f172a; }
.x-label.likely { text-align: center; }
.x-label.late { text-align: right; }

/* Stats row */
.stats-row {
  display: flex;
  margin-bottom: 16px;
  border: 1px solid #f1f5f9;
  border-radius: 12px;
  overflow: hidden;
}

.stat-item {
  flex: 1;
  padding: 12px 14px;
  border-right: 1px solid #f1f5f9;
}
.stat-item:last-child { border-right: none; }
.stat-item.center { text-align: center; }
.stat-item.right { text-align: right; }

.stat-val { font-size: 15px; font-weight: 800; margin-bottom: 2px; }
.stat-val.green { color: #16a34a; }
.stat-val.red { color: #dc2626; }
.stat-lbl { font-size: 10px; color: #94a3b8; }

/* Risk breakdown */
.risk-breakdown {
  margin-bottom: 16px;
}
.rb-header {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #64748b;
  margin-bottom: 6px;
}
.rb-pct { font-weight: 800; font-size: 14px; }
.rb-bar {
  height: 8px;
  background: #f1f5f9;
  border-radius: 99px;
  overflow: hidden;
  margin-bottom: 6px;
}
.rb-fill {
  height: 100%;
  border-radius: 99px;
  transition: width 0.6s ease;
}
.rb-sub { font-size: 11px; color: #94a3b8; }

/* CI List */
.ci-list { display: flex; flex-direction: column; gap: 8px; }
.ci-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 11px;
}
.ci-label { width: 56px; color: #94a3b8; font-weight: 600; flex-shrink: 0; }
.ci-bar-wrap { flex: 1; }
.ci-bar {
  height: 6px;
  background: #f1f5f9;
  border-radius: 99px;
  position: relative;
  overflow: visible;
}
.ci-fill {
  position: absolute;
  height: 100%;
  border-radius: 99px;
}
.ci-range { color: #64748b; white-space: nowrap; min-width: 120px; text-align: right; }

/* ── Detail Table ── */
.detail-table-card { overflow-x: auto; }

.table-actions {
  display: flex;
  gap: 10px;
  align-items: center;
}

.sim-badge {
  background: linear-gradient(90deg, #eff6ff, #f0fdf4);
  border: 1px solid #bfdbfe;
  color: #2563eb;
  font-size: 11px;
  font-weight: 700;
  padding: 4px 12px;
  border-radius: 99px;
}

.risk-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
}

.risk-table th {
  text-align: left;
  font-size: 11px;
  font-weight: 700;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.4px;
  padding: 10px 12px;
  border-bottom: 1px solid #f1f5f9;
  white-space: nowrap;
}

.risk-table td {
  padding: 12px 12px;
  border-bottom: 1px solid #f8fafc;
  vertical-align: middle;
}

.risk-table tr { cursor: pointer; transition: background 0.15s; }
.risk-table tr:hover td { background: #f8fafc; }
.risk-table tr.row-selected td { background: #eff6ff; }

.feat-cell { display: flex; align-items: center; gap: 8px; }
.feat-dot {
  width: 8px; height: 8px; border-radius: 50%; flex-shrink: 0;
}
.feat-dot.green { background: #22c55e; }
.feat-dot.yellow { background: #eab308; }
.feat-dot.red { background: #ef4444; }

.risk-pill {
  font-size: 11px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 99px;
  white-space: nowrap;
}
.risk-pill.green { background: #f0fdf4; color: #16a34a; }
.risk-pill.yellow { background: #fefce8; color: #ca8a04; }
.risk-pill.red { background: #fef2f2; color: #dc2626; }

.prob-cell { display: flex; align-items: center; gap: 8px; }
.prob-mini-bar {
  width: 60px; height: 5px;
  background: #f1f5f9; border-radius: 99px; overflow: hidden;
}
.prob-mini-fill { height: 100%; border-radius: 99px; }

.mono { font-family: 'Courier New', monospace; font-size: 12px; color: #374151; }

.delay-tag {
  font-size: 11px;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 99px;
}
.delay-tag.low { background: #f0fdf4; color: #16a34a; }
.delay-tag.medium { background: #fefce8; color: #ca8a04; }
.delay-tag.high { background: #fef2f2; color: #dc2626; }

.deps-cell { display: flex; gap: 4px; flex-wrap: wrap; }
.dep-tag {
  font-size: 10px;
  background: #f1f5f9;
  color: #64748b;
  padding: 2px 7px;
  border-radius: 99px;
}

.signal-cell { font-size: 12px; color: #374151; white-space: nowrap; }

/* ── Responsive ── */
@media (max-width: 1200px) {
  .main-grid { grid-template-columns: 1fr; }
  .kpi-row { grid-template-columns: repeat(2, 1fr); }
}

@media (max-width: 1024px) {
  .sim-page { padding: 24px; }
}

@media (max-width: 768px) {
  .sim-page { padding: 20px 16px 100px; }
  h1 { font-size: 22px; }
  .page-header { flex-direction: column; }
  .kpi-row { grid-template-columns: 1fr 1fr; }
  .gantt-head-spacer, .gantt-label { width: 90px; min-width: 90px; }
  .feature-name { max-width: 76px; font-size: 11px; }
  .risk-table th:nth-child(n+5),
  .risk-table td:nth-child(n+5) { display: none; }
}

</style>