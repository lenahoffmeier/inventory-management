# SaaS UI Redesign Skill - Vue 3 Left Sidebar Navigation

## Overview

This skill provides a reusable, production-ready approach to transforming Vue 3 applications from traditional top navigation bars to modern SaaS-style interfaces with:

- ✨ **Left Vertical Sidebar** - Primary navigation with collapsible functionality
- 📐 **Consistent Spacing System** - 4px/8px grid-based layout
- 🎨 **Professional Design** - Modern color palette and typography
- 📱 **Responsive Design** - Mobile-friendly with adaptive sidebar
- ⚡ **Smooth Interactions** - Transitions and visual feedback
- 🎯 **Clear Hierarchy** - Visual organization and focus

---

## Part 1: Architecture Overview

### Layout Structure

```
┌─────────────────────────────────────────┐
│  Desktop (>1024px)                      │
├──────────┬──────────────────────────────┤
│          │                              │
│ Sidebar  │    Main Content Area        │
│ (250px)  │                              │
│          │                              │
└──────────┴──────────────────────────────┘

┌──────────────────────────┐
│  Mobile (<1024px)        │
├──────────────────────────┤
│  Hamburger + Header      │
├──────────────────────────┤
│  Main Content Area       │
│  (Sidebar as overlay)    │
└──────────────────────────┘
```

### Component Hierarchy

```
App.vue
├── SidebarLayout.vue
│   ├── Sidebar.vue
│   │   ├── SidebarHeader.vue (Logo/Brand)
│   │   ├── NavLinks.vue (Menu items)
│   │   └── SidebarFooter.vue (User profile/settings)
│   ├── TopBar.vue (Mobile header)
│   └── MainContent.vue (router-view)
```

---

## Part 2: Design Tokens & Spacing System

### Color Palette

```javascript
// light mode (preferred default)
const colors = {
  // Backgrounds
  bg: {
    primary: '#ffffff',      // Main background
    secondary: '#f8f9fa',    // Hover/inactive states
    tertiary: '#f1f3f5',     // Disabled
    sidebar: '#ffffff'       // Sidebar background
  },
  
  // Text
  text: {
    primary: '#0f172a',      // Headlines, body text
    secondary: '#64748b',    // Muted labels
    tertiary: '#94a3b8',     // Disabled text
    inverse: '#ffffff'       // On dark backgrounds
  },
  
  // Accents
  accent: {
    primary: '#3b82f6',      // Primary action/active state
    secondary: '#06b6d4',    // Secondary actions
    hover: '#2563eb',        // Primary hover
    focus: '#dbeafe'         // Focus ring
  },
  
  // Status
  status: {
    success: '#10b981',
    warning: '#f59e0b',
    error: '#ef4444',
    info: '#3b82f6'
  },
  
  // Borders
  border: {
    light: '#e2e8f0',
    medium: '#cbd5e1',
    dark: '#94a3b8'
  },
  
  // Shadows
  shadow: {
    sm: '0 1px 2px rgba(0, 0, 0, 0.05)',
    md: '0 4px 6px rgba(0, 0, 0, 0.07)',
    lg: '0 10px 15px rgba(0, 0, 0, 0.1)'
  }
}

// dark mode (optional)
// Similar structure with inverted values
```

### Spacing Scale (4px base unit)

```javascript
const spacing = {
  xs: '4px',   // 1 unit
  sm: '8px',   // 2 units
  md: '12px',  // 3 units
  lg: '16px',  // 4 units
  xl: '24px',  // 6 units
  '2xl': '32px' // 8 units
}

// Usage: margin-left: var(--spacing-lg) = 16px
```

### Typography

