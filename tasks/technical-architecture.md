# Technical Architecture: Love Letter to the Form

## 0. File Structure

Single file: `index.html` containing all HTML, CSS (`<style>`), and JS (`<script>`). No external dependencies, no build step, no frameworks.

---

## 1. Sequencing Engine

### State Model

The piece is a linear sequence of **parts**, each containing **steps**. The global state is a single object:

```js
const state = {
  part: 1,       // 1–5
  step: 0,       // current micro-step within the part
  inverted: false // color inversion state (true = dark mode)
};
```

### Step Registry

Each part registers its steps as an array of functions. A step function performs one atomic visual change and returns `void`. The registry:

```js
const PARTS = {
  1: {
    el: null, // DOM ref, set on DOMContentLoaded
    steps: [
      // step 0: show opening line + red streak
      // step 1: show "the lens" button
      // step 2: lens click — text cycles to "a border"
      // step 3: lens click — text cycles to "And borders..."
      // step 4: lens click — text cycles to "reveal themselves..."
      // step 5: invert colors (red streak → white, bg → black, text → white)
      // step 6: complete — enable arrow to Part 2
    ]
  },
  2: {
    el: null,
    steps: [
      // step 0: show slot machine with "I am not trying to hide"
      // step 1: slot spin → "I am trying to hide"
      // step 2: slot spin → "I am trying"
      // step 3: slot spin → "I am not"
      // step 4: slot spin → "I am" + color un-invert (back to white bg)
      // step 5: show "I am peeling parts" (first wall phrase)
      // step 6: show "if you were to see me" (stacked)
      // step 7: show "really see me" (stacked)
      // step 8: show "we would both blush" (stacked)
      // step 9: show "and so I build borders" (stacked)
      // step 10: show draggable rectangles
      // step 11: (drag completion detected) show curling text
      // step 12: complete — enable arrow to Part 3
    ]
  },
  3: {
    el: null,
    steps: [
      // step 0: "Take my face for instance" + open camera
      // step 1: "I am not shy" + shy animation
      // step 2: show eyes with iris tracking
      // step 3: (center-zone pause detected) explosion
      // step 4: complete — enable arrow to Part 4
    ]
  },
  4: {
    el: null,
    steps: [
      // step 0: show "Like the maiko..." with drag-to-transform
      // step 1: (drag completion) show floating text passage
      // step 2: (float completion) complete — enable arrow to Part 5
    ]
  },
  5: {
    el: null,
    steps: [
      // step 0: "And I am not writing about power"
      // step 1: "Believe me"
      // step 2: "I only write about one thing"
      // step 3: "Love" (first glitch word)
      // step 4: glitch → "Money"
      // step 5: glitch → "Fasting"
      // step 6: glitch → "The synapse"
      // step 7: "The negative space of mind"
      // step 8: "The electric reach of every pumped neuron"
      // step 9: "I write about the psychic mirror..."
      // step 10: complete — piece ends
    ]
  }
};
```

### Advance Logic

```js
function advance() {
  const part = PARTS[state.part];
  if (state.step < part.steps.length) {
    part.steps[state.step]();
    state.step++;
  }
  updateArrow();
}

function advancePart() {
  state.part++;
  state.step = 0;
  scrollToPartElement(state.part);
  advance(); // auto-trigger first step of new part
}
```

### Click vs Arrow Distinction

- **Click anywhere in the active section** advances to the next micro-step within the current part.
- **Arrow button** (inline, between sections) advances to the next part. It only appears when all steps in the current part are complete.

The click handler on the active section:

```js
function handleSectionClick(e) {
  // Ignore clicks on interactive elements (buttons, draggables, the arrow itself)
  if (e.target.closest('[data-interactive]')) return;
  advance();
}
```

### Gating

Some steps are not click-driven but completion-driven (drag finished, center-zone timer fired, float animation done). These steps register a completion callback instead of advancing on click:

```js
// For gated steps, the step function sets up the interaction
// and calls advanceWhenReady() when the interaction completes
function advanceWhenReady() {
  state.step++;
  updateArrow();
}
```

### Arrow Button

The arrow is an inline `<button>` element placed between part sections in the DOM. Only the arrow for the current part-boundary is visible:

```html
<section id="part-1" data-part="1">
  <!-- Part 1 content -->
</section>
<button class="advance-arrow" data-advances-to="2" aria-label="Continue to next section" hidden>&#8594;</button>
<section id="part-2" data-part="2" hidden>
  <!-- Part 2 content -->
</section>
<button class="advance-arrow" data-advances-to="3" aria-label="Continue to next section" hidden>&#8594;</button>
<!-- ... -->
```

```css
.advance-arrow {
  display: block;
  margin: 2rem auto;
  padding: 0.75rem 1.5rem;
  font-family: 'Courier New', Courier, monospace;
  font-size: 1.5rem;
  background: none;
  border: 1px solid currentColor;
  cursor: pointer;
  color: inherit;
  transition: opacity 0.3s ease;
}
.advance-arrow:hover {
  background: #f3f3f3;
}
.advance-arrow[hidden] {
  display: none;
}
/* In inverted mode */
.inverted .advance-arrow:hover {
  background: #333;
}
```

When `updateArrow()` detects the current part is complete, it removes `hidden` from the relevant arrow. The arrow's click handler calls `advancePart()`.

---

## 2. Color Inversion System

### Mechanism

A single CSS class `.inverted` on `<body>` toggles the entire palette:

```css
body {
  background-color: #fff;
  color: #000;
  transition: background-color 0.8s ease, color 0.8s ease;
}
body.inverted {
  background-color: #000;
  color: #fff;
}
```

All elements inherit `color` via `currentColor` or explicit `inherit` where needed. Border colors use `currentColor`.

### Trigger Points

