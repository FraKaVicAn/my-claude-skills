---
name: frontend-dev
description: Auto-activating frontend development skill — React/Vue components, hooks, state management (Redux/Pinia/Zustand), Tailwind, accessibility, performance, PWA, and SEO
origin: jeremylongshore/claude-code-plugins-plus-skills/05-frontend-dev
tools: Read, Write, Edit, Bash, Grep
---

# Frontend Dev Skill

Automated assistance for modern frontend development: component generation, state management, styling, performance optimization, accessibility, and SEO.

## When to Activate

Activates automatically when you mention: React, Vue, component, hook, Redux, Pinia, Zustand, Tailwind, CSS module, styled-components, accessibility, ARIA, PWA, Lighthouse, Core Web Vitals, SEO, bundle size, lazy loading, code splitting.

## Covered Skills (26 total)

| Skill | What it does |
|-------|-------------|
| `react-component-generator` | Functional components, TypeScript props, patterns |
| `vue-component-generator` | Composition API, script setup, defineProps |
| `react-hook-creator` | Custom hooks with proper dependency arrays |
| `vue-composable-creator` | Composables with reactive refs and computed |
| `redux-slice-generator` | RTK slices, createAsyncThunk, selectors |
| `pinia-store-setup` | Pinia stores with actions and getters |
| `zustand-store-creator` | Zustand stores with slices and middleware |
| `css-module-generator` | CSS Modules with TypeScript support |
| `tailwind-class-optimizer` | Class deduplication, variant organization |
| `bundle-size-analyzer` | webpack/vite bundle analysis and optimization |
| `code-splitting-helper` | Route-based and component-level splitting |
| `lazy-loading-implementer` | React.lazy, Intersection Observer patterns |
| `accessibility-audit-runner` | axe-core integration, WCAG 2.1 checks |
| `aria-attribute-helper` | ARIA roles, labels, live regions |
| `color-contrast-checker` | WCAG AA/AAA contrast ratio validation |
| `keyboard-navigation-tester` | Focus management, tab order, skip links |
| `seo-meta-generator` | Meta tags, Open Graph, JSON-LD structured data |
| `pwa-manifest-generator` | Web app manifest, service worker setup |
| `web-vitals-monitor` | LCP, FID/INP, CLS measurement and optimization |
| `performance-lighthouse-runner` | Lighthouse CI integration |

## React Component Pattern

```tsx
import { useState, useCallback } from 'react'

interface ButtonProps {
  label: string
  onClick: () => void
  variant?: 'primary' | 'secondary' | 'danger'
  disabled?: boolean
  isLoading?: boolean
}

export function Button({
  label, onClick, variant = 'primary', disabled = false, isLoading = false
}: ButtonProps) {
  const handleClick = useCallback(() => {
    if (!disabled && !isLoading) onClick()
  }, [disabled, isLoading, onClick])

  return (
    <button
      onClick={handleClick}
      disabled={disabled || isLoading}
      aria-busy={isLoading}
      className={`btn btn-${variant} ${isLoading ? 'btn-loading' : ''}`}
    >
      {isLoading ? <span aria-hidden>...</span> : label}
    </button>
  )
}
```

## Custom Hook Pattern

```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(timer)
  }, [value, delay])
  return debouncedValue
}
```

## Zustand Store

```ts
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'

interface AuthStore {
  user: User | null
  login: (credentials: Credentials) => Promise<void>
  logout: () => void
}

export const useAuthStore = create<AuthStore>()(
  devtools(persist((set) => ({
    user: null,
    login: async (credentials) => {
      const user = await api.login(credentials)
      set({ user }, false, 'auth/login')
    },
    logout: () => set({ user: null }, false, 'auth/logout')
  }), { name: 'auth-store' }))
)
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `05-frontend-dev` (26 skills)