```javascript
const typography = {
  heading: {
    h1: { size: '32px', weight: 700, lineHeight: 1.2 },
    h2: { size: '24px', weight: 700, lineHeight: 1.3 },
    h3: { size: '20px', weight: 600, lineHeight: 1.4 },
    h4: { size: '16px', weight: 600, lineHeight: 1.4 }
  },
  body: {
    lg: { size: '16px', weight: 400, lineHeight: 1.6 },
    base: { size: '14px', weight: 400, lineHeight: 1.6 },
    sm: { size: '12px', weight: 400, lineHeight: 1.5 }
  },
  family: "'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
}
```

---

## Part 3: Reusable Components

### 3.1 SidebarLayout.vue (Main wrapper)

```vue
<template>
  <div class="sidebar-layout" :class="{ 'sidebar-collapsed': !sidebarOpen }">
    <Sidebar 
      v-model:open="sidebarOpen"
      :nav-items="navItems"
    />
    
    <div class="layout-main">
      <TopBar 
        :show-hamburger="!sidebarOpen"
        @toggle-sidebar="sidebarOpen = !sidebarOpen"
      />
      <main class="layout-content">
        <router-view />
      </main>
    </div>
    
    <!-- Mobile overlay backdrop -->
    <Transition name="backdrop">
      <div 
        v-if="sidebarOpen && isMobile"
        class="sidebar-backdrop"
        @click="sidebarOpen = false"
      />
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import Sidebar from './Sidebar.vue'
import TopBar from './TopBar.vue'

const sidebarOpen = ref(true)
const windowWidth = ref(typeof window !== 'undefined' ? window.innerWidth : 1024)

const isMobile = computed(() => windowWidth.value < 1024)

// Close sidebar on mobile by default
onMounted(() => {
  const handleResize = () => {
    windowWidth.value = window.innerWidth
    if (windowWidth.value < 1024) {
      sidebarOpen.value = false
    }
  }
  
  window.addEventListener('resize', handleResize)
  handleResize()
})

const navItems = [
  { label: 'Dashboard', icon: 'dashboard', to: '/' },
  { label: 'Inventory', icon: 'inventory', to: '/inventory' },
  { label: 'Orders', icon: 'orders', to: '/orders' },
  { label: 'Finance', icon: 'finance', to: '/spending' },
  { label: 'Demand', icon: 'forecast', to: '/demand' },
  { label: 'Reports', icon: 'reports', to: '/reports' }
]
</script>

<style scoped>
.sidebar-layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  min-height: 100vh;
  background: var(--bg-primary);
}

.layout-main {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.layout-content {
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

/* Transitions */
.backdrop-enter-active,
.backdrop-leave-active {
  transition: opacity 200ms ease;
}

.backdrop-enter-from,
.backdrop-leave-to {
  opacity: 0;
}

/* Mobile responsiveness */
@media (max-width: 1023px) {
  .sidebar-layout {
    grid-template-columns: 1fr;
  }
  
  .sidebar-collapsed :deep(.sidebar) {
    position: fixed;
    left: -100%;
    top: 0;
    width: 250px;
    height: 100vh;
    z-index: 50;
    transition: left 300ms ease;
  }
  
  .sidebar-layout:not(.sidebar-collapsed) :deep(.sidebar) {
    left: 0;
  }
  
  .layout-content {
    padding: var(--spacing-lg);
  }
}
</style>
```

### 3.2 Sidebar.vue

