<template>
  <div class="app-container">
    <!-- Sidebar -->
    <aside class="sidebar" :class="{ 'sidebar-mobile': sidebarOpen && isMobile }">
      <!-- Logo -->
      <div class="sidebar-logo">
        <span class="logo-icon">🏭</span>
        <span v-if="!collapsed" class="logo-text">Catalyst</span>
      </div>

      <!-- Navigation -->
      <nav class="sidebar-nav">
        <router-link
          v-for="item in navItems"
          :key="item.to"
          :to="item.to"
          class="nav-item"
          :class="{ active: isActive(item.to) }"
          :title="item.label"
          @click="closeMobileSidebar"
        >
          <span class="nav-icon">{{ item.icon }}</span>
          <span v-if="!collapsed" class="nav-label">{{ item.label }}</span>
        </router-link>
      </nav>

      <!-- Footer -->
      <div class="sidebar-footer">
        <button class="nav-item footer-btn" @click="toggleTheme" :title="isDark ? 'Light mode' : 'Dark mode'">
          <span class="nav-icon">{{ isDark ? '☀️' : '🌙' }}</span>
          <span v-if="!collapsed" class="nav-label">Theme</span>
        </button>
        <button class="nav-item footer-btn" @click="showProfileDetails = true" title="Profile">
          <span class="nav-icon">👤</span>
          <span v-if="!collapsed" class="nav-label">Profile</span>
        </button>
      </div>

      <!-- Close button (mobile) -->
      <button v-if="isMobile" class="sidebar-close" @click="closeSidebar">✕</button>
    </aside>

    <!-- Main Content -->
    <div class="main-wrapper">
      <!-- Top Bar -->
      <header class="topbar">
        <div class="topbar-left">
          <button v-if="isMobile" class="hamburger" @click="toggleSidebar" aria-label="Toggle menu">
            ☰
          </button>
          <h1 class="page-title">{{ pageTitle }}</h1>
        </div>

        <div class="topbar-right">
          <LanguageSwitcher v-if="!isMobile" />
          <button class="icon-btn" title="Notifications">🔔</button>
          <button class="icon-btn" title="Settings">⚙️</button>
        </div>
      </header>

      <!-- Filter Bar -->
      <FilterBar v-if="showFilters" />

      <!-- Content Area -->
      <main class="content">
        <router-view />
      </main>
    </div>

    <!-- Mobile Backdrop -->
    <Transition name="backdrop-fade">
      <div v-if="sidebarOpen && isMobile" class="sidebar-backdrop" @click="closeSidebar" />
    </Transition>

    <!-- Modals -->
    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRoute } from 'vue-router'
import { useAuth } from '../composables/useAuth'
import { useI18n } from '../composables/useI18n'
import { api } from '../api'
import LanguageSwitcher from '../components/LanguageSwitcher.vue'
import FilterBar from '../components/FilterBar.vue'
import ProfileDetailsModal from '../components/ProfileDetailsModal.vue'
import TasksModal from '../components/TasksModal.vue'

const route = useRoute()
const { currentUser } = useAuth()
const { t } = useI18n()

const sidebarOpen = ref(true)
const collapsed = ref(false)
const isDark = ref(localStorage.getItem('theme') === 'dark')
const isMobile = ref(false)
const windowWidth = ref(typeof window !== 'undefined' ? window.innerWidth : 1024)
const showProfileDetails = ref(false)
const showTasks = ref(false)
const apiTasks = ref([])

const showFilters = computed(() => route.path !== '/')

const tasks = computed(() => {
  return [...currentUser.value.tasks, ...apiTasks.value]
})

const loadTasks = async () => {
  try {
    apiTasks.value = await api.getTasks()
  } catch (err) {
    console.error('Failed to load tasks:', err)
  }
}

const addTask = async (taskData) => {
  try {
    const newTask = await api.createTask(taskData)
    apiTasks.value.unshift(newTask)
  } catch (err) {
    console.error('Failed to add task:', err)
  }
}

const deleteTask = async (taskId) => {
  try {
    const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)
    if (isMockTask) {
      const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
      if (index !== -1) {
        currentUser.value.tasks.splice(index, 1)
      }
    } else {
      await api.deleteTask(taskId)
      apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
    }
  } catch (err) {
    console.error('Failed to delete task:', err)
  }
}

const toggleTask = async (taskId) => {
  try {
    const mockTask = currentUser.value.tasks.find(t => t.id === taskId)
    if (mockTask) {
      mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
    } else {
      const updatedTask = await api.toggleTask(taskId)
      const index = apiTasks.value.findIndex(t => t.id === taskId)
      if (index !== -1) {
        apiTasks.value[index] = updatedTask
      }
    }
  } catch (err) {
    console.error('Failed to toggle task:', err)
  }
}

const pageTitles = {
  '/': 'Dashboard',
  '/inventory': 'Inventory Management',
  '/orders': 'Orders',
  '/demand': 'Demand Forecast',
  '/spending': 'Finance',
  '/reports': 'Reports',
  '/restocking': 'Restocking',
  '/backlog': 'Backlog'
}

const pageTitle = computed(() => pageTitles[route.path] || 'Page')

