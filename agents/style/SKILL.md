# STYLE BRAIN — Design System Engineering Knowledge Base
# Role: Design System owner at BYMAV. Creates base reusable components. Reports to LEO.
# This file: KNOWLEDGE. Architecture, tokens, governance, real references.
# Max 200 lines. Dense. No fluff. REAL references from REAL systems.

---

## 1. REAL PEOPLE & REFERENCES

### Brad Frost — "Atomic Design" (2013, atomicdesign.bradfrost.com)
- Creator of Atomic Design. The framework is NOT linear — it's a concurrent mental model.
- **Atoms:** Irreducible UI elements (button, input, label, icon). Can't simplify further without losing function.
- **Molecules:** Simple groups that take on NEW properties (search form = input + button + icon).
- **Organisms:** Complex UI sections from molecules (header, product grid, navigation system).
- **Templates:** Page-level skeletons — layout structure without content. "The page's skeleton."
- **Pages:** Real content in templates. Tests whether patterns hold with dynamic content (long names, empty carts).
- Key insight: "The right question isn't 'Is this an atom or a molecule?' It's 'Does our system help build better products faster?'"
- On versioning: single-library vs component-level — both valid. Ref: [bradfrost.com](https://bradfrost.com/blog/post/design-system-versioning-single-library-or-individual-components/)

### Nathan Curtis — EightShapes
- "A design system is a product serving products." Treat it like a product with users (the dev teams).
- Versioning: batch breaking changes. Deprecate gradually with EOL dates. Changelogs + migration guides.
- Ref: [medium.com/eightshapes-llc](https://medium.com/eightshapes-llc/versioning-design-systems-48cceb5ace4d)

### Jina Anne — Design Tokens Community Group (W3C)
- Co-creator of Design Tokens at Salesforce Lightning. Three tiers: Global → Alias → Component.
- **W3C Spec 2025.10 (STABLE)**: First production-ready, vendor-neutral token format.
  - Files: `.tokens` or `.tokens.json` (media type `application/design-tokens+json`).
  - Supports: composite types (shadows, gradients, borders, typography), aliases via curly braces.
  - Adopted by: Figma, Sketch, Penpot, Framer, Style Dictionary, Tokens Studio, Terrazzo.
  - Ref: [designtokens.org](https://www.designtokens.org/tr/drafts/format/), [W3C announcement](https://www.w3.org/community/design-tokens/2025/10/28/design-tokens-specification-reaches-first-stable-version/)

---

## 2. REAL DESIGN SYSTEMS — ARCHITECTURE DEEP DIVES

### Shopify Polaris (2025 — Web Components era)
- Rebuilt on **Web Components**: framework-agnostic (React, Vue, vanilla JS, none). Smaller, faster.
- **Unified across surfaces**: Admin, Checkout, Customer Accounts share components and props.
- CDN-delivered: apps auto-inherit updates without code changes.
- Token architecture: CSS custom properties for color, spacing, border-radius, typography, motion.
- Ref: [shopify.com/partners/blog/polaris-unified-and-for-the-web](https://www.shopify.com/partners/blog/polaris-unified-and-for-the-web)

### GitHub Primer
- Design tokens stored as JSON in `src/tokens/`, compiled with Style Dictionary.
- **Multi-mode approach**: light + dark + high-contrast. Overrides system: only include differences from main mode.
- Philosophy: simplicity and clarity. No color as primary emphasis — structure first.
- Ref: [primer.style](https://primer.style/), [github.com/primer/primitives](https://github.com/primer/primitives)

### IBM Carbon
- **Role-based token system**: tokens represent PURPOSE, not appearance. Themes assign values to roles.
- Three token domains: Color (role-based), Typography (pre-set size/weight/leading for IBM Plex), Spacing (2 scales — component + layout).
- v11 migrated to robust token architecture for better clarity and flexibility.
- Ref: [carbondesignsystem.com](https://carbondesignsystem.com/)

### Material Design 3 (Google, 2025)
- Dynamic Color: algorithmic scheme from user wallpaper. Personalization at system level.
- Token coverage: color, type, shape, spacing, motion, elevation. ALL tokenized.
- 2025 addition: shape and motion tokens system-wide for "Expressive" experiences.
- Ref: [m3.material.io/foundations/design-tokens](https://m3.material.io/foundations/design-tokens)

---

## 3. DESIGN TOKENS — BYMAV ARCHITECTURE

### Three Tiers (industry standard)
```
Global Tokens    → --color-green-500: #22c55e      (raw values)
                     ↓
Alias Tokens     → --color-success: var(--color-green-500)  (semantic meaning)
                     ↓
Component Tokens → --button-success-bg: var(--color-success)  (scoped to component)
```

### BYMAV Token Table
| Token | Light | Dark |
|-------|-------|------|
| --background | #ffffff | #08080C |
| --card | #f9fafb | #0E0E14 |
| --border | rgba(0,0,0,0.06) | rgba(255,255,255,0.06) |
| --text | #1f2937 | #d4d4d4 |
| --muted | #6b7280 | rgba(156,163,175,0.8) |
| --success | #22c55e | #22c55e |
| --danger | #ef4444 | #ef4444 |

### Scales
- **Spacing (4px base):** 4 → 8 → 12 → 16 → 20 → 24 → 32 → 40 → 48 → 64
- **Typography:** 11/12/14/16/18/20/24/30/36/48 (ratio ~1.2 minor third)
- **Border radius:** 6px (inputs) → 8px (buttons) → 10px (small cards) → 12px (large cards)

---

## 4. COMPONENT ENGINEERING PATTERNS

### Structure (cva + TypeScript)
```typescript
import { cva, type VariantProps } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-lg font-medium transition-colors',
  {
    variants: {
      variant: {
        primary: 'bg-emerald-600 text-white hover:bg-emerald-500',
        ghost: 'text-gray-400 hover:text-white hover:bg-white/5',
        danger: 'bg-red-600/10 text-red-400 hover:bg-red-600/20',
      },
      size: { sm: 'h-8 px-3 text-xs', md: 'h-10 px-4 text-sm', lg: 'h-12 px-6 text-base' },
    },
    defaultVariants: { variant: 'primary', size: 'md' },
  }
)
```

### Checklist Per Component
- [ ] Props typed with `interface` (not `type`)
- [ ] Variants with `cva()` (not inline ternaries)
- [ ] Design tokens (not hardcoded colors)
- [ ] Dark mode via Tailwind `dark:` or isDark conditional
- [ ] aria-label/role for accessibility
- [ ] Responsive (mobile-first)
- [ ] Documented in DESIGN_CATALOG.md

---

## 5. STORYBOOK & DOCUMENTATION

### Best Practices (2025)
- **Autodocs**: auto-generates usage, props, code snippets. Single source of truth.
- **MDX**: long-form docs mixing Markdown + interactive JSX. Beyond simple demos.
- **Stories as tests**: stories double as regression tests. Visual + functional.
- **Per-component docs**: description, variants, states, responsive behavior, do's/don'ts.
- **Interactive theming toggle**: show components across brand themes in real-time.
- Ref: [storybook.js.org/docs/writing-docs](https://storybook.js.org/docs/writing-docs)

---

## 6. VERSIONING & GOVERNANCE

### SemVer for Design Systems (Nathan Curtis pattern)
- MAJOR: breaking changes (API changes, removed props, renamed tokens)
- MINOR: new features, new components, backward-compatible additions
- PATCH: bug fixes, visual tweaks, accessibility fixes

### Governance Rules
- **Batch breaking changes** — never piecemeal. Schedule major releases.
- **Deprecation warnings** — mark deprecated in code AND design libs. Set EOL dates.
- **Changelogs** — what changed + why + migration steps. ALWAYS.
- **Preview environments** — teams test upcoming changes before GA.
- **Contribution flow:** proposal → design review → code + tests → doc update → release.

---

## 7. ANTI-PATTERNS

| Error | Why Bad | Do Instead |
|-------|---------|------------|
| Hex hardcoded | Breaks themes | CSS var / token |
| Prop `className` passthrough | Breaks consistency | cva variants |
| `!important` | Unpredictable cascade | Correct specificity |
| Component >200 LOC | Hard to maintain | Split sub-components |
| State inline (`useState` in UI) | Coupling | Separate logic from presentation |
| No token tiers | Changing one color = find-replace | Global → Alias → Component |
| No versioning | Silent breaking changes | SemVer + changelogs |
| Framework-locked tokens | Can't share cross-platform | W3C format (.tokens.json) |

---

## 8. ACCESSIBILITY (WCAG 2.1 AA)

| Criterion | Minimum | Ideal |
|-----------|---------|-------|
| Normal text contrast | 4.5:1 | 7:1 (AAA) |
| Large text contrast | 3:1 | 4.5:1 |
| Touch target | 44x44px | 48x48px |
| Focus visible | outline 2px | outline + ring |
| Screen reader | aria-label | aria-describedby |

Tools: WebAIM Contrast Checker, axe DevTools (Chrome), Lighthouse, Figma A11y plugins.

---

## TEST ACCOUNTS — SCREENSHOTS LOGADOS
# Se precisar screenshot de componente em contexto real (nao Storybook):
# VITA: qa@vita-ai.cloud / VitaTest2026
# PIXIO: test@pixio.cloud / Test123!
# REGRA: LOGAR PRIMEIRO → navegar → SO ENTAO screenshot.
# Screenshot de login/splash = INVALIDO.


## Licoes Aprendidas (auto-feedback)
- [2026-03-12] [edit_without_read] SEMPRE ler o arquivo antes de editar. Edit sem Read = edit cego = erros evitaveis.
