# Code Quality

## Mundarija
- [ESLint va TypeScript](#eslint-va-typescript)
- [Code Style](#code-style)
- [Testing](#testing)
- [Error Handling](#error-handling)
- [Code Review Guidelines](#code-review-guidelines)
- [Git Workflow](#git-workflow)

---

## ESLint va TypeScript

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

// Type safety ✅
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

### ✅ ESLint Configuration

```javascript
// .eslintrc.js
module.exports = {
  root: true,
  env: {
    node: true,
    browser: true
  },
  extends: [
    'plugin:vue/vue3-recommended',
    'eslint:recommended',
    '@vue/typescript/recommended'
  ],
  parserOptions: {
    ecmaVersion: 2021
  },
  rules: {
    // Vue rules
    'vue/multi-word-component-names': 'error',
    'vue/no-unused-vars': 'error',
    'vue/require-default-prop': 'error',
    'vue/require-prop-types': 'error',
    'vue/component-tags-order': ['error', {
      order: ['script', 'template', 'style']
    }],

    // TypeScript rules
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-non-null-assertion': 'warn',

    // General rules
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'prefer-const': 'error',
    'no-var': 'error'
  }
}
```

### ✅ TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "preserve",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "types": ["vite/client"],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.vue"],
  "exclude": ["node_modules"]
}
```

**Sabab:**
- **@ts-ignore:** Type xatosini berkitadi, lekin muammo qoladi
- **Type Safety:** Runtime da xato bo'lishi mumkin
- **Code Quality:** Xatolar development da topilishi kerak
- **Maintainability:** Ignore comment refactoring qilishni qiyinlashtiradi
- **Team Standards:** Umumiy code style

---

## Code Style

### ✅ Naming Conventions

```typescript
// Variables & Functions - camelCase ✅
const userName = 'John'
const isActive = true
function fetchUserData() {}

// Constants - UPPER_SNAKE_CASE ✅
const API_BASE_URL = 'https://api.example.com'
const MAX_RETRY_COUNT = 3

// Types & Interfaces - PascalCase ✅
interface User {}
type UserRole = 'admin' | 'user'

// Components - PascalCase ✅
// UserCard.vue, ProductList.vue

// Composables - camelCase with 'use' prefix ✅
function useAuth() {}
function useApi() {}

// Files
// Components: PascalCase - UserCard.vue
// Composables: camelCase - useAuth.ts
// Utils: camelCase - formatDate.ts
// Types: camelCase - user.ts
```

### ✅ Code Organization

```vue
<script setup lang="ts">
// 1. Imports
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import type { User } from '@/types'

// 2. Props & Emits
interface Props {
  userId: number
}

const props = defineProps<Props>()
const emit = defineEmits<{
  (e: 'update', user: User): void
}>()

// 3. Composables
const router = useRouter()
const userStore = useUserStore()

// 4. State
const user = ref<User | null>(null)
const loading = ref(false)

// 5. Computed
const isAdmin = computed(() => user.value?.role === 'admin')

// 6. Methods
const fetchUser = async () => {
  loading.value = true
  try {
    const response = await fetch(`/api/users/${props.userId}`)
    user.value = await response.json()
  } finally {
    loading.value = false
  }
}

// 7. Lifecycle
onMounted(() => {
  fetchUser()
})

// 8. Watchers
watch(() => props.userId, () => {
  fetchUser()
})
</script>

<template>
  <!-- Template -->
</template>

<style scoped>
/* Styles */
</style>
```

**Sabab:**
- **Consistency:** Code o'qish va tushunish oson
- **Predictability:** Developer qayerda nima borligini biladi
- **Maintainability:** O'zgartirishlar oson
- **Team Collaboration:** Umumiy standard

---

## Testing

### ✅ Unit Tests

```typescript
// composables/useCounter.spec.ts
import { describe, it, expect } from 'vitest'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('should initialize with 0', () => {
    const { count } = useCounter()
    expect(count.value).toBe(0)
  })

  it('should increment count', () => {
    const { count, increment } = useCounter()
    increment()
    expect(count.value).toBe(1)
  })

  it('should decrement count', () => {
    const { count, decrement } = useCounter()
    decrement()
    expect(count.value).toBe(-1)
  })

  it('should reset count', () => {
    const { count, increment, reset } = useCounter()
    increment()
    increment()
    reset()
    expect(count.value).toBe(0)
  })
})
```

### ✅ Component Tests

```typescript
// components/UserCard.spec.ts
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import UserCard from './UserCard.vue'

describe('UserCard', () => {
  const mockUser = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com'
  }

  it('renders user information', () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser }
    })

    expect(wrapper.text()).toContain('John Doe')
    expect(wrapper.text()).toContain('john@example.com')
  })

  it('emits delete event when button clicked', async () => {
    const wrapper = mount(UserCard, {
      props: { user: mockUser }
    })

    await wrapper.find('[data-test="delete-btn"]').trigger('click')

    expect(wrapper.emitted('delete')).toBeTruthy()
    expect(wrapper.emitted('delete')?.[0]).toEqual([mockUser.id])
  })

  it('shows loading state', () => {
    const wrapper = mount(UserCard, {
      props: {
        user: mockUser,
        loading: true
      }
    })

    expect(wrapper.find('[data-test="loading"]').exists()).toBe(true)
  })
})
```

### ✅ E2E Tests

```typescript
// e2e/login.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Login', () => {
  test('should login successfully', async ({ page }) => {
    await page.goto('/login')

    await page.fill('[data-test="email-input"]', 'user@example.com')
    await page.fill('[data-test="password-input"]', 'password123')
    await page.click('[data-test="login-btn"]')

    await expect(page).toHaveURL('/dashboard')
    await expect(page.locator('[data-test="user-menu"]')).toBeVisible()
  })

  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login')

    await page.fill('[data-test="email-input"]', 'wrong@example.com')
    await page.fill('[data-test="password-input"]', 'wrongpassword')
    await page.click('[data-test="login-btn"]')

    await expect(page.locator('[data-test="error-message"]')).toBeVisible()
    await expect(page.locator('[data-test="error-message"]')).toContainText(
      'Invalid credentials'
    )
  })
})
```

**Test data attributes:**

```vue
<template>
  <!-- Use data-test for test selectors ✅ -->
  <button data-test="submit-btn">Submit</button>
  <input data-test="email-input" v-model="email" />
  <div data-test="error-message" v-if="error">{{ error }}</div>
