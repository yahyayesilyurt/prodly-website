<template>
  <div class="feedback-page">
    <h1>Feedback Analysis</h1>
    <p class="subtitle">Review and filter user submitted feedback</p>

    <!-- Summary Cards -->
    <div class="summary-row">
      <div class="stat-card" v-for="s in stats" :key="s.label">
        <span class="stat-icon">{{ s.icon }}</span>
        <div>
          <div class="stat-value">{{ s.value }}</div>
          <div class="stat-label">{{ s.label }}</div>
        </div>
      </div>
    </div>

    <!-- Filters -->
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

    <!-- Feedback List -->
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
          <span :class="['priority', item.priority]">{{ item.priority }}</span>
          <span class="chevron">{{ expandedId === item.id ? "▲" : "▼" }}</span>
        </div>
        <div class="card-title">{{ item.title }}</div>
        <div v-if="expandedId === item.id" class="card-body">
          <p>{{ item.detail }}</p>
          <div class="card-actions">
            <button
              :class="['action-btn', { liked: item.liked }]"
              @click.stop="
                item.liked = !item.liked;
                item.votes += item.liked ? 1 : -1;
              "
            >
              👍 {{ item.votes }}
            </button>
            <button class="action-btn" @click.stop>💬 Reply</button>
            <button
              class="action-btn mark"
              @click.stop="item.marked = !item.marked"
            >
              {{ item.marked ? "✅ Marked" : "🔖 Mark for Roadmap" }}
            </button>
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
import { ref, computed } from "vue";

const activeFilter = ref("all");
const searchQuery = ref("");
const expandedId = ref(null);

function toggle(id) {
  expandedId.value = expandedId.value === id ? null : id;
}

const stats = [
  { icon: "📥", value: 392, label: "Total Feedback" },
  { icon: "🚀", value: 124, label: "Feature Requests" },
  { icon: "🐛", value: 53, label: "Bug Reports" },
  { icon: "💡", value: 215, label: "General Ideas" },
];

const filters = [
  { label: "All", value: "all" },
  { label: "🚀 Feature Request", value: "feature" },
  { label: "🐛 Bug Report", value: "bug" },
  { label: "💡 General", value: "general" },
];

const feedback = ref([
  {
    id: 1,
    type: "feature",
    typeLabel: "🚀 Feature Request",
    user: "alice@corp.com",
    date: "2026-03-28",
    title: "Add dark mode support across the dashboard",
    detail:
      "Many users work at night and would benefit from a dark mode. Should apply to all views including charts and the sidebar.",
    priority: "high",
    votes: 47,
    liked: false,
    marked: false,
  },
  {
    id: 2,
    type: "bug",
    typeLabel: "🐛 Bug Report",
    user: "bob@startup.io",
    date: "2026-03-27",
    title: "Roadmap simulation freezes on large datasets",
    detail:
      "When uploading more than 500 feedback entries, the Run Simulation button becomes unresponsive. Reproducible on Chrome 120+.",
    priority: "high",
    votes: 31,
    liked: false,
    marked: false,
  },
  {
    id: 3,
    type: "general",
    typeLabel: "💡 General",
    user: "carol@agency.co",
    date: "2026-03-26",
    title: "Onboarding flow is confusing for new users",
    detail:
      "The first-time experience lacks guidance. A short interactive tour or tooltip hints would help new product managers get started faster.",
    priority: "medium",
    votes: 22,
    liked: false,
    marked: false,
  },
  {
    id: 4,
    type: "feature",
    typeLabel: "🚀 Feature Request",
    user: "dave@enterprise.com",
    date: "2026-03-25",
    title: "Export reports as PDF or CSV",
    detail:
      "It would be very helpful to download reports in PDF or CSV format to share with stakeholders who don't have access to the platform.",
    priority: "medium",
    votes: 58,
    liked: false,
    marked: false,
  },
  {
    id: 5,
    type: "bug",
    typeLabel: "🐛 Bug Report",
    user: "eve@product.dev",
    date: "2026-03-24",
    title: "Customer Demand Score not updating after filter change",
    detail:
      "When changing the date filter, the demand score widget stays at its initial value and does not reflect the filtered feedback.",
    priority: "low",
    votes: 14,
    liked: false,
    marked: false,
  },
  {
    id: 6,
    type: "general",
    typeLabel: "💡 General",
    user: "frank@scaleup.io",
    date: "2026-03-23",
    title: "Slack integration for feedback notifications",
    detail:
      "It would be great to receive a Slack notification whenever high-priority feedback comes in, with a direct link to the item.",
    priority: "low",
    votes: 19,
    liked: false,
    marked: false,
  },
]);

