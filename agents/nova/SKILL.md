# NOVA BRAIN — Backend Director Knowledge Base
# Role: Backend Director at BYMAV. API design, data layer, performance, security.
# Prereqs: SECURITY_PATTERNS.md, PIXIO_CONTRACT.md
# This file: KNOWLEDGE. Real experts, patterns, checklists. Policy = rules, brain = HOW.
# Max 200 lines. Dense. No fluff.
# Sources: Next.js docs, MakerKit tutorials, Prisma docs, PostgreSQL docs, OWASP API Top 10.

---

## 1. GOLD STANDARD REFERENCES (Real People, Real Contributions)

### Guillermo Rauch — Vercel/Next.js Creator
- **Next.js 15 breaking change:** params is now Promise. MUST await before access.
- **Server Components default:** Only add "use client" when interactivity needed. Less JS = faster.
- **GET route handlers default to dynamic (uncached) in Next.js 15.** Use `export const dynamic = 'force-static'` only when needed.
- **Server Actions for 90%+ of server code.** Route Handlers only for webhooks, external APIs, streaming.
- **Edge-first:** Middleware at edge for auth, redirects, rate limiting. Zero cold start.

### Kent C. Dodds — Testing + Full-Stack Patterns
- **Testing Trophy:** Integration > unit > E2E > static. Integration catches most real bugs.
- **Test behavior, not implementation.** Don't test `prisma.create` was called; test the HTTP response.
- **Colocation:** Tests next to source (`route.test.ts` beside `route.ts`). Never a separate test tree.
- **Zod as single source of truth.** Define schema once, validate on client (React Hook Form resolver) AND server (Server Action).

### Martin Fowler — Enterprise Architecture
- **Repository Pattern:** Prisma IS the repository — don't wrap it again needlessly.
- **CQRS:** Separate read/write models when reads are 10x writes. Dashboard queries != transaction inserts.
- **Strangler Fig:** Migrate incrementally. NEVER big-bang rewrite. Ref: memory/migration-lessons.md.

### Sam Newman — "Building Microservices" 2nd ed (2021)
- **Start monolith, extract later.** Pixio = modular monolith (Next.js). CORRECT at our scale.
- **Idempotency keys:** Every mutation endpoint should accept idempotency key. Stripe pattern: 24h expiry.

### Stripe API — Gold Standard REST Design (dev.to/yukioikeda analysis)
- **Prefixed IDs:** `ch_`, `cus_`, `pi_`. Instant type recognition in logs. We use UUIDs — but prefix in logs.
- **Cursor pagination (not offset).** `?cursor=abc&limit=20`. Offset skips/duplicates on concurrent writes.
- **Expandable objects:** `?expand[]=customer`. Reduce round trips. Implement for nested resources.
- **Idempotency-Key header.** Same key = same result. Prevents double charges on retries.
- **Date-based versioning:** `Stripe-Version: 2024-10-28`. Breaking changes never affect unless explicitly upgraded.
- **Actionable errors:** type, code, decline_reason, message, param, doc_url, request_id.
- **Metadata:** Custom key-value on any object. 50 keys, 40-char names, 500-char values. Flexible extension.

---

## 2. API DESIGN STANDARDS

### RESTful Conventions (BYMAV Standard)
| Pattern | Implementation | Example |
|---------|---------------|---------|
| **Resource naming** | Plural nouns, kebab-case | `/api/transactions`, `/api/bank-accounts` |
| **HTTP methods** | GET=read, POST=create, PATCH=update, DELETE=remove | `PATCH /api/transactions/123` |
| **Status codes** | 200/201/204/400/401/403/404/409/422/500 | Never 200 with error body |
| **Pagination** | Cursor-based (Stripe model). `cursor-pagination.ts` | `?cursor=abc&limit=20` |
| **Error format** | `{ error: string, code: string, details?: {} }` | Stripe-style actionable errors |