</template>
```

**Sabab:**
- **Reliability:** Code to'g'ri ishlashini ta'minlash
- **Refactoring Confidence:** Test bor bo'lsa, refactoring xavfli emas
- **Documentation:** Test code ni qanday ishlatishni ko'rsatadi
- **Regression Prevention:** Eski xatolar qaytib kelmaydi

---

## Error Handling

### ❌ NOTO'G'RI - Error handling yo'q

```vue
<script setup lang="ts">
const fetchData = async () => {
  // Error handling yo'q ❌
  const response = await fetch('/api/data')
  const data = await response.json()
  return data
}
</script>
```

### ✅ TO'G'RI - To'liq error handling

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()
const loading = ref(false)
const error = ref<string | null>(null)

const fetchData = async () => {
  loading.value = true
  error.value = null

  try {
    const response = await fetch('/api/data')

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()
    return data
  } catch (e) {
    const errorMessage = e instanceof Error ? e.message : 'Unknown error'
    error.value = errorMessage

    // Show user-friendly error
    $q.notify({
      type: 'negative',
      message: 'Failed to fetch data. Please try again.',
      timeout: 3000
    })

    // Log to error tracking service
    console.error('Fetch error:', e)
    // logToSentry(e)

    throw e // Re-throw if needed
  } finally {
    loading.value = false
  }
}
</script>
```

### ✅ Global Error Handler

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// Global error handler
app.config.errorHandler = (err, instance, info) => {
  console.error('Global error:', err)
  console.error('Component:', instance)
  console.error('Error info:', info)

  // Log to error tracking service
  // logToSentry(err, { component: instance, info })
}

// Global warning handler
app.config.warnHandler = (msg, instance, trace) => {
  console.warn('Warning:', msg)
  console.warn('Component:', instance)
  console.warn('Trace:', trace)
}

app.mount('#app')
```

### ✅ Error Boundary Component

```vue
<!-- ErrorBoundary.vue -->
<script setup lang="ts">
import { ref, onErrorCaptured } from 'vue'

const error = ref<Error | null>(null)

onErrorCaptured((err) => {
  error.value = err
  console.error('Error captured:', err)
  return false // Prevent propagation
})

const retry = () => {
  error.value = null
}
</script>

