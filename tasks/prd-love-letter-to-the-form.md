[PRD]
# PRD: Love Letter to the Form

## 1. Introduction / Overview

"Love Letter to the Form" is an interactive web poetry piece where the reader controls pacing through a single-page, click-to-advance experience. Each section reveals text and animations sequentially; clicking a styled arrow button advances to the next interaction. The piece uses HTML elements (lenses, slot machines, draggable rectangles, camera, etc.) as poetic devices. It draws aesthetic inspiration from "We Called Us Poetry" — Times New Roman, Courier New, minimal black/white/grey palette, thin borders — and must be fully responsive with mobile support (tap = click, hover effects become tap-to-reveal).

**Problem it solves:** Delivers a literary experience that blends poetry with interface as medium, where interaction and pacing are central to meaning.

---

## 2. Goals

- Deliver a single-page, responsive interactive poetry experience
- Ensure every interaction advances via a visible, well-designed arrow button
- Match the visual style of the reference HTML (Times New Roman, Courier New, minimal, restrained)
- Support mobile: tap = click; hover effects become tap-to-reveal
- Pass HTML validation, manual Chrome + mobile viewport checks, and Lighthouse accessibility
- Integrate footnotes as cryptic markers (asterisk/similar) that reveal white rectangles with black text on click

---

## 3. Quality Gates

These commands/checks must pass for every user story:

- **HTML validation:** Run through W3C Validator (or equivalent); no validation errors
- **Console:** No JavaScript errors in browser console during normal interaction flow
- **Manual check:** Visual and functional verification in Chrome (desktop) and mobile viewport (DevTools or real device)
- **Lighthouse accessibility:** Score of 90+ on Lighthouse accessibility audit

---

## 4. User Stories

### US-001: Project Setup and Base Styling

**Description:** As a developer, I want a minimal project structure and base styling so that all subsequent interactions share a consistent aesthetic.

**Acceptance Criteria:**

