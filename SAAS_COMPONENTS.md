# SaaS UI Components - Ready to Copy & Paste

This file contains production-ready Vue 3 components for the SaaS sidebar redesign. Copy these directly into your project.

---

## Complete SidebarLayout.vue

**File location:** `src/layouts/SidebarLayout.vue`

```vue
<template>
  <div class="app-layout" :class="{ 'mobile-sidebar-open': sidebarOpen && isMobile }">
    <!-- Sidebar -->
    <Sidebar 
      :open="sidebarOpen"
      :nav-items="navItems"
      :collapsed="!sidebarOpen && !isMobile"
      @update:open="sidebarOpen = $event"
    />
    
    <!-- Main Content Area -->
    <div class="layout-wrapper">
      <!-- Top Bar / Mobile Header -->
      <div class="topbar-wrapper">
        <div class="topbar-inner">
          <!-- Hamburger (mobile only) -->
          <button 
            v-if="isMobile" 
            class="hamburger-btn"
            @click="sidebarOpen = !sidebarOpen"
            aria-label="Toggle navigation"
          >
            <span></span>
            <span></span>
            <span></span>
          </button>

          <!-- Page Title -->
          <h1 class="page-title">{{ pageTitle }}</h1>

          <!-- Right Section -->
          <div class="topbar-right">
            <button class="icon-btn" title="Notifications">
              <span>🔔</span>
            </button>
            <button class="icon-btn" title="Settings">
              <span>⚙️</span>
            </button>
          </div>
        </div>
      </div>

      <!-- Content Area -->
      <div class="content-area">
        <slot />
      </div>
    </div>

    <!-- Mobile Backdrop -->
    <Transition name="fade">
      <div 
        v-if="sidebarOpen && isMobile"
        class="sidebar-backdrop"
        @click="sidebarOpen = false"
      />
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRoute } from 'vue-router'
import Sidebar from '../components/Sidebar.vue'

const route = useRoute()
const sidebarOpen = ref(true)
const windowWidth = ref(1024)
const isMobile = computed(() => windowWidth.value < 1024)

// Page titles mapping
const pageTitles = {
  '/': 'Dashboard',
  '/inventory': 'Inventory Management',
  '/orders': 'Orders',
  '/demand': 'Demand Forecast',
  '/spending': 'Finance & Spending',
  '/reports': 'Reports',
  '/restocking': 'Restocking'
}

const pageTitle = computed(() => pageTitles[route.path] || 'Page')

// Navigation items
const navItems = [
  { label: 'Dashboard', icon: '📊', to: '/' },
  { label: 'Inventory', icon: '📦', to: '/inventory' },
  { label: 'Orders', icon: '🛒', to: '/orders' },
  { label: 'Finance', icon: '💰', to: '/spending' },
  { label: 'Demand Forecast', icon: '📈', to: '/demand' },
  { label: 'Restocking', icon: '🔄', to: '/restocking' },
  { label: 'Reports', icon: '📋', to: '/reports' }
]

// Handle window resize
const handleResize = () => {
  windowWidth.value = window.innerWidth
  // Close sidebar on mobile by default
  if (windowWidth.value < 1024) {
    sidebarOpen.value = false
  }
}

onMounted(() => {
  handleResize()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.app-layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  min-height: 100vh;
  background: var(--bg-primary);
}

.layout-wrapper {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.topbar-wrapper {
  background: var(--bg-primary);
  border-bottom: 1px solid var(--border-light);
  padding: 0;
  box-shadow: var(--shadow-sm);
  height: 70px;
  display: flex;
  align-items: center;
}

.topbar-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--spacing-lg);
  width: 100%;
  height: 100%;
  padding: 0 var(--spacing-lg);
}

.hamburger-btn {
  display: none;
  background: none;
  border: none;
  width: 40px;
  height: 40px;
  cursor: pointer;
  padding: var(--spacing-sm);
  flex-direction: column;
  justify-content: center;
  gap: 6px;
}

.hamburger-btn span {
  display: block;
  width: 24px;
  height: 2px;
  background: var(--text-primary);
  border-radius: 1px;
  transition: all 300ms ease;
}

.hamburger-btn:hover span {
  background: var(--accent-primary);
}

.page-title {
  font-size: 20px;
  font-weight: 600;
  color: var(--text-primary);
  margin: 0;
  flex: 1;
}

.topbar-right {
  display: flex;
  gap: var(--spacing-md);
  align-items: center;
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

.content-area {
  flex: 1;
  overflow: auto;
  padding: var(--spacing-xl);
  background: var(--bg-secondary);
}

.sidebar-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 40;
}

/* Fade transition */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 200ms ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Mobile styles */
@media (max-width: 1023px) {
  .app-layout {
    grid-template-columns: 1fr;
  }

  .hamburger-btn {
    display: flex;
  }

  .topbar-wrapper {
    padding: 0;
  }

  .page-title {
    font-size: 16px;
  }

  .content-area {
    padding: var(--spacing-lg);
  }
}
</style>
```