1. **Part 1, final step**: "the figure from the ground" — `document.body.classList.add('inverted')`, red streak transitions to white.
2. **Part 2, step 4**: "I am" — `document.body.classList.remove('inverted')`.

### The Red Streak Transition

The red streak is an absolutely-positioned `<div>` behind the Part 1 text:

```css
.red-streak {
  position: absolute;
  width: 60%;
  height: 4px;
  background: #c0392b;
  top: 50%;
  left: 20%;
  transition: background-color 0.8s ease;
  z-index: -1;
}
body.inverted .red-streak {
  background: #fff;
}
```

When inversion triggers, the streak transitions from red to white simultaneously with the background going black. The `transition` property handles the smooth crossfade.

### prefers-reduced-motion

```css
@media (prefers-reduced-motion: reduce) {
  body,
  .red-streak,
  .advance-arrow {
    transition: none;
  }
}
```

---

## 3. Scroll and Reveal Pattern

### DOM Structure

All five parts exist in the DOM from the start, but parts 2–5 are `hidden`:

```html
<main id="poem">
  <section id="part-1" data-part="1" aria-label="Part 1">
    <!-- content -->
  </section>
  <button class="advance-arrow" data-advances-to="2" hidden>&#8594;</button>
  <section id="part-2" data-part="2" aria-label="Part 2" hidden>
    <!-- content -->
  </section>
  <!-- ... -->
</main>
```

### Revealing

When `advancePart()` fires:

1. Remove `hidden` from the target section.
2. Smooth-scroll to the section:

```js
function scrollToPartElement(partNum) {
  const section = document.getElementById(`part-${partNum}`);
  section.hidden = false;
  // Allow one frame for layout
  requestAnimationFrame(() => {
    section.scrollIntoView({ behavior: 'smooth', block: 'start' });
  });
}
```

### Why Not IntersectionObserver

IntersectionObserver is useful for lazy-reveal on free-scroll pages, but this piece is sequenced — the user cannot scroll ahead into unrevealed content. The arrow is the gate. So we use manual `scrollIntoView` rather than scroll-driven reveals. This keeps the model simpler: state drives visibility, not scroll position.

However, we do use IntersectionObserver for one purpose: pausing expensive animations (camera feed, eye tracking) when their section scrolls out of the viewport to save resources:

```js
const visibilityObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (!entry.isIntersecting) {
      pauseSectionAnimations(entry.target.dataset.part);
    }
  });
}, { threshold: 0 });
```

### Within-Step Text Reveal

Text lines within a step appear one at a time via click. Each text element starts with `opacity: 0` and a class that transitions it in:

```css
.line {
  opacity: 0;
  transform: translateY(8px);
  transition: opacity 0.4s ease, transform 0.4s ease;
}
.line.visible {
  opacity: 1;
  transform: translateY(0);
}
@media (prefers-reduced-motion: reduce) {
  .line {
    transform: none;
    transition: none;
  }
  .line.visible {
    opacity: 1;
  }
}
```

---

## 4. Slot Machine (Part 2)

### Reel Data

The slot machine has 5 reel positions, one per word slot. Each reel has a vertical strip of word options:

```
Reel 0: "I"     → "I"     → "I"     → "I"     → "I"
Reel 1: "am"    → "am"    → "am"    → "am"    → "am"
Reel 2: "not"   → ""      → ""      → "not"   → ""
Reel 3: "trying"→ "trying"→ "trying"→ ""       → ""
Reel 4: "to"    → "to"    → ""      → ""       → ""
Reel 5: "hide"  → "hide"  → ""      → ""       → ""
```

States in sequence:
- State 0: I am not trying to hide
- State 1: I am _ trying to hide  ("not" disappears)
- State 2: I am _ trying           ("to hide" disappears)
- State 3: I am not                ("trying" returns "not", rest gone)
- State 4: I am                    (just "I am")

### HTML Structure

```html
<div class="slot-machine" data-interactive>
  <div class="reel" data-reel="0">
    <div class="reel-strip">
      <span class="reel-word">I</span>
      <!-- repeated for each state -->
    </div>
  </div>
  <div class="reel" data-reel="1">
    <div class="reel-strip">
      <span class="reel-word">am</span>
    </div>
  </div>
  <!-- reels 2-5 -->
</div>
```

### CSS Approach

Each reel is a fixed-height window with `overflow: hidden`. The inner strip translates vertically to show different words:

```css
.slot-machine {
  display: flex;
  gap: 0.5ch;
  align-items: center;
  justify-content: center;
  font-family: 'Times New Roman', Times, serif;
  font-size: 1.5rem;
  padding: 2rem 0;
}

.reel {
  height: 1.8em;
  overflow: hidden;
  position: relative;
}

.reel-strip {
  display: flex;
  flex-direction: column;
  transition: transform 0.6s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

.reel-word {
  height: 1.8em;
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 1ch;
}
```

### Spin Animation

On click, the reels that change spin briefly (a CSS animation applied, then removed):

```css
@keyframes reel-spin {
  0% { transform: translateY(0); }
  25% { transform: translateY(-100%); }
  50% { transform: translateY(-200%); }
  75% { transform: translateY(-100%); }
  100% { transform: translateY(var(--target-offset)); }
}

.reel-strip.spinning {
  animation: reel-spin 0.6s cubic-bezier(0.35, 0, 0.25, 1) forwards;
}

@media (prefers-reduced-motion: reduce) {
  .reel-strip.spinning {
    animation: none;
    /* Instant switch */
  }
}
```

### JS Logic

