# State Management

## Mundarija
- [Prop Drilling vs Pinia](#prop-drilling-vs-pinia)
- [Pinia Store Pattern](#pinia-store-pattern)
- [Provide/Inject Pattern](#provideinject-pattern)
- [Store Composition](#store-composition)
- [State Persistence](#state-persistence)

---

## Prop Drilling vs Pinia

### ❌ NOTO'G'RI - Prop drilling (5+ level)

```vue
<!-- App.vue -->
<template>
  <LayoutComponent :user="user" :theme="theme" />
</template>

<!-- LayoutComponent.vue -->
<template>
  <HeaderComponent :user="user" :theme="theme" />
</template>

<!-- HeaderComponent.vue -->
<template>
  <NavigationComponent :user="user" :theme="theme" />
</template>

<!-- NavigationComponent.vue -->
<template>
  <UserMenuComponent :user="user" :theme="theme" />
</template>

<!-- UserMenuComponent.vue - faqat shu yerda ishlatiladi! -->
<template>
  <div>{{ user.name }} - {{ theme }}</div>
</template>
```

### ✅ TO'G'RI - Pinia store ishlatish

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
}

export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)
  const theme = ref<'light' | 'dark'>('light')

  const isAdmin = computed(() => user.value?.role === 'admin')

  const setUser = (newUser: User) => {
    user.value = newUser
  }

  const toggleTheme = () => {
    theme.value = theme.value === 'light' ? 'dark' : 'light'
  }

  return {
    user,
    theme,
    isAdmin,
    setUser,
    toggleTheme
  }
})
```

```vue
<!-- UserMenuComponent.vue -->
<script setup lang="ts">
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()
</script>

<template>
  <div>{{ userStore.user?.name }} - {{ userStore.theme }}</div>
  <button @click="userStore.toggleTheme">Toggle Theme</button>
</template>
```

**Sabab:**
- **Maintainability:** Prop drilling refactoring qilish qiyin
- **Performance:** Har bir level component re-render bo'lishi mumkin
- **Code Clarity:** Oraliq componentlar faqat props uzatish uchun ishlatiladi
- **Centralized State:** Bir joyda state management
- **DevTools:** Pinia DevTools bilan debugging oson
- **Type Safety:** To'liq TypeScript support

---

## Pinia Store Pattern

### Setup Stores (Recommended)

```typescript
// stores/auth.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { Ref, ComputedRef } from 'vue'

interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user' | 'guest'
}

interface AuthState {
  user: Ref<User | null>
  token: Ref<string | null>
  isAuthenticated: ComputedRef<boolean>
  isAdmin: ComputedRef<boolean>
}

export const useAuthStore = defineStore('auth', () => {
  // State
  const user = ref<User | null>(null)
  const token = ref<string | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  // Getters
  const isAuthenticated = computed(() => !!token.value)
  const isAdmin = computed(() => user.value?.role === 'admin')
  const userName = computed(() => user.value?.name ?? 'Guest')

  // Actions
  const login = async (email: string, password: string) => {
    loading.value = true
    error.value = null

    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      })

      const data = await response.json()

      if (!response.ok) {
        throw new Error(data.message)
      }

      user.value = data.user
      token.value = data.token

      // Save to localStorage
      localStorage.setItem('token', data.token)
    } catch (e) {
      error.value = (e as Error).message
      throw e
    } finally {
      loading.value = false
    }
  }

  const logout = () => {
    user.value = null
    token.value = null
    localStorage.removeItem('token')
  }

  const refreshUser = async () => {
    if (!token.value) return

    try {
      const response = await fetch('/api/auth/me', {
        headers: { Authorization: `Bearer ${token.value}` }
      })

      const data = await response.json()
      user.value = data.user
    } catch (e) {
      logout()
    }
  }

  // Initialize from localStorage
  const initAuth = () => {
    const savedToken = localStorage.getItem('token')
    if (savedToken) {
      token.value = savedToken
      refreshUser()
    }
  }

  return {
    // State
    user,
    token,
    loading,
    error,
    // Getters
    isAuthenticated,
    isAdmin,
    userName,
    // Actions
    login,
    logout,
    refreshUser,
    initAuth
  }
})
```

### Options Stores (Alternative)

```typescript
// stores/cart.ts
import { defineStore } from 'pinia'

interface Product {
  id: number
  name: string
  price: number
}

interface CartItem extends Product {
  quantity: number
}

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [] as CartItem[],
    loading: false
  }),

  getters: {
    totalItems: (state) =>
      state.items.reduce((sum, item) => sum + item.quantity, 0),

    totalPrice: (state) =>
      state.items.reduce((sum, item) => sum + item.price * item.quantity, 0),

    isEmpty: (state) => state.items.length === 0
  },

  actions: {
    addItem(product: Product) {
      const existingItem = this.items.find(item => item.id === product.id)

      if (existingItem) {
        existingItem.quantity++
      } else {
        this.items.push({ ...product, quantity: 1 })
      }
    },

    removeItem(productId: number) {
      const index = this.items.findIndex(item => item.id === productId)
      if (index > -1) {
        this.items.splice(index, 1)
      }
    },

    updateQuantity(productId: number, quantity: number) {
      const item = this.items.find(item => item.id === productId)
      if (item) {
        item.quantity = quantity
      }
    },

    clearCart() {
      this.items = []
    }
  }
})
```

**Sabab:**
- **Setup Stores:** Composition API style, more flexible
- **Options Stores:** Options API style, structured
- **Type Safety:** Full TypeScript inference
- **DevTools:** Time-travel debugging, state inspection
- **HMR:** Hot Module Replacement support

---

## Provide/Inject Pattern

### ❌ NOTO'G'RI - Provide/Inject ni type qilmaslik

```vue
<!-- Parent.vue -->
<script setup lang="ts">
import { provide } from 'vue'

