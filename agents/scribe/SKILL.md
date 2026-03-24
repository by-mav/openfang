# SCRIBE BRAIN — Content & SEO Knowledge Base
# Role: Content specialist at BYMAV. Blog posts, landing pages, SEO, technical writing.
# Prereqs: Growth strategy context, BYMAV brand guidelines
# This file: KNOWLEDGE. Experts, frameworks, SEO methodology, content patterns.
# Max 200 lines. Dense. No fluff.

---

## 1. GOLD STANDARD REFERENCES

### Divio Documentation System (documentation.divio.com)
- **Four types of documentation:** Each serves a different purpose. NEVER mix them.
  - **Tutorials:** Learning-oriented. "Follow these steps to build X." Hands-on, complete.
  - **How-to guides:** Task-oriented. "How to do X." Assume knowledge, solve specific problem.
  - **Reference:** Information-oriented. API docs, schemas, configs. Complete, accurate, dry.
  - **Explanation:** Understanding-oriented. "Why X works this way." Conceptual, discursive.
- **Key insight:** Most docs fail because they mix types. A tutorial that stops to explain theory loses the reader.

### Google E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness)
- **Experience:** Show first-hand experience. "We tested 5 budgeting apps for 3 months."
- **Expertise:** Author credentials. Financial content = qualified author or reviewed by expert.
- **Authoritativeness:** Citations, backlinks, brand recognition. Link to BCB, CVM, official sources.
- **Trustworthiness:** Accurate data, transparent methodology, clear disclaimers.
- **YMYL (Your Money or Your Life):** Pixio content = financial. Google holds to HIGHEST standard.
- **Practical:** Every blog post needs author bio, sources cited, last-updated date.

### Ann Handley — "Everybody Writes" (2014, 2nd ed 2022)
- **"Good writing is good thinking made visible."** If the article is confused, the thinking is confused.
- **Write for one reader.** Picture a specific person. For Pixio: 25-35 year old Brazilian, first budget app.
- **12-word rule:** If you can't summarize in 12 words, you don't understand it well enough.
- **Draft ugly, edit ruthlessly.** First draft = get ideas out. Second pass = cut 30%. Third = polish.

### Ahrefs Team — Tim Soulo, Joshua Hardwick (ahrefs.com/blog)
- **Topic clusters > random posts.** Pillar page + supporting posts. Internal linking strategy.
- **Search intent > keyword volume.** Understanding WHAT the user wants > how many search for it.
- **4 types of search intent:** Informational, navigational, transactional, commercial investigation.
- **Content freshness:** Update posts every 6 months. Google rewards recently updated content.
- **10x content:** Don't write another "how to save money" article. Write THE BEST one.

### Nubank Blog (blog.nubank.com.br) — Benchmark for Brazilian Fintech Content
- **Plain language for complex topics.** "Investir" explained like a conversation, not a textbook.
- **Visuals with every post.** Custom illustrations, not stock photos. Brand consistency.
- **SEO + brand voice.** They rank AND sound like Nubank. Not keyword-stuffed robot text.
- **Regulatory compliance.** Financial disclaimers where needed. Links to official BCB sources.

---

## 2. SEO METHODOLOGY

### Content Brief Framework (Before Writing ANYTHING)
```markdown
## CONTENT BRIEF — [Title]
**Target keyword:** [primary keyword]
**Search volume:** [monthly searches]
**Search intent:** [informational/transactional/commercial]
**Target audience:** [who, pain point, knowledge level]
**Competing content:** [top 3 ranking URLs + what they cover]
**Angle:** [what makes OUR version different/better]
**Word count:** [target based on competitors + intent]
**CTA:** [what action we want reader to take]
**Internal links:** [3+ existing pages to link to]
**Sources:** [official references to cite]
```

### On-Page SEO Checklist
- [ ] **H1:** One per page. Contains primary keyword. Under 60 characters.
- [ ] **Meta title:** Primary keyword + brand. `[Topic] — Pixio`. Under 60 chars.
- [ ] **Meta description:** Value prop + keyword. Under 160 chars. Compels click.
- [ ] **URL slug:** Short, descriptive, keyword-rich. `/como-fazer-orcamento-pessoal`
- [ ] **H2/H3 structure:** Logical hierarchy. Include LSI keywords naturally.
- [ ] **First paragraph:** Keyword in first 100 words. Hook the reader immediately.
- [ ] **Images:** Alt text with keyword variation. Compressed (<100KB). WebP format.
- [ ] **Internal links:** Minimum 3 to related content. Natural anchor text.
- [ ] **External links:** Cite authoritative sources (BCB, CVM, IBGE, etc.).
- [ ] **Schema markup:** Article, FAQ, HowTo where appropriate.
- [ ] **Mobile-friendly:** Paragraphs <3 lines on mobile. Short sentences.

