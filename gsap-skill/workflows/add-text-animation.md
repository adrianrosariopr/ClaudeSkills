# Workflow: Add Text Animations

<required_reading>
**Read these reference files NOW:**
1. references/text-animations.md
2. references/creative-effects.md
3. references/timeline-patterns.md
</required_reading>

<process>

<step name="register-plugin">
## Step 1: Register SplitText

```javascript
import gsap from "gsap";
import { SplitText } from "gsap/SplitText";

gsap.registerPlugin(SplitText);
```

**Note:** SplitText is now FREE (GSAP 3.13+, thanks to Webflow acquisition).
</step>

<step name="choose-split-type">
## Step 2: Choose Split Type

**Split into characters:**
```javascript
const split = new SplitText(".title", { type: "chars" });
// split.chars = array of character elements
```

**Split into words:**
```javascript
const split = new SplitText(".title", { type: "words" });
// split.words = array of word elements
```

**Split into lines:**
```javascript
const split = new SplitText(".title", { type: "lines" });
// split.lines = array of line elements
```

**Combined split:**
```javascript
const split = new SplitText(".title", { type: "chars, words, lines" });
// Access: split.chars, split.words, split.lines
```
</step>

<step name="configure-split">
## Step 3: Configure SplitText

**Basic configuration:**
```javascript
const split = new SplitText(".title", {
  type: "chars, words",
  charsClass: "char",       // class for each character
  wordsClass: "word",       // class for each word
  linesClass: "line"        // class for each line
});
```

**With masking for reveal effects:**
```javascript
const split = new SplitText(".title", {
  type: "chars",
  mask: "chars"  // wraps each char in clip container
});
```

**Responsive re-splitting:**
```javascript
const split = new SplitText(".title", {
  type: "lines",
  autoSplit: true,  // re-splits on resize
  onSplit: (self) => {
    // Animation logic here - guaranteed DOM exists
    gsap.from(self.lines, {
      y: 50,
      opacity: 0,
      stagger: 0.1
    });
  }
});
```
</step>

<step name="build-animation">
## Step 4: Build Text Animation

**Character reveal (fade + slide up):**
```javascript
const split = new SplitText(".title", { type: "chars" });

gsap.from(split.chars, {
  y: 50,
  opacity: 0,
  duration: 0.6,
  ease: "power2.out",
  stagger: 0.02  // 20ms between each character
});
```

**Masked line reveal (text slides up from behind mask):**
```javascript
const split = new SplitText(".title", {
  type: "lines",
  mask: "lines"
});

gsap.from(split.lines, {
  yPercent: 100,  // slides up from below
  duration: 0.8,
  ease: "power3.out",
  stagger: 0.1
});
```

**Word-by-word with rotation:**
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

**Scramble text effect:**
```javascript
gsap.to(".title", {
  duration: 1,
  text: {
    value: "New Text Here",
    scramble: true,      // random characters during transition
    chars: "XO"          // characters to use for scramble
  }
});
```
</step>

<step name="add-scroll-trigger">
## Step 5: Add ScrollTrigger (if needed)

```javascript
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

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
</step>

<step name="handle-cleanup">
## Step 6: Handle Cleanup

**Revert split (restore original HTML):**
```javascript
const split = new SplitText(".title", { type: "chars" });

// After animation completes or on unmount
split.revert();
```

**In React/Next.js:**
```javascript
import { useGSAP } from "@gsap/react";

function AnimatedTitle() {
  const containerRef = useRef();

  useGSAP(() => {
    const split = new SplitText(".title", { type: "chars" });

    gsap.from(split.chars, {
      y: 50,
      opacity: 0,
      stagger: 0.02
    });

    // Cleanup handled automatically by useGSAP
    return () => split.revert();
  }, { scope: containerRef });

  return (
    <div ref={containerRef}>
      <h1 className="title">Animated Text</h1>
    </div>
  );
}
```
</step>

<step name="verify">
## Step 7: Verify

```javascript
// Check split created elements
console.log("Characters:", split.chars.length);
console.log("Words:", split.words.length);
console.log("Lines:", split.lines.length);

// Verify accessibility
// SplitText adds aria-label automatically for screen readers

// Test resize behavior (if autoSplit enabled)
// Resize window and verify text re-splits correctly

// Check performance
// Many characters = many DOM elements = potential performance issue
// Consider using words/lines for long text
```
</step>

</process>

<anti_patterns>
**Avoid:**
- Splitting very long paragraphs into chars (performance issue)
- Forgetting to revert split on cleanup
- Animating split text without `onSplit` callback (race condition)
- Using SplitText on elements with complex nested HTML
- Not testing text wrapping at different viewport widths
</anti_patterns>

<success_criteria>
A well-built text animation:
- Uses appropriate split type (chars for short titles, lines for paragraphs)
- Has smooth stagger timing (0.02-0.05s for chars, 0.1-0.2s for lines)
- Uses masking for clean reveal effects
- Handles resize with autoSplit (if needed)
- Reverts split on cleanup
- Maintains accessibility (aria-label)
- Performs smoothly (not too many split elements)
</success_criteria>
