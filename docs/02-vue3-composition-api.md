# Vue 3 Composition API va Reactivity System

## Mundarija
- [Composition API vs Options API](#composition-api-vs-options-api)
- [Reactivity System](#reactivity-system)
- [Lifecycle Hooks](#lifecycle-hooks)
- [Advanced Composition Patterns](#advanced-composition-patterns)

---

## Composition API vs Options API

### ❌ NOTO'G'RI - Options API (yangi loyihalar uchun)

```vue
<script lang="ts">
export default {
  data() {
    return {
      count: 0,
      user: null as User | null
    }
  },
  computed: {
    doubleCount(): number {
      return this.count * 2
    }
  },
  methods: {
    increment() {
      this.count++
    },
    async fetchUser() {
      this.user = await userApi.getUser(1)
    }
  },
  mounted() {
    this.fetchUser()
  }
}
</script>
```

**Muammolar:**
- `this` context - TypeScript bilan yomon ishlaydi
- Logic type bo'yicha bo'lingan (data, methods, computed)
- Logic qayta ishlatish qiyin
- Tree-shaking yo'q

### ✅ TO'G'RI - Composition API + `<script setup>`

```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { userApi } from '@/services/api/user.api'
import type { User } from '@/types/models/user'

// State
const count = ref(0)
const user = ref<User | null>(null)

// Computed
const doubleCount = computed(() => count.value * 2)

// Methods
const increment = () => {
  count.value++
}

const fetchUser = async () => {
  user.value = await userApi.getUser(1)
}

// Lifecycle
onMounted(() => {
  fetchUser()
})
</script>
```

**Afzalliklari:**
- To'liq TypeScript support, `this` yo'q
- Logic feature bo'yicha guruplanadi
- Composables orqali logic reuse qilish oson
- Tree-shaking - faqat ishlatilgan features bundle ga kiradi
- Performance yaxshiroq

---

## Reactivity System

### ref() vs reactive()

#### ref() - Primitiv qiymatlar uchun

```vue
<script setup lang="ts">
import { ref } from 'vue'

// ✅ Primitiv qiymatlar
const count = ref(0)
const message = ref('')
const isActive = ref(false)

// ✅ Array
const items = ref<string[]>([])

// ✅ Single object (yoki ref yoki reactive)
const user = ref<User>({
  name: 'John',
  age: 30
})

// Access - .value orqali
console.log(count.value) // 0
count.value = 1
user.value.name = 'Jane'
</script>

<template>
  <!-- Template da .value kerak emas -->
  <p>{{ count }}</p>
  <p>{{ user.name }}</p>
</template>
```

#### reactive() - Objects uchun

```vue
<script setup lang="ts">
import { reactive } from 'vue'

// ✅ Object
const state = reactive({
  count: 0,
  message: 'Hello'
})

// ✅ Nested objects
const user = reactive({
  profile: {
    name: 'John',
    age: 30,
    address: {
      city: 'New York'
    }
  },
  settings: {
    theme: 'dark'
  }
})

// Access - to'g'ridan-to'g'ri
console.log(state.count) // 0
state.count = 1
user.profile.name = 'Jane'
user.profile.address.city = 'London'
</script>

<template>
  <p>{{ state.count }}</p>
  <p>{{ user.profile.name }}</p>
</template>
```

#### ❌ NOTO'G'RI - Noto'g'ri ishlatish

```vue
<script setup lang="ts">
// ❌ Primitive qiymat uchun reactive
const count = reactive(0) // Bu ishlamaydi!

// ❌ reactive() ni reassign qilish
let state = reactive({ count: 0 })
state = reactive({ count: 1 }) // Reactivity yo'qoladi! ❌

// ❌ reactive() ni destructure qilish
const { count } = reactive({ count: 0 }) // Reactivity yo'qoladi! ❌
</script>
```

#### ✅ TO'G'RI - To'g'ri ishlatish

```vue
<script setup lang="ts">
import { ref, reactive, toRefs } from 'vue'

// ✅ Primitive uchun ref
const count = ref(0)

// ✅ Object uchun reactive
const state = reactive({ count: 0 })

// ✅ reactive() ni reassign qilmaslik, property o'zgartirish
const state = reactive({ count: 0 })
// state = reactive({ count: 1 }) // ❌
state.count = 1 // ✅

// ✅ toRefs() bilan destructure
const state = reactive({ count: 0, name: '' })
const { count, name } = toRefs(state) // Reactivity saqlanadi ✅

// Endi count va name ref lar
console.log(count.value) // 0
count.value = 1
</script>
```

---

### Shallow Reactivity - Performance uchun

#### shallowRef() - Faqat .value reactive

```vue
<script setup lang="ts">
import { shallowRef } from 'vue'

// ✅ Large dataset - faqat top-level reactivity
const data = shallowRef({
  items: Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`,
    nested: { value: i }
  }))
})