### Zod Validation (MANDATORY — Single Source of Truth)
```typescript
// src/lib/validations/transaction.ts — shared between client & server
export const CreateTransactionSchema = z.object({
  amount: z.number().positive(),
  description: z.string().min(1).max(255),
  categoryId: z.string().uuid(),
})
export type CreateTransactionInput = z.infer<typeof CreateTransactionSchema>

// Server Action: safeParse → 400 with result.error.flatten()
// Route Handler: safeParse → NextResponse.json({ error, details }, { status: 400 })
// Client: zodResolver(CreateTransactionSchema) with React Hook Form
```

### Auth Wrapper Pattern (Next.js 15 — from MakerKit)
```typescript
type AuthenticatedHandler = (
  request: NextRequest,
  context: { params: Promise<Record<string, string>>; user: User }
) => Promise<Response>

export function withAuth(handler: AuthenticatedHandler) {
  return async (request: NextRequest, context: any) => {
    const session = await getSession()
    if (!session?.user) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    return handler(request, { ...context, user: session.user })
  }
}
// Composable: withRole(['admin'], handler) wraps withAuth
```

### Error Handling (Structured)
- **External errors:** Catch, log full (Pino), return generic to client. NEVER leak table names.
- **Validation:** 400 + Zod `.flatten()` for field-level errors.
- **Auth:** 401 (not logged in) vs 403 (no permission). NEVER 200 for auth failures.
- **Not found:** 404. NEVER empty 200 for missing resources.
- **OWASP API1:2023 (BOLA):** userId in EVERY WHERE clause for UPDATE/DELETE. #1 API vulnerability.

---

## 3. DATA LAYER PATTERNS

### Prisma (Current) -> Drizzle (Migration Target)
| Concern | Prisma Pattern | Drizzle Equivalent |
|---------|---------------|-------------------|
| **Decimal** | `toNumberOrZero(field)` ALWAYS | Same custom wrapper |
| **N+1** | `include:` / `relationLoadStrategy: "join"` | `.leftJoin()` or `with:` |
| **Transactions** | `prisma.$transaction([...])` | `db.transaction(async (tx) => {...})` |
| **Raw queries** | `$queryRaw` tagged template ONLY | `sql` tagged template |
| **Connection** | Single global instance (CRITICAL) | Same — never new client per request |
| **Indexes** | `@@index` in schema on WHERE/ORDER cols | Schema-level index definition |

**Prisma 7 (late 2025):** Rust engine GONE. Pure TypeScript. 3.4x faster queries, 9x better cold start.
**Drizzle edge:** 7.4KB min+gzip, zero binary deps. Better for serverless/edge.
**Migration rule:** `strict: true` in Drizzle Kit. Without it, renames = drop+add = DATA LOSS.

### Decimal Handling (CRITICAL — fintech)
- **NEVER** `number` for money. `0.1 + 0.2 = 0.30000000000000004`.
- `toNumberOrZero()` at API boundary. Decimal.js for server-side math.
- Display: `Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' })`.

### Caching Strategy
| Layer | Tool | TTL | Invalidation | Use Case |
|-------|------|-----|-------------|----------|
| HTTP | Cloudflare CDN | 1y static | Purge on deploy | Assets, fonts |
| App | Redis (Upstash) | 5-60 min | On mutation | Aggregates, category lists |
| Query | React Query | staleTime: 30s | `invalidateQueries` | Transaction lists |
| Compute | `unstable_cache` | 60s revalidate | `revalidateTag` | Expensive DB aggregations |
| Rate Limit | @upstash/ratelimit | Edge middleware | Token bucket per IP | API abuse prevention |

---

## 4. NEXT.JS APP ROUTER PATTERNS (v15+)

### Route Handler Structure
```
src/app/api/
  transactions/
    route.ts          # GET (list), POST (create)
    [id]/route.ts     # GET (detail), PATCH (update), DELETE
  webhooks/
    stripe/route.ts   # POST only — signature verification
```

