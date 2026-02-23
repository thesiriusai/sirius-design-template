# Sirius Pages

This repository contains the page components for your Sirius AI funnels. Each HTML file in `pages/` is a single step/screen in a funnel. Changes pushed to `main` are automatically synced to your Sirius workspace.

## Repository Structure

```
pages/
  hero.html              ← Simple page (single file)
  results/
    index.html           ← Page with separate JS/CSS
    chart.js
    style.css
sirius.config.json       ← Registry of all pages
```

## How It Works

1. You create HTML files in `pages/`
2. You register them in `sirius.config.json` (including elements, scripts, styles)
3. You push to `main`
4. Sirius auto-syncs the pages into your workspace
5. Pages appear in the Canvas editor where you drag them into funnels

## sirius.config.json

This file is the registry. Every page must be listed here to be synced.

```json
{
  "version": 1,
  "pages": {
    "skin-type": {
      "file": "pages/skin-type.html",
      "elements": ["oily", "dry", "combination", "normal"]
    },
    "email-capture": {
      "file": "pages/email-capture.html",
      "elements": ["submit"]
    },
    "results": {
      "file": "pages/results/index.html",
      "elements": ["cta"],
      "scripts": ["pages/results/chart.js"],
      "styles": ["pages/results/style.css"]
    }
  }
}
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `file` | Yes | Path to the HTML file. Must start with `pages/`. |
| `elements` | Yes | Array of element IDs. Each becomes an **output handle** on the Canvas node. |
| `scripts` | No | Array of JS file paths to include with the page. |
| `styles` | No | Array of CSS file paths to include with the page. |

### Elements

Elements are the interactive parts of a page — buttons, form submits, choices. Each element ID in the config must match a `data-element-id` attribute in the HTML.

On the Canvas, each element becomes an **output handle** that the user connects to the next step. This is how branching works:

- `"elements": ["submit"]` → **1 output** (linear flow)
- `"elements": ["yes", "no"]` → **2 outputs** (branching)
- `"elements": ["oily", "dry", "combo", "normal"]` → **4 outputs** (quiz branching)

The Canvas enforces this — you can only connect the outputs that exist.

## Writing a Page Component

Each page is an HTML file. It receives dynamic content through template variables. You can include everything inline, or split JS/CSS into separate files.

### Simple Page (single file)

```html
<section data-sirius-page>
  <h1>{{title}}</h1>
  <p>{{subtitle}}</p>
  <button data-element-id="cta">{{button_text}}</button>
</section>

<style>
  [data-sirius-page] { padding: 2rem; text-align: center; }
</style>
```

### Page with Separate JS/CSS

For interactive pages (charts, animations, visualizations), split into multiple files:

**pages/results/index.html**
```html
<section data-sirius-page>
  <h1>Your Results</h1>
  <canvas id="results-chart" width="400" height="300"></canvas>
  <button data-element-id="cta">{{cta_text}}</button>
</section>
```

**pages/results/chart.js**
```js
const ctx = document.getElementById('results-chart').getContext('2d');
// Use any vanilla JS — Chart.js, D3, GSAP, Lottie, Three.js, etc.
// Access bag data via window.__sirius.bag
const score = window.__sirius.bag.score || 75;
// ... draw chart based on user's answers
```

**pages/results/style.css**
```css
#results-chart { max-width: 100%; margin: 1rem auto; }
```

**Config:**
```json
"results": {
  "file": "pages/results/index.html",
  "elements": ["cta"],
  "scripts": ["pages/results/chart.js"],
  "styles": ["pages/results/style.css"]
}
```

Scripts have access to `window.__sirius.bag` — the session data collected from previous steps.

### Template Variables `{{variable_name}}`

Variables are replaced with actual content at runtime. They come from two sources:

1. **Props** — configured by the user in the Canvas editor (e.g., `{{title}}`, `{{button_text}}`)
2. **Session Bag** — collected from the user during the funnel (e.g., `{{name}}`, `{{email}}`)

### Elements `data-element-id="..."`

Every interactive element must have a `data-element-id` attribute **and** be listed in `elements` in the config.

```html
<!-- Single CTA — elements: ["cta"] -->
<button data-element-id="cta">Continue</button>

