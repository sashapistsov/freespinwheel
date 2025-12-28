# CLAUDE.md - Free Spin Wheel Project Documentation

## Project Overview

**Free Spin Wheel** is a self-contained, physics-based spinning wheel prize game implemented as a vanilla web component. The project emphasizes high-fidelity motion design, realistic physics simulation, and gorgeous visual presentation without requiring any frameworks.

### Core Features
- Physics-based wheel spinning with realistic motion (torque, inertia, friction, settling)
- Modular visual architecture with layered DOM elements
- Deterministic winner selection with natural-feeling animation
- Responsive design (mobile-friendly)
- Fully customizable visual elements
- Sound/event hooks for integration

## Repository Structure

```
freespinwheel/
├── index.html              # Main entry point (self-contained demo)
├── src/
│   ├── wheel.js           # Core wheel component class
│   ├── physics.js         # Physics simulation engine
│   ├── renderer.js        # Visual rendering (SVG/Canvas)
│   ├── styles.css         # Component styles
│   └── config.js          # Default configuration
├── assets/
│   ├── sounds/            # Optional tick/win sounds
│   └── images/            # Optional textures/backgrounds
├── examples/
│   └── custom-themes.html # Customization examples
├── docs/
│   └── physics-model.md   # Detailed physics explanation
├── tests/
│   └── wheel.test.html    # Manual testing page
└── CLAUDE.md              # This file
```

## Codebase Architecture

### 1. Modular Visual Layers

The wheel is composed of distinct, independently styled layers:

```
┌─────────────────────────┐
│  Background Layer       │  <- Static backdrop
├─────────────────────────┤
│  Wheel Rim Layer        │  <- Outer ring with depth/highlights
├─────────────────────────┤
│  Slices Layer           │  <- Segmented wheel with labels (ROTATES)
├─────────────────────────┤
│  Pointer/Arrow Layer    │  <- Fixed at top, does not rotate
├─────────────────────────┤
│  Button Layer           │  <- Bottom centered "Spin" button
└─────────────────────────┘
```

Each layer should be:
- A separate DOM element or SVG group
- Independently styled with CSS classes
- Easily customizable via configuration

### 2. Physics Model

The spin simulation uses **requestAnimationFrame** for smooth, physics-based motion:

#### State Variables
- `angularVelocity` - Current rotation speed (radians/frame)
- `currentAngle` - Current wheel rotation (radians)
- `targetAngle` - Final destination angle (deterministic)
- `isSpinning` - Animation state flag

#### Motion Phases

**Phase 1: Acceleration (Initial Torque)**
```javascript
// Quick ramp-up to initial speed
angularVelocity = initialTorque * accelerationFactor
```

**Phase 2: Inertial Coast**
```javascript
// Maintain high speed briefly
angularVelocity = maxVelocity
```

**Phase 3: Friction Deceleration**
```javascript
// Gradually slow down
angularVelocity *= frictionCoefficient
currentAngle += angularVelocity
```

**Phase 4: Settling**
```javascript
// When near target, smoothly ease to final position
// Optional: tiny overshoot + micro-bounce
if (closeToTarget) {
  currentAngle = lerp(currentAngle, targetAngle, settleSpeed)
}
```

#### Deterministic Winner Selection

```javascript
// 1. Choose winner FIRST
const winnerIndex = selectWinner() // or random

// 2. Calculate target angle that lands winner under pointer
const sliceAngle = (2 * Math.PI) / slices.length
const targetAngle = calculateTargetAngle(winnerIndex, sliceAngle)

// 3. Add multiple full rotations for dramatic effect
const finalAngle = targetAngle + (Math.PI * 2 * minSpins)

// 4. Run physics simulation to reach finalAngle naturally
runPhysicsSimulation(finalAngle)
```

### 3. Visual Effects & Polish

#### High-Quality Shading
- Use CSS gradients or SVG filters for depth
- Subtle highlights on rim (light source from top-left)
- Per-slice gradients (radial or linear)
- Drop shadows for layering

#### Motion Enhancements
- **Wobble**: Apply tiny sine wave to rotation during high speed
  ```javascript
  wobble = sin(time * frequency) * amplitude * (speed / maxSpeed)
  visualAngle = currentAngle + wobble
  ```

- **Pointer Tick**: Detect slice boundary crossings, trigger bounce animation
  ```javascript
  if (crossedBoundary) {
    pointerBounce = maxBounce
    onTick() // Sound hook
  }
  pointerBounce *= dampingFactor
  ```

