# SAGE BRAIN — AI Director Knowledge Base (Gold Standard)
# Role: AI Director at BYMAV. LLM integration, RAG, prompt engineering, Pixio AI assistant.
# Updated: 2026-02-28 with REAL research, references, and industry practices.
# Max 200 lines. Dense. No fluff. Every line backed by real source.

---

## 1. REAL PEOPLE & REFERENCES

### Andrej Karpathy — ex-Tesla AI, OpenAI Founding Member
- Prompt engineering IS engineering. Tokenization matters (BPE). Eval sets FIRST.
- Fine-tuning vs prompting: try prompting + few-shot first. Fine-tune only when plateaus.

### Chip Huyen — "Designing ML Systems" (O'Reilly)
- Model is 10% of work. Monitor input/output distribution shift. Feature stores. Feedback loops.

### Harrison Chase — LangChain Creator
- Chain of thought > single prompt. Tool use > computation. Memory: short-term (buffer) vs long-term (vector).

### Simon Willison — LLM Pragmatist (simonwillison.net)
- "LLMs are compression of the internet." Structured output (JSON + Zod). Prompt injection is real. Cache everything.

### Cleo AI (meetcleo.com) — REAL Financial AI Reference
- Cleo 3.0: LLM agent plans multi-step interactions, invokes tools, reasons over financial+emotional context.
- Smart Insights Agent (OpenAI o3): reviews 6 months of transactions, surfaces actionable insights WITHOUT prompting.
- Conversational memory: each conversation builds on the last. Captures goals, stress signals, financial anxiety.
- Memory is personal: remembers WHY user set goals, adjusts tone/timing/content based on reactions.
- Fully agentic: autonomously selects tools and reasons through each step. No rigid flows.

### OpenAI — Agent Building (2025-2026)
- Responses API replaces Assistants API (deprecated Aug 2026). Agent-native APIs + reasoning models.
- Agents SDK + AgentKit for multi-step workflows. Think of agents like employees with job descriptions.
- Triage agent (receptionist) routes to specialist agents (domain experts). Modular.

### Anthropic — Constitutional AI
- HHH: Helpful, Harmless, Honest. Constitutional AI = ethical principles > human feedback alone.
- Claude crisis response: 98.6-99.3% accuracy with near-zero false positives (2025 System Card).
- Priority: broadly safe > broadly ethical > compliant with guidelines > genuinely helpful.

---

## 2. RAG ARCHITECTURE (Real Patterns 2025-2026)

### RAG Variants (from arxiv.org/abs/2501.07391 + EdenAI 2025 Guide)
| Pattern | Strategy | Best For |
|---------|----------|----------|
| **Traditional** | Chunk → Embed → Retrieve → Generate | Static knowledge bases |
| **Self-RAG** | Reflection tokens (ISREL/ISSUP) for self-critique | High-stakes accuracy |
| **Corrective RAG** | Retrieval evaluator + confidence scores, web fallback | Dynamic/unreliable sources |
| **Adaptive RAG** | Classify query complexity → adjust strategy | Cost optimization |
| **Graph RAG** | Knowledge graphs + vector search, up to 99% precision | Entity relationships |
| **Long RAG** | Larger coherent sections instead of tiny chunks | Context preservation |
| **Golden-Retriever** | Jargon dictionary before retrieval | Financial domain terms |

### Pixio RAG Pipeline (Production)
1. **Chunk:** 500 tokens, 50-token overlap. Financial docs need LARGER chunks (preserve context).
2. **Embed:** text-embedding-3-small (1536d). Supabase pgvector with HNSW index.
3. **Hybrid search:** BM25 (lexical) + dense vectors. Double-digit relevance gains (Microsoft 2025).
4. **Top-k=5**, cosine similarity > 0.7. Filter by content type (education/transaction/article).
5. **Metadata blocks:** structured source lists for interpretability. Clear source links.
6. **Evaluate:** Separate retrieval quality, groundedness, and answer quality (Microsoft RAG Evaluators).

### When RAG vs Direct Query
| "Quanto gastei?" | Direct DB → no LLM needed |
| "Estou gastando muito?" | DB + LLM analysis |
| "O que é CDI?" | RAG from knowledge base |
| "Como economizar?" | RAG + user data personalization |

---

## 3. FINANCIAL AI GUARDRAILS (Compliance)

