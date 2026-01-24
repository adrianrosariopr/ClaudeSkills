<overview>
Vue 3 and Nuxt 3 frontend patterns for building beautiful, performant interfaces. Composition API, Nuxt UI components, and Vue-specific optimizations.
</overview>

<composition_api>
**Script setup syntax (recommended):**
```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';

// Reactive state
const count = ref(0);
const doubled = computed(() => count.value * 2);

// Methods
function increment() {
  count.value++;
}

// Lifecycle
onMounted(() => {
  console.log('Component mounted');
});
</script>

<template>
  <button @click="increment">
    Count: {{ count }} (doubled: {{ doubled }})
  </button>
</template>
```

**Props and emits:**
```vue
<script setup lang="ts">
interface Props {
  title: string;
  count?: number;
}

const props = withDefaults(defineProps<Props>(), {
  count: 0
});

const emit = defineEmits<{
  (e: 'update', value: number): void;
  (e: 'close'): void;
}>();

function handleClick() {
  emit('update', props.count + 1);
}
</script>
```
</composition_api>

<nuxt_ui>
**Nuxt UI component library (recommended for Nuxt projects):**

**Button:**
```vue
<template>
  <UButton color="primary" size="lg" icon="i-heroicons-arrow-right">
    Get Started
  </UButton>

  <UButton variant="outline" color="gray">
    Secondary Action
  </UButton>

  <UButton variant="ghost" :loading="isLoading">
    Submit
  </UButton>
</template>
```

**Card:**
```vue
<template>
  <UCard>
    <template #header>
      <h3 class="text-lg font-semibold">Card Title</h3>
    </template>

    <p>Card content goes here.</p>

    <template #footer>
      <UButton color="primary">Action</UButton>
    </template>
  </UCard>
</template>
```

**Form with validation:**
```vue
<script setup lang="ts">
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Must be at least 8 characters')
});

type Schema = z.output<typeof schema>;

const state = reactive<Partial<Schema>>({
  email: undefined,
  password: undefined
});

async function onSubmit(event: FormSubmitEvent<Schema>) {
  console.log(event.data);
}
</script>

<template>
  <UForm :schema="schema" :state="state" @submit="onSubmit">
    <UFormGroup label="Email" name="email">
      <UInput v-model="state.email" />
    </UFormGroup>

    <UFormGroup label="Password" name="password">
      <UInput v-model="state.password" type="password" />
    </UFormGroup>

    <UButton type="submit">Submit</UButton>
  </UForm>
</template>
```

**Table:**
```vue
<script setup lang="ts">
const columns = [
  { key: 'name', label: 'Name' },
  { key: 'email', label: 'Email' },
  { key: 'actions' }
];

const users = ref([
  { id: 1, name: 'John Doe', email: 'john@example.com' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
]);
</script>

<template>
  <UTable :columns="columns" :rows="users">
    <template #actions-data="{ row }">
      <UButton size="xs" color="gray" @click="editUser(row)">
        Edit
      </UButton>
    </template>
  </UTable>
</template>
```

**Modal:**
```vue
<script setup lang="ts">
const isOpen = ref(false);
</script>

<template>
  <UButton @click="isOpen = true">Open Modal</UButton>

  <UModal v-model="isOpen">
    <UCard>
      <template #header>
        <h3>Modal Title</h3>
      </template>

      <p>Modal content here.</p>

      <template #footer>
        <div class="flex justify-end gap-3">
          <UButton color="gray" @click="isOpen = false">Cancel</UButton>
          <UButton color="primary" @click="handleConfirm">Confirm</UButton>
        </div>
      </template>
    </UCard>
  </UModal>
</template>
```
</nuxt_ui>

<composables>
**Custom composables pattern:**
```typescript
// composables/useCounter.ts
export function useCounter(initial = 0) {
  const count = ref(initial);

  function increment() {
    count.value++;
  }

  function decrement() {
    count.value--;
  }

  function reset() {
    count.value = initial;
  }

  return {
    count: readonly(count),
    increment,
    decrement,
    reset
  };
}

// Usage in component
const { count, increment } = useCounter(10);
```

**Data fetching with useFetch:**
```vue
<script setup lang="ts">
// Auto-deduped, SSR-friendly
const { data: users, pending, error, refresh } = await useFetch('/api/users');

// With options
const { data: post } = await useFetch(`/api/posts/${id}`, {
  pick: ['title', 'content'],
  transform: (post) => ({
    ...post,
    createdAt: new Date(post.createdAt)
  })
});
</script>

<template>
  <div v-if="pending">Loading...</div>
  <div v-else-if="error">Error: {{ error.message }}</div>
  <ul v-else>
    <li v-for="user in users" :key="user.id">
      {{ user.name }}
    </li>
  </ul>
</template>
```
</composables>

