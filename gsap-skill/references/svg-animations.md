<overview>
GSAP provides powerful SVG animation capabilities through plugins like DrawSVG (stroke drawing effects) and MorphSVG (shape morphing). As of GSAP 3.13, all SVG plugins are FREE.
</overview>

<setup>
```javascript
import gsap from "gsap";
import { DrawSVGPlugin } from "gsap/DrawSVGPlugin";
import { MorphSVGPlugin } from "gsap/MorphSVGPlugin";
import { MotionPathPlugin } from "gsap/MotionPathPlugin";

gsap.registerPlugin(DrawSVGPlugin, MorphSVGPlugin, MotionPathPlugin);
```
</setup>

<basic_svg_animation>
Animate SVG attributes directly:

```javascript
// Scale and rotate SVG element
gsap.to("#circle", {
  scale: 1.5,
  rotation: 360,
  transformOrigin: "center center"
});

// Animate fill color
gsap.to("#rect", {
  fill: "#ff0000",
  stroke: "#00ff00",
  strokeWidth: 5
});

// Animate path data (simple changes)
gsap.to("#path", {
  attr: { d: "M10,10 L100,100" }
});
```
</basic_svg_animation>

<drawsvg>
DrawSVG animates the stroke of SVG paths to create "drawing" effects.

<drawsvg_basic>
```javascript
// Draw from 0% to 100%
gsap.from("#line", {
  drawSVG: 0,  // starts at 0%, animates to 100%
  duration: 2
});

// Draw from 0% to 50%
gsap.to("#line", {
  drawSVG: "0% 50%",
  duration: 2
});

// Draw middle section
gsap.to("#line", {
  drawSVG: "25% 75%",
  duration: 2
});
```
</drawsvg_basic>

<drawsvg_patterns>
**Line drawing animation:**
```javascript
gsap.set("#path", { drawSVG: 0 });

gsap.to("#path", {
  drawSVG: "100%",
  duration: 2,
  ease: "power2.inOut"
});
```

**Self-drawing logo:**
```javascript
const paths = gsap.utils.toArray("#logo path");

gsap.set(paths, { drawSVG: 0 });

gsap.to(paths, {
  drawSVG: "100%",
  duration: 1.5,
  stagger: 0.2,
  ease: "power2.inOut"
});
```

**Erase effect:**
```javascript
gsap.to("#path", {
  drawSVG: "100% 100%",  // both ends at 100% = invisible
  duration: 1
});
```

**Animated underline:**
```javascript
// On hover
gsap.to(".underline-path", {
  drawSVG: "0% 100%",
  duration: 0.3,
  ease: "power2.out"
});

// On hover out
gsap.to(".underline-path", {
  drawSVG: "100% 100%",
  duration: 0.3,
  ease: "power2.in"
});
```
</drawsvg_patterns>

<drawsvg_requirements>
DrawSVG only works on elements with a stroke:

```html
<svg>
  <!-- Works - has stroke -->
  <path stroke="#000" fill="none" d="M10,10 L100,100" />

  <!-- Doesn't work - no stroke -->
  <rect fill="#000" width="100" height="100" />
</svg>
```

```css
/* Ensure stroke is visible */
#path {
  stroke: #000;
  stroke-width: 2;
  fill: none;
}
```
</drawsvg_requirements>
</drawsvg>

<morphsvg>
MorphSVG morphs one SVG path into another.

<morphsvg_basic>
```javascript
// Morph one path into another
gsap.to("#diamond", {
  morphSVG: "#lightning",
  duration: 1
});

// Morph to path data string
gsap.to("#shape", {
  morphSVG: "M10,10 L50,90 L90,10 Z",
  duration: 1
});
```
</morphsvg_basic>

<morphsvg_options>
```javascript
gsap.to("#shape1", {
  morphSVG: {
    shape: "#shape2",

    // Matching algorithm
    type: "size",      // "size" (default), "position", "complexity"

    // Origin point for rotation
    origin: "50% 50%",

    // Add rotation during morph
    rotational: true,
    rotation: 180,

    // Preserve stroke direction
    shapeIndex: 0      // auto-calculated, or specify
  },
  duration: 1
});
```

