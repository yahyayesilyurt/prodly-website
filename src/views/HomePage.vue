<template>
  <div class="home-page">
    <!-- HERO -->
    <section class="hero">
      <div class="hero-badge">✦ AI-Powered Product Intelligence</div>
      <h1 class="hero-title">
        Build better products<br />
        <span class="gradient-text">with confidence</span>
      </h1>
      <p class="hero-sub">
        Prodly turns raw user feedback into clear roadmap decisions — powered by
        probabilistic AI simulation and real delivery risk analysis.
      </p>
      <div class="hero-actions">
        <button class="btn-primary" @click="$emit('navigate', 'dashboard')">
          Go to Dashboard →
        </button>
        <button class="btn-ghost" @click="scrollToPricing">View Pricing</button>
      </div>

      <!-- Floating stat cards -->
      <div class="hero-stats">
        <div class="stat-bubble" v-for="s in heroStats" :key="s.label">
          <span class="bubble-val">{{ s.val }}</span>
          <span class="bubble-label">{{ s.label }}</span>
        </div>
      </div>

      <!-- Decorative grid -->
      <div class="hero-grid" aria-hidden="true">
        <div v-for="n in 48" :key="n" class="grid-dot"></div>
      </div>
    </section>

    <!-- FEATURES -->
    <section class="features">
      <h2 class="section-title">Everything you need to ship smarter</h2>
      <p class="section-sub">
        One platform. Four powerful views. Zero guesswork.
      </p>
      <div class="features-grid">
        <div class="feature-card" v-for="f in features" :key="f.title">
          <div class="feature-icon">{{ f.icon }}</div>
          <h3>{{ f.title }}</h3>
          <p>{{ f.desc }}</p>
          <button class="feature-link" @click="$emit('navigate', f.page)">
            Open {{ f.title }} →
          </button>
        </div>
      </div>
    </section>

    <!-- PRICING -->
    <section class="pricing" id="pricing-section">
      <h2 class="section-title">Simple, transparent pricing</h2>
      <p class="section-sub">Start free. Scale as you grow. No hidden fees.</p>

      <!-- Toggle -->
      <div class="billing-toggle">
        <span
          :class="{ active: billing === 'monthly' }"
          @click="billing = 'monthly'"
          >Monthly</span
        >
        <div
          class="toggle-track"
          @click="billing = billing === 'monthly' ? 'annual' : 'monthly'"
        >
          <div :class="['toggle-thumb', { right: billing === 'annual' }]"></div>
        </div>
        <span
          :class="{ active: billing === 'annual' }"
          @click="billing = 'annual'"
        >
          Annual <em class="save-badge">Save 20%</em>
        </span>
      </div>

      <div class="pricing-grid">
        <div
          v-for="plan in plans"
          :key="plan.name"
          :class="['pricing-card', { popular: plan.popular }]"
        >
          <div v-if="plan.popular" class="popular-badge">Most Popular</div>
          <div class="plan-icon">{{ plan.icon }}</div>
          <h3 class="plan-name">{{ plan.name }}</h3>
          <p class="plan-desc">{{ plan.desc }}</p>

          <div class="plan-price">
            <span class="currency">$</span>
            <span class="amount">{{
              billing === "annual" ? plan.annualPrice : plan.monthlyPrice
            }}</span>
            <span class="per">/mo</span>
          </div>
          <p class="billed-note">
            {{ billing === "annual" ? "Billed annually" : "Billed monthly" }}
          </p>

          <button
            :class="plan.popular ? 'btn-primary' : 'btn-outline'"
            @click="selectPlan(plan.name)"
          >
            {{ plan.cta }}
          </button>

          <ul class="plan-features">
            <li v-for="feat in plan.features" :key="feat">
              <span class="check">✓</span> {{ feat }}
            </li>
          </ul>
        </div>
      </div>

      <!-- FAQ strip -->
      <div class="faq-strip">
        <div
          class="faq-item"
          v-for="q in faqs"
          :key="q.q"
          @click="toggleFaq(q)"
        >
          <div class="faq-q">
            {{ q.q }}
            <span class="faq-chevron">{{ q.open ? "▲" : "▼" }}</span>
          </div>
          <div v-if="q.open" class="faq-a">{{ q.a }}</div>
        </div>
      </div>
    </section>

    <!-- Toast -->
    <div v-if="toast" class="toast">{{ toast }}</div>
  </div>
</template>

