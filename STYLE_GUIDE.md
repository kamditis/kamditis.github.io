# Website Style Guide

This guide defines the visual identity for all plugin/tool index pages hosted on `kamditis.github.io`. Follow these specifications to ensure consistency across all project pages.

## Design Principles

1. **Professional** - Clean, minimal, no unnecessary decoration
2. **BIM-focused** - Technical, precise, engineering aesthetic
3. **Dark theme** - Easy on eyes, modern appearance
4. **No source references** - Repos are private; never link to GitHub source code
5. **No company branding** - Personal portfolio, not company-affiliated

---

## Color Palette

```css
:root {
    --bg-primary: #0f172a;      /* Main background */
    --bg-secondary: #1e293b;    /* Cards, sections */
    --bg-tertiary: #334155;     /* Icons, badges */
    --text-primary: #f1f5f9;    /* Headings, important text */
    --text-secondary: #94a3b8;  /* Body text, descriptions */
    --text-muted: #64748b;      /* Meta text, labels */
    --accent: #60a5fa;          /* Links, icons, buttons */
    --accent-hover: #3b82f6;    /* Button hover state */
    --border: #334155;          /* Borders, dividers */
    --success: #22c55e;         /* Grade A, checkmarks */
    --warning: #eab308;         /* Grade C */
    --error: #ef4444;           /* Grade F, errors */
}
```

### Grade Colors (if applicable)
- **A**: `#22c55e` (green)
- **B**: `#84cc16` (lime)
- **C**: `#eab308` (yellow)
- **D**: `#f97316` (orange)
- **F**: `#ef4444` (red)

---

## Typography

**Font Family**: Inter (Google Fonts)

```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

```css
font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
```

### Font Weights
- **400**: Body text
- **500**: Navigation, buttons, labels
- **600**: Section headers, card titles
- **700**: Page title (h1)

### Font Sizes
| Element | Size | Weight |
|---------|------|--------|
| Page title (h1) | 3rem | 700 |
| Section header (h2) | 1.5rem | 600 |
| Card title (h3) | 0.95rem | 600 |
| Body text | 0.875rem | 400 |
| Meta/labels | 0.75rem | 500 |
| Small text | 0.7rem | 400 |

---

## Layout

### Max Widths
- **Content sections**: 1000px
- **Hero section**: 900px
- **Narrow content**: 600px

### Spacing
- **Section padding**: 80px vertical, 40px horizontal
- **Card padding**: 24px
- **Gap between cards**: 16px
- **Gap between sections**: 48-60px

### Responsive Breakpoints
```css
@media (max-width: 768px) {
    /* Tablet and below */
    section { padding: 60px 20px; }
    .hero h1 { font-size: 2rem; }
}

@media (max-width: 600px) {
    /* Mobile */
    header { padding: 60px 20px 40px; }
}
```

---

## Components

### Header (Sticky Navigation)

```html
<header>
    <a href="https://kamditis.github.io" class="logo">
        PluginName
    </a>
    <nav>
        <a href="#features">Features</a>
        <a href="#section2">Section 2</a>
        <!-- Add relevant sections -->
    </nav>
</header>
```

```css
header {
    border-bottom: 1px solid var(--border);
    padding: 16px 40px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    position: sticky;
    top: 0;
    background: rgba(15, 23, 42, 0.95);
    backdrop-filter: blur(10px);
    z-index: 100;
}

.logo {
    font-weight: 600;
    font-size: 1rem;
    color: var(--text-primary);
    text-decoration: none;
}

nav { display: flex; gap: 32px; align-items: center; }
nav a {
    color: var(--text-secondary);
    text-decoration: none;
    font-size: 0.875rem;
    font-weight: 500;
    transition: color 0.2s;
}
nav a:hover { color: var(--text-primary); }
```

### Hero Section

```html
<section class="hero">
    <span class="hero-badge">PLATFORM TYPE</span>
    <h1>Plugin Name</h1>
    <p>Brief description of what the plugin does. Keep to 1-2 sentences.</p>
    <div class="hero-actions">
        <a href="DOWNLOAD_URL" class="btn btn-primary">
            <!-- Download SVG icon -->
            Download v1.0.0
        </a>
    </div>
</section>
```

```css
.hero {
    padding: 100px 40px 80px;
    max-width: 900px;
    margin: 0 auto;
}

.hero-badge {
    display: inline-block;
    background: var(--bg-secondary);
    border: 1px solid var(--border);
    padding: 6px 12px;
    border-radius: 6px;
    font-size: 0.75rem;
    font-weight: 500;
    color: var(--text-secondary);
    margin-bottom: 24px;
    text-transform: uppercase;
    letter-spacing: 0.05em;
}

.hero h1 {
    font-size: 3rem;
    font-weight: 700;
    line-height: 1.1;
    margin-bottom: 20px;
    letter-spacing: -0.02em;
}