provide('user', { name: 'John', age: 30 }) // Type yo'q!
</script>

<!-- Child.vue -->
<script setup lang="ts">
import { inject } from 'vue'

const user = inject('user') // any type ❌
</script>
```

### ✅ TO'G'RI - InjectionKey bilan type safety

```typescript
// types/injection-keys.ts
import type { InjectionKey, Ref } from 'vue'

export interface User {
  name: string
  age: number
  role: 'admin' | 'user'
}

export const UserKey: InjectionKey<Ref<User>> = Symbol('user')
```

```vue
<!-- Parent.vue -->
<script setup lang="ts">
import { provide, ref } from 'vue'
import { UserKey, type User } from '@/types/injection-keys'

const user = ref<User>({
  name: 'John',
  age: 30,
  role: 'admin'
})

provide(UserKey, user)
</script>

<!-- Child.vue -->
<script setup lang="ts">
import { inject } from 'vue'
import { UserKey } from '@/types/injection-keys'

const user = inject(UserKey) // Typed! Ref<User> | undefined

// Default qiymat bilan
const userWithDefault = inject(UserKey, ref({
  name: 'Guest',
  age: 0,
  role: 'user' as const
}))
</script>
```

**Advanced Provide/Inject Pattern:**

```typescript
// composables/useTheme.ts
import { provide, inject, ref, readonly, type InjectionKey, type Ref } from 'vue'

type Theme = 'light' | 'dark'

interface ThemeContext {
  theme: Readonly<Ref<Theme>>
  setTheme: (newTheme: Theme) => void
  toggleTheme: () => void
}

const ThemeKey: InjectionKey<ThemeContext> = Symbol('theme')

// Provider composable
export function provideTheme() {
  const theme = ref<Theme>('light')

  const setTheme = (newTheme: Theme) => {
    theme.value = newTheme
  }

  const toggleTheme = () => {
    theme.value = theme.value === 'light' ? 'dark' : 'light'
  }

  const themeContext: ThemeContext = {
    theme: readonly(theme),
    setTheme,
    toggleTheme
  }

  provide(ThemeKey, themeContext)

  return themeContext
}

// Consumer composable
export function useTheme(): ThemeContext {
  const themeContext = inject(ThemeKey)

  if (!themeContext) {
    throw new Error('useTheme must be used within a ThemeProvider')
  }

  return themeContext
}
```

```vue
<!-- App.vue (Provider) -->
<script setup lang="ts">
import { provideTheme } from '@/composables/useTheme'

const { theme, toggleTheme } = provideTheme()
</script>

<template>
  <div :class="`theme-${theme}`">
    <button @click="toggleTheme">Toggle Theme</button>
    <router-view />
  </div>
</template>

<!-- AnyChildComponent.vue (Consumer) -->
<script setup lang="ts">
import { useTheme } from '@/composables/useTheme'

const { theme } = useTheme()
</script>

<template>
  <div>Current theme: {{ theme }}</div>
</template>
```

**Sabab:**
- **Type Safety:** Inject qilingan qiymat to'liq typed
- **Autocomplete:** IDE autocomplete ishlaydi
- **Refactoring Safety:** Type o'zgarganda barcha joylar tekshiriladi
- **Documentation:** InjectionKey o'zi documentation
- **Error Handling:** Runtime error agar provide qilinmagan bo'lsa

---

## Store Composition

### ✅ Store'larni birlashtirib ishlatish

```typescript
// stores/todos.ts
import { defineStore } from 'pinia'
import { useAuthStore } from './auth'

interface Todo {
  id: number
  title: string
  completed: boolean
  userId: number
}