<script setup>
import { ref } from "vue";

defineEmits(["navigate"]);

const billing = ref("monthly");
const toast = ref("");

const heroStats = [
  { val: "392+", label: "Feedback items analyzed" },
  { val: "80%", label: "Avg success probability" },
  { val: "3x", label: "Faster roadmap decisions" },
];

const features = [
  {
    icon: "📈",
    title: "Dashboard",
    desc: "Get a birds-eye view of feedback volume, demand scores, risk levels and roadmap simulations — all in one place.",
    page: "dashboard",
  },
  {
    icon: "💬",
    title: "Feedback Analysis",
    desc: "Filter, search and prioritize user feedback by type and sentiment. Vote, tag and mark items directly for your roadmap.",
    page: "feedback",
  },
  {
    icon: "📋",
    title: "Roadmap Simulation",
    desc: "Run probabilistic simulations across features and get AI-powered recommendations based on delivery risk and demand.",
    page: "roadmap",
  },
  {
    icon: "📊",
    title: "Reports",
    desc: "Generate exportable summaries in PDF, CSV or JSON. Visualize trends over time and share insights with stakeholders.",
    page: "reports",
  },
];

const plans = [
  {
    icon: "🌱",
    name: "Starter",
    popular: false,
    desc: "Perfect for solo PMs and early-stage teams.",
    monthlyPrice: 0,
    annualPrice: 0,
    cta: "Get started free",
    features: [
      "Up to 100 feedback items",
      "1 roadmap simulation / month",
      "Basic dashboard",
      "CSV export",
      "Email support",
    ],
  },
  {
    icon: "🏢",
    name: "Enterprise",
    popular: false,
    desc: "Custom solutions for large organisations.",
    monthlyPrice: 79,
    annualPrice: 63,
    cta: "Contact sales",
    features: [
      "Everything in Starter",
      "SSO / SAML login",
      "Custom AI model tuning",
      "Dedicated success manager",
      "SLA guarantee",
      "On-premise option",
    ],
  },
];

const faqs = ref([
  {
    q: "Can I change plans later?",
    a: "Absolutely. You can upgrade or downgrade at any time from your account settings. Changes take effect immediately.",
    open: false,
  },
  {
    q: "Is there a free trial for Pro?",
    a: "Yes — Pro comes with a 14-day free trial, no credit card required. You will get full access to all features during the trial.",
    open: false,
  },
  {
    q: "What payment methods do you accept?",
    a: "We accept all major credit cards (Visa, Mastercard, Amex) as well as bank transfers for annual Enterprise plans.",
    open: false,
  },
  {
    q: "Is my data secure?",
    a: "Yes. All data is encrypted at rest (AES-256) and in transit (TLS 1.3). We are SOC 2 Type II certified.",
    open: false,
  },
]);

function toggleFaq(q) {
  q.open = !q.open;
}

function scrollToPricing() {
  document
    .getElementById("pricing-section")
    ?.scrollIntoView({ behavior: "smooth" });
}

function selectPlan(name) {
  showToast(`${name} plan selected! Redirecting to checkout...`);
}

function showToast(msg) {
  toast.value = msg;
  setTimeout(() => {
    toast.value = "";
  }, 3000);
}
</script>

<style scoped>
.home-page {
  overflow-x: hidden;
}

/* ── HERO ─────────────────────────────────── */
.hero {
  position: relative;
  padding: 72px 60px 80px;
  background: #fff;
  border-bottom: 1px solid var(--border);
  overflow: hidden;
}

.hero-badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  font-weight: 600;
  color: var(--blue);
  background: var(--blue-light);
  padding: 5px 14px;
  border-radius: 99px;
  margin-bottom: 24px;
  letter-spacing: 0.3px;
}

.hero-title {
  font-size: 52px;
  font-weight: 800;
  line-height: 1.15;
  letter-spacing: -1.5px;
  color: #0f172a;
  margin-bottom: 20px;
  max-width: 560px;
}

