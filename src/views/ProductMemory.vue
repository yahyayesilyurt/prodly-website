<template>
  <div class="memory-page">

    <!-- Page Header -->
    <div class="page-header">
      <div class="header-left">
        <div class="header-badge">
          <span class="badge-dot pulse"></span>
          Live · Updated 2 min ago
        </div>
        <h1>Product Memory</h1>
        <p class="subtitle">
          Bayesian intelligence layer tracking your team's delivery patterns, velocity trends, and recurring planning errors.
        </p>
      </div>
      <div class="header-right">
        <div class="model-status-card">
          <div class="model-icon">🧬</div>
          <div class="model-info">
            <div class="model-title">Bayesian Model <span class="updated-badge">Updated</span></div>
            <div class="model-meta">Sprint 14 · 847 data points ingested</div>
            <div class="model-confidence">
              <span>Model Confidence</span>
              <div class="conf-bar">
                <div class="conf-fill" style="width: 87%"></div>
              </div>
              <span class="conf-val">87%</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Top KPI Strip -->
    <div class="kpi-strip">
      <div class="kpi-item" v-for="k in kpis" :key="k.label">
        <div class="kpi-icon">{{ k.icon }}</div>
        <div class="kpi-body">
          <div class="kpi-value">{{ k.value }}</div>
          <div class="kpi-label">{{ k.label }}</div>
        </div>
        <div :class="['kpi-delta', k.positive ? 'pos' : 'neg']">
          {{ k.positive ? '▲' : '▼' }} {{ k.delta }}
        </div>
      </div>
    </div>

    <!-- Main Grid -->
    <div class="main-grid">

      <!-- LEFT COL -->
      <div class="col-left">

        <!-- Sprint Velocity Chart -->
        <div class="card velocity-card">
          <div class="card-header-row">
            <h3>Sprint Velocity History</h3>
            <div class="legend-row">
              <span class="leg planned">Planned</span>
              <span class="leg actual">Actual</span>
            </div>
          </div>
          <p class="card-sub">Bayesian model learned from 14 sprints of delivery data</p>

          <div class="velocity-chart">
            <div class="y-axis">
              <span>60</span>
              <span>45</span>
              <span>30</span>
              <span>15</span>
              <span>0</span>
            </div>
            <div class="chart-body">
              <div class="grid-lines">
                <div class="grid-line" v-for="n in 4" :key="n"></div>
              </div>
              <div class="bars-group">
                <div class="sprint-group" v-for="s in sprints" :key="s.name">
                  <div class="bar-pair">
                    <div
                      class="bar planned-bar"
                      :style="{ height: (s.planned / 60 * 100) + '%' }"
                      :title="`Planned: ${s.planned} pts`"
                    ></div>
                    <div
                      class="bar actual-bar"
                      :class="{ over: s.actual > s.planned, under: s.actual < s.planned }"
                      :style="{ height: (s.actual / 60 * 100) + '%' }"
                      :title="`Actual: ${s.actual} pts`"
                    ></div>
                  </div>
                  <span class="sprint-label">{{ s.name }}</span>
                </div>
              </div>
            </div>
          </div>

          <div class="velocity-insight">
            <span class="insight-icon">💡</span>
            <span>Team consistently delivers <strong>12% below</strong> planned capacity in Q1. Model adjusted future estimates accordingly.</span>
          </div>
        </div>

        <!-- Recurring Errors Prevented -->
        <div class="card errors-card">
          <div class="card-header-row">
            <h3>Recurring Planning Errors Prevented</h3>
            <div class="errors-counter">
              <span class="counter-num">{{ errorsAvoided }}</span>
              <span class="counter-label">this quarter</span>
            </div>
          </div>
          <p class="card-sub">Patterns identified from historical sprint data</p>

          <div class="error-list">
            <div class="error-item" v-for="e in errors" :key="e.id">
              <div class="error-left">
                <div :class="['error-sev', e.severity]">{{ e.severity }}</div>
                <div class="error-info">
                  <div class="error-title">{{ e.title }}</div>
                  <div class="error-desc">{{ e.desc }}</div>
                </div>
              </div>
              <div class="error-right">
                <div class="prevented-count">{{ e.count }}x prevented</div>
                <div class="error-saving">{{ e.saving }} saved</div>
              </div>
            </div>
          </div>
        </div>

      </div>

      <!-- RIGHT COL -->
      <div class="col-right">

        <!-- What AI Learned This Sprint -->
        <div class="card learned-card">
          <div class="card-header-row">
            <h3>What the AI Learned — Sprint 14</h3>
            <span class="sprint-tag">Latest</span>
          </div>
          <p class="card-sub">New patterns ingested into the Bayesian model</p>

          <div class="learned-list">
            <div class="learned-item" v-for="l in learned" :key="l.id">
              <div class="learned-icon">{{ l.icon }}</div>
              <div class="learned-body">
                <div class="learned-title">{{ l.title }}</div>
                <div class="learned-desc">{{ l.desc }}</div>
                <div class="learned-confidence">
                  <span>Confidence</span>
                  <div class="mini-bar">
                    <div class="mini-fill" :style="{ width: l.confidence + '%', background: l.color }"></div>
                  </div>
                  <span class="mini-val">{{ l.confidence }}%</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Delivery Pattern Timeline -->
        <div class="card timeline-card">
          <div class="card-header-row">
            <h3>Learned Delivery Patterns</h3>
            <span class="pattern-count">{{ patterns.length }} active patterns</span>
          </div>
          <p class="card-sub">AI-detected regularities shaping future forecasts</p>

          <div class="timeline">
            <div class="timeline-track"></div>
            <div class="timeline-item" v-for="p in patterns" :key="p.id">
              <div :class="['timeline-dot', p.type]"></div>
              <div class="timeline-content">
                <div class="timeline-header">
                  <span class="tl-label">{{ p.label }}</span>
                  <span :class="['tl-type', p.type]">{{ p.typeLabel }}</span>
                </div>
                <div class="tl-desc">{{ p.desc }}</div>
                <div class="tl-impact">
                  <span class="impact-icon">{{ p.impactIcon }}</span>
                  {{ p.impact }}
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Model Accuracy Card -->
        <div class="card accuracy-card">
          <h3>Forecast Accuracy Over Time</h3>
          <p class="card-sub">How well the model predicted actual delivery</p>
          <div class="accuracy-bars">
            <div class="acc-row" v-for="a in accuracy" :key="a.sprint">
              <span class="acc-sprint">{{ a.sprint }}</span>
              <div class="acc-bar">
                <div
                  class="acc-fill"
                  :style="{ width: a.pct + '%', background: a.pct >= 80 ? '#22c55e' : a.pct >= 60 ? '#eab308' : '#ef4444' }"
                ></div>
              </div>
              <span class="acc-pct" :style="{ color: a.pct >= 80 ? '#16a34a' : a.pct >= 60 ? '#ca8a04' : '#dc2626' }">
                {{ a.pct }}%
              </span>
            </div>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const errorsAvoided = ref(23)

