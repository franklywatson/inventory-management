# Collapsible Sidebar Design

**Date:** 2026-05-26  
**Status:** Approved  
**Scope:** `client/src/App.vue` only

---

## Problem

The dark slate sidebar occupies 240px at all times. On smaller or split-screen windows, this crowds the content area. Users need a way to reclaim that space while keeping navigation accessible.

## Solution

A manual toggle button in the sidebar footer collapses the sidebar to a 56px icon-only rail. State persists in `localStorage`. A single `.collapsed` CSS class on `.app-shell` drives all visual changes — no JS style manipulation.

---

## States

### Expanded (default, 240px)

- Brand logo + "Catalyst Components" name + subtitle
- Nav items: icon + label side by side
- Footer: `⟨⟨ Collapse` button row, LanguageSwitcher, ProfileMenu (avatar + name)

### Collapsed (56px)

- Brand: logo avatar only, centered
- Nav items: icon only, centered; native `title` attribute provides hover tooltip for label
- Footer: `⟩` expand chevron centered, LanguageSwitcher hidden, ProfileMenu avatar only (name/chevron hidden)

---

## Implementation

### State management (`App.vue` setup)

```js
// Persist collapse state across page refreshes
const collapsed = ref(localStorage.getItem('sidebar-collapsed') === 'true')

const toggleSidebar = () => {
  collapsed.value = !collapsed.value
  localStorage.setItem('sidebar-collapsed', collapsed.value)
}
```

### Template changes

1. Bind `.collapsed` class on `.app-shell`:
   ```html
   <div class="app-shell" :class="{ collapsed }">
   ```

2. Add `title` attribute to each nav `router-link` for collapsed tooltips:
   ```html
   <router-link ... :title="item.label">
   ```

3. Replace the static `⟨⟨ Collapse` row in `.sidebar-footer` with a toggle button:
   ```html
   <button class="sidebar-toggle" @click="toggleSidebar" :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'">
     <span class="toggle-icon" v-html="collapsed ? expandIcon : collapseIcon"></span>
     <span class="toggle-label">Collapse</span>
   </button>
   ```

4. Add SVG refs to `setup()` return:
   ```js
   const collapseIcon = `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="15 18 9 12 15 6"/><polyline points="9 18 3 12 9 6"/></svg>`
   const expandIcon   = `<svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"/><polyline points="15 18 21 12 15 6"/></svg>`
   ```

### CSS changes (all scoped to `.app-shell.collapsed`)

```css
/* Sidebar width transition */
.sidebar {
  transition: width 0.2s ease;
}
.app-shell.collapsed .sidebar {
  width: 56px;
}

/* Brand: hide text, center logo */
.sidebar-brand {
  transition: padding 0.2s ease;
}
.app-shell.collapsed .sidebar-brand {
  justify-content: center;
  padding: 20px 0 16px;
}
.app-shell.collapsed .sidebar-brand-text {
  display: none;
}

/* Nav items: hide label, center icon */
.nav-item {
  transition: padding 0.15s ease;
}
.app-shell.collapsed .nav-item {
  justify-content: center;
  padding: 9px 0;
}
.app-shell.collapsed .nav-label {
  display: none;
}

/* Toggle button */
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

/* Footer: hide LanguageSwitcher in collapsed mode */
.app-shell.collapsed .sidebar-footer .language-switcher {
  display: none;
}
```

---

## What Does NOT Change

- All view files (Dashboard, Inventory, Orders, etc.)
- All modal components
- All composables
- FilterBar
- Backend

---

## Verification

1. Click "Collapse" → sidebar animates to 56px, icons centered, labels gone
2. Hover a nav icon → browser native tooltip shows the route label
3. Click expand chevron → sidebar animates back to 240px
4. Refresh the page → collapsed state is preserved (localStorage)
5. Navigate between routes → active icon stays highlighted in collapsed mode
6. Open ProfileMenu modal → still works from avatar-only collapsed footer