### What Pixio MUST NOT Do (SEC/FINRA/CVM Guidelines)
- NEVER give specific investment recommendations ("compre X", "venda Y")
- NEVER fabricate numbers. Every number must come from user data or cited source.
- NEVER show bias toward specific products/assets. No "AI-washing."
- NEVER operate without human oversight path. User can always escalate to human.
- NEVER overstate AI capabilities. Transparently identify as AI assistant.
- ALWAYS validate AI output before displaying financial figures.

### Brazil-Specific (CVM + APIMEC)
- PL 2.338/23: Sistema Nacional de Governança de IA (SIA). CVM + Banco Central como supervisores.
- Resolução CVM 179: estrutura regulatória para assessoria de investimentos.
- APIMEC diretriz: IA como FERRAMENTA de apoio ao analista, NUNCA substitui julgamento humano.
- Pixio positioning: **educação financeira + organização**, NOT consultoria de investimentos.
- Safe harbor: "Isso não é recomendação de investimento" disclaimer when discussing products.

### Hallucination Prevention (FailSafeQA Benchmark)
- LLMs hallucinate up to 41% of finance queries (2024 study). CRITICAL for financial domain.
- Mitigation: RAG grounding + structured output + source citation + confidence scoring.
- DeepEval framework: hallucination detection + factuality assessment + contextual appropriateness.
- HaluCheck: up to 24% F1 gains in medical/financial domains with Direct Preference Optimization.

---

## 4. COST OPTIMIZATION (Production-Proven)

### Model Routing Strategy (Multi-Model)
| Task | Model | Cost/1M tokens | Latency |
|------|-------|----------------|---------|
| Intent classification | Haiku 4.5 | ~$0.25 | <500ms |
| Transaction categorization | Haiku 4.5 | ~$0.25 | <500ms |
| Financial analysis | Sonnet 4 | ~$3 | ~2s |
| Complex planning/coaching | Opus 4 | ~$15 | ~5s |
| Embeddings | text-embedding-3-small | $0.02 | <200ms |
| Quick replies generation | Haiku 4.5 | ~$0.25 | <500ms |

### Cost Reduction (target: 60-80% savings)
1. **Classify first:** Route 60%+ of queries to DB directly. No LLM for "qual meu saldo?"
2. **Cache:** Same question within 1hr = cached. 15-30% cost reduction. Redis/KV store.
3. **Prompt compression:** Remove redundant instructions. 50-80% token reduction possible.
4. **Batch embeddings:** Nightly batch, not per-request. Batch API = 50% discount.
5. **RAG context:** Only top-k relevant docs, not full history. 70% context token savings.
6. **Streaming:** Better UX + early termination if user interrupts.

### Budget Guardrails
- Max 500 tokens/response. Max 4000 tokens context window.
- Per-user: 50 AI queries/day. "Limite atingido" message after.
- Monthly alert: >$50 investigate. >$100 escalate. Target: <$50/mo at current scale.

---

## 5. PROMPT ENGINEERING (Financial Domain)

### System Prompt Structure
```
[IDENTITY] Pixio, assistente financeiro para brasileiros. pt-BR sempre.
[RULES] NUNCA dar recomendação de investimento. NUNCA inventar números. NUNCA dados fabricados.
[PERSONALITY] Amigo que entende de finanças. Informal, primeiro nome. Empático com estresse financeiro.
[FORMAT] 1.Observação (o que vê) 2.Interpretação (o que significa) 3.Sugestão (o que fazer) 4.Pergunta follow-up
[CONTEXT] User: {name}, Family: {members}, Accounts: {count}, Goals: {list}
[GUARDRAILS] Max 3 números/resposta. Resumir, oferecer detalhar. Disclaimer se mencionar produtos.
```

### Advanced Techniques (from SSRN research + Deloitte)
- **Graph-of-Thought:** 15-25% higher accuracy vs chain-of-thought for financial reasoning.
- **Few-shot:** 2-3 examples of desired format. Essential for structured output.
- **Domain vocabulary:** Embed Selic/CDI/IPCA/CDB definitions in system prompt context.
- **Layered prompts:** Break complex financial analysis into sub-steps. Each step = one reasoning call.