```js
const SLOT_STATES = [
  ['I', 'am', 'not', 'trying', 'to', 'hide'],
  ['I', 'am', '',    'trying', 'to', 'hide'],
  ['I', 'am', '',    'trying', '',   ''    ],
  ['I', 'am', 'not', '',       '',   ''    ],
  ['I', 'am', '',    '',       '',   ''    ],
];

let slotState = 0;

function advanceSlot() {
  if (slotState >= SLOT_STATES.length - 1) return;
  slotState++;
  const nextWords = SLOT_STATES[slotState];

  document.querySelectorAll('.reel').forEach((reel, i) => {
    const strip = reel.querySelector('.reel-strip');
    const currentWord = SLOT_STATES[slotState - 1][i];
    const newWord = nextWords[i];

    if (currentWord !== newWord) {
      // Create new word element, add to strip
      const newEl = document.createElement('span');
      newEl.className = 'reel-word';
      newEl.textContent = newWord;
      strip.appendChild(newEl);

      // Trigger spin
      strip.classList.add('spinning');
      strip.style.setProperty('--target-offset', `-${slotState * 1.8}em`);

      strip.addEventListener('animationend', () => {
        strip.classList.remove('spinning');
      }, { once: true });
    }
  });

  // On final state "I am" — trigger color un-inversion
  if (slotState === SLOT_STATES.length - 1) {
    document.body.classList.remove('inverted');
  }
}
```

### Empty Reel Handling

When a reel's word becomes empty, the reel visually collapses. Use `width` transition:

```css
.reel {
  transition: width 0.4s ease, opacity 0.4s ease;
}
.reel.empty {
  width: 0;
  opacity: 0;
  overflow: hidden;
}
```

---

## 5. Camera System (Part 3)

### Permission Flow

```js
async function initCamera(videoEl, canvasEl) {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: 'user', width: { ideal: 320 }, height: { ideal: 240 } }
    });
    videoEl.srcObject = stream;
    await videoEl.play();
    startRetroFilter(videoEl, canvasEl);
    return true;
  } catch (err) {
    // Permission denied or no camera
    startFallbackSurface(canvasEl);
    return false;
  }
}
```

### B&W Pixelated Retro Filter

The `<video>` element is hidden. A `<canvas>` displays the processed feed:

```html
<div class="camera-container" data-interactive>
  <video id="camera-video" autoplay playsinline muted hidden></video>
  <canvas id="camera-canvas" width="320" height="240"></canvas>
</div>
```

```css
.camera-container {
  width: min(320px, 80vw);
  aspect-ratio: 4/3;
  margin: 1rem auto;
  border: 1px solid currentColor;
}
.camera-container canvas {
  width: 100%;
  height: 100%;
  image-rendering: pixelated;
}
```

Processing pipeline:

```js
function startRetroFilter(video, canvas) {
  const ctx = canvas.getContext('2d');
  // Draw at low resolution for pixel effect
  const PIXEL_SIZE = 4; // each "pixel" is 4x4 real pixels
  const w = canvas.width / PIXEL_SIZE;
  const h = canvas.height / PIXEL_SIZE;

  // Create offscreen canvas for downsampling
  const offscreen = document.createElement('canvas');
  offscreen.width = w;
  offscreen.height = h;
  const offCtx = offscreen.getContext('2d');

  let animId;
  function render() {
    // Draw video to small canvas (automatic downsampling)
    offCtx.drawImage(video, 0, 0, w, h);
    const imageData = offCtx.getImageData(0, 0, w, h);
    const data = imageData.data;

    // Convert to B&W (luminance)
    for (let i = 0; i < data.length; i += 4) {
      const gray = data[i] * 0.299 + data[i+1] * 0.587 + data[i+2] * 0.114;
      data[i] = data[i+1] = data[i+2] = gray;
    }

    offCtx.putImageData(imageData, 0, 0);

    // Draw pixelated result to main canvas
    ctx.imageSmoothingEnabled = false;
    ctx.drawImage(offscreen, 0, 0, canvas.width, canvas.height);

    animId = requestAnimationFrame(render);
  }
  render();

  // Return cleanup function
  return () => cancelAnimationFrame(animId);
}
```

### Fallback: Glinting Surface

When camera permission is denied, the canvas shows an animated white/grey surface — a shimmering, reflective quality:

```js
function startFallbackSurface(canvas) {
  const ctx = canvas.getContext('2d');
  const w = canvas.width;
  const h = canvas.height;
  let t = 0;
  let animId;

  function render() {
    t += 0.02;
    const imageData = ctx.createImageData(w, h);
    const data = imageData.data;

    for (let y = 0; y < h; y++) {
      for (let x = 0; x < w; x++) {
        const i = (y * w + x) * 4;
        // Layered sine waves for glinting effect
        const v = 200 + 55 * Math.sin(x * 0.05 + t)
                      * Math.cos(y * 0.07 + t * 0.7)
                      * Math.sin((x + y) * 0.03 + t * 1.3);
        const gray = Math.max(180, Math.min(255, v));
        data[i] = data[i+1] = data[i+2] = gray;
        data[i+3] = 255;
      }
    }

    ctx.putImageData(imageData, 0, 0);
    animId = requestAnimationFrame(render);
  }
  render();

  return () => cancelAnimationFrame(animId);
}
```

For `prefers-reduced-motion`, the fallback renders a single static frame (no animation loop).

### Resource Cleanup

When the user advances past Part 3, stop the camera stream and cancel the animation frame:

```js
function stopCamera() {
  const video = document.getElementById('camera-video');
  if (video.srcObject) {
    video.srcObject.getTracks().forEach(track => track.stop());
    video.srcObject = null;
  }
}
```

---

## 6. Drag System (Parts 2 & 4)

### Pointer Events Architecture

All drag uses Pointer Events for unified mouse/touch/pen support:

```js
function makeDraggable(el, options = {}) {
  let isDragging = false;
  let startX, startY, origX, origY;

  el.style.touchAction = 'none'; // Prevent scroll during drag
  el.setAttribute('role', 'application');
  el.setAttribute('aria-roledescription', 'draggable');
  el.setAttribute('tabindex', '0');

  el.addEventListener('pointerdown', (e) => {
    isDragging = true;
    el.setPointerCapture(e.pointerId);
    startX = e.clientX;
    startY = e.clientY;
    const rect = el.getBoundingClientRect();
    origX = rect.left;
    origY = rect.top;
    el.classList.add('dragging');
  });

  el.addEventListener('pointermove', (e) => {
    if (!isDragging) return;
    e.preventDefault();
    const dx = e.clientX - startX;
    const dy = e.clientY - startY;
    el.style.transform = `translate(${dx}px, ${dy}px)`;
    if (options.onMove) options.onMove(dx, dy, e);
  });

  el.addEventListener('pointerup', (e) => {
    if (!isDragging) return;
    isDragging = false;
    el.classList.remove('dragging');
    if (options.onDrop) options.onDrop(e);
  });

  el.addEventListener('pointercancel', (e) => {
    isDragging = false;
    el.classList.remove('dragging');
  });

  // Keyboard fallback: arrow keys move element
  el.addEventListener('keydown', (e) => {
    const STEP = 20;
    const current = new DOMMatrix(getComputedStyle(el).transform);
    let tx = current.m41, ty = current.m42;
    switch (e.key) {
      case 'ArrowLeft':  tx -= STEP; break;
      case 'ArrowRight': tx += STEP; break;
      case 'ArrowUp':    ty -= STEP; break;
      case 'ArrowDown':  ty += STEP; break;
      default: return;
    }
    e.preventDefault();
    el.style.transform = `translate(${tx}px, ${ty}px)`;
    if (options.onMove) options.onMove(tx, ty, e);
  });
}
```

### Part 2: Draggable Rectangles

The phrases are stacked absolutely, overlapping. The top one ("the interface") is draggable. When dragged away from center, the next one below becomes active:

```html
<div class="drag-stack" data-interactive>
  <div class="drag-rect" data-index="0">the interface</div>
  <div class="drag-rect" data-index="1">a drag</div>
  <div class="drag-rect" data-index="2">sequin bodysuit</div>
  <div class="drag-rect" data-index="3">drag</div>
  <div class="drag-rect" data-index="4">of course both</div>
  <div class="drag-rect" data-index="5">gesture and</div>
  <div class="drag-rect" data-index="6">posture</div>
</div>
```

```css
.drag-rect {
  position: absolute;
  padding: 0.75rem 1.25rem;
  border: 1px solid currentColor;
  background: inherit;
  font-family: 'Times New Roman', Times, serif;
  cursor: grab;
  user-select: none;
  touch-action: none;
}
.drag-rect.dragging {
  cursor: grabbing;
  z-index: 10;
}
```

Detection for "dragged away": when the drag rect's center is more than 150px from its origin, consider it removed. Then make the next rect in the stack draggable:

```js
function initDragStack() {
  const rects = document.querySelectorAll('.drag-rect');
  let currentIndex = 0;

  function activateRect(index) {
    if (index >= rects.length) {
      advanceWhenReady(); // All dragged away
      return;
    }
    makeDraggable(rects[index], {
      onDrop: (e) => {
        const rect = rects[index].getBoundingClientRect();
        const parent = rects[index].parentElement.getBoundingClientRect();
        const dist = Math.hypot(
          rect.left - parent.left,
          rect.top - parent.top
        );
        if (dist > 150) {
          rects[index].style.opacity = '0.3';
          rects[index].style.pointerEvents = 'none';
          currentIndex++;
          activateRect(currentIndex);
        }
      }
    });
  }

  activateRect(0);
}
```

### Part 4: Word-to-Word Transformation Drag

"Like the maiko, my economy of feelings" sits on the left. An oblique line divider. On the right: "is a management of asymmetry" with words initially hidden.

As the user drags individual words from left to right across the oblique line, each source word transforms (text replacement + position animation) into the corresponding target word:

```html
<div class="drag-transform" data-interactive>
  <div class="source-words">
    <span class="drag-word" data-index="0" draggable="false">Like</span>
    <span class="drag-word" data-index="1" draggable="false">the</span>
    <span class="drag-word" data-index="2" draggable="false">maiko,</span>
    <span class="drag-word" data-index="3" draggable="false">my</span>
    <span class="drag-word" data-index="4" draggable="false">economy</span>
    <span class="drag-word" data-index="5" draggable="false">of</span>
    <span class="drag-word" data-index="6" draggable="false">feelings</span>
  </div>
  <div class="oblique-line" aria-hidden="true"></div>
  <div class="target-words">
    <span class="target-word" data-index="0">is</span>
    <span class="target-word" data-index="1">a</span>
    <span class="target-word" data-index="2">management</span>
    <span class="target-word" data-index="3">of</span>
    <span class="target-word" data-index="4">asymmetry</span>
  </div>
</div>
```

```css
.oblique-line {
  width: 2px;
  height: 100%;
  background: currentColor;
  transform: rotate(15deg);
  margin: 0 2rem;
}
.target-word {
  opacity: 0;
  transition: opacity 0.4s ease;
}
.target-word.revealed {
  opacity: 1;
}
```

When a source word is dragged past the oblique line's X position, it fades out and the next target word fades in. The mapping is sequential — the first word dragged reveals target word 0, the second reveals target word 1, etc.

---

## 7. Eye Tracking (Part 3)

### HTML Structure

```html
<div class="eyes-container" data-interactive>
  <div class="eye" aria-hidden="true">
    <div class="iris"></div>
    <div class="pupil"></div>
  </div>
  <div class="eye" aria-hidden="true">
    <div class="iris"></div>
    <div class="pupil"></div>
  </div>
</div>
```

### CSS

