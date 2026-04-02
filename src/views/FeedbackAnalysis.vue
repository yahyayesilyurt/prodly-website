<template>
  <div class="feedback-page">

    <div class="page-header">
      <div>
        <h1>Feedback Analysis</h1>
        <p class="subtitle">AI-powered NLP engine · Review and filter user submitted feedback</p>
      </div>
    </div>

    <!-- ═══════════════════════════════════════════
         AI NLP SHOWCASE PANEL
    ═══════════════════════════════════════════ -->
    <div class="nlp-showcase">
      <div class="showcase-header">
        <div class="showcase-title">
          <span class="ai-chip">✦ AI NLP Engine</span>
          <h2>From Raw Feedback → Structured Feature Brief</h2>
          <p class="showcase-sub">Prodly's NLP pipeline transforms unstructured customer noise into actionable product intelligence.</p>
        </div>
        <button
          class="btn-analyze"
          :class="{ processing: nlpRunning, done: nlpDone }"
          @click="runNLP"
        >
          <span v-if="!nlpRunning && !nlpDone">⚡ Analyze with AI</span>
          <span v-else-if="nlpRunning" class="processing-label">
            <span class="spin">⟳</span> Processing {{ nlpStep }}
          </span>
          <span v-else>✓ Analysis Complete · Run Again</span>
        </button>
      </div>

      <div class="nlp-columns">

        <!-- LEFT: Raw unstructured feedback -->
        <div class="nlp-col raw-col">
          <div class="col-label">
            <span class="col-dot chaos"></span>
            Unstructured Customer Feedback
          </div>
          <div class="raw-feed">
            <div
              class="raw-item"
              v-for="(r, i) in rawFeedback"
              :key="i"
              :class="{ highlighted: nlpRunning && nlpIndex === i, faded: nlpDone && !r.selected }"
            >
              <div class="raw-meta">
                <span class="raw-source">{{ r.source }}</span>
                <span class="raw-user">{{ r.user }}</span>
                <span class="raw-date">{{ r.date }}</span>
              </div>
              <p class="raw-text">{{ r.text }}</p>
              <div v-if="nlpRunning && nlpIndex === i" class="scan-line"></div>
            </div>
          </div>
        </div>

        <!-- CENTER: Arrow + pipeline steps -->
        <div class="nlp-arrow-col">
          <div class="pipeline-steps">
            <div
              class="pipe-step"
              v-for="(step, i) in pipelineSteps"
              :key="i"
              :class="{
                active: nlpRunning && currentPipeStep === i,
                done: nlpDone || (nlpRunning && currentPipeStep > i)
              }"
            >
              <div class="pipe-icon">{{ step.icon }}</div>
              <div class="pipe-label">{{ step.label }}</div>
              <div class="pipe-connector" v-if="i < pipelineSteps.length - 1"></div>
            </div>
          </div>
          <div class="big-arrow" :class="{ animated: nlpRunning || nlpDone }">→</div>
        </div>

        <!-- RIGHT: Structured Feature Brief -->
        <div class="nlp-col brief-col" :class="{ visible: nlpDone }">
          <div class="col-label">
            <span class="col-dot clean"></span>
            Structured Feature Brief
            <span class="generated-badge" v-if="nlpDone">AI Generated</span>
          </div>

          <div class="brief-card" :class="{ appear: nlpDone }">
            <div class="brief-header">
              <div class="brief-type">🚀 Feature Request</div>
              <div class="brief-priority high">High Priority</div>
            </div>

            <h4 class="brief-title">Unified Dark Mode & Theme System</h4>

            <div class="brief-section">
              <div class="brief-label">📊 Customer Demand Score</div>
              <div class="demand-row">
                <div class="demand-bar">
                  <div class="demand-fill" :style="{ width: nlpDone ? '78%' : '0%' }"></div>
                </div>
                <span class="demand-val">0.78</span>
              </div>
            </div>

            <div class="brief-section">
              <div class="brief-label">🎯 Extracted User Intent</div>
              <div class="intent-tags">
                <span class="intent-tag" v-for="t in intentTags" :key="t">{{ t }}</span>
              </div>
            </div>

            <div class="brief-section">
              <div class="brief-label">📝 AI Summary</div>
              <p class="brief-summary">
                {{ typedSummary }}<span class="cursor" v-if="typing">|</span>
              </p>
            </div>

            <div class="brief-section">
              <div class="brief-label">⚠️ Delivery Risk</div>
              <div class="risk-row">
                <span class="risk-pill yellow">Moderate · 61% confidence</span>
                <span class="risk-hint">2 sprint estimate</span>
              </div>
            </div>

            <div class="brief-actions">
              <button class="brief-btn primary">📋 Add to Roadmap</button>
              <button class="brief-btn secondary">💾 Save Brief</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ═══════════════════════════════════════════
         EXISTING: Summary Cards
    ═══════════════════════════════════════════ -->
    <div class="summary-row">
      <div class="stat-card" v-for="s in stats" :key="s.label">
        <span class="stat-icon">{{ s.icon }}</span>
        <div>
          <div class="stat-value">{{ s.value }}</div>
          <div class="stat-label">{{ s.label }}</div>
        </div>
      </div>
    </div>

    <!-- EXISTING: Filters -->
    <div class="filters">
      <button
        v-for="f in filters"
        :key="f.value"
        :class="['filter-btn', { active: activeFilter === f.value }]"
        @click="activeFilter = f.value"
      >
        {{ f.label }}
      </button>
      <input
        v-model="searchQuery"
        class="search-input"
        placeholder="🔍  Search feedback..."
      />
    </div>

    <!-- EXISTING: Feedback List -->
    <div class="feedback-list">
      <div
        v-for="item in filteredFeedback"
        :key="item.id"
        class="feedback-card"
        :class="{ expanded: expandedId === item.id }"
        @click="toggle(item.id)"
      >
        <div class="card-header">
          <span :class="['tag', item.type]">{{ item.typeLabel }}</span>
          <span class="user">👤 {{ item.user }}</span>
          <span class="date">{{ item.date }}</span>
          <span class="chevron">{{ expandedId === item.id ? '▲' : '▼' }}</span>
        </div>
        <div class="card-title">{{ item.title }}</div>
        <div v-if="expandedId === item.id" class="card-body">
          <p>{{ item.detail }}</p>
          <div class="card-actions">
            <button
              :class="['action-btn', { liked: item.liked }]"
              @click.stop="item.liked = !item.liked; item.votes += item.liked ? 1 : -1"
            >
              👍 {{ item.votes }}
            </button>
            <button class="action-btn" @click.stop>💬 Reply</button>
          </div>
        </div>
      </div>
      <div v-if="filteredFeedback.length === 0" class="empty">
        No feedback found for this filter.
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// ── NLP Showcase state ──────────────────────────────
const nlpRunning = ref(false)
const nlpDone = ref(false)
const nlpIndex = ref(-1)
const nlpStep = ref('')
const currentPipeStep = ref(-1)
const typing = ref(false)
const typedSummary = ref('')

