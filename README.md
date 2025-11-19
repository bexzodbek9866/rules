# Frontend Development Guide

## Mundarija
1. [Loyiha Strukturasi](#loyiha-strukturasi)
2. [Vue 3 Composition API Qoidalari](#vue-3-composition-api-qoidalari)
3. [TypeScript Best Practices](#typescript-best-practices)
   - Type Assertions va Validation
   - Utility Types
   - Generic Types
   - Const Assertions
4. [Template Qoidalari](#template-qoidalari)
   - Template Logic
   - v-if va v-for
   - v-html Security
   - Event Handlers
   - Optional Chaining
   - Method Chaining
   - v-model Modifiers
5. [State Management](#state-management)
6. [Component Tashkiloti](#component-tashkiloti)
   - Composables
   - Naming Conventions
   - Prop Validation
   - Event Naming
7. [Performance Optimization](#performance-optimization)
   - Reactivity Optimization
   - Computed vs Watch
   - v-show vs v-if
   - KeepAlive
   - Virtual Scrolling
   - Debounce va Throttle
8. [Code Quality](#code-quality)
9. [Quasar Specific Guidelines](#quasar-specific-guidelines)
---

## Loyiha Strukturasi

### Tavsiya etilgan folder struktura

```
src/
├── assets/              # Statik fayllar (images, fonts, etc)
├── components/          # Reusable components
│   ├── common/         # Umumiy komponentlar (Button, Input, etc)
│   ├── layout/         # Layout komponentlar (Header, Footer, Sidebar)
│   └── features/       # Feature-specific komponentlar
├── composables/        # Reusable composition functions
├── layouts/            # Quasar layout files
├── pages/              # Route pages (SFC)
├── router/             # Vue Router configuration
├── stores/             # Pinia stores
├── services/           # API va backend services
├── types/              # TypeScript type definitions
├── utils/              # Helper functions
├── constants/          # Constants va enums
```

**Sabab:** Bu struktura:
- **Scalability:** Loyiha o'sishi bilan oson boshqariladi
- **Maintainability:** Har bir fayl o'z joyida, osongina topiladi
- **Team collaboration:** Jamoa a'zolari qayerda nima borligini tezda tushunadi
- **Feature isolation:** Har bir feature mustaqil rivojlantirilishi mumkin

---

## Vue 3 Composition API Qoidalari

### ❌ NOTO'G'RI - Options API ishlatish (yangi loyihalarda)

```vue
<script lang="ts">
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  }
}
</script>
```

### ✅ TO'G'RI - Composition API + `<script setup>`

```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
const increment = () => {
  count.value++
}
</script>
```

**Sabab:**
- **Type Safety:** TypeScript bilan yaxshi ishlaydi, `this` context muammosi yo'q
- **Tree-shaking:** Faqat ishlatilgan Vue features bundle ga kiradi
- **Code organization:** Logic bo'yicha guruplash oson (Options API da esa type bo'yicha - data, methods, computed)
- **Reusability:** Composables orqali logic qayta ishlatish ancha oson
- **Performance:** Minifikatsiya va optimization yaxshiroq

---

### ❌ NOTO'G'RI - Reactive ma'lumotlarni to'g'ri ishlatmaslik

```vue
<script setup lang="ts">
let count = 0 // Bu reactive emas!

const increment = () => {
  count++ // UI yangilanmaydi
}
</script>
```

### ✅ TO'G'RI - `ref` yoki `reactive` ishlatish

```vue
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)

const increment = () => {
  count.value++ // UI yangilanadi
}
</script>
```

**Sabab:**
- **Reactivity:** Vue faqat `ref`/`reactive` orqali o'zgarishlarni kuzatadi
- **Performance tracking:** Vue qaysi componentlarni qayta render qilish kerakligini biladi

---

### ❌ NOTO'G'RI - Ref va Reactive aralashtirib yuborish

```vue
<script setup lang="ts">
import { reactive, ref } from 'vue'

// Primitiv qiymat uchun reactive - XATO!
const count = reactive(0) // Bu ishlamaydi

// Object uchun ref - keraksiz
const user = ref({ name: '', age: 0 }) // .value orqali kirish kerak
</script>
```

### ✅ TO'G'RI - Har birini o'z joyida ishlatish

```vue
<script setup lang="ts">
import { reactive, ref } from 'vue'

// Primitiv qiymatlar uchun ref
const count = ref(0)
const message = ref('')

// Obyektlar uchun reactive (yoki ref)
const user = reactive({
  name: '',
  age: 0
})

// Yoki interface bilan
interface User {
  name: string
  age: number
}

const userRef = ref<User>({
  name: '',
  age: 0
})
</script>
```

**Sabab:**
- **ref:** Primitiv qiymatlar (string, number, boolean) va yagona obyektlar uchun. `.value` orqali kirish kerak.
- **reactive:** Ko'p propertyga ega obyektlar uchun. To'g'ridan-to'g'ri property access.
- **Type safety:** TypeScript bilan yaxshi type inference

---

## TypeScript Best Practices

### ❌ NOTO'G'RI - `any` type ishlatish

```vue
<script setup lang="ts">
const fetchData = async (): Promise<any> => { // any - XATO!
  const response = await fetch('/api/users')
  return response.json()
}

const userData: any = await fetchData() // Type safety yo'q
</script>
```

### ✅ TO'G'RI - Aniq type definition

```vue
<script setup lang="ts">
interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user' | 'guest'
}

interface ApiResponse<T> {
  data: T
  status: number
  message: string
}

const fetchData = async (): Promise<ApiResponse<User[]>> => {
  const response = await fetch('/api/users')
  return response.json()
}

const userData = await fetchData() // To'liq type safety
// userData.data[0].name - autocomplete ishlaydi
</script>
```

**Sabab:**
- **Type Safety:** Compile time da xatolarni aniqlash
- **IDE Support:** Intellisense va autocomplete
- **Documentation:** Type o'zi documentation vazifasini bajaradi
- **Refactoring:** Type o'zgarganda barcha joylar avtomatik tekshiriladi
- **Runtime Errors Prevention:** Ko'p xatolar development davrida topiladi

---

### ❌ NOTO'G'RI - Props va Emits ni type qilmaslik

```vue
<script setup lang="ts">
const props = defineProps({
  title: String, // Type safety zaif
  count: Number
})

const emit = defineEmits(['update', 'delete']) // Payload type yo'q
</script>
```

### ✅ TO'G'RI - TypeScript bilan to'liq type safety

```vue
<script setup lang="ts">
interface Props {
  title: string
  count: number
  items?: string[] // Optional property
  status: 'active' | 'inactive' | 'pending' // Union type
}

interface Emits {
  (e: 'update', id: number, data: Partial<Props>): void
  (e: 'delete', id: number): void
  (e: 'change', value: string): void
}

const props = withDefaults(defineProps<Props>(), {
  items: () => [], // Default qiymat
  status: 'active'
})

const emit = defineEmits<Emits>()

// Ishlatish
const handleUpdate = () => {
  emit('update', 123, { title: 'New Title' }) // Type checked!
  // emit('update', 'wrong') // ❌ Compile error
}
</script>
```

**Sabab:**
- **Type Safety:** Props va emit qilingan eventlar to'liq type-safe
- **Better DX:** IDE autocomplete va error detection
- **API Documentation:** Interface o'zi component API ni tasvirlaydi
- **Compile-time validation:** Xatolar runtime dan oldin aniqlanadi

---

### ❌ NOTO'G'RI - Type assertion bilan type safety buzish

```vue
<script setup lang="ts">
// Unsafe type assertion ❌
const data = fetchData() as MyType // Runtime da boshqa type bo'lishi mumkin

// Unknown typedan to'g'ridan-to'g'ri cast ❌
const apiResponse = await fetch('/api') as ApiResponse

// Non-null assertion operator keraksiz ❌
const element = document.getElementById('app')!.innerHTML // null bo'lishi mumkin

// Double assertion ❌
const value = (unknownValue as any) as SpecificType
</script>
```

### ✅ TO'G'RI - Type guard va validation

```vue
<script setup lang="ts">
import { z } from 'zod' // Runtime validation

// Zod schema - runtime type checking ✅
const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  role: z.enum(['admin', 'user', 'guest'])
})

type User = z.infer<typeof UserSchema>

// Type guard ✅
function isUser(obj: unknown): obj is User {
  try {
    UserSchema.parse(obj)
    return true
  } catch {
    return false
  }
}

// Safe API call ✅
const fetchUser = async (id: number): Promise<User> => {
  const response = await fetch(`/api/users/${id}`)
  const data = await response.json()

  // Runtime validation
  if (!isUser(data)) {
    throw new Error('Invalid user data')
  }

  return data // Type safe!
}

// Safe DOM access ✅
const element = document.getElementById('app')
if (element) {
  element.innerHTML = 'Content'
}

// Yoki optional chaining ✅
const content = document.getElementById('app')?.innerHTML ?? ''

// Type guard bilan safe narrowing ✅
function processValue(value: unknown) {
  if (typeof value === 'string') {
    return value.toUpperCase() // string methods safe
  }

  if (typeof value === 'number') {
    return value.toFixed(2) // number methods safe
  }

  throw new Error('Unexpected value type')
}
</script>
```

**Sabab:**
- **Type Assertion:** Faqat compiler ga ta'sir qiladi, runtime da tekshirmaydi
- **Runtime Validation:** Zod kabi kutubxonalar runtime da validate qiladi
- **Type Guards:** Compile time va runtime type safety
- **Non-null Assertion (!):** Null bo'lsa crash, avval check qilish kerak
- **Unknown Type:** `unknown` dan type narrowing bilan safe conversion

---

### ❌ NOTO'G'RI - Utility types ishlatmaslik

```vue
<script setup lang="ts">
interface User {
  id: number
  name: string
  email: string
  password: string
  createdAt: Date
}

// Barcha fieldlarni qayta yozish ❌
interface UpdateUser {
  id: number
  name: string
  email: string
  password: string
  // createdAt yo'q
}

// Optional fieldlarni manual ❌
interface PartialUser {
  id?: number
  name?: string
  email?: string
  password?: string
}

// Readonly manual ❌
interface ReadonlyUser {
  readonly id: number
  readonly name: string
  readonly email: string
  readonly password: string
}
</script>
```

### ✅ TO'G'RI - TypeScript Utility Types

```vue
<script setup lang="ts">
interface User {
  id: number
  name: string
  email: string
  password: string
  role: 'admin' | 'user'
  createdAt: Date
  updatedAt: Date
}

// Partial - barcha property optional ✅
type PartialUser = Partial<User>

// Pick - faqat kerakli propertylar ✅
type UserCredentials = Pick<User, 'email' | 'password'>

// Omit - ba'zi propertylarni olib tashlash ✅
type UserWithoutPassword = Omit<User, 'password'>
type CreateUserDTO = Omit<User, 'id' | 'createdAt' | 'updatedAt'>

// Required - barcha property required ✅
type RequiredUser = Required<PartialUser>

// Readonly - barcha property readonly ✅
type ImmutableUser = Readonly<User>

// Record - dynamic keys ✅
type UserMap = Record<number, User> // { [id: number]: User }
type RolePermissions = Record<User['role'], string[]>

// Extract va Exclude ✅
type AdminRole = Extract<User['role'], 'admin'> // 'admin'
type NonAdminRole = Exclude<User['role'], 'admin'> // 'user'

// ReturnType - function return type ✅
const getUser = (): User => ({} as User)
type UserReturn = ReturnType<typeof getUser> // User

// Parameters - function parameters ✅
const updateUser = (id: number, data: Partial<User>) => {}
type UpdateUserParams = Parameters<typeof updateUser> // [number, Partial<User>]

// Amaliy misol ✅
interface ApiResponse<T> {
  data: T
  status: number
  message: string
}

// Component props
interface UserCardProps {
  user: Readonly<UserWithoutPassword> // Password yo'q, readonly
  onUpdate: (data: Partial<CreateUserDTO>) => void
  permissions: RolePermissions[User['role']]
}

const props = defineProps<UserCardProps>()
</script>
```

**Sabab:**
- **DRY Principle:** Type'larni qayta yozmaslik
- **Maintainability:** Asosiy interface o'zgarganda, utility types avtomatik yangilanadi
- **Expressiveness:** Code niyatini aniq ifodalaydi
- **Type Safety:** Compile-time type checking

**Utility Types ro'yxati:**
- `Partial<T>` - barcha property optional
- `Required<T>` - barcha property required
- `Readonly<T>` - barcha property readonly
- `Pick<T, K>` - faqat K propertylar
- `Omit<T, K>` - K propertylardan tashqari
- `Record<K, T>` - K key va T value type
- `Extract<T, U>` - T dan faqat U ga mos keluvchi
- `Exclude<T, U>` - T dan U ni olib tashlash
- `ReturnType<T>` - function return type
- `Parameters<T>` - function parameters tuple

---

### ❌ NOTO'G'RI - Generic types ishlatmaslik

```vue
<script setup lang="ts">
// Har bir type uchun alohida interface ❌
interface StringResponse {
  data: string
  status: number
}

interface NumberResponse {
  data: number
  status: number
}

interface UserResponse {
  data: User
  status: number
}

// Array typelarni takrorlash ❌
interface StringArray {
  items: string[]
  total: number
}

interface NumberArray {
  items: number[]
  total: number
}
</script>
```

### ✅ TO'G'RI - Generic types ishlatish

```vue
<script setup lang="ts">
// Generic interface ✅
interface ApiResponse<T> {
  data: T
  status: number
  message: string
  timestamp: Date
}

interface PaginatedResponse<T> {
  items: T[]
  total: number
  page: number
  pageSize: number
}

// Ishlatish
type UserResponse = ApiResponse<User>
type UsersListResponse = ApiResponse<PaginatedResponse<User>>
type StringDataResponse = ApiResponse<string>

// Generic function ✅
const fetchData = async <T>(
  url: string,
  validator: (data: unknown) => data is T
): Promise<ApiResponse<T>> => {
  const response = await fetch(url)
  const data = await response.json()

  if (!validator(data)) {
    throw new Error('Invalid data format')
  }

  return {
    data,
    status: response.status,
    message: 'Success',
    timestamp: new Date()
  }
}

// Ishlatish
const userData = await fetchData<User>('/api/user/1', isUser)

// Generic composable ✅
function useResource<T>(
  fetchFn: () => Promise<T>
) {
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

  return {
    data: readonly(data),
    loading: readonly(loading),
    error: readonly(error),
    execute
  }
}

// Ishlatish
const { data: users, loading, execute } = useResource<User[]>(
  () => fetch('/api/users').then(r => r.json())
)

// Generic component props ✅
interface TableColumn<T> {
  key: keyof T
  label: string
  formatter?: (value: T[keyof T]) => string
  sortable?: boolean
}

interface TableProps<T> {
  data: T[]
  columns: TableColumn<T>[]
  onRowClick?: (row: T) => void
}

// Type-safe table component
const tableProps = defineProps<TableProps<User>>()

// Constrained generics ✅
interface HasId {
  id: number | string
}

function findById<T extends HasId>(items: T[], id: T['id']): T | undefined {
  return items.find(item => item.id === id)
}

// Multiple type parameters ✅
function mapObject<K extends string | number, V, R>(
  obj: Record<K, V>,
  mapper: (value: V, key: K) => R
): Record<K, R> {
  const result = {} as Record<K, R>

  for (const key in obj) {
    result[key] = mapper(obj[key], key)
  }

  return result
}
</script>
```

**Sabab:**
- **Code Reusability:** Bir generic type ko'p hollarda ishlatiladi
- **Type Safety:** Generic parameter type safety beradi
- **Flexibility:** Turli xil type'lar bilan ishlaydi
- **IntelliSense:** IDE autocomplete ishlaydi
- **Constraints:** `extends` bilan type constraint qo'shish mumkin

---

### ❌ NOTO'G'RI - Const assertions va type inference ishlatmaslik

```vue
<script setup lang="ts">
// Manual type definition ❌
const statuses: string[] = ['pending', 'active', 'completed']
type Status = 'pending' | 'active' | 'completed' // Takrorlanish

// Object literal type yo'q ❌
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retryAttempts: 3
}
// config.apiUrl = 'new-url' // O'zgartirish mumkin ❌

// Enum ishlatish (ko'p hollarda keraksiz) ❌
enum UserRole {
  Admin = 'admin',
  User = 'user',
  Guest = 'guest'
}
</script>
```

### ✅ TO'G'RI - Const assertions va type inference

```vue
<script setup lang="ts">
// Const assertion - literal types ✅
const STATUSES = ['pending', 'active', 'completed'] as const
type Status = typeof STATUSES[number] // 'pending' | 'active' | 'completed'

// Readonly object ✅
const CONFIG = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retryAttempts: 3
} as const

type Config = typeof CONFIG
// CONFIG.apiUrl = 'new' // ❌ Error: readonly property

// Const assertion bilan object ✅
const USER_ROLES = {
  ADMIN: 'admin',
  USER: 'user',
  GUEST: 'guest'
} as const

type UserRole = typeof USER_ROLES[keyof typeof USER_ROLES]
// 'admin' | 'user' | 'guest'

// Type inference from value ✅
const defaultUser = {
  name: 'Guest',
  role: 'guest' as const,
  permissions: ['read']
}

type DefaultUser = typeof defaultUser

// Satisfies operator (TypeScript 4.9+) ✅
type RouteConfig = {
  path: string
  component: string
  meta?: {
    requiresAuth: boolean
    roles: readonly string[]
  }
}

const routes = [
  {
    path: '/dashboard',
    component: 'Dashboard',
    meta: {
      requiresAuth: true,
      roles: ['admin', 'user']
    }
  },
  {
    path: '/login',
    component: 'Login'
  }
] as const satisfies readonly RouteConfig[]

// Type safe va literal types saqlanadi!
// routes[0].path - literal type '/dashboard'
// routes[0].meta.roles - readonly ['admin', 'user']

// Const assertion va generic ✅
function createStore<T extends Record<string, unknown>>(
  initialState: T
) {
  const state = reactive(initialState)

  return {
    state: readonly(state),
    update(key: keyof T, value: T[typeof key]) {
      state[key] = value
    }
  }
}

const userStore = createStore({
  name: 'John',
  age: 30,
  role: 'admin' as const
})

// Type-safe access
userStore.update('name', 'Jane') // ✅
// userStore.update('name', 123) // ❌ Error
</script>
```

**Sabab:**
- **Const Assertion (`as const`):**
  - Literal types saqlanadi
  - Barcha propertylar readonly
  - Array tuple ga aylanadi
- **Type Inference:** Qo'shimcha type yozmaslik
- **Satisfies:** Type checking + literal type retention
- **Enum vs Const Objects:**
  - Enum: Extra runtime code generate qiladi
  - Const object: Zero runtime overhead, literal types
- **DRY:** Type va value bir joyda

---

## Template Qoidalari

### ❌ NOTO'G'RI - Template ichida murakkab logic

```vue
<template>
  <!-- Filter va method chaqirish template da - XATO! -->
  <div
    v-for="item in items.filter(i => i.status === 'active' && i.price > 100)"
    :key="item.id"
  >
    {{ item.name }}
  </div>

  <!-- Inline complex calculations -->
  <p>{{ user.firstName + ' ' + user.lastName + ' (' + user.age + ')' }}</p>

  <!-- Template da ternary ichida ternary -->
  <span :class="status === 'active' ? 'green' : status === 'pending' ? 'yellow' : 'red'">
    {{ status }}
  </span>
</template>
```

### ✅ TO'G'RI - Logic ni computed/methods ga ajratish

```vue
<script setup lang="ts">
import { computed, ref } from 'vue'

interface Item {
  id: number
  name: string
  status: 'active' | 'inactive'
  price: number
}

interface User {
  firstName: string
  lastName: string
  age: number
}

const items = ref<Item[]>([])
const user = ref<User>({
  firstName: 'John',
  lastName: 'Doe',
  age: 30
})
const status = ref<'active' | 'pending' | 'rejected'>('active')

// Computed properties - cached va reactive
const activeExpensiveItems = computed(() =>
  items.value.filter(i => i.status === 'active' && i.price > 100)
)

const fullUserInfo = computed(() =>
  `${user.value.firstName} ${user.value.lastName} (${user.value.age})`
)

const statusClass = computed(() => {
  const statusMap = {
    active: 'green',
    pending: 'yellow',
    rejected: 'red'
  }
  return statusMap[status.value]
})
</script>

<template>
  <div v-for="item in activeExpensiveItems" :key="item.id">
    {{ item.name }}
  </div>

  <p>{{ fullUserInfo }}</p>

  <span :class="statusClass">{{ status }}</span>
</template>
```

**Sabab:**
- **Performance:**
  - `v-for` ichida `.filter()` har bir re-render da qayta ishlaydi (JUDA SEKIN!)
  - `computed` natijani cache qiladi, dependency o'zgarmasa qayta hisoblamaydi
  - Template ichidagi har bir expression har safar baholanadi
- **Readability:** Template tozaroq, logic ajratilgan
- **Testability:** Computed properties alohida test qilish mumkin
- **Maintainability:** Logic bir joyda, o'zgartirish oson
- **Debugging:** Console.log computed ichida ishlaydi, template da emas

**Performance ta'siri:**
```
1000 ta item bilan ro'yxat:
❌ Template filter: ~50ms har bir render da
✅ Computed filter: ~50ms birinchi marta, keyin ~0ms (cached)
```

---

### ❌ NOTO'G'RI - v-if va v-for bir elementda

```vue
<template>
  <!-- v-if va v-for bitta element da - XATO! -->
  <div
    v-for="item in items"
    v-if="item.isVisible"
    :key="item.id"
  >
    {{ item.name }}
  </div>
</template>
```

### ✅ TO'G'RI - Computed yoki wrapper element

```vue
<script setup lang="ts">
import { computed, ref } from 'vue'

interface Item {
  id: number
  name: string
  isVisible: boolean
}

const items = ref<Item[]>([])

const visibleItems = computed(() =>
  items.value.filter(item => item.isVisible)
)
</script>

<template>
  <!-- Variant 1: Computed (BEST) -->
  <div v-for="item in visibleItems" :key="item.id">
    {{ item.name }}
  </div>

  <!-- Variant 2: Wrapper element (agar dynamic condition bo'lsa) -->
  <template v-for="item in items" :key="item.id">
    <div v-if="item.isVisible">
      {{ item.name }}
    </div>
  </template>
</template>
```

**Sabab:**
- **Vue 3 Priority:** `v-if` `v-for` dan yuqori prioritetga ega, item o'zgaruvchisi mavjud emas
- **Performance:** Har bir render da barcha itemlar iterate qilinadi, keyin condition tekshiriladi
- **Computed approach:** Faqat visible itemlar iterate qilinadi
- **Memory:** Keraksiz DOM nodelar yaratilmaydi

---

### ❌ NOTO'G'RI - v-for da index ni key sifatida ishlatish

```vue
<template>
  <!-- Index key sifatida - KO'P HOLLARDA XATO! -->
  <div v-for="(item, index) in items" :key="index">
    <input v-model="item.name" />
  </div>
</template>
```

### ✅ TO'G'RI - Unique ID ishlatish

```vue
<script setup lang="ts">
import { ref } from 'vue'

interface Item {
  id: string // yoki number, lekin UNIQUE bo'lishi kerak
  name: string
}

const items = ref<Item[]>([
  { id: 'uuid-1', name: 'Item 1' },
  { id: 'uuid-2', name: 'Item 2' }
])
</script>

<template>
  <div v-for="item in items" :key="item.id">
    <input v-model="item.name" />
  </div>
</template>
```

**Sabab:**
- **DOM Reuse Issues:** Index o'zgarganda Vue noto'g'ri elementlarni reuse qiladi
- **State Bugs:** Input value, checkbox state noto'g'ri element bilan bog'lanadi
- **Animation Problems:** Transition/animation noto'g'ri ishlaydi
- **Performance:** Vue element yangilashni optimize qila olmaydi

**Misolda muammo:**
```
Items: [A, B, C] (index: 0, 1, 2)
B ni o'chirsak: [A, C] (index: 0, 1)
Vue index 1 ni yangilaydi (oldin B, hozir C)
Lekin input value B da qoladi! ❌
```

---

### ❌ NOTO'G'RI - Direct DOM manipulation

```vue
<script setup lang="ts">
import { onMounted } from 'vue'

onMounted(() => {
  // Direct DOM manipulation - XATO!
  document.getElementById('myElement')!.style.color = 'red'
  document.querySelector('.title')!.textContent = 'New Title'
})
</script>

<template>
  <div id="myElement">Content</div>
  <h1 class="title">Old Title</h1>
</template>
```

### ✅ TO'G'RI - Vue reactive system ishlatish

```vue
<script setup lang="ts">
import { ref } from 'vue'

const textColor = ref('red')
const title = ref('New Title')

// Agar DOM ga to'g'ridan-to'g'ri access kerak bo'lsa
const myElementRef = ref<HTMLDivElement | null>(null)

onMounted(() => {
  // Faqat zarur hollarda (masalan, 3rd party library)
  if (myElementRef.value) {
    // myElementRef.value.focus()
  }
})
</script>

<template>
  <div ref="myElementRef" :style="{ color: textColor }">
    Content
  </div>
  <h1>{{ title }}</h1>
</template>
```

**Sabab:**
- **Reactivity Loss:** Direct DOM manipulation Vue ning reactivity systemini bypass qiladi
- **Unpredictable Behavior:** Vue virtual DOM bilan sync emas
- **Hard to Debug:** State va UI mos kelmaydi
- **SSR Issues:** Server-side rendering ishlamaydi
- **Testing:** Component test qilish qiyin

---

### ❌ NOTO'G'RI - v-html bilan unsanitized content

```vue
<template>
  <!-- XSS vulnerability! ❌ -->
  <div v-html="userInput"></div>

  <!-- Backend dan kelgan HTML ❌ -->
  <div v-html="apiResponse.html"></div>
</template>

<script setup lang="ts">
const userInput = ref('<script>alert("XSS")</script>')
const apiResponse = ref({ html: '<img src=x onerror=alert(1)>' })
</script>
```

### ✅ TO'G'RI - Text interpolation yoki sanitization

```vue
<template>
  <!-- Text interpolation - xavfsiz ✅ -->
  <div>{{ userInput }}</div>

  <!-- Agar HTML kerak bo'lsa - sanitize qilish ✅ -->
  <div v-html="sanitizedHtml"></div>

  <!-- Yoki markdown ishlatish ✅ -->
  <div v-html="parsedMarkdown"></div>
</template>

<script setup lang="ts">
import DOMPurify from 'dompurify'
import { marked } from 'marked'

const userInput = ref('User text')

// HTML sanitization
const sanitizedHtml = computed(() =>
  DOMPurify.sanitize(rawHtml.value, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
)

// Markdown parsing (safer)
const parsedMarkdown = computed(() =>
  DOMPurify.sanitize(marked(userMarkdown.value))
)
</script>
```

**Sabab:**
- **XSS Prevention:** `v-html` bilan user input bevosita render qilish XSS xavfi
- **Security First:** Har doim user input ni trust qilmaslik kerak
- **Text Interpolation:** `{{ }}` avtomatik escape qiladi
- **Sanitization:** Agar HTML kerak bo'lsa, DOMPurify kabi kutubxona ishlatish
- **Markdown:** HTML o'rniga markdown safer alternativa

---

### ❌ NOTO'G'RI - Inline event handlers bilan murakkab logic

```vue
<template>
  <!-- Murakkab logic inline ❌ -->
  <button @click="
    if (user.role === 'admin') {
      items.forEach(item => {
        item.status = 'approved'
        saveItem(item)
      })
      notify('Approved')
    } else {
      notify('No permission')
    }
  ">
    Approve All
  </button>

  <!-- Multiple function calls ❌ -->
  <input
    @input="
      validateInput($event);
      updateModel($event);
      trackChange($event)
    "
  />
</template>
```

### ✅ TO'G'RI - Method yoki function reference

```vue
<template>
  <!-- Method reference ✅ -->
  <button @click="handleApproveAll">
    Approve All
  </button>

  <!-- Bitta function ✅ -->
  <input @input="handleInput" />

  <!-- Argument uzatish kerak bo'lsa ✅ -->
  <button
    v-for="item in items"
    :key="item.id"
    @click="() => deleteItem(item.id)"
  >
    Delete
  </button>
</template>

<script setup lang="ts">
const handleApproveAll = () => {
  if (user.value.role !== 'admin') {
    notify('No permission')
    return
  }

  items.value.forEach(item => {
    item.status = 'approved'
    saveItem(item)
  })

  notify('All items approved')
}

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement
  validateInput(target.value)
  updateModel(target.value)
  trackChange(target.value)
}
</script>
```

**Sabab:**
- **Readability:** Inline logic template ni iflos qiladi
- **Testability:** Inline code ni test qilish mumkin emas
- **Reusability:** Bir xil logic ni qayta ishlatish qiyin
- **Debugging:** Inline code da breakpoint qo'yish qiyin
- **Performance:** Arrow function `v-for` ichida har safar yangi function yaratadi

---

### ❌ NOTO'G'RI - Template da optional chaining ko'p ishlatish

```vue
<template>
  <!-- Ko'p optional chaining - bad sign ❌ -->
  <div>{{ user?.profile?.settings?.theme?.primaryColor }}</div>

  <div>{{ data?.items?.[0]?.meta?.createdBy?.name }}</div>

  <!-- Default value inline ❌ -->
  <p>{{ user?.name || 'Guest' }}</p>
  <p>{{ items?.length || 0 }}</p>
</template>
```

### ✅ TO'G'RI - Computed properties bilan default values

```vue
<template>
  <div>{{ primaryColor }}</div>
  <div>{{ firstItemCreator }}</div>
  <p>{{ userName }}</p>
  <p>{{ itemCount }}</p>
</template>

<script setup lang="ts">
import { computed } from 'vue'

// Default values bilan computed
const primaryColor = computed(() =>
  user.value?.profile?.settings?.theme?.primaryColor ?? '#000000'
)

const firstItemCreator = computed(() =>
  data.value?.items?.[0]?.meta?.createdBy?.name ?? 'Unknown'
)

const userName = computed(() => user.value?.name ?? 'Guest')

const itemCount = computed(() => items.value?.length ?? 0)

// Yoki yaxshiroq - type guard va interface
interface UserProfile {
  settings: {
    theme: {
      primaryColor: string
    }
  }
}

const hasValidProfile = (user: any): user is { profile: UserProfile } => {
  return user?.profile?.settings?.theme?.primaryColor !== undefined
}

const safeColor = computed(() => {
  if (hasValidProfile(user.value)) {
    return user.value.profile.settings.theme.primaryColor
  }
  return '#000000'
})
</script>
```

**Sabab:**
- **Data Structure:** Ko'p optional chaining - data structure muammosi belgisi
- **Type Safety:** Computed ichida to'g'ri type checking
- **Default Values:** Bir joyda default value management
- **Performance:** Template expression har safar baholanadi
- **Maintainability:** Default valueni bir joyda o'zgartirish oson

---

### ❌ NOTO'G'RI - Template da array/object methods zanjiri

```vue
<template>
  <!-- Method chaining template da ❌ -->
  <div v-for="item in items
    .filter(i => i.active)
    .map(i => ({ ...i, formatted: formatDate(i.date) }))
    .sort((a, b) => a.priority - b.priority)
    .slice(0, 10)"
    :key="item.id"
  >
    {{ item.name }}
  </div>

  <!-- Complex calculation ❌ -->
  <p>
    Total: {{
      orders
        .filter(o => o.status === 'completed')
        .reduce((sum, o) => sum + o.total, 0)
        .toFixed(2)
    }}
  </p>
</template>
```

### ✅ TO'G'RI - Computed properties bilan

```vue
<template>
  <div v-for="item in topPriorityItems" :key="item.id">
    {{ item.name }} - {{ item.formatted }}
  </div>

  <p>Total: {{ completedOrdersTotal }}</p>
</template>

<script setup lang="ts">
import { computed } from 'vue'

interface Item {
  id: number
  name: string
  active: boolean
  date: Date
  priority: number
}

interface FormattedItem extends Item {
  formatted: string
}

// Cached computed property ✅
const topPriorityItems = computed<FormattedItem[]>(() => {
  return items.value
    .filter(i => i.active)
    .map(i => ({
      ...i,
      formatted: formatDate(i.date)
    }))
    .sort((a, b) => a.priority - b.priority)
    .slice(0, 10)
})

const completedOrdersTotal = computed(() => {
  const total = orders.value
    .filter(o => o.status === 'completed')
    .reduce((sum, o) => sum + o.total, 0)

  return total.toFixed(2)
})

// Yoki alohida computed'lar
const completedOrders = computed(() =>
  orders.value.filter(o => o.status === 'completed')
)

const totalAmount = computed(() =>
  completedOrders.value.reduce((sum, o) => sum + o.total, 0)
)

const formattedTotal = computed(() => totalAmount.value.toFixed(2))
</script>
```

**Sabab:**
- **Performance:**
  - Template method chaining har render da qayta ishlaydi
  - Computed faqat dependency o'zgarganda ishlaydi
  - 1000 itemda: ~50ms vs ~0ms (cached)
- **Readability:** Template tozaroq
- **Debugging:** Computed ichida console.log va breakpoint
- **Composition:** Kichik computed'larni birlashtirish mumkin

---

### ❌ NOTO'G'RI - v-model modifiers noto'g'ri ishlatish

```vue
<template>
  <!-- Manual trim va type conversion ❌ -->
  <input
    :value="username"
    @input="username = $event.target.value.trim()"
  />

  <input
    :value="age"
    @input="age = parseInt($event.target.value)"
  />

  <!-- Lazy update manual ❌ -->
  <input
    :value="searchQuery"
    @change="searchQuery = $event.target.value"
  />
</template>
```

### ✅ TO'G'RI - v-model modifiers ishlatish

```vue
<template>
  <!-- Built-in modifiers ✅ -->
  <input v-model.trim="username" />

  <input v-model.number="age" type="number" />

  <input v-model.lazy="searchQuery" />

  <!-- Multiple modifiers ✅ -->
  <input v-model.trim.lazy="description" />

  <!-- Custom modifier (Vue 3.4+) ✅ -->
  <CustomInput v-model:title.capitalize="postTitle" />
</template>

<script setup lang="ts">
import { ref } from 'vue'

const username = ref('')
const age = ref(0)
const searchQuery = ref('')
const description = ref('')
const postTitle = ref('')
</script>
```

**Custom v-model modifier:**
```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
interface Props {
  title: string
  titleModifiers?: { capitalize?: boolean }
}

const props = defineProps<Props>()
const emit = defineEmits<{
  (e: 'update:title', value: string): void
}>()

const handleInput = (event: Event) => {
  let value = (event.target as HTMLInputElement).value

  if (props.titleModifiers?.capitalize) {
    value = value.charAt(0).toUpperCase() + value.slice(1)
  }

  emit('update:title', value)
}
</script>

<template>
  <input :value="title" @input="handleInput" />
</template>
```

**Sabab:**
- **Built-in Modifiers:**
  - `.trim` - whitespace olib tashlaydi
  - `.number` - string ni number ga convert qiladi
  - `.lazy` - `input` o'rniga `change` event ishlatadi
- **Less Code:** Manual handling o'rniga modifier
- **Consistency:** Standard Vue modifiers
- **Custom Modifiers:** Component library uchun reusable modifiers

---

## State Management

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

**Sabab:**
- **Type Safety:** Inject qilingan qiymat to'liq typed
- **Autocomplete:** IDE autocomplete ishlaydi
- **Refactoring Safety:** Type o'zgarganda barcha joylar tekshiriladi
- **Documentation:** InjectionKey o'zi documentation

---

## Component Tashkiloti

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

**Sabab:**
- **Consistency:** Bir xil naming pattern
- **Discoverability:** Component topish oson
- **Autocomplete:** IDE da component name autocomplete
- **Scalability:** Feature bo'yicha gruppalash kengayishga yordam beradi
- **Base Components:** `Base` prefix - umumiy, qayta ishlatiladigan
- **App Components:** `App` prefix - layout va app-wide
- **Feature Components:** Feature nomi bilan boshlanadi

---

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

**Advanced prop patterns:**
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

**More advanced composables:**
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

**Event naming conventions:**
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

**Event payload patterns:**
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

## Performance Optimization

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

### ❌ NOTO'G'RI - Computed vs Watch noto'g'ri ishlatish

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

**Qachon qaysi birini ishlatish:**
```typescript
// ✅ Computed
const fullName = computed(() => `${first.value} ${last.value}`)
const filteredItems = computed(() => items.value.filter(i => i.active))
const totalPrice = computed(() => cart.value.reduce((sum, i) => sum + i.price, 0))

// ✅ Watch
watch(searchQuery, async (query) => {
  await fetchSearchResults(query) // API call
})

watch(userSettings, (settings) => {
  localStorage.setItem('settings', JSON.stringify(settings)) // Side effect
})
```

---

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

**Qachon qaysi birini ishlatish:**
```vue
<!-- v-if: Kamdan-kam o'zgaradi -->
<AdminPanel v-if="isAdmin" />
<FeatureFlag v-if="features.newUI" />
<Modal v-if="isOpen" />

<!-- v-show: Tez-tez toggle -->
<Dropdown v-show="isDropdownOpen" />
<Tooltip v-show="isHovered" />
<Sidebar v-show="isSidebarVisible" />
```

---

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

**Advanced KeepAlive usage:**
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

**Qachon ishlatish:**
- Tab/Step navigation
- Multi-step forms
- List va Detail view o'rtasida
- Frequently accessed routes

**Qachon ishlatmaslik:**
- Har doim fresh data kerak bo'lsa
- Memory constraint bor bo'lsa
- Component state yo'qolishi kerak bo'lsa (logout, reset)

---

### ❌ NOTO'G'RI - Virtual scrolling ishlatmaslik (large lists)

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

### ✅ TO'G'RI - Virtual scrolling (vue-virtual-scroller yoki custom)

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

**Custom virtual scroll implementation:**
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
- **Virtual Scrolling:**
  - Faqat ko'rinadigan itemlar DOM da
  - Scroll qilganda items recycled/reused
  - Constant DOM node count
- **When to use:**
  - 100+ items
  - Large datasets (pagination mumkin emas bo'lsa)
  - Table/list views
  - Chat messages, logs

---

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

  <!-- Resize event listener ❌ -->
  <div ref="container">
    {{ windowWidth }} x {{ windowHeight }}
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
  // Expensive calculation
  recalculateLayout()
}
</script>
```

### ✅ TO'G'RI - Debounce va Throttle ishlatish

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
  // Expensive calculation
  recalculateLayout()
  // Maximum 10 calls per second (instead of 100+)
}, 100)

// Manual implementation (VueUse yo'q bo'lsa)
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
</script>
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
- **Debounce:**
  - Wait until user finishes action
  - Good for: search, autocomplete, validation
  - "hello" typed → 1 API call instead of 5
- **Throttle:**
  - Limit execution frequency
  - Good for: scroll, resize, mousemove
  - 1000 events/sec → 10 events/sec
- **Performance:**
  - Reduces API calls, CPU usage, memory
  - Improves UX (less jank)
- **VueUse:** Built-in utilities, tested, TypeScript support

---

## Code Quality

### ❌ NOTO'G'RI - ESLint va TypeScript xatolarini ignore qilish

```vue
<script setup lang="ts">
// @ts-ignore - XATO! ❌
const user: any = fetchUser()

// eslint-disable-next-line - XATO! ❌
const result = dangerousFunction()

// Type assertion noto'g'ri ishlatish
const element = document.getElementById('app') as HTMLDivElement // null bo'lishi mumkin! ❌
</script>
```

### ✅ TO'G'RI - Xatolarni to'g'ri tuzatish

```vue
<script setup lang="ts">
import type { User } from '@/types'

// Type assertion to'g'ri ✅
const user = await fetchUser() as User

// Null check ✅
const element = document.getElementById('app')
if (element instanceof HTMLDivElement) {
  // Safe to use element here
}

// Better approach - ref with type ✅
const appRef = ref<HTMLDivElement | null>(null)

// Type guard ✅
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj
  )
}

const data = await fetchData()
if (isUser(data)) {
  // data is User type here
}
</script>
```

**Sabab:**
- **@ts-ignore:** Type xatosini berkitadi, lekin muammo qoladi
- **Type Safety:** Runtime da xato bo'lishi mumkin
- **Code Quality:** Xatolar development da topilishi kerak
- **Maintainability:** Ignore comment refactoring qilishni qiyinlashtiradi

---

## Quasar Specific Guidelines

### ✅ Quasar Component Props to'g'ri ishlatish

```vue
<script setup lang="ts">
import { QTable, type QTableColumn } from 'quasar'
import type { User } from '@/types'

// To'liq type safety ✅
const columns: QTableColumn<User>[] = [
  {
    name: 'name',
    label: 'Name',
    field: 'name',
    align: 'left',
    sortable: true
  },
  {
    name: 'email',
    label: 'Email',
    field: 'email',
    align: 'left'
  }
]

const rows = ref<User[]>([])
</script>

<template>
  <q-table
    :columns="columns"
    :rows="rows"
    row-key="id"
    :pagination="{ rowsPerPage: 10 }"
  />
</template>
```

**Sabab:**
- **Type Safety:** Quasar types to'liq TypeScript support beradi
- **Autocomplete:** IDE da Quasar component props autocomplete
- **Compile-time Validation:** Xato props compile time da aniqlanadi

---

### ✅ Quasar Notify, Dialog, Loading to'g'ri ishlatish

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

const showSuccessNotify = () => {
  $q.notify({
    type: 'positive',
    message: 'Operation successful',
    position: 'top',
    timeout: 2000
  })
}

const showConfirmDialog = async () => {
  $q.dialog({
    title: 'Confirm',
    message: 'Are you sure?',
    cancel: true,
    persistent: true
  }).onOk(() => {
    // User confirmed
  })
}

const performLongOperation = async () => {
  $q.loading.show({
    message: 'Loading...'
  })

  try {
    await someAsyncOperation()
  } finally {
    $q.loading.hide()
  }
}
</script>
```

**Sabab:**
- **Quasar Plugins:** `useQuasar()` orqali barcha Quasar plugins ga access
- **Consistent UX:** Quasar components consistent UI/UX beradi
- **Accessibility:** Quasar components accessibility built-in

---

### ✅ Quasar Layouts va Responsive Design

```vue
<!-- layouts/MainLayout.vue -->
<template>
  <q-layout view="hHh lpR fFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          @click="toggleLeftDrawer"
          class="lt-md"
        />
        <q-toolbar-title>My App</q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      :breakpoint="1024"
      bordered
    >
      <q-list>
        <q-item clickable to="/dashboard">
          <q-item-section avatar>
            <q-icon name="dashboard" />
          </q-item-section>
          <q-item-section>Dashboard</q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const leftDrawerOpen = ref(false)

const toggleLeftDrawer = () => {
  leftDrawerOpen.value = !leftDrawerOpen.value
}
</script>
```

**Sabab:**
- **Quasar Layout System:** Responsive layout built-in
- **Breakpoints:** `lt-md`, `gt-sm` - Quasar breakpoint classes
- **Drawer Behavior:** Automatic drawer behavior mobile/desktop


---

## Qo'shimcha Resources

- [Vue 3 Official Docs](https://vuejs.org/)
- [TypeScript Vue Plugin](https://github.com/vuejs/language-tools)
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Quasar Framework](https://quasar.dev/)
- [Vue Test Utils](https://test-utils.vuejs.org/)
- [Vitest](https://vitest.dev/)

---

