# Template Qoidalari

## Mundarija
- [Template Logic](#template-logic)
- [v-if va v-for](#v-if-va-v-for)
- [v-for Key Handling](#v-for-key-handling)
- [v-html Security](#v-html-security)
- [Event Handlers](#event-handlers)
- [Optional Chaining](#optional-chaining)
- [Method Chaining](#method-chaining)
- [v-model Modifiers](#v-model-modifiers)

---

## Template Logic

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

## v-if va v-for

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

## v-for Key Handling

### ❌ NOTO'G'RI - Index ni key sifatida ishlatish

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

## v-html Security

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

## Event Handlers

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

## Optional Chaining

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

## Method Chaining

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

## v-model Modifiers

### ❌ NOTO'G'RI - Manual trim va type conversion

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

## Direct DOM Manipulation

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

## Xulosa

Template qoidalari:

1. **Murakkab logic computed ga** - Template oddiy va o'qilishi oson
2. **v-if va v-for bitta elementda emas** - Computed yoki wrapper element
3. **Unique ID key sifatida** - Index ishlatmaslik
4. **v-html ehtiyotkorlik bilan** - Faqat sanitized content
5. **Event handlers method sifatida** - Inline logic yo'q
6. **Optional chaining computed da** - Default values bilan
7. **Method chaining computed da** - Template tozaligi
8. **v-model modifiers** - Built-in modifiers ishlatish
9. **Vue reactive system** - Direct DOM manipulation yo'q

Template - presentational layer, logic computed/methods da!
