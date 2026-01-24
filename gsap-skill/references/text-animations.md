<overview>
SplitText is GSAP's text splitting plugin that divides HTML text into characters, words, and/or lines for granular animation control. As of GSAP 3.13 (2024), SplitText is FREE for all users including commercial projects.
</overview>

<setup>
```javascript
import gsap from "gsap";
import { SplitText } from "gsap/SplitText";

gsap.registerPlugin(SplitText);
```
</setup>

<basic_usage>
```javascript
// Split into characters
const split = new SplitText(".title", { type: "chars" });
// split.chars = array of character elements

// Split into words
const split = new SplitText(".title", { type: "words" });
// split.words = array of word elements

// Split into lines
const split = new SplitText(".title", { type: "lines" });
// split.lines = array of line elements

// Combined
const split = new SplitText(".title", { type: "chars, words, lines" });
// Access: split.chars, split.words, split.lines
```
</basic_usage>

<configuration>
```javascript
const split = new SplitText(".title", {
  type: "chars, words, lines",

  // Custom classes
  charsClass: "char",
  wordsClass: "word",
  linesClass: "line",

  // Masking for reveal effects
  mask: "chars",    // or "words" or "lines"

  // Responsive re-splitting
  autoSplit: true,

  // Callback when split (use for animations)
  onSplit: (self) => {
    // Animation code here - DOM guaranteed to exist
    gsap.from(self.chars, {
      y: 50,
      opacity: 0,
      stagger: 0.02
    });
  }
});
```
</configuration>

<masking>
Masking wraps split elements in a container with `overflow: clip` for reveal effects:

```javascript
// Without mask - characters animate in place
const split = new SplitText(".title", { type: "chars" });

// With mask - characters reveal from behind mask
const split = new SplitText(".title", {
  type: "chars",
  mask: "chars"  // creates clipping container
});

// Now animate from below the mask
gsap.from(split.chars, {
  yPercent: 100,  // start below visible area
  duration: 0.6,
  stagger: 0.02
});
```

**Visual difference:**
- Without mask: Characters fade/move from their animated position
- With mask: Characters slide up into view (hidden until they enter)
</masking>

<common_animations>
**Character reveal (fade + slide):**
```javascript
const split = new SplitText(".title", { type: "chars" });

gsap.from(split.chars, {
  y: 50,
  opacity: 0,
  duration: 0.6,
  ease: "power2.out",
  stagger: 0.02
});
```

**Masked line reveal (slide up):**
```javascript
const split = new SplitText(".title", {
  type: "lines",
  mask: "lines"
});

gsap.from(split.lines, {
  yPercent: 100,
  duration: 0.8,
  ease: "power3.out",
  stagger: 0.1
});
```

**Word rotation:**
```javascript
const split = new SplitText(".title", { type: "words" });

gsap.from(split.words, {
  y: 30,
  opacity: 0,
  rotationX: -90,
  transformOrigin: "top center",
  duration: 0.6,
  ease: "back.out(1.7)",
  stagger: 0.05
});
```

**Scramble text:**
```javascript
// No SplitText needed - uses TextPlugin
gsap.to(".title", {
  duration: 1,
  text: {
    value: "New Text Here",
    scramble: true,
    chars: "XO"  // characters to use during scramble
  }
});
```

**Typewriter effect:**
```javascript
gsap.to(".title", {
  duration: 2,
  text: "This text types out...",
  ease: "none"
});
```

**Random character animation:**
```javascript
const split = new SplitText(".title", { type: "chars" });

gsap.from(split.chars, {
  y: gsap.utils.random(-100, 100),
  opacity: 0,
  rotation: gsap.utils.random(-180, 180),
  duration: 0.8,
  ease: "power2.out",
  stagger: {
    each: 0.03,
    from: "random"
  }
});
```
</common_animations>

<scroll_triggered>
```javascript
const split = new SplitText(".title", {
  type: "chars",
  mask: "chars"
});

gsap.from(split.chars, {
  yPercent: 100,
  duration: 0.6,
  ease: "power2.out",
  stagger: 0.02,
  scrollTrigger: {
    trigger: ".title",
    start: "top 80%",
    toggleActions: "play none none reverse"
  }
});
```
</scroll_triggered>

<responsive>
Auto-resplit when viewport changes (prevents broken lines):

```javascript
const split = new SplitText(".title", {
  type: "lines",
  autoSplit: true,
  onSplit: (self) => {
    // This runs on initial split AND every resize
    gsap.from(self.lines, {
      y: 50,
      opacity: 0,
      stagger: 0.1
    });
  }
});
```
</responsive>

<cleanup>
**Revert to original HTML:**
```javascript
const split = new SplitText(".title", { type: "chars" });

// When done or on unmount
split.revert();
```

**In React:**
```javascript
useGSAP(() => {
  const split = new SplitText(".title", { type: "chars" });

  gsap.from(split.chars, {
    y: 50,
    opacity: 0,
    stagger: 0.02
  });

  return () => split.revert(); // cleanup
}, { scope: containerRef });
```
</cleanup>

<accessibility>
SplitText automatically adds `aria-label` with original text for screen readers:

```html
<!-- Before split -->
<h1>Hello World</h1>

<!-- After split -->
<h1 aria-label="Hello World">
  <div class="char">H</div>
  <div class="char">e</div>
  <!-- ... -->
</h1>
```
</accessibility>

<stagger_patterns>
```javascript
// From start (default)
stagger: 0.02

// From end (reverse)
stagger: { each: 0.02, from: "end" }

// From center outward
stagger: { each: 0.02, from: "center" }

// From edges inward
stagger: { each: 0.02, from: "edges" }

// Random order
stagger: { each: 0.02, from: "random" }

// Total time instead of each
stagger: { amount: 0.5, from: "start" } // 0.5s total spread
```
</stagger_patterns>

<performance_tips>
- **Chars:** Best for short text (titles, headlines). Creates many DOM elements.
- **Words:** Good balance for medium text.
- **Lines:** Best for paragraphs. Fewest DOM elements.

```javascript
// BAD for performance - 500+ elements
new SplitText(".long-paragraph", { type: "chars" });

// GOOD - ~20 elements
new SplitText(".long-paragraph", { type: "lines" });
```
</performance_tips>

<anti_patterns>
**DON'T:**
- Split long paragraphs into characters (performance)
- Forget to revert on cleanup
- Animate without `onSplit` callback (race condition)
- Use on complex nested HTML
- Forget to test text wrapping at different viewports

**DO:**
- Use `onSplit` callback for animations
- Use `autoSplit` for responsive layouts
- Revert splits on component unmount
- Use `mask` for clean reveal effects
- Test at multiple viewport widths
</anti_patterns>
