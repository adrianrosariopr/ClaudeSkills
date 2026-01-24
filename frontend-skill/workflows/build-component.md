# Workflow: Build Component

<required_reading>
**Read these reference files NOW:**
1. references/react-patterns.md
2. references/tailwind-patterns.md
3. references/accessibility.md
4. references/design-principles.md
</required_reading>

<context>
Building components from scratch requires: clear purpose, proper HTML semantics, accessible by default, styled consistently, and optimized for performance.
</context>

<process>

<step name="clarify-requirements">
Before building, understand:
- What does this component do?
- What are the variants/states? (hover, active, disabled, loading, error)
- What props does it need?
- Does it need to be controlled or uncontrolled?
- Are there accessibility requirements? (ARIA, keyboard)
- What's the design system context? (colors, spacing, typography)
</step>

<step name="choose-base-element">
Start with the right HTML:

| Component | Use |
|-----------|-----|
| Button | `<button>` (not `<div onClick>`) |
| Link | `<a href>` (not `<span onClick>`) |
| Input | `<input>` with `<label>` |
| List | `<ul>/<ol>` with `<li>` |
| Nav | `<nav>` with links |
| Modal | `<dialog>` |
| Tabs | `role="tablist"`, `role="tab"`, `role="tabpanel"` |

**Never use `<div>` for interactive elements.**
</step>

<step name="define-interface">
Design the props API:

```tsx
interface ButtonProps {
  /** Visual style variant */
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger';
  /** Size variant */
  size?: 'sm' | 'md' | 'lg';
  /** Shows loading spinner and disables interaction */
  loading?: boolean;
  /** Disabled state */
  disabled?: boolean;
  /** Full width button */
  fullWidth?: boolean;
  /** Button content */
  children: React.ReactNode;
  /** Click handler */
  onClick?: () => void;
}
```

Keep the API minimal. Add props only when needed.
</step>

<step name="implement-structure">
Build with proper semantics:

```tsx
function Button({
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
  fullWidth = false,
  children,
  onClick,
  ...props
}: ButtonProps) {
  return (
    <button
      type="button"
      disabled={disabled || loading}
      onClick={onClick}
      aria-busy={loading}
      className={cn(
        // Base styles
        'inline-flex items-center justify-center font-medium transition-colors',
        'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2',
        // Size variants
        {
          'h-8 px-3 text-sm': size === 'sm',
          'h-10 px-4 text-sm': size === 'md',
          'h-12 px-6 text-base': size === 'lg',
        },
        // Visual variants
        {
          'bg-primary text-white hover:bg-primary/90': variant === 'primary',
          'bg-secondary text-secondary-foreground hover:bg-secondary/80': variant === 'secondary',
          'hover:bg-accent hover:text-accent-foreground': variant === 'ghost',
          'bg-destructive text-destructive-foreground hover:bg-destructive/90': variant === 'danger',
        },
        // States
        {
          'opacity-50 cursor-not-allowed': disabled || loading,
          'w-full': fullWidth,
        }
      )}
      {...props}
    >
      {loading && <Spinner className="mr-2 h-4 w-4 animate-spin" />}
      {children}
    </button>
  );
}
```
</step>

<step name="add-accessibility">
Accessibility checklist:

```tsx
// Keyboard navigation
<button
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      onClick?.();
    }
  }}
>

// Screen reader context
<button aria-label="Close dialog">
  <XIcon aria-hidden="true" />
</button>

// Loading state announced
<button aria-busy={loading} aria-live="polite">
  {loading ? 'Saving...' : 'Save'}
</button>

// Disabled state
<button disabled aria-disabled="true">

// Focus management
<button ref={buttonRef}>
// Later: buttonRef.current?.focus()
```
</step>

<step name="handle-states">
All interactive states:

```tsx
// Visual states in Tailwind
className={cn(
  // Default
  'bg-primary text-white',
  // Hover
  'hover:bg-primary/90',
  // Focus
  'focus-visible:ring-2 focus-visible:ring-primary',
  // Active/pressed
  'active:scale-[0.98]',
  // Disabled
  'disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:bg-primary',
)}
```
</step>

<step name="add-variants">
Use CVA (Class Variance Authority) for complex variants:

```tsx
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  'inline-flex items-center justify-center font-medium transition-colors focus-visible:outline-none focus-visible:ring-2',
  {
    variants: {
      variant: {
        primary: 'bg-primary text-white hover:bg-primary/90',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        danger: 'bg-destructive text-white hover:bg-destructive/90',
      },
      size: {
        sm: 'h-8 px-3 text-sm',
        md: 'h-10 px-4 text-sm',
        lg: 'h-12 px-6 text-base',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  loading?: boolean;
}

function Button({ variant, size, loading, className, children, ...props }: ButtonProps) {
  return (
    <button className={cn(buttonVariants({ variant, size }), className)} {...props}>
      {loading && <Spinner className="mr-2" />}
      {children}
    </button>
  );
}
```
</step>

<step name="verify">
Test the component:

1. **Visual:** All variants render correctly
2. **Interaction:** Click, hover, focus states work
3. **Keyboard:** Tab to focus, Enter/Space to activate
4. **Screen reader:** Announces correctly (test with VoiceOver)
5. **Responsive:** Works at all breakpoints
6. **Loading:** Shows loading state, prevents double-click
7. **Disabled:** Cannot be interacted with
</step>

</process>

<anti_patterns>
Avoid:
- `<div onClick>` for buttons (use `<button>`)
- Missing focus states
- Disabled buttons without visual indication
- Loading state that allows re-clicking
- Props that duplicate HTML attributes
- Over-engineered API (keep it simple)
</anti_patterns>

<success_criteria>
- Semantic HTML base element
- TypeScript interface with JSDoc
- All states handled (hover, focus, active, disabled, loading)
- Keyboard accessible
- Screen reader friendly
- Tailwind styles with CVA for variants
- Works at all breakpoints
</success_criteria>
