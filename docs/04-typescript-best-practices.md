# TypeScript Best Practices

## Mundarija
- [Type Assertions va Validation](#type-assertions-va-validation)
- [Utility Types](#utility-types)
- [Generic Types](#generic-types)
- [Const Assertions va Type Inference](#const-assertions-va-type-inference)
- [Props va Emits Type Safety](#props-va-emits-type-safety)
- [Advanced TypeScript Patterns (Senior Level)](#advanced-typescript-patterns-senior-level)
  - [Conditional Types](#conditional-types)
  - [Template Literal Types](#template-literal-types)
  - [Branded Types (Nominal Typing)](#branded-types-nominal-typing)
  - [Discriminated Unions (Advanced)](#discriminated-unions-advanced)
  - [Mapped Types (Advanced)](#mapped-types-advanced)
  - [Type Testing](#type-testing)
  - [Performance Considerations](#performance-considerations)

---

## Type Assertions va Validation

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

## Utility Types

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

## Generic Types

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

## Const Assertions va Type Inference

### ❌ NOTO'G'RI - Const assertions ishlatmaslik

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

## Props va Emits Type Safety

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

## Prop Validation Advanced

### ✅ Runtime validator bilan

```vue
<script setup lang="ts">
import { type PropType } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

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

### ✅ Polymorphic component props

```vue
<script setup lang="ts">
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

---

## Advanced TypeScript Patterns (Senior Level)

### Conditional Types

Conditional types TypeScript ning eng kuchli xususiyatlaridan biri. Type-level programming uchun ishlatiladi.

```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false

type A = IsString<string> // true
type B = IsString<number> // false

// Practical example: Extract return type
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T

type AsyncResult = UnwrapPromise<Promise<User>> // User
type SyncResult = UnwrapPromise<User> // User

// Advanced: Deep partial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P]
}

interface User {
  id: number
  profile: {
    name: string
    settings: {
      theme: string
    }
  }
}

// Barcha nested propertylar optional
type PartialUser = DeepPartial<User>

// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never

type StringOrNumber = string | number
type ArrayType = ToArray<StringOrNumber> // string[] | number[]
```

### Template Literal Types

Route typing va string manipulation uchun juda foydali.

```typescript
// Route typing
type Route = '/users' | '/products' | '/settings'
type DynamicRoute = `/users/${string}` | `/products/${string}`

// String manipulation
type Uppercase<S extends string> = Intrinsic
type Lowercase<S extends string> = Intrinsic

type EventName = 'click' | 'focus' | 'blur'
type EventHandler = `on${Capitalize<EventName>}` // 'onClick' | 'onFocus' | 'onBlur'

// Practical: API endpoints type-safe
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE'
type Endpoint = '/users' | '/products'
type ApiRoute = `${HttpMethod} ${Endpoint}`

// Usage
const route: ApiRoute = 'GET /users' // ✅
// const invalid: ApiRoute = 'GET /invalid' // ❌ Error

// Advanced: Extract path parameters
type ExtractParams<Path extends string> =
  Path extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? Param | ExtractParams<`/${Rest}`>
    : Path extends `${infer _Start}:${infer Param}`
    ? Param
    : never

type UserParams = ExtractParams<'/users/:id/posts/:postId'> // 'id' | 'postId'

// Type-safe route params
interface RouteParams {
  [K in ExtractParams<'/users/:id/posts/:postId'>]: string
}

const params: RouteParams = {
  id: '123',
  postId: '456'
} // ✅
```

### Branded Types (Nominal Typing)

Runtime da bir xil lekin semantically turli xil typelar uchun.

```typescript
// Branded type helper
type Brand<K, T> = K & { __brand: T }

// Domain types
type UserId = Brand<number, 'UserId'>
type ProductId = Brand<number, 'ProductId'>
type Euros = Brand<number, 'Euros'>
type Dollars = Brand<number, 'Dollars'>

// Helper functions
const createUserId = (id: number): UserId => id as UserId
const createProductId = (id: number): ProductId => id as ProductId

// Usage
const userId = createUserId(123)
const productId = createProductId(456)

// ❌ Type mismatch prevents bugs
// const wrongAssignment: UserId = productId // Error!

// Practical example: Money handling
type Money<Currency extends string> = Brand<number, Currency>

type USD = Money<'USD'>
type EUR = Money<'EUR'>

const usd = (amount: number): USD => amount as USD
const eur = (amount: number): EUR => amount as EUR

function convertUsdToEur(amount: USD, rate: number): EUR {
  return eur(amount * rate)
}

const dollars = usd(100)
const euros = convertUsdToEur(dollars, 0.85)

// ❌ Can't mix currencies
// const mixed: USD = euros // Error!

// Validation bilan branded types
type Email = Brand<string, 'Email'>

function createEmail(input: string): Email | null {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(input) ? (input as Email) : null
}

// Type-safe va validated
function sendEmail(to: Email, subject: string): void {
  // Email guaranteed to be valid
  console.log(`Sending to ${to}`)
}

const email = createEmail('user@example.com')
if (email) {
  sendEmail(email, 'Hello') // ✅
}
```

### Discriminated Unions (Advanced)

Type-safe state machines va error handling uchun.

```typescript
// Result type pattern
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E }

// Usage
async function fetchUser(id: number): Promise<Result<User>> {
  try {
    const response = await fetch(`/api/users/${id}`)
    const data = await response.json()
    return { success: true, data }
  } catch (error) {
    return { success: false, error: error as Error }
  }
}

// Type-safe handling
const result = await fetchUser(123)

if (result.success) {
  console.log(result.data.name) // Type: User
} else {
  console.error(result.error.message) // Type: Error
}

// Advanced: API state machine
type ApiState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error }

function useApiState<T>() {
  const state = ref<ApiState<T>>({ status: 'idle' })

  const execute = async (fetcher: () => Promise<T>) => {
    state.value = { status: 'loading' }

    try {
      const data = await fetcher()
      state.value = { status: 'success', data }
    } catch (error) {
      state.value = { status: 'error', error: error as Error }
    }
  }

  return { state: readonly(state), execute }
}

// Type-safe usage
const { state, execute } = useApiState<User[]>()

watch(state, (newState) => {
  // TypeScript narrows type automatically
  if (newState.status === 'success') {
    console.log(newState.data) // Type: User[]
  } else if (newState.status === 'error') {
    console.error(newState.error) // Type: Error
  }
})
```

### Mapped Types (Advanced)

Type transformation uchun kuchli pattern.

```typescript
// Make all properties optional deep
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object
    ? DeepPartial<T[P]>
    : T[P]
}

// Make all properties readonly deep
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P]
}

// Require selected properties
type RequireKeys<T, K extends keyof T> = Required<Pick<T, K>> & Omit<T, K>

interface User {
  id?: number
  name?: string
  email?: string
}

type UserWithEmail = RequireKeys<User, 'email'>
// { email: string; id?: number; name?: string }

// Create immutable version
type Immutable<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? Immutable<T[P]>
    : T[P]
}

const config: Immutable<Config> = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
}

// config.apiUrl = 'new-url' // ❌ Error: readonly

// Advanced: Getters type
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P]
}

interface State {
  count: number
  name: string
}

type StateGetters = Getters<State>
// { getCount: () => number; getName: () => string }
```

### Type Testing

Type correctness ni test qilish - production code uchun muhim.

```typescript
// Type assertion helpers
type Expect<T extends true> = T
type Equal<X, Y> = (<T>() => T extends X ? 1 : 2) extends <T>() => T extends Y
  ? 1
  : 2
  ? true
  : false

// Test examples
type TestResult = Expect<Equal<string, string>> // ✅
type TestResult2 = Expect<Equal<string, number>> // ❌ Error

// Test utility types
type TestPartial = Expect<
  Equal<Partial<{ a: string }>, { a?: string }>
>

// Test custom types
type ExtractId<T extends { id: any }> = T['id']

type Test1 = Expect<Equal<ExtractId<{ id: number }>, number>>
type Test2 = Expect<Equal<ExtractId<{ id: string }>, string>>

// Using tsd library (npm install -D tsd)
import { expectType, expectError } from 'tsd'

const userId: UserId = createUserId(123)
expectType<UserId>(userId) // ✅

const productId: ProductId = createProductId(456)
expectError<UserId>(productId) // ✅ Should error
```

### Performance Considerations

Type inference performance muhim - complex typelar compile time ni sekinlashtiradi.

```typescript
// ❌ Slow - recursive depth too high
type BadRecursive<T, Depth extends any[] = []> =
  Depth['length'] extends 50
    ? T
    : BadRecursive<T, [...Depth, any]>

// ✅ Good - limited recursion depth
type GoodRecursive<T, Depth extends any[] = []> =
  Depth['length'] extends 10
    ? T
    : T extends object
    ? { [K in keyof T]: GoodRecursive<T[K], [...Depth, any]> }
    : T

// ❌ Slow - too many union members
type SlowUnion = 'a' | 'b' | 'c' | /* ... 1000+ members */

// ✅ Good - reasonable union size
type FastUnion = 'small' | 'medium' | 'large'

// Tip: Use index types instead of unions for large sets
interface SizeMap {
  small: string
  medium: string
  large: string
}

type Size = keyof SizeMap // Same result, better performance
```

---

## Xulosa

TypeScript Best Practices (Senior Level):

### Basic Principles
1. **Har doim type definition yozing** - `any` type ishlatmang
2. **Type guard va runtime validation** - Zod kabi library ishlatish
3. **Utility types maksimal darajada** - DRY principle
4. **Generic types reusable code** uchun
5. **Const assertions** - Literal types va readonly
6. **Props va Emits to'liq type-safe**

### Advanced Patterns
7. **Conditional types** - Type-level programming
8. **Template literal types** - Route va string typing
9. **Branded types** - Nominal typing for domain models
10. **Discriminated unions** - Type-safe state machines
11. **Mapped types** - Advanced type transformations
12. **Type testing** - Type correctness verification

### Best Practices
- Development vaqtida xatolarni topish
- Refactoring osonlashtirish
- Code documentation (types as documentation)
- IDE support maksimal darajada
- Runtime errors kamaytirish
- Performance-aware type design
- Type testing for critical types