.gradient-text {
  background: linear-gradient(90deg, #2563eb, #3a9a7a);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero-sub {
  font-size: 16px;
  color: #64748b;
  line-height: 1.7;
  max-width: 480px;
  margin-bottom: 36px;
}

.hero-actions {
  display: flex;
  gap: 14px;
  margin-bottom: 48px;
}

.btn-primary {
  background: var(--blue);
  color: #fff;
  border: none;
  padding: 13px 26px;
  border-radius: 10px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition:
    background 0.2s,
    transform 0.1s;
}
.btn-primary:hover {
  background: #1d4ed8;
  transform: translateY(-1px);
}

.btn-ghost {
  background: transparent;
  color: #374151;
  border: 1.5px solid var(--border);
  padding: 13px 26px;
  border-radius: 10px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}
.btn-ghost:hover {
  border-color: var(--blue);
  color: var(--blue);
}

/* Stat bubbles */
.hero-stats {
  display: flex;
  gap: 16px;
}

.stat-bubble {
  background: #f8fafc;
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 14px 22px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.bubble-val {
  font-size: 22px;
  font-weight: 800;
  color: #0f172a;
}

.bubble-label {
  font-size: 11px;
  color: #64748b;
}

/* Decorative dots */
.hero-grid {
  position: absolute;
  right: -10px;
  top: 0;
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 24px;
  padding: 40px;
  opacity: 0.35;
  pointer-events: none;
}

.grid-dot {
  width: 4px;
  height: 4px;
  background: #94a3b8;
  border-radius: 50%;
}

/* ── FEATURES ─────────────────────────────── */
.features {
  padding: 72px 60px;
  background: #f8fafc;
}

.section-title {
  font-size: 32px;
  font-weight: 800;
  letter-spacing: -0.8px;
  color: #0f172a;
  margin-bottom: 8px;
  text-align: center;
}

.section-sub {
  font-size: 15px;
  color: #64748b;
  text-align: center;
  margin-bottom: 48px;
}

.features-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
}

.feature-card {
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 28px 24px;
  transition:
    box-shadow 0.2s,
    transform 0.2s;
}

.feature-card:hover {
  box-shadow: 0 8px 32px rgba(37, 99, 235, 0.08);
  transform: translateY(-3px);
}

.feature-icon {
  font-size: 28px;
  margin-bottom: 14px;
}

.feature-card h3 {
  font-size: 16px;
  font-weight: 700;
  margin-bottom: 10px;
  color: #0f172a;
}

.feature-card p {
  font-size: 13px;
  color: #64748b;
  line-height: 1.65;
  margin-bottom: 18px;
}

.feature-link {
  background: none;
  border: none;
  color: var(--blue);
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  padding: 0;
  transition: gap 0.2s;
}
.feature-link:hover {
  text-decoration: underline;
}

/* ── PRICING ──────────────────────────────── */
.pricing {
  padding: 80px 60px;
  background: #fff;
  border-top: 1px solid var(--border);
}

/* Billing toggle */
.billing-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 14px;
  margin-bottom: 52px;
  font-size: 14px;
  font-weight: 500;
  color: #64748b;
}

.billing-toggle span {
  cursor: pointer;
  transition: color 0.2s;
}
.billing-toggle span.active {
  color: #0f172a;
  font-weight: 700;
}

.toggle-track {
  width: 44px;
  height: 24px;
  background: #e2e8f0;
  border-radius: 99px;
  position: relative;
  cursor: pointer;
  transition: background 0.2s;
}
.toggle-track:has(.right) {
  background: var(--blue);
}

.toggle-thumb {
  position: absolute;
  top: 3px;
  left: 3px;
  width: 18px;
  height: 18px;
  background: #fff;
  border-radius: 50%;
  transition: left 0.2s;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.15);
}
.toggle-thumb.right {
  left: 23px;
}

.save-badge {
  font-style: normal;
  font-size: 10px;
  font-weight: 700;
  background: var(--green-light);
  color: #166534;
  padding: 2px 7px;
  border-radius: 99px;
  margin-left: 4px;
}

/* Pricing cards */
.pricing-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 24px;
  max-width: 900px;
  margin: 0 auto 64px;
}

.pricing-card {
  border: 1.5px solid var(--border);
  border-radius: 20px;
  padding: 36px 30px;
  position: relative;
  background: #fff;
  transition:
    box-shadow 0.2s,
    transform 0.2s;
}

.pricing-card:hover {
  box-shadow: 0 8px 40px rgba(0, 0, 0, 0.08);
  transform: translateY(-4px);
}

.pricing-card.popular {
  border-color: var(--blue);
  background: #f0f7ff;
  box-shadow: 0 4px 24px rgba(37, 99, 235, 0.12);
}

.popular-badge {
  position: absolute;
  top: -13px;
  left: 50%;
  transform: translateX(-50%);
  background: var(--blue);
  color: #fff;
  font-size: 11px;
  font-weight: 700;
  padding: 4px 14px;
  border-radius: 99px;
  white-space: nowrap;
}