```css
.eyes-container {
  display: flex;
  justify-content: center;
  gap: 3rem;
  padding: 2rem 0;
}
.eye {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  border: 2px solid currentColor;
  position: relative;
  overflow: hidden;
  background: #fff;
}
.iris {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #666;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  transition: transform 0.1s ease-out;
}
.pupil {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #000;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  transition: transform 0.1s ease-out;
}
```

### Iris Following

```js
function initEyeTracking() {
  const eyes = document.querySelectorAll('.eye');
  const MAX_OFFSET = 15; // max px iris can move from center

  function updateEyes(clientX, clientY) {
    eyes.forEach(eye => {
      const rect = eye.getBoundingClientRect();
      const centerX = rect.left + rect.width / 2;
      const centerY = rect.top + rect.height / 2;

      const angle = Math.atan2(clientY - centerY, clientX - centerX);
      const distance = Math.min(
        Math.hypot(clientX - centerX, clientY - centerY) / 10,
        MAX_OFFSET
      );

      const iris = eye.querySelector('.iris');
      const pupil = eye.querySelector('.pupil');
      const dx = Math.cos(angle) * distance;
      const dy = Math.sin(angle) * distance;

      iris.style.transform = `translate(calc(-50% + ${dx}px), calc(-50% + ${dy}px))`;
      pupil.style.transform = `translate(calc(-50% + ${dx * 1.2}px), calc(-50% + ${dy * 1.2}px))`;
    });
  }

  // Works for both mouse and touch
  document.addEventListener('pointermove', (e) => {
    updateEyes(e.clientX, e.clientY);
    checkCenterZone(e.clientX, e.clientY);
  });
}
```

### 3-Second Center Zone Pause Detection

The "center zone" is a circle around the midpoint between the two eyes:

```js
function initCenterZoneDetection() {
  let pauseTimer = null;
  const PAUSE_DURATION = 3000;
  const CENTER_RADIUS = 40; // px

  function getCenterZone() {
    const eyes = document.querySelectorAll('.eye');
    const rect0 = eyes[0].getBoundingClientRect();
    const rect1 = eyes[1].getBoundingClientRect();
    return {
      x: (rect0.left + rect0.width / 2 + rect1.left + rect1.width / 2) / 2,
      y: (rect0.top + rect0.height / 2 + rect1.top + rect1.height / 2) / 2
    };
  }

  window.checkCenterZone = function(clientX, clientY) {
    const center = getCenterZone();
    const dist = Math.hypot(clientX - center.x, clientY - center.y);

    if (dist <= CENTER_RADIUS) {
      if (!pauseTimer) {
        pauseTimer = setTimeout(() => {
          triggerExplosion();
        }, PAUSE_DURATION);
      }
    } else {
      clearTimeout(pauseTimer);
      pauseTimer = null;
    }
  };
}
```

### Explosion Animation

When the 3-second threshold is met, all words and the eye elements scatter outward:

```js
function triggerExplosion() {
  const container = document.getElementById('part-3');
  const elements = container.querySelectorAll('.line.visible, .eye, .camera-container');

  elements.forEach(el => {
    // Calculate random direction from center
    const rect = el.getBoundingClientRect();
    const centerX = window.innerWidth / 2;
    const centerY = window.innerHeight / 2;
    const elX = rect.left + rect.width / 2;
    const elY = rect.top + rect.height / 2;

    const angle = Math.atan2(elY - centerY, elX - centerX);
    const distance = 800 + Math.random() * 400;
    const tx = Math.cos(angle) * distance;
    const ty = Math.sin(angle) * distance;
    const rotation = (Math.random() - 0.5) * 720;

    el.style.transition = 'transform 1s cubic-bezier(0.25, 0, 0.5, 1), opacity 1s ease';
    el.style.transform = `translate(${tx}px, ${ty}px) rotate(${rotation}deg)`;
    el.style.opacity = '0';
  });

  // After animation completes, advance
  setTimeout(() => {
    advanceWhenReady();
  }, 1200);
}
```

For `prefers-reduced-motion`: skip the explosion animation, just fade out:

```js
if (prefersReducedMotion()) {
  elements.forEach(el => {
    el.style.transition = 'opacity 0.3s ease';
    el.style.opacity = '0';
  });
}
```

---

## 8. Text Effects

### 8a. Curling Text (Part 2)

Text deforms based on cursor proximity. Each character is wrapped in a `<span>` and displaced based on distance from the pointer:

```js
function initCurlingText(container) {
  const text = container.textContent;
  container.innerHTML = '';
  container.setAttribute('aria-label', text); // Accessibility: screen readers get plain text

  // Wrap each character
  [...text].forEach(char => {
    const span = document.createElement('span');
    span.textContent = char;
    span.classList.add('curl-char');
    span.style.display = 'inline-block';
    span.style.transition = 'transform 0.15s ease-out';
    container.appendChild(span);
  });

  function handleMove(e) {
    const chars = container.querySelectorAll('.curl-char');
    chars.forEach(span => {
      const rect = span.getBoundingClientRect();
      const charX = rect.left + rect.width / 2;
      const charY = rect.top + rect.height / 2;
      const dist = Math.hypot(e.clientX - charX, e.clientY - charY);
      const maxDist = 150;

      if (dist < maxDist) {
        const force = (1 - dist / maxDist) * 20;
        const angle = Math.atan2(charY - e.clientY, charX - e.clientX);
        const dx = Math.cos(angle) * force;
        const dy = Math.sin(angle) * force;
        span.style.transform = `translate(${dx}px, ${dy}px)`;
      } else {
        span.style.transform = 'translate(0, 0)';
      }
    });
  }

  container.addEventListener('pointermove', handleMove);

  // On mobile, handle touchmove on the document when touch is near
  document.addEventListener('pointermove', (e) => {
    const rect = container.getBoundingClientRect();
    const buffer = 150;
    if (e.clientX > rect.left - buffer && e.clientX < rect.right + buffer &&
        e.clientY > rect.top - buffer && e.clientY < rect.bottom + buffer) {
      handleMove(e);
    }
  });
}
```