const fullSummary = '7 customers across 4 channels requested a system-wide dark mode with theme persistence. Sentiment analysis shows high frustration with eye strain. Feature maps to existing Design System epic — estimated low engineering effort with high retention impact.'

const intentTags = ['Dark mode', 'Theme persistence', 'Eye strain', 'Accessibility', 'Retention']

const pipelineSteps = [
  { icon: '🔍', label: 'NLP Parse' },
  { icon: '🧠', label: 'Intent Extract' },
  { icon: '🔗', label: 'Cluster & Dedupe' },
  { icon: '📊', label: 'Score Demand' },
  { icon: '✅', label: 'Generate Brief' },
]

function runNLP() {
  nlpDone.value = false
  nlpRunning.value = true
  nlpIndex.value = -1
  currentPipeStep.value = 0
  typedSummary.value = ''
  typing.value = false

  // Scan raw items one by one
  let itemIdx = 0
  const scanInterval = setInterval(() => {
    nlpIndex.value = itemIdx
    itemIdx++
    if (itemIdx >= rawFeedback.length) clearInterval(scanInterval)
  }, 420)

  // Advance pipeline steps
  let step = 0
  const pipeInterval = setInterval(() => {
    currentPipeStep.value = step
    step++
    if (step >= pipelineSteps.length) {
      clearInterval(pipeInterval)
      setTimeout(() => {
        nlpRunning.value = false
        nlpDone.value = true
        typing.value = true
        typeText()
      }, 300)
    }
  }, 480)
}

function typeText() {
  let i = 0
  const interval = setInterval(() => {
    typedSummary.value = fullSummary.slice(0, i)
    i++
    if (i > fullSummary.length) {
      clearInterval(interval)
      typing.value = false
    }
  }, 18)
}