<!-- Multiple choices — elements: ["option-a", "option-b", "option-c"] -->
<button data-element-id="option-a">Option A</button>
<button data-element-id="option-b">Option B</button>
<button data-element-id="option-c">Option C</button>

<!-- Form submit — elements: ["submit"] -->
<form>
  <input type="email" name="email" placeholder="Your email" required />
  <button type="submit" data-element-id="submit">Get Results</button>
</form>
```

### Forms and Data Collection

Form field `name` attributes become keys in the session bag:

```html
<input type="text" name="name" />    → bag.name = "John"
<input type="email" name="email" />  → bag.email = "john@example.com"
```

This data is available in later pages via `{{name}}`, `{{email}}`, and `window.__sirius.bag`.

### Images

Use template variables for dynamic images:

```html
<img src="{{hero_image}}" alt="{{hero_alt}}" />
```

## Page Patterns

### Quiz / Selection (multiple outputs)

```html
<section data-sirius-page>
  <h2>{{title}}</h2>
  <div class="options">
    <button data-element-id="option-1">{{option_1_label}}</button>
    <button data-element-id="option-2">{{option_2_label}}</button>
    <button data-element-id="option-3">{{option_3_label}}</button>
  </div>
</section>
```

Config: `"elements": ["option-1", "option-2", "option-3"]`

### Lead Capture (single output)

```html
<section data-sirius-page>
  <h2>{{title}}</h2>
  <form>
    <input type="text" name="name" placeholder="Your name" required />
    <input type="email" name="email" placeholder="Your email" required />
    <button type="submit" data-element-id="submit">{{button_text}}</button>
  </form>
</section>
```

Config: `"elements": ["submit"]`

### Interactive Results with Chart

```html
<section data-sirius-page>
  <h1>{{headline}}</h1>
  <canvas id="chart"></canvas>
  <p>{{recommendation}}</p>
  <button data-element-id="cta">{{cta_text}}</button>
</section>

<script>
  const bag = window.__sirius.bag;
  const ctx = document.getElementById('chart').getContext('2d');
  // Draw personalized chart based on bag data
</script>
```

Config: `"elements": ["cta"]`

### Animated Loading / Interstitial

```html
<section data-sirius-page>
  <div class="loading">
    <div class="spinner"></div>
    <h2>{{message}}</h2>
  </div>
</section>

<style>
  .spinner {
    width: 40px; height: 40px;
    border: 3px solid #2563EB;
    border-top-color: transparent;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }
</style>
```

Config: `"elements": ["auto"]` — auto-advance after a timeout set in the Canvas.

## Adding a New Page

1. Create your HTML file (and optional JS/CSS) in `pages/`
2. Add `data-element-id` to every button/submit
3. Register in `sirius.config.json`:
   ```json
   "my-page": {
     "file": "pages/my-page.html",
     "elements": ["submit"]
   }
   ```
   Or with separate files:
   ```json
   "my-page": {
     "file": "pages/my-page/index.html",
     "elements": ["submit"],
     "scripts": ["pages/my-page/app.js"],
     "styles": ["pages/my-page/style.css"]
   }
   ```
4. Commit and push to `main`
5. Sirius auto-syncs within seconds

## Rules

- All files must be inside `pages/`
- Every page must be registered in `sirius.config.json`
- Page slugs must be unique, kebab-case
- Every `data-element-id` in HTML must be listed in `elements` in the config
- Use `{{variables}}` for dynamic content (HTML only, not in JS/CSS files)
- Access session data in JS via `window.__sirius.bag`
- No external CDN dependencies — bundle everything locally or inline