- **Motion Blur** (optional): Increase opacity/blur of slices at high speed

#### Text Rendering
- Labels centered in each slice
- Rotated to match slice angle
- High contrast with background
- Readable font (avoid overly decorative fonts)

## Development Conventions

### Code Style

**JavaScript**
- Use ES6+ features (classes, arrow functions, destructuring)
- Clear variable names (e.g., `angularVelocity` not `av`)
- Separate concerns: physics logic != rendering logic
- Use constants for magic numbers
```javascript
const FRICTION_COEFFICIENT = 0.98
const MIN_SPINS = 3
const MAX_VELOCITY = 0.5
```

**CSS**
- Use CSS custom properties for theming
```css
:root {
  --wheel-rim-color: #d4af37;
  --slice-bg-primary: #ff6b6b;
  --slice-bg-secondary: #4ecdc4;
}
```
- Mobile-first responsive design
- Prefer `transform` and `opacity` for animations (GPU-accelerated)

**HTML**
- Semantic element names
- ARIA labels for accessibility
- Clean structure with comments for major sections

### File Organization

**Single-File Mode** (for demos/quick integration)
- `index.html` contains everything (HTML + inline CSS + inline JS)
- Should work immediately when opened in browser
- Good for CodePen, JSFiddle, or quick prototypes

**Modular Mode** (for production)
- Separate files for maintainability
- Build step optional (not required)
- ES6 modules for clean imports

### Configuration API

All visual and behavioral aspects should be configurable:

```javascript
const wheelConfig = {
  // Prize data
  slices: [
    { label: "10$", value: 10, color: "#ff6b6b" },
    { label: "20$", value: 20, color: "#4ecdc4" },
    // ...
  ],

  // Physics parameters
  physics: {
    friction: 0.98,
    minSpins: 3,
    maxSpins: 6,
    initialTorque: 0.5,
    wobbleAmount: 0.005,
    settleDuration: 500 // ms
  },

  // Visual customization
  visual: {
    wheelSize: 400, // pixels
    rimWidth: 20,
    rimColor: "#d4af37",
    pointerColor: "#ff0000",
    fontFamily: "Arial, sans-serif",
    fontSize: 18
  },

  // Hooks
  onTick: () => {}, // Fired when slice boundary passes pointer
  onSpinStart: () => {},
  onSpinEnd: (result) => {},
  onResult: (value) => {} // Fired with winning value
}
```

## Key Development Workflows

### Adding a New Feature

1. **Read existing code first** - Understand current implementation
2. **Identify affected layers** - Which visual/logic modules need changes?
3. **Make minimal changes** - Don't refactor unrelated code
4. **Test on mobile** - Ensure responsive behavior
5. **Update config API** - Add new options if needed
6. **Document in CLAUDE.md** - Update this file

### Customizing Visual Elements

**To change wheel appearance:**

1. Locate rendering code (SVG groups or Canvas draw calls)
2. Modify colors, gradients, or shapes
3. Extract hardcoded values to config object
4. Test with different slice counts (3, 6, 8, 12, etc.)

**Example: Custom rim style**
```javascript
// In renderer.js or drawRim()
function drawRim(ctx, config) {
  const gradient = ctx.createLinearGradient(0, -radius, 0, radius)
  gradient.addColorStop(0, config.visual.rimHighlight)
  gradient.addColorStop(0.5, config.visual.rimColor)
  gradient.addColorStop(1, config.visual.rimShadow)

  ctx.fillStyle = gradient
  ctx.arc(centerX, centerY, radius, 0, Math.PI * 2)
  ctx.fill()
}
```

### Debugging Physics Issues

**Common problems:**

1. **Wheel stops abruptly**: Increase friction coefficient (closer to 1.0)
2. **Wheel spins forever**: Decrease friction or add minimum velocity threshold
3. **Wrong winner**: Check angle calculations and slice boundary logic
4. **Jittery motion**: Ensure `requestAnimationFrame` is used, not `setInterval`

**Debug helpers:**
```javascript
// Add to physics loop for console output
if (DEBUG_MODE) {
  console.log(`Angle: ${currentAngle}, Velocity: ${angularVelocity}`)
}
```

### Performance Optimization

**Canvas rendering:**
- Only redraw when angle changes
- Use `ctx.save()` / `ctx.restore()` efficiently
- Pre-calculate slice angles on initialization

