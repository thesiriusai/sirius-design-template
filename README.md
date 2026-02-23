# Sirius Pages

This repository contains the page components for your Sirius AI funnels. Each HTML file in `pages/` is a single step/screen in a funnel. Changes pushed to `main` are automatically synced to your Sirius workspace.

## Repository Structure

```
pages/              ← Your page components (one HTML file per page)
  hero.html         ← Example page
sirius.config.json  ← Registry of all pages (must be kept in sync)
```

## How It Works

1. You create HTML files in `pages/`
2. You register them in `sirius.config.json`
3. You push to `main`
4. Sirius auto-syncs the pages into your workspace
5. Pages appear in the Canvas editor where you drag them into funnels

## sirius.config.json

This file is the registry. Every page must be listed here to be synced.

```json
{
  "version": 1,
  "pages": {
    "page-slug": {
      "file": "pages/filename.html",
      "format": "html"
    }
  }
}
```

- **page-slug**: Unique identifier (kebab-case). Used in URLs and the Canvas.
- **file**: Path to the HTML file, must start with `pages/`.
- **format**: Always `"html"` for page components.

## Writing a Page Component

Each page is a self-contained HTML file. It receives data through CSS custom properties (theme) and JavaScript template variables (content).

### Minimal Example

```html
<section data-sirius-page>
  <h1>{{title}}</h1>
  <p>{{subtitle}}</p>
  <button data-element-id="cta">{{button_text}}</button>
</section>

<style>
  [data-sirius-page] {
    font-family: var(--sirius-font, 'Inter', sans-serif);
    background: var(--sirius-bg, #ffffff);
    color: var(--sirius-text, #111827);
    padding: 2rem;
    text-align: center;
  }
  button {
    background: var(--sirius-primary, #2563EB);
    color: #fff;
    border: none;
    border-radius: var(--sirius-radius, 8px);
    padding: 0.75rem 2rem;
    font-size: 1rem;
    cursor: pointer;
  }
</style>
```

### Key Concepts

#### Template Variables `{{variable_name}}`

Variables are replaced with actual content at runtime. They come from two sources:

1. **Props** — configured by the user in the Canvas editor (e.g., `{{title}}`, `{{button_text}}`)
2. **Session Bag** — collected from the user during the funnel (e.g., `{{name}}`, `{{email}}`)

#### Elements `data-element-id="..."`

Interactive elements (buttons, links, form submits) must have a `data-element-id` attribute. These become **output handles** on the Canvas — the user connects them to the next step.

```html
<!-- Single CTA -->
<button data-element-id="cta">Continue</button>

<!-- Multiple choices -->
<button data-element-id="option-a">Option A</button>
<button data-element-id="option-b">Option B</button>
```

Each element ID creates a separate output handle, enabling branching logic.

#### Theme Variables (CSS Custom Properties)

The brand's design tokens are injected as CSS custom properties:

| Variable | Description | Default |
|----------|-------------|---------|
| `--sirius-primary` | Primary brand color | `#2563EB` |
| `--sirius-secondary` | Secondary color | `#1E40AF` |
| `--sirius-bg` | Background color | `#ffffff` |
| `--sirius-text` | Text color | `#111827` |
| `--sirius-font` | Font family | `'Inter', sans-serif` |
| `--sirius-radius` | Border radius | `8px` |

Always use `var(--sirius-*)` with fallbacks so pages look good even without theme injection.

#### Forms and Data Collection

To collect user input, use standard HTML forms with `data-element-id` on the submit button:

```html
<form>
  <input type="email" name="email" placeholder="Enter your email" required />
  <button type="submit" data-element-id="submit">Get My Results</button>
</form>
```

Form field `name` attributes become keys in the session bag. When the user submits, `bag.email = "user@example.com"` is set automatically.

#### Images

Use `{{variable_name}}` for dynamic images:

```html
<img src="{{hero_image}}" alt="{{hero_alt}}" />
```

The user uploads/selects images in the Canvas editor when configuring the page.

### Page Types & Patterns

#### Quiz / Selection Step

```html
<section data-sirius-page>
  <h2>{{title}}</h2>
  <div class="options">
    <button data-element-id="option-1" class="option-card">
      <img src="{{option_1_image}}" alt="" />
      <span>{{option_1_label}}</span>
    </button>
    <button data-element-id="option-2" class="option-card">
      <img src="{{option_2_image}}" alt="" />
      <span>{{option_2_label}}</span>
    </button>
  </div>
</section>
```

#### Lead Capture

```html
<section data-sirius-page>
  <h2>{{title}}</h2>
  <p>{{subtitle}}</p>
  <form>
    <input type="text" name="name" placeholder="Your name" required />
    <input type="email" name="email" placeholder="Your email" required />
    <button type="submit" data-element-id="submit">{{button_text}}</button>
  </form>
</section>
```

#### Results / Personalized

```html
<section data-sirius-page>
  <h1>{{personalized_headline}}</h1>
  <p>{{recommendation}}</p>
  <button data-element-id="cta">{{cta_text}}</button>
</section>
```

#### Interstitial / Loading

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
    border: 3px solid var(--sirius-primary, #2563EB);
    border-top-color: transparent;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }
</style>
```

## Adding a New Page

1. Create `pages/my-page.html` with your HTML + CSS
2. Add it to `sirius.config.json`:
   ```json
   {
     "version": 1,
     "pages": {
       "hero": { "file": "pages/hero.html", "format": "html" },
       "my-page": { "file": "pages/my-page.html", "format": "html" }
     }
   }
   ```
3. Commit and push to `main`
4. Sirius auto-syncs within seconds

## Rules

- Every HTML file must be inside `pages/`
- Every page must be registered in `sirius.config.json`
- Page slugs must be unique, kebab-case
- Use `data-element-id` on all clickable/submittable elements
- Use `{{variables}}` for dynamic content
- Use `var(--sirius-*)` CSS properties for theming
- Keep pages self-contained (inline styles, no external dependencies)
- Pages are rendered inside an iframe — external scripts won't work