const rawFeedback = [
  {
    source: 'Intercom',
    user: 'alice@corp.com',
    date: 'Mar 28',
    text: 'Please add dark mode!!! My eyes are killing me after 8 hours of staring at this white screen.',
    selected: true
  },
  {
    source: 'G2 Review',
    user: 'Anonymous',
    date: 'Mar 26',
    text: 'Great product overall but really needs a dark mode. 4/5 stars.',
    selected: true
  },
  {
    source: 'Email',
    user: 'bob@startup.io',
    date: 'Mar 25',
    text: 'Hey team — we love Prodly but our devs all use dark mode in their editors. Would be amazing to have this in the dashboard too.',
    selected: true
  },
  {
    source: 'Intercom',
    user: 'carol@agency.co',
    date: 'Mar 24',
    text: 'The export button is broken on Safari but also dark mode would be great',
    selected: false
  },
  {
    source: 'Twitter',
    user: '@productguy',
    date: 'Mar 23',
    text: '@ProdlyHQ love the roadmap features but please please please dark mode 🙏 my eyes at night',
    selected: true
  },
  {
    source: 'Intercom',
    user: 'dave@enterprise.com',
    date: 'Mar 22',
    text: 'We work late hours and a dark theme would really help the team. Happy to beta test it.',
    selected: true
  },
  {
    source: 'G2 Review',
    user: 'PM at SaaS Co',
    date: 'Mar 20',
    text: 'Missing: dark mode, PDF export, and Slack integration. Otherwise solid.',
    selected: false
  },
]

// ── Existing logic ──────────────────────────────────
const activeFilter = ref('all')
const searchQuery = ref('')
const expandedId = ref(null)

function toggle(id) {
  expandedId.value = expandedId.value === id ? null : id
}

const stats = [
  { icon: '📥', value: 392, label: 'Total Feedback' },
  { icon: '🚀', value: 124, label: 'Feature Requests' },
  { icon: '🐛', value: 53, label: 'Bug Reports' },
  { icon: '💡', value: 215, label: 'General Ideas' },
]

const filters = [
  { label: 'All', value: 'all' },
  { label: '🚀 Feature Request', value: 'feature' },
  { label: '🐛 Bug Report', value: 'bug' },
  { label: '💡 General', value: 'general' },
]

const feedback = ref([
  {
    id: 1, type: 'feature', typeLabel: '🚀 Feature Request',
    user: 'alice@corp.com', date: '2026-03-28',
    title: 'Add dark mode support across the dashboard',
    detail: 'Many users work at night and would benefit from a dark mode. Should apply to all views including charts and the sidebar.',
    priority: 'high', votes: 47, liked: false,
  },
  {
    id: 2, type: 'bug', typeLabel: '🐛 Bug Report',
    user: 'bob@startup.io', date: '2026-03-27',
    title: 'Roadmap simulation freezes on large datasets',
    detail: 'When uploading more than 500 feedback entries, the Run Simulation button becomes unresponsive. Reproducible on Chrome 120+.',
    priority: 'high', votes: 31, liked: false,
  },
  {
    id: 3, type: 'general', typeLabel: '💡 General',
    user: 'carol@agency.co', date: '2026-03-26',
    title: 'Onboarding flow is confusing for new users',
    detail: 'The first-time experience lacks guidance. A short interactive tour or tooltip hints would help new product managers get started faster.',
    priority: 'medium', votes: 22, liked: false,
  },
  {
    id: 4, type: 'feature', typeLabel: '🚀 Feature Request',
    user: 'dave@enterprise.com', date: '2026-03-25',
    title: 'Export reports as PDF or CSV',
    detail: 'It would be very helpful to download reports in PDF or CSV format to share with stakeholders who don\'t have access to the platform.',
    priority: 'medium', votes: 58, liked: false,
  },
  {
    id: 5, type: 'bug', typeLabel: '🐛 Bug Report',
    user: 'eve@product.dev', date: '2026-03-24',
    title: 'Customer Demand Score not updating after filter change',
    detail: 'When changing the date filter, the demand score widget stays at its initial value and does not reflect the filtered feedback.',
    priority: 'low', votes: 14, liked: false,
  },
  {
    id: 6, type: 'general', typeLabel: '💡 General',
    user: 'frank@scaleup.io', date: '2026-03-23',
    title: 'Slack integration for feedback notifications',
    detail: 'It would be great to receive a Slack notification whenever high-priority feedback comes in, with a direct link to the item.',
    priority: 'low', votes: 19, liked: false,
  },
])