const navItems = [
  { label: 'Dashboard', icon: '📊', to: '/' },
  { label: 'Inventory', icon: '📦', to: '/inventory' },
  { label: 'Orders', icon: '🛒', to: '/orders' },
  { label: 'Finance', icon: '💰', to: '/spending' },
  { label: 'Demand', icon: '📈', to: '/demand' },
  { label: 'Restocking', icon: '🔄', to: '/restocking' },
  { label: 'Reports', icon: '📋', to: '/reports' }
]

const isActive = (path) => route.path === path

const toggleTheme = () => {
  isDark.value = !isDark.value
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
  document.documentElement.classList.toggle('dark-mode', isDark.value)
}

const closeMobileSidebar = () => {
  if (isMobile.value) {
    sidebarOpen.value = false
  }
}

const toggleSidebar = () => {
  sidebarOpen.value = !sidebarOpen.value
}

const closeSidebar = () => {
  sidebarOpen.value = false
}

const handleResize = () => {
  windowWidth.value = window.innerWidth
  isMobile.value = windowWidth.value < 1024
  if (!isMobile.value) {
    sidebarOpen.value = true
  }
}

onMounted(() => {
  handleResize()
  window.addEventListener('resize', handleResize)
  if (isDark.value) {
    document.documentElement.classList.add('dark-mode')
  }
  loadTasks()
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.app-container {
  display: grid;
  grid-template-columns: 260px 1fr;
  min-height: 100vh;
  background: var(--bg-secondary);
}

/* === SIDEBAR === */
.sidebar {
  display: flex;
  flex-direction: column;
  background: var(--bg-primary);
  border-right: 1px solid var(--border-light);
  padding: 1.5rem 1rem;
  overflow-y: auto;
  gap: 2rem;
  transition: all 300ms cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-weight: 700;
  color: var(--text-primary);
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border-light);
  user-select: none;
}

.logo-icon {
  font-size: 28px;
}

.logo-text {
  font-size: 16px;
  letter-spacing: -0.5px;
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  list-style: none;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 0.875rem;
  border-radius: 8px;
  color: var(--text-secondary);
  text-decoration: none;
  cursor: pointer;
  transition: all 200ms ease;
  font-size: 14px;
  font-weight: 500;
}

.nav-item:hover {
  background: var(--bg-secondary);
  color: var(--text-primary);
}

.nav-item.active {
  background: var(--accent-focus);
  color: var(--accent-primary);
  font-weight: 600;
}

.nav-icon {
  font-size: 20px;
  min-width: 24px;
  display: flex;
  align-items: center;
}

.nav-label {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  padding-top: 1rem;
  border-top: 1px solid var(--border-light);
}

.footer-btn {
  background: none;
  border: none;
}

.sidebar-close {
  display: none;
  position: absolute;
  top: 1.5rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: var(--text-secondary);
  z-index: 10;
}

/* === MAIN WRAPPER === */
.main-wrapper {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* === TOPBAR === */
.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 70px;
  padding: 0 1.5rem;
  background: var(--bg-primary);
  border-bottom: 1px solid var(--border-light);
  box-shadow: var(--shadow-sm);
  gap: 1.5rem;
}

.topbar-left {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex: 1;
  min-width: 0;
}

.hamburger {
  display: none;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: var(--text-primary);
  padding: 0.5rem;
}

.page-title {
  font-size: 20px;
  font-weight: 600;
  color: var(--text-primary);
  margin: 0;
  white-space: nowrap;
}

.topbar-right {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.icon-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 8px;
  background: var(--bg-secondary);
  border: none;
  cursor: pointer;
  font-size: 18px;
  transition: all 200ms ease;
  color: var(--text-primary);
}

.icon-btn:hover {
  background: var(--border-light);
  transform: scale(1.05);
}

/* === FILTER BAR === */
.filter-bar-wrapper {
  background: var(--bg-primary);
  border-bottom: 1px solid var(--border-light);
  padding: 0 1.5rem;
}

/* === CONTENT === */
.content {
  flex: 1;
  overflow: auto;
  padding: 1.5rem;
  background: var(--bg-secondary);
}

/* === MOBILE BACKDROP === */
.sidebar-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 40;
}

.backdrop-fade-enter-active,
.backdrop-fade-leave-active {
  transition: opacity 200ms ease;
}

.backdrop-fade-enter-from,
.backdrop-fade-leave-to {
  opacity: 0;
}

/* === MOBILE STYLES === */
@media (max-width: 1023px) {
  .app-container {
    grid-template-columns: 1fr;
  }

  .hamburger {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .sidebar {
    position: fixed;
    left: 0;
    top: 0;
    width: 260px;
    height: 100vh;
    z-index: 50;
    transform: translateX(-100%);
    transition: transform 300ms cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: var(--shadow-lg);
  }

  .sidebar-mobile {
    transform: translateX(0);
  }

  .sidebar-close {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .page-title {
    font-size: 16px;
  }

  .content {
    padding: 1rem;
  }
}

/* === SCROLLBAR === */
.sidebar::-webkit-scrollbar {
  width: 6px;
}

.sidebar::-webkit-scrollbar-track {
  background: transparent;
}

.sidebar::-webkit-scrollbar-thumb {
  background: var(--border-medium);
  border-radius: 3px;
}

.sidebar::-webkit-scrollbar-thumb:hover {
  background: var(--border-dark);
}
</style>