const kpis = [
  { icon: '⚡', value: '42 pts', label: 'Avg Sprint Velocity', delta: '+4 pts vs last quarter', positive: true },
  { icon: '🎯', value: '87%', label: 'Forecast Accuracy', delta: '+11% vs Sprint 1', positive: true },
  { icon: '🛡️', value: '23', label: 'Errors Prevented', delta: '↓ repeat mistakes', positive: true },
  { icon: '📉', value: '−18%', label: 'Estimation Bias', delta: 'reduced bias', positive: true },
]

const sprints = [
  { name: 'S7', planned: 50, actual: 41 },
  { name: 'S8', planned: 48, actual: 45 },
  { name: 'S9', planned: 52, actual: 38 },
  { name: 'S10', planned: 45, actual: 47 },
  { name: 'S11', planned: 50, actual: 44 },
  { name: 'S12', planned: 48, actual: 50 },
  { name: 'S13', planned: 52, actual: 46 },
  { name: 'S14', planned: 50, actual: 43 },
]

const errors = [
  {
    id: 1,
    severity: 'high',
    title: 'Q1 Capacity Overestimation',
    desc: 'Team overestimates sprint capacity by ~15% every January due to holiday carry-over.',
    count: 3,
    saving: '~6 sprint days'
  },
  {
    id: 2,
    severity: 'medium',
    title: 'Feature Scope Creep on Auth Stories',
    desc: 'Authentication-related features historically take 2.3x longer than estimated.',
    count: 5,
    saving: '~14 story pts'
  },
  {
    id: 3,
    severity: 'medium',
    title: 'Backend Dependency Underestimation',
    desc: 'Frontend tasks with backend blockers slip by avg. 4 days when not flagged early.',
    count: 8,
    saving: '~3 sprints'
  },
  {
    id: 4,
    severity: 'low',
    title: 'Review Cycle Bottleneck',
    desc: 'PR review time spikes on Fridays, pushing completion to following Monday.',
    count: 7,
    saving: '~2 days/sprint'
  },
]

