# Vue Composables for Inertia

## useFilters — URL-synced filters

```typescript
// resources/js/Composables/useFilters.ts
import { ref, watch } from 'vue'
import { router } from '@inertiajs/vue3'
import { debounce } from 'lodash-es'

export function useFilters<T extends Record<string, any>>(
    initialFilters: T,
    routeName: string,
    options: { debounceMs?: number; only?: string[] } = {}
) {
    const filters = ref({ ...initialFilters }) as { value: T }
    const { debounceMs = 300, only } = options

    const applyFilters = debounce(() => {
        router.get(routeName, filters.value as any, {
            preserveState: true,
            preserveScroll: true,
            only,
            replace: true,
        })
    }, debounceMs)

    watch(filters, applyFilters, { deep: true })

    function reset() {
        filters.value = { ...initialFilters }
    }

    return { filters, reset }
}
```

Usage:
```vue
<script setup lang="ts">
import { useFilters } from '@/Composables/useFilters'

const props = defineProps<{
    users: { data: any[]; links: any[] }
    filters: { search: string; role: string; status: string }
}>()

const { filters, reset } = useFilters(props.filters, '/users', {
    only: ['users'],
})
</script>

<template>
    <input v-model="filters.search" placeholder="Search..." />
    <select v-model="filters.role">
        <option value="">All Roles</option>
        <option value="admin">Admin</option>
        <option value="user">User</option>
    </select>
    <button @click="reset">Clear Filters</button>
</template>
```

## useConfirmation — Delete/destructive action dialogs

```typescript
// resources/js/Composables/useConfirmation.ts
import { ref } from 'vue'
import { router } from '@inertiajs/vue3'

export function useConfirmation() {
    const showing = ref(false)
    const processing = ref(false)
    const pendingAction = ref<(() => void) | null>(null)

    function confirm(action: () => void) {
        pendingAction.value = action
        showing.value = true
    }

    function confirmDelete(url: string, options?: Record<string, any>) {
        confirm(() => {
            processing.value = true
            router.delete(url, {
                preserveScroll: true,
                onFinish: () => {
                    processing.value = false
                    showing.value = false
                },
                ...options,
            })
        })
    }

    function proceed() {
        pendingAction.value?.()
    }

    function cancel() {
        showing.value = false
        pendingAction.value = null
    }

    return { showing, processing, confirm, confirmDelete, proceed, cancel }
}
```

## useModal — Inertia modal management

```typescript
// resources/js/Composables/useModal.ts
import { ref, watch } from 'vue'

export function useModal(onClose?: () => void) {
    const show = ref(false)
    const data = ref<any>(null)

    function open(payload?: any) {
        data.value = payload ?? null
        show.value = true
    }

    function close() {
        show.value = false
        data.value = null
        onClose?.()
    }

    return { show, data, open, close }
}
```

## useBulkActions — Table row selection

```typescript
// resources/js/Composables/useBulkActions.ts
import { ref, computed } from 'vue'

export function useBulkActions<T extends { id: number | string }>(items: () => T[]) {
    const selected = ref<Set<number | string>>(new Set())

    const allSelected = computed(() => {
        const all = items()
        return all.length > 0 && all.every(item => selected.value.has(item.id))
    })

    const someSelected = computed(() => selected.value.size > 0)

    function toggle(id: number | string) {
        if (selected.value.has(id)) {
            selected.value.delete(id)
        } else {
            selected.value.add(id)
        }
        // Force reactivity
        selected.value = new Set(selected.value)
    }

    function toggleAll() {
        if (allSelected.value) {
            selected.value = new Set()
        } else {
            selected.value = new Set(items().map(item => item.id))
        }
    }

    function clear() {
        selected.value = new Set()
    }

    const selectedIds = computed(() => Array.from(selected.value))

    return { selected, allSelected, someSelected, toggle, toggleAll, clear, selectedIds }
}
```

## useFlash — Flash message handling

```typescript
// resources/js/Composables/useFlash.ts
import { computed } from 'vue'
import { usePage } from '@inertiajs/vue3'

export function useFlash() {
    const page = usePage()

    const success = computed(() => page.props.flash?.success as string | null)
    const error = computed(() => page.props.flash?.error as string | null)
    const warning = computed(() => page.props.flash?.warning as string | null)

    const hasFlash = computed(() => !!(success.value || error.value || warning.value))

    return { success, error, warning, hasFlash }
}
```