<template>
  <div v-if="error" class="error-boundary">
    <h2>Something went wrong</h2>
    <p>{{ error.message }}</p>
    <button @click="retry">Try Again</button>
  </div>
  <slot v-else />
</template>
```

**Usage:**

```vue
<template>
  <ErrorBoundary>
    <SomeComponentThatMightFail />
  </ErrorBoundary>
</template>
```

**Sabab:**
- **User Experience:** User-friendly error messages
- **Debugging:** Error logs help debugging
- **Monitoring:** Track errors in production
- **Recovery:** Graceful error recovery

---

## Code Review Guidelines

### ✅ Code Review Checklist

**Functionality:**
- [ ] Code ishlaydimi?
- [ ] Edge cases handle qilinganmi?
- [ ] Error handling to'g'rimi?

**Code Quality:**
- [ ] Code o'qilishi osonmi?
- [ ] Naming conventions to'g'rimi?
- [ ] DRY principle bajarilganmi?
- [ ] SOLID principles bajarilganmi?

**Performance:**
- [ ] Performance issues bormi?
- [ ] Unnecessary re-renders yo'qmi?
- [ ] Large arrays optimized mi?

**Type Safety:**
- [ ] TypeScript types to'g'rimi?
- [ ] `any` type ishlatilmaganmi?
- [ ] Props va emits typed mi?

**Testing:**
- [ ] Tests yozilganmi?
- [ ] Edge cases test qilinganmi?
- [ ] Tests o'tayaptimi?

**Security:**
- [ ] XSS vulnerabilities yo'qmi?
- [ ] User input sanitized mi?
- [ ] Sensitive data exposed emas mi?

**Documentation:**
- [ ] Complex logic comment qilinganmi?
- [ ] README updated mi?
- [ ] API documentation to'g'rimi?

### ✅ Code Review Comments

```typescript
// ❌ Bad review comment
"This is wrong"

// ✅ Good review comment
"This function might throw if user is null. Consider adding a null check:
if (!user) return

Or use optional chaining:
const name = user?.name ?? 'Guest'
"

// ❌ Bad review comment
"Change this"

// ✅ Good review comment
"Consider extracting this logic into a composable for reusability:

// composables/useUserData.ts
export function useUserData(userId: Ref<number>) {
  // logic here
}

This would make it reusable in other components."
```

**Sabab:**
- **Constructive Feedback:** Specific, actionable suggestions
- **Knowledge Sharing:** Teach best practices
- **Code Quality:** Maintain high standards
- **Team Growth:** Everyone learns

---

## Git Workflow

### ✅ Commit Messages

```bash
# ✅ Good commit messages
feat: add user authentication
fix: resolve login redirect issue
refactor: extract user logic to composable
docs: update API documentation
test: add user service tests
perf: optimize product list rendering
style: fix linting issues
chore: update dependencies

# ❌ Bad commit messages
"fix"
"update"
"changes"
"wip"
"asdf"
```

**Commit message format:**

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Example:**

```
feat(auth): add social login

- Add Google OAuth integration
- Add Facebook login button
- Update user model with provider field

Closes #123
```

### ✅ Branch Naming

```bash
# ✅ Good branch names
feature/user-authentication
fix/login-redirect-bug
refactor/user-composable
docs/api-documentation
test/user-service
hotfix/critical-security-issue

# ❌ Bad branch names
"new-branch"
"fix"
"test"
"john-work"
```

### ✅ Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] E2E tests pass
- [ ] Manual testing done

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review done
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings

## Screenshots (if applicable)
```

**Sabab:**
- **History:** Clear project history
- **Collaboration:** Easy to track changes
- **Rollback:** Easy to revert if needed
- **Release Notes:** Automatic changelog generation

---

## Xulosa

Code quality best practices:

1. **ESLint & TypeScript:** Strict configuration, fix errors
2. **Code Style:** Consistent naming, organization
3. **Testing:** Unit, component, E2E tests
4. **Error Handling:** Try-catch, global handlers, error boundaries
5. **Code Review:** Constructive feedback, checklist
6. **Git Workflow:** Clear commits, branch naming, PR template

**Code quality metrics:**
- Type coverage > 90%
- Test coverage > 80%
- ESLint errors = 0
- Build warnings = 0
- Performance score > 90

**Tools:**
- ESLint + TypeScript
- Vitest (unit tests)
- Playwright (E2E)
- Prettier (formatting)
- Husky (git hooks)
- Commitlint (commit messages)