// Trigger reactivity - to'liq object replace qilish
data.value = {
  items: [...newItems]
}

// ❌ Nested property o'zgartirish - reactivity trigger qilmaydi
data.value.items[0].name = 'New Name' // UI yangilanmaydi!

// ✅ Force update
import { triggerRef } from 'vue'
data.value.items[0].name = 'New Name'
triggerRef(data) // Manual trigger
</script>
```

#### shallowReactive() - Faqat top-level reactive

```vue
<script setup lang="ts">
import { shallowReactive } from 'vue'

const state = shallowReactive({
  count: 0,  // ✅ Reactive
  nested: {  // ❌ nested properties reactive emas
    value: 0
  }
})

// ✅ Top-level o'zgarishi - UI yangilanadi
state.count = 1

// ❌ Nested o'zgarishi - UI yangilanmaydi
state.nested.value = 1
</script>
```

**Qachon ishlatish:**
- Large datasets (10,000+ items)
- Immutable data structures
- Third-party library integrations

**Performance impact:**
```
10,000 nested objects:
reactive():        ~150ms init, ~50MB memory
shallowReactive(): ~15ms init,  ~15MB memory
```

---

### readonly() - Immutable state

```vue
<script setup lang="ts">
import { reactive, readonly } from 'vue'

// Internal state - editable
const state = reactive({
  count: 0,
  items: []
})

// Public state - readonly
const publicState = readonly(state)

// ❌ Cannot modify
// publicState.count = 1 // Error: Cannot assign to read only property

// ✅ Can read
console.log(publicState.count) // 0

// Expose readonly to parent component
defineExpose({
  state: publicState
})
</script>
```

**Use cases:**
- Store state ni component ga expose qilish
- Props ni child component ichida o'zgartirmaslik
- Immutable configuration

---

### markRaw() - Non-reactive

```vue
<script setup lang="ts">
import { reactive, markRaw } from 'vue'
import Chart from 'chart.js'

// ❌ Third-party instance - reactive qilish keraksiz va xavfli
const state = reactive({
  chartInstance: new Chart(ctx, config) // ❌ Performance issue!
})

// ✅ markRaw() - skip reactivity
const state = reactive({
  chartInstance: markRaw(new Chart(ctx, config)) // ✅ No Proxy wrapper
})
</script>
```

**Qachon ishlatish:**
- Third-party library instances (Chart.js, Leaflet, etc.)
- DOM elements
- Large objects that never change

---

## Lifecycle Hooks

### Composition API Lifecycle

```vue
<script setup lang="ts">
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onActivated,
  onDeactivated,
  onErrorCaptured
} from 'vue'

// Setup - component instance yaratildi
console.log('Setup running...')

// Before Mount - DOM mount bo'lishdan oldin
onBeforeMount(() => {
  console.log('Before mount')
})

// Mounted - DOM mount bo'ldi ✅
onMounted(() => {
  console.log('Mounted')
  // DOM access safe
  // API calls
  // Event listeners
})

// Before Update - reactive state o'zgardi, lekin DOM hali yangilanmagan
onBeforeUpdate(() => {
  console.log('Before update')
})

// Updated - DOM yangilandi
onUpdated(() => {
  console.log('Updated')
  // DOM manipulations after update
})

// Before Unmount - component unmount bo'lishdan oldin
onBeforeUnmount(() => {
  console.log('Before unmount')
  // Cleanup
})

// Unmounted - component unmount bo'ldi ✅
onUnmounted(() => {
  console.log('Unmounted')
  // Remove event listeners
  // Clear timers
  // Cancel pending requests
})

// KeepAlive - component activated
onActivated(() => {
  console.log('Activated')
  // Refresh data
})

// KeepAlive - component deactivated
onDeactivated(() => {
  console.log('Deactivated')
  // Pause/stop operations
})

// Error handling
onErrorCaptured((err, instance, info) => {
  console.error('Error captured:', err, info)
  return false // Propagate error
})
</script>
```

### Lifecycle Best Practices

#### ✅ onMounted() - API calls, DOM access

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const data = ref([])
const chartRef = ref<HTMLCanvasElement | null>(null)

onMounted(async () => {
  // ✅ API calls
  data.value = await fetchData()

  // ✅ DOM access
  if (chartRef.value) {
    new Chart(chartRef.value, config)
  }

  // ✅ Event listeners
  window.addEventListener('resize', handleResize)

  // ✅ Third-party library initialization
  initializeMap()
})
</script>
```