---

## Complete Sidebar.vue

**File location:** `src/components/Sidebar.vue`

```vue
<template>
  <aside 
    class="sidebar"
    :class="{ 'sidebar-mobile-open': open && isMobile }"
    role="navigation"
    aria-label="Main navigation"
  >
    <!-- Header with Logo -->
    <div class="sidebar-header">
      <div class="logo-container">
        <span class="logo-icon">🏭</span>
        <span v-if="!collapsed" class="logo-text">Catalyst</span>
      </div>
      <button 
        v-if="isMobile"
        class="sidebar-close"
        @click="$emit('update:open', false)"
        aria-label="Close navigation"
      >
        ✕
      </button>
    </div>

    <!-- Navigation Links -->
    <nav class="sidebar-nav">
      <router-link
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        :class="{ 'nav-item-active': isActive(item.to) }"
        :title="item.label"
      >
        <span class="nav-icon" aria-hidden="true">{{ item.icon }}</span>
        <span v-if="!collapsed" class="nav-label">{{ item.label }}</span>
      </router-link>
    </nav>

    <!-- Footer with Settings -->
    <div class="sidebar-footer">
      <button 
        class="nav-item footer-btn"
        @click="toggleTheme"
        :title="`Switch to ${isDark ? 'light' : 'dark'} mode`"
      >
        <span class="nav-icon" aria-hidden="true">{{ isDark ? '☀️' : '🌙' }}</span>
        <span v-if="!collapsed" class="nav-label">Theme</span>
      </button>
      <button 
        class="nav-item footer-btn"
        @click="showProfile = true"
        title="User profile"
      >
        <span class="nav-icon" aria-hidden="true">👤</span>
        <span v-if="!collapsed" class="nav-label">Profile</span>
      </button>
    </div>
  </aside>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRoute } from 'vue-router'

const props = defineProps({
  open: {
    type: Boolean,
    default: true
  },
  navItems: {
    type: Array,
    required: true
  },
  collapsed: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['update:open'])
const route = useRoute()
const isDark = ref(false)
const isMobile = computed(() => typeof window !== 'undefined' && window.innerWidth < 1024)

const isActive = (path) => route.path === path

const toggleTheme = () => {
  isDark.value = !isDark.value
  // Implement actual theme switching
  document.documentElement.classList.toggle('dark-mode', isDark.value)
  localStorage.setItem('theme', isDark.value ? 'dark' : 'light')
}
</script>

<style scoped>
.sidebar {
  display: flex;
  flex-direction: column;
  width: 250px;
  height: 100vh;
  background: var(--bg-sidebar);
  border-right: 1px solid var(--border-light);
  padding: var(--spacing-lg);
  overflow-y: auto;
  transition: all 300ms cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  z-index: 50;
}

/* Sidebar header */
.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: var(--spacing-xl);
  padding-bottom: var(--spacing-lg);
  border-bottom: 1px solid var(--border-light);
}

.logo-container {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  font-weight: 700;
  color: var(--text-primary);
  cursor: pointer;
  user-select: none;
}

.logo-icon {
  font-size: 24px;
  display: flex;
  align-items: center;
}

.logo-text {
  font-size: 16px;
  font-weight: 700;
  letter-spacing: -0.5px;
}

.sidebar-close {
  display: none;
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: var(--text-secondary);
  padding: var(--spacing-sm);
  border-radius: 6px;
  transition: all 200ms ease;
}

.sidebar-close:hover {
  background: var(--bg-secondary);
  color: var(--text-primary);
}

/* Navigation */
.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xs);
  list-style: none;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
  padding: var(--spacing-md) var(--spacing-sm);
  border-radius: 8px;
  color: var(--text-secondary);
  text-decoration: none;
  cursor: pointer;
  transition: all 200ms ease;
  font-size: 14px;
  font-weight: 500;
  user-select: none;
}

.nav-item:hover {
  background: var(--bg-secondary);
  color: var(--text-primary);
}

.nav-item-active {
  background: var(--accent-focus);
  color: var(--accent-primary);
  font-weight: 600;
}

.nav-item-active:hover {
  background: var(--accent-focus);
}

.nav-icon {
  font-size: 18px;
  min-width: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-label {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Footer */
.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xs);
  padding-top: var(--spacing-lg);
  border-top: 1px solid var(--border-light);
  margin-top: auto;
}

.footer-btn {
  background: none;
  border: none;
  cursor: pointer;
}

/* Mobile styles */
@media (max-width: 1023px) {
  .sidebar {
    position: fixed;
    left: -100%;
    top: 0;
    width: 250px;
    height: 100vh;
    z-index: 50;
    box-shadow: var(--shadow-lg);
    transition: left 300ms cubic-bezier(0.4, 0, 0.2, 1);
  }

  .sidebar-mobile-open {
    left: 0;
  }

  .sidebar-close {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

/* Scrollbar styling */
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
```