### prefers-reduced-motion

Skip the curling entirely. The text just sits still. The click-to-advance still works.

### 8b. Floating/Drifting Text (Part 4)

Words appear one by one, then drift off-screen:

```js
function initFloatingText(words, container) {
  let index = 0;
  const interval = 600; // ms between words

  function showNextWord() {
    if (index >= words.length) {
      // All words shown and drifted — advance
      setTimeout(() => advanceWhenReady(), 1000);
      return;
    }

    const span = document.createElement('span');
    span.className = 'float-word';
    span.textContent = words[index];
    container.appendChild(span);

    // Random drift direction
    const angle = Math.random() * Math.PI * 2;
    const distance = window.innerHeight;
    const dx = Math.cos(angle) * distance;
    const dy = Math.sin(angle) * distance;
    const duration = 3000 + Math.random() * 2000;

    // Force reflow before animation
    span.offsetHeight;
    span.style.transition = `transform ${duration}ms linear, opacity ${duration}ms ease-in`;
    span.style.transform = `translate(${dx}px, ${dy}px) rotate(${(Math.random()-0.5)*90}deg)`;
    span.style.opacity = '0';

    index++;
    setTimeout(showNextWord, interval);
  }

  showNextWord();
}
```

```css
.float-word {
  display: inline-block;
  position: relative;
  margin: 0 0.3em;
  opacity: 1;
}
```

For `prefers-reduced-motion`: words appear and simply fade out in place (no movement).

### 8c. Glitch/Scramble Effect (Part 5)

The "Love / Money / Fasting / The synapse" transition. When the user clicks, the current word glitches — rapid random character cycling — then resolves to the next word:

```js
function glitchTransition(el, targetText, callback) {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%&*';
  const duration = 600; // total glitch time
  const frameInterval = 40; // ms between character changes
  const frames = duration / frameInterval;
  let frame = 0;

  const maxLen = Math.max(el.textContent.length, targetText.length);

  const interval = setInterval(() => {
    frame++;
    let result = '';
    for (let i = 0; i < maxLen; i++) {
      // Gradually settle characters from left to right
      const settleFrame = (i / maxLen) * frames;
      if (frame > settleFrame + frames * 0.6) {
        result += targetText[i] || '';
      } else {
        result += chars[Math.floor(Math.random() * chars.length)];
      }
    }
    el.textContent = result;

    if (frame >= frames) {
      clearInterval(interval);
      el.textContent = targetText;
      if (callback) callback();
    }
  }, frameInterval);
}
```

For `prefers-reduced-motion`: skip the scramble, just swap the text instantly.

### 8d. "Shy" Animation (Part 3)

"Shy" undergoes three sequential transformations: bold, jump, grow.

```css
.shy-word {
  display: inline-block;
  font-weight: normal;
  transition: font-weight 0.2s, transform 0.3s, font-size 0.4s;
}

@keyframes shy-jump {
  0%   { transform: translateY(0); }
  30%  { transform: translateY(-30px); }
  50%  { transform: translateY(-30px); }
  100% { transform: translateY(0); }
}

.shy-word.bold {
  font-weight: bold;
}

.shy-word.jump {
  animation: shy-jump 0.5s ease-in-out;
}

.shy-word.grow {
  font-size: 4rem;
  transition: font-size 0.4s cubic-bezier(0.17, 0.67, 0.2, 1.2);
}
```

```js
function animateShy(el) {
  // Step 1: bold
  el.classList.add('bold');

  setTimeout(() => {
    // Step 2: jump
    el.classList.add('jump');

    setTimeout(() => {
      // Step 3: grow
      el.classList.add('grow');
    }, 500);
  }, 300);
}
```

---

## 9. Footnote System

### HTML Structure

Each part ends with a footnote marker:

```html
<section id="part-1" data-part="1">
  <!-- part content -->
  <span class="footnote-anchor" data-footnote="1" aria-label="Footnote">
    <span class="last-word">ground.</span><sup class="footnote-marker" role="button" tabindex="0" aria-expanded="false">*</sup>
  </span>
  <div class="footnote-box" id="footnote-1" role="tooltip" hidden>
    <p>Each border has a flavor, a taste. The aesthetics of semantics...</p>
  </div>
</section>
```

### CSS

```css
.footnote-marker {
  cursor: pointer;
  font-family: 'Courier New', Courier, monospace;
  font-size: 0.75rem;
  vertical-align: super;
  margin-left: 2px;
}

.footnote-box {
  position: absolute;
  background: #fff;
  color: #000;
  border: 1px solid #000;
  padding: 1rem;
  max-width: min(400px, 80vw);
  font-family: 'Courier New', Courier, monospace;
  font-size: 0.85rem;
  line-height: 1.5;
  z-index: 100;
}

.footnote-box[hidden] {
  display: none;
}
```

### Positioning

The footnote box is positioned relative to the marker. On desktop, it floats beside the last word. On mobile (narrow viewports), it appears below:

```js
function positionFootnote(marker, box) {
  const markerRect = marker.getBoundingClientRect();
  const viewportWidth = window.innerWidth;

  box.hidden = false;
  box.style.position = 'absolute';

  if (viewportWidth > 600) {
    // Desktop: position to the right of the marker
    box.style.left = `${marker.offsetLeft + marker.offsetWidth + 10}px`;
    box.style.top = `${marker.offsetTop - 10}px`;
  } else {
    // Mobile: position below
    box.style.left = '1rem';
    box.style.right = '1rem';
    box.style.width = 'auto';
    box.style.top = `${marker.offsetTop + marker.offsetHeight + 10}px`;
  }
}
```

