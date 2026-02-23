# HTML & CSS Conventions

Purpose
- Styling and markup conventions to ensure maintainable and accessible front-end code.

Structure
- Organize styles per component: `styles/components/`, global styles under `styles/global/`.
- Use a CSS preprocessor (SCSS) or utility-first frameworks (Tailwind) consistently across the project.

Naming
- Prefer BEM for component classes: `block__element--modifier` to avoid specificity issues.
- Avoid deep selector nesting and avoid styling by tag names only (prefer classes).

Accessibility
- Follow WCAG 2.1 AA standards: semantic HTML, proper ARIA where needed, color contrast, keyboard navigation.
- Include `alt` attributes for images and label form controls correctly.

Responsive design
- Mobile-first approach; use relative units (rem, %) and media queries for breakpoints.

Linting and tooling
- Use `stylelint` with a shared config and run `stylelint "**/*.{css,scss}" --config .stylelintrc` in CI.
- Use Prettier for CSS formatting where applicable.

Testing
- Component tests: use Testing Library (React Testing Library) to assert on behavior, not implementation details.
- E2E/UI tests: use Playwright or Cypress for browser-level tests; include selectors that are stable (data-testid attributes) and avoid brittle CSS selector tests.

Performance
- Avoid large CSS bundles; use code-splitting and critical CSS inlining for initial render.

Examples

```html
<!-- BEM example -->
<div class="card card--featured">
  <h2 class="card__title">Title</h2>
  <p class="card__body">Body text</p>
</div>
```

```scss
.card {
  &__title { font-weight: 600; }
  &__body { color: $text-color; }
  &--featured { border: 1px solid $accent; }
}
```