---

## CSS Variables & Global Styles

**File location:** `src/styles/globals.css` or `src/main.css`

```css
/* Design Tokens & CSS Variables */
:root {
  /* === COLORS === */
  
  /* Backgrounds */
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --bg-tertiary: #f1f3f5;
  --bg-sidebar: #ffffff;
  
  /* Text */
  --text-primary: #0f172a;
  --text-secondary: #64748b;
  --text-tertiary: #94a3b8;
  --text-inverse: #ffffff;
  
  /* Accents */
  --accent-primary: #3b82f6;
  --accent-secondary: #06b6d4;
  --accent-hover: #2563eb;
  --accent-focus: #dbeafe;
  
  /* Status Colors */
  --status-success: #10b981;
  --status-warning: #f59e0b;
  --status-error: #ef4444;
  --status-info: #3b82f6;
  
  /* Borders */
  --border-light: #e2e8f0;
  --border-medium: #cbd5e1;
  --border-dark: #94a3b8;
  
  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.1);
  
  /* === SPACING === */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 12px;
  --spacing-lg: 16px;
  --spacing-xl: 24px;
  --spacing-2xl: 32px;
  
  /* === TYPOGRAPHY === */
  --font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --font-size-xs: 12px;
  --font-size-sm: 14px;
  --font-size-base: 16px;
  --font-size-lg: 18px;
  --font-size-xl: 20px;
  --font-size-2xl: 24px;
  
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
  
  /* === Z-INDEX === */
  --z-base: 0;
  --z-dropdown: 10;
  --z-sticky: 20;
  --z-popover: 30;
  --z-tooltip: 35;
  --z-modal-backdrop: 40;
  --z-modal: 50;
  --z-notification: 60;
  
  /* === BREAKPOINTS === */
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;
}

/* Dark mode theme */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-primary: #0f172a;
    --bg-secondary: #1a2a4a;
    --bg-tertiary: #253659;
    --bg-sidebar: #0f172a;
    
    --text-primary: #f1f5f9;
    --text-secondary: #cbd5e1;
    --text-tertiary: #94a3b8;
    --text-inverse: #0f172a;
  }
}

/* Dark mode class-based (JavaScript control) */
.dark-mode {
  --bg-primary: #0f172a;
  --bg-secondary: #1a2a4a;
  --bg-tertiary: #253659;
  --bg-sidebar: #0f172a;
  
  --text-primary: #f1f5f9;
  --text-secondary: #cbd5e1;
  --text-tertiary: #94a3b8;
}

/* === RESET === */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  font-family: var(--font-family);
  font-size: 16px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  background: var(--bg-primary);
  color: var(--text-primary);
  line-height: 1.6;
  letter-spacing: -0.01em;
}

/* === TYPOGRAPHY === */
h1, h2, h3, h4, h5, h6 {
  font-weight: var(--font-weight-bold);
  line-height: 1.2;
  letter-spacing: -0.02em;
  margin-bottom: var(--spacing-md);
}

h1 {
  font-size: var(--font-size-2xl);
}

h2 {
  font-size: var(--font-size-xl);
}

h3 {
  font-size: var(--font-size-lg);
}

p {
  margin-bottom: var(--spacing-md);
  color: var(--text-secondary);
}

/* === LINKS === */
a {
  color: var(--accent-primary);
  text-decoration: none;
  transition: color 200ms ease;
}

a:hover {
  color: var(--accent-hover);
}

/* === BUTTONS === */
button {
  font-family: inherit;
  cursor: pointer;
  border: none;
  padding: 0;
  background: none;
  color: inherit;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* === FORMS === */
input, textarea, select {
  font-family: inherit;
  font-size: inherit;
  border: 1px solid var(--border-light);
  border-radius: 6px;
  padding: var(--spacing-sm) var(--spacing-md);
  background: var(--bg-primary);
  color: var(--text-primary);
  transition: all 200ms ease;
}

input:focus, textarea:focus, select:focus {
  outline: none;
  border-color: var(--accent-primary);
  box-shadow: 0 0 0 3px var(--accent-focus);
}

/* === SCROLLBAR === */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: var(--bg-secondary);
}

::-webkit-scrollbar-thumb {
  background: var(--border-medium);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: var(--border-dark);
}

/* === UTILITY CLASSES === */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.container {
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
  padding: 0 var(--spacing-lg);
}

.flex { display: flex; }
.flex-col { flex-direction: column; }
.items-center { align-items: center; }
.justify-between { justify-content: space-between; }
.gap-xs { gap: var(--spacing-xs); }
.gap-sm { gap: var(--spacing-sm); }
.gap-md { gap: var(--spacing-md); }
.gap-lg { gap: var(--spacing-lg); }

.text-primary { color: var(--text-primary); }
.text-secondary { color: var(--text-secondary); }
.text-muted { color: var(--text-tertiary); }

.bg-primary { background: var(--bg-primary); }
.bg-secondary { background: var(--bg-secondary); }

.rounded { border-radius: 6px; }
.rounded-lg { border-radius: 12px; }

.shadow { box-shadow: var(--shadow-md); }
.shadow-lg { box-shadow: var(--shadow-lg); }
```