<transitions>
**Built-in transition component:**
```vue
<template>
  <button @click="show = !show">Toggle</button>

  <Transition name="fade">
    <div v-if="show" class="box">Content</div>
  </Transition>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

**List transitions:**
```vue
<template>
  <TransitionGroup name="list" tag="ul">
    <li v-for="item in items" :key="item.id">
      {{ item.text }}
    </li>
  </TransitionGroup>
</template>

<style>
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}

.list-move {
  transition: transform 0.3s ease;
}
</style>
```
</transitions>

<performance>
**Lazy loading components:**
```vue
<script setup>
// Lazy load heavy components
const HeavyChart = defineAsyncComponent(() =>
  import('./HeavyChart.vue')
);

// With loading state
const AsyncModal = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  loadingComponent: LoadingSpinner,
  delay: 200,
  timeout: 3000
});
</script>
```

**v-once for static content:**
```vue
<template>
  <!-- Only rendered once, never re-evaluated -->
  <div v-once>
    <h1>{{ staticTitle }}</h1>
    <p>{{ staticDescription }}</p>
  </div>
</template>
```

**v-memo for expensive lists:**
```vue
<template>
  <div v-for="item in list" :key="item.id" v-memo="[item.id === selected]">
    <!-- Only re-renders when memo deps change -->
    <ExpensiveComponent :item="item" />
  </div>
</template>
```

**Shallowref for large objects:**
```typescript
// Use shallowRef when you replace entire object
const bigList = shallowRef<Item[]>([]);

function updateList(newList: Item[]) {
  bigList.value = newList; // Triggers reactivity
}
```
</performance>

<tailwind_integration>
**Using Tailwind with Vue:**
```vue
<script setup>
import { computed } from 'vue';

const props = defineProps<{
  variant: 'primary' | 'secondary' | 'danger';
  size: 'sm' | 'md' | 'lg';
}>();

const buttonClasses = computed(() => {
  const variants = {
    primary: 'bg-blue-600 hover:bg-blue-700 text-white',
    secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-800',
    danger: 'bg-red-600 hover:bg-red-700 text-white'
  };

  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };

  return [
    'rounded-md font-medium transition-colors',
    variants[props.variant],
    sizes[props.size]
  ].join(' ');
});
</script>

<template>
  <button :class="buttonClasses">
    <slot />
  </button>
</template>
```

**Class binding patterns:**
```vue
<template>
  <!-- Object syntax -->
  <div :class="{ 'bg-blue-500': isActive, 'opacity-50': isDisabled }">
    Content
  </div>

  <!-- Array syntax -->
  <div :class="[baseClasses, isActive ? 'bg-blue-500' : 'bg-gray-500']">
    Content
  </div>

  <!-- With Tailwind merge (recommended) -->
  <script setup>
  import { twMerge } from 'tailwind-merge';

  const props = defineProps<{ class?: string }>();
  const classes = computed(() =>
    twMerge('px-4 py-2 rounded', props.class)
  );
  </script>
</template>
```
</tailwind_integration>

<dark_mode>
**Dark mode with Nuxt Color Mode:**
```vue
<script setup>
const colorMode = useColorMode();
</script>

<template>
  <button @click="colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark'">
    <Icon :name="colorMode.value === 'dark' ? 'i-heroicons-sun' : 'i-heroicons-moon'" />
  </button>

  <div class="bg-white dark:bg-gray-900 text-gray-900 dark:text-white">
    Content adapts to color mode
  </div>
</template>
```
</dark_mode>

<anti_patterns>
**Avoid these Vue patterns:**

```vue
<!-- BAD: Mutating props -->
<script setup>
const props = defineProps<{ count: number }>();
props.count++; // DON'T DO THIS
</script>

<!-- GOOD: Emit to parent -->
<script setup>
const props = defineProps<{ count: number }>();
const emit = defineEmits<{ (e: 'update:count', value: number): void }>();

function increment() {
  emit('update:count', props.count + 1);
}
</script>

<!-- BAD: v-if and v-for on same element -->
<li v-for="user in users" v-if="user.isActive" :key="user.id">
  {{ user.name }}
</li>

<!-- GOOD: Filter first or use computed -->
<template v-for="user in users" :key="user.id">
  <li v-if="user.isActive">{{ user.name }}</li>
</template>

<!-- OR -->
<script setup>
const activeUsers = computed(() => users.filter(u => u.isActive));
</script>
<li v-for="user in activeUsers" :key="user.id">{{ user.name }}</li>

<!-- BAD: Inline complex logic -->
<div>{{ items.filter(i => i.active).map(i => i.name).join(', ') }}</div>

<!-- GOOD: Use computed -->
<script setup>
const activeNames = computed(() =>
  items.filter(i => i.active).map(i => i.name).join(', ')
);
</script>
<div>{{ activeNames }}</div>
```
</anti_patterns>
