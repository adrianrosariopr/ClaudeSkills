<overview>
Laravel frontend patterns including Blade templating, Livewire components, and Inertia.js with React/Vue. Patterns for building beautiful, responsive interfaces in Laravel applications.
</overview>

<blade_components>
**Anonymous components:**
```blade
{{-- resources/views/components/button.blade.php --}}
@props([
    'type' => 'button',
    'variant' => 'primary',
    'size' => 'md'
])

@php
    $variants = [
        'primary' => 'bg-blue-600 hover:bg-blue-700 text-white',
        'secondary' => 'bg-gray-200 hover:bg-gray-300 text-gray-800',
        'danger' => 'bg-red-600 hover:bg-red-700 text-white',
        'ghost' => 'hover:bg-gray-100 text-gray-700',
    ];

    $sizes = [
        'sm' => 'px-3 py-1.5 text-sm',
        'md' => 'px-4 py-2 text-base',
        'lg' => 'px-6 py-3 text-lg',
    ];

    $classes = "rounded-md font-medium transition-colors " .
               $variants[$variant] . " " . $sizes[$size];
@endphp

<button {{ $attributes->merge(['type' => $type, 'class' => $classes]) }}>
    {{ $slot }}
</button>
```

**Usage:**
```blade
<x-button>Primary Button</x-button>
<x-button variant="secondary" size="lg">Secondary</x-button>
<x-button variant="danger" class="mt-4">Delete</x-button>
```

**Card component:**
```blade
{{-- resources/views/components/card.blade.php --}}
@props(['header' => null, 'footer' => null])

<div {{ $attributes->merge(['class' => 'bg-white dark:bg-gray-800 rounded-lg shadow-sm border border-gray-200 dark:border-gray-700']) }}>
    @if($header)
        <div class="px-6 py-4 border-b border-gray-200 dark:border-gray-700">
            {{ $header }}
        </div>
    @endif

    <div class="px-6 py-4">
        {{ $slot }}
    </div>

    @if($footer)
        <div class="px-6 py-4 border-t border-gray-200 dark:border-gray-700 bg-gray-50 dark:bg-gray-900 rounded-b-lg">
            {{ $footer }}
        </div>
    @endif
</div>
```

**Usage:**
```blade
<x-card>
    <x-slot:header>
        <h3 class="text-lg font-semibold">Card Title</h3>
    </x-slot>

    <p>Card content goes here.</p>

    <x-slot:footer>
        <x-button>Save Changes</x-button>
    </x-slot>
</x-card>
```
</blade_components>

<blade_layouts>
**App layout:**
```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}" class="h-full">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ $title ?? config('app.name') }}</title>

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="h-full bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-gray-100">
    <div class="min-h-full">
        @include('layouts.navigation')

        @isset($header)
            <header class="bg-white dark:bg-gray-800 shadow">
                <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
                    {{ $header }}
                </div>
            </header>
        @endisset

        <main class="py-10">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                {{ $slot }}
            </div>
        </main>
    </div>

    @stack('scripts')
</body>
</html>
```

**Using layouts:**
```blade
<x-app-layout>
    <x-slot:header>
        <h1 class="text-2xl font-bold">Dashboard</h1>
    </x-slot>

    <x-card>
        <p>Welcome to your dashboard!</p>
    </x-card>
</x-app-layout>
```
</blade_layouts>

<livewire>
**Livewire component:**
```php
<?php
// app/Livewire/UserSearch.php
namespace App\Livewire;

use App\Models\User;
use Livewire\Component;
use Livewire\WithPagination;

class UserSearch extends Component
{
    use WithPagination;

    public string $search = '';
    public string $sortBy = 'name';
    public string $sortDirection = 'asc';

    public function updatingSearch()
    {
        $this->resetPage();
    }

    public function sort(string $column)
    {
        if ($this->sortBy === $column) {
            $this->sortDirection = $this->sortDirection === 'asc' ? 'desc' : 'asc';
        } else {
            $this->sortBy = $column;
            $this->sortDirection = 'asc';
        }
    }

    public function render()
    {
        $users = User::query()
            ->when($this->search, fn($q) => $q->where('name', 'like', "%{$this->search}%"))
            ->orderBy($this->sortBy, $this->sortDirection)
            ->paginate(10);

        return view('livewire.user-search', compact('users'));
    }
}
```