.hero p {
    font-size: 1.125rem;
    color: var(--text-secondary);
    max-width: 600px;
    margin-bottom: 40px;
}
```

### Buttons

```html
<!-- Primary button (accent color) -->
<a href="#" class="btn btn-primary">
    <svg><!-- icon --></svg>
    Button Text
</a>

<!-- Secondary button (outline) -->
<a href="#" class="btn btn-secondary">
    Button Text
</a>
```

```css
.btn {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 12px 20px;
    border-radius: 8px;
    text-decoration: none;
    font-weight: 500;
    font-size: 0.875rem;
    transition: all 0.2s;
    border: 1px solid transparent;
}

.btn-primary {
    background: var(--accent);
    color: var(--bg-primary);
}
.btn-primary:hover {
    background: var(--accent-hover);
}

.btn-secondary {
    background: transparent;
    color: var(--text-primary);
    border-color: var(--border);
}
.btn-secondary:hover {
    background: var(--bg-secondary);
    border-color: var(--text-muted);
}
```

### Info Bar (Key Facts)

```html
<div class="info-bar">
    <div class="info-bar-content">
        <div class="info-item">
            <div class="info-icon">
                <svg><!-- icon --></svg>
            </div>
            <div class="info-text">
                <strong>Main Text</strong>
                <span>Subtext</span>
            </div>
        </div>
        <!-- Repeat for each info item -->
    </div>
</div>
```

### Feature Cards

```html
<div class="features-grid">
    <div class="feature">
        <div class="feature-header">
            <div class="feature-icon">
                <svg><!-- icon --></svg>
            </div>
            <h3>Feature Name</h3>
        </div>
        <p>Feature description. Keep concise, 1-2 sentences.</p>
    </div>
    <!-- Repeat for each feature -->
</div>
```

```css
.features-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 16px;
}

.feature {
    background: var(--bg-secondary);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 24px;
    transition: border-color 0.2s;
}

.feature:hover {
    border-color: var(--text-muted);
}
```

### Section Headers

```html
<div class="section-header">
    <h2>Section Title</h2>
    <p>Brief section description</p>
</div>
```

```css
.section-header {
    margin-bottom: 48px;
}

.section-header h2 {
    font-size: 1.5rem;
    font-weight: 600;
    margin-bottom: 8px;
    letter-spacing: -0.01em;
}

.section-header p {
    color: var(--text-secondary);
    font-size: 0.95rem;
}
```

### Footer

```html
<footer>
    <p><a href="https://kamditis.github.io">All Projects</a></p>
</footer>
```

```css
footer {
    border-top: 1px solid var(--border);
    padding: 32px 40px;
    text-align: center;
}

footer p {
    font-size: 0.8rem;
    color: var(--text-muted);
}

footer a {
    color: var(--text-secondary);
    text-decoration: none;
}

footer a:hover {
    color: var(--text-primary);
}
```

---

## Icons

Use inline SVGs from [Feather Icons](https://feathericons.com/) or similar minimal icon sets.

**Standard attributes:**
```html
<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
    <!-- path data -->
</svg>
```

**Common icons:**
- Download: `<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/>`
- Check: `<polyline points="20 6 9 17 4 12"/>`
- Globe: `<circle cx="12" cy="12" r="10"/><path d="M2 12h20"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/>`

**Do NOT use emojis.** Always use SVG icons.

---

## Page Structure Template

```
1. Header (sticky)
   - Logo/plugin name (links to kamditis.github.io)
   - Navigation links to page sections

2. Hero Section
   - Badge (platform type)
   - Title (plugin name)
   - Description (1-2 sentences)
   - Download button

3. Info Bar (optional)
   - 2-4 key facts (versions, requirements, etc.)

4. Features Section
   - 2-column grid of feature cards
   - 4-6 features typical

5. Additional Sections (as needed)
   - Workflow steps
   - Scoring system
   - Requirements list

6. Footer
   - "All Projects" link back to portfolio
```

---

## Do's and Don'ts

### DO:
- Use the exact color values from the palette
- Keep descriptions concise
- Use SVG icons consistently
- Link download buttons to GitHub Releases
- Include "All Projects" link in footer
- Test responsive behavior

### DON'T:
- Link to GitHub source code (repos are private)
- Use emojis anywhere
- Add company branding
- Use gradients or flashy effects
- Add unnecessary animations
- Include copyright notices

---

## Example: Complete Page HTML

See `ScanAnalysis/docs/index.html` for a complete reference implementation.

---

## Adding to Portfolio

After creating a plugin page, add it to the main portfolio (`kamditis.github.io/index.html`):

```html
<!-- In the appropriate platform section -->
<div class="plugin-card">
    <h3>Plugin Name</h3>
    <div class="plugin-meta">Platform Version Range</div>
    <p>Brief description of the plugin.</p>
    <a href="https://kamditis.github.io/PluginName/" class="btn btn-primary">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
            <polyline points="7 10 12 15 17 10"/>
            <line x1="12" y1="15" x2="12" y2="3"/>
        </svg>
        Download
    </a>
</div>
```