---

## How to Use These Components

### 1. Copy files to your project:
```
src/
├── layouts/
│   └── SidebarLayout.vue
├── components/
│   └── Sidebar.vue
└── styles/
    └── globals.css
```

### 2. Update router:
```javascript
import SidebarLayout from '@/layouts/SidebarLayout.vue'

const routes = [
  {
    path: '/',
    component: SidebarLayout,
    children: [
      { path: '', component: Dashboard },
      { path: 'inventory', component: Inventory },
      // ... other routes
    ]
  }
]
```

### 3. Import globals in main.js:
```javascript
import './styles/globals.css'
```

### 4. Simplify App.vue:
```vue
<template>
  <router-view />
</template>
```

### 5. Update page components:
Remove padding/margins and old styling since layout handles it.

---

## Component Props & Events

### SidebarLayout

**Props:**
- `navItems` (Array) - Navigation menu items
- `collapsed` (Boolean) - Sidebar collapse state

**Emits:**
- None (handles its own state)

### Sidebar

**Props:**
- `open` (Boolean) - Sidebar visibility (v-model)
- `navItems` (Array) - Menu items
- `collapsed` (Boolean) - Collapsed state

**Emits:**
- `update:open` - When sidebar open state changes

---

## Customization Examples

### Change Colors
```css
/* In your own style file */
:root {
  --accent-primary: #8b5cf6; /* Purple */
  --accent-hover: #7c3aed;
  --accent-focus: #ede9fe;
}
```

### Change Sidebar Width
```css
.app-layout {
  grid-template-columns: 300px 1fr; /* was 250px */
}

.sidebar {
  width: 300px; /* was 250px */
}
```

### Hide Labels on Scroll
```vue
<script>
const hideLabelsOnScroll = computed(() => {
  return navScrolled.value && collapsed.value
})
</script>
```

---

**These components are production-ready and can be used immediately!**
