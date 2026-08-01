*Version: v1.0 (2026-08-01)*

> **로드 시점**: 사용자가 움직임·활성 경로·traveling signal·animated flow를 명시했을 때만 로드함. 정적 다이어그램에는 로드하지 않음.

# Flow Motion Primitive

Optional primitive for showing one meaningful active route while preserving a complete static connector underneath. Contract is renderer-neutral: downstream HTML, React, SVG, and slide consumers may implement it with their own adapters.

## Default contract

- Base connector is static `rounded-orthogonal`. Same-axis relationships may use a straight segment.
- Base connector remains complete content: route, arrow marker, semantic weight, and short label stay readable without motion.
- Add one CSS-only normalized overlay only when motion is explicitly requested. Use the same path geometry and `pathLength="100"`; do not replace base connector with overlay.
- Allow at most one active motion edge per diagram. Keep semantic line rules at three or fewer. Use one connector path shape per diagram.
- No JavaScript is required. D3 is a downstream data/layout adapter only when changing data or layout genuinely requires it; fixed diagrams must not add D3.

## SVG shape

Use one SVG namespace and define marker/overlay IDs inside that SVG. Every instance needs a unique ID prefix. Include an accessible name and description.

```html
<svg viewBox="0 0 320 120" role="img" aria-labelledby="flow-title-01 flow-desc-01">
  <title id="flow-title-01">Active route</title>
  <desc id="flow-desc-01">A static connector with one optional moving signal.</desc>
  <defs>
    <marker id="flow-arrow-01" markerWidth="8" markerHeight="6" refX="7" refY="3" orient="auto">
      <path d="M0 0L8 3L0 6Z" fill="currentColor" />
    </marker>
  </defs>
  <path id="flow-base-01" class="flow-base" d="M24 60H144Q152 60 152 68V92H296"
        pathLength="100" marker-end="url(#flow-arrow-01)" />
  <path id="flow-overlay-01" class="flow-overlay" d="M24 60H144Q152 60 152 68V92H296"
        pathLength="100" aria-hidden="true" />
</svg>
```

`flow-overlay-01` is an example ID only. Replace the prefix for every SVG instance. Keep marker and overlay definitions in the same SVG; do not rely on a document-global ID.

## CSS-only motion

```css
.flow-base {
  fill: none;
  stroke: var(--link);
  stroke-width: var(--edge-width);
  stroke-linecap: round;
  stroke-linejoin: round;
}

.flow-overlay {
  fill: none;
  stroke: var(--accent);
  stroke-width: var(--focus-width);
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-dasharray: 30 70;
  animation: flow-travel 1400ms linear infinite;
}

@keyframes flow-travel {
  to { stroke-dashoffset: -100; }
}

@media (prefers-reduced-motion: reduce) {
  .flow-overlay { animation: none; visibility: hidden; }
}
```

The normalized values above are a consumer example, not a production recipe copied from a source. Map colors and widths through the consumer's semantic tokens. Keep source observations and provenance warnings in the token catalog, then implement independently.

## Static export and PPT freeze

For static SVG, PNG, PDF, print, or PPT output, remove or hide the overlay and freeze meaning into the base connector: `arrow + focus weight + accent + short label`. Editable PPT uses native shapes/connectors when editability matters; use one static SVG when visual fidelity matters more. Never promise animation in PPT/PDF/print output.

## Accessibility and runtime

- `prefers-reduced-motion` is mandatory. Reduced motion removes or stops overlay; base connector remains.
- `<title>` and `<desc>` are mandatory for the SVG instance.
- IDs must be unique within a page and namespaced per SVG instance.
- If a downstream JS adapter is added for dynamic data/layout, pause motion while the SVG is offscreen and preserve static fallback when JS is unavailable.
- Do not stack async dash and packet dash when their meanings are unresolved. One active overlay is the hard maximum.
