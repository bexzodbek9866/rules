# Component Tashkiloti

## Mundarija
- [Component Structure](#component-structure)
- [Composables Pattern](#composables-pattern)
- [Component Naming](#component-naming)
- [Prop Validation](#prop-validation)
- [Event Naming](#event-naming)
- [Slots Pattern](#slots-pattern)

---

## Component Structure

### ❌ NOTO'G'RI - Bir komponentda barcha logic

```vue
<!-- UserDashboard.vue - 500+ lines ❌ -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'

// User management logic
const users = ref([])
const fetchUsers = async () => { /* ... */ }
const updateUser = async () => { /* ... */ }
const deleteUser = async () => { /* ... */ }

// Analytics logic
const analytics = ref({})
const fetchAnalytics = async () => { /* ... */ }
const processAnalytics = () => { /* ... */ }

// Notification logic
const notifications = ref([])
const fetchNotifications = async () => { /* ... */ }
const markAsRead = async () => { /* ... */ }

// Settings logic
const settings = ref({})
const updateSettings = async () => { /* ... */ }

// ... 300+ lines more
</script>

<template>
  <!-- 200+ lines template -->
</template>
```

### ✅ TO'G'RI - Composables va kichik componentlar

```typescript
// composables/useUsers.ts
import { ref } from 'vue'
import type { Ref } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

export function useUsers() {
  const users = ref<User[]>([])
  const loading = ref(false)
  const error = ref<string | null>(null)

  const fetchUsers = async () => {
    loading.value = true
    try {
      const response = await fetch('/api/users')
      users.value = await response.json()
    } catch (e) {
      error.value = 'Failed to fetch users'
    } finally {
      loading.value = false
    }
  }

  const updateUser = async (id: number, data: Partial<User>) => {
    // Implementation
  }

  const deleteUser = async (id: number) => {
    // Implementation
  }

  return {
    users,
    loading,
    error,
    fetchUsers,
    updateUser,
    deleteUser
  }
}
```

```typescript
// composables/useAnalytics.ts
export function useAnalytics() {
  // Analytics logic
  return {
    // ...
  }
}
```

```vue
<!-- UserDashboard.vue - 50 lines ✅ -->
<script setup lang="ts">
import { onMounted } from 'vue'
import { useUsers } from '@/composables/useUsers'
import { useAnalytics } from '@/composables/useAnalytics'
import UserList from '@/components/UserList.vue'
import AnalyticsPanel from '@/components/AnalyticsPanel.vue'

const { users, fetchUsers, updateUser, deleteUser } = useUsers()
const { analytics, fetchAnalytics } = useAnalytics()

onMounted(async () => {
  await Promise.all([fetchUsers(), fetchAnalytics()])
})
</script>

<template>
  <div class="dashboard">
    <UserList
      :users="users"
      @update="updateUser"
      @delete="deleteUser"
    />
    <AnalyticsPanel :data="analytics" />
  </div>
</template>
```

**Sabab:**
- **Reusability:** Composables boshqa componentlarda qayta ishlatiladi
- **Testability:** Har bir composable alohida test qilinadi
- **Maintainability:** Kichik fayllarni boshqarish oson
- **Code Organization:** Har bir logic o'z joyida
- **Team Collaboration:** Bir nechta developer parallel ishlashi mumkin
- **Single Responsibility:** Har bir component bitta vazifani bajaradi

---

## Composables Pattern

### ❌ NOTO'G'RI - Composable reusability yo'q

```vue
<!-- UserList.vue -->
<script setup lang="ts">
const users = ref<User[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

const fetchUsers = async () => {
  loading.value = true
  try {
    const response = await fetch('/api/users')
    users.value = await response.json()
  } catch (e) {
    error.value = 'Failed to fetch users'
  } finally {
    loading.value = false
  }
}

onMounted(fetchUsers)
</script>

<!-- ProductList.vue - XUDDI SHU CODE TAKRORLANGAN ❌ -->
<script setup lang="ts">
const products = ref<Product[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

const fetchProducts = async () => {
  loading.value = true
  try {
    const response = await fetch('/api/products')
    products.value = await response.json()
  } catch (e) {
    error.value = 'Failed to fetch products'
  } finally {
    loading.value = false
  }
}

onMounted(fetchProducts)
</script>
```

### ✅ TO'G'RI - Generic composable

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
    } catch (e) {
      error.value = e as Error
      onError?.(e as Error)
    } finally {
      loading.value = false
    }
  }

  const refresh = () => execute()

  if (immediate) {
    execute()
  }

  return {
    data: readonly(data),
    loading: readonly(loading),
    error: readonly(error),
    execute,
    refresh
  }
}
```

```vue
<!-- UserList.vue ✅ -->
<script setup lang="ts">
import { useAsyncData } from '@/composables/useAsyncData'

const { data: users, loading, error, refresh } = useAsyncData(
  () => fetch('/api/users').then(r => r.json()),
  {
    onSuccess: (data) => console.log('Users loaded:', data.length),
    onError: (err) => console.error('Failed to load users', err)
  }
)
</script>

<!-- ProductList.vue ✅ -->
<script setup lang="ts">
import { useAsyncData } from '@/composables/useAsyncData'

const { data: products, loading, error } = useAsyncData(
  () => fetch('/api/products').then(r => r.json())
)
</script>
```

### ✅ Advanced Composables

```typescript
// composables/usePagination.ts
export function usePagination<T>(
  fetchFn: (page: number, pageSize: number) => Promise<{ items: T[], total: number }>,
  options = { pageSize: 10 }
) {
  const currentPage = ref(1)
  const pageSize = ref(options.pageSize)
  const items = ref<T[]>([])
  const total = ref(0)
  const loading = ref(false)

  const totalPages = computed(() => Math.ceil(total.value / pageSize.value))

  const fetchPage = async () => {
    loading.value = true
    try {
      const result = await fetchFn(currentPage.value, pageSize.value)
      items.value = result.items
      total.value = result.total
    } finally {
      loading.value = false
    }
  }

  const goToPage = (page: number) => {
    if (page >= 1 && page <= totalPages.value) {
      currentPage.value = page
      fetchPage()
    }
  }

  const nextPage = () => goToPage(currentPage.value + 1)
  const prevPage = () => goToPage(currentPage.value - 1)

  watch([pageSize], () => {
    currentPage.value = 1
    fetchPage()
  })

  fetchPage()

  return {
    items: readonly(items),
    loading: readonly(loading),
    currentPage: readonly(currentPage),
    totalPages,
    total: readonly(total),
    goToPage,
    nextPage,
    prevPage
  }
}
```

**Sabab:**
- **DRY Principle:** Bir marta yozish, ko'p joyda ishlatish
- **Testability:** Composable alohida test qilinadi
- **Type Safety:** Generic types to'liq type safety
- **Flexibility:** Options pattern flexible API beradi
- **Maintainability:** Bir joyda o'zgartirish barcha joyga ta'sir qiladi

---

## Component Naming

### ❌ NOTO'G'RI - Component naming va folder structure

```
components/
├── btn.vue                    ❌ Noma'lum abbreviation
├── Card.vue                   ❌ Generic name
├── userTable.vue              ❌ camelCase fayl nomi
├── user-profile-settings.vue  ❌ Juda uzun
└── page-header.vue            ❌ Generic name
```

### ✅ TO'G'RI - Clear naming va folder grouping

```
components/
├── common/
│   ├── BaseButton.vue         ✅ Base prefix
│   ├── BaseInput.vue
│   └── BaseCard.vue
├── user/
│   ├── UserList.vue           ✅ Feature-based grouping
│   ├── UserCard.vue           ✅ PascalCase
│   └── UserProfileForm.vue    ✅ Descriptive name
└── layout/
    ├── AppHeader.vue          ✅ App prefix
    ├── AppSidebar.vue
    └── AppFooter.vue
```

**Naming Conventions:**

```typescript
// ✅ Good component names
BaseButton.vue      // Base components
BaseInput.vue
BaseCard.vue

AppHeader.vue       // App-wide components
AppSidebar.vue
AppFooter.vue

UserList.vue        // Feature components
UserCard.vue
UserProfileForm.vue

ProductCard.vue
ProductDetail.vue
ProductFilter.vue

// ❌ Bad component names
Button.vue          // Too generic
UI.vue              // Meaningless
Comp1.vue           // Non-descriptive
user_card.vue       // snake_case
userCard.vue        // camelCase
```

**Sabab:**
- **Consistency:** Bir xil naming pattern
- **Discoverability:** Component topish oson
- **Autocomplete:** IDE da component name autocomplete
- **Scalability:** Feature bo'yicha gruppalash kengayishga yordam beradi
- **Base Components:** `Base` prefix - umumiy, qayta ishlatiladigan
- **App Components:** `App` prefix - layout va app-wide
- **Feature Components:** Feature nomi bilan boshlanadi

---

## Prop Validation

### ❌ NOTO'G'RI - Prop validation yo'q

```vue
<script setup lang="ts">
// Hech qanday validation yo'q ❌
const props = defineProps<{
  size: string
  count: number
  user: object
}>()

// User xato qiymat bersa - runtime error ❌
</script>
```

### ✅ TO'G'RI - To'liq prop validation

```vue
<script setup lang="ts">
import { type PropType } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

// Type + Runtime validation ✅
const props = withDefaults(
  defineProps<{
    size: 'small' | 'medium' | 'large'
    count: number
    items: string[]
    user: User
    status?: 'active' | 'inactive'
  }>(),
  {
    size: 'medium',
    items: () => [],
    status: 'active'
  }
)

// Yoki runtime validator bilan ✅
defineProps({
  size: {
    type: String as PropType<'small' | 'medium' | 'large'>,
    default: 'medium',
    validator: (value: string) => {
      return ['small', 'medium', 'large'].includes(value)
    }
  },
  count: {
    type: Number,
    required: true,
    validator: (value: number) => value >= 0
  },
  email: {
    type: String,
    validator: (value: string) => {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
    }
  }
})
</script>
```

### ✅ Polymorphic Components

```vue
<script setup lang="ts">
// Polymorphic component props ✅
interface BaseProps {
  variant: 'primary' | 'secondary'
  size: 'sm' | 'md' | 'lg'
}

interface LinkProps extends BaseProps {
  as: 'a'
  href: string
  target?: string
}

interface ButtonProps extends BaseProps {
  as: 'button'
  type?: 'button' | 'submit' | 'reset'
  disabled?: boolean
}

type Props = LinkProps | ButtonProps

const props = defineProps<Props>()

// Type guard
const isLink = (props: Props): props is LinkProps => {
  return props.as === 'a'
}
</script>

<template>
  <a
    v-if="isLink(props)"
    :href="props.href"
    :target="props.target"
  >
    <slot />
  </a>
  <button
    v-else
    :type="props.type"
    :disabled="props.disabled"
  >
    <slot />
  </button>
</template>
```

**Sabab:**
- **Type Safety:** Compile time + runtime validation
- **Developer Experience:** IDE autocomplete va error checking
- **User Error Prevention:** Noto'g'ri props ishlatilsa warning
- **Documentation:** Props interface documentation vazifasini bajaradi
- **Polymorphic Components:** As prop pattern flexible API beradi

---

## Event Naming

### ❌ NOTO'G'RI - Event naming inconsistent

```vue
<script setup lang="ts">
// Inconsistent event names ❌
const emit = defineEmits<{
  (e: 'click'): void // DOM event bilan konflikt
  (e: 'change', value: string): void
  (e: 'deleteItem', id: number): void // camelCase
  (e: 'item-updated', item: Item): void // kebab-case
}>()
</script>

<template>
  <button @click="emit('click')">Click</button>
  <!-- Parent da @click vs @deleteItem vs @item-updated - inconsistent! -->
</template>
```

### ✅ TO'G'RI - Consistent event naming

```vue
<script setup lang="ts">
// Consistent naming convention ✅
const emit = defineEmits<{
  // update:* pattern for v-model
  (e: 'update:modelValue', value: string): void
  (e: 'update:title', value: string): void

  // Action verbs for events
  (e: 'submit', data: FormData): void
  (e: 'delete', id: number): void
  (e: 'cancel'): void

  // Lifecycle events
  (e: 'ready'): void
  (e: 'loading', isLoading: boolean): void
  (e: 'error', error: Error): void
}>()
</script>

<template>
  <!-- Consistent usage -->
  <button @click="emit('delete', item.id)">Delete</button>
  <button @click="emit('cancel')">Cancel</button>
</template>
```

**Event Naming Conventions:**

```typescript
// ✅ Good event names
'update:modelValue'  // v-model standard
'submit'             // Action
'delete'             // Action
'create'             // Action
'change'             // State change
'select'             // User interaction
'ready'              // Lifecycle
'error'              // Error state

// ❌ Bad event names
'click'              // Too generic, conflicts with DOM
'onClick'            // camelCase instead of kebab-case
'item-was-deleted'   // Past tense
'deleteTheItem'      // Too verbose
'del'                // Abbreviation
```

**Event Payload Patterns:**

```vue
<script setup lang="ts">
// Single value ✅
const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()

// Multiple related values - object ✅
const emit = defineEmits<{
  (e: 'submit', payload: {
    data: FormData
    meta: { timestamp: Date }
  }): void
}>()

// Event object pattern ✅
interface DeleteEvent {
  id: number
  confirmed: boolean
  reason?: string
}

const emit = defineEmits<{
  (e: 'delete', event: DeleteEvent): void
}>()

// Usage
emit('delete', {
  id: 123,
  confirmed: true,
  reason: 'User requested'
})
</script>
```

**Sabab:**
- **Consistency:** Bir xil naming convention
- **Clarity:** Event nomi nima qilishini aniq bildiradi
- **No Conflicts:** DOM event'lar bilan konflikt yo'q
- **Discoverability:** IDE autocomplete yaxshi ishlaydi
- **Type Safety:** Event payload to'liq typed

---

## Slots Pattern

### ✅ Named Slots with Types

```vue
<!-- BaseCard.vue -->
<script setup lang="ts">
interface Props {
  title?: string
  loading?: boolean
}

const props = defineProps<Props>()

// Slot types (Vue 3.3+)
defineSlots<{
  default(props: {}): any
  header(props: { title: string }): any
  footer(props: { close: () => void }): any
}>()
</script>

<template>
  <div class="card">
    <div v-if="$slots.header" class="card-header">
      <slot name="header" :title="title ?? 'Default Title'" />
    </div>

    <div class="card-body">
      <div v-if="loading">Loading...</div>
      <slot v-else />
    </div>

    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer" :close="() => console.log('Close')" />
    </div>
  </div>
</template>
```

```vue
<!-- Usage -->
<template>
  <BaseCard title="User Profile" :loading="false">
    <template #header="{ title }">
      <h2>{{ title }}</h2>
    </template>

    <template #default>
      <p>Card content goes here</p>
    </template>

    <template #footer="{ close }">
      <button @click="close">Close</button>
    </template>
  </BaseCard>
</template>
```

### ✅ Renderless Components

```vue
<!-- FetchData.vue - Renderless component -->
<script setup lang="ts">
import { ref, watch } from 'vue'

interface Props {
  url: string
}

const props = defineProps<Props>()

const data = ref(null)
const loading = ref(false)
const error = ref<Error | null>(null)

const fetchData = async () => {
  loading.value = true
  error.value = null

  try {
    const response = await fetch(props.url)
    data.value = await response.json()
  } catch (e) {
    error.value = e as Error
  } finally {
    loading.value = false
  }
}

watch(() => props.url, fetchData, { immediate: true })

defineSlots<{
  default(props: {
    data: any
    loading: boolean
    error: Error | null
    refresh: () => void
  }): any
}>()
</script>

<template>
  <slot
    :data="data"
    :loading="loading"
    :error="error"
    :refresh="fetchData"
  />
</template>
```

```vue
<!-- Usage -->
<template>
  <FetchData url="/api/users" v-slot="{ data, loading, error, refresh }">
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">Error: {{ error.message }}</div>
    <div v-else>
      <div v-for="user in data" :key="user.id">
        {{ user.name }}
      </div>
      <button @click="refresh">Refresh</button>
    </div>
  </FetchData>
</template>
```

**Sabab:**
- **Named Slots:** Multiple injection points
- **Scoped Slots:** Pass data to slot content
- **Renderless Components:** Logic reuse without UI
- **Type Safety:** Slot props typed (Vue 3.3+)
- **Flexibility:** Consumer controls rendering

---

## Xulosa

Component tashkiloti best practices:

1. **Kichik componentlar** - Single responsibility principle
2. **Composables** - Logic reuse va separation
3. **Clear naming** - PascalCase, descriptive names
4. **Folder grouping** - Feature-based organization
5. **Prop validation** - Type + runtime validation
6. **Event naming** - Consistent, action verbs
7. **Slots pattern** - Named, scoped, renderless

**Component hierarchy:**
```
Base components (BaseButton, BaseInput)
    ↓
Feature components (UserCard, ProductList)
    ↓
Page components (UserDashboardPage)
```

**File size guidelines:**
- Component: < 200 lines
- Composable: < 150 lines
- Agar ko'p bo'lsa - ajratish kerak!