#### ✅ onUnmounted() - Cleanup

```vue
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'

let timerId: number
let subscription: Subscription

onMounted(() => {
  // Start timer
  timerId = setInterval(() => {
    console.log('Tick')
  }, 1000)

  // Subscribe
  subscription = eventBus.subscribe('event', handler)

  // Event listener
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  // ✅ Clear timer
  clearInterval(timerId)

  // ✅ Unsubscribe
  subscription.unsubscribe()

  // ✅ Remove event listener
  window.removeEventListener('resize', handleResize)

  // ✅ Cancel pending requests
  abortController.abort()
})
</script>
```

#### ❌ NOTO'G'RI - Cleanup qilmaslik

```vue
<script setup lang="ts">
onMounted(() => {
  setInterval(() => {
    console.log('Tick')
  }, 1000)
  // ❌ onUnmounted da clear qilinmagan - memory leak!

  window.addEventListener('resize', handleResize)
  // ❌ Remove qilinmagan - memory leak!
})

// onUnmounted yo'q ❌
</script>
```

---

## Advanced Composition Patterns

### Composable Pattern

#### Basic Composable

```typescript
// composables/useCounter.ts
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)

  const doubled = computed(() => count.value * 2)

  const increment = () => {
    count.value++
  }

  const decrement = () => {
    count.value--
  }

  const reset = () => {
    count.value = initialValue
  }

  return {
    count: readonly(count),
    doubled,
    increment,
    decrement,
    reset
  }
}

// Usage
const { count, doubled, increment } = useCounter(10)
```

#### Async Composable

```typescript
// composables/useAsyncData.ts
import { ref, type Ref } from 'vue'

interface UseAsyncDataOptions<T> {
  immediate?: boolean
  initialData?: T
  onSuccess?: (data: T) => void
  onError?: (error: Error) => void
}

export function useAsyncData<T>(
  fetcher: () => Promise<T>,
  options: UseAsyncDataOptions<T> = {}
) {
  const {
    immediate = true,
    initialData,
    onSuccess,
    onError
  } = options

  const data = ref<T | undefined>(initialData) as Ref<T | undefined>
  const loading = ref(false)
  const error = ref<Error | null>(null)

  const execute = async () => {
    loading.value = true
    error.value = null

    try {
      const result = await fetcher()
      data.value = result
      onSuccess?.(result)
      return result
    } catch (e) {
      const err = e as Error
      error.value = err
      onError?.(err)
      throw err
    } finally {
      loading.value = false
    }
  }

  if (immediate) {
    execute()
  }

  return {
    data: readonly(data),
    loading: readonly(loading),
    error: readonly(error),
    execute,
    refresh: execute
  }
}

// Usage
const { data, loading, error, refresh } = useAsyncData(
  () => fetch('/api/users').then(r => r.json()),
  {
    onSuccess: (users) => console.log(`Loaded ${users.length} users`),
    onError: (err) => console.error('Failed to load users:', err)
  }
)
```

#### Composable with Lifecycle

```typescript
// composables/useEventListener.ts
import { onMounted, onUnmounted } from 'vue'

export function useEventListener(
  target: EventTarget,
  event: string,
  handler: EventListener,
  options?: AddEventListenerOptions
) {
  onMounted(() => {
    target.addEventListener(event, handler, options)
  })

  onUnmounted(() => {
    target.removeEventListener(event, handler, options)
  })
}

// Usage
import { ref } from 'vue'
import { useEventListener } from '@/composables/useEventListener'

const count = ref(0)

useEventListener(window, 'click', () => {
  count.value++
})
// Automatic cleanup! ✅
```

---

### Composable Composition

#### Composing Multiple Composables

```typescript
// composables/useUserManagement.ts
import { useAsyncData } from './useAsyncData'
import { usePagination } from './usePagination'
import { useSearch } from './useSearch'
import { userApi } from '@/services/api/user.api'

export function useUserManagement() {
  // Search
  const { searchQuery, debouncedQuery } = useSearch()

  // Pagination
  const { currentPage, pageSize, totalPages, goToPage } = usePagination()

  // Fetch users
  const { data: users, loading, error, refresh } = useAsyncData(
    async () => {
      const params = {
        page: currentPage.value,
        pageSize: pageSize.value,
        search: debouncedQuery.value
      }
      return userApi.getUsers(params)
    }
  )

  // Refresh when dependencies change
  watch([currentPage, pageSize, debouncedQuery], () => {
    refresh()
  })

  return {
    // Search
    searchQuery,

    // Pagination
    currentPage,
    totalPages,
    goToPage,

    // Data
    users,
    loading,
    error,
    refresh
  }
}

// Usage - All functionality in one composable!
const {
  searchQuery,
  currentPage,
  totalPages,
  goToPage,
  users,
  loading
} = useUserManagement()
```