**SVG rendering:**
- Use CSS transforms for rotation (not re-rendering)
- Cache slice elements, don't recreate each frame
- Use `will-change: transform` for hint to browser

## Testing Checklist

Before considering a feature complete:

- [ ] Works in Chrome, Firefox, Safari
- [ ] Mobile responsive (tested on real device or devtools)
- [ ] Button disabled while spinning
- [ ] Correct winner announced
- [ ] No console errors
- [ ] Smooth 60fps animation
- [ ] Tick sound hook fires at slice boundaries (if implemented)
- [ ] Accessible (keyboard navigation, screen reader friendly)

## Common Pitfalls

### Don't:
- ❌ Use `Math.random()` for target angle (breaks determinism)
- ❌ Mix animation logic with rendering logic
- ❌ Hardcode slice count assumptions (support 3-20 slices)
- ❌ Ignore mobile viewport (wheel should scale)
- ❌ Create abstractions for single-use code
- ❌ Add comments for obvious code
- ❌ Over-engineer with excessive configuration options

### Do:
- ✅ Choose winner first, then calculate physics to reach it
- ✅ Separate physics state from visual representation
- ✅ Use constants for magic numbers
- ✅ Test with different slice configurations
- ✅ Keep code simple and readable
- ✅ Profile performance on low-end devices

## Physics Model Deep Dive

### Angular Velocity Calculation

```
ω(t) = ω₀ * f^t

Where:
  ω(t) = angular velocity at time t
  ω₀   = initial angular velocity
  f    = friction coefficient (0 < f < 1, typically 0.96-0.99)
  t    = time in frames
```

### Settling Algorithm

To prevent oscillation and ensure clean stop:

```javascript
const SETTLE_THRESHOLD = 0.01 // radians
const SETTLE_SPEED = 0.15 // lerp factor

if (Math.abs(targetAngle - currentAngle) < SETTLE_THRESHOLD) {
  currentAngle = targetAngle // Snap to exact position
  angularVelocity = 0
  isSpinning = false
} else if (angularVelocity < MIN_VELOCITY) {
  // Smoothly ease to target
  currentAngle += (targetAngle - currentAngle) * SETTLE_SPEED
}
```

### Wobble Implementation

```javascript
const wobbleFrequency = 0.3
const wobbleAmplitude = 0.005
const speedFactor = angularVelocity / MAX_VELOCITY

const wobble = Math.sin(frameCount * wobbleFrequency)
             * wobbleAmplitude
             * speedFactor

const visualAngle = currentAngle + wobble
```

## AI Assistant Guidelines

When working on this codebase:

1. **Always read files before modifying** - Don't propose changes to code you haven't seen
2. **Maintain the physics model** - Don't break deterministic winner selection
3. **Preserve layer separation** - Keep visual layers independent
4. **Test edge cases** - Try with 3, 6, 10, 20 slices
5. **Mobile-first** - Always consider responsive behavior
6. **Keep it vanilla** - No frameworks, libraries only if absolutely necessary
7. **Document complex logic** - Physics calculations deserve comments
8. **Use the config API** - Don't hardcode visual values
9. **Performance matters** - Profile on low-end devices
10. **Simplicity over cleverness** - Readable code beats clever code

### When Asked to Customize

1. Identify which layer needs modification
2. Check if config already supports it
3. If not, add config option
4. Update default values
5. Document the new option in this file

### When Fixing Bugs

1. Reproduce the issue
2. Check if it's physics or rendering
3. Use console.log or debugger strategically
4. Fix root cause, not symptoms
5. Test with multiple slice configurations

## Future Enhancements (Potential)

Ideas for future development:

- Sound effects (tick, win celebration)
- Confetti animation on win
- Multiple wheel styles (3D, neon, wooden)
- Save/load wheel configurations
- Share wheel via URL
- Analytics tracking
- Themed presets (casino, carnival, modern)
- API endpoint integration for remote prize data

## Resources

- **Physics simulation**: Classical mechanics (angular motion)
- **Canvas API**: [MDN Canvas Tutorial](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial)
- **SVG transforms**: [MDN SVG Tutorial](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial)
- **requestAnimationFrame**: [MDN RAF Guide](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)

## Contact & Contribution

This is a personal/educational project. When making changes:
- Keep commits atomic and descriptive
- Test thoroughly before committing
- Update this CLAUDE.md if architecture changes
- Follow the established code style

---

**Last Updated**: 2025-12-28
**Version**: 1.0.0
**Maintained for**: AI-assisted development with Claude
