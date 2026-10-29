---
description: Redesign the Vue 3 app UI into a modern SaaS-style interface with a vertical left sidebar navigation replacing the top nav bar
---

Redesign the application layout from a horizontal top nav bar to a modern SaaS-style vertical sidebar. The result should look like a polished B2B SaaS product (similar in feel to Linear, Vercel dashboard, or Stripe).

## Phase 1 — Explore

Before changing anything, read these files:
- `client/src/App.vue` — current layout, nav links, global CSS, all components used
- `client/src/main.js` — all route paths and their labels
- `client/src/components/FilterBar.vue` — understand the filter bar so you can reposition it correctly

## Phase 2 — Delegate to vue-expert

**MANDATORY**: Delegate ALL `.vue` file changes to the `vue-expert` subagent. Pass the full specification below in your prompt.

---

### Specification for vue-expert

Rewrite `client/src/App.vue` to implement a sidebar layout. Preserve all existing script logic, composable usage, component imports, and event handlers exactly — only the template structure and global CSS change.

#### New HTML structure

```
<div class="app-layout">
  <aside class="sidebar">
    <!-- Brand -->
    <div class="sidebar-brand">
      <div class="brand-mark"></div>
      <div class="brand-text">
        <span class="brand-name">{{ t('nav.companyName') }}</span>
        <span class="brand-sub">{{ t('nav.subtitle') }}</span>
      </div>
    </div>

    <!-- Nav -->
    <nav class="sidebar-nav">
      <router-link to="/"          class="nav-item" :class="{ active: $route.path === '/' }">
        <!-- SVG: grid/dashboard icon -->  Dashboard
      </router-link>
      <router-link to="/inventory" class="nav-item" :class="{ active: $route.path === '/inventory' }">
        <!-- SVG: cube/box icon -->  Inventory
      </router-link>
      <router-link to="/orders"    class="nav-item" :class="{ active: $route.path === '/orders' }">
        <!-- SVG: clipboard/list icon -->  Orders
      </router-link>
      <router-link to="/spending"  class="nav-item" :class="{ active: $route.path === '/spending' }">
        <!-- SVG: chart/bar icon -->  Finance
      </router-link>
      <router-link to="/demand"    class="nav-item" :class="{ active: $route.path === '/demand' }">
        <!-- SVG: trending-up arrow icon -->  Demand
      </router-link>
      <router-link to="/reports"   class="nav-item" :class="{ active: $route.path === '/reports' }">
        <!-- SVG: document/file icon -->  Reports
      </router-link>
      <router-link to="/restocking" class="nav-item" :class="{ active: $route.path === '/restocking' }">
        <!-- SVG: refresh/restock arrows icon -->  Restocking
      </router-link>
    </nav>

    <!-- Bottom: language + profile -->
    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu @show-profile-details="..." @show-tasks="..." />
    </div>
  </aside>

  <div class="main-wrapper">
    <div class="filter-bar-wrapper">
      <FilterBar />
    </div>
    <main class="main-content">
      <router-view />
    </main>
  </div>
</div>
```

#### SVG icons to use inline in each nav item (no external library needed)

Use simple 18×18 stroke-based SVGs (`stroke="currentColor" stroke-width="1.75" fill="none"`):

- **Dashboard**: `<rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/>` (viewBox="0 0 24 24")
- **Inventory**: `<path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/>` (viewBox="0 0 24 24")
- **Orders**: `<path d="M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2"/><rect x="9" y="3" width="6" height="4" rx="1" ry="1"/><line x1="9" y1="12" x2="15" y2="12"/><line x1="9" y1="16" x2="13" y2="16"/>` (viewBox="0 0 24 24")
- **Finance**: `<line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/>` (viewBox="0 0 24 24")
- **Demand**: `<polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/>` (viewBox="0 0 24 24")
- **Reports**: `<path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/>` (viewBox="0 0 24 24")
- **Restocking**: `<polyline points="1 4 1 10 7 10"/><path d="M3.51 15a9 9 0 1 0 .49-3.1"/>` (viewBox="0 0 24 24")

Each nav item renders as:
```vue
<router-link to="/" class="nav-item" :class="{ active: $route.path === '/' }">
  <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round">
    <!-- path data here -->
  </svg>
  <span>Dashboard</span>
</router-link>
```

#### Global CSS replacement (replace ALL of `<style>` in App.vue)

The layout tokens:
- Sidebar width: `240px`
- Sidebar bg: `#0f172a`
- Sidebar border: `1px solid #1e293b` on right edge
- Nav item active bg: `#1e3a5f`; active text: `#93c5fd`; active left border: `2px solid #3b82f6`
- Nav item hover bg: `#1e293b`
- Body bg: `#f1f5f9`
- Main content bg: `#f1f5f9`
- Brand mark: a 28px square with `background: #3b82f6; border-radius: 6px`

