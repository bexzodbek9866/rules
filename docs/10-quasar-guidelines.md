# Quasar Specific Guidelines

## Mundarija
- [Quasar Setup](#quasar-setup)
- [Quasar Components](#quasar-components)
- [Quasar Plugins](#quasar-plugins)
- [Quasar Layout System](#quasar-layout-system)
- [Quasar Utilities](#quasar-utilities)
- [Quasar Directives](#quasar-directives)
- [Theme Customization](#theme-customization)

---

## Quasar Setup

### ✅ Quasar Configuration

```typescript
// quasar.config.js
import { configure } from 'quasar/wrappers'

export default configure((ctx) => {
  return {
    framework: {
      config: {
        brand: {
          primary: '#1976D2',
          secondary: '#26A69A',
          accent: '#9C27B0',
          dark: '#1d1d1d',
          positive: '#21BA45',
          negative: '#C10015',
          info: '#31CCEC',
          warning: '#F2C037'
        },
        notify: {
          position: 'top',
          timeout: 2500,
          textColor: 'white',
          actions: [{ icon: 'close', color: 'white' }]
        },
        loading: {
          delay: 400,
          message: 'Loading...',
          spinnerColor: 'primary'
        }
      },
      plugins: [
        'Notify',
        'Dialog',
        'Loading',
        'LocalStorage',
        'SessionStorage'
      ],
      cssAddon: true
    },
    build: {
      vueRouterMode: 'history',
      typescript: {
        strict: true,
        vueShim: false
      }
    }
  }
})
```

### ✅ Auto-import Configuration

```typescript
// quasar.config.js
export default configure(() => {
  return {
    framework: {
      // Auto import Quasar components
      components: 'all', // yoki specific components ro'yxati

      // Auto import Quasar directives
      directives: 'all',

      // Auto import Quasar plugins
      plugins: [
        'Notify',
        'Dialog',
        'Loading'
      ]
    }
  }
})
```

**Sabab:**
- **Type Safety:** TypeScript support built-in
- **Tree Shaking:** Faqat ishlatilgan componentlar bundle ga kiradi
- **Theme:** Global theme configuration
- **Plugins:** Quasar plugins configuration

---

## Quasar Components

### ✅ QTable with TypeScript

```vue
<script setup lang="ts">
import { ref } from 'vue'
import type { QTableColumn } from 'quasar'

interface User {
  id: number
  name: string
  email: string
  role: 'admin' | 'user'
  status: 'active' | 'inactive'
}

const columns: QTableColumn<User>[] = [
  {
    name: 'id',
    label: 'ID',
    field: 'id',
    align: 'left',
    sortable: true
  },
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
  },
  {
    name: 'role',
    label: 'Role',
    field: 'role',
    align: 'center',
    sortable: true
  },
  {
    name: 'status',
    label: 'Status',
    field: 'status',
    align: 'center',
    format: (val: string) => val.toUpperCase()
  },
  {
    name: 'actions',
    label: 'Actions',
    field: 'id',
    align: 'center'
  }
]

const rows = ref<User[]>([
  {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    role: 'admin',
    status: 'active'
  }
])

const pagination = ref({
  sortBy: 'name',
  descending: false,
  page: 1,
  rowsPerPage: 10
})

const onRequest = (props: any) => {
  const { page, rowsPerPage, sortBy, descending } = props.pagination

  // Fetch data from API
  // ...

  pagination.value = props.pagination
}
</script>

<template>
  <q-table
    :rows="rows"
    :columns="columns"
    row-key="id"
    :pagination="pagination"
    @request="onRequest"
    :loading="loading"
    binary-state-sort
  >
    <!-- Custom column slots -->
    <template #body-cell-status="props">
      <q-td :props="props">
        <q-badge
          :color="props.row.status === 'active' ? 'positive' : 'negative'"
          :label="props.row.status"
        />
      </q-td>
    </template>

    <template #body-cell-actions="props">
      <q-td :props="props">
        <q-btn
          flat
          round
          dense
          icon="edit"
          @click="editUser(props.row)"
        />
        <q-btn
          flat
          round
          dense
          icon="delete"
          @click="deleteUser(props.row.id)"
        />
      </q-td>
    </template>
  </q-table>
</template>
```

### ✅ QForm with Validation

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

interface LoginForm {
  email: string
  password: string
  remember: boolean
}

const form = ref<LoginForm>({
  email: '',
  password: '',
  remember: false
})

const formRef = ref()

// Validation rules
const emailRules = [
  (val: string) => !!val || 'Email is required',
  (val: string) => /.+@.+\..+/.test(val) || 'Email must be valid'
]

const passwordRules = [
  (val: string) => !!val || 'Password is required',
  (val: string) => val.length >= 6 || 'Password must be at least 6 characters'
]

const onSubmit = () => {
  formRef.value.validate().then((success: boolean) => {
    if (success) {
      // Form is valid, submit
      submitLogin(form.value)
    } else {
      $q.notify({
        type: 'negative',
        message: 'Please fix the errors'
      })
    }
  })
}

const onReset = () => {
  form.value = {
    email: '',
    password: '',
    remember: false
  }
  formRef.value.resetValidation()
}
</script>

<template>
  <q-form
    ref="formRef"
    @submit="onSubmit"
    @reset="onReset"
    class="q-gutter-md"
  >
    <q-input
      v-model="form.email"
      label="Email"
      type="email"
      :rules="emailRules"
      lazy-rules
      outlined
    >
      <template #prepend>
        <q-icon name="email" />
      </template>
    </q-input>

    <q-input
      v-model="form.password"
      label="Password"
      type="password"
      :rules="passwordRules"
      lazy-rules
      outlined
    >
      <template #prepend>
        <q-icon name="lock" />
      </template>
    </q-input>

    <q-checkbox
      v-model="form.remember"
      label="Remember me"
    />

    <div class="row q-gutter-sm">
      <q-btn
        label="Login"
        type="submit"
        color="primary"
      />
      <q-btn
        label="Reset"
        type="reset"
        color="secondary"
        flat
      />
    </div>
  </q-form>
</template>
```

**Sabab:**
- **Type Safety:** QTableColumn generic type
- **Built-in Validation:** QForm validation rules
- **Responsive:** Quasar components responsive by default
- **Accessibility:** ARIA attributes built-in

---

## Quasar Plugins

### ✅ Notify Plugin

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

const showSuccess = () => {
  $q.notify({
    type: 'positive',
    message: 'Operation successful!',
    position: 'top',
    timeout: 2000
  })
}

const showError = () => {
  $q.notify({
    type: 'negative',
    message: 'Something went wrong',
    caption: 'Please try again later',
    position: 'top',
    timeout: 3000,
    actions: [
      {
        label: 'Dismiss',
        color: 'white',
        handler: () => { /* ... */ }
      }
    ]
  })
}

const showCustom = () => {
  $q.notify({
    message: 'Custom notification',
    color: 'purple',
    textColor: 'white',
    icon: 'announcement',
    position: 'bottom-right',
    timeout: 2500,
    actions: [
      {
        icon: 'close',
        color: 'white'
      }
    ]
  })
}
</script>
```

### ✅ Dialog Plugin

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

const showConfirm = () => {
  $q.dialog({
    title: 'Confirm',
    message: 'Are you sure you want to delete this item?',
    cancel: true,
    persistent: true
  }).onOk(() => {
    // User confirmed
    deleteItem()
  }).onCancel(() => {
    // User cancelled
  })
}

const showPrompt = () => {
  $q.dialog({
    title: 'Prompt',
    message: 'Enter your name',
    prompt: {
      model: '',
      type: 'text'
    },
    cancel: true,
    persistent: true
  }).onOk((data) => {
    console.log('User entered:', data)
  })
}

const showCustomDialog = () => {
  $q.dialog({
    component: CustomDialogComponent,
    componentProps: {
      userId: 123
    }
  }).onOk((result) => {
    console.log('Dialog result:', result)
  })
}
</script>
```

### ✅ Loading Plugin

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

const performLongOperation = async () => {
  $q.loading.show({
    message: 'Loading...',
    boxClass: 'bg-grey-2 text-grey-9',
    spinnerColor: 'primary'
  })

  try {
    await someAsyncOperation()
  } finally {
    $q.loading.hide()
  }
}

// With custom spinner
const withCustomSpinner = async () => {
  $q.loading.show({
    spinner: QSpinnerGears,
    spinnerColor: 'primary',
    spinnerSize: 140,
    message: 'Processing...',
    messageColor: 'black'
  })

  try {
    await heavyOperation()
  } finally {
    $q.loading.hide()
  }
}
</script>
```

### ✅ LocalStorage Plugin

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

// Set item
$q.localStorage.set('user', {
  id: 1,
  name: 'John'
})

// Get item
const user = $q.localStorage.getItem('user')

// Check if exists
if ($q.localStorage.has('token')) {
  // Token exists
}

// Remove item
$q.localStorage.remove('token')

// Get all keys
const keys = $q.localStorage.getAllKeys()

// Clear all
$q.localStorage.clear()

// With type safety
interface User {
  id: number
  name: string
}

const getUser = (): User | null => {
  return $q.localStorage.getItem<User>('user')
}
</script>
```

**Sabab:**
- **useQuasar composable:** Access to all Quasar plugins
- **Consistent API:** Bir xil API pattern
- **Type Safety:** TypeScript support
- **Promise-based:** Async operations support

---

## Quasar Layout System

### ✅ Main Layout

```vue
<!-- layouts/MainLayout.vue -->
<script setup lang="ts">
import { ref } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()
const leftDrawerOpen = ref(false)
const miniState = ref(false)

const toggleLeftDrawer = () => {
  leftDrawerOpen.value = !leftDrawerOpen.value
}

const toggleMini = () => {
  miniState.value = !miniState.value
}
</script>

<template>
  <q-layout view="hHh lpR fFf">
    <!-- Header -->
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title>
          My Application
        </q-toolbar-title>

        <q-btn
          flat
          round
          dense
          icon="account_circle"
        >
          <q-menu>
            <q-list style="min-width: 100px">
              <q-item clickable v-close-popup>
                <q-item-section>Profile</q-item-section>
              </q-item>
              <q-item clickable v-close-popup>
                <q-item-section>Settings</q-item-section>
              </q-item>
              <q-separator />
              <q-item clickable v-close-popup>
                <q-item-section>Logout</q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </q-btn>
      </q-toolbar>
    </q-header>

    <!-- Left Drawer -->
    <q-drawer
      v-model="leftDrawerOpen"
      :mini="miniState"
      :breakpoint="1024"
      bordered
      show-if-above
    >
      <q-scroll-area class="fit">
        <q-list>
          <q-item
            clickable
            to="/"
            exact
          >
            <q-item-section avatar>
              <q-icon name="home" />
            </q-item-section>
            <q-item-section>Home</q-item-section>
          </q-item>

          <q-item
            clickable
            to="/users"
          >
            <q-item-section avatar>
              <q-icon name="people" />
            </q-item-section>
            <q-item-section>Users</q-item-section>
          </q-item>

          <q-item
            clickable
            to="/products"
          >
            <q-item-section avatar>
              <q-icon name="inventory" />
            </q-item-section>
            <q-item-section>Products</q-item-section>
          </q-item>

          <q-separator />

          <q-item
            clickable
            @click="toggleMini"
          >
            <q-item-section avatar>
              <q-icon :name="miniState ? 'chevron_right' : 'chevron_left'" />
            </q-item-section>
            <q-item-section>Collapse</q-item-section>
          </q-item>
        </q-list>
      </q-scroll-area>
    </q-drawer>

    <!-- Page Container -->
    <q-page-container>
      <router-view />
    </q-page-container>

    <!-- Footer -->
    <q-footer elevated class="bg-grey-8 text-white">
      <q-toolbar>
        <q-toolbar-title>
          <div class="text-center">
            © 2024 My Application
          </div>
        </q-toolbar-title>
      </q-toolbar>
    </q-footer>
  </q-layout>
</template>
```

### ✅ Layout View Property

```
Layout view prop: "hHh lpR fFf"

Position:  h H h    l p r    f F f
           -----    -----    -----
Modifier:  h = header, l = left, p = page, r = right, f = footer
Type:      Lowercase = hidden on mobile
           Uppercase = always visible

Examples:
hHh lpR fFf - Header, drawer, page, footer
lHh lpr fff - Header always visible, footer hidden on mobile
hHh Lpr fFf - Left drawer always visible (overlay on mobile)
```

**Sabab:**
- **Responsive:** Automatic responsive behavior
- **Mini mode:** Collapsible drawer
- **Breakpoint:** Show/hide drawer based on screen size
- **Flexible:** View property for different layouts

---

## Quasar Utilities

### ✅ Platform Detection

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

// Platform detection
if ($q.platform.is.mobile) {
  console.log('Mobile device')
}

if ($q.platform.is.desktop) {
  console.log('Desktop')
}

if ($q.platform.is.ios) {
  console.log('iOS device')
}

if ($q.platform.is.android) {
  console.log('Android device')
}

// Screen size
if ($q.screen.lt.sm) {
  console.log('Extra small screen')
}

if ($q.screen.gt.md) {
  console.log('Large screen')
}

// Dark mode
const isDark = $q.dark.isActive

// Set dark mode
$q.dark.set(true) // or false or 'auto'
</script>
```

### ✅ Screen Plugin

```vue
<script setup lang="ts">
import { useQuasar } from 'quasar'

const $q = useQuasar()

// Breakpoints
// xs: 0-599px
// sm: 600-1023px
// md: 1024-1439px
// lg: 1440-1919px
// xl: 1920px+

const isMobile = $q.screen.lt.md // < 1024px
const isTablet = $q.screen.gt.sm && $q.screen.lt.lg
const isDesktop = $q.screen.gt.md

// Reactive screen size
watch(() => $q.screen.width, (width) => {
  console.log('Screen width:', width)
})
</script>

<template>
  <!-- Conditional rendering based on screen size -->
  <q-btn v-if="$q.screen.gt.sm" label="Desktop Button" />
  <q-btn v-else icon="menu" round />
</template>
```

### ✅ Colors Utility

```vue
<script setup lang="ts">
import { colors } from 'quasar'

const { getPaletteColor, getBrand, lighten, luminosity } = colors

// Get brand color
const primary = getBrand('primary') // '#1976D2'

// Get palette color
const red = getPaletteColor('red-5')

// Lighten/darken
const lightPrimary = lighten(primary, 50)

// Check luminosity
const isLight = luminosity(primary) > 0.5
</script>
```

**Sabab:**
- **Platform-specific logic:** Different behavior per platform
- **Responsive design:** Breakpoint-based rendering
- **Dark mode:** Built-in dark mode support
- **Color utilities:** Color manipulation functions

---

## Quasar Directives

### ✅ v-ripple

```vue
<template>
  <!-- Basic ripple -->
  <div v-ripple class="cursor-pointer q-pa-md">
    Click me
  </div>

  <!-- Custom color -->
  <div v-ripple.primary class="cursor-pointer q-pa-md">
    Primary ripple
  </div>

  <!-- Early stop -->
  <div v-ripple.early class="cursor-pointer q-pa-md">
    Early stop ripple
  </div>

  <!-- Center -->
  <div v-ripple.center class="cursor-pointer q-pa-md">
    Center ripple
  </div>
</template>
```

### ✅ v-touch-pan

```vue
<script setup lang="ts">
const onPan = (details: any) => {
  console.log('Pan:', details)
  // details.evt - JS event
  // details.direction - 'left', 'right', 'up', 'down'
  // details.delta - { x, y }
  // details.distance - { x, y }
}
</script>

<template>
  <div
    v-touch-pan="onPan"
    class="bg-primary text-white q-pa-md"
  >
    Pan me
  </div>
</template>
```

### ✅ v-intersection

```vue
<script setup lang="ts">
const onIntersection = (entry: IntersectionObserverEntry) => {
  if (entry.isIntersecting) {
    console.log('Element is visible')
    // Load data, animate, etc.
  }
}
</script>

<template>
  <div v-intersection="onIntersection">
    Lazy loaded content
  </div>
</template>
```

**Sabab:**
- **Material Design:** Ripple effect
- **Touch gestures:** Pan, swipe, hold
- **Intersection observer:** Lazy loading, infinite scroll
- **Performance:** Native directive performance

---

## Theme Customization

### ✅ SASS Variables

```scss
// src/css/quasar.variables.scss

// Brand colors
$primary   : #1976D2;
$secondary : #26A69A;
$accent    : #9C27B0;

$dark      : #1d1d1d;

$positive  : #21BA45;
$negative  : #C10015;
$info      : #31CCEC;
$warning   : #F2C037;

// Typography
$body-font-size: 14px;
$body-line-height: 1.5;

// Spacing
$space-base: 16px;
$space-xs: 4px;
$space-sm: 8px;
$space-md: 16px;
$space-lg: 24px;
$space-xl: 48px;

// Border radius
$generic-border-radius: 4px;

// Shadows
$shadow-1: 0 1px 3px rgba(0, 0, 0, 0.2);
$shadow-2: 0 1px 5px rgba(0, 0, 0, 0.2);
```

### ✅ CSS Classes

```vue
<template>
  <!-- Spacing -->
  <div class="q-pa-md">Padding medium</div>
  <div class="q-ma-lg">Margin large</div>
  <div class="q-px-sm q-py-md">Horizontal sm, vertical md</div>

  <!-- Colors -->
  <div class="bg-primary text-white">Primary background</div>
  <div class="text-negative">Negative text</div>

  <!-- Typography -->
  <div class="text-h4">Heading 4</div>
  <div class="text-body1">Body text 1</div>
  <div class="text-weight-bold">Bold text</div>

  <!-- Flex -->
  <div class="row q-gutter-md">
    <div class="col">Column 1</div>
    <div class="col">Column 2</div>
  </div>

  <!-- Visibility -->
  <div class="gt-sm">Visible on > small screens</div>
  <div class="lt-md">Visible on < medium screens</div>
</template>
```

**Sabab:**
- **Consistent theming:** SASS variables
- **Utility classes:** Fast styling
- **Responsive:** Breakpoint-based classes
- **Material Design:** Material design spec

---

## Xulosa

Quasar specific guidelines:

1. **Configuration:** quasar.config.js setup
2. **Components:** Type-safe component usage
3. **Plugins:** Notify, Dialog, Loading
4. **Layout:** Responsive layout system
5. **Utilities:** Platform, Screen, Colors
6. **Directives:** v-ripple, v-touch, v-intersection
7. **Theme:** SASS variables, utility classes

**Quasar advantages:**
- All-in-one framework
- Material Design
- Responsive by default
- TypeScript support
- Great documentation
- Active community
- SSR/SPA/PWA/Mobile support

**Resources:**
- [Quasar Documentation](https://quasar.dev)
- [Quasar GitHub](https://github.com/quasarframework/quasar)
- [Quasar Discord](https://chat.quasar.dev)
