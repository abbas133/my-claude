---
name: ui-ux-expert
description: Expert UI/UX design and frontend development guidance. Use when designing interfaces, creating design systems, improving user experience, implementing accessible components, or building responsive layouts. Covers WCAG compliance, design patterns, and CSS/component best practices.
---

# UI/UX Expert

## Design System Tokens

```css
/* Spacing (8px grid) */
--space-1: 4px;  --space-2: 8px;  --space-4: 16px;  --space-6: 24px;

/* Typography */
--text-sm: 14px;  --text-base: 16px;  --text-lg: 18px;  --text-xl: 20px;

/* Touch targets */
--touch-min: 48px;  /* Android */
--touch-min-ios: 44px;  /* iOS */
```

## Accessibility (WCAG 2.1 AA)

| Element | Contrast Ratio |
|---------|---------------|
| Normal text | 4.5:1 |
| Large text (>=18px) | 3:1 |
| UI components | 3:1 |

**Requirements:**
- All buttons: semantic labels/tooltips
- All images: alt text or decorative marker
- Touch targets: 48x48 minimum
- Focus: visible indicators
- Motion: respect prefers-reduced-motion

## Responsive Breakpoints

```css
--bp-sm: 640px;   /* Tablets */
--bp-md: 768px;   /* Small laptops */
--bp-lg: 1024px;  /* Desktops */
--bp-xl: 1280px;  /* Large screens */
```

## Component Patterns

### Buttons
- Primary, Secondary, Ghost, Danger variants
- sm/md/lg sizes
- Loading state with spinner
- Disabled state at 50% opacity

### Forms
- Labels above inputs
- 16px font (prevents iOS zoom)
- Visible focus ring
- Inline validation errors

### Cards
- Elevated (shadow), Outlined, Filled variants
- Consistent padding (16px or 24px)
- Clear visual hierarchy

## Animation

```css
--duration-fast: 150ms;
--duration-normal: 200ms;
--ease-out: cubic-bezier(0, 0, 0.2, 1);

/* Respect user preference */
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; }
}
```

## UX Heuristics

1. System status visibility (loading states)
2. Match real world (familiar language)
3. User control (undo, cancel options)
4. Consistency (follow platform conventions)
5. Error prevention (confirmations, constraints)
6. Recognition over recall (visible options)
