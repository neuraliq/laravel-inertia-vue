---
name: laravel-inertia-vue
description: Laravel + Inertia.js v2 + Vue 3 Composition API integration patterns. Use when building Inertia page components, handling forms with useForm, implementing layouts, configuring SSR, or working with Vue composables in Laravel Inertia stack.
compatible_agents:
  - Claude Code
  - Cursor
  - Windsurf
  - Copilot
tags:
  - laravel
  - inertia
  - vue
  - vue3
  - composition-api
  - spa
  - frontend
  - ssr
---

# Laravel + Inertia.js v2 + Vue 3

Build modern SPA-like Laravel applications using Inertia.js v2 with Vue 3 Composition API. No client-side routing, no API layer — server-driven pages with Vue component rendering.

## Context

You are building Laravel applications with Inertia.js and Vue 3. This skill covers project setup, page components, form handling, shared data, persistent layouts, and advanced patterns like deferred props and SSR.

## Rules

1. Always use `<script setup lang="ts">` — no Options API, no plain JavaScript
2. Always use `useForm` for form submissions — never raw `router.post()` calls
3. Always define `Props` interface with `defineProps<Props>()` — never use runtime prop validation
4. Always use `<Link>` for navigation — never `<a>` tags for internal links
5. Always use `preserveScroll: true` on form submissions unless intentionally scrolling to top
6. Always use persistent layouts via `defineOptions({ layout })` — never wrap in template
7. Use `router.reload({ only: [...] })` to refresh specific props without full navigation
8. Use computed properties for derived data from `usePage()` — props are reactive
9. Handle flash messages via shared data, not prop passing
10. Keep page components thin — extract complex logic into composables

## Examples

### Project Setup

```bash
# New project with starter kit
laravel new my-app
# Select Vue with Inertia

# Existing project
composer require inertiajs/inertia-laravel
npm install @inertiajs/vue3
```

### Root Template

```blade
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    @vite(['resources/css/app.css', 'resources/js/app.ts'])
    @inertiaHead
</head>
<body>
    @inertia
</body>
</html>
```

### App Entry

```typescript
// resources/js/app.ts
import { createApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/vue3'
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers'

createInertiaApp({
    title: (title) => title ? `${title} - MyApp` : 'MyApp',
    resolve: (name) => resolvePageComponent(
        `./Pages/${name}.vue`,
        import.meta.glob('./Pages/**/*.vue')
    ),
    setup({ el, App, props, plugin }) {
        createApp({ render: () => h(App, props) })
            .use(plugin)
            .mount(el)
    },
})
```

### Page Component

```vue
<script setup lang="ts">
import { Head, Link } from '@inertiajs/vue3'
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue'

interface Props {
    users: { id: number; name: string; email: string }[]
    filters: { search: string }
}

const props = defineProps<Props>()
</script>

<template>
    <Head title="Users" />
    <AuthenticatedLayout>
        <div v-for="user in users" :key="user.id">
            <Link :href="`/users/${user.id}`">{{ user.name }}</Link>
        </div>
    </AuthenticatedLayout>
</template>
```

### Controller

```php
use Inertia\Inertia;

class UserController extends Controller
{
    public function index(Request $request): \Inertia\Response
    {
        return Inertia::render('Users/Index', [
            'users' => User::query()
                ->when($request->search, fn ($q, $s) => $q->where('name', 'like', "%{$s}%"))
                ->paginate(15),
            'filters' => $request->only('search'),
        ]);
    }
}
```

### Form with useForm

```vue
<script setup lang="ts">
import { useForm } from '@inertiajs/vue3'

const form = useForm({
    name: '',
    email: '',
})

function submit() {
    form.post('/users', {
        preserveScroll: true,
        onSuccess: () => form.reset(),
    })
}
</script>

<template>
    <form @submit.prevent="submit">
        <input v-model="form.name" type="text" />
        <span v-if="form.errors.name">{{ form.errors.name }}</span>
        <button type="submit" :disabled="form.processing">Save</button>
    </form>
</template>
```

### File Upload

```vue
<script setup lang="ts">
const form = useForm({
    avatar: null as File | null,
})

function submit() {
    form.post('/profile', { forceFormData: true })
}
</script>

<template>
    <input type="file" @input="form.avatar = ($event.target as HTMLInputElement).files?.[0]" />
    <progress v-if="form.progress" :value="form.progress.percentage" max="100" />
</template>
```

### Shared Data

```php
// HandleInertiaRequests.php
public function share(Request $request): array
{
    return [
        ...parent::share($request),
        'auth' => ['user' => $request->user()],
        'flash' => ['success' => $request->session()->get('success')],
    ];
}
```

```vue
<script setup lang="ts">
import { usePage } from '@inertiajs/vue3'

const page = usePage()
const user = computed(() => page.props.auth.user)
</script>
```

### Persistent Layout

```vue
<!-- Layout: resources/js/Layouts/AuthenticatedLayout.vue -->
<script setup lang="ts">
import { Link } from '@inertiajs/vue3'
</script>

<template>
    <div class="min-h-screen">
        <nav><Link href="/dashboard">Dashboard</Link></nav>
        <main><slot /></main>
    </div>
</template>
```

```vue
<!-- Page: resources/js/Pages/Dashboard.vue -->
<script setup lang="ts">
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue'

defineOptions({ layout: AuthenticatedLayout })
</script>

<template>
    <h1>Dashboard</h1>
</template>
```

### Navigation

```vue
<script setup lang="ts">
import { Link, router } from '@inertiajs/vue3'

function refresh() {
    router.reload({ only: ['users'] })
}

function deleteUser(id: number) {
    router.delete(`/users/${id}`, { preserveScroll: true })
}
</script>

<template>
    <Link href="/users">Users</Link>
    <Link href="/users" preserve-scroll>Preserve Scroll</Link>
    <Link href="/logout" method="post" as="button">Logout</Link>
</template>
```

## Anti-Patterns

- Using `<a>` tags for internal navigation — defeats Inertia's purpose
- Using raw axios/fetch instead of `useForm` — lose error handling and progress
- Defining props without TypeScript — loses type safety
- Re-mounting layout on every page — use persistent layouts
- Passing flash messages as props — use shared data
- Thick page components — extract to composables

## References

- [Inertia.js](https://inertiajs.com/)
- [Inertia Vue Adapter](https://inertiajs.com/vue)
- [Laravel Integration](https://inertiajs.com/server-side-setup#laravel)

## When to Use This Skill

Use this skill when:
- Building Inertia page components with Vue
- Handling forms with useForm
- Managing shared data with usePage
- Implementing persistent layouts
- Configuring SSR
- Working with Vue 3 composables
- Setting up new Laravel + Vue + Inertia projects
