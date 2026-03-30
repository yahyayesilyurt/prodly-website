<template>
  <div class="sim-page">
    <h1>AI Roadmap Recommendation</h1>
    <p class="subtitle">
      Probabilistic simulation based on user feedback and delivery risk
    </p>

    <div class="results-box">
      <h2>Results</h2>

      <div v-if="loading" class="features-list">
        <div v-for="f in features" :key="f.name" class="feature-row loading">
          <span class="fname">{{ f.name }}</span>
          <div class="bar">
            <div class="fill-sm" :style="{ background: f.color }"></div>
          </div>
        </div>
      </div>

      <div v-else class="features-list">
        <div v-for="f in features" :key="f.name" class="feature-row">
          <span class="fname">{{ f.name }}</span>
          <div class="bar">
            <div
              class="fill"
              :style="{ width: f.prob + '%', background: f.color }"
            ></div>
          </div>
          <span class="prob-label">{{ f.prob }}% success probability</span>
        </div>

        <div class="recommendation">
          <h3>AI Recommendation</h3>
          <p>
            <strong>Feature A</strong> is the most feasible roadmap option based
            on delivery probability and customer demand.
          </p>
        </div>
      </div>
    </div>

    <button class="btn-back" @click="$emit('navigate', 'dashboard')">
      ← Back to Dashboard
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

defineEmits(["navigate"]);

const loading = ref(true);

const features = [
  { name: "Feature A", prob: 80, color: "#2563eb" },
  { name: "Feature B", prob: 55, color: "#eab308" },
  { name: "Feature C", prob: 63, color: "#22c55e" },
];

onMounted(() => {
  setTimeout(() => {
    loading.value = false;
  }, 1800);
});
</script>

<style scoped>
.sim-page {
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

.results-box {
  background: #fff;
  border-radius: 12px;
  padding: 32px;
  border: 1px solid var(--border);
  max-width: 600px;
}

h2 {
  font-size: 20px;
  font-weight: 600;
  margin-bottom: 24px;
}

.features-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.feature-row {
  display: flex;
  align-items: center;
  gap: 14px;
  font-size: 14px;
}

.fname {
  width: 80px;
  font-weight: 500;
}

.bar {
  width: 240px;
  height: 10px;
  background: #f3f4f6;
  border-radius: 99px;
  overflow: hidden;
}

.fill {
  height: 100%;
  border-radius: 99px;
  transition: width 0.8s ease;
}

.loading .bar {
  width: 30px;
  height: 10px;
}
.fill-sm {
  width: 100%;
  height: 100%;
  border-radius: 4px;
}

.prob-label {
  color: var(--gray);
  font-size: 13px;
  white-space: nowrap;
}

.recommendation {
  margin-top: 8px;
  padding-top: 20px;
  border-top: 1px solid var(--border);
}

.recommendation h3 {
  font-size: 15px;
  font-weight: 600;
  margin-bottom: 8px;
}
.recommendation p {
  font-size: 13px;
  color: #374151;
  line-height: 1.6;
}

.btn-back {
  margin-top: 24px;
  background: transparent;
  border: 1px solid var(--border);
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 13px;
  color: var(--gray);
  transition: all 0.2s;
}
.btn-back:hover {
  background: #f9fafb;
  color: #111;
}

@media (max-width: 768px) {
  .sim-page {
    padding: 20px 16px 100px;
  }

  h1 {
    font-size: 22px;
  }

  .results-box {
    padding: 20px 16px;
  }

  .feature-row {
    flex-wrap: wrap;
  }

  .bar {
    width: 100%;
  }

  .prob-label {
    width: 100%;
    text-align: right;
    font-size: 12px;
  }
}
</style>
