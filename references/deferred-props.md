# Deferred Props, Lazy Evaluation & Prefetching

## Deferred Props (Inertia v2)

Deferred props load after the initial page render, allowing fast page loads with slow queries.

### Server Side

```php
use Inertia\Inertia;

class DashboardController extends Controller
{
    public function index(): \Inertia\Response
    {
        return Inertia::render('Dashboard', [
            // Loads immediately
            'user' => auth()->user(),
            
            // Deferred — loads after page render
            'stats' => Inertia::defer(fn () => [
                'total_users' => User::count(),
                'revenue' => Order::sum('total'),
                'conversion_rate' => $this->calculateConversionRate(),
            ]),
            
            // Deferred with group — loads together
            'recentOrders' => Inertia::defer(
                fn () => Order::latest()->take(10)->get(),
                'activity'
            ),
            'recentUsers' => Inertia::defer(
                fn () => User::latest()->take(10)->get(),
                'activity'
            ),
        ]);
    }
}
```

### Client Side

```vue
<script setup lang="ts">
import { Deferred } from '@inertiajs/vue3'

defineProps<{
    user: { name: string }
    stats?: { total_users: number; revenue: number }
    recentOrders?: { id: number; total: number }[]
    recentUsers?: { id: number; name: string }[]
}>()
</script>

<template>
    <h1>Welcome, {{ user.name }}</h1>

    <!-- Built-in Deferred component with fallback -->
    <!-- Props are accessed via defineProps, NOT via slot props -->
    <Deferred prop="stats">
        <template #fallback>
            <div class="animate-pulse h-20 bg-gray-200 rounded" />
        </template>
        <StatsGrid :stats="stats" />
    </Deferred>

    <!-- Grouped deferred props resolve together -->
    <Deferred prop="recentOrders">
        <template #fallback>
            <LoadingSkeleton />
        </template>
        <OrderList :orders="recentOrders" />
    </Deferred>
</template>
```

## Merge Props (Infinite Scroll)

Merge new data into existing props instead of replacing them.

### Server Side

```php
class PostController extends Controller
{
    public function index(Request $request): \Inertia\Response
    {
        return Inertia::render('Posts/Index', [
            'posts' => Inertia::merge(
                Post::latest()
                    ->paginate(15, page: $request->integer('page', 1))
                    ->through(fn ($post) => [
                        'id' => $post->id,
                        'title' => $post->title,
                        'excerpt' => $post->excerpt,
                    ])
            ),
        ]);
    }
}
```

### Client Side

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { router, usePage } from '@inertiajs/vue3'

const props = defineProps<{
    posts: {
        data: { id: number; title: string; excerpt: string }[]
        next_page_url: string | null
    }
}>()

const loading = ref(false)

function loadMore() {
    if (!props.posts.next_page_url || loading.value) return

    loading.value = true
    router.reload({
        data: { page: new URL(props.posts.next_page_url).searchParams.get('page') },
        only: ['posts'],
        onFinish: () => { loading.value = false },
    })
}
</script>

<template>
    <div v-for="post in posts.data" :key="post.id">
        <h2>{{ post.title }}</h2>
        <p>{{ post.excerpt }}</p>
    </div>

    <button
        v-if="posts.next_page_url"
        :disabled="loading"
        @click="loadMore"
    >
        {{ loading ? 'Loading...' : 'Load More' }}
    </button>
</template>
```

## Prefetching

Pre-load pages on hover or mount for instant navigation.

```vue
<script setup lang="ts">
import { Link } from '@inertiajs/vue3'
</script>

<template>
    <!-- Prefetch on hover (default 75ms delay) -->
    <Link href="/users" prefetch>Users</Link>

    <!-- Prefetch on mount (good for likely next pages) -->
    <Link href="/dashboard" prefetch="mount">Dashboard</Link>

    <!-- Prefetch with custom cache duration -->
    <Link href="/settings" prefetch :cache-for="30000">Settings</Link>

    <!-- Stale-while-revalidate pattern -->
    <Link href="/users" prefetch :cache-for="[15000, 60000]">Users</Link>
</template>
```

## Conditional & Optional Props

```php
return Inertia::render('Users/Show', [
    'user' => $user,
    
    // Only include when explicitly requested via partial reload
    'activityLog' => Inertia::optional(fn () => $user->activityLog()->paginate()),
    
    // Always evaluate but allow partial reloads to skip
    'permissions' => fn () => $user->getAllPermissions(),
]);
```

```vue
<script setup lang="ts">
import { router } from '@inertiajs/vue3'

// Load optional prop on demand
function loadActivity() {
    router.reload({ only: ['activityLog'] })
}
</script>
```
