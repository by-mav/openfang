# LEO BRAIN — Design Director Knowledge Base
# Role: Design Director at BYMAV. Gate-keeper of visual quality. Specs, not code.
# Prereqs: design-dna.md (tokens/templates), design-rejections.md (mistakes), component-cookbook.md (implementation)
# This file: KNOWLEDGE. Principles, experts, methodologies, review protocols.
# Max 200 lines. Dense. No fluff. REAL references from REAL people.

---

## 1. GOLD STANDARD REFERENCES (Real People, Real Principles)

### Katie Dill — Head of Design at Stripe (ex-Airbnb, ex-Lyft)
- **MVQP over MVP:** "Minimum Viable Quality Product — solves the user problem completely, with refinement that builds trust."
- **Quality = Utility + Usability + Beauty.** Beauty drives conversion: Stripe improved email typography+layout → +20% conversion.
- **"15 Essential Journeys"**: Stripe maps top 15 user flows. Each quarter teams friction-log them, score on 5-point scale (red→green).
- **Friction Logs:** "Walk the Store" — PMs/designers/engineers USE the product and document every pain point. Not theoretical.
- **"Minimize Presentationing":** Review products directly, not decks. Decks bias executives with narrative, not reality.
- **Hiring:** Craft mastery + systems thinking + intellectual curiosity + humility with conviction.
- Refs: [creatoreconomy.so](https://creatoreconomy.so/p/how-stripe-crafts-quality-products-katie-dill), [Lenny's Newsletter](https://www.lennysnewsletter.com/p/building-beautiful-products-with)

### Karri Saarinen — CEO/Designer at Linear (ex-Airbnb Design Systems)
- **"Quality is our first principle. Every other metric and decision flows from that."**
- **Opinionated > flexible:** "You can't build the optimal tool if it's endlessly customizable."
- **Speed as feature:** Performance complaints about incumbents → speed baked into architecture, not afterthought.
- **Intuition > A/B tests:** "We don't make decisions based on data. Intuition is trained, not magical."
- **Design for someone, not everyone.** Linear designed for high-ICs at tech companies. Niche focus → product-market fit.
- **AI age:** Move beyond chat-only interfaces. "Workbench model" — AI as tool within structured workflows, not replacement.
- Refs: [Figma Blog 10 Rules](https://www.figma.com/blog/karri-saarinens-10-rules-for-crafting-products-that-stand-out/), [First Round Review](https://review.firstround.com/linears-path-to-product-market-fit/)

### Steve Krug — "Don't Make Me Think" (2000, 3rd ed. 2014)
- **First law:** If something requires thought, redesign it. Self-evident > clever.
- **Satisficing:** Users pick the FIRST reasonable option, not the best. Make primary actions obvious.
- **Trunk test:** Any page: What site? What page? Major sections? Where am I? How to search?

### Luke Wroblewski — "Mobile First" (2011)
- **Design for constraints first.** Mobile forces prioritization. Desktop expands, not the reverse.
- **One thumb, one eyeball:** Primary actions in thumb zone (bottom 1/3). Critical info above fold.
- **Touch targets:** 44pt (Apple HIG), 48dp (Material). Never compromise.

### Adam Wathan & Steve Schoger — "Refactoring UI" (2018)
- **Start with feature, not layout.** Spacing scale (4/8/12/16/24/32/48/64). Never arbitrary values.
- **Typography hierarchy through weight+size+color, not font variety.** Max 2 font families.
- **Don't use labels as crutch.** If context clear, data speaks. Empty states = design opportunities.

### Jon Yablonski — "Laws of UX" (lawsofux.com)
- Fitts (size+distance), Hick (fewer choices), Jakob (follow conventions), Miller (7+-2 chunks)
- **Doherty Threshold:** Response <400ms or users lose flow. Skeleton > spinner.
- **Aesthetic-Usability Effect:** Beautiful interfaces FEEL easier to use. Polish IS function.

### Dieter Rams — 10 Principles (Braun/Vitsoe)
- Principle 10 supreme for LEO: **"As little design as possible."** Remove until it breaks.

---

## 2. PLATFORM DESIGN STANDARDS

### Apple HIG (2025 — Liquid Glass era)
- Core: Clarity, Consistency, Deference, Depth. UI shouldn't distract from content.
- 2025 "Liquid Glass": translucency, depth, fluid responsiveness. Unified HIG across platforms.
- Ref: [developer.apple.com/design](https://developer.apple.com/design/human-interface-guidelines/)

### Material Design 3 (Google, 2025)
- Dynamic Color: entire scheme adapts to user wallpaper. Algorithmic personalization.
- Token system: color, type, shape, spacing, motion, elevation — ALL tokenized.
- Adaptive layouts via Window Size Classes + Canonical Layouts (compact→expanded).
- Ref: [m3.material.io](https://m3.material.io/foundations/design-tokens)

### Nubank Design System (NuDS) — Brazilian gold standard
- 100+ reusable components, server-driven UI (80% of screens), Figma-native compliance plugin.
- Multi-market: PT/ES/EN toggling via Figma variables without duplication.
- NuDS Foundations: standardized tokens, themes, modes, analytics. Designers embedded per market.
- Ref: [Figma customers/nubank](https://www.figma.com/customers/nubank-design-system-accessible-experiences-with-figma/)

---

## 3. FINTECH DESIGN STANDARDS

### Brazilian Fintech DNA (Nubank, Inter, C6, PicPay)
| Pattern | Nubank | Inter | C6 | PicPay |
|---------|--------|-------|----|--------|
| Primary color | Purple #820AD1 | Orange #FF7A00 | Black | Green #21C25E |
| Card style | Rounded 16px, soft shadow | Rounded 12px, flat | Sharp 8px | Rounded 16px, gradient |
| Navigation | Bottom tabs (5) | Bottom tabs (5) | Bottom tabs (4) | Bottom tabs (5) |
| Number font | Monospace/tabular | System tabular-nums | Custom mono | System tabular-nums |

### Trust Indicators
- Consistent layout every screen. Lock icon near sensitive data. Monospace for money values.
- R$ 1.234,56 ALWAYS (dot=thousands, comma=decimal). NEVER truncate. tabular-nums + right-align.
- Confirmation steps: review → confirm → success. NEVER one-tap for money movement.

### States (EVERY screen MUST define all 4)
1. **Loading:** Skeleton pulse matching layout shape. NEVER spinner alone. NEVER blank.
2. **Empty:** Illustration/icon + message + primary CTA.
3. **Error:** Icon + message + retry. Distinguish retryable vs permanent.
4. **Success:** Actual content/data.

### Dark Mode Fintech
- Background #08080C (near-black, not gray). Cards #0E0E14. Border rgba(255,255,255,0.06).
- NEVER pure #FFFFFF text on dark — use #F5F5F5. Primary text min 15.3:1 contrast.

---

## 4. REVIEW PROTOCOL

### Katie Dill's Framework Applied to LEO
- **Essential Journeys:** Score each user flow quarterly. Red→green. Track over time.
- **Friction Logs:** Experience the product as user. Document every friction point. Not hypothetical.
- **Direct Review:** Look at the product, not the presentation. No bias from persuasive decks.

### LEO Review Checklist (EVERY spec/mockup)
**Visual Hierarchy (5):** Primary CTA dominates | Info hierarchy (size+weight+color) | F/Z-pattern flow | Largest element = most important | Secondary actions subordinate
**Spacing (4):** 4px grid consistent | Gestalt proximity | Cards 16-20px padding | Sections 12-16px gap
**Accessibility (5):** WCAG AA 4.5:1 normal / 3:1 large | 44px touch targets | Color not only indicator | Focus states | aria-labels
**Fintech (4):** Money tabular-nums + right-aligned | All 4 states | Show/hide sensitive data | Confirmation before financial action
**Mobile-First (3):** Designed at 375px first | Primary actions in thumb zone | No hover-dependent interactions

### Anti-Patterns to Flag
| Anti-Pattern | Fix |
|-------------|-----|
| Centered paragraph >3 lines | Left-align body text |
| Gray on gray | Increase contrast ratio |
| Icon-only buttons | Add text label |
| Horizontal scroll on mobile | Stack vertically |
| Modal inside modal | Flatten or new page |
| >5 nav items | Group under categories |
| Carousel for important content | Show all or tabs |
| Placeholder as label | Floating label above input |

---

## 5. TOOLS & DESIGN SYSTEMS

| System | Learn | URL |
|--------|-------|-----|
| Radix UI | Accessible primitives | radix-ui.com |
| Shadcn/ui | Tailwind + copy-paste | ui.shadcn.com |
| Material 3 | Dynamic color, adaptive | m3.material.io |
| Apple HIG | Platform conventions | developer.apple.com/design |
| Polaris (Shopify) | Commerce UI, Web Components | polaris.shopify.com |
| Carbon (IBM) | Data-dense, accessibility-first | carbondesignsystem.com |

### LEO Decision Framework (in order)
1. Does it solve the user's problem? (Pretty but useless = fail)
2. Can user figure it out without instructions? (Krug test)
3. Does it follow platform conventions? (Jakob's Law)
4. Is visual hierarchy clear in 3 seconds? (Squint test)
5. Would Nubank/Linear ship this? (Gold standard benchmark)
6. All 4 states defined? (Loading, empty, error, success)
7. Is it accessible? (Contrast, touch targets, screen readers)

**If ANY answer is NO, the spec goes back. No exceptions.**

### TEST ACCOUNTS — ENFORCE EM CRAFT/STYLE
**VITA:** qa@vita-ai.cloud / VitaTest2026 | atlas@vita-ai.cloud / AtlasTest2026
**PIXIO:** test@pixio.cloud / Test123!
**REGRA:** Screenshot/preview = LOGADO com dados reais. Login screen = REJEITAR.
Detalhes: memory/test-accounts.md


## Licoes Aprendidas (auto-feedback)
- [2026-03-21] [token_burn] Gastou 198k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [edit_without_read] SEMPRE ler o arquivo antes de editar. Edit sem Read = edit cego = erros evitaveis.
