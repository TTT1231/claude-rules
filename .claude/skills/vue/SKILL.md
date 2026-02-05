---
name: vue
description: 'Vue component development and code review skill - Development mode: generate high-quality Vue components/Composables from requirements - Review mode: /vue <path> to audit all Vue files in the path'
argument-hint: describe requirements or /vue <path>
user-invocable: true
paths:
   - '**/*.vue'
   - '**/composables/**/*.ts'
---

# Vue Component Development Skill

## Feature Overview

### Development Mode

Generate high-quality Vue code when users describe requirements:

```
Create a user list component
Wrap Element Plus Table component
Write a useTable composable
This component is too large, help me split it
```

**Trigger conditions**: User description contains keywords like "create", "generate", "wrap", "write", "split"

### Review Mode

When users use the `/vue <path>` command, audit Vue files in the specified path:

```
/vue src/components
/vue src/views/Dashboard.vue
/vue                          # Audit current directory
```

**Execution flow**:

1. Scan all `.vue` files in the specified path
2. Check each item in the review checklist
3. Output results (pass/fail) and improvement suggestions

---

## 1. Development Guide

### 1.1 Component Classification

| Type       | Characteristics                              | Examples                       |
| ---------- | -------------------------------------------- | ------------------------------ |
| **Presentational** | Pure UI, stateless, data from props | UserCard, StatusBadge          |
| **Container** | Contains state/logic, composes multiple components | UserList, Dashboard       |
| **Functional** | Reusable logic, no UI               | useTable, useForm (Composable) |

### 1.2 Component Wrapping

**When to use**: Wrapping component library components, must ensure original component's props/emits type hints are fully forwarded

**Key points**:

```vue
<script setup lang="ts">
import { useAttrs, useSlots } from 'vue'
import { ElButton } from 'element-plus'
import type { ButtonProps, ButtonEmits } from 'element-plus'

// Extended props
interface Props extends /* @vue-ignore */ ButtonProps {
   customProp?: string
}

defineOptions({ inheritAttrs: false })

const props = defineProps<Props>()
const attrs = useAttrs()
const slots = useSlots()

// Extended emits
const emit = defineEmits<{
   customEvent: [value: string]
}>()

// Expose original component methods
const buttonRef = ref<InstanceType<typeof ElButton>>()
defineExpose({
   focus: () => buttonRef.value?.focus()
})
</script>

<template>
   <ElButton ref="buttonRef" v-bind="attrs">
      <slot v-for="(_, name) in slots" :name="name" />
   </ElButton>
</template>
```

**Checklist**:

- Use `v-bind="$attrs"` or `v-bind="attrs"` to forward attributes
- Use `useSlots()` to forward all slots
- Inherit original component types (`extends ButtonProps`)
- Use `defineExpose` to expose original component methods

### 1.3 Large Component Splitting

**Splitting criteria**:

- **Primary**: Complex business logic (3+ independent business logics, mixed state management)
- **Secondary**: Code lines > 1300

**Splitting strategy**:

```
Split by UI region (first level)
  ↓
Split by concern separation (second level)
```

**Directory structure**:

```
UserTable/
├── UserTable.vue            # Main component
├── UserTableHeader.vue      # Table header
├── UserTableBody.vue        # Table body
├── UserTableFilters.vue     # Filters
└── composables/
    ├── useTableData.ts      # Data logic
    └── useTableFilters.ts   # Filter logic
```

### 1.4 Composable Rules

**Core rules**:

```typescript
// ✅ Single responsibility
export function useTableData() {
   /* Handle data only */
}
export function useTableFilters() {
   /* Handle filters only */
}

// ✅ Call API layer, don't use axios directly
import { getUserList } from '@/api/user'

export function useUserList() {
   const data = ref<User[]>([])
   const loading = ref(false)
   const error = ref<Error | null>(null)

   const fetch = async () => {
      loading.value = true
      try {
         data.value = await getUserList()
      } catch (e) {
         error.value = e as Error
      } finally {
         loading.value = false
      }
   }

   return { data, loading, error, fetch }
}

// ✅ Return reactive data correctly
return { data, loading } // Return refs directly
return state // Return reactive object as whole

// ❌ Wrong: destructuring reactive loses reactivity
return { ...state }
```

**Type exports**:

```typescript
export interface UseTableDataOptions {
   url: string
   immediate?: boolean
}

export interface UseTableDataReturn {
   data: Ref<DataItem[]>
   loading: Ref<boolean>
   fetch: () => Promise<void>
}

export function useTableData(options: UseTableDataOptions): UseTableDataReturn
```

### 1.5 Modern Vue Features

**defineModel (Vue 3.3+)**:

```vue
<script setup lang="ts">
// ✅ Simplify v-model
const modelValue = defineModel<string>({ required: true })
const title = defineModel<string>('title', { default: '' })
</script>

<template>
   <input v-model="modelValue" />
</template>
```

**useTemplateRef (Vue 3.5+)**:

```vue
<script setup lang="ts">
const inputRef = useTemplateRef<HTMLInputElement>('input')
</script>

<template>
   <input ref="input" />
</template>
```

**Generic components**:

```vue
<script setup lang="ts" generic="T">
interface Props {
   data: T[]
   columns: { key: keyof T; label: string }[]
}
defineProps<Props>()
</script>
```

---

## 2. Code Review Checklist

When using `/vue <path>` for review, check each item in the following checklist:

### 2.1 Component Structure

- [ ] Single file lines < 1300 (must split if exceeded)
- [ ] Template code < 500 lines
- [ ] Conditional nesting < 5 levels
- [ ] Props count < 15 (consider config object if exceeded)
- [ ] Complex business logic split into child components

### 2.2 TypeScript Types

- [ ] Props use `defineProps<Props>()` for type definition
- [ ] Emits use `defineEmits<Emits>()` for type definition
- [ ] No `any` type (must have explicit types)
- [ ] Async functions have return types
- [ ] Optional props use `withDefaults()` for default values

### 2.3 Component Wrapping

- [ ] Forward `$attrs` when wrapping components
- [ ] Forward all slots when wrapping components
- [ ] Inherit original component types (complete TypeScript hints)
- [ ] Use `defineExpose` to expose necessary methods
- [ ] No unnecessary wrapping (style-only changes don't need wrapping)

### 2.4 Composable

- [ ] Single responsibility (one Composable does one thing)
- [ ] Call API layer, don't use axios directly
- [ ] Async operations have try-catch error handling
- [ ] Return values have correct reactivity (no destructured reactive)
- [ ] Export parameter types and return types

### 2.5 Performance

- [ ] Long lists (>100 items) use virtual scrolling
- [ ] All `v-for` have unique `key` attributes
- [ ] Repeated calculations use `computed` for caching
- [ ] Large components use `defineAsyncComponent` for lazy loading
- [ ] Static content considers using `v-once`

### 2.6 Modern Syntax

- [ ] Vue 3.3+ uses `defineModel` to simplify v-model
- [ ] Vue 3.5+ uses `useTemplateRef` instead of string ref
- [ ] Use `useId()` to generate unique IDs
- [ ] Use generic components when needed

### 2.7 Code Standards

- [ ] Complex logic has explanatory comments
- [ ] Semantic naming (component names, variable names, function names)
- [ ] No duplicate code (>10 lines of identical code should be extracted)
- [ ] Component file names use PascalCase
- [ ] Composable file names use `use` prefix

### 2.8 Accessibility (a11y)

- [ ] Form elements have associated `<label>`
- [ ] Images have `alt` attributes
- [ ] Clickable elements are keyboard accessible (button or tabindex)
- [ ] External links use `rel="noopener noreferrer"`

### 2.9 Testing

- [ ] Core components have unit tests
- [ ] Composables have unit tests
- [ ] Tests cover main business logic

---

## 3. Quick Decision Tree

```
User request → Determine type

Is it pure logic reuse?
  → YES: Composable (useXxx)
  → NO: Continue

Is it wrapping a component library component?
  → YES: Wrapper (ensure type forwarding)
  → NO: Continue

Complex business logic OR >1300 lines?
  → YES: Split into child components + Composables
  → NO: Single-file component
```

---

## 4. Review Output Format

After review completion, output results in the following format:

```markdown
## 📋 [Filename] Review Results

### ✅ Passed Items

- [x] Component structure is reasonable
- [x] TypeScript types are complete
- [x] ...

### ❌ Issues

- [ ] **[Issue Title]**
   - Location: [file:line]
   - Problem: [description]
   - Suggestion: [improvement plan]

### 📊 Overall Score: X/10
```