const learned = [
  {
    id: 1,
    icon: '📊',
    title: 'Velocity Drop Pattern Detected',
    desc: 'Sprint 14 confirmed: team output drops 18% in the final 3 days when release is same week.',
    confidence: 91,
    color: '#2563eb'
  },
  {
    id: 2,
    icon: '🔗',
    title: 'API Integration Complexity Cluster',
    desc: 'Third-party API stories cluster at 2–3x estimate variance. New prior added to model.',
    confidence: 78,
    color: '#eab308'
  },
  {
    id: 3,
    icon: '👥',
    title: 'Team Composition Effect',
    desc: 'Sprints with >2 new engineers show 22% lower throughput. Onboarding cost now modeled.',
    confidence: 84,
    color: '#22c55e'
  },
]

const patterns = [
  {
    id: 1,
    type: 'risk',
    typeLabel: 'Risk Pattern',
    label: 'End-of-quarter slowdown',
    desc: 'Delivery rate drops 20% in last 2 weeks of each quarter across all historical data.',
    impactIcon: '⚠️',
    impact: 'Forecasts auto-adjusted for Q2 end',
  },
  {
    id: 2,
    type: 'insight',
    typeLabel: 'Velocity Insight',
    label: 'Post-release productivity spike',
    desc: 'Team velocity increases 28% in the sprint immediately following a major release.',
    impactIcon: '🚀',
    impact: 'Sprint 15 capacity bumped by 12%',
  },
  {
    id: 3,
    type: 'risk',
    typeLabel: 'Risk Pattern',
    label: 'Feature clustering cascade',
    desc: 'When 3+ large features land same sprint, bug count in next sprint increases 2.1x.',
    impactIcon: '🐛',
    impact: 'Sprint 15 has buffer capacity reserved',
  },
  {
    id: 4,
    type: 'learning',
    typeLabel: 'New Learning',
    label: 'Design review cadence effect',
    desc: 'Features with async design review (vs sync) complete 1.5 days faster on average.',
    impactIcon: '✅',
    impact: 'Workflow recommendation sent to team',
  },
]

const accuracy = [
  { sprint: 'S9', pct: 58 },
  { sprint: 'S10', pct: 67 },
  { sprint: 'S11', pct: 74 },
  { sprint: 'S12', pct: 81 },
  { sprint: 'S13', pct: 85 },
  { sprint: 'S14', pct: 87 },
]
</script>

<style scoped>
.memory-page {
  padding: 32px 40px;
  background: #f8fafc;
  min-height: 100%;
}

/* ── Header ── */
.page-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 24px;
  margin-bottom: 28px;
}

.header-badge {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.6px;
  color: #16a34a;
  background: #f0fdf4;
  border: 1px solid #bbf7d0;
  border-radius: 99px;
  padding: 4px 12px;
  margin-bottom: 10px;
  text-transform: uppercase;
}

.badge-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #22c55e;
  display: inline-block;
}

.badge-dot.pulse {
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(0.8); }
}

h1 {
  font-size: 28px;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 6px;
  letter-spacing: -0.5px;
}

.subtitle {
  font-size: 14px;
  color: #64748b;
  max-width: 480px;
  line-height: 1.6;
}

.model-status-card {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  padding: 20px 24px;
  display: flex;
  gap: 16px;
  align-items: flex-start;
  min-width: 300px;
  box-shadow: 0 1px 6px rgba(0,0,0,0.06);
  flex-shrink: 0;
}

.model-icon {
  font-size: 28px;
  line-height: 1;
  padding-top: 2px;
}

.model-title {
  font-size: 14px;
  font-weight: 700;
  color: #0f172a;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 4px;
}