```vue
<template>
  <aside class="sidebar" :class="{ 'sidebar-mobile-open': open && isMobile }">
    <!-- Header -->
    <div class="sidebar-header">
      <div class="sidebar-logo">
        <span class="logo-icon">🏭</span>
        <span v-if="!collapsed" class="logo-text">Catalyst</span>
      </div>
    </div>

    <!-- Navigation -->
    <nav class="sidebar-nav">
      <router-link
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-link"
        :class="{ active: $route.path === item.to }"
      >
        <span class="nav-icon">{{ getIcon(item.icon) }}</span>
        <span v-if="!collapsed" class="nav-label">{{ item.label }}</span>
      </router-link>
    </nav>

    <!-- Footer -->
    <div class="sidebar-footer">
      <button class="sidebar-footer-button" @click="toggleTheme">
        <span class="footer-icon">🌙</span>
        <span v-if="!collapsed" class="footer-label">Theme</span>
      </button>
      <button class="sidebar-footer-button" @click="showProfile = true">
        <span class="footer-icon">👤</span>
        <span v-if="!collapsed" class="footer-label">Profile</span>
      </button>
    </div>

    <!-- Close button for mobile -->
    <button 
      v-if="isMobile" 
      class="sidebar-close"
      @click="$emit('update:open', false)"
    >
      ✕
    </button>
  </aside>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'

const props = defineProps({
  open: Boolean,
  navItems: Array,
  collapsed: { type: Boolean, default: false }
})

const emit = defineEmits(['update:open'])
const route = useRoute()

const isMobile = computed(() => window.innerWidth < 1024)

const getIcon = (icon) => {
  const icons = {
    dashboard: '📊',
    inventory: '📦',
    orders: '🛒',
    finance: '💰',
    forecast: '📈',
    reports: '📋'
  }
  return icons[icon] || '▸'
}

const toggleTheme = () => {
  // Implement theme toggle
}
</script>

<style scoped>
.sidebar {
  display: flex;
  flex-direction: column;
  background: var(--bg-sidebar);
  border-right: 1px solid var(--border-light);
  padding: var(--spacing-lg);
  overflow-y: auto;
  transition: all 300ms ease;
}

.sidebar-header {
  margin-bottom: var(--spacing-xl);
  padding-bottom: var(--spacing-lg);
  border-bottom: 1px solid var(--border-light);
}

.sidebar-logo {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  font-size: 18px;
  font-weight: 700;
  color: var(--text-primary);
}

.logo-text {
  font-size: 16px;
  font-weight: 700;
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xs);
}

.nav-link {
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
  padding: var(--spacing-md) var(--spacing-sm);
  border-radius: 8px;
  color: var(--text-secondary);
  text-decoration: none;
  transition: all 200ms ease;
  cursor: pointer;
}

.nav-link:hover {
  background: var(--bg-secondary);
  color: var(--text-primary);
}

.nav-link.active {
  background: var(--accent-focus);
  color: var(--accent-primary);
  font-weight: 500;
}

.nav-icon {
  font-size: 18px;
  min-width: 24px;
  display: flex;
  align-items: center;
}

.nav-label {
  font-size: 14px;
  white-space: nowrap;
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-xs);
  padding-top: var(--spacing-lg);
  border-top: 1px solid var(--border-light);
}

.sidebar-footer-button {
  display: flex;
  align-items: center;
  gap: var(--spacing-md);
  padding: var(--spacing-md) var(--spacing-sm);
  background: none;
  border: none;
  border-radius: 8px;
  color: var(--text-secondary);
  cursor: pointer;
  transition: all 200ms ease;
}

.sidebar-footer-button:hover {
  background: var(--bg-secondary);
  color: var(--text-primary);
}

.sidebar-close {
  display: none;
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
    border-radius: 0;
    padding: var(--spacing-lg);
    box-shadow: var(--shadow-lg);
  }

  .sidebar-mobile-open {
    left: 0;
  }

  .sidebar-close {
    display: flex;
    align-items: center;
    justify-content: center;
    position: absolute;
    top: var(--spacing-lg);
    right: var(--spacing-lg);
    width: 32px;
    height: 32px;
    background: none;
    border: none;
    font-size: 20px;
    cursor: pointer;
    color: var(--text-secondary);
  }
}
```

### 3.3 TopBar.vue (Mobile header)