const filteredFeedback = computed(() => {
  return feedback.value
    .filter(
      (f) => activeFilter.value === "all" || f.type === activeFilter.value,
    )
    .filter(
      (f) =>
        f.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        f.user.toLowerCase().includes(searchQuery.value.toLowerCase()),
    );
});
</script>

<style scoped>
.feedback-page {
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

/* Summary */
.summary-row {
  display: flex;
  gap: 16px;
  margin-bottom: 28px;
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

.stat-icon {
  font-size: 24px;
}
.stat-value {
  font-size: 24px;
  font-weight: 700;
}
.stat-label {
  font-size: 12px;
  color: var(--gray);
  margin-top: 2px;
}

/* Filters */
.filters {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
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
.filter-btn:hover {
  border-color: var(--blue);
  color: var(--blue);
}
.filter-btn.active {
  background: var(--blue);
  color: #fff;
  border-color: var(--blue);
}

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
.search-input:focus {
  border-color: var(--blue);
}

/* Cards */
.feedback-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.feedback-card {
  background: #fff;
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 18px 22px;
  cursor: pointer;
  transition:
    box-shadow 0.2s,
    border-color 0.2s;
}
.feedback-card:hover {
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.07);
}
.feedback-card.expanded {
  border-color: var(--blue);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;
  flex-wrap: wrap;
}

.tag {
  font-size: 11px;
  padding: 3px 10px;
  border-radius: 99px;
  font-weight: 600;
}
.tag.feature {
  background: var(--blue-light);
  color: var(--blue);
}
.tag.bug {
  background: #fee2e2;
  color: var(--red);
}
.tag.general {
  background: var(--yellow-light);
  color: #92400e;
}

.user,
.date {
  font-size: 12px;
  color: var(--gray);
}

.priority {
  font-size: 11px;
  font-weight: 600;
  padding: 3px 10px;
  border-radius: 99px;
  margin-left: auto;
}
.priority.high {
  background: #fee2e2;
  color: var(--red);
}
.priority.medium {
  background: var(--yellow-light);
  color: #92400e;
}
.priority.low {
  background: var(--green-light);
  color: #166534;
}

.chevron {
  font-size: 11px;
  color: var(--gray);
}

.card-title {
  font-size: 15px;
  font-weight: 600;
  color: #111;
}

.card-body {
  margin-top: 14px;
  padding-top: 14px;
  border-top: 1px solid var(--border);
}
.card-body p {
  font-size: 13px;
  color: #374151;
  line-height: 1.7;
  margin-bottom: 14px;
}

.card-actions {
  display: flex;
  gap: 10px;
}

.action-btn {
  padding: 6px 14px;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: #f9fafb;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s;
}
.action-btn:hover {
  background: var(--blue-light);
  border-color: var(--blue);
  color: var(--blue);
}
.action-btn.liked {
  background: var(--blue-light);
  color: var(--blue);
  border-color: var(--blue);
}
.action-btn.mark {
  margin-left: auto;
}

.empty {
  text-align: center;
  padding: 48px;
  color: var(--gray);
  font-size: 14px;
}

@media (max-width: 1024px) {
  .feedback-page {
    padding: 24px;
  }
  .summary-row {
    flex-wrap: wrap;
  }
  .stat-card {
    flex: calc(50% - 8px);
    min-width: calc(50% - 8px);
  }
}

@media (max-width: 768px) {
  .feedback-page {
    padding: 20px 16px 100px;
  }

  h1 {
    font-size: 22px;
  }

  .summary-row {
    flex-direction: column;
  }
  .stat-card {
    width: 100%;
  }

  .filters {
    gap: 8px;
  }

  .search-input {
    width: 100%;
    margin-left: 0;
  }

  .card-header {
    gap: 8px;
  }

  .priority {
    display: none;
  }

  .card-actions {
    flex-wrap: wrap;
  }
  .action-btn.mark {
    margin-left: 0;
  }
}
</style>