.updated-badge {
  background: linear-gradient(90deg, #2563eb, #3a9a7a);
  color: #fff;
  font-size: 10px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 99px;
  letter-spacing: 0.3px;
}

.model-meta {
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 12px;
}

.model-confidence {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: #64748b;
}

.conf-bar {
  flex: 1;
  height: 6px;
  background: #f1f5f9;
  border-radius: 99px;
  overflow: hidden;
}

.conf-fill {
  height: 100%;
  background: linear-gradient(90deg, #2563eb, #3a9a7a);
  border-radius: 99px;
  transition: width 1s ease;
}

.conf-val {
  font-weight: 700;
  color: #2563eb;
  min-width: 32px;
}

/* ── KPI Strip ── */
.kpi-strip {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 14px;
  margin-bottom: 24px;
}

.kpi-item {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 14px;
  padding: 18px 20px;
  display: flex;
  align-items: center;
  gap: 12px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.04);
}

.kpi-icon {
  font-size: 22px;
  flex-shrink: 0;
}

.kpi-body { flex: 1 }

.kpi-value {
  font-size: 20px;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.3px;
}

.kpi-label {
  font-size: 11px;
  color: #94a3b8;
  margin-top: 2px;
}

.kpi-delta {
  font-size: 11px;
  font-weight: 600;
  white-space: nowrap;
}

.kpi-delta.pos { color: #16a34a; }
.kpi-delta.neg { color: #dc2626; }

/* ── Main Grid ── */
.main-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.col-left, .col-right {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

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
  align-items: center;
  margin-bottom: 4px;
}

h3 {
  font-size: 15px;
  font-weight: 700;
  color: #0f172a;
}

.card-sub {
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 20px;
}

/* ── Velocity Chart ── */
.legend-row {
  display: flex;
  gap: 14px;
  align-items: center;
  font-size: 12px;
  color: #64748b;
}

.leg::before {
  content: '';
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 3px;
  margin-right: 5px;
}

.leg.planned::before { background: #cbd5e1; }
.leg.actual::before { background: #2563eb; }

.velocity-chart {
  display: flex;
  gap: 8px;
  height: 140px;
  margin-bottom: 16px;
}

.y-axis {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  font-size: 10px;
  color: #cbd5e1;
  text-align: right;
  padding-bottom: 2px;
  width: 22px;
  flex-shrink: 0;
}

.chart-body {
  flex: 1;
  position: relative;
}

.grid-lines {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  pointer-events: none;
}

.grid-line {
  border-top: 1px dashed #f1f5f9;
  width: 100%;
}

.bars-group {
  display: flex;
  align-items: flex-end;
  justify-content: space-around;
  height: 100%;
  position: relative;
}

.sprint-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  flex: 1;
}

.bar-pair {
  display: flex;
  align-items: flex-end;
  gap: 3px;
  height: 120px;
  width: 100%;
  justify-content: center;
}

.bar {
  width: 12px;
  border-radius: 4px 4px 0 0;
  transition: height 0.8s ease;
}

.planned-bar {
  background: #e2e8f0;
}

.actual-bar {
  background: #2563eb;
}

.actual-bar.over {
  background: #22c55e;
}

.actual-bar.under {
  background: #2563eb;
}

.sprint-label {
  font-size: 10px;
  color: #94a3b8;
}

.velocity-insight {
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  border-radius: 10px;
  padding: 12px 14px;
  font-size: 12px;
  color: #1e40af;
  display: flex;
  gap: 8px;
  align-items: flex-start;
  line-height: 1.5;
}

.insight-icon { flex-shrink: 0; }

/* ── Errors Card ── */
.errors-counter {
  text-align: right;
}

.counter-num {
  font-size: 28px;
  font-weight: 800;
  color: #2563eb;
  display: block;
  line-height: 1;
}

.counter-label {
  font-size: 11px;
  color: #94a3b8;
}

.error-list {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.error-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  padding: 14px;
  background: #f8fafc;
  border: 1px solid #f1f5f9;
  border-radius: 12px;
  gap: 12px;
}

.error-left {
  display: flex;
  gap: 12px;
  align-items: flex-start;
  flex: 1;
}

.error-sev {
  font-size: 10px;
  font-weight: 700;
  padding: 3px 8px;
  border-radius: 99px;
  text-transform: uppercase;
  letter-spacing: 0.4px;
  flex-shrink: 0;
}

.error-sev.high {
  background: #fee2e2;
  color: #dc2626;
}

.error-sev.medium {
  background: #fef9c3;
  color: #ca8a04;
}

.error-sev.low {
  background: #f0fdf4;
  color: #16a34a;
}

.error-title {
  font-size: 13px;
  font-weight: 600;
  color: #0f172a;
  margin-bottom: 3px;
}

.error-desc {
  font-size: 12px;
  color: #64748b;
  line-height: 1.5;
}

.error-right {
  text-align: right;
  flex-shrink: 0;
}

.prevented-count {
  font-size: 13px;
  font-weight: 700;
  color: #2563eb;
}

.error-saving {
  font-size: 11px;
  color: #94a3b8;
  margin-top: 2px;
}

/* ── Learned Card ── */
.sprint-tag {
  background: linear-gradient(90deg, #2563eb18, #3a9a7a18);
  color: #2563eb;
  font-size: 11px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 99px;
  border: 1px solid #bfdbfe;
}

.learned-list {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.learned-item {
  display: flex;
  gap: 14px;
  align-items: flex-start;
  padding: 14px;
  background: #f8fafc;
  border: 1px solid #f1f5f9;
  border-radius: 12px;
  transition: border-color 0.2s;
}

.learned-item:hover {
  border-color: #bfdbfe;
}

.learned-icon {
  font-size: 22px;
  flex-shrink: 0;
  line-height: 1;
  padding-top: 2px;
}

.learned-title {
  font-size: 13px;
  font-weight: 600;
  color: #0f172a;
  margin-bottom: 4px;
}

.learned-desc {
  font-size: 12px;
  color: #64748b;
  line-height: 1.5;
  margin-bottom: 10px;
}

.learned-confidence {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 11px;
  color: #94a3b8;
}

.mini-bar {
  flex: 1;
  height: 5px;
  background: #f1f5f9;
  border-radius: 99px;
  overflow: hidden;
}

.mini-fill {
  height: 100%;
  border-radius: 99px;
}

.mini-val {
  font-weight: 700;
  color: #374151;
  min-width: 28px;
}

/* ── Timeline ── */
.pattern-count {
  font-size: 12px;
  color: #94a3b8;
  background: #f1f5f9;
  padding: 3px 10px;
  border-radius: 99px;
}

.timeline {
  position: relative;
  padding-left: 28px;
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.timeline-track {
  position: absolute;
  left: 8px;
  top: 8px;
  bottom: 8px;
  width: 2px;
  background: #e2e8f0;
  border-radius: 2px;
}

.timeline-item {
  position: relative;
  display: flex;
  gap: 12px;
  align-items: flex-start;
}

.timeline-dot {
  position: absolute;
  left: -24px;
  top: 4px;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: 2px solid #fff;
  flex-shrink: 0;
}

.timeline-dot.risk { background: #ef4444; }
.timeline-dot.insight { background: #2563eb; }
.timeline-dot.learning { background: #22c55e; }

.timeline-content {
  flex: 1;
  background: #f8fafc;
  border: 1px solid #f1f5f9;
  border-radius: 10px;
  padding: 12px 14px;
}

.timeline-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 5px;
  gap: 8px;
}

.tl-label {
  font-size: 13px;
  font-weight: 600;
  color: #0f172a;
}

.tl-type {
  font-size: 10px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 99px;
  white-space: nowrap;
}

.tl-type.risk { background: #fee2e2; color: #dc2626; }
.tl-type.insight { background: #eff6ff; color: #2563eb; }
.tl-type.learning { background: #f0fdf4; color: #16a34a; }

.tl-desc {
  font-size: 12px;
  color: #64748b;
  line-height: 1.5;
  margin-bottom: 8px;
}

.tl-impact {
  font-size: 11px;
  color: #374151;
  font-weight: 500;
  display: flex;
  gap: 5px;
  align-items: center;
}

.impact-icon { font-size: 12px; }

/* ── Accuracy Card ── */
.accuracy-bars {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.acc-row {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 12px;
}

.acc-sprint {
  width: 28px;
  color: #94a3b8;
  font-weight: 500;
}

.acc-bar {
  flex: 1;
  height: 8px;
  background: #f1f5f9;
  border-radius: 99px;
  overflow: hidden;
}

.acc-fill {
  height: 100%;
  border-radius: 99px;
  transition: width 0.8s ease;
}

.acc-pct {
  width: 36px;
  text-align: right;
  font-weight: 700;
  font-size: 12px;
}

/* ── Responsive ── */
@media (max-width: 1200px) {
  .main-grid {
    grid-template-columns: 1fr;
  }
  .kpi-strip {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 1024px) {
  .memory-page {
    padding: 24px;
  }
  .page-header {
    flex-direction: column;
  }
  .model-status-card {
    min-width: unset;
    width: 100%;
  }
}

@media (max-width: 768px) {
  .memory-page {
    padding: 20px 16px 100px;
  }
  h1 { font-size: 22px; }
  .kpi-strip {
    grid-template-columns: 1fr 1fr;
  }
  .kpi-item {
    flex-wrap: wrap;
  }
}
</style>
