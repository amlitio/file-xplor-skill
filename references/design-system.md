# File Xplor Design System

## Philosophy

Dark-first, minimal chrome, data-dense. The graph is the hero — everything else
supports it. No heavy UI frameworks; inline styles for zero-dependency portability.

## Color Palette

### Backgrounds
```
Page:       #0A0A0F  (near-black with blue undertone)
Alt page:   #06060B  (deeper black for landing)
Surface:    rgba(255,255,255, 0.03)   (cards, panels)
Hover:      rgba(255,255,255, 0.06)   (interactive surfaces)
Elevated:   rgba(0,0,0, 0.6–0.85)    (tooltips, overlays)
Blur:       backdropFilter: blur(8–12px)
```

### Brand Gradient
```
Primary:    linear-gradient(135deg, #FF6B6B, #4ECDC4)
Used for:   CTAs, logo mark, active states, loading bars
```

### Accent Colors
```
Coral:      #FF6B6B   (entities, errors, delete actions)
Teal:       #4ECDC4   (connections, success, pro badge)
Sky:        #45B7D1   (documents, info, links)
Yellow:     #F7DC6F   (dates, warnings)
Purple:     #BB8FCE   (money, financial)
Green:      #82E0AA   (concepts, ideas)
Orange:     #F0B27A   (documents, references)
```

### Text Opacity Scale
```
Full:       rgba(255,255,255, 1.0)    (headings, selected items)
Primary:    rgba(255,255,255, 0.6)    (body text, names)
Secondary:  rgba(255,255,255, 0.4)    (subtitles, counts)
Tertiary:   rgba(255,255,255, 0.3)    (labels, metadata)
Muted:      rgba(255,255,255, 0.2)    (timestamps, dividers)
Ghost:      rgba(255,255,255, 0.1)    (borders, outlines)
```

### Border Scale
```
Subtle:     rgba(255,255,255, 0.04)   (card dividers)
Light:      rgba(255,255,255, 0.06)   (card borders)
Medium:     rgba(255,255,255, 0.10)   (button borders)
Strong:     rgba(255,255,255, 0.15)   (active borders)
Highlight:  rgba(255,255,255, 0.25)   (hover borders)
```

## Typography

### Fonts
```
Headings:   'Space Grotesk', sans-serif  (geometric, techy)
Body:       'Outfit', system-ui, sans-serif  (clean, modern)
```

Load via Google Fonts in layout.js:
```html
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;600;700;800&family=Outfit:wght@300;400;500;600;700&display=swap" rel="stylesheet">
```

### Scale
```
Hero title:     36px, weight 800, Space Grotesk
Section title:  24px, weight 700, Space Grotesk
Component title: 15px, weight 700
Body:           13–14px, weight 400–500
Small:          11–12px, weight 400–600
Micro:          10px, weight 400–600 (labels, legends)
Tiny:           9px (zoom controls)
```

## Component Patterns

### Cards
```javascript
{
  padding: 20,
  borderRadius: 12,
  background: "rgba(255,255,255,0.03)",
  border: "1px solid rgba(255,255,255,0.06)",
  cursor: "pointer",
  transition: "all 0.2s",
}
// Hover: background 0.06, borderColor 0.12
```

### Buttons (Primary)
```javascript
{
  padding: "10px 20px",
  borderRadius: 8,
  border: "none",
  background: "linear-gradient(135deg, #FF6B6B, #4ECDC4)",
  color: "#000",
  fontSize: 13,
  fontWeight: 700,
  cursor: "pointer",
}
```

### Buttons (Ghost)
```javascript
{
  padding: "6px 14px",
  borderRadius: 6,
  border: "1px solid rgba(255,255,255,0.1)",
  background: "transparent",
  color: "rgba(255,255,255,0.4)",
  fontSize: 11,
  cursor: "pointer",
}
```

### Inputs
```javascript
{
  padding: "10px 14px",
  borderRadius: 8,
  border: "1px solid rgba(255,255,255,0.1)",
  background: "rgba(255,255,255,0.04)",
  color: "#fff",
  fontSize: 13,
  outline: "none",
}
```

### Type Badges
```javascript
{
  display: "inline-block",
  padding: "3px 10px",
  borderRadius: 4,
  background: entityColor + "20", // 20% opacity of entity color
  color: entityColor,
  fontSize: 11,
  fontWeight: 600,
  textTransform: "capitalize",
}
```

### Logo Mark
```javascript
// Diamond shape with gradient
{
  width: 32, height: 32,
  borderRadius: 8,
  background: "linear-gradient(135deg, #FF6B6B, #4ECDC4)",
  display: "flex", alignItems: "center", justifyContent: "center",
  fontSize: 14, fontWeight: 800, color: "#000",
}
// Content: ◆ (Unicode 25C6)
```

## Responsive Notes

- Graph canvas resizes with container (ResizeObserver pattern)
- Entity sidebar is 320px fixed, overlays on mobile
- Dashboard grid: `repeat(auto-fill, minmax(280px, 1fr))`
- Max content width on dashboard: 960px centered

## Animation

Keep animations minimal and functional:
- `transition: "all 0.2s"` on interactive elements
- Loading bar: `animation: "loading 1.5s ease-in-out infinite"`
- No spring physics or complex easing
