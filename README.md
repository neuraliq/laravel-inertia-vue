# Laravel + Inertia.js v2 + Vue 3 Skill

An AI agent skill that teaches coding assistants how to build modern SPA-like Laravel applications using Inertia.js v2 with Vue 3 Composition API.

## What This Skill Does

This skill provides procedural knowledge for AI agents working with the Laravel + Inertia.js + Vue 3 stack. It covers the full development lifecycle — from project setup to production deployment — with emphasis on type-safe, maintainable patterns.

### Topics Covered

- **Project Setup** — Root template, app entry point, Vite configuration
- **Page Components** — TypeScript props, `defineProps<Props>()`, `<script setup lang="ts">`
- **Form Handling** — `useForm` for submissions, file uploads, error display, progress tracking
- **Shared Data** — `usePage()`, `HandleInertiaRequests` middleware, flash messages
- **Persistent Layouts** — `defineOptions({ layout })` pattern to prevent layout re-mounting
- **Navigation** — `<Link>` component, `router.visit()`, `router.reload()`, partial reloads
- **Deferred Props** — `Inertia::defer()`, grouped loading, `<Deferred>` component with fallbacks
- **Merge Props** — Infinite scroll with `Inertia::merge()`
- **Prefetching** — Hover and mount prefetching, cache-for, stale-while-revalidate
- **SSR** — Server-side rendering setup with `@vue/server-renderer`, Supervisor config
- **TypeScript** — Global types, augmenting Inertia types, Ziggy typed routes
- **Composables** — `useFilters`, `useConfirmation`, `useModal`, `useBulkActions`, `useFlash`
- **Testing** — Pest assertions with `assertInertia()`, filtering, validation, shared data

## Installation

### Via skills CLI (skills.sh)

```bash
npx skills add neuraliq/laravel-inertia-vue
```

### Via Laravel Boost (skills.laravel.cloud)

```bash
php artisan boost:add-skill neuraliq/laravel-inertia-vue
```

### Manual

Copy the `skills/SKILL.md` file into your project's `.cursor/skills/`, `.claude/skills/`, or equivalent agent skills directory.

## Compatibility

| Agent | Supported |
|-------|-----------|
| Claude Code | Yes |
| Cursor | Yes |
| Windsurf | Yes |
| GitHub Copilot | Yes |

## Requirements

- Laravel 11+
- Inertia.js v2
- Vue 3.4+ with Composition API
- TypeScript
- Vite

## File Structure

```
├── skills/
│   └── SKILL.md          # Main skill definition
├── references/
│   ├── composables.md    # Reusable Vue composables for Inertia patterns
│   ├── deferred-props.md # Deferred props, merge props, prefetching
│   ├── ssr.md            # Server-side rendering setup
│   ├── testing.md        # Pest testing patterns for Inertia pages
│   └── typescript.md     # TypeScript configuration and typed patterns
└── README.md
```

## Key Rules

1. Always use `<script setup lang="ts">` — no Options API
2. Always use `useForm` for form submissions
3. Always define typed props with `defineProps<Props>()`
4. Always use `<Link>` for navigation — never `<a>` tags
5. Always use persistent layouts via `defineOptions({ layout })`
6. Keep page components thin — extract to composables

## License

MIT
