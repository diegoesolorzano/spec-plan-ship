# Eval Case 04: Life of a Particle — Interactive Canvas Visualization

**Complexity:** Medium-High
**Stack:** Single HTML file, vanilla JS, Canvas 2D API
**Why this case:** Stress-tests architectural planning for creative/visual code where multiple valid approaches exist. Output is visually verifiable — you can RUN both plans' implementations and compare. Exposes whether typed signatures help communicate rendering architecture vs leaving it ambiguous.

## Simulated Project Context

None. This is a greenfield single-file project. The plan must define ALL architecture from scratch — no existing patterns to follow. This isolates the plan's ability to communicate intent vs the ability to reference existing code.

## Spec (Input)

Feature: "Life of a Particle" — an interactive HTML visualization showing particles being born, living (affected by forces), and dying. Single self-contained HTML file, no external dependencies.

### Requirements

**Particle Lifecycle:**
- FR-1: Particles spawn at a configurable rate (particles/second) from a configurable origin point
- FR-2: Each particle has a finite lifespan (randomized within a range)
- FR-3: Particles age over their lifespan — visual properties change as they age (color, size, opacity)
- FR-4: Particles die (removed from system) when they exceed their lifespan
- FR-5: Dead particles are recycled (object pool) to avoid GC pressure

**Physics:**
- FR-6: Gravity pulls particles downward (configurable strength)
- FR-7: Wind pushes particles horizontally (configurable direction + strength)
- FR-8: Initial velocity is randomized within a cone (spread angle configurable)
- FR-9: Optional: turbulence (Perlin-like noise displacement)

**Rendering:**
- FR-10: Canvas 2D rendering at 60fps
- FR-11: Particles render as soft circles (radial gradient, not hard edges)
- FR-12: Color transitions over lifetime: birth color → mid color → death color (HSL interpolation)
- FR-13: Size changes over lifetime: small at birth → peak at 30% life → shrink to death
- FR-14: Opacity fades in first 10% of life, fades out last 30%
- FR-15: Optional: additive blending mode for glow effect

**Controls:**
- FR-16: Spawn rate slider (1-500 particles/sec)
- FR-17: Gravity slider (-2 to 2)
- FR-18: Wind slider (-3 to 3)
- FR-19: Color palette selector (at least 3 presets: fire, ocean, forest)
- FR-20: Particle count display (current alive count)
- FR-21: FPS counter

**Interaction:**
- FR-22: Click/touch on canvas changes spawn origin to click position
- FR-23: Mouse move while holding creates a "trail" of spawn points

### Acceptance Criteria

- Given the page loads, When 2 seconds pass, Then particles are visible, moving, and dying naturally
- Given gravity is set to max, When particles spawn, Then they arc downward visibly
- Given wind is set to max-right, When particles spawn, Then they drift right
- Given "fire" palette is selected, When particles age, Then they transition from yellow → orange → red → transparent
- Given user clicks canvas center, When next particles spawn, Then they originate from click point
- Given 500 particles/sec rate, When running for 5 seconds, Then FPS stays above 30 (performance requirement)
- Given the page is opened on mobile, When interacting, Then touch events work identically to mouse

### Design Constraints

- Single `index.html` file — all CSS and JS inline
- No build tools, no imports, no CDN dependencies
- Must work in modern browsers (Chrome, Firefox, Safari latest)
- Canvas must be full-viewport (no scrolling)
- Controls overlay on top of canvas (semi-transparent panel)

## Scoring Dimensions

Rate 1-5 for each:

| Dimension | What to evaluate |
|-----------|-----------------|
| **Completeness** | All 23 FRs addressed? Physics, rendering, controls, interaction all present? Object pool included? |
| **Specificity** | Is the particle data structure unambiguous? Is the render loop clear? Are color interpolation mechanics defined? |
| **Architecture** | Update/render separation? Object pool design? Event handling approach? Module organization within single file? |
| **Cross-layer coherence** | Do controls connect to simulation params correctly? Does particle state have what renderer needs? Does pool interface match lifecycle needs? |
| **Signature quality** (hybrid only) | Is Particle shape defined with all needed properties? Are lifecycle functions typed? Does the config interface cover all sliders? |
| **Implementability** | Could a different agent produce a working, performant, visually pleasing particle system from this plan alone? |

## Bonus: Visual Eval

Unlike other cases, this one produces RUNNABLE output. After generating plans and implementing them:

1. Open both implementations in a browser
2. Compare visually:
   - Do particles look alive? (smooth motion, natural feel)
   - Do controls work?
   - Is performance acceptable at 500 particles/sec?
   - Does the color lifecycle feel intentional or random?
3. Score visual quality 1-5 as a tiebreaker dimension

This is the ultimate test: does a better plan produce a better visual result?