### Open/Close Logic

```js
function initFootnotes() {
  document.querySelectorAll('.footnote-marker').forEach(marker => {
    const footnoteId = marker.closest('[data-footnote]').dataset.footnote;
    const box = document.getElementById(`footnote-${footnoteId}`);

    marker.addEventListener('click', (e) => {
      e.stopPropagation();
      const isOpen = !box.hidden;
      closeAllFootnotes();
      if (!isOpen) {
        positionFootnote(marker, box);
        marker.setAttribute('aria-expanded', 'true');
      }
    });

    // Keyboard: Enter/Space
    marker.addEventListener('keydown', (e) => {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        marker.click();
      }
    });
  });

  // Close on click outside
  document.addEventListener('click', (e) => {
    if (!e.target.closest('.footnote-box') && !e.target.closest('.footnote-marker')) {
      closeAllFootnotes();
    }
  });
}

function closeAllFootnotes() {
  document.querySelectorAll('.footnote-box').forEach(box => box.hidden = true);
  document.querySelectorAll('.footnote-marker').forEach(m => m.setAttribute('aria-expanded', 'false'));
}
```

---

## 10. Part 1: The Lens Mechanism

### HTML

```html
<div class="lens-interaction" data-interactive>
  <span class="lens-text" id="lens-target">is a locked bedroom door</span>
  <button class="lens-button" aria-label="Toggle lens">the lens</button>
  <div class="lens-circle" hidden aria-hidden="true"></div>
</div>
```

### The Lens Circle

An optometrist's trial lens — a black circle ring:

```css
.lens-circle {
  position: absolute;
  width: 120px;
  height: 120px;
  border-radius: 50%;
  border: 3px solid #000;
  background: transparent;
  pointer-events: none;
  /* Inner ring for optometrist feel */
  box-shadow: inset 0 0 0 2px #000,
              inset 0 0 0 4px transparent,
              inset 0 0 0 5px #000;
  transform: translate(-50%, -50%);
}
```

### Cycling Text

The lens button cycles through text states. The lens circle is positioned over the text:

```js
const LENS_STATES = [
  'is a locked bedroom door',
  'a border',
  'And borders, as you get to know them,',
  'reveal themselves as more than enclosures'
];
let lensState = 0;

function handleLensClick() {
  const textEl = document.getElementById('lens-target');
  const lens = document.querySelector('.lens-circle');

  if (lensState === 0) {
    // First click: show the lens
    lens.hidden = false;
    positionLensOverText(lens, textEl);
  }

  lensState++;

  if (lensState < LENS_STATES.length) {
    textEl.textContent = LENS_STATES[lensState];
    positionLensOverText(lens, textEl);
  } else {
    // Final click: lens disappears, trigger color inversion
    lens.hidden = true;
    // advance to next step (inversion)
    advance();
  }
}

function positionLensOverText(lens, textEl) {
  const rect = textEl.getBoundingClientRect();
  const parent = textEl.closest('.lens-interaction').getBoundingClientRect();
  lens.style.left = `${rect.left - parent.left + rect.width / 2}px`;
  lens.style.top = `${rect.top - parent.top + rect.height / 2}px`;
}
```

---

## 11. Part 5: HTML Element Poetry

Each line uses a contextually relevant HTML form element. The poem spec says "figure out a cool contextually relevant way to display / animate the words."

### Design Choices

| Line | Element | Rationale |
|------|---------|-----------|
| "And I am not writing about power." | `<meter>` element (or custom progress bar) filled to 0 — "not about power" = empty power meter | Power as measurable, but empty |
| "Believe me," | `<details><summary>` — click to expand, trust as disclosure | The ask to believe = revealing hidden content |
| "I only write about one thing" | `<select>` dropdown with single option | "One thing" = single option in a multi-option container |
| Love / Money / Fasting / The synapse | Glitch text (see 8c above) | Rapid transformation between concepts |
| "The negative space of mind." | Text revealed by `<input type="range">` slider — slider reveals/hides text | Sliding between presence and absence |
| "The electric reach of every pumped neuron." | Characters appear on keypress in a readonly `<input>` that auto-types | Electric / neural firing = keystrokes |
| "I write about the psychic mirror..." | `<textarea>` that mirrors/reverses what appears in it | Psychic mirror = reflected text |

### Example: The Meter

```html
<div class="p5-line" data-interactive>
  <label for="power-meter" class="p5-label">And I am not writing about power.</label>
  <meter id="power-meter" min="0" max="100" value="0" style="width: 100%"></meter>
</div>
```

### Example: The Select Dropdown

```html
<div class="p5-line" data-interactive>
  <label for="one-thing" class="p5-label">I only write about one thing</label>
  <select id="one-thing">
    <option>one thing</option>
  </select>
</div>
```

All elements use the reference styling:

```css
.p5-line select,
.p5-line input,
.p5-line textarea,
.p5-line meter {
  font-family: 'Courier New', Courier, monospace;
  border: 1px solid currentColor;
  background: none;
  color: inherit;
  padding: 0.25rem 0.5rem;
}
```

---

## 12. Performance and Accessibility

### Semantic HTML

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Love Letter to the Form — an interactive web poem by Halim Madi">
  <title>Love Letter to the Form</title>
</head>
<body>
  <main id="poem" role="main">
    <h1 class="sr-only">Love Letter to the Form</h1>
    <section id="part-1" aria-label="Part 1: Autobiography of Red">
      <!-- ... -->
    </section>
    <!-- ... -->
  </main>