**Type options:**
- `"size"` - Matches segments by size (default, usually best)
- `"position"` - Matches by proximity
- `"complexity"` - Matches by anchor point count (fastest)
</morphsvg_options>

<morphsvg_convert>
Convert basic shapes to paths for morphing:

```javascript
// Convert all shapes in an SVG to paths
MorphSVGPlugin.convertToPath("circle, rect, ellipse, line, polygon, polyline");

// Convert specific elements
MorphSVGPlugin.convertToPath("#myCircle, #myRect");
```

Now you can morph between any shapes:
```javascript
MorphSVGPlugin.convertToPath("#circle, #star");

gsap.to("#circle", {
  morphSVG: "#star",
  duration: 1
});
```
</morphsvg_convert>

<morphsvg_patterns>
**Shape transformation:**
```javascript
const tl = gsap.timeline({ repeat: -1, yoyo: true });

tl.to("#shape", { morphSVG: "#star", duration: 0.5 })
  .to("#shape", { morphSVG: "#heart", duration: 0.5 })
  .to("#shape", { morphSVG: "#circle", duration: 0.5 });
```

**Icon morph on hover:**
```javascript
const icon = document.querySelector("#icon");
const playPath = "#playIcon path";
const pausePath = "#pauseIcon path";

icon.addEventListener("mouseenter", () => {
  gsap.to(playPath, { morphSVG: pausePath, duration: 0.3 });
});

icon.addEventListener("mouseleave", () => {
  gsap.to(playPath, { morphSVG: playPath, duration: 0.3 });
});
```
</morphsvg_patterns>
</morphsvg>

<motionpath>
Animate elements along SVG paths.

<motionpath_basic>
```javascript
gsap.to("#element", {
  motionPath: {
    path: "#motionPath",
    align: "#motionPath",
    alignOrigin: [0.5, 0.5],
    autoRotate: true
  },
  duration: 5,
  ease: "power1.inOut"
});
```
</motionpath_basic>

<motionpath_options>
```javascript
gsap.to("#element", {
  motionPath: {
    path: "#path",           // path to follow
    align: "#path",          // align element to path
    alignOrigin: [0.5, 0.5], // center of element
    autoRotate: true,        // rotate to follow path direction
    autoRotate: 90,          // offset rotation by 90 degrees
    start: 0,                // start position (0-1)
    end: 1,                  // end position (0-1)
    offsetX: 10,             // offset from path
    offsetY: 10
  },
  duration: 3
});
```
</motionpath_options>

<motionpath_patterns>
**Looping path animation:**
```javascript
gsap.to("#car", {
  motionPath: {
    path: "#track",
    align: "#track",
    autoRotate: true,
    alignOrigin: [0.5, 0.5]
  },
  duration: 10,
  repeat: -1,
  ease: "none"
});
```

**Scroll-driven path:**
```javascript
gsap.to("#icon", {
  motionPath: {
    path: "#scrollPath",
    align: "#scrollPath",
    alignOrigin: [0.5, 0.5]
  },
  scrollTrigger: {
    trigger: ".section",
    start: "top center",
    end: "bottom center",
    scrub: 1
  }
});
```
</motionpath_patterns>
</motionpath>

<transform_origin>
SVG transform origin behaves differently:

```javascript
// Set transform origin for SVG elements
gsap.to("#svgElement", {
  rotation: 360,
  transformOrigin: "center center", // or "50% 50%"
  svgOrigin: "200 150"  // specific SVG coordinate point
});
```
</transform_origin>

<performance_tips>
- Use `will-change: transform` sparingly on SVG
- Avoid animating complex SVG filters
- Simplify paths when possible (fewer points = better performance)
- Use `force3D: false` for simple SVG animations
</performance_tips>

<anti_patterns>
**DON'T:**
- Try to morph between paths with vastly different complexity
- Forget stroke on DrawSVG elements
- Animate very complex SVG filters
- Use huge SVGs with many paths

**DO:**
- Convert shapes to paths before morphing
- Simplify paths where possible
- Use transform origin appropriate to your animation
- Test performance with complex SVGs
</anti_patterns>