.plan-icon {
  font-size: 28px;
  margin-bottom: 12px;
}

.plan-name {
  font-size: 20px;
  font-weight: 800;
  color: #0f172a;
  margin-bottom: 6px;
}

.plan-desc {
  font-size: 13px;
  color: #64748b;
  margin-bottom: 24px;
  line-height: 1.5;
}

.plan-price {
  display: flex;
  align-items: flex-end;
  gap: 2px;
  margin-bottom: 4px;
}

.currency {
  font-size: 20px;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 6px;
}
.amount {
  font-size: 48px;
  font-weight: 800;
  color: #0f172a;
  line-height: 1;
}
.per {
  font-size: 14px;
  color: #64748b;
  margin-bottom: 8px;
}

.billed-note {
  font-size: 12px;
  color: #94a3b8;
  margin-bottom: 24px;
}

.btn-outline {
  width: 100%;
  padding: 12px;
  border: 1.5px solid var(--blue);
  background: transparent;
  color: var(--blue);
  border-radius: 10px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  margin-bottom: 28px;
  transition: all 0.2s;
}
.btn-outline:hover {
  background: var(--blue);
  color: #fff;
}

.pricing-card .btn-primary {
  width: 100%;
  padding: 12px;
  margin-bottom: 28px;
  border-radius: 10px;
}

.plan-features {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.plan-features li {
  font-size: 13px;
  color: #374151;
  display: flex;
  align-items: flex-start;
  gap: 8px;
}

.check {
  color: var(--green);
  font-weight: 700;
  font-size: 14px;
  flex-shrink: 0;
}

/* FAQ strip */
.faq-strip {
  max-width: 640px;
  margin: 0 auto;
  border: 1px solid var(--border);
  border-radius: 16px;
  overflow: hidden;
}

.faq-item {
  padding: 18px 24px;
  border-bottom: 1px solid var(--border);
  cursor: pointer;
  transition: background 0.15s;
}
.faq-item:last-child {
  border-bottom: none;
}
.faq-item:hover {
  background: #f8fafc;
}

.faq-q {
  font-size: 14px;
  font-weight: 600;
  color: #0f172a;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.faq-chevron {
  font-size: 11px;
  color: #94a3b8;
}

.faq-a {
  font-size: 13px;
  color: #64748b;
  line-height: 1.7;
  margin-top: 10px;
}

/* ── TOAST ────────────────────────────────── */
.toast {
  position: fixed;
  bottom: 28px;
  left: 50%;
  transform: translateX(-50%);
  background: #0f172a;
  color: #fff;
  padding: 12px 24px;
  border-radius: 99px;
  font-size: 13px;
  font-weight: 500;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  animation: fadeUp 0.3s ease;
  z-index: 200;
  white-space: nowrap;
}

@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}

/* Tablet */
@media (max-width: 1024px) {
  .hero {
    padding: 52px 32px 60px;
  }
  .hero-title {
    font-size: 38px;
  }
  .features-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  .pricing-grid {
    gap: 16px;
  }
  .pricing {
    padding: 60px 32px;
  }
  .features {
    padding: 52px 32px;
  }
  .kpi-row {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Mobile */
@media (max-width: 768px) {
  .hero {
    padding: 36px 20px 120px;
  }

  .hero-title {
    font-size: 28px;
    letter-spacing: -0.5px;
  }

  .hero-sub {
    font-size: 14px;
  }

  .hero-actions {
    flex-direction: column;
  }

  .hero-actions .btn-primary,
  .hero-actions .btn-ghost {
    width: 100%;
    text-align: center;
  }

  .hero-stats {
    flex-direction: column;
    gap: 10px;
  }

  .stat-bubble {
    flex-direction: row;
    align-items: center;
    gap: 12px;
  }

  .hero-grid {
    display: none;
  }

  .features {
    padding: 40px 20px 20px;
  }
  .features-grid {
    grid-template-columns: 1fr;
  }

  .pricing {
    padding: 40px 20px 100px;
  }
  .pricing-grid {
    grid-template-columns: 1fr;
    max-width: 100%;
  }

  .section-title {
    font-size: 24px;
  }

  .billing-toggle {
    font-size: 13px;
    gap: 10px;
  }

  .faq-strip {
    border-radius: 12px;
  }
}
</style>
