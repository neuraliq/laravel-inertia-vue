# Server-Side Rendering (SSR)

## Setup

```bash
npm install @vue/server-renderer
```

### SSR Entry Point (`resources/js/ssr.ts`)

```typescript
import { createInertiaApp } from '@inertiajs/vue3'
import createServer from '@inertiajs/vue3/server'
import { renderToString } from '@vue/server-renderer'
import { createSSRApp, h } from 'vue'
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers'

createServer((page) =>
    createInertiaApp({
        page,
        render: renderToString,
        title: (title) => title ? `${title} - MyApp` : 'MyApp',
        resolve: (name) => resolvePageComponent(
            `./Pages/${name}.vue`,
            import.meta.glob('./Pages/**/*.vue')
        ),
        setup({ App, props, plugin }) {
            return createSSRApp({ render: () => h(App, props) })
                .use(plugin)
        },
    })
)
```

### Vite Config

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.ts',
            ssr: 'resources/js/ssr.ts',
            refresh: true,
        }),
        vue({
            template: {
                transformAssetUrls: {
                    base: null,
                    includeAbsolute: false,
                },
            },
        }),
    ],
    resolve: {
        alias: {
            '@': '/resources/js',
        },
    },
})
```

### Build & Run

```bash
# Build SSR bundle
npm run build

# Start SSR server
php artisan inertia:start-ssr

# Stop SSR server
php artisan inertia:stop-ssr
```

### Production (Supervisor)

```ini
[program:inertia-ssr]
command=php artisan inertia:start-ssr
directory=/var/www/html
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/inertia-ssr.log
```

## SSR Gotchas

1. **No `window` or `document`** — Browser APIs don't exist on the server. Guard with:
   ```typescript
   import { onMounted } from 'vue'
   
   onMounted(() => {
       // Safe to use browser APIs here — only runs client-side
       window.addEventListener('scroll', handleScroll)
   })
   ```

2. **No `localStorage`** — Use `usePage().props` for initial state, not browser storage.

3. **Third-party libraries** — Some UI libraries access `window` on import. Use dynamic imports:
   ```vue
   <script setup lang="ts">
   import { defineAsyncComponent } from 'vue'
   
   const MapComponent = defineAsyncComponent(() => import('@/Components/Map.vue'))
   </script>
   ```

4. **Memory leaks** — Each request creates a new Vue app. Avoid module-level state:
   ```typescript
   // BAD — shared across all requests
   const sharedState = reactive({})
   
   // GOOD — per-component state
   const state = ref({})
   ```