- [ ] Single `index.html` with valid HTML5 structure, viewport meta, and lang="en"
- [ ] Base styles match reference: `font-family: "Times New Roman", Times, serif` for body; `Courier New` for interface elements
- [ ] Color palette: black text on white background by default; grey (#ccc, #f3f3f3) for accents; thin black borders (1px solid)
- [ ] Page is responsive; no horizontal overflow on mobile viewports (320px–428px)
- [ ] Body has appropriate padding/margin; overflow-y: scroll for long content

---

### US-002: Global Sequencing Engine and Arrow Button

**Description:** As a reader, I want a clear "next" control so that I advance through the piece at my own pace.

**Acceptance Criteria:**

- [ ] A styled arrow button (→ or similar) is visible and consistently positioned (e.g. bottom-right or fixed)
- [ ] Arrow matches reference aesthetic: simple, readable, no heavy gradients
- [ ] Clicking the arrow advances to the next interaction/step in the sequence
- [ ] Each section/part is gated; the arrow only appears or becomes active when the current step is complete
- [ ] Tap on arrow works on mobile (touch events)

---

### US-003: Footnote System

**Description:** As a reader, I want to discover optional footnotes so that I can access deeper commentary without cluttering the main text.

**Acceptance Criteria:**

- [ ] Footnote markers (asterisk or similar) appear floating next to the last word of each part
- [ ] Clicking the marker reveals a white rectangle with black text, thin black border (matching reference bottom-left-box style)
- [ ] Footnote content is hidden until marker is clicked
- [ ] Footnotes can be dismissed (click outside or close control)
- [ ] Works on mobile (tap to open, tap outside to close)

---

### US-004: Part 1 — Autobiography of Red

**Description:** As a reader, I want Part 1’s interactions so that I experience the camera-as-regulator metaphor through visual feedback.

**Acceptance Criteria:**

- [ ] Opening line: "In 'Autobiography of Red', the camera acts as Geryon's emotional regulator" — a red elegant streak appears behind the text and remains
- [ ] "The lens is a locked bedroom door. A border." — a button labeled "the lens" toggles a round black lens (optometrist-style) over the text
- [ ] Lens clicks cycle text: "is a locked bedroom door" → "a border" → "And borders, as you get to know them," → "reveal themselves as more than enclosures"
- [ ] "the figure from the ground" — red streak turns white; background turns black; text turns white
- [ ] Arrow advances to Part 2 after final step
- [ ] Footnote for Part 1 is present and functional

---

### US-005: Part 2 — Slot Machine (I am not trying to hide)

**Description:** As a reader, I want the slot-machine interaction so that I experience the shifting of "not" through a visible reel mechanic.

**Acceptance Criteria:**

- [ ] Words "I am not trying to hide" are displayed in slot-machine style (visible reels, spinning aesthetic)
- [ ] Click 1: "I am not trying to hide" → "I am trying to hide" (not → blank)
- [ ] Click 2: → "I am trying"
- [ ] Click 3: → "I am not"
- [ ] Click 4: → "I am"; screen shifts from black to white, text from white to black
- [ ] Slot machine uses reference styling (serif font, minimal, no heavy decoration)
- [ ] Arrow advances to next Part 2 segment after final step

---

### US-006: Part 2 — Stacked Wall and Draggable Rectangles

**Description:** As a reader, I want the stacked wall and draggable phrases so that I experience "building borders" through layout and drag.

**Acceptance Criteria:**

- [ ] "I am peeling parts – if you were to see me – really see me – we would both blush – and so I build borders" — each phrase (separated by –) appears sequentially on click, stacked one above the other like a wall
- [ ] "the interface / a drag / sequin bodysuit / drag / of course both / gesture and / posture" — each segment (separated by /) is in an individual black rectangle with thin border
- [ ] Rectangles are draggable; "the interface" on top, others stacked behind in sequence
- [ ] "it's hard to write a straight text anymore" — text curls when mouse (or touch) is close
- [ ] Arrow advances to Part 3 after final step
- [ ] Footnote for Part 2 is present and functional

---

### US-007: Part 3 — Camera, Shy, and Following Eyes

**Description:** As a reader, I want Part 3’s camera and eye interactions so that I experience the maiko’s gaze and the risk of direct eye contact.

**Acceptance Criteria:**

- [ ] "Take my face for instance" — opens device camera so the reader sees their own face (or placeholder if permission denied)
- [ ] "I am not shy" — "Shy" becomes bold, then jumps, then grows rapidly in size, sequentially
- [ ] "Take the maiko's eyes..." — two eyes displayed; irises follow mouse/touch position
- [ ] When mouse/touch pauses in center zone for 3 seconds, a small explosion animation occurs; words/pixels explode outward and fade out
- [ ] Arrow advances to Part 4 after final step
- [ ] Footnote for Part 3 is present and functional

---

### US-008: Part 4 — Drag Words and Floating Text

**Description:** As a reader, I want Part 4’s drag and float interactions so that I experience asymmetry and tragedy through motion.

**Acceptance Criteria:**

- [ ] "Like the maiko, my economy of feelings" appears with an oblique line to the right; dragging words from left to right turns them one by one into "is a management of asymmetry"
- [ ] "Cruel not like a destiny..." passage — words appear one by one, then float away and off the screen
- [ ] Arrow advances to Part 5 after final step
- [ ] Footnote for Part 4 is present and functional

---

### US-009: Part 5 — HTML Element Poetry

**Description:** As a reader, I want Part 5’s varied HTML elements so that I experience "Love / Money / Fasting / The synapse" through interface as metaphor.

**Acceptance Criteria:**

- [ ] "And I am not writing about power" — displayed with a contextually relevant HTML element (e.g. dropdown, slider, or similar)
- [ ] "Believe me" — contextually relevant element
- [ ] "I only write about one thing" — contextually relevant element
- [ ] "Love / Money / Fasting / The synapse" — Love appears first; each click triggers a glitch and reveals next: Money → Fasting → The synapse
- [ ] "The negative space of mind" — contextually relevant element
- [ ] "The electric reach of every pumped neuron" — contextually relevant element
- [ ] "I write about the psychic mirror..." — contextually relevant element
- [ ] All elements use reference styling; no heavy decoration
- [ ] Footnote for Part 5 is present and functional

---

### US-010: Mobile Adaptations

**Description:** As a mobile reader, I want the piece to work fully on touch devices so that I can experience it without a mouse.

**Acceptance Criteria:**

- [ ] Tap anywhere on arrow = click; advances sequence
- [ ] Hover effects (e.g. text curl, iris follow) have touch equivalents: tap-to-reveal or touch-move to simulate hover
- [ ] Camera permission flow works on mobile browsers
- [ ] Draggable elements work with touch (touch events or pointer events)
- [ ] No elements require hover-only interaction without a touch alternative
- [ ] Layout remains readable and usable at 320px width

---

### US-011: Quality Gates Verification

**Description:** As a developer, I want to verify all quality gates so that the piece meets the defined standards.

**Acceptance Criteria:**

- [ ] HTML passes W3C validation (or equivalent)
- [ ] No console errors during full read-through
- [ ] Manual check in Chrome desktop and mobile viewport confirms all interactions work
- [ ] Lighthouse accessibility score ≥ 90

---

## 5. Functional Requirements

- **FR-1:** The system must display all five parts in sequence on a single page.
- **FR-2:** The system must provide a visible arrow button that advances to the next step when clicked or tapped.
- **FR-3:** The system must gate advancement so the arrow only progresses when the current step is complete.
- **FR-4:** The system must display footnote markers; clicking a marker must reveal the footnote in a white rectangle with black text and thin black border.
- **FR-5:** Part 1 must render the red streak, lens toggle, and black/white inversion as specified.
- **FR-6:** Part 2 must render the slot machine, stacked wall, draggable rectangles, and curling text as specified.
- **FR-7:** Part 3 must open the camera, animate "Shy," and render following eyes with center-zone explosion.
- **FR-8:** Part 4 must support drag-to-transform and floating words as specified.
- **FR-9:** Part 5 must use contextually relevant HTML elements (dropdown, slider, etc.) for each phrase.
- **FR-10:** The system must request camera permission and display the user’s face (or fallback) when Part 3 is active.
- **FR-11:** All interactions must have touch equivalents for mobile.
- **FR-12:** The system must use Times New Roman for body text and Courier New for interface elements.
- **FR-13:** The system must use a minimal black/white/grey palette with thin black borders.

---

## 6. Non-Goals (Out of Scope)

- Presentation vs. non-presentation mode (deferred)
- Back navigation or undo (reader moves forward only)
- Server-side logic or user accounts
- Persistence of reader progress across sessions
- Audio or video beyond the camera feed
- Frameworks (React, Vue, etc.); vanilla HTML/CSS/JS only

---

## 7. Technical Considerations

- **Stack:** Vanilla HTML, CSS, JavaScript only.
- **Camera:** `getUserMedia`; handle permission denied with a placeholder or message.
- **Mobile:** Use `pointerevents` or `touchstart`/`touchend` where hover is required.
- **Accessibility:** Semantic HTML, ARIA where needed, keyboard support for arrow where possible.
- **Style reference:** Match "We Called Us Poetry" — Times New Roman, Courier New, minimal palette, thin borders, no heavy shadows or gradients.

---

## 8. Success Metrics

- All five parts render and advance correctly on desktop and mobile.
- No HTML validation errors; no console errors.
- Lighthouse accessibility score ≥ 90.
- Reader can complete the full piece in one session with clear pacing control.

---

## 9. Open Questions

- Exact placement of arrow (fixed bottom-right vs. inline vs. other).
- Fallback content when camera permission is denied.
- Whether "I am not trying to hide" vs. "I am not hiding" in Part 2 is intentional; PRD assumes slot machine uses "I am not trying to hide" per WEB spec.
[/PRD]