```vue
<template>
  <header class="topbar">
    <div class="topbar-content">
      <!-- Hamburger for mobile -->
      <button 
        v-if="showHamburger" 
        class="hamburger"
        @click="$emit('toggle-sidebar')"
      >
        ☰
      </button>

      <!-- Breadcrumb/Title -->
      <div class="topbar-title">
        <h1>{{ pageTitle }}</h1>
      </div>

      <!-- Right actions -->
      <div class="topbar-actions">
        <button class="icon-button">
          <span>🔔</span>
        </button>
        <button class="icon-button">
          <span>⚙️</span>
        </button>
      </div>
    </div>
  </header>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'

defineProps({
  showHamburger: Boolean
})

defineEmits(['toggle-sidebar'])

const route = useRoute()

const pageTitle = computed(() => {
  const titles = {
    '/': 'Dashboard',
    '/inventory': 'Inventory',
    '/orders': 'Orders',
    '/spending': 'Finance',
    '/demand': 'Demand Forecast',
    '/reports': 'Reports'
  }
  return titles[route.path] || 'Page'
})
</script>

<style scoped>
.topbar {
  background: var(--bg-primary);
  border-bottom: 1px solid var(--border-light);
  padding: var(--spacing-lg);
  height: 70px;
  display: flex;
  align-items: center;
  box-shadow: var(--shadow-sm);
}

.topbar-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  gap: var(--spacing-lg);
}

.hamburger {
  display: none;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: var(--text-primary);
  padding: var(--spacing-sm);
}

.topbar-title h1 {
  font-size: 20px;
  font-weight: 600;
  color: var(--text-primary);
  margin: 0;
}

.topbar-actions {
  display: flex;
  gap: var(--spacing-md);
}

.icon-button {
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
}

.icon-button:hover {
  background: var(--bg-tertiary);
}

@media (max-width: 1023px) {
  .hamburger {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}
</style>
```

---

## Part 4: CSS Global Styles

### Global Variables & Reset

```css
/* root/main.css */
:root {
  /* Colors */
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --bg-tertiary: #f1f3f5;
  --bg-sidebar: #ffffff;
  
  --text-primary: #0f172a;
  --text-secondary: #64748b;
  --text-tertiary: #94a3b8;
  --text-inverse: #ffffff;
  
  --accent-primary: #3b82f6;
  --accent-secondary: #06b6d4;
  --accent-hover: #2563eb;
  --accent-focus: #dbeafe;
  
  --status-success: #10b981;
  --status-warning: #f59e0b;
  --status-error: #ef4444;
  --status-info: #3b82f6;
  
  --border-light: #e2e8f0;
  --border-medium: #cbd5e1;
  --border-dark: #94a3b8;
  
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  
  /* Spacing */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 12px;
  --spacing-lg: 16px;
  --spacing-xl: 24px;
  --spacing-2xl: 32px;
  
  /* Typography */
  --font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  --font-size-sm: 12px;
  --font-size-base: 14px;
  --font-size-lg: 16px;
  
  /* Z-index scale */
  --z-dropdown: 10;
  --z-sticky: 20;
  --z-modal-backdrop: 40;
  --z-modal: 50;
  --z-popover: 30;
  --z-tooltip: 35;
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-primary: #1a1a2e;
    --bg-secondary: #16213e;
    --bg-tertiary: #0f3460;
    --bg-sidebar: #1a1a2e;
    
    --text-primary: #f8f9fa;
    --text-secondary: #cbd5e1;
    --text-tertiary: #94a3b8;
  }
}

/* Reset */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  font-family: var(--font-family);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  background: var(--bg-primary);
  color: var(--text-primary);
  line-height: 1.6;
}

/* Remove default link styles */
a {
  color: inherit;
  text-decoration: none;
}

button {
  font-family: inherit;
}

/* Scrollbar styling */
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
```

---

## Part 5: Step-by-Step Migration Guide

### Step 1: Backup & Prepare
```bash
# Create a new branch for UI redesign
git checkout -b feature/saas-ui-redesign
```

### Step 2: Create Layout Components
- Create `src/layouts/SidebarLayout.vue`
- Create `src/components/Sidebar.vue`
- Create `src/components/TopBar.vue`

