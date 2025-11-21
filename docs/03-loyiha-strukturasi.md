# Loyiha Strukturasi

## Mundarija
- [Tavsiya etilgan folder struktura](#tavsiya-etilgan-folder-struktura)
- [Folder strukturaning afzalliklari](#folder-strukturaning-afzalliklari)

---

## Tavsiya etilgan folder struktura

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

---

## Folder strukturaning afzalliklari

**Sabab:** Bu struktura:

### 1. Scalability (Kengayish qobiliyati)
Loyiha o'sishi bilan oson boshqariladi. Yangi feature qo'shishda, uning barcha qismlari (components, composables, stores) o'z joyida joylashadi.

### 2. Maintainability (Saqlash va ta'mirlash)
Har bir fayl o'z joyida, osongina topiladi. Developer yangi loyihaga qo'shilganda, fayl strukturasini tushunish oson.

### 3. Team Collaboration (Jamoa hamkorligi)
Jamoa a'zolari qayerda nima borligini tezda tushunadi. Git conflict kamayadi chunki har bir feature o'z folderida ishlaydi.

### 4. Feature Isolation (Feature ajratilishi)
Har bir feature mustaqil rivojlantirilishi mumkin. Bitta feature o'zgarganda, boshqalarga ta'sir qilmaydi.

---

## Folder tafsilotlari

### `src/assets/`
**Maqsad:** Statik fayllar (rasmlar, fontlar, global CSS)

```
assets/
├── images/
│   ├── logo.png
│   ├── banner.jpg
│   └── icons/
├── fonts/
│   ├── Roboto-Regular.ttf
│   └── Roboto-Bold.ttf
└── styles/
    ├── variables.scss
    └── global.scss
```

### `src/components/`
**Maqsad:** Reusable Vue komponentlar

```
components/
├── common/              # Base komponentlar
│   ├── BaseButton.vue
│   ├── BaseInput.vue
│   ├── BaseCard.vue
│   └── BaseModal.vue
├── layout/              # Layout strukturalar
│   ├── AppHeader.vue
│   ├── AppSidebar.vue
│   └── AppFooter.vue
└── features/            # Feature-specific
    ├── user/
    │   ├── UserCard.vue
    │   ├── UserList.vue
    │   └── UserProfileForm.vue
    └── product/
        ├── ProductCard.vue
        ├── ProductList.vue
        └── ProductDetail.vue
```

### `src/composables/`
**Maqsad:** Qayta ishlatiladigan composition functions

```
composables/
├── useAuth.ts           # Authentication logic
├── useApi.ts            # API calls
├── usePagination.ts     # Pagination logic
├── useDebounce.ts       # Debounce utility
└── useLocalStorage.ts   # LocalStorage wrapper
```

### `src/stores/`
**Maqsad:** Pinia state management stores

```
stores/
├── user.ts              # User store
├── auth.ts              # Authentication store
├── product.ts           # Product store
└── ui.ts                # UI state (sidebar, modals, etc)
```

### `src/services/`
**Maqsad:** Backend API va external service integratsiyalari

```
services/
├── api/
│   ├── user.ts         # User API calls
│   ├── product.ts      # Product API calls
│   └── auth.ts         # Auth API calls
├── http.ts             # Axios/Fetch wrapper
└── websocket.ts        # WebSocket service
```

### `src/types/`
**Maqsad:** TypeScript type va interface definitions

```
types/
├── models/
│   ├── user.ts         # User interfaces
│   ├── product.ts      # Product interfaces
│   └── common.ts       # Common types
├── api/
│   ├── request.ts      # API request types
│   └── response.ts     # API response types
└── injection-keys.ts   # Vue provide/inject keys
```

### `src/utils/`
**Maqsad:** Helper functions va utilities

```
utils/
├── formatters.ts       # Date, number, string formatters
├── validators.ts       # Validation functions
├── helpers.ts          # General helper functions
└── constants.ts        # Shared constants
```

### `src/pages/`
**Maqsad:** Route pages (Vue Router bilan ishlatiladigan)

```
pages/
├── HomePage.vue
├── AboutPage.vue
├── user/
│   ├── UserListPage.vue
│   ├── UserDetailPage.vue
│   └── UserEditPage.vue
└── product/
    ├── ProductListPage.vue
    └── ProductDetailPage.vue
```

### `src/router/`
**Maqsad:** Vue Router konfiguratsiyasi

```
router/
├── index.ts            # Main router config
├── routes.ts           # Route definitions
└── guards.ts           # Navigation guards
```

### `src/layouts/` (Quasar)
**Maqsad:** Quasar layout fayllar

```
layouts/
├── MainLayout.vue      # Main app layout
├── AuthLayout.vue      # Auth pages layout
└── EmptyLayout.vue     # Empty layout (login, 404)
```

---

## Naming Conventions

### Files
- **Components:** PascalCase - `UserCard.vue`, `ProductList.vue`
- **Composables:** camelCase with `use` prefix - `useAuth.ts`, `useApi.ts`
- **Stores:** camelCase - `user.ts`, `product.ts`
- **Types:** camelCase - `user.ts`, `api-response.ts`
- **Utils:** camelCase - `formatters.ts`, `validators.ts`

### Folders
- **Lowercase with hyphens:** `user-profile/`, `api-client/`
- **Feature-based grouping:** `features/user/`, `features/product/`

---

## Best Practices

### ✅ DO
- Feature bo'yicha guruplash (user, product, order)
- Har bir fayl bitta maqsadga xizmat qilishi kerak
- Umumiy komponentlar `common/` folderida
- Feature-specific komponentlar `features/` folderida
- Type definition'lar alohida `types/` folderida

### ❌ DON'T
- Barcha komponentlarni bitta `components/` folderga joylashmaslik
- `utils/` ni "qoldiq" folder sifatida ishlatmaslik
- Juda chuqur nested folderlar yaratmaslik (3-4 leveldan oshmaslik)
- Generic nomlar ishlatmaslik (`misc/`, `common/`, `stuff/`)

---

## Migration Strategy

Agar mavjud loyihangiz boshqacha struktura bo'lsa:

### 1-qadam: Yangi strukturani yaratish
```bash
mkdir -p src/{components/{common,layout,features},composables,stores,services/api,types/{models,api},utils}
```

### 2-qadam: Fayllarni bosqichma-bosqich ko'chirish
- Birinchi `common` komponentlarni ko'chiring
- Keyin feature-specific komponentlarni
- Nihoyat composables va stores

### 3-qadam: Import pathlarni yangilash
```typescript
// Eski
import UserCard from '@/components/UserCard.vue'

// Yangi
import UserCard from '@/components/features/user/UserCard.vue'
```

### 4-qadam: Path alias sozlash (tsconfig.json)
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@composables/*": ["./src/composables/*"],
      "@stores/*": ["./src/stores/*"],
      "@types/*": ["./src/types/*"]
    }
  }
}
```

---

## Xulosa

To'g'ri loyiha strukturasi:
- Development tezligini oshiradi
- Code maintainability yaxshilaydi
- Team collaboration osonlashtiradi
- Scalability ta'minlaydi
- Bug topish va tuzatish osonroq

**Eslatma:** Bu struktura tavsiya, har bir loyihaning o'z xususiyatlari bor. Lekin asosiy prinsiplar bir xil qoladi: **modularity, separation of concerns, va clear organization**.
