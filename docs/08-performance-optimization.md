# Performance Optimization

## Mundarija
- [Reactivity Optimization](#reactivity-optimization)
- [Computed vs Watch](#computed-vs-watch)
- [v-show vs v-if](#v-show-vs-v-if)
- [KeepAlive](#keepalive)
- [Virtual Scrolling](#virtual-scrolling)
- [Debounce va Throttle](#debounce-va-throttle)
- [Code Splitting](#code-splitting)

---

## Reactivity Optimization

### ❌ NOTO'G'RI - Barcha joyda reactivity

```vue
<script setup lang="ts">
import { ref, reactive } from 'vue'

// Bu data faqat initialization uchun, o'zgarmaydi
const config = reactive({
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retryAttempts: 3
}) // Keraksiz reactivity! ❌

// Large immutable data
const categories = ref([
  { id: 1, name: 'Electronics', subcategories: [...] },
  { id: 2, name: 'Clothing', subcategories: [...] },
  // ... 100+ items
]) // Keraksiz reactivity overhead! ❌
</script>
```

### ✅ TO'G'RI - shallowRef, shallowReactive va readonly

```vue
<script setup lang="ts">
import { ref, shallowRef, readonly, markRaw } from 'vue'

// Immutable config - reactivity kerak emas
const config = readonly({
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retryAttempts: 3
})

// Large data - faqat top-level reactivity
const categories = shallowRef([
  { id: 1, name: 'Electronics', subcategories: [...] },
  { id: 2, name: 'Clothing', subcategories: [...] },
  // ... 100+ items
])

// Third-party library instance
const chartInstance = markRaw(new Chart()) // Proxy yo'q
</script>
```

**API Reference:**

```typescript
// ref - Deep reactivity ✅
const state = ref({ nested: { value: 1 } })
state.value.nested.value = 2 // Reactive

// shallowRef - Shallow reactivity ✅
const state = shallowRef({ nested: { value: 1 } })
state.value = { nested: { value: 2 } } // Reactive
state.value.nested.value = 2 // NOT reactive

// reactive - Deep reactivity ✅
const state = reactive({ nested: { value: 1 } })
state.nested.value = 2 // Reactive

// shallowReactive - Shallow reactivity ✅
const state = shallowReactive({ nested: { value: 1 } })
state.nested = { value: 2 } // Reactive
state.nested.value = 2 // NOT reactive

// readonly - Immutable ✅
const state = readonly({ value: 1 })
// state.value = 2 // Error

// markRaw - Non-reactive ✅
const obj = markRaw({ value: 1 })
const state = reactive({ obj })
// state.obj.value = 2 // NOT reactive
```

**Sabab:**
- **Performance:** Deep reactivity qimmat (har bir nested property uchun Proxy)
- **Memory:** Shallow reactivity kamroq memory ishlatadi
- **Large Datasets:** 1000+ itemlar uchun sezilarli performance o'sish
- **Immutable Data:** O'zgarmaydigan data uchun reactivity keraksiz overhead
- **Third-party Objects:** Library instances reactive bo'lmasligi kerak

**Performance ta'siri:**
```
10,000 nested objects:
❌ reactive(): ~150ms initialization, ~50MB memory
✅ shallowReactive(): ~15ms initialization, ~15MB memory
```

---

## Computed vs Watch

### ❌ NOTO'G'RI - Watch ishlatish (computed kerak bo'lganda)

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')
const fullName = ref('')

// Watch ishlatish - NOTO'G'RI! ❌
watch([firstName, lastName], ([first, last]) => {
  fullName.value = `${first} ${last}` // Bu computed bo'lishi kerak!
})
</script>
```

### ✅ TO'G'RI - Computed ishlatish

```vue
<script setup lang="ts">
import { ref, computed, watch } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

// Computed - cached va reactive ✅
const fullName = computed(() => `${firstName.value} ${lastName.value}`)

// Watch - side effects uchun ✅
const userId = ref(1)
watch(userId, async (newId) => {
  // API call, localStorage, etc.
  await fetchUserData(newId)
})
</script>
```

**Qachon qaysi birini ishlatish:**

```typescript
// ✅ Computed - derived state
const fullName = computed(() => `${first.value} ${last.value}`)
const filteredItems = computed(() => items.value.filter(i => i.active))
const totalPrice = computed(() => cart.value.reduce((sum, i) => sum + i.price, 0))

// ✅ Watch - side effects
watch(searchQuery, async (query) => {
  await fetchSearchResults(query) // API call
})

watch(userSettings, (settings) => {
  localStorage.setItem('settings', JSON.stringify(settings)) // Side effect
})

watch(route, (newRoute) => {
  trackPageView(newRoute.path) // Analytics
})
```

**Sabab:**
- **Computed:**
  - Dependency o'zgarmasa qayta hisoblanmaydi (cached)
  - Synchronous va pure function bo'lishi kerak
  - Return value kerak
  - Ko'p joyda ishlatilsa ham bir marta hisoblanadi
- **Watch:**
  - Side effects uchun (API calls, localStorage, logging)
  - Asynchronous bo'lishi mumkin
  - Return value kerak emas
  - Har bir o'zgarishda ishga tushadi

---

## v-show vs v-if

### ❌ NOTO'G'RI - v-show va v-if ni noto'g'ri ishlatish

```vue
<template>
  <!-- Tez-tez toggle bo'ladigan element uchun v-if ❌ -->
  <div v-if="isMenuOpen">
    <ExpensiveComponent /> <!-- Har safar mount/unmount -->
  </div>

  <!-- Kamdan-kam ko'rsatiladigan uchun v-show ❌ -->
  <div v-show="isModalOpen">
    <HeavyModal /> <!-- DOM da har doim bor, visibility bilan yashirilgan -->
  </div>
</template>
```

### ✅ TO'G'RI - Use case ga qarab tanlash

```vue
<template>
  <!-- Tez-tez toggle - v-show ✅ -->
  <div v-show="isMenuOpen">
    <ExpensiveComponent />
    <!-- Bir marta mount qilinadi, CSS display bilan toggle -->
  </div>

  <!-- Kamdan-kam ko'rsatiladi - v-if ✅ -->
  <div v-if="isModalOpen">
    <HeavyModal />
    <!-- Faqat kerak bo'lganda DOM ga qo'shiladi -->
  </div>

  <!-- Conditional rendering with permissions -->
  <button v-if="userRole === 'admin'">
    <!-- User admin bo'lmasa hech qachon render bo'lmaydi -->
    Delete User
  </button>
</template>
```

**Decision Table:**

| Scenario | Use | Sabab |
|----------|-----|-------|
| Tez-tez toggle (dropdown, tooltip) | `v-show` | CSS toggle tez |
| Kamdan-kam o'zgaradi (modal, admin panel) | `v-if` | Lazy rendering |
| Initial false bo'lishi kerak | `v-if` | Erta render skip |
| Permission-based | `v-if` | Security |
| Heavy component | `v-if` yoki `v-show` + `KeepAlive` | Depends |

**Sabab:**
- **v-if:**
  - Element DOM dan to'liq o'chiriladi/qo'shiladi
  - Initial render tezroq (agar false bo'lsa)
  - Toggle qilish qimmat (mount/unmount lifecycle)
  - Lazy rendering - shart true bo'lguncha render qilmaydi
- **v-show:**
  - CSS `display: none` bilan yashiradi
  - Initial render sekinroq (har doim render qiladi)
  - Toggle qilish arzon (faqat CSS)
  - Element har doim DOM da

---

## KeepAlive

### ❌ NOTO'G'RI - KeepAlive ishlatmaslik (kerak bo'lganda)

```vue
<template>
  <!-- Tab switching - component har safar mount/unmount ❌ -->
  <component :is="currentTab" />

  <!-- Route switching - state yo'qoladi ❌ -->
  <router-view />
</template>

<script setup lang="ts">
const currentTab = ref('TabOne')

// TabOne dan TabTwo ga o'tganda TabOne unmount bo'ladi
// State, scroll position, form data yo'qoladi ❌
</script>
```

### ✅ TO'G'RI - KeepAlive ishlatish

```vue
<template>
  <!-- Tab switching - state saqlanadi ✅ -->
  <KeepAlive :max="10">
    <component :is="currentTab" />
  </KeepAlive>

  <!-- Route switching - selected routes uchun ✅ -->
  <router-view v-slot="{ Component }">
    <KeepAlive :include="['UserDashboard', 'ProductList']">
      <component :is="Component" />
    </KeepAlive>
  </router-view>

  <!-- Conditional caching ✅ -->
  <KeepAlive>
    <TabOne v-if="currentTab === 'one'" />
    <TabTwo v-else-if="currentTab === 'two'" />
    <TabThree v-else />
  </KeepAlive>
</template>

<script setup lang="ts">
import { ref, onActivated, onDeactivated } from 'vue'

const currentTab = ref('one')

// KeepAlive lifecycle hooks
onActivated(() => {
  console.log('Component activated (shown)')
  // Refresh data if needed
})

onDeactivated(() => {
  console.log('Component deactivated (hidden)')
  // Cleanup if needed
})
</script>
```

**Advanced KeepAlive:**

```vue
<!-- include/exclude patterns ✅ -->
<template>
  <!-- String pattern -->
  <KeepAlive include="UserDashboard,ProductList">
    <component :is="view" />
  </KeepAlive>

  <!-- Regex pattern -->
  <KeepAlive :include="/Dashboard|List/">
    <component :is="view" />
  </KeepAlive>

  <!-- Array pattern -->
  <KeepAlive :include="['UserDashboard', 'ProductList']">
    <component :is="view" />
  </KeepAlive>

  <!-- Max instances -->
  <KeepAlive :max="5">
    <!-- Faqat 5 ta component instance cache qilinadi -->
    <!-- LRU (Least Recently Used) eviction policy -->
    <component :is="view" />
  </KeepAlive>
</template>

<script setup lang="ts">
// Component name define qilish (cache key)
defineOptions({
  name: 'UserDashboard' // KeepAlive uchun name kerak
})
</script>
```

**Sabab:**
- **Performance:**
  - Component state saqlanadi (form data, scroll position)
  - Mount/unmount lifecycle kerak bo'lmaganda skip qilinadi
  - API calls qayta yuklanmaydi
- **User Experience:**
  - Form data yo'qolmaydi
  - Scroll position saqlanadi
  - Tab switching instant
- **Memory Management:**
  - `max` prop bilan memory leak oldini olish
  - `include/exclude` faqat kerakli componentlarni cache qilish

---

## Virtual Scrolling

### ❌ NOTO'G'RI - Large lists without virtualization

```vue
<template>
  <!-- 10,000+ items - JUDA SEKIN! ❌ -->
  <div class="list">
    <div
      v-for="item in allItems"
      :key="item.id"
      class="list-item"
    >
      {{ item.name }} - {{ item.description }}
    </div>
  </div>
</template>

<script setup lang="ts">
// 10,000 items - barcha DOM nodes yaratiladi ❌
const allItems = ref(Array.from({ length: 10000 }, (_, i) => ({
  id: i,
  name: `Item ${i}`,
  description: `Description for item ${i}`
})))

// Performance: ~5000ms render time, ~200MB memory
</script>
```

### ✅ TO'G'RI - Virtual scrolling

```vue
<template>
  <!-- Quasar virtual scroll ✅ -->
  <q-virtual-scroll
    :items="allItems"
    virtual-scroll-item-size="48"
    style="max-height: 600px"
  >
    <template v-slot="{ item }">
      <q-item :key="item.id">
        <q-item-section>
          <q-item-label>{{ item.name }}</q-item-label>
          <q-item-label caption>{{ item.description }}</q-item-label>
        </q-item-section>
      </q-item>
    </template>
  </q-virtual-scroll>

  <!-- Vue-virtual-scroller alternative ✅ -->
  <RecycleScroller
    :items="allItems"
    :item-size="48"
    key-field="id"
    class="scroller"
  >
    <template #default="{ item }">
      <div class="list-item">
        {{ item.name }} - {{ item.description }}
      </div>
    </template>
  </RecycleScroller>
</template>

<script setup lang="ts">
import { RecycleScroller } from 'vue-virtual-scroller'
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'

const allItems = ref(Array.from({ length: 10000 }, (_, i) => ({
  id: i,
  name: `Item ${i}`,
  description: `Description for item ${i}`
})))

// Performance: ~50ms render time, ~10MB memory
// Faqat visible items DOM da
</script>

<style scoped>
.scroller {
  height: 600px;
}
</style>
```

**Custom Virtual Scroll:**

```vue
<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

interface Props {
  items: any[]
  itemHeight: number
  containerHeight: number
}

const props = defineProps<Props>()

const scrollTop = ref(0)
const containerRef = ref<HTMLElement | null>(null)

// Visible range hisoblash
const visibleRange = computed(() => {
  const start = Math.floor(scrollTop.value / props.itemHeight)
  const visibleCount = Math.ceil(props.containerHeight / props.itemHeight)
  const end = start + visibleCount

  return {
    start: Math.max(0, start - 5), // Buffer
    end: Math.min(props.items.length, end + 5)
  }
})

const visibleItems = computed(() => {
  return props.items.slice(visibleRange.value.start, visibleRange.value.end)
})

const totalHeight = computed(() => props.items.length * props.itemHeight)
const offsetY = computed(() => visibleRange.value.start * props.itemHeight)

const handleScroll = (e: Event) => {
  scrollTop.value = (e.target as HTMLElement).scrollTop
}

onMounted(() => {
  containerRef.value?.addEventListener('scroll', handleScroll)
})

onBeforeUnmount(() => {
  containerRef.value?.removeEventListener('scroll', handleScroll)
})
</script>

<template>
  <div
    ref="containerRef"
    class="virtual-scroll-container"
    :style="{ height: `${containerHeight}px`, overflow: 'auto' }"
  >
    <div :style="{ height: `${totalHeight}px`, position: 'relative' }">
      <div :style="{ transform: `translateY(${offsetY}px)` }">
        <div
          v-for="(item, index) in visibleItems"
          :key="item.id"
          :style="{ height: `${itemHeight}px` }"
        >
          <slot :item="item" :index="visibleRange.start + index" />
        </div>
      </div>
    </div>
  </div>
</template>
```

**Sabab:**
- **Performance Impact:**
  - 10,000 items: ~5000ms → ~50ms render time (100x faster)
  - Memory: ~200MB → ~10MB (20x less)
  - Scroll performance: janky → smooth 60fps
- **When to use:**
  - 100+ items
  - Large datasets (pagination mumkin emas bo'lsa)
  - Table/list views
  - Chat messages, logs

---

## Debounce va Throttle

### ❌ NOTO'G'RI - Debounce/Throttle ishlatmaslik

```vue
<template>
  <!-- Har bir keystroke da API call ❌ -->
  <input
    v-model="searchQuery"
    @input="handleSearch"
  />

  <!-- Har bir scroll event da expensive calc ❌ -->
  <div @scroll="handleScroll">
    <!-- Content -->
  </div>
</template>

<script setup lang="ts">
const searchQuery = ref('')

// Her keystroke da API call - VERY EXPENSIVE! ❌
const handleSearch = async () => {
  await fetch(`/api/search?q=${searchQuery.value}`)
  // User "hello" yozsa: 5 API calls!
}

// Har bir scroll event - 100+ times per second ❌
const handleScroll = () => {
  recalculateLayout() // Expensive
}
</script>
```

### ✅ TO'G'RI - Debounce va Throttle

```vue
<template>
  <!-- Debounced search ✅ -->
  <input
    v-model="searchQuery"
    @input="debouncedSearch"
    placeholder="Search..."
  />

  <!-- Throttled scroll ✅ -->
  <div @scroll="throttledScroll">
    <!-- Content -->
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { useDebounceFn, useThrottleFn } from '@vueuse/core'

const searchQuery = ref('')

// Debounce - 300ms wait after last keystroke ✅
const debouncedSearch = useDebounceFn(async () => {
  if (!searchQuery.value) return

  await fetch(`/api/search?q=${searchQuery.value}`)
  // User "hello" yozsa: 1 API call (300ms after typing stopped)
}, 300)

// Watch bilan debounce ✅
watch(
  searchQuery,
  useDebounceFn(async (newQuery) => {
    await fetch(`/api/search?q=${newQuery}`)
  }, 300)
)

// Throttle - maximum 1 call per 100ms ✅
const throttledScroll = useThrottleFn(() => {
  recalculateLayout()
  // Maximum 10 calls per second (instead of 100+)
}, 100)
</script>
```

**Manual Implementation:**

```typescript
// Debounce
function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: ReturnType<typeof setTimeout>

  return function(this: any, ...args: Parameters<T>) {
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => fn.apply(this, args), delay)
  }
}

// Throttle
function throttle<T extends (...args: any[]) => any>(
  fn: T,
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle: boolean

  return function(this: any, ...args: Parameters<T>) {
    if (!inThrottle) {
      fn.apply(this, args)
      inThrottle = true
      setTimeout(() => inThrottle = false, limit)
    }
  }
}
```

**Debounce vs Throttle:**

```typescript
// DEBOUNCE - wait until action stops
// Use case: Search input, autocomplete, resize events
// Example: User types "hello"
// Events:  h-e-l-l-o----[300ms]----[API CALL]

// THROTTLE - limit frequency
// Use case: Scroll, mousemove, drag events
// Example: Scroll event
// Events:  ↓-↓-↓-[100ms]-↓-↓-↓-[100ms]-↓-↓-↓
// Calls:   ↓-----------↓-----------↓
```

**Sabab:**
- **Debounce:** Wait until user finishes action
- **Throttle:** Limit execution frequency
- **Performance:** Reduces API calls, CPU usage, memory
- **User Experience:** Less jank, smoother interaction

---

## Code Splitting

### ✅ Lazy Loading Routes

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      component: () => import('@/pages/HomePage.vue') // Lazy loaded
    },
    {
      path: '/admin',
      component: () => import('@/pages/AdminPage.vue'), // Separate chunk
      meta: { requiresAuth: true }
    },
    {
      path: '/dashboard',
      component: () => import('@/pages/DashboardPage.vue')
    }
  ]
})
```

### ✅ Lazy Loading Components

```vue
<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

// Lazy load heavy component
const HeavyChart = defineAsyncComponent(() =>
  import('@/components/HeavyChart.vue')
)

// With loading and error components
const AsyncComponent = defineAsyncComponent({
  loader: () => import('@/components/AsyncComponent.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorDisplay,
  delay: 200,
  timeout: 3000
})
</script>

<template>
  <HeavyChart v-if="showChart" />
</template>
```

**Sabab:**
- **Initial Bundle Size:** Kichikroq initial bundle
- **Faster Load:** Critical code birinchi yuklanadi
- **Better UX:** Progressive loading
- **Code Splitting:** Route/component level splitting

---

## Xulosa

Performance optimization best practices:

1. **Reactivity:** `shallowRef`, `readonly`, `markRaw` where appropriate
2. **Computed vs Watch:** Computed for derived state, watch for side effects
3. **v-show vs v-if:** Based on toggle frequency
4. **KeepAlive:** For frequently accessed components
5. **Virtual Scrolling:** For lists > 100 items
6. **Debounce/Throttle:** For high-frequency events
7. **Code Splitting:** Lazy load routes and heavy components

**Performance checklist:**
- [ ] Large arrays use `shallowRef`
- [ ] Template logic in `computed`
- [ ] Virtual scroll for long lists
- [ ] Debounce search inputs
- [ ] Lazy load routes
- [ ] KeepAlive for tabs
- [ ] Optimize images