export const useTodosStore = defineStore('todos', () => {
  const authStore = useAuthStore() // Boshqa store'ni ishlatish

  const todos = ref<Todo[]>([])
  const loading = ref(false)

  // Faqat current user todoları
  const userTodos = computed(() => {
    if (!authStore.user) return []
    return todos.value.filter(todo => todo.userId === authStore.user!.id)
  })

  const fetchTodos = async () => {
    if (!authStore.isAuthenticated) {
      throw new Error('User not authenticated')
    }

    loading.value = true
    try {
      const response = await fetch('/api/todos', {
        headers: {
          Authorization: `Bearer ${authStore.token}`
        }
      })
      todos.value = await response.json()
    } finally {
      loading.value = false
    }
  }

  const addTodo = async (title: string) => {
    const response = await fetch('/api/todos', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${authStore.token}`
      },
      body: JSON.stringify({
        title,
        userId: authStore.user!.id
      })
    })

    const newTodo = await response.json()
    todos.value.push(newTodo)
  }

  return {
    todos,
    userTodos,
    loading,
    fetchTodos,
    addTodo
  }
})
```

### ✅ Shared composable in stores

```typescript
// composables/useApi.ts
import { ref } from 'vue'

export function useApi<T>(fetchFn: () => Promise<T>) {
  const data = ref<T | null>(null)
  const loading = ref(false)
  const error = ref<Error | null>(null)

  const execute = async () => {
    loading.value = true
    error.value = null

    try {
      data.value = await fetchFn()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }

  return { data, loading, error, execute }
}

// stores/products.ts
import { defineStore } from 'pinia'
import { useApi } from '@/composables/useApi'

export const useProductsStore = defineStore('products', () => {
  const { data: products, loading, error, execute: fetchProducts } = useApi(() =>
    fetch('/api/products').then(r => r.json())
  )

  return {
    products,
    loading,
    error,
    fetchProducts
  }
})
```

**Sabab:**
- **Modularity:** Har bir store o'z vazifasiga ega
- **Composition:** Store'larni birlashtirib ishlatish
- **Code Reuse:** Umumiy logic composable'larda
- **Separation of Concerns:** Har bir store bitta domain

---

## State Persistence

### ✅ LocalStorage bilan persist qilish

```typescript
// stores/settings.ts
import { defineStore } from 'pinia'
import { ref, watch } from 'vue'

interface Settings {
  theme: 'light' | 'dark'
  language: 'en' | 'uz' | 'ru'
  notifications: boolean
}

const DEFAULT_SETTINGS: Settings = {
  theme: 'light',
  language: 'uz',
  notifications: true
}

export const useSettingsStore = defineStore('settings', () => {
  // Load from localStorage
  const loadSettings = (): Settings => {
    const saved = localStorage.getItem('settings')
    if (saved) {
      try {
        return { ...DEFAULT_SETTINGS, ...JSON.parse(saved) }
      } catch {
        return DEFAULT_SETTINGS
      }
    }
    return DEFAULT_SETTINGS
  }

  const settings = ref<Settings>(loadSettings())

  // Auto-save to localStorage
  watch(
    settings,
    (newSettings) => {
      localStorage.setItem('settings', JSON.stringify(newSettings))
    },
    { deep: true }
  )

  const updateSettings = (partial: Partial<Settings>) => {
    settings.value = { ...settings.value, ...partial }
  }

  const resetSettings = () => {
    settings.value = DEFAULT_SETTINGS
  }

  return {
    settings,
    updateSettings,
    resetSettings
  }
})
```

### ✅ Pinia plugin bilan persistence

```typescript
// plugins/pinia-persist.ts
import { type PiniaPluginContext } from 'pinia'

export function piniaPersistedState(context: PiniaPluginContext) {
  const { store, options } = context

  // @ts-ignore - custom option
  if (options.persist) {
    const storageKey = `pinia-${store.$id}`

    // Load from storage
    const saved = localStorage.getItem(storageKey)
    if (saved) {
      store.$patch(JSON.parse(saved))
    }

    // Save to storage on change
    store.$subscribe(() => {
      localStorage.setItem(storageKey, JSON.stringify(store.$state))
    })
  }
}

// main.ts
import { createPinia } from 'pinia'
import { piniaPersistedState } from './plugins/pinia-persist'

const pinia = createPinia()
pinia.use(piniaPersistedState)

// stores/cart.ts
export const useCartStore = defineStore('cart', {
  // @ts-ignore
  persist: true,

  state: () => ({
    items: []
  }),
  // ...
})
```

**Yoki pinia-plugin-persistedstate package:**

```typescript
// npm install pinia-plugin-persistedstate

// main.ts
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

// stores/auth.ts
export const useAuthStore = defineStore('auth', {
  state: () => ({
    token: null,
    user: null
  }),

  persist: {
    key: 'auth-storage',
    storage: localStorage,
    paths: ['token', 'user'] // Faqat shu fieldlar persist qilinadi
  }
})
```

**Sabab:**
- **User Experience:** State browser refresh da saqlanadi
- **Selective Persistence:** Faqat kerakli state persist qilish
- **Storage Options:** localStorage, sessionStorage, cookies
- **Serialization:** Automatic JSON serialize/deserialize

---

## Xulosa

State Management best practices:

1. **Pinia for global state** - Prop drilling o'rniga
2. **Setup stores** - Composition API style
3. **Type safety** - InjectionKey bilan provide/inject
4. **Store composition** - Store'larni birlashtirib ishlatish
5. **Composables in stores** - Reusable logic
6. **State persistence** - localStorage/plugin
7. **Single responsibility** - Har bir store bitta domain

**Qachon qaysi birini ishlatish:**
- **Local state:** `ref`/`reactive` component ichida
- **Shared state (2-3 components):** Provide/Inject
- **Global state:** Pinia store
- **URL state:** Vue Router query params
