<overview>
Web accessibility (WCAG 2.2) fundamentals: semantic HTML, ARIA, keyboard navigation, screen readers, and creating inclusive interfaces.
</overview>

<semantic_html>
**Use the right element for the job:**

| Purpose | Use | Not |
|---------|-----|-----|
| Button (action) | `<button>` | `<div onClick>` |
| Link (navigation) | `<a href>` | `<span onClick>` |
| Heading | `<h1>-<h6>` | `<div class="heading">` |
| List | `<ul>/<ol>` | `<div>` with line breaks |
| Navigation | `<nav>` | `<div class="nav">` |
| Main content | `<main>` | `<div class="main">` |
| Form field | `<input>` + `<label>` | `<input>` alone |
| Table data | `<table>` | `<div>` grid |

**Document structure:**
```html
<body>
  <header>
    <nav aria-label="Main">...</nav>
  </header>

  <main>
    <h1>Page Title</h1>
    <article>
      <h2>Section</h2>
      <p>Content...</p>
    </article>
  </main>

  <aside aria-label="Related">
    ...
  </aside>

  <footer>...</footer>
</body>
```

**Heading hierarchy:**
```html
<!-- Correct: Logical hierarchy -->
<h1>Page Title</h1>
  <h2>Section</h2>
    <h3>Subsection</h3>
  <h2>Another Section</h2>

<!-- Wrong: Skipping levels -->
<h1>Page Title</h1>
  <h4>Section</h4>  <!-- Skipped h2, h3 -->
```
</semantic_html>

<aria>
**ARIA rules:**
1. Use semantic HTML first (ARIA is a last resort)
2. Don't change native semantics unless necessary
3. All interactive elements must be keyboard accessible
4. Don't hide focusable elements
5. All interactive elements need accessible names

**Common ARIA attributes:**

```html
<!-- Labels -->
<button aria-label="Close dialog">
  <XIcon aria-hidden="true" />
</button>

<button aria-labelledby="btn-label">
  <span id="btn-label" class="sr-only">Submit form</span>
  <ArrowIcon />
</button>

<!-- Descriptions -->
<input
  aria-describedby="password-help"
  type="password"
/>
<p id="password-help">Must be at least 8 characters</p>

<!-- States -->
<button aria-pressed="true">Toggle</button>
<button aria-expanded="false">Menu</button>
<div aria-busy="true">Loading...</div>
<input aria-invalid="true" />

<!-- Live regions (announce changes) -->
<div aria-live="polite">Status updates appear here</div>
<div aria-live="assertive">Urgent announcements</div>

<!-- Hidden from assistive tech -->
<span aria-hidden="true">🎉</span>  <!-- Decorative -->
```

**ARIA roles:**
```html
<!-- Only when HTML element doesn't exist -->
<div role="tablist">
  <button role="tab" aria-selected="true">Tab 1</button>
  <button role="tab" aria-selected="false">Tab 2</button>
</div>
<div role="tabpanel">Content 1</div>

<!-- Dialog/modal -->
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
>
  <h2 id="dialog-title">Confirm Action</h2>
  ...
</div>
```
</aria>

<keyboard_navigation>
**Focus order:**
- Follows DOM order (use CSS for visual order, not DOM manipulation)
- All interactive elements focusable
- Focus visible at all times

**Tab navigation:**
```html
<!-- Naturally focusable -->
<button>Focusable</button>
<a href="#">Focusable</a>
<input type="text" />

<!-- Make focusable -->
<div tabindex="0">Now focusable</div>

<!-- Remove from tab order (but still focusable via JS) -->
<button tabindex="-1">Skip in tab order</button>

<!-- Never use tabindex > 0 -->
```

**Keyboard interactions:**
| Component | Keys |
|-----------|------|
| Button | Enter, Space |
| Link | Enter |
| Checkbox | Space |
| Radio | Arrow keys |
| Select | Arrow keys, Enter |
| Dialog | Escape to close |
| Menu | Arrow keys, Enter, Escape |
| Tabs | Arrow keys |

