<overview>
WCAG 2.2, released October 2023, adds 9 new success criteria. This reference covers what's new and how to implement the changes.
</overview>

<summary>
**What's New in WCAG 2.2**

- 9 new success criteria (6 Level AA, 3 Level AAA)
- Removes 4.1.1 Parsing (no longer needed)
- Focuses on cognitive disabilities, mobile/touch, and users who have difficulty with fine motor control
- Backward compatible with WCAG 2.1
</summary>

<new_criteria>

<criterion id="2.4.11" level="AA">
**2.4.11 Focus Not Obscured (Minimum)**

When an element receives keyboard focus, it must not be entirely hidden by other content.

**Problem it solves:** Sticky headers or footers covering focused elements, making it impossible for keyboard users to see where they are.

**Requirements:**
- Focused element must be at least partially visible
- Content like sticky headers/footers, modals, or chat widgets must not completely cover focused items

**Implementation:**
```css
/* Ensure sticky header doesn't cover focused content */
:target {
  scroll-margin-top: 80px; /* Height of sticky header */
}

/* Or use scroll-padding on the container */
html {
  scroll-padding-top: 80px;
}
```

**Testing:**
- Tab through page with sticky header
- Ensure focused elements are never fully hidden
</criterion>

<criterion id="2.4.12" level="AAA">
**2.4.12 Focus Not Obscured (Enhanced)**

The focused element must be fully visible (not just partially).

**Same as 2.4.11 but stricter** - no part of the focused element can be hidden.
</criterion>

<criterion id="2.4.13" level="AAA">
**2.4.13 Focus Appearance**

Focus indicators must meet minimum size and contrast requirements.

**Requirements:**
- Focus indicator must be at least 2px thick
- Must have 3:1 contrast ratio between focused and unfocused states
- Must contrast with adjacent colors

**Implementation:**
```css
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
}

/* Or use box-shadow for more control */
:focus-visible {
  outline: none;
  box-shadow: 0 0 0 3px #005fcc;
}
```

**Note:** Browser default focus styles pass if not modified by the author.
</criterion>

<criterion id="2.5.7" level="AA">
**2.5.7 Dragging Movements**

Any functionality that uses dragging must also be operable without dragging.

**Problem it solves:** Users with motor disabilities or using voice control cannot perform drag-and-drop.

**What requires alternatives:**
- Drag and drop interfaces
- Sliders
- Sortable lists
- Custom carousels
- Map panning

**Implementation examples:**

**Sortable list:**
```html
<!-- Drag works, but also provide buttons -->
<li draggable="true">
  Item 1
  <button aria-label="Move up">↑</button>
  <button aria-label="Move down">↓</button>
</li>
```

**Slider:**
```html
<!-- Native range input works with arrow keys -->
<input type="range" min="0" max="100" value="50">

<!-- Or provide text input alternative -->
<input type="number" min="0" max="100" value="50">
```

**Drag and drop:**
```html
<!-- Provide click-to-select alternative -->
<button onclick="selectItem(1)">Select Item 1</button>
<button onclick="moveTo('folder-a')">Move to Folder A</button>
```

**Exceptions:**
- Dragging is essential (like a drawing app)
- Functionality is determined by user agent
</criterion>

<criterion id="2.5.8" level="AA">
**2.5.8 Target Size (Minimum)**

Interactive elements must be at least 24x24 CSS pixels.

**Replaces** 2.5.5 Target Size (Enhanced) which required 44x44px at Level AAA.

**Requirements:**
- Touch/click targets must be at least 24x24 CSS pixels, OR
- Have sufficient spacing from other targets (so combined area is equivalent)

**Implementation:**
```css
/* Minimum touch target size */
button, a, input, [role="button"] {
  min-width: 24px;
  min-height: 24px;
}

/* Better: larger targets (44px recommended) */
button, a {
  min-width: 44px;
  min-height: 44px;
  padding: 10px;
}

/* Inline links: ensure adequate spacing */
.nav-links a {
  display: inline-block;
  padding: 8px 12px;
  margin: 4px;
}
```

**Exceptions:**
- Links within sentences (inline text)
- User agent controlled (native checkboxes, radios)
- Essential (target size is required for information)
- Equivalent alternative exists
</criterion>

<criterion id="3.2.6" level="A">
**3.2.6 Consistent Help**