### Anti-Patterns
| Anti-Pattern | Fix |
|-------------|-----|
| Raw data dump (50 transactions listed) | Summarize + interpret + offer detail |
| Generic quick replies ("Ok", "Entendi") | Context-aware continuations from conversation |
| No prompt injection defense | Separate user/system. Sanitize. Never inject raw user text into system. |
| Expensive model for "qual meu saldo?" | Intent classifier routes to DB. Zero LLM cost. |
| No evaluation set | Build 50+ test cases. LLM-as-judge for subjective. Exact match for factual. |
| Hallucinated financial figures | Ground EVERY number in DB data or cited source. Confidence scoring. |

---

## 6. TRANSACTION CATEGORIZATION (Real Approaches)

### Pipeline (from Neontri + Spring/Journal of Big Data)
1. **Data collection:** ISO 8583 (card) + ISO 20022 (universal) + Open Finance APIs.
2. **Tokenization:** Transaction descriptions → tokenized → numerical vectors.
3. **Hybrid model:** Rules (known merchants) + ML (unknown descriptions). 95%+ accuracy target.
4. **Challenges:** Abbreviations, limited context, imbalanced labels, new merchant types.
5. **Enrichment:** Supplement with merchant metadata, location, MCC codes.
6. **Continuous learning:** User corrections feed back into model. Log misclassifications.
7. **UniTTAB:** First foundation model for transactional data (Transformer for time series).

---

## 7. CONVERSATIONAL UX (Financial AI)

### Real-World Patterns (Bank of America Erica: 3B interactions, 50M users)
- Search-style interface > floating chat button. Contextual entry points inside banking flow.
- Conversation starters: "Quanto posso realmente economizar?" inside the banking interface.
- Professional brevity for finance. Friendly but concise. NEVER verbose paragraphs.
- Proactive insights: AI initiates ("Percebi que seus gastos com delivery dobraram") rather than waits.
- Transparently AI: user must know it's not human. Builds trust in regulated context.
- 98% answers within 44 seconds (Erica benchmark). Speed = trust in financial context.

---

## 8. SAGE DECISION FRAMEWORK

1. **Needs AI?** Direct DB > LLM for factual. "Qual meu saldo?" = zero LLM cost.
2. **Right model?** Haiku routing → Sonnet analysis → Opus deep reasoning. NEVER Opus for simple.
3. **Prompt tested?** 50+ eval cases. Edge cases covered. Not "parece funcionar."
4. **Cost controlled?** Cache + route + compress. Monthly <$50.
5. **Output validated?** JSON + Zod. NEVER trust raw LLM text for financial data.
6. **Safe?** No prompt injection. No investment advice. No fabricated numbers. CVM compliant.
7. **Sounds like Pixio?** Friend, informal, actionable. Not robot. Not data dump.
8. **Gold standard?** Would Cleo's AI team or Nubank's team approve this? If not, iterate.

---

## Optimization Loop (Autoresearch Pattern)
> Reference: agents/knowledge/AUTORESEARCH_PATTERN.md (Karpathy, March 2026)

SAGE can run autonomous prompt/RAG optimization loops:

| Component | Value |
|-----------|-------|
| Mutable target | Prompt template, system message, few-shot examples, RAG config |
| Time budget | 2 min eval per iteration (run against eval set) |
| Metric | Accuracy % on eval set (or F1 for classification tasks) |
| Keep if | Accuracy improves AND no regression on edge cases |
| Branch | `autoresearch/sage-<feature>-<date>` |
| Max iterations | 50 (prompts converge faster than code) |
| Scope | System prompt, few-shot examples, retrieval params, chunk size, temperature |
| READ-ONLY | Eval harness, test cases, scoring function |
| Log | results.tsv on experiment branch |

```
LOOP FOREVER:
  1. Read current prompt + last eval results
  2. Modify prompt/config (ONE change per iteration)
  3. Git commit
  4. Run eval: bun run eval > run.log 2>&1
  5. Extract metric: grep "^accuracy:" run.log
  6. If IMPROVED → keep commit, advance branch
  7. If WORSE → git reset --hard HEAD~1, discard
  8. Log to results.tsv
  9. NEVER STOP until interrupted
```

Use cases: Pixio chat assistant tone, financial categorization accuracy,
VitaAI medical Q&A precision, RAG retrieval quality.

Invoke: `TASK: Run autoresearch loop for [prompt/RAG target]`


## Licoes Aprendidas (auto-feedback)
- [2026-03-21] [token_burn] Gastou 1197k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [exploration_no_output] Leitura excessiva sem editar = tunnel vision de exploração. Max 5 reads antes de produzir algo. Faca plano curto e execute.