```blade
{{-- resources/views/livewire/user-search.blade.php --}}
<div>
    <div class="mb-4">
        <input
            type="text"
            wire:model.live.debounce.300ms="search"
            placeholder="Search users..."
            class="w-full px-4 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500"
        >
    </div>

    <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
            <tr>
                <th wire:click="sort('name')" class="cursor-pointer px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                    Name
                    @if($sortBy === 'name')
                        <span>{{ $sortDirection === 'asc' ? '↑' : '↓' }}</span>
                    @endif
                </th>
                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                    Email
                </th>
            </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
            @foreach($users as $user)
                <tr>
                    <td class="px-6 py-4 whitespace-nowrap">{{ $user->name }}</td>
                    <td class="px-6 py-4 whitespace-nowrap">{{ $user->email }}</td>
                </tr>
            @endforeach
        </tbody>
    </table>

    <div class="mt-4">
        {{ $users->links() }}
    </div>
</div>
```
</livewire>

<inertia_react>
**Inertia.js with React patterns:**

```typescript
// resources/js/Pages/Users/Index.tsx
import { Head, Link, router } from '@inertiajs/react';
import AppLayout from '@/Layouts/AppLayout';
import { User, PaginatedData } from '@/types';

interface Props {
    users: PaginatedData<User>;
    filters: {
        search: string;
    };
}

export default function UsersIndex({ users, filters }: Props) {
    const [search, setSearch] = useState(filters.search);

    const handleSearch = useDebouncedCallback((value: string) => {
        router.get('/users', { search: value }, { preserveState: true });
    }, 300);

    return (
        <AppLayout>
            <Head title="Users" />

            <div className="py-12">
                <div className="max-w-7xl mx-auto sm:px-6 lg:px-8">
                    <div className="bg-white overflow-hidden shadow-xl sm:rounded-lg">
                        <div className="p-6">
                            <input
                                type="text"
                                value={search}
                                onChange={(e) => {
                                    setSearch(e.target.value);
                                    handleSearch(e.target.value);
                                }}
                                placeholder="Search users..."
                                className="w-full px-4 py-2 border rounded-md"
                            />

                            <table className="min-w-full mt-4">
                                <thead>
                                    <tr>
                                        <th className="px-6 py-3 text-left">Name</th>
                                        <th className="px-6 py-3 text-left">Email</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    {users.data.map((user) => (
                                        <tr key={user.id}>
                                            <td className="px-6 py-4">{user.name}</td>
                                            <td className="px-6 py-4">{user.email}</td>
                                        </tr>
                                    ))}
                                </tbody>
                            </table>

                            <Pagination links={users.links} />
                        </div>
                    </div>
                </div>
            </div>
        </AppLayout>
    );
}
```

**Form handling with Inertia:**
```typescript
// resources/js/Pages/Users/Create.tsx
import { useForm } from '@inertiajs/react';

export default function CreateUser() {
    const { data, setData, post, processing, errors, reset } = useForm({
        name: '',
        email: '',
        password: '',
    });

    function handleSubmit(e: FormEvent) {
        e.preventDefault();
        post('/users', {
            onSuccess: () => reset(),
        });
    }

    return (
        <form onSubmit={handleSubmit} className="space-y-6">
            <div>
                <label className="block text-sm font-medium text-gray-700">
                    Name
                </label>
                <input
                    type="text"
                    value={data.name}
                    onChange={(e) => setData('name', e.target.value)}
                    className="mt-1 block w-full rounded-md border-gray-300"
                />
                {errors.name && (
                    <p className="mt-1 text-sm text-red-600">{errors.name}</p>
                )}
            </div>

            <div>
                <label className="block text-sm font-medium text-gray-700">
                    Email
                </label>
                <input
                    type="email"
                    value={data.email}
                    onChange={(e) => setData('email', e.target.value)}
                    className="mt-1 block w-full rounded-md border-gray-300"
                />
                {errors.email && (
                    <p className="mt-1 text-sm text-red-600">{errors.email}</p>
                )}
            </div>

            <button
                type="submit"
                disabled={processing}
                className="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 disabled:opacity-50"
            >
                {processing ? 'Creating...' : 'Create User'}
            </button>
        </form>
    );
}
```
</inertia_react>

