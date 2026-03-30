<template>
  <aside class="sidebar">
    <nav>
      <ul>
        <li
          v-for="item in navItems"
          :key="item.id"
          :class="{ active: activePage === item.id }"
          @click="$emit('navigate', item.id)"
        >
          <span class="icon">{{ item.icon }}</span>
          <span class="nav-label">{{ item.label }}</span>
        </li>
      </ul>
    </nav>
  </aside>
</template>

<script setup>
defineProps({ activePage: String });
defineEmits(["navigate"]);

const navItems = [
  { id: "home", icon: "🏠", label: "Home" },
  { id: "dashboard", icon: "📈", label: "Dashboard" },
  { id: "feedback", icon: "💬", label: "Feedback Analysis" },
  { id: "roadmap", icon: "📋", label: "Roadmap Simulation" },
  { id: "reports", icon: "📊", label: "Reports" },
];
</script>

<style scoped>
.sidebar {
  width: var(--sidebar-w);
  background: #fff;
  border-right: 1px solid var(--border);
  padding: 24px 0;
  min-height: 100vh;
  flex-shrink: 0;
  transition: width 0.2s;
}

ul {
  list-style: none;
}

li {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 24px;
  cursor: pointer;
  font-size: 14px;
  color: var(--gray);
  border-radius: 0 8px 8px 0;
  margin-right: 12px;
  transition: all 0.2s;
  white-space: nowrap;
}

li:hover {
  background: var(--blue-light);
  color: var(--blue);
}
li.active {
  background: var(--blue-light);
  color: var(--blue);
  font-weight: 600;
}

.icon {
  font-size: 16px;
  flex-shrink: 0;
}

/* Tablet */
@media (max-width: 1024px) {
  .sidebar {
    padding: 16px 0;
  }

  li {
    padding: 12px 16px;
    justify-content: center;
    margin-right: 0;
    border-radius: 8px;
    margin: 2px 8px;
  }

  .nav-label {
    display: none;
  }
  .icon {
    font-size: 20px;
  }
}

/* Mobile */
@media (max-width: 768px) {
  .sidebar {
    width: 100%;
    min-height: unset;
    border-right: none;
    border-top: 1px solid var(--border);
    padding: 0;
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 40;
    background: #fff;
  }

  nav {
    width: 100%;
  }

  ul {
    display: flex;
    flex-direction: row;
    justify-content: space-around;
  }

  li {
    flex-direction: column;
    gap: 4px;
    padding: 10px 8px;
    margin: 0;
    border-radius: 0;
    flex: 1;
    justify-content: center;
    font-size: 10px;
  }

  li.active {
    border-radius: 0;
    border-top: 2px solid var(--blue);
    background: var(--blue-light);
  }

  .nav-label {
    display: block;
    font-size: 10px;
  }
  .icon {
    font-size: 18px;
  }
}
</style>