```css
/* === Reset === */
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: #f1f5f9;
  color: #1e293b;
  -webkit-font-smoothing: antialiased;
}

/* === Layout shell === */
.app-layout {
  display: flex;
  min-height: 100vh;
}

/* === Sidebar === */
.sidebar {
  width: 240px;
  flex-shrink: 0;
  background: #0f172a;
  border-right: 1px solid #1e293b;
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  z-index: 100;
  overflow-y: auto;
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1.25rem 1rem;
  border-bottom: 1px solid #1e293b;
  flex-shrink: 0;
}

.brand-mark {
  width: 28px;
  height: 28px;
  background: #3b82f6;
  border-radius: 6px;
  flex-shrink: 0;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.brand-name {
  font-size: 0.875rem;
  font-weight: 700;
  color: #f1f5f9;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  letter-spacing: -0.01em;
}

.brand-sub {
  font-size: 0.7rem;
  color: #475569;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-top: 1px;
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0.5rem;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  color: #94a3b8;
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  transition: background 0.15s, color 0.15s;
  border-left: 2px solid transparent;
}

.nav-item:hover {
  background: #1e293b;
  color: #e2e8f0;
}

.nav-item.active {
  background: #1e3a5f;
  color: #93c5fd;
  border-left-color: #3b82f6;
}

.nav-icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

.sidebar-footer {
  padding: 0.75rem 0.5rem;
  border-top: 1px solid #1e293b;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  flex-shrink: 0;
}

/* === Main area (offset by sidebar width) === */
.main-wrapper {
  margin-left: 240px;
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.filter-bar-wrapper {
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  position: sticky;
  top: 0;
  z-index: 50;
}

.main-content {
  flex: 1;
  padding: 1.75rem 2rem;
  max-width: 1400px;
  width: 100%;
}

/* === Global component styles (keep these — views rely on them) === */
.page-header { margin-bottom: 1.5rem; }
.page-header h2 { font-size: 1.75rem; font-weight: 700; color: #0f172a; margin-bottom: 0.25rem; letter-spacing: -0.025em; }
.page-header p { color: #64748b; font-size: 0.9rem; }

.stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; margin-bottom: 1.5rem; }

.stat-card { background: white; padding: 1.25rem; border-radius: 10px; border: 1px solid #e2e8f0; transition: box-shadow 0.2s; }
.stat-card:hover { box-shadow: 0 4px 12px rgba(0,0,0,0.06); }
.stat-label { color: #64748b; font-size: 0.75rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 0.5rem; }
.stat-value { font-size: 2rem; font-weight: 700; color: #0f172a; letter-spacing: -0.025em; }
.stat-card.success .stat-value { color: #059669; }
.stat-card.warning .stat-value { color: #ea580c; }
.stat-card.danger  .stat-value { color: #dc2626; }
.stat-card.info    .stat-value { color: #2563eb; }

.card { background: white; border-radius: 10px; padding: 1.25rem; border: 1px solid #e2e8f0; margin-bottom: 1.25rem; }
.card-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; padding-bottom: 0.75rem; border-bottom: 1px solid #f1f5f9; }
.card-title { font-size: 1rem; font-weight: 700; color: #0f172a; }

.table-container { overflow-x: auto; }
table { width: 100%; border-collapse: collapse; }
thead { background: #f8fafc; border-top: 1px solid #e2e8f0; border-bottom: 1px solid #e2e8f0; }
th { text-align: left; padding: 0.5rem 0.75rem; font-weight: 600; color: #475569; font-size: 0.72rem; text-transform: uppercase; letter-spacing: 0.05em; }
td { padding: 0.5rem 0.75rem; border-top: 1px solid #f1f5f9; color: #334155; font-size: 0.875rem; }
tbody tr:hover { background: #f8fafc; }

.badge { display: inline-block; padding: 0.25rem 0.625rem; border-radius: 5px; font-size: 0.72rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.025em; }
.badge.success   { background: #d1fae5; color: #065f46; }
.badge.warning   { background: #fed7aa; color: #92400e; }
.badge.danger    { background: #fecaca; color: #991b1b; }
.badge.info      { background: #dbeafe; color: #1e40af; }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #fecaca; color: #991b1b; }
.badge.stable    { background: #e0e7ff; color: #3730a3; }
.badge.high      { background: #fecaca; color: #991b1b; }
.badge.medium    { background: #fed7aa; color: #92400e; }
.badge.low       { background: #dbeafe; color: #1e40af; }

.loading { text-align: center; padding: 3rem; color: #64748b; }
.error { background: #fef2f2; border: 1px solid #fecaca; color: #991b1b; padding: 1rem; border-radius: 8px; margin: 1rem 0; font-size: 0.875rem; }
```

#### Constraints
- Do NOT change any `<script>` logic — only template structure and `<style>`
- Keep all `@show-profile-details`, `@show-tasks`, modal components, and task handlers exactly as-is
- No emojis anywhere in the UI
- Add non-obvious logic comments only where placement decisions are non-obvious (e.g. why FilterBar is in `.filter-bar-wrapper`)

---

## Phase 3 — Verify

After the vue-expert agent completes, open `http://localhost:3000` in the browser and confirm:
1. Sidebar appears on the left with all 7 nav items and SVG icons
2. Clicking each nav item navigates correctly and shows active state
3. FilterBar appears in a sticky strip below the sidebar brand (in the main area header)
4. Main content area scrolls independently
5. ProfileMenu and LanguageSwitcher are accessible at the bottom of the sidebar
6. No layout breakage on the existing pages (Dashboard, Inventory, Orders, etc.)