<tailwind_laravel>
**Tailwind setup in Laravel:**
```javascript
// tailwind.config.js
export default {
    content: [
        './resources/**/*.blade.php',
        './resources/**/*.js',
        './resources/**/*.tsx',
        './resources/**/*.vue',
    ],
    darkMode: 'class',
    theme: {
        extend: {
            colors: {
                primary: {
                    50: '#eff6ff',
                    500: '#3b82f6',
                    600: '#2563eb',
                    700: '#1d4ed8',
                },
            },
        },
    },
    plugins: [
        require('@tailwindcss/forms'),
        require('@tailwindcss/typography'),
    ],
}
```

**App CSS:**
```css
/* resources/css/app.css */
@import "tailwindcss";

@layer components {
    .btn {
        @apply inline-flex items-center px-4 py-2 rounded-md font-medium transition-colors;
    }

    .btn-primary {
        @apply bg-primary-600 text-white hover:bg-primary-700;
    }

    .input {
        @apply block w-full rounded-md border-gray-300 shadow-sm
               focus:border-primary-500 focus:ring-primary-500;
    }
}
```
</tailwind_laravel>

<flash_messages>
**Flash message component:**
```blade
{{-- resources/views/components/flash-messages.blade.php --}}
@if (session('success'))
    <div
        x-data="{ show: true }"
        x-show="show"
        x-transition
        x-init="setTimeout(() => show = false, 5000)"
        class="mb-4 p-4 rounded-md bg-green-50 border border-green-200"
    >
        <div class="flex">
            <svg class="h-5 w-5 text-green-400" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"/>
            </svg>
            <p class="ml-3 text-sm font-medium text-green-800">
                {{ session('success') }}
            </p>
            <button @click="show = false" class="ml-auto text-green-500 hover:text-green-600">
                <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 20 20">
                    <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"/>
                </svg>
            </button>
        </div>
    </div>
@endif

@if (session('error'))
    <div
        x-data="{ show: true }"
        x-show="show"
        x-transition
        class="mb-4 p-4 rounded-md bg-red-50 border border-red-200"
    >
        <div class="flex">
            <svg class="h-5 w-5 text-red-400" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
            </svg>
            <p class="ml-3 text-sm font-medium text-red-800">
                {{ session('error') }}
            </p>
            <button @click="show = false" class="ml-auto text-red-500 hover:text-red-600">
                <svg class="h-5 w-5" fill="currentColor" viewBox="0 0 20 20">
                    <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"/>
                </svg>
            </button>
        </div>
    </div>
@endif
```
</flash_messages>

<dark_mode>
**Alpine.js dark mode toggle:**
```blade
<div
    x-data="{
        dark: localStorage.getItem('darkMode') === 'true',
        toggle() {
            this.dark = !this.dark;
            localStorage.setItem('darkMode', this.dark);
            document.documentElement.classList.toggle('dark', this.dark);
        }
    }"
    x-init="document.documentElement.classList.toggle('dark', dark)"
>
    <button @click="toggle()" class="p-2 rounded-full hover:bg-gray-100 dark:hover:bg-gray-800">
        <svg x-show="!dark" class="h-5 w-5 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"/>
        </svg>
        <svg x-show="dark" class="h-5 w-5 text-yellow-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"/>
        </svg>
    </button>
</div>
```
</dark_mode>

<anti_patterns>
**Avoid these Laravel frontend patterns:**

```blade
{{-- BAD: Logic in views --}}
@php
    $users = App\Models\User::where('active', true)->get();
@endphp
@foreach($users as $user)
    <li>{{ $user->name }}</li>
@endforeach

{{-- GOOD: Pass data from controller --}}
{{-- Controller: return view('users.index', ['users' => $users]); --}}
@foreach($users as $user)
    <li>{{ $user->name }}</li>
@endforeach

{{-- BAD: Inline styles --}}
<div style="margin-top: 20px; padding: 15px; background-color: #f3f4f6;">

{{-- GOOD: Tailwind classes --}}
<div class="mt-5 p-4 bg-gray-100">

{{-- BAD: Duplicating component code --}}
<button class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">Save</button>
<button class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">Submit</button>

{{-- GOOD: Use component --}}
<x-button>Save</x-button>
<x-button>Submit</x-button>

{{-- BAD: Not using @stack for scripts --}}
<script>
    // Inline script that might load before dependencies
</script>

{{-- GOOD: Use @push and @stack --}}
@push('scripts')
    <script>
        // Script added to proper location
    </script>
@endpush
```
</anti_patterns>