**Focus management:**
```tsx
// Focus trap in modal
function Modal({ isOpen, onClose, children }) {
  const firstFocusableRef = useRef(null);

  useEffect(() => {
    if (isOpen) {
      firstFocusableRef.current?.focus();
    }
  }, [isOpen]);

  return (
    <dialog open={isOpen}>
      <button ref={firstFocusableRef} onClick={onClose}>
        Close
      </button>
      {children}
    </dialog>
  );
}

// Return focus after closing
function ModalTrigger() {
  const triggerRef = useRef(null);
  const [isOpen, setIsOpen] = useState(false);

  const handleClose = () => {
    setIsOpen(false);
    triggerRef.current?.focus();  // Return focus to trigger
  };

  return (
    <>
      <button ref={triggerRef} onClick={() => setIsOpen(true)}>
        Open Modal
      </button>
      <Modal isOpen={isOpen} onClose={handleClose}>
        Modal content
      </Modal>
    </>
  );
}
```

**Focus styles:**
```css
/* Visible focus indicator */
:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* Remove default outline only if custom is applied */
:focus:not(:focus-visible) {
  outline: none;
}
```
</keyboard_navigation>

<screen_readers>
**Testing tools:**
- macOS: VoiceOver (Cmd + F5)
- Windows: NVDA (free), JAWS
- iOS: VoiceOver (Settings > Accessibility)
- Android: TalkBack

**Screen reader only content:**
```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

```html
<!-- Icon-only button -->
<button>
  <XIcon aria-hidden="true" />
  <span class="sr-only">Close dialog</span>
</button>

<!-- Skip link -->
<a href="#main-content" class="sr-only focus:not-sr-only">
  Skip to main content
</a>

<!-- Live announcements -->
<div aria-live="polite" class="sr-only">
  {announcement}
</div>
```

**What screen readers announce:**
- Text content
- Element role (button, link, heading level)
- Element name (from label, aria-label)
- Element state (expanded, selected, checked)
- Element description (aria-describedby)
</screen_readers>

<forms>
**Always associate labels:**
```html
<!-- Explicit association -->
<label for="email">Email</label>
<input id="email" type="email" />

<!-- Implicit association -->
<label>
  Email
  <input type="email" />
</label>
```

**Error handling:**
```html
<div class="form-group">
  <label for="password">Password</label>
  <input
    id="password"
    type="password"
    aria-describedby="password-error password-help"
    aria-invalid="true"
  />
  <p id="password-help">Must be at least 8 characters</p>
  <p id="password-error" class="error" role="alert">
    Password is required
  </p>
</div>
```

**Required fields:**
```html
<label for="name">
  Name <span aria-hidden="true">*</span>
  <span class="sr-only">(required)</span>
</label>
<input id="name" required aria-required="true" />
```
</forms>

<color_contrast>
**WCAG requirements:**
- Normal text: 4.5:1
- Large text (18px+ or 14px+ bold): 3:1
- UI components: 3:1

**Testing tools:**
- Chrome DevTools: Inspect element > Contrast ratio
- axe DevTools extension
- WebAIM Contrast Checker
- Stark (Figma plugin)

**Don't rely on color alone:**
```html
<!-- Bad: Color only -->
<span class="text-red-500">Error</span>

<!-- Good: Color + icon + text -->
<span class="text-red-500">
  <ErrorIcon aria-hidden="true" />
  Error: Password is required
</span>
```
</color_contrast>

<motion>
**Respect user preferences:**
```css
/* Reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

```tsx
// React hook for reduced motion
function usePrefersReducedMotion() {
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReducedMotion(mediaQuery.matches);

    const handler = (e) => setPrefersReducedMotion(e.matches);
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, []);

  return prefersReducedMotion;
}
```
</motion>

<accessibility_checklist>
```markdown
Semantic HTML
- [ ] Using correct HTML elements
- [ ] Logical heading hierarchy
- [ ] Landmarks used (nav, main, aside, footer)

Keyboard
- [ ] All interactive elements focusable
- [ ] Focus order makes sense
- [ ] Focus indicator visible
- [ ] No keyboard traps
- [ ] Escape closes modals/menus

Screen Readers
- [ ] Images have alt text
- [ ] Icons have accessible names
- [ ] Form fields have labels
- [ ] Errors announced
- [ ] Dynamic content uses aria-live

Visual
- [ ] Color contrast meets WCAG
- [ ] Not relying on color alone
- [ ] Text resizable to 200%
- [ ] Content reflows at 320px

Motion
- [ ] Respects prefers-reduced-motion
- [ ] No auto-playing video/audio
- [ ] Animations can be paused
```
</accessibility_checklist>
