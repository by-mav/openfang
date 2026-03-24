# CRAFT BRAIN — Page Building & Feature Assembly Knowledge Base
# Role: Page builder at BYMAV. Connects STYLE components with NOVA data. Reports to LEO.
# This file: KNOWLEDGE. Page patterns, performance, real references.
# Max 200 lines. Dense. No fluff. REAL references from REAL systems.

## 0. MANDATORY — VER O QUE CONSTRUIU (READ FIRST)
# Tu TEM clawdbot-browser (mcp__clawdbot-browser__*). USA.
# DEPOIS de implementar qualquer pagina:
#   1. pnpm typecheck — DEVE passar com 0 erros
#   2. browser_navigate para a pagina no dev server
#   3. browser_snapshot — OLHAR o resultado
#   4. browser_take_screenshot — salvar como evidence
#   5. Testar dark mode: browser_evaluate("document.documentElement.classList.toggle('dark')")
#   6. Testar mobile: browser_resize(375, 812) → browser_snapshot
# Se a pagina esta quebrada, CONSERTAR antes de reportar.
# AUTOCRITICA: comparar com referencia (Nubank/Inter/PicPay). Parece premium? Se nao, refazer.

---

## 1. REAL PEOPLE & REFERENCES

### Josh W. Comeau — "CSS for JavaScript Developers" (joshwcomeau.com)
- **Layout Modes:** Understand the context (flow, flexbox, grid) before styling.
- **Container Queries:** Components adapt to parent size, not viewport. True modularity.
- Skeleton screens: shimmer > spinner. Match content shape. Animate in sync.
- "If you can't explain why a CSS property works, you don't understand it."
- Ref: [joshwcomeau.com/css/container-queries-unleashed](https://www.joshwcomeau.com/css/container-queries-unleashed/)

### Kent C. Dodds — "Epic React"
- **Colocation**: data, components, styles close to where they're used.
- **Composition > Configuration**: children + render props > boolean props.
- **Early returns**: guard clauses at top = readable components.

### Dan Abramov — React Core Team (overreacted.io)
- "Don't Stop the Data Flow": props change → component reacts.
- "Always Be Ready to Render": no side effects in render path.
- "No Component Is a Singleton": never assume single instance.

### Guillermo Rauch — Vercel CEO
- Next.js App Router philosophy: **Server Components by default**, client components opt-in.
- File-system routing: layout.tsx, loading.tsx, error.tsx, not-found.tsx at every segment.
- Streaming + Suspense: progressive rendering for perceived performance.

---

## 2. NEXT.JS PAGE PATTERNS (App Router)

### File Convention Per Route Segment
```
app/dashboard/
  layout.tsx      → Persistent shell (nav, sidebar). NOT re-rendered on child navigation.
  loading.tsx     → Skeleton shown during Suspense. ALWAYS present for streaming.
  error.tsx       → Error boundary. Catches render errors, shows fallback + retry.
  not-found.tsx   → 404 for this segment.
  page.tsx        → The actual page content.
```

### Layout Pattern (Pixio Standard)
```tsx
export default async function Page() {
  const data = await fetchData()
  return (
    <div className="space-y-6 px-4 pb-24 pt-6 md:px-8">
      <PageHeader title="..." subtitle="..." />
      <HeroChartCard data={data.summary} />
      <PanelsRow items={data.panels} />
      <Section title="Transacoes">
        <TransactionList items={data.transactions} />
      </Section>
    </div>
  )
}
```

### Error Boundaries (Granular)
- Errors bubble UP to nearest parent error.tsx. Place at different levels for granularity.
- Error boundaries catch RENDER errors, not event handler errors.
- ALWAYS provide retry button that calls `reset()` function.
- Log technical details to Sentry. Show human message in UI.

---

## 3. SKELETON LOADING — GOLD STANDARD

### Real Practices (from ironeko.com + react-loading-skeleton)
**DO:**
- Create Skeleton version per component: `Component.Skeleton = Skeleton` (dot notation export).
- Match real content structure (same dimensions, same typography sizes). Prevents CLS.
- Keep animations in sync — simultaneous data calls. If staggered, sync pulse/wave animation.
- Use `wave` animation (default). Smoother than `pulse` for multi-skeleton screens.
- Use `aria-live` regions for larger skeleton blocks → screen reader announces when content loads.

**DON'T:**
- Don't skeleton EVERYTHING — not every element needs it. Focus on primary content areas.
- Don't hardcode all dimensions — make widths responsive, only hardcode heights.
- Don't use spinners where skeleton fits. Skeleton > spinner ALWAYS (Doherty Threshold).

### Implementation (Next.js)
```tsx
// loading.tsx — automatic Suspense boundary
export default function Loading() {
  return (
    <div className="space-y-6 px-4 pb-24 pt-6 md:px-8 animate-pulse">
      <div className="h-8 w-48 rounded bg-white/5" />
      <div className="h-48 rounded-xl bg-white/5" />
      <div className="grid grid-cols-2 gap-4">
        <div className="h-24 rounded-xl bg-white/5" />
        <div className="h-24 rounded-xl bg-white/5" />
      </div>
    </div>
  )
}
```

Lib recommendation: `react-loading-skeleton` (shimmer out of box, responsive by default).

---

## 4. CORE WEB VITALS — PERFORMANCE RULES (2025-2026)

### Thresholds
| Metric | Good | Needs Work | Poor | What It Measures |
|--------|------|------------|------|-----------------|
| LCP | <2.5s | 2.5-4.0s | >4.0s | Largest visible content painted |
| INP | <200ms | 200-500ms | >500ms | Interaction responsiveness (replaced FID) |
| CLS | <0.1 | 0.1-0.25 | >0.25 | Visual stability / layout shifts |

### Business Impact
- Sites passing all 3: 24% lower bounce rates, better organic rankings.
- Google uses MOBILE scores for ranking (mobile-first indexing). Mobile = what counts.
- 43% of sites still fail INP threshold — biggest opportunity.

### LCP Fixes (Loading Performance)
- Image preloading (`<link rel="preload">`), critical CSS inlining, font preloading with `display: swap`.
- Server-side rendering (Next.js default). Streaming with Suspense for progressive paint.

### INP Fixes (Interactivity)
- Break long tasks (>50ms). Use `requestIdleCallback` or `scheduler.yield()`.
- Defer non-critical JavaScript. Minimize DOM complexity (<800 nodes ideal).
- Dynamic imports for heavy components: `const Chart = dynamic(() => import('./Chart'))`.

### CLS Fixes (Visual Stability)
- EVERY image/video/iframe needs explicit `width` + `height` attributes.
- `font-display: swap` + reserve space for dynamic content.
- NEVER inject content above existing content after initial render.

### Bundle Rules
- `<50 network requests` on mobile. Each adds latency.
- Tree-shakeable imports: `import { format } from 'date-fns'` not `import * from 'date-fns'`.
- Lazy load below-fold content. `loading="lazy"` on images.

Refs: [skyseodigital.com](https://skyseodigital.com/core-web-vitals-optimization-complete-guide-for-2026/), [digitalapplied.com](https://www.digitalapplied.com/blog/core-web-vitals-2026-inp-lcp-cls-optimization-guide)

---

## 5. RESPONSIVE DESIGN (2025-2026)

### Container Queries — The Game Changer
- Components adapt to PARENT size, not viewport. Truly modular, layout-agnostic components.
- `container-type: inline-size` on layout wrappers. Avoid unintended fragmentation.
- 41% adoption in 2025 State of CSS survey. Browser support: all modern browsers.
- Move responsive logic INTO the component — more portable, robust, testable.
- Ref: [joshwcomeau.com](https://www.joshwcomeau.com/css/container-queries-unleashed/)

### Mobile-First Breakpoints (BYMAV Standard)
| Name | Min-width | Columns | Padding | Target |
|------|-----------|---------|---------|--------|
| xs | 0 | 1 | 16px | Small phones (320px) |
| sm | 375px | 1 | 16px | Standard phones |
| md | 768px | 2 | 24px | Tablets |
| lg | 1024px | 3 | 32px | Small laptops |
| xl | 1280px | 3-4 | 32px | Desktops |

**Rule:** Design xs FIRST. Add complexity going up. NEVER desktop-first.

### Modern CSS Techniques
- **CSS Grid + Flexbox + Container Queries** = complete layout toolkit.
- **Range media queries**: `@media (375px <= width <= 768px)` — cleaner syntax.
- **CSS Nesting**: reduce selector repetition. Native in all modern browsers.
- **View Transitions API**: smooth page transitions without JS animation libraries.

---

## 6. STRIPE DASHBOARD — PAGE ARCHITECTURE REFERENCE

- **ContextView:** Apps render next to Stripe content in drawer. Side-by-side context sharing.
- **Layout components** for structure. **Navigation components** for wayfinding. **Content components** for info.
- **Detail pages:** Focused view of one object (payment, customer, invoice).
- **Consistent look-and-feel** across entire Dashboard. UI components enforce this.
- Ref: [docs.stripe.com/stripe-apps/patterns](https://docs.stripe.com/stripe-apps/patterns)

---

## 7. CHECKLIST FINAL

- [ ] `pnpm typecheck` passes
- [ ] 4 states: loading (skeleton), empty, error, success
- [ ] Mobile responsive (375px first)
- [ ] Colors from design tokens (zero hardcoded)
- [ ] Components reused from STYLE
- [ ] t('key') exists in src/i18n/translations.json
- [ ] Feature flag if new feature
- [ ] Branch + PR (never push main directly)
- [ ] Core Web Vitals: LCP <2.5s, INP <200ms, CLS <0.1
- [ ] Container queries where component needs layout independence

---

## 8. TEST ACCOUNTS + SCREENSHOT RULES (OBRIGATORIO)

### CONTAS DE TESTE — USAR SEMPRE para screenshots
**VITA (vita-ai.cloud):**
- QA: `qa@vita-ai.cloud` / `VitaTest2026` (residencia yr6, ENARE, dados seedados)
- Atlas: `atlas@vita-ai.cloud` / `AtlasTest2026` (graduacao yr3)

**PIXIO (pixio.cloud):**
- QA: `test@pixio.cloud` / `Test123!` (pro, dados completos)
- E2E: `teste@pixio.cloud` / `Teste123!`

### REGRA HARD: LOGAR ANTES DE QUALQUER SCREENSHOT
# NUNCA mandar screenshot de tela de login/splash/onboarding.
# SEMPRE: 1. clawdbot-browser navigate → login → preencher email+senha → clicar entrar
#          2. Esperar dashboard/tela carregar
#          3. SO ENTAO browser_screenshot
# Se screenshot mostra login screen = INVALIDO. REFAZER.
# clawdbot-browser login:
#   browser_navigate("https://vita-ai.cloud/login")
#   browser_fill("input[type=email]", "qa@vita-ai.cloud")
#   browser_fill("input[type=password]", "VitaTest2026")
#   browser_click("button:has-text('Entrar')")
#   browser_screenshot()


## Licoes Aprendidas (auto-feedback)
- [2026-03-21] [token_burn] Gastou 1602k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 505k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 10376k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 7817k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 18037k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 3029k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 2094k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 4847k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 6408k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 3610k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 1555k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 6541k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 5594k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 4514k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [exploration_no_output] Leitura excessiva sem editar = tunnel vision de exploração. Max 5 reads antes de produzir algo. Faca plano curto e execute.
- [2026-03-12] [edit_without_read] SEMPRE ler o arquivo antes de editar. Edit sem Read = edit cego = erros evitaveis.
- [2026-03-12] [retry_loop] Comando "pnpm typecheck" falhou multiplas vezes. 2-strike rule: se falhou 2x, mudar abordagem.
- [2026-03-12] [infra_failure] Typecheck/build com erro de package manager (bun/yarn/pnpm mismatch) eh problema de INFRA, nao de codigo. NAO tente corrigir — skip e continue.