### Server Actions vs Route Handlers
| Use Case | Approach | Why |
|----------|----------|-----|
| Form mutations from React | Server Actions | Progressive enhancement, type safety |
| External API / mobile consumers | Route Handlers | Standard REST, reusable |
| Webhooks (Stripe, Celcoin) | Route Handlers | Signature verification, explicit HTTP |
| Streaming (AI responses) | Route Handlers | ReadableStream control |
| Server-side data fetch | Server Components + Prisma | No API hop needed |

### PostgreSQL RLS (Defense in Depth)
RLS via PostgreSQL nativo. better-auth gerencia sessoes. NUNCA Supabase.
- **Enable RLS on tables with sensitive data.** PostgreSQL native RLS policies.
- **Index columns used in policies.** Missing indexes = full table scan per row.
- **`security_invoker = true` on views** (PostgreSQL 15+). Views bypass RLS by default.

### OWASP API Security Top 10 (2023) Applied to Next.js
| # | Vulnerability | NOVA Mitigation |
|---|--------------|-----------------|
| API1 | Broken Object Level Auth (BOLA) | userId in EVERY WHERE clause |
| API2 | Broken Authentication | better-auth + session validation |
| API3 | Broken Object Property Level Auth | Zod strict schemas, no mass assignment |
| API4 | Unrestricted Resource Consumption | Rate limiting + pagination |
| API5 | Broken Function Level Auth | withRole() wrapper |
| API8 | Security Misconfiguration | RLS + env validation + Zod |

---

## 5. PERFORMANCE CHECKLIST (Before Every Endpoint Ships)
- [ ] Response < 200ms at p95 (Pino timing)
- [ ] N+1 eliminated (`include`/`join`, verify with query log)
- [ ] Pagination with cursor + take (NEVER unbounded arrays)
- [ ] Indexes on WHERE/ORDER BY columns (`EXPLAIN ANALYZE`)
- [ ] `toNumberOrZero()` on ALL money fields
- [ ] Zod validation on ALL inputs
- [ ] Auth + IDOR check (userId in WHERE)
- [ ] Generic error responses (no internal details)
- [ ] Structured logging (Pino, not console.log)

### NOVA's Decision Framework
1. **Does it already exist?** Search codebase. Duplicate endpoints = tech debt.
2. **Is input validated?** Zod schema mandatory.
3. **Is auth enforced?** `getAuthUserId()` on line 1.
4. **Is it IDOR-safe?** userId in every WHERE for UPDATE/DELETE.
5. **Is it paginated?** Unbounded queries = production bomb.
6. **Is Decimal handled?** `toNumberOrZero()` for every money field.
7. **Would Nubank/Stripe's backend team approve this?** Gold standard.

## 11. INFRA GOVERNANCE — REGRAS PARA DEPLOY (05/03/2026)
Source: memory/infra-governance.md

1. **CAPACITY CHECK**: Antes de subir QUALQUER serviço novo → verificar budget RAM. Sobra 30%? Se não → NÃO SOBE
2. **RESOURCE LIMITS**: Todo container DEVE ter mem_limit + cpus no compose
3. **1 LUGAR POR SERVIÇO**: VPS OU local. NUNCA ambos
4. **BUDGET VPS**: 22/32 GB. pixio-web=4GB max, vita-web=2GB, hub=1GB, redis=512MB
5. **restart: unless-stopped** (NUNCA always)


## Licoes Aprendidas (auto-feedback)
- [2026-03-21] [token_burn] Gastou 490k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 2091k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 1667k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 7381k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 112k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 5981k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 6194k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 2332k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 2746k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 124k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [exploration_no_output] Leitura excessiva sem editar = tunnel vision de exploração. Max 5 reads antes de produzir algo. Faca plano curto e execute.
- [2026-03-12] [edit_without_read] SEMPRE ler o arquivo antes de editar. Edit sem Read = edit cego = erros evitaveis.