### Topic Cluster Strategy
```
PILLAR: "Educação Financeira" (broad, 5000+ words, comprehensive)
  ├─ "Como fazer orçamento pessoal" (how-to guide)
  ├─ "O que é CDI e como funciona" (explanation)
  ├─ "Melhores apps de controle financeiro 2026" (commercial)
  ├─ "Como sair das dívidas" (how-to guide)
  ├─ "Reserva de emergência: quanto guardar" (guide)
  └─ "Investir com pouco dinheiro" (tutorial)
All supporting posts link to pillar. Pillar links to all.
```

---

## 3. CONTENT TYPES & TEMPLATES

### Blog Post Template
```markdown
---
title: "[H1 with keyword]"
description: "[Meta description, <160 chars]"
author: "Pixio"
date: "2026-MM-DD"
updated: "2026-MM-DD"
category: "[educação financeira/investimentos/dicas]"
tags: ["tag1", "tag2", "tag3"]
---

## [H2 - Hook / Context]
[1-2 paragraphs establishing the problem. Stats from official sources.]

## [H2 - Core Content]
[Main body. Numbered lists for steps. Tables for comparisons.]

### [H3 - Sub-section]
[Deeper detail on a sub-topic.]

## [H2 - Practical Application]
[How to use this information. Pixio-specific examples.]

## [H2 - Conclusion + CTA]
[Summary in 2-3 sentences. Link to Pixio feature that helps.]

---
*Fontes: [BCB](url), [CVM](url). Atualizado em [date].*
*Este conteúdo é informativo e não constitui recomendação de investimento.*
```

### Landing Page Template
```markdown
# [Hero headline with value prop]
## [Sub-headline explaining how]

[FEATURE 1] → [BENEFIT in user's words]
[FEATURE 2] → [BENEFIT in user's words]
[FEATURE 3] → [BENEFIT in user's words]

[Social proof: "X mil pessoas já usam"]
[CTA: "Comece grátis" / "Baixe o app"]

[FAQ section for SEO + objection handling]
```

---

## 4. WRITING STANDARDS

### Voice & Tone (Pixio Brand)
| Attribute | DO | DON'T |
|-----------|-----|-------|
| **Approachable** | "Vamos simplificar isso" | "Conforme legislação vigente..." |
| **Confident** | "Você pode economizar R$ 500/mês" | "Talvez seja possível..." |
| **Empowering** | "Aqui está o que fazer" | "Consulte um especialista" (cop-out) |
| **Brazilian** | pt-BR, informal tu/você | Formal "Vossa Senhoria" |
| **Data-backed** | "67% dos brasileiros..." (source) | "Muitas pessoas dizem..." |

### Readability Rules
- **Paragraphs:** Max 3-4 lines on mobile. Break long blocks.
- **Sentences:** Max 25 words average. Mix short and medium.
- **Vocabulary:** 8th grade reading level. "Rendimento" not "rentabilidade composta".
- **Lists:** Use bullets/numbers for 3+ items. Never bury a list in a paragraph.
- **Bold:** Key terms, numbers, takeaways. Don't bold entire sentences.

### Financial Content Compliance
- **Disclaimer required:** "Este conteúdo é informativo e não constitui recomendação de investimento."
- **Sources required:** Every claim about rates, returns, regulations must cite official source.
- **No guarantees:** NEVER "você VAI ganhar X%". Always "historicamente" or "em média".
- **BCB data:** Interest rates, SELIC, inflation from BCB API. Never stale numbers.
- **CVM compliance:** Investment content must note CVM guidelines where applicable.

---

## 5. ANTI-PATTERNS

| Anti-Pattern | Impact | Fix |
|-------------|--------|-----|
| Keyword stuffing | Google penalty, reads like spam | Natural density 1-2%, LSI keywords |
| No internal links | Orphaned pages, poor SEO | Min 3 internal links per post |
| Stock photo hero | Generic, no brand identity | Custom illustrations or branded graphics |
| Wall of text | Mobile users bounce at 100% | Short paragraphs, lists, visuals |
| No author/date | Fails E-E-A-T, no trust | Author bio + published/updated date |
| Copying competitors | Duplicate content penalty | Original angle, unique data/perspective |
| No CTA | Content without conversion | Clear CTA: download app, sign up, read more |
| Stale content | Rankings decay, inaccurate info | Update every 6 months minimum |

---

## 6. SCRIBE DECISION FRAMEWORK

Before publishing any content:
1. **Content brief done?** Keyword, intent, audience, angle, competitors. Research FIRST.
2. **Right type?** Tutorial vs how-to vs reference vs explanation. Don't mix.
3. **SEO checklist complete?** H1, meta, URL, internal links, schema, images.
4. **E-E-A-T signals present?** Author, sources, last-updated date, disclaimers.
5. **Voice on brand?** Reads like Pixio, not like a textbook or chatbot.
6. **Mobile-friendly?** Short paragraphs, scannable, no horizontal scroll.
7. **Financial compliance?** Disclaimer, cited sources, no guarantees.
8. **Better than what's ranking?** If not, why would Google rank it?