const filteredFeedback = computed(() =>
  feedback.value
    .filter(f => activeFilter.value === 'all' || f.type === activeFilter.value)
    .filter(f =>
      f.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      f.user.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
)
</script>

<style scoped>
.feedback-page {
  padding: 32px 40px;
  display: flex;
  flex-direction: column;
  gap: 24px;
  background: #f8fafc;
  min-height: 100%;
}

.page-header { margin-bottom: 0; }

h1 {
  font-size: 28px;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.5px;
  margin-bottom: 4px;
}
.subtitle { font-size: 13px; color: #94a3b8; }

/* ══════════════════════════════════
   NLP SHOWCASE
══════════════════════════════════ */
.nlp-showcase {
  background: #fff;
  border: 1px solid #e2e8f0;
  border-radius: 20px;
  padding: 28px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.05);
}

.showcase-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  margin-bottom: 24px;
}

.ai-chip {
  display: inline-block;
  background: linear-gradient(90deg, #2563eb18, #3a9a7a18);
  border: 1px solid #bfdbfe;
  color: #2563eb;
  font-size: 11px;
  font-weight: 700;
  padding: 3px 12px;
  border-radius: 99px;
  margin-bottom: 8px;
  letter-spacing: 0.3px;
}

.showcase-title h2 {
  font-size: 18px;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 6px;
}

.showcase-sub {
  font-size: 13px;
  color: #64748b;
  max-width: 480px;
  line-height: 1.5;
}

.btn-analyze {
  background: linear-gradient(135deg, #2563eb, #1d4ed8);
  color: #fff;
  border: none;
  padding: 11px 24px;
  border-radius: 10px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  white-space: nowrap;
  transition: all 0.2s;
  flex-shrink: 0;
  min-width: 180px;
}
.btn-analyze:hover { background: linear-gradient(135deg, #1d4ed8, #1e40af); }
.btn-analyze.processing { background: linear-gradient(135deg, #7c3aed, #6d28d9); cursor: not-allowed; }
.btn-analyze.done { background: linear-gradient(135deg, #16a34a, #15803d); }

.processing-label {
  display: flex;
  align-items: center;
  gap: 8px;
  justify-content: center;
}

.spin {
  display: inline-block;
  animation: spin 0.7s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* ── Three columns ── */
.nlp-columns {
  display: grid;
  grid-template-columns: 1fr 80px 1fr;
  gap: 16px;
  align-items: start;
}

.col-label {
  display: flex;
  align-items: center;
  gap: 7px;
  font-size: 11px;
  font-weight: 700;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 12px;
}

.col-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}
.col-dot.chaos { background: #f59e0b; }
.col-dot.clean { background: #22c55e; }

.generated-badge {
  background: linear-gradient(90deg, #2563eb, #3a9a7a);
  color: #fff;
  font-size: 9px;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 99px;
  margin-left: 4px;
}

/* Raw feed */
.raw-feed {
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-height: 420px;
  overflow-y: auto;
}

.raw-item {
  background: #f8fafc;
  border: 1px solid #f1f5f9;
  border-radius: 10px;
  padding: 12px 14px;
  position: relative;
  overflow: hidden;
  transition: all 0.3s;
}

.raw-item.highlighted {
  border-color: #2563eb;
  background: #eff6ff;
  box-shadow: 0 0 0 2px #bfdbfe40;
}

.raw-item.faded {
  opacity: 0.45;
}

.raw-meta {
  display: flex;
  gap: 8px;
  align-items: center;
  margin-bottom: 6px;
}

.raw-source {
  font-size: 10px;
  font-weight: 700;
  background: #e2e8f0;
  color: #64748b;
  padding: 2px 7px;
  border-radius: 99px;
}
.raw-user {
  font-size: 11px;
  color: #94a3b8;
}
.raw-date {
  font-size: 11px;
  color: #cbd5e1;
  margin-left: auto;
}
.raw-text {
  font-size: 12px;
  color: #374151;
  line-height: 1.5;
  margin: 0;
}

/* Scan line animation */
.scan-line {
  position: absolute;
  top: 0;
  left: -100%;
  width: 60%;
  height: 100%;
  background: linear-gradient(90deg, transparent, #2563eb20, transparent);
  animation: scan 0.5s ease forwards;
}
@keyframes scan {
  from { left: -60%; }
  to { left: 120%; }
}

/* Arrow column */
.nlp-arrow-col {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 28px;
  gap: 16px;
}

.pipeline-steps {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.pipe-step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  opacity: 0.3;
  transition: all 0.3s;
}

.pipe-step.active {
  opacity: 1;
  transform: scale(1.1);
}
.pipe-step.done { opacity: 1; }

.pipe-icon {
  font-size: 14px;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  transition: all 0.3s;
}
.pipe-step.active .pipe-icon {
  background: #eff6ff;
  border-color: #2563eb;
}
.pipe-step.done .pipe-icon {
  background: #f0fdf4;
  border-color: #22c55e;
}

.pipe-label {
  font-size: 9px;
  color: #94a3b8;
  font-weight: 600;
  text-align: center;
  white-space: nowrap;
}

.pipe-connector {
  width: 1px;
  height: 8px;
  background: #e2e8f0;
}

.big-arrow {
  font-size: 28px;
  color: #cbd5e1;
  transition: all 0.4s;
  margin-top: 4px;
}
.big-arrow.animated {
  color: #2563eb;
  animation: pulse-arrow 1.2s ease infinite;
}
@keyframes pulse-arrow {
  0%, 100% { transform: translateX(0); opacity: 1; }
  50% { transform: translateX(4px); opacity: 0.7; }
}

/* Brief card */
.brief-col {
  opacity: 0;
  transform: translateX(12px);
  transition: all 0.5s ease;
}
.brief-col.visible {
  opacity: 1;
  transform: translateX(0);
}

.brief-card {
  background: #f8fafc;
  border: 1.5px solid #e2e8f0;
  border-radius: 14px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.brief-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.brief-type {
  font-size: 12px;
  font-weight: 700;
  color: #2563eb;
  background: #eff6ff;
  padding: 3px 10px;
  border-radius: 99px;
}

.brief-priority {
  font-size: 11px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 99px;
}
.brief-priority.high { background: #fee2e2; color: #dc2626; }

.brief-title {
  font-size: 16px;
  font-weight: 800;
  color: #0f172a;
  margin: 0;
  line-height: 1.3;
}

.brief-section { display: flex; flex-direction: column; gap: 6px; }

.brief-label {
  font-size: 11px;
  font-weight: 700;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.4px;
}

.demand-row {
  display: flex;
  align-items: center;
  gap: 10px;
}
.demand-bar {
  flex: 1;
  height: 8px;
  background: #e2e8f0;
  border-radius: 99px;
  overflow: hidden;
}
.demand-fill {
  height: 100%;
  background: linear-gradient(90deg, #2563eb, #3a9a7a);
  border-radius: 99px;
  transition: width 1s ease 0.3s;
}
.demand-val { font-size: 14px; font-weight: 800; color: #2563eb; }

.intent-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}
.intent-tag {
  font-size: 11px;
  font-weight: 600;
  background: #eff6ff;
  color: #2563eb;
  border: 1px solid #bfdbfe;
  padding: 3px 10px;
  border-radius: 99px;
}

.brief-summary {
  font-size: 12px;
  color: #374151;
  line-height: 1.6;
  margin: 0;
  min-height: 56px;
}

.cursor {
  display: inline-block;
  animation: blink 0.7s step-end infinite;
  color: #2563eb;
  font-weight: 700;
}
@keyframes blink { 50% { opacity: 0; } }

.source-pills {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}
.source-pill {
  font-size: 11px;
  background: #f1f5f9;
  color: #64748b;
  padding: 3px 10px;
  border-radius: 99px;
}

.risk-row {
  display: flex;
  align-items: center;
  gap: 10px;
}
.risk-pill {
  font-size: 11px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 99px;
}
.risk-pill.yellow { background: #fef9c3; color: #ca8a04; }
.risk-hint { font-size: 11px; color: #94a3b8; }

.brief-actions {
  display: flex;
  gap: 8px;
  margin-top: 4px;
}
.brief-btn {
  padding: 8px 16px;
  border-radius: 8px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}
.brief-btn.primary {
  background: #2563eb;
  color: #fff;
  border: none;
}
.brief-btn.primary:hover { background: #1d4ed8; }
.brief-btn.secondary {
  background: transparent;
  color: #64748b;
  border: 1px solid #e2e8f0;
}
.brief-btn.secondary:hover { background: #f8fafc; }

/* ══════════════════════════════════
   EXISTING STYLES (unchanged)
══════════════════════════════════ */
.summary-row {
  display: flex;
  gap: 16px;
}

.stat-card {
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 16px 22px;
  display: flex;
  align-items: center;
  gap: 14px;
  flex: 1;
}
.stat-icon { font-size: 24px; }
.stat-value { font-size: 24px; font-weight: 700; }
.stat-label { font-size: 12px; color: var(--gray); margin-top: 2px; }

.filters {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}
.filter-btn {
  padding: 7px 16px;
  border-radius: 99px;
  border: 1px solid var(--border);
  background: #fff;
  cursor: pointer;
  font-size: 13px;
  color: var(--gray);
  transition: all 0.2s;
}
.filter-btn:hover { border-color: var(--blue); color: var(--blue); }
.filter-btn.active { background: var(--blue); color: #fff; border-color: var(--blue); }

.search-input {
  margin-left: auto;
  padding: 8px 16px;
  border: 1px solid var(--border);
  border-radius: 99px;
  font-size: 13px;
  width: 220px;
  outline: none;
  transition: border-color 0.2s;
}
.search-input:focus { border-color: var(--blue); }

.feedback-list { display: flex; flex-direction: column; gap: 12px; }

.feedback-card {
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 18px 22px;
  cursor: pointer;
  transition: box-shadow 0.2s, border-color 0.2s;
}
.feedback-card:hover { box-shadow: 0 2px 12px rgba(0,0,0,0.07); }
.feedback-card.expanded { border-color: var(--blue); }

.card-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;
  flex-wrap: wrap;
}
.tag { font-size: 11px; padding: 3px 10px; border-radius: 99px; font-weight: 600; }
.tag.feature { background: var(--blue-light); color: var(--blue); }
.tag.bug { background: #fee2e2; color: var(--red); }
.tag.general { background: var(--yellow-light); color: #92400e; }
.user, .date { font-size: 12px; color: var(--gray); }
.priority { font-size: 11px; font-weight: 600; padding: 3px 10px; border-radius: 99px; margin-left: auto; }
.priority.high { background: #fee2e2; color: var(--red); }
.priority.medium { background: var(--yellow-light); color: #92400e; }
.priority.low { background: var(--green-light); color: #166534; }
.chevron { font-size: 11px; color: var(--gray); }
.card-title { font-size: 15px; font-weight: 600; color: #111; }

.card-body {
  margin-top: 14px;
  padding-top: 14px;
  border-top: 1px solid var(--border);
}
.card-body p { font-size: 13px; color: #374151; line-height: 1.7; margin-bottom: 14px; }
.card-actions { display: flex; gap: 10px; }
.action-btn {
  padding: 6px 14px;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: #f9fafb;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s;
}
.action-btn:hover { background: var(--blue-light); border-color: var(--blue); color: var(--blue); }
.action-btn.liked { background: var(--blue-light); color: var(--blue); border-color: var(--blue); }
.empty { text-align: center; padding: 48px; color: var(--gray); font-size: 14px; }

/* ── Responsive ── */
@media (max-width: 1100px) {
  .nlp-columns {
    grid-template-columns: 1fr;
  }
  .nlp-arrow-col {
    flex-direction: row;
    padding-top: 0;
    justify-content: center;
  }
  .pipeline-steps { flex-direction: row; }
  .pipe-connector { width: 12px; height: 1px; align-self: center; margin: 0; }
  .big-arrow { font-size: 20px; }
}

@media (max-width: 1024px) {
  .feedback-page { padding: 24px; }
  .summary-row { flex-wrap: wrap; }
  .stat-card { flex: calc(50% - 8px); min-width: calc(50% - 8px); }
  .showcase-header { flex-direction: column; }
}

@media (max-width: 768px) {
  .feedback-page { padding: 20px 16px 100px; }
  h1 { font-size: 22px; }
  .summary-row { flex-direction: column; }
  .stat-card { width: 100%; }
  .search-input { width: 100%; margin-left: 0; }
  .priority { display: none; }
  .card-actions { flex-wrap: wrap; }
}


</style>