If a page provides help mechanisms, they must appear in consistent locations across pages.

**Help mechanisms include:**
- Contact information (phone, email)
- Human contact (chat, form)
- Self-help (FAQ, instructions)
- Automated help (chatbot)

**Requirements:**
- Help appears in same relative order on each page
- Help appears in same location (e.g., always in header or footer)

**Implementation:**
```html
<!-- Consistent help in footer across all pages -->
<footer>
  <nav aria-label="Help">
    <a href="/contact">Contact Us</a>
    <a href="/faq">FAQ</a>
    <a href="/support">Support</a>
  </nav>
</footer>
```

**Note:** Doesn't require help to exist, just that if it does, it's consistent.
</criterion>

<criterion id="3.3.7" level="A">
**3.3.7 Redundant Entry**

Information already provided should be auto-populated or available for selection.

**Problem it solves:** Users with cognitive or memory disabilities shouldn't need to re-enter information multiple times in the same session.

**Requirements:**
- Auto-populate previously entered info, OR
- Provide selection from previous entries

**Examples:**

**Shipping = Billing address:**
```html
<label>
  <input type="checkbox" onchange="copyAddress()">
  Shipping address same as billing
</label>
```

**Multi-step forms:**
```html
<!-- Store and recall previous entries -->
<label for="email">Email</label>
<input type="email" id="email" value="user@example.com" readonly>
```

**Exceptions:**
- Re-entry is essential (security verification)
- Information is no longer valid
</criterion>

<criterion id="3.3.8" level="AA">
**3.3.8 Accessible Authentication (Minimum)**

Authentication must not require cognitive function tests (unless alternatives exist).

**What's prohibited (without alternative):**
- Remembering passwords (must allow paste/autofill)
- Solving puzzles (CAPTCHA without alternative)
- Transcribing characters
- Performing calculations
- Object recognition (unless personal objects)

**What's allowed:**
- Password managers (allow paste)
- Autofill
- Copy/paste from email link
- Biometrics (fingerprint, face)
- Hardware tokens
- Personal object recognition (your photos)

**Implementation:**
```html
<!-- GOOD: Allow paste in password fields -->
<input type="password" id="password" autocomplete="current-password">

<!-- GOOD: Provide multiple auth options -->
<button>Sign in with password</button>
<button>Sign in with email link</button>
<button>Sign in with passkey</button>

<!-- BAD: Preventing paste -->
<input type="password" onpaste="return false">
```
</criterion>

<criterion id="3.3.9" level="AAA">
**3.3.9 Accessible Authentication (Enhanced)**

Same as 3.3.8 but no cognitive function tests at all (even with alternatives).

Biometrics and hardware tokens are the main compliant methods.
</criterion>

</new_criteria>

<removed_criteria>

**4.1.1 Parsing (Removed)**

Previously required:
- No duplicate IDs
- Complete start/end tags
- Proper nesting
- Unique attributes

**Why removed:** Modern browsers and assistive technologies handle parsing errors gracefully. The criterion was obsolete.

**Still good practice** to write valid HTML, but it's no longer a WCAG requirement.
</removed_criteria>

<implementation_checklist>
**WCAG 2.2 AA Checklist (New Criteria)**

- [ ] **2.4.11 Focus Not Obscured:** Focused elements not hidden by sticky content
- [ ] **2.5.7 Dragging:** All drag functions have non-drag alternative
- [ ] **2.5.8 Target Size:** Touch targets at least 24x24px (or adequately spaced)
- [ ] **3.2.6 Consistent Help:** Help in same location across pages
- [ ] **3.3.7 Redundant Entry:** Previous entries auto-populated or selectable
- [ ] **3.3.8 Accessible Auth:** Password paste allowed, alternative to cognitive tests
</implementation_checklist>

<migration_from_21>
**Upgrading from WCAG 2.1 to 2.2**

WCAG 2.2 is backward compatible. If you're 2.1 AA compliant, you need to add:

1. Check focus visibility with sticky headers
2. Add alternatives to drag operations
3. Ensure touch targets are 24px+ or spaced
4. Keep help mechanisms consistent
5. Auto-populate repeated form fields
6. Don't block password paste/autofill

**Typical effort:** 2-4 weeks for most sites with moderate work.
</migration_from_21>
