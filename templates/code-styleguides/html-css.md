# HTML/CSS Styleguide

Standards for writing semantic, accessible, and maintainable HTML and CSS.

## HTML General Principles

- Use semantic elements
- Ensure accessibility (WCAG 2.1)
- Write valid markup
- Keep structure separate from style

## HTML Naming

| Element | Convention | Example |
|---------|------------|---------|
| IDs | kebab-case | `user-profile` |
| Classes | kebab-case or BEM | `card`, `card__header` |
| Data attributes | kebab-case | `data-user-id` |
| Custom elements | kebab-case | `user-card` |

## Semantic Structure

### Document Outline

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
</head>
<body>
  <header>
    <nav aria-label="Main navigation">...</nav>
  </header>

  <main>
    <article>
      <h1>Article Title</h1>
      <section>...</section>
    </article>
  </main>

  <footer>...</footer>
</body>
</html>
```

### Semantic Elements

```html
<!-- Use semantic elements over generic divs -->
<article>  <!-- Self-contained content -->
<section>  <!-- Thematic grouping -->
<aside>    <!-- Tangentially related content -->
<nav>      <!-- Navigation links -->
<header>   <!-- Introductory content -->
<footer>   <!-- Footer content -->
<main>     <!-- Main content (one per page) -->
<figure>   <!-- Self-contained media -->
```

### Heading Hierarchy

```html
<!-- Maintain logical heading order -->
<h1>Main Title</h1>
  <h2>Section</h2>
    <h3>Subsection</h3>
  <h2>Another Section</h2>

<!-- Don't skip levels -->
<!-- Bad: h1 → h3 -->
<!-- Good: h1 → h2 → h3 -->
```

## Accessibility

### ARIA Labels

```html
<!-- Add labels to interactive elements -->
<button aria-label="Close dialog">
  <svg>...</svg>
</button>

<!-- Use aria-labelledby for complex labels -->
<section aria-labelledby="section-title">
  <h2 id="section-title">User Settings</h2>
</section>

<!-- Describe current state -->
<button aria-expanded="false" aria-controls="menu">
  Menu
</button>
```

### Forms

```html
<form>
  <div class="form-group">
    <label for="email">Email Address</label>
    <input
      type="email"
      id="email"
      name="email"
      required
      aria-describedby="email-hint"
    >
    <p id="email-hint" class="hint">We'll never share your email.</p>
  </div>

  <fieldset>
    <legend>Notification Preferences</legend>
    <label>
      <input type="checkbox" name="notify-email">
      Email notifications
    </label>
  </fieldset>
</form>
```

### Images

```html
<!-- Meaningful images need alt text -->
<img src="chart.png" alt="Sales increased 25% in Q4">

<!-- Decorative images use empty alt -->
<img src="decoration.svg" alt="">

<!-- Complex images -->
<figure>
  <img src="diagram.png" alt="System architecture diagram">
  <figcaption>Figure 1: High-level system architecture</figcaption>
</figure>
```

## CSS General Principles

- Use mobile-first approach
- Avoid `!important`
- Keep specificity low
- Use CSS custom properties for theming

## CSS Naming (BEM)

```css
/* Block: Standalone component */
.card { }

/* Element: Part of block (double underscore) */
.card__header { }
.card__body { }
.card__footer { }

/* Modifier: Variation (double hyphen) */
.card--featured { }
.card__header--large { }
```

### BEM in HTML

```html
<article class="card card--featured">
  <header class="card__header card__header--large">
    <h2 class="card__title">Title</h2>
  </header>
  <div class="card__body">
    <p class="card__text">Content</p>
  </div>
  <footer class="card__footer">
    <button class="card__action">Read More</button>
  </footer>
</article>
```

## CSS Custom Properties

```css
:root {
  /* Colors */
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --color-success: #22c55e;
  --color-error: #ef4444;

  /* Typography */
  --font-family: system-ui, sans-serif;
  --font-size-base: 1rem;
  --line-height: 1.5;

  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 2rem;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.card {
  font-family: var(--font-family);
  padding: var(--spacing-md);
  box-shadow: var(--shadow-md);
}
```

## Layout

### Flexbox

```css
/* Common flex patterns */
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.flex-between {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.flex-column {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-md);
}
```

### Grid

```css
/* Responsive grid */
.grid {
  display: grid;
  gap: var(--spacing-md);
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
}

/* Named areas */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 250px 1fr;
}
```

## Responsive Design

### Mobile-First

```css
/* Base styles for mobile */
.card {
  padding: var(--spacing-sm);
}

/* Tablet and up */
@media (min-width: 768px) {
  .card {
    padding: var(--spacing-md);
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .card {
    padding: var(--spacing-lg);
  }
}
```

### Container Queries

```css
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    display: flex;
    gap: var(--spacing-md);
  }
}
```

## SCSS/Sass

### Nesting (Max 3 Levels)

```scss
// Good: Shallow nesting
.card {
  padding: var(--spacing-md);

  &__header {
    border-bottom: 1px solid var(--color-border);
  }

  &--featured {
    border: 2px solid var(--color-primary);
  }
}

// Avoid: Deep nesting
// .card {
//   .header {
//     .title {
//       .icon {
//         // Too deep!
//       }
//     }
//   }
// }
```

### Mixins and Functions

```scss
@mixin button-variant($bg, $text) {
  background-color: $bg;
  color: $text;

  &:hover {
    background-color: darken($bg, 10%);
  }
}

.button--primary {
  @include button-variant(var(--color-primary), white);
}
```

### File Organization

```
styles/
├── _variables.scss
├── _mixins.scss
├── _reset.scss
├── _typography.scss
├── components/
│   ├── _button.scss
│   ├── _card.scss
│   └── _form.scss
├── layouts/
│   ├── _header.scss
│   └── _grid.scss
└── main.scss
```

## Quick Reference

| Aspect | Recommendation |
|--------|----------------|
| Naming | BEM methodology |
| Specificity | Keep low, avoid IDs for styling |
| Units | rem for fonts, px for borders |
| Colors | CSS custom properties |
| Layout | Flexbox/Grid over floats |
| Responsive | Mobile-first |
| Nesting (SCSS) | Max 3 levels |
| !important | Avoid |
