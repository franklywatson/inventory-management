# Collapsible Sidebar Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a manual toggle button to the sidebar footer that collapses the sidebar to a 56px icon-only rail, with state persisted in localStorage.

**Architecture:** A single `collapsed` ref in `App.vue`'s `setup()` drives everything. Toggling it flips a `.collapsed` class on `.app-shell`. All visual changes (width, label visibility, icon centering) are handled entirely in CSS scoped to `.app-shell.collapsed`. No other files are touched.

**Tech Stack:** Vue 3 Composition API, CSS transitions, localStorage

---

### Task 1: Add collapse state and SVG icons to `App.vue` script

**Files:**
- Modify: `client/src/App.vue` (the `setup()` function and its `return`)

- [ ] **Step 1: Add `collapsed` ref with localStorage persistence**

In `App.vue`, inside `setup()`, after the existing `const showTasks = ref(false)` line, add:

```js
// Persist sidebar collapsed state across page refreshes
const collapsed = ref(localStorage.getItem('sidebar-collapsed') === 'true')

const toggleSidebar = () => {
  collapsed.value = !collapsed.value
  localStorage.setItem('sidebar-collapsed', String(collapsed.value))
}
```

- [ ] **Step 2: Add collapse/expand SVG icon strings**

Still inside `setup()`, after `toggleSidebar`, add:

```js
// Double-chevron icons for the collapse toggle button
const collapseIcon = `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="15 18 9 12 15 6"/><polyline points="9 18 3 12 9 6"/></svg>`
const expandIcon   = `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"/><polyline points="15 18 21 12 15 6"/></svg>`
```

- [ ] **Step 3: Expose new values in the `return` statement**

Add `collapsed`, `toggleSidebar`, `collapseIcon`, and `expandIcon` to the existing `return` object:

```js
return {
  t,
  navItems,
  collapsed,       // ← add
  toggleSidebar,   // ← add
  collapseIcon,    // ← add
  expandIcon,      // ← add
  showProfileDetails,
  showTasks,
  tasks,
  addTask,
  deleteTask,
  toggleTask
}
```

- [ ] **Step 4: Commit**

```bash
cd client
git add src/App.vue
git commit -m "feat(sidebar): add collapsed ref, toggleSidebar, and icon strings"
```

---

### Task 2: Update the `App.vue` template

**Files:**
- Modify: `client/src/App.vue` (`<template>` block)

- [ ] **Step 1: Bind `.collapsed` class on `.app-shell`**

Find the root div and change it from:

```html
<div class="app-shell">
```

to:

```html
<div class="app-shell" :class="{ collapsed }">
```

- [ ] **Step 2: Add `title` attribute to each nav `router-link`**

Find the `router-link` inside `sidebar-nav` and add `:title="item.label"` so collapsed icon-only nav items show a browser tooltip on hover:

```html
<router-link
  v-for="item in navItems"
  :key="item.path"
  :to="item.path"
  :title="item.label"
  class="nav-item"
  :class="{ active: $route.path === item.path }"
>
  <span class="nav-icon" v-html="item.icon"></span>
  <span class="nav-label">{{ item.label }}</span>
</router-link>
```

- [ ] **Step 3: Add the collapse toggle button to `.sidebar-footer`**

Inside `.sidebar-footer`, add this button **above** the `<LanguageSwitcher />` line:

```html
<button
  class="sidebar-toggle"
  @click="toggleSidebar"
  :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
>
  <span class="toggle-icon" v-html="collapsed ? expandIcon : collapseIcon"></span>
  <span class="toggle-label">Collapse</span>
</button>
```

The full `.sidebar-footer` block should now look like:

```html
<div class="sidebar-footer">
  <button
    class="sidebar-toggle"
    @click="toggleSidebar"
    :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
  >
    <span class="toggle-icon" v-html="collapsed ? expandIcon : collapseIcon"></span>
    <span class="toggle-label">Collapse</span>
  </button>
  <LanguageSwitcher />
  <ProfileMenu
    @show-profile-details="showProfileDetails = true"
    @show-tasks="showTasks = true"
  />
