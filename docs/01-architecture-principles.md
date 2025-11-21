# Loyiha Strukturasi va Architecture Principles

## Mundarija
- [Loyiha Strukturasi](#loyiha-strukturasi)
- [SOLID Principles](#solid-principles)
- [Boshqa Architecture Principles](#boshqa-architecture-principles)
- [Folder Structure Best Practices](#folder-structure-best-practices)

---

## Loyiha Strukturasi

### Tavsiya etilgan folder struktura

```
src/
├── assets/              # Statik fayllar
│   ├── images/         # Rasm fayllar
│   ├── fonts/          # Font fayllar
│   ├── icons/          # Icon fayllar (SVG)
│   └── styles/         # Global styles (SCSS/CSS)
│       ├── variables.scss
│       ├── mixins.scss
│       └── global.scss
│
├── components/          # Reusable components
│   ├── common/         # Umumiy komponentlar (atomic design)
│   │   ├── BaseButton.vue
│   │   ├── BaseInput.vue
│   │   ├── BaseCard.vue
│   │   └── BaseModal.vue
│   ├── layout/         # Layout komponentlar
│   │   ├── AppHeader.vue
│   │   ├── AppFooter.vue
│   │   ├── AppSidebar.vue
│   │   └── AppNavigation.vue
│   └── features/       # Feature-specific komponentlar
│       ├── user/
│       │   ├── UserCard.vue
│       │   ├── UserList.vue
│       │   └── UserProfileForm.vue
│       └── product/
│           ├── ProductCard.vue
│           └── ProductList.vue
│
├── composables/        # Reusable composition functions
│   ├── useAuth.ts
│   ├── useApi.ts
│   ├── usePagination.ts
│   ├── useDebounce.ts
│   └── useForm.ts
│
├── layouts/            # Quasar layout files
│   ├── MainLayout.vue
│   ├── AuthLayout.vue
│   └── EmptyLayout.vue
│
├── pages/              # Route pages (SFC)
│   ├── auth/
│   │   ├── LoginPage.vue
│   │   └── RegisterPage.vue
│   ├── dashboard/
│   │   └── DashboardPage.vue
│   └── users/
│       ├── UsersPage.vue
│       └── UserDetailPage.vue
│
├── router/             # Vue Router configuration
│   ├── index.ts
│   ├── routes.ts
│   └── guards.ts       # Route guards
│
├── stores/             # Pinia stores
│   ├── user.ts
│   ├── auth.ts
│   └── products.ts
│
├── services/           # API va backend services
│   ├── api/
│   │   ├── axios.ts    # Axios instance
│   │   ├── user.api.ts
│   │   └── product.api.ts
│   └── storage/        # LocalStorage, SessionStorage wrapper
│       └── storage.service.ts
│
├── types/              # TypeScript type definitions
│   ├── models/         # Domain models
│   │   ├── user.ts
│   │   └── product.ts
│   ├── api/            # API response types
│   │   └── responses.ts
│   └── injection-keys.ts
│
├── utils/              # Helper functions
│   ├── formatters.ts   # Date, number, currency formatters
│   ├── validators.ts   # Form validators
│   └── helpers.ts      # General helpers
│
├── constants/          # Constants va enums
│   ├── api.constants.ts
│   ├── app.constants.ts
│   └── routes.constants.ts
│
└── plugins/            # Vue plugins
    ├── axios.ts
    └── i18n.ts
```

**Sabab:**
- **Scalability:** Loyiha o'sishi bilan oson boshqariladi
- **Maintainability:** Har bir fayl o'z joyida, osongina topiladi
- **Team collaboration:** Jamoa a'zolari qayerda nima borligini tezda tushunadi
- **Feature isolation:** Har bir feature mustaqil rivojlantirilishi mumkin
- **Separation of Concerns:** Har bir papka o'z mas'uliyatiga ega

---

## SOLID Principles

SOLID - object-oriented programming va software design uchun 5 ta asosiy printsip.

### S - Single Responsibility Principle (SRP)

**Printsip:** Har bir class/module/function faqat bitta mas'uliyatga ega bo'lishi kerak.

#### ❌ NOTO'G'RI - Ko'p mas'uliyat

```typescript
// user.service.ts
class UserService {
  // Database access ❌
  async getUser(id: number) {
    const db = await connectDB()
    return db.users.findById(id)
  }

  // Data validation ❌
  validateEmail(email: string) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }

  // UI formatting ❌
  formatUserName(user: User) {
    return `${user.firstName} ${user.lastName}`
  }

  // API call ❌
  async sendEmail(user: User) {
    return fetch('/api/email', {
      method: 'POST',
      body: JSON.stringify(user)
    })
  }
}
```

#### ✅ TO'G'RI - Bitta mas'uliyat

```typescript
// services/api/user.api.ts - Faqat API calls
export class UserApiService {
  async getUser(id: number): Promise<User> {
    const response = await apiClient.get(`/users/${id}`)
    return response.data
  }

  async updateUser(id: number, data: Partial<User>): Promise<User> {
    const response = await apiClient.put(`/users/${id}`, data)
    return response.data
  }
}

// utils/validators.ts - Faqat validation
export class EmailValidator {
  static validate(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }
}

// utils/formatters.ts - Faqat formatting
export class UserFormatter {
  static fullName(user: User): string {
    return `${user.firstName} ${user.lastName}`
  }

  static displayName(user: User): string {
    return user.displayName || this.fullName(user)
  }
}

// services/email.service.ts - Faqat email
export class EmailService {
  async send(to: string, subject: string, body: string): Promise<void> {
    await apiClient.post('/email', { to, subject, body })
  }
}
```

**Vue Component SRP:**

```vue
<!-- ❌ NOTO'G'RI - Ko'p mas'uliyat -->
<script setup lang="ts">
// Data fetching, validation, formatting, UI logic - hammasi bir joyda ❌
const users = ref([])
const loading = ref(false)

const fetchUsers = async () => {
  loading.value = true
  const response = await fetch('/api/users')
  users.value = await response.json()
  loading.value = false
}

const validateUser = (user: User) => { /* ... */ }
const formatUser = (user: User) => { /* ... */ }
const exportUsers = () => { /* ... */ }
</script>
```

```vue
<!-- ✅ TO'G'RI - Composables bilan ajratish -->
<script setup lang="ts">
import { useUsers } from '@/composables/useUsers'
import { useUserValidation } from '@/composables/useUserValidation'
import { useUserFormatter } from '@/composables/useUserFormatter'
import { useUserExport } from '@/composables/useUserExport'

// Har bir composable o'z mas'uliyatiga ega
const { users, loading, fetchUsers } = useUsers()
const { validate } = useUserValidation()
const { formatFullName } = useUserFormatter()
const { exportToCSV } = useUserExport()
</script>
```

---

### O - Open/Closed Principle (OCP)

**Printsip:** Software entities open for extension, closed for modification.

#### ❌ NOTO'G'RI - Har safar o'zgartirish

```typescript
// ❌ Har safar yangi payment method qo'shganda PaymentService o'zgaradi
class PaymentService {
  processPayment(amount: number, method: string) {
    if (method === 'credit-card') {
      // Credit card logic
      return this.processCreditCard(amount)
    } else if (method === 'paypal') {
      // PayPal logic
      return this.processPayPal(amount)
    } else if (method === 'crypto') {
      // Crypto logic ← Yangi method qo'shish uchun class o'zgartirish kerak ❌
      return this.processCrypto(amount)
    }
  }
}
```

#### ✅ TO'G'RI - Extension orqali

```typescript
// Interface - contract
interface PaymentMethod {
  process(amount: number): Promise<PaymentResult>
  validate(amount: number): boolean
}

// Har bir payment method - alohida class
class CreditCardPayment implements PaymentMethod {
  async process(amount: number): Promise<PaymentResult> {
    // Credit card logic
    return { success: true, transactionId: '...' }
  }

  validate(amount: number): boolean {
    return amount > 0 && amount <= 10000
  }
}

class PayPalPayment implements PaymentMethod {
  async process(amount: number): Promise<PaymentResult> {
    // PayPal logic
    return { success: true, transactionId: '...' }
  }

  validate(amount: number): boolean {
    return amount > 0
  }
}

// Yangi method qo'shish - faqat yangi class yaratish ✅
class CryptoPayment implements PaymentMethod {
  async process(amount: number): Promise<PaymentResult> {
    // Crypto logic
    return { success: true, transactionId: '...' }
  }

  validate(amount: number): boolean {
    return amount > 0.01
  }
}

// Service - o'zgarishsiz qoladi
class PaymentService {
  private methods = new Map<string, PaymentMethod>()

  registerMethod(name: string, method: PaymentMethod) {
    this.methods.set(name, method)
  }

  async processPayment(amount: number, methodName: string): Promise<PaymentResult> {
    const method = this.methods.get(methodName)
    if (!method) throw new Error(`Unknown payment method: ${methodName}`)

    if (!method.validate(amount)) {
      throw new Error('Invalid amount')
    }

    return method.process(amount)
  }
}

// Usage
const paymentService = new PaymentService()
paymentService.registerMethod('credit-card', new CreditCardPayment())
paymentService.registerMethod('paypal', new PayPalPayment())
paymentService.registerMethod('crypto', new CryptoPayment()) // Yangi method - service o'zgarmadi! ✅
```

**Vue Component OCP:**

```vue
<!-- ❌ NOTO'G'RI - Har safad yangi button type uchun o'zgartirish -->
<script setup lang="ts">
const props = defineProps<{
  type: 'primary' | 'secondary' | 'danger' | 'success' // Yangi type qo'shish qiyin
}>()

const getClasses = () => {
  if (props.type === 'primary') return 'btn-primary'
  if (props.type === 'secondary') return 'btn-secondary'
  if (props.type === 'danger') return 'btn-danger'
  if (props.type === 'success') return 'btn-success'
}
</script>
```

```vue
<!-- ✅ TO'G'RI - Config-driven approach -->
<script setup lang="ts">
// types/button.ts
export const BUTTON_VARIANTS = {
  primary: {
    class: 'bg-blue-500 text-white hover:bg-blue-600',
    icon: 'check'
  },
  secondary: {
    class: 'bg-gray-500 text-white hover:bg-gray-600',
    icon: 'info'
  },
  danger: {
    class: 'bg-red-500 text-white hover:bg-red-600',
    icon: 'warning'
  },
  // Yangi variant qo'shish - faqat config ga qo'shish ✅
  success: {
    class: 'bg-green-500 text-white hover:bg-green-600',
    icon: 'check-circle'
  }
} as const

type ButtonVariant = keyof typeof BUTTON_VARIANTS

// component
const props = defineProps<{
  variant: ButtonVariant
}>()

const config = computed(() => BUTTON_VARIANTS[props.variant])
</script>

<template>
  <button :class="config.class">
    <q-icon :name="config.icon" />
    <slot />
  </button>
</template>
```

---

### L - Liskov Substitution Principle (LSP)

**Printsip:** Subtypes base type ni to'liq replace qila olishi kerak.

#### ❌ NOTO'G'RI - Subtype behavior buzadi

```typescript
class Bird {
  fly() {
    console.log('Flying...')
  }
}

// Penguin uchib olmaydi, lekin Bird dan inherit qiladi ❌
class Penguin extends Bird {
  fly() {
    throw new Error('Penguins cannot fly!') // LSP buzildi!
  }
}

function makeBirdFly(bird: Bird) {
  bird.fly() // Penguin bo'lsa - error! ❌
}

const sparrow = new Bird()
const penguin = new Penguin()

makeBirdFly(sparrow) // ✅ Works
makeBirdFly(penguin) // ❌ Error!
```

#### ✅ TO'G'RI - To'g'ri abstraction

```typescript
// Base interface - minimal contract
interface Animal {
  move(): void
  eat(): void
}

// Flying animals
interface FlyingAnimal extends Animal {
  fly(): void
}

class Bird implements FlyingAnimal {
  move() {
    this.fly()
  }

  fly() {
    console.log('Flying...')
  }

  eat() {
    console.log('Eating seeds...')
  }
}

// Penguin - faqat Animal interface
class Penguin implements Animal {
  move() {
    this.swim()
  }

  swim() {
    console.log('Swimming...')
  }

  eat() {
    console.log('Eating fish...')
  }
}

// Functions faqat kerakli interface bilan ishlaydi
function makeAnimalMove(animal: Animal) {
  animal.move() // ✅ Barcha animals uchun ishlaydi
}

function makeFlyingAnimalFly(animal: FlyingAnimal) {
  animal.fly() // ✅ Faqat flying animals
}

const sparrow = new Bird()
const penguin = new Penguin()

makeAnimalMove(sparrow) // ✅
makeAnimalMove(penguin) // ✅
makeFlyingAnimalFly(sparrow) // ✅
// makeFlyingAnimalFly(penguin) // ❌ Compile error - type safe!
```

**Vue Component LSP:**

```vue
<!-- ❌ NOTO'G'RI - BaseInput dan inherit, lekin behavior boshqa -->
<!-- BaseInput.vue -->
<script setup lang="ts">
const props = defineProps<{
  modelValue: string
}>()

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()
</script>

<!-- ReadOnlyInput.vue - LSP buzadi ❌ -->
<script setup lang="ts">
// BaseInput kabi ko'rinadi, lekin emit qilmaydi
const props = defineProps<{
  modelValue: string // modelValue bor, lekin o'zgartirib bo'lmaydi!
}>()

// emit yo'q - bu LSP buzish!
</script>
```

```vue
<!-- ✅ TO'G'RI - Alohida props interface -->
<!-- EditableInput.vue -->
<script setup lang="ts">
interface EditableInputProps {
  modelValue: string
  editable: true
}

const props = defineProps<EditableInputProps>()
const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void
}>()
</script>

<!-- ReadOnlyInput.vue -->
<script setup lang="ts">
interface ReadOnlyInputProps {
  value: string // modelValue emas!
  editable: false
}

const props = defineProps<ReadOnlyInputProps>()
// emit yo'q - bu aniq!
</script>

<!-- Parent component -->
<script setup lang="ts">
const isEditing = ref(false)

// Type-safe component selection
const InputComponent = computed(() =>
  isEditing.value ? EditableInput : ReadOnlyInput
)
</script>

<template>
  <!-- Har bir component o'z kontraktiga ega -->
  <EditableInput v-if="isEditing" v-model="text" :editable="true" />
  <ReadOnlyInput v-else :value="text" :editable="false" />
</template>
```

---

### I - Interface Segregation Principle (ISP)

**Printsip:** Clientlar foydalanmaydigan interfacelarni implement qilishga majburlanmasligi kerak.

#### ❌ NOTO'G'RI - Fat interface

```typescript
// ❌ Barcha funksiyalar bitta interface da
interface Worker {
  work(): void
  eat(): void
  sleep(): void
  attendMeeting(): void
  writeCode(): void
}

// Robot worker - eat, sleep kerak emas! ❌
class RobotWorker implements Worker {
  work() {
    console.log('Working...')
  }

  eat() {
    throw new Error('Robots do not eat!') // Keraksiz method! ❌
  }

  sleep() {
    throw new Error('Robots do not sleep!') // Keraksiz method! ❌
  }

  attendMeeting() {
    console.log('Attending meeting...')
  }

  writeCode() {
    console.log('Writing code...')
  }
}
```

#### ✅ TO'G'RI - Interface segregation

```typescript
// Kichik, focused interfaces ✅
interface Workable {
  work(): void
}

interface Eatable {
  eat(): void
}

interface Sleepable {
  sleep(): void
}

interface Attendable {
  attendMeeting(): void
}

interface Codeable {
  writeCode(): void
}

// Human - barcha interfaces
class HumanWorker implements Workable, Eatable, Sleepable, Attendable, Codeable {
  work() {
    console.log('Working...')
  }

  eat() {
    console.log('Eating...')
  }

  sleep() {
    console.log('Sleeping...')
  }

  attendMeeting() {
    console.log('Attending meeting...')
  }

  writeCode() {
    console.log('Writing code...')
  }
}

// Robot - faqat kerakli interfaces ✅
class RobotWorker implements Workable, Attendable, Codeable {
  work() {
    console.log('Working...')
  }

  attendMeeting() {
    console.log('Attending meeting...')
  }

  writeCode() {
    console.log('Writing code...')
  }

  // eat va sleep yo'q - keraksiz methods implement qilish shart emas! ✅
}
```

**Vue Composable ISP:**

```typescript
// ❌ NOTO'G'RI - Fat composable
function useUserManagement() {
  // CRUD operations
  const getUser = () => {}
  const createUser = () => {}
  const updateUser = () => {}
  const deleteUser = () => {}

  // Authentication
  const login = () => {}
  const logout = () => {}

  // Permissions
  const checkPermission = () => {}

  // Profile
  const updateProfile = () => {}
  const uploadAvatar = () => {}

  // Hammasi return ❌
  return {
    getUser, createUser, updateUser, deleteUser,
    login, logout,
    checkPermission,
    updateProfile, uploadAvatar
  }
}

// Component faqat getUser kerak, lekin barcha methods keladi ❌
```

```typescript
// ✅ TO'G'RI - Segregated composables
function useUserApi() {
  const getUser = () => {}
  const createUser = () => {}
  const updateUser = () => {}
  const deleteUser = () => {}

  return { getUser, createUser, updateUser, deleteUser }
}

function useAuth() {
  const login = () => {}
  const logout = () => {}
  const isAuthenticated = () => {}

  return { login, logout, isAuthenticated }
}

function usePermissions() {
  const checkPermission = () => {}
  const hasRole = () => {}

  return { checkPermission, hasRole }
}

function useUserProfile() {
  const updateProfile = () => {}
  const uploadAvatar = () => {}

  return { updateProfile, uploadAvatar }
}

// Component faqat kerakli composables ni import qiladi ✅
const { getUser } = useUserApi()
const { isAuthenticated } = useAuth()
```

---

### D - Dependency Inversion Principle (DIP)

**Printsip:** High-level modules low-level modulelarga bog'liq bo'lmasligi kerak. Ikkalasi ham abstraction ga bog'liq bo'lishi kerak.

#### ❌ NOTO'G'RI - Concrete dependency

```typescript
// ❌ UserService to'g'ridan-to'g'ri LocalStorage ga bog'liq
class UserService {
  saveUser(user: User) {
    // Concrete implementation ga bog'liq ❌
    localStorage.setItem('user', JSON.stringify(user))
  }

  getUser(): User | null {
    const data = localStorage.getItem('user')
    return data ? JSON.parse(data) : null
  }
}

// LocalStorage dan SessionStorage ga o'tish - UserService o'zgartirish kerak ❌
```

#### ✅ TO'G'RI - Depend on abstraction

```typescript
// Abstraction - interface
interface StorageService {
  setItem(key: string, value: string): void
  getItem(key: string): string | null
  removeItem(key: string): void
}

// Concrete implementations
class LocalStorageService implements StorageService {
  setItem(key: string, value: string): void {
    localStorage.setItem(key, value)
  }

  getItem(key: string): string | null {
    return localStorage.getItem(key)
  }

  removeItem(key: string): void {
    localStorage.removeItem(key)
  }
}

class SessionStorageService implements StorageService {
  setItem(key: string, value: string): void {
    sessionStorage.setItem(key, value)
  }

  getItem(key: string): string | null {
    return sessionStorage.getItem(key)
  }

  removeItem(key: string): void {
    sessionStorage.removeItem(key)
  }
}

class InMemoryStorageService implements StorageService {
  private storage = new Map<string, string>()

  setItem(key: string, value: string): void {
    this.storage.set(key, value)
  }

  getItem(key: string): string | null {
    return this.storage.get(key) || null
  }

  removeItem(key: string): void {
    this.storage.delete(key)
  }
}

// UserService abstraction ga bog'liq ✅
class UserService {
  constructor(private storage: StorageService) {}

  saveUser(user: User) {
    this.storage.setItem('user', JSON.stringify(user))
  }

  getUser(): User | null {
    const data = this.storage.getItem('user')
    return data ? JSON.parse(data) : null
  }
}

// Dependency injection
const localStorageService = new LocalStorageService()
const userService = new UserService(localStorageService)

// Storage implementation o'zgartirish - UserService o'zgarmaydi ✅
const sessionStorageService = new SessionStorageService()
const userService2 = new UserService(sessionStorageService)
```

**Vue Dependency Injection:**

```typescript
// types/injection-keys.ts
export const StorageServiceKey: InjectionKey<StorageService> = Symbol('StorageService')

// App.vue - provide
import { provide } from 'vue'
import { LocalStorageService } from '@/services/storage'

const storageService = new LocalStorageService()
provide(StorageServiceKey, storageService)

// Component - inject
import { inject } from 'vue'
import { StorageServiceKey } from '@/types/injection-keys'

const storage = inject(StorageServiceKey)
if (!storage) throw new Error('StorageService not provided')

// Component abstraction ga bog'liq, concrete implementation ga emas ✅
```

---

## Boshqa Architecture Principles

### DRY - Don't Repeat Yourself

**Printsip:** Har qanday knowledge yoki logic faqat bir joyda bo'lishi kerak.

#### ❌ NOTO'G'RI - Code duplication

```vue
<!-- UserList.vue -->
<script setup lang="ts">
const users = ref([])
const loading = ref(false)
const error = ref(null)

const fetchUsers = async () => {
  loading.value = true
  error.value = null
  try {
    const response = await fetch('/api/users')
    if (!response.ok) throw new Error('Failed to fetch')
    users.value = await response.json()
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
}
</script>

<!-- ProductList.vue - XUDDI SHU CODE! ❌ -->
<script setup lang="ts">
const products = ref([])
const loading = ref(false)
const error = ref(null)

const fetchProducts = async () => {
  loading.value = true
  error.value = null
  try {
    const response = await fetch('/api/products')
    if (!response.ok) throw new Error('Failed to fetch')
    products.value = await response.json()
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
}
</script>
```

#### ✅ TO'G'RI - Reusable composable

```typescript
// composables/useAsyncData.ts
export function useAsyncData<T>(fetcher: () => Promise<T>) {
  const data = ref<T | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  const execute = async () => {
    loading.value = true
    error.value = null

    try {
      data.value = await fetcher()
    } catch (e) {
      error.value = e instanceof Error ? e.message : 'Unknown error'
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

// Usage - DRY ✅
const { data: users, loading, error, execute: fetchUsers } = useAsyncData(
  () => fetch('/api/users').then(r => r.json())
)

const { data: products } = useAsyncData(
  () => fetch('/api/products').then(r => r.json())
)
```

---

### KISS - Keep It Simple, Stupid

**Printsip:** Kod simple va tushunarli bo'lishi kerak.

#### ❌ NOTO'G'RI - Over-engineered

```typescript
// ❌ Juda murakkab abstraction
class AbstractFactoryProducerSingletonBuilder<T extends BaseEntity> {
  private static instances = new Map()

  static getInstance<U extends BaseEntity>(
    type: new () => U
  ): AbstractFactoryProducerSingletonBuilder<U> {
    if (!this.instances.has(type)) {
      this.instances.set(type, new this())
    }
    return this.instances.get(type)
  }

  createFactory(): AbstractFactory<T> {
    return new ConcreteFactory<T>()
  }
}

// Faqat "Hello World" chiqarish uchun! ❌
```

#### ✅ TO'G'RI - Simple solution

```typescript
// ✅ Oddiy, tushunarli
function greet(name: string): string {
  return `Hello, ${name}!`
}

console.log(greet('World'))
```

**Real example:**

```vue
<!-- ❌ NOTO'G'RI - Over-complicated -->
<script setup lang="ts">
import { useCompositionProvider } from '@/composables/composition-provider'
import { useStrategyPattern } from '@/composables/strategy-pattern'

const compositionProvider = useCompositionProvider()
const strategy = useStrategyPattern('default')

const result = computed(() => {
  return strategy.execute(
    compositionProvider.getContext().transform(
      data.value
    )
  )
})
</script>

<!-- ✅ TO'G'RI - Simple -->
<script setup lang="ts">
const fullName = computed(() => `${firstName.value} ${lastName.value}`)
</script>
```

---

### YAGNI - You Aren't Gonna Need It

**Printsip:** Hozir kerak bo'lmagan feature ni implement qilmaslik.

#### ❌ NOTO'G'RI - Premature features

```typescript
// ❌ Feature hali kerak emas, lekin implement qilingan
interface User {
  id: number
  name: string
  email: string

  // Hozir ishlatilmaydi ❌
  phoneNumber?: string
  address?: Address
  socialMedia?: SocialMediaLinks
  preferences?: UserPreferences
  subscriptions?: Subscription[]
  paymentMethods?: PaymentMethod[]
}

// Murakkab logic - hali kerak emas ❌
class UserService {
  async getUserWithFullProfile(id: number) {
    const user = await this.getUser(id)
    const preferences = await this.getPreferences(id)
    const subscriptions = await this.getSubscriptions(id)
    const paymentMethods = await this.getPaymentMethods(id)

    return {
      ...user,
      preferences,
      subscriptions,
      paymentMethods
    }
  }

  // Bu method hech qachon chaqirilmagan! ❌
}
```

#### ✅ TO'G'RI - Only what you need now

```typescript
// ✅ Faqat hozir kerakli
interface User {
  id: number
  name: string
  email: string
}

class UserService {
  async getUser(id: number): Promise<User> {
    return apiClient.get(`/users/${id}`)
  }
}

// Keyin kerak bo'lsa - qo'shamiz
```

---

## Folder Structure Best Practices

### Feature-based vs Type-based Structure

#### Type-based (Traditional) - Kichik loyihalar uchun

```
src/
├── components/
│   ├── UserCard.vue
│   ├── UserList.vue
│   ├── ProductCard.vue
│   └── ProductList.vue
├── composables/
│   ├── useUser.ts
│   └── useProduct.ts
└── stores/
    ├── user.ts
    └── product.ts
```

#### Feature-based (Recommended) - Katta loyihalar uchun

```
src/
├── features/
│   ├── user/
│   │   ├── components/
│   │   │   ├── UserCard.vue
│   │   │   └── UserList.vue
│   │   ├── composables/
│   │   │   └── useUser.ts
│   │   ├── stores/
│   │   │   └── user.store.ts
│   │   ├── types/
│   │   │   └── user.types.ts
│   │   └── api/
│   │       └── user.api.ts
│   └── product/
│       ├── components/
│       ├── composables/
│       └── stores/
└── shared/
    ├── components/
    ├── composables/
    └── utils/
```

**Sabab:**
- Feature-based: Har bir feature mustaqil module
- Team collaboration: Har bir team o'z feature folder ida ishlaydi
- Scalability: Yangi feature qo'shish oson
- Maintainability: Feature o'chirsangiz - bir papkani o'chirasiz

---

## Xulosa

### SOLID Principles Summary

1. **SRP**: Har bir modul faqat bitta sabab uchun o'zgarishi kerak
2. **OCP**: Extension uchun ochiq, modification uchun yopiq
3. **LSP**: Subtype base type ni buzmasdan replace qilishi kerak
4. **ISP**: Kichik, focused interfaces
5. **DIP**: Abstraction ga bog'liq, concrete implementation ga emas

### Other Principles Summary

- **DRY**: Duplication yo'q, reusable code
- **KISS**: Oddiy, tushunarli solution
- **YAGNI**: Faqat kerakli feature

### Best Practices

- Feature-based folder structure katta loyihalar uchun
- Type-based structure kichik loyihalar uchun
- SOLID principles har doim follow qilish
- Over-engineering dan qoching
- Premature optimization qilmaslik