---

### Provide/Inject Pattern

#### Typed Provide/Inject

```typescript
// types/injection-keys.ts
import type { InjectionKey } from 'vue'

export interface ThemeConfig {
  primary: string
  secondary: string
  dark: boolean
}

export const ThemeKey: InjectionKey<ThemeConfig> = Symbol('theme')
```

```vue
<!-- App.vue - Provider -->
<script setup lang="ts">
import { provide, reactive } from 'vue'
import { ThemeKey, type ThemeConfig } from '@/types/injection-keys'

const theme = reactive<ThemeConfig>({
  primary: '#1976D2',
  secondary: '#26A69A',
  dark: false
})

provide(ThemeKey, theme)
</script>
```

```vue
<!-- Child.vue - Consumer -->
<script setup lang="ts">
import { inject } from 'vue'
import { ThemeKey } from '@/types/injection-keys'

const theme = inject(ThemeKey)
if (!theme) {
  throw new Error('Theme not provided')
}

// ✅ Fully typed!
console.log(theme.primary) // string
console.log(theme.dark) // boolean
</script>
```

#### Composable with Provide/Inject

```typescript
// composables/useTheme.ts
import { inject, provide, reactive } from 'vue'
import { ThemeKey, type ThemeConfig } from '@/types/injection-keys'

// Provider
export function provideTheme() {
  const theme = reactive<ThemeConfig>({
    primary: '#1976D2',
    secondary: '#26A69A',
    dark: false
  })

  const toggleDark = () => {
    theme.dark = !theme.dark
  }

  const setTheme = (config: Partial<ThemeConfig>) => {
    Object.assign(theme, config)
  }

  provide(ThemeKey, theme)

  return {
    theme,
    toggleDark,
    setTheme
  }
}

// Consumer
export function useTheme() {
  const theme = inject(ThemeKey)

  if (!theme) {
    throw new Error('useTheme must be called in a component with provideTheme')
  }

  return theme
}

// Usage
// App.vue
const { theme, toggleDark } = provideTheme()

// Child.vue
const theme = useTheme()
console.log(theme.primary)
```

---

### Template Refs

#### Basic Template Ref

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const inputRef = ref<HTMLInputElement | null>(null)

onMounted(() => {
  // ✅ DOM access
  inputRef.value?.focus()
})

const focusInput = () => {
  inputRef.value?.focus()
}
</script>

<template>
  <input ref="inputRef" type="text" />
  <button @click="focusInput">Focus</button>
</template>
```

#### Component Ref

```vue
<!-- Child.vue -->
<script setup lang="ts">
const count = ref(0)
const increment = () => count.value++

// Expose methods to parent
defineExpose({
  count,
  increment
})
</script>

<!-- Parent.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const childRef = ref<InstanceType<typeof Child> | null>(null)

onMounted(() => {
  // ✅ Access exposed properties
  console.log(childRef.value?.count)
  childRef.value?.increment()
})
</script>

<template>
  <Child ref="childRef" />
</template>
```

#### Template Ref in v-for

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

const itemRefs = ref<HTMLElement[]>([])

onMounted(() => {
  console.log(itemRefs.value) // Array of elements
})

// Function ref
const setItemRef = (el: HTMLElement | null) => {
  if (el) {
    itemRefs.value.push(el)
  }
}
</script>

<template>
  <div
    v-for="item in items"
    :key="item.id"
    :ref="setItemRef"
  >
    {{ item.name }}
  </div>
</template>
```

---

## Xulosa

### Best Practices Summary

1. **Composition API:**
   - `<script setup>` ishlatish
   - Options API yangi loyihalarda ishlatmaslik

2. **Reactivity:**
   - Primitive uchun `ref()`
   - Object uchun `reactive()`
   - Large data uchun `shallowRef()/shallowReactive()`
   - Readonly exposure uchun `readonly()`
   - Third-party uchun `markRaw()`

3. **Lifecycle:**
   - `onMounted()` - API calls, DOM access
   - `onUnmounted()` - Cleanup (mandatory!)

4. **Composables:**
   - Reusable logic
   - TypeScript support
   - Return readonly state
   - Composition pattern

5. **Provide/Inject:**
   - InjectionKey ishlatish
   - Type safety
   - Error handling