### Step 3: Update Router
```javascript
// src/main.js
import SidebarLayout from './layouts/SidebarLayout.vue'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      component: SidebarLayout,
      children: [
        { path: '/', component: Dashboard },
        { path: '/inventory', component: Inventory },
        // ... other routes
      ]
    }
  ]
})
```

### Step 4: Update App.vue
```vue
<template>
  <router-view />
</template>

<script setup>
import { useI18n } from './composables/useI18n'

const { t } = useI18n()
</script>

<style>
/* Remove old App.vue styles */
</style>
```

### Step 5: Apply Global Styles
- Import CSS variables file in `main.js`
- Update any hardcoded colors to use CSS variables

### Step 6: Update Page Content
- Remove padding/margin from pages (handled by layout)
- Update page headers for new top bar
- Ensure content respects new sidebar layout

### Step 7: Test & Iterate
- Desktop: Sidebar, navigation, responsiveness
- Mobile: Hamburger menu, overlay, touch interactions
- Dark mode: CSS variable switching
- Accessibility: Keyboard navigation, ARIA labels

---

## Part 6: Migration Checklist

### Design System
- [ ] CSS variables defined
- [ ] Color palette established
- [ ] Spacing scale defined
- [ ] Typography system in place
- [ ] Shadow/elevation system ready

### Components
- [ ] SidebarLayout created
- [ ] Sidebar component complete
- [ ] TopBar component complete
- [ ] Mobile responsive tested
- [ ] Animations/transitions smooth

### Integration
- [ ] Router updated with layout wrapper
- [ ] App.vue simplified
- [ ] Old top nav removed
- [ ] Filter bar repositioned/updated
- [ ] Modals/overlays z-index updated

### Content Pages
- [ ] Dashboard updated
- [ ] Inventory page styled
- [ ] Orders page styled
- [ ] Finance page styled
- [ ] Demand page styled
- [ ] Reports page styled

### Polish
- [ ] Dark mode support
- [ ] Loading states
- [ ] Error states
- [ ] Focus states (accessibility)
- [ ] Print styles

### Testing
- [ ] Desktop (1920px, 1440px, 1024px)
- [ ] Tablet (768px)
- [ ] Mobile (375px, 414px)
- [ ] Touch interactions
- [ ] Keyboard navigation
- [ ] Screen reader testing

---

## Part 7: Advanced Patterns

### 7.1 Theme Toggle with Composable

```javascript
// composables/useTheme.js
import { ref, watch, computed } from 'vue'

export function useTheme() {
  const isDark = ref(localStorage.getItem('theme') === 'dark')
  
  const theme = computed(() => isDark.value ? 'dark' : 'light')
  
  watch(isDark, (newValue) => {
    localStorage.setItem('theme', newValue ? 'dark' : 'light')
    document.documentElement.classList.toggle('dark-mode', newValue)
  }, { immediate: true })
  
  const toggle = () => {
    isDark.value = !isDark.value
  }
  
  return { isDark, theme, toggle }
}
```

### 7.2 Responsive Sidebar Hook

```javascript
// composables/useSidebar.js
import { ref, computed, onMounted } from 'vue'

export function useSidebar() {
  const sidebarOpen = ref(true)
  const isMobile = ref(false)
  
  const collapsed = computed(() => {
    return isMobile.value ? !sidebarOpen.value : false
  })
  
  const updateViewport = () => {
    isMobile.value = window.innerWidth < 1024
    if (isMobile.value) {
      sidebarOpen.value = false
    }
  }
  
  onMounted(() => {
    updateViewport()
    window.addEventListener('resize', updateViewport)
  })
  
  return { sidebarOpen, isMobile, collapsed }
}
```

### 7.3 Breadcrumb Navigation