</body>
```

### Screen Reader Only Class

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

### ARIA Patterns

- Arrow button: `aria-label="Continue to next section"`
- Footnote markers: `role="button"`, `tabindex="0"`, `aria-expanded="true/false"`
- Draggable elements: `role="application"`, `aria-roledescription="draggable"`
- Camera canvas: `aria-label="Camera feed with retro filter"` or `aria-label="Animated reflective surface"` (fallback)
- Interactive sections: `aria-live="polite"` on containers where text appears dynamically
- Eyes: `aria-hidden="true"` (decorative)

### Keyboard Support

- **Arrow button**: focusable, activates on Enter/Space
- **Footnote markers**: focusable, activates on Enter/Space
- **Draggable elements**: Arrow keys move them (see drag system section)
- **Slot machine**: Space/Enter advances state
- **Lens button**: standard `<button>`, keyboard-accessible by default
- **Part 5 form elements**: natively keyboard-accessible

### prefers-reduced-motion

Master utility:

```js
function prefersReducedMotion() {
  return window.matchMedia('(prefers-reduced-motion: reduce)').matches;
}
```

Used to:
- Disable CSS transitions/animations via the media query
- Skip JS-driven animations (curling, floating, explosion, glitch)
- Use instant state changes instead
- Still show a static frame of camera fallback surface

### Performance Targets

- **No external resources**: everything inline, no network requests except `getUserMedia`
- **Lazy initialization**: only initialize a part's interactions when it becomes visible
- **Animation cleanup**: cancel `requestAnimationFrame` loops and stop camera when part scrolls out
- **Efficient DOM**: character-wrapping for curling text is the most DOM-heavy operation — limited to one short sentence
- **No layout thrashing**: batch reads/writes in animation loops, use `transform` and `opacity` for animations (compositor-only properties)
- **Canvas resolution**: camera canvas is 320x240, processed at 80x60 — minimal GPU load

### Lighthouse 90+ Strategy

- Semantic heading hierarchy (h1 sr-only, sections with aria-labels)
- All interactive elements have accessible names
- Color contrast: black on white = 21:1 (passes AAA)
- Focus indicators: browser default (or custom `outline: 2px solid` on `:focus-visible`)
- No auto-playing animations without `prefers-reduced-motion` check
- `lang="en"` on `<html>`
- Viewport meta tag present
- No render-blocking resources (everything inline)

---

## 13. Edge Cases and Guards

### Camera

- **Permission denied**: fallback surface, no error thrown to user
- **No camera API** (older browsers): fallback surface
- **User revokes permission mid-stream**: `track.onended` listener triggers fallback
- **Multiple cameras**: `facingMode: 'user'` prefers front-facing

### Drag

- **Multi-touch**: `setPointerCapture` locks to a single pointer, preventing multi-touch conflicts
- **Drag outside viewport**: `pointerup` fires on captured element regardless; `pointercancel` handles edge cases
- **Small screens**: drag threshold reduced on `window.innerWidth < 500`

### Scroll

- **Fast clicking**: debounce `advance()` to prevent double-firing during transitions (300ms cooldown)
- **Browser auto-restore**: `history.scrollRestoration = 'manual'` to prevent scroll position restoration on reload

### Color Inversion

- **Elements with hardcoded colors**: the footnote box always uses `background: #fff; color: #000; border-color: #000` regardless of inversion state — it is an overlay that needs to be readable in both modes

### Text Overflow

- **Long footnote text on mobile**: `max-width: min(400px, 80vw)` and `overflow-wrap: break-word`
- **Curling text displacement**: characters are clamped to not overflow the section bounds

### Animation Frame Cleanup

```js
// Track all active animation frame IDs for cleanup
const activeAnimations = new Set();

function trackAnimation(id) {
  activeAnimations.add(id);
  return id;
}

function cleanupAnimations() {
  activeAnimations.forEach(id => cancelAnimationFrame(id));
  activeAnimations.clear();
}
```

---

## 14. CSS Reset and Base Styles Summary

```css
*, *::before, *::after {
  box-sizing: border-box;
}

html {
  scroll-behavior: smooth;
}

@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
}

body {
  font-family: 'Times New Roman', Times, serif;
  margin: 0;
  padding: 1.5rem;
  padding-bottom: 50vh;
  background-color: #fff;
  color: #000;
  line-height: 1.6;
  transition: background-color 0.8s ease, color 0.8s ease;
  overflow-x: hidden;
}

body.inverted {
  background-color: #000;
  color: #fff;
}

section {
  max-width: 640px;
  margin: 0 auto;
  padding: 2rem 0;
  position: relative;
}

section[hidden] {
  display: none;
}

button, select, input, textarea {
  font-family: 'Courier New', Courier, monospace;
  font-size: inherit;
  color: inherit;
}

@media (max-width: 480px) {
  body {
    padding: 1rem;
  }
  section {
    padding: 1rem 0;
  }
}
```

---

## 15. Initialization Sequence

```js
document.addEventListener('DOMContentLoaded', () => {
  // Set scroll restoration
  if ('scrollRestoration' in history) {
    history.scrollRestoration = 'manual';
  }

  // Cache DOM references
  Object.keys(PARTS).forEach(key => {
    PARTS[key].el = document.getElementById(`part-${key}`);
  });

  // Initialize footnotes (always available)
  initFootnotes();

  // Initialize Part 1 (visible on load)
  advance(); // triggers step 0

  // Set up arrow buttons
  document.querySelectorAll('.advance-arrow').forEach(arrow => {
    arrow.addEventListener('click', () => {
      advancePart();
    });
  });

  // Set up section click handlers
  document.querySelectorAll('section[data-part]').forEach(section => {
    section.addEventListener('click', handleSectionClick);
  });

  // Debounce guard
  let advancing = false;
  const originalAdvance = advance;
  advance = function() {
    if (advancing) return;
    advancing = true;
    originalAdvance();
    setTimeout(() => { advancing = false; }, 300);
  };
});
```
