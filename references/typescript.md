# TypeScript Setup for Inertia + Vue 3

## Global Type Definitions

```typescript
// resources/js/types/index.d.ts
export interface User {
    id: number
    name: string
    email: string
    email_verified_at: string | null
    created_at: string
    updated_at: string
}

export interface PaginatedData<T> {
    data: T[]
    links: {
        first: string | null
        last: string | null
        prev: string | null
        next: string | null
    }
    meta: {
        current_page: number
        from: number | null
        last_page: number
        per_page: number
        to: number | null
        total: number
        links: { url: string | null; label: string; active: boolean }[]
    }
}

export interface FlashMessages {
    success: string | null
    error: string | null
    warning: string | null
}

export type PageProps<T extends Record<string, unknown> = Record<string, unknown>> = T & {
    auth: {
        user: User | null
    }
    flash: FlashMessages
}
```

## Augmenting Inertia Types

```typescript
// resources/js/types/inertia.d.ts
import type { PageProps } from '.'

declare module '@inertiajs/vue3' {
    interface InertiaPageProps extends PageProps {}
}
```

## tsconfig.json

```json
{
    "compilerOptions": {
        "target": "ESNext",
        "module": "ESNext",
        "moduleResolution": "bundler",
        "strict": true,
        "jsx": "preserve",
        "paths": {
            "@/*": ["./resources/js/*"]
        },
        "types": ["vite/client"]
    },
    "include": [
        "resources/js/**/*.ts",
        "resources/js/**/*.d.ts",
        "resources/js/**/*.vue"
    ]
}
```

## Typed Page Components Pattern

```vue
<script setup lang="ts">
import type { PaginatedData, User } from '@/types'

interface Props {
    users: PaginatedData<User>
    filters: {
        search: string
        role: string
    }
    roles: { value: string; label: string }[]
}

const props = defineProps<Props>()
</script>
```

## Ziggy (Typed Routes)

```bash
composer require tightenco/ziggy
php artisan ziggy:generate --types
```

```typescript
// resources/js/types/ziggy.d.ts (auto-generated)
// Use route() with type safety
import { route as routeFn } from 'ziggy-js'

declare global {
    function route(name: string, params?: Record<string, any>): string
}
```

```vue
<script setup lang="ts">
import { router } from '@inertiajs/vue3'

// Type-safe route usage
router.visit(route('users.show', { user: 1 }))
</script>
```