```vue
<template>
  <div class="breadcrumbs">
    <router-link v-for="crumb in breadcrumbs" :key="crumb.to" :to="crumb.to">
      {{ crumb.label }}
    </router-link>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

const breadcrumbs = computed(() => {
  const crumbs = [{ label: 'Home', to: '/' }]
  // Build breadcrumbs from route
  return crumbs
})
</script>

<style scoped>
.breadcrumbs {
  display: flex;
  gap: var(--spacing-sm);
  font-size: var(--font-size-sm);
  color: var(--text-secondary);
}

.breadcrumbs a {
  color: var(--accent-primary);
  cursor: pointer;
}

.breadcrumbs a:hover {
  text-decoration: underline;
}
</style>
```

---

## Part 8: Performance Optimization

### CSS-in-JS vs External CSS
- ✅ **External CSS** (recommended)
  - Better caching
  - Faster initial render
  - Easier to maintain
  - Better browser devtools support

### Component Lazy Loading
```javascript
const Dashboard = defineAsyncComponent(() =>
  import('./views/Dashboard.vue')
)
```

### Sidebar Animation Optimization
- Use CSS transforms for smooth animations
- Avoid layout thrashing
- Throttle resize handlers

---

## Part 9: Accessibility Checklist

### Keyboard Navigation
- [ ] Tab through sidebar links
- [ ] Escape key closes mobile sidebar
- [ ] Arrow keys navigate menu items
- [ ] Enter/Space to activate links

### Screen Readers
- [ ] Proper heading hierarchy
- [ ] ARIA labels on icon buttons
- [ ] Landmark regions defined
- [ ] Link purposes clear

### Visual
- [ ] Color contrast WCAG AA
- [ ] Focus indicators visible
- [ ] No seizure-inducing animations
- [ ] Responsive text sizing

```html
<!-- Example: Accessible sidebar -->
<nav class="sidebar" role="navigation" aria-label="Main">
  <a href="/" class="nav-link" aria-current="page">
    <span aria-hidden="true">📊</span>
    <span>Dashboard</span>
  </a>
</nav>
```

---

## Part 10: Example: Applying to Inventory Management App

### Before (Top Nav Structure)
```
┌─────────────────────────┐
│ Logo | Nav Links | User │
├─────────────────────────┤
│ Filter Bar              │
├─────────────────────────┤
│ Content                 │
└─────────────────────────┘
```

### After (SaaS Sidebar Structure)
```
┌─────────┬──────────────────┐
│         │ Icon Bar + Title │
│ Sidebar ├──────────────────┤
│         │ Filter Bar       │
│         ├──────────────────┤
│         │ Content Area     │
│         │                  │
└─────────┴──────────────────┘
```

### Migration Steps for Inventory App

**1. Create layout wrapper:**
```vue
<!-- src/layouts/AppLayout.vue -->
<template>
  <SidebarLayout :nav-items="navItems">
    <FilterBar />
    <router-view />
  </SidebarLayout>
</template>
```

**2. Update router:**
```javascript
{
  path: '/',
  component: AppLayout,
  children: [
    { path: '/', component: Dashboard },
    { path: '/inventory', component: Inventory },
    // ... rest
  ]
}
```

**3. Remove from App.vue:**
- Top navigation bar
- Old styling
- Old layout logic

**4. Test each page:**
- Dashboard: Responsive stats cards
- Inventory: Table adapts to sidebar width
- Orders: Lists work with new layout
- Filters: Still functional at top of content

---

## Summary

This skill provides a complete, production-ready system for transforming Vue 3 applications into modern SaaS-style interfaces. Key benefits:

✨ **Modern Design** - Professional, polished appearance
📐 **Consistent** - Design tokens ensure cohesion
📱 **Responsive** - Works seamlessly across devices
♿ **Accessible** - WCAG compliant
⚡ **Performant** - CSS-based, no JS overhead
🔄 **Reusable** - Apply to multiple projects

Use this as a reference guide or adapt the components to your specific needs!