</div>
```

- [ ] **Step 4: Commit**

```bash
git add src/App.vue
git commit -m "feat(sidebar): update template for collapsible sidebar"
```

---

### Task 3: Add CSS for collapsed state

**Files:**
- Modify: `client/src/App.vue` (`<style>` block — the global, non-scoped block)

- [ ] **Step 1: Add sidebar width transition**

Find the existing `.sidebar` rule and add a `transition` property to it:

```css
.sidebar {
  width: 240px;
  flex-shrink: 0;
  background: #0f172a;
  border-right: 1px solid #1e293b;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  transition: width 0.2s ease; /* smooth collapse animation */
}
```

- [ ] **Step 2: Add all collapsed-state CSS rules**

At the end of the `<style>` block, append the following block. Do not place it inside any existing rule — add it as a new section at the bottom:

```css
/* ── Collapsed sidebar state ─────────────────────────── */

/* Narrow the sidebar to icon-rail width */
.app-shell.collapsed .sidebar {
  width: 56px;
}

/* Brand: center the logo, hide the text */
.app-shell.collapsed .sidebar-brand {
  justify-content: center;
  padding: 20px 0 16px;
}
.app-shell.collapsed .sidebar-brand-text {
  display: none;
}

/* Nav items: center icon, hide label */
.app-shell.collapsed .nav-item {
  justify-content: center;
  padding: 9px 0;
}
.app-shell.collapsed .nav-label {
  display: none;
}

/* Toggle button: center icon, hide label */
.sidebar-toggle {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
  padding: 8px 12px;
  border: none;
  background: transparent;
  border-radius: 7px;
  color: #64748b;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s ease, color 0.15s ease;
  margin-bottom: 4px;
}
.sidebar-toggle:hover {
  background: #1e293b;
  color: #cbd5e1;
}
.toggle-icon {
  display: flex;
  align-items: center;
  flex-shrink: 0;
}
.app-shell.collapsed .sidebar-toggle {
  justify-content: center;
  padding: 8px 0;
}
.app-shell.collapsed .toggle-label {
  display: none;
}

/* Hide LanguageSwitcher in collapsed mode (too wide for 56px) */
.app-shell.collapsed .sidebar-footer .language-switcher {
  display: none;
}

/* ProfileMenu: show avatar only, hide name and chevron */
.app-shell.collapsed .sidebar-footer .profile-name,
.app-shell.collapsed .sidebar-footer .chevron {
  display: none;
}
.app-shell.collapsed .sidebar-footer .profile-button {
  justify-content: center;
  padding: 4px;
  width: 100%;
}
```

- [ ] **Step 3: Commit**

```bash
git add src/App.vue
git commit -m "feat(sidebar): add collapsed CSS — width transition, icon-only rail"
```

---

### Task 4: Verify in browser

**Files:** None — read-only verification

- [ ] **Step 1: Confirm dev server is running**

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
```

Expected: `200`

If not running: `cd client && npm run dev`

- [ ] **Step 2: Open the app and verify expanded state**

```bash
open http://localhost:3000
```

Confirm:
- Sidebar shows at 240px with brand name, nav labels, and "Collapse" button with `⟨⟨` icon at the bottom

- [ ] **Step 3: Click "Collapse" and verify collapsed state**

- Sidebar animates smoothly to ~56px
- Brand name hidden, logo centered
- Nav labels hidden, icons centered
- "Collapse" label hidden, `⟩` chevron centered
- LanguageSwitcher hidden
- ProfileMenu avatar still visible

- [ ] **Step 4: Hover a nav icon and verify tooltip**

Hover over any nav icon in collapsed mode. A native browser tooltip should show the route label (e.g. "Inventory").

- [ ] **Step 5: Verify localStorage persistence**

With the sidebar collapsed, refresh the page (`Cmd+R`). Sidebar should remain collapsed on reload.

- [ ] **Step 6: Click expand and verify restoration**

Click the `⟩` chevron. Sidebar animates back to 240px. All labels, brand name, and LanguageSwitcher reappear.

- [ ] **Step 7: Verify active route highlight works in both states**

Navigate to `/inventory`. In both expanded and collapsed mode, the Inventory nav icon should be highlighted with the cyan accent.

- [ ] **Step 8: Verify ProfileMenu modal still works**

Open ProfileMenu from the sidebar footer in collapsed mode. The profile modal should open correctly.

- [ ] **Step 9: Final commit**

```bash
git add -A
git commit -m "feat(sidebar): collapsible icon-rail sidebar with localStorage persistence"
```
