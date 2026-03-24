# DRAFT BRAIN — Mockup & Prototyping Knowledge Base
# Role: Mockup/prototype creator at BYMAV. Creates HTML/CSS mockups. Reports to LEO.
# This file: KNOWLEDGE. Prototyping methods, fidelity decisions, AI tools, real references.
# Max 200 lines. Dense. No fluff. REAL references from REAL people.

---

## 1. REAL PEOPLE & REFERENCES

### Jake Knapp — "Sprint" (Google Ventures Design Sprint)
- **5-day framework:** Monday(map) → Tuesday(sketch) → Wednesday(decide) → Thursday(prototype) → Friday(test).
- **Monday:** Start at the end. Agree on long-term goal. Map the challenge. Pick target for the week.
- **Tuesday:** 4-step sketch process emphasizing CRITICAL THINKING over artistry. Everyone sketches independently.
- **Wednesday:** "Sticky Decision" 5-step method to pick best solutions. Decider has final call.
- **Thursday:** "Fake it till you make it." Build realistic FACADE in one day. Customer-facing surface only.
- **Friday:** 5 customers, 5 separate 1:1 interviews. Quick-and-dirty answers to pressing questions.
- **Key insight:** 5 tests surface 85% of usability issues. Don't wait for launch — test NOW.
- Refs: [thesprintbook.com](https://www.thesprintbook.com/the-design-sprint), [gv.com/sprint](https://www.gv.com/sprint/)

### Nielsen Norman Group — Prototype Fidelity Framework
- **Three dimensions of fidelity** (not just "low" vs "high"):
  - **Interactivity:** Non-functional static → Fully clickable with real responses
  - **Visuals:** Black-and-white sketches → Production-quality graphics
  - **Content:** Placeholder "lorem ipsum" → Real articles, real data
- **Low-fi benefits:** Faster creation, easy mid-test changes, less designer attachment, stakeholders expect revisions.
- **High-fi benefits:** Realistic behavior, users act naturally, tests complex workflows, minimizes testing errors.
- **Decision rule:** Use low-fi for information architecture + user flows (early). Use high-fi for usability + stakeholder buy-in (late).
- **Static testing methods:** Wizard of Oz (operator controls screen), Paper Prototype, Steal-the-Mouse.
- Ref: [nngroup.com/articles/ux-prototype-hi-lo-fidelity](https://www.nngroup.com/articles/ux-prototype-hi-lo-fidelity/)

### Steve Schoger — "Refactoring UI"
- **Start with lots of whitespace, then reduce.** Not the reverse.
- **Shadows > borders:** elevation with box-shadow, not 1px border.
- **Hierarchy by weight, not size:** font-weight 600/700 + color for emphasis.
- **Limit colors:** max 2 primary + neutrals. Border radius: 1-2 values max.
- **Never center long text.** Left-align >2 lines.

### Erik Kennedy — "Learn UI Design"
- **Overlap elements** for depth. **Consistent border-radius.** **One strong idea per screen.**

---

## 2. FIDELITY DECISION MATRIX

| Situation | Fidelity | Why | Time |
|-----------|----------|-----|------|
| Exploring IA / user flows | Low (wireframe) | Test structure before polish | 1-2 hours |
| Aligning with Rafael on direction | Low-Mid (layout mockup) | Speed + cheap iteration | 2-4 hours |
| Testing with users | Mid-High (clickable) | Realistic behavior for valid data | 4-8 hours |
| Stakeholder presentation | High (polished HTML) | Confidence + buy-in | 8-16 hours |
| Handoff to CRAFT | High (production-ready) | Minimize interpretation gaps | 8-16 hours |
| Quick validation of 2-3 options | Low (sketches) | Speed wins, compare fast | 30 min |

### Rules
- NEVER start at high fidelity unless direction is already validated.
- ALWAYS do low-fi first for new features. High-fi = iteration on validated concepts.
- If more than 2 major changes after high-fi review → WRONG APPROACH. Should have tested low-fi.

---

## 3. AI-POWERED PROTOTYPING TOOLS (2025-2026)

### v0 by Vercel — Component Prototyping
- **Best for:** React component generation, design system work, Tailwind/shadcn/ui components.
- **Strengths:** Image-to-code (Figma → code), production-grade React, Framer Motion animations.
- **Weaknesses:** React-dependent, inconsistent with complex patterns, backend still developing.
- **BYMAV use:** Quick component mockups, exploring UI variants, generating starting points.

### Bolt.new by StackBlitz — Full-Stack MVP
- **Best for:** Full-stack prototypes, hackathon demos, quick deployable MVPs.
- **Strengths:** Browser-based IDE (no local setup), framework-agnostic, file locking for selective AI edits.
- **Weaknesses:** Browser performance limits with large projects, security considerations.
- **Stats:** $40M ARR in 4.5 months. Screenshots → full UIs instantly.

### Lovable.dev — Guided Full-Stack
- **Best for:** Team collaboration, apps with backend requirements (native Supabase integration).
- **Strengths:** GitHub-first workflow, "Select Element" UI iteration, Custom Knowledge Base.
- **Weaknesses:** Opinionated architecture, token-based limits, smaller community.
- **Stats:** $20M ARR in 2 months. Claims 20x faster development. 12-min MVP verified.

### The 70% Problem
- ALL AI tools hit a complexity threshold where local development is required.
- Use AI tools for 0→70%. Use manual coding for 70→100%. NEVER expect full production from AI alone.

Ref: [addyo.substack.com](https://addyo.substack.com/p/ai-driven-prototyping-v0-bolt-and)

---

## 4. WIREFRAMING CONVENTIONS

### Information Architecture Standards
- **Hierarchy types:** Hierarchical (most common), Sequential (checkout flows), Matrix (filtering).
- **Labeling:** Follow industry conventions. "Adicionar ao carrinho" not "Colocar na sacola". Familiarity > creativity.
- **Navigation patterns:** Bottom tabs (mobile, max 5), sidebar (desktop), breadcrumbs (deep hierarchy).
- **Content priority:** Larger placeholders = key elements, smaller = secondary. F-pattern for reading flow.

### Wireframe Rules
- Use SAME symbol for all buttons, DIFFERENT symbol for all images. Visual language must be consistent.
- Name every element. "Frame 47" = unacceptable. "hero-section/balance-card" = correct.
- Annotate decisions: WHY is this element here? What user need does it serve?
- Ref: [balsamiq.com/learn/articles/ten-principles-effective-wireframes](https://balsamiq.com/learn/articles/ten-principles-effective-wireframes/)

---

## 5. FIGMA PROTOTYPING (2025-2026)

### Core Best Practices
- **Components + Variants** for reusable interactive elements. States (hover, click, disabled) as variants, not separate frames.
- **Auto Layout** extensively — dynamic resizing that responds to content. No absolute positioning except overlays.
- **Timed delays + overlays** for realistic interactions.
- **Observation Mode** during user testing — watch without interfering.
- **Name layers properly.** Auto Layout + named layers = dev handoff ready.

### Figma Make (2025)
- AI-powered: natural language → working prototype. Embeddable in Design, FigJam, Slides.
- Turns product ideas + user flows + logic into explorable prototypes.
- Shifts Figma from pixel-pushing to logic definition.
- Ref: [figma.com/best-practices](https://www.figma.com/best-practices/)

---

## 6. DRAFT WORKFLOW (Gold Standard)

### 1. Research (5min)
- Check `/knowledge/references/` — existing reference?
- If not: pixio-scraper xray on reference site (Nubank, Inter, Revolut, Linear, Stripe).
- Extract: layout grid, visual hierarchy, states (loading/empty/error/success).

### 2. Template (never from zero)
- ALWAYS start from `/knowledge/templates/`. If none exists → create generic first, then specialize.
- Dark mode as default: background #08080C, card #0E0E14.

### 3. Build
- CSS variables ALWAYS — zero hex hardcoded in body.
- Realistic BR data — Brazilian names, R$, formatted CPF. NEVER "lorem ipsum."
- Mobile-first. Test at 375px. Responsive.
- 4 states per section: Loading, Empty, Error, Success.

### 4. Verify
- `grep -E '#[0-9a-fA-F]{3,8}'` in HTML → ZERO matches outside CSS vars.
- Screenshot via clawdbot-browser — MANDATORY.
- If reference exists → side-by-side comparison, report similarity %.

---

## 7. ANTI-PATTERNS (Real Rejections)

| Anti-pattern | Problem | Solution |
|-------------|---------|---------|
| Letter avatars (AT, JV) | Amateur | SVG gradients or real icons |
| Lorem ipsum | Unrealistic | Real BR data |
| Colorful generic icons | Childish | Monochrome real icons |
| White background, no variation | Too flat | Elevation layers |
| Centered text blocks | Hard to read | Left-align |
| Too many primary colors | Confusing | Max 2 + neutrals |
| Starting high-fi without direction | Wasted effort | Wireframe → validate → polish |
| Skipping user testing | Assumptions | 5 users find 85% of issues (Krug) |

---

## 8. TOOLS

| Tool | Use | When |
|------|-----|------|
| clawdbot-browser | Screenshot + preview | ALWAYS (mandatory) |
| pixio-scraper xray | Extract tokens from reference sites | Research phase |
| v0 by Vercel | Quick component mockups | Exploring variants |
| Figma | Wireframes, clickable prototypes | Team review, user testing |
| Google Fonts CDN | Typography | Only allowed external CDN |
| Simple Icons | Brand icons | cdn.simpleicons.org |

---

## 9. USER TESTING WITH PROTOTYPES

### Jake Knapp's Rules
- **5 interviews, 1:1, same day.** Patterns emerge by interview 3.
- **Prototype = hypothesis.** Test whether CONCEPT works, not pixel perfection.
- **Facilitator minimal intervention.** If something breaks: "That isn't working. What were you expecting?"
- **Pilot test first.** Wrong pages during test damage mental models and invalidate data.

### NNG Decision Framework
- **Early stage:** Low-fi + few features → validate concept.
- **Mid stage:** Mid-fi + key flows → validate navigation + IA.
- **Late stage:** High-fi + all states → validate usability + edge cases.
- **NEVER test only happy path.** Error states, empty states, edge cases reveal real issues.
