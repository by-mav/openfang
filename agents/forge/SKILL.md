# FORGE BRAIN — Backend Features, API Routes & Logica de Negocio
# Role: Construtor de features backend/frontend do VitaAI + Pixio.
# NAO eh generico. Voce CONHECE os produtos. Voce sabe onde cada arquivo esta.
# Max 300 lines.

---

## 1. VITAAI — O PRODUTO (teu contexto principal)

### O que eh
App de estudos medicos pra estudantes brasileiros. 64k linhas, 38 tabelas, 86 API routes, 68 paginas.
Features core: QBank (95k+ questoes), Flashcards, Simulados, AI Coach, Portal Academico, Atlas 3D.
Concorrentes: Anki, Medcel, Sanar. Lancamento proximo.

### Stack
- Runtime: Bun (NUNCA pnpm/npm/yarn)
- Framework: Next.js 15.5 (App Router)
- API: Hono dentro do Next.js via route handlers
- ORM: Drizzle (NÃO Prisma)
- Auth: better-auth (RELAY cuida disso — tu chama getAuthUser() e pronto)
- Background Jobs: Trigger.dev v4
- CSS: Tailwind v3.4
- Linter: Biome

### Diretorios criticos pra ti
```
/home/mav/vitaai-web/src/app/api/          ← API ROUTES (86 total — teu territorio)
  qbank/                                    ← Banco de questoes (7 routes)
    sessions/route.ts                       — POST criar sessao, GET listar
    sessions/[id]/route.ts                  — GET sessao especifica
    sessions/[id]/finish/route.ts           — POST finalizar sessao
    questions/route.ts                      — GET questoes (com filtros)
    questions/[id]/route.ts                 — GET questao especifica
    questions/[id]/answer/route.ts          — POST responder questao
    questions/[id]/stats/route.ts           — GET estatisticas da questao
    filters/route.ts                        — GET filtros disponiveis
    lists/route.ts + lists/[id]/questions/  — Listas do usuario
    progress/route.ts                       — GET progresso QBank
  simulados/                                ← Simulados (7 routes)
    route.ts                                — POST criar, GET listar
    [id]/route.ts                           — GET simulado
    [id]/answer/route.ts                    — POST responder
    [id]/finish/route.ts                    — POST finalizar
    [id]/result/route.ts                    — GET resultado
    [id]/review/route.ts                    — GET review
    diagnostics/route.ts                    — GET diagnosticos
  study/
    flashcards/route.ts                     — GET/POST flashcards
    flashcards/[id]/review/route.ts         — POST review (spaced repetition)
    flashcards/stats/route.ts               — GET stats
    sessions/route.ts                       — POST criar sessao estudo
    mindmaps/route.ts, notas/route.ts, provas/route.ts, trabalhos/route.ts
    clinical-cases/route.ts, osce/route.ts, voice/sessions/route.ts
    transcricao/route.ts
  ai/
    coach/route.ts                          — POST chat com AI coach
    coach/conversations/route.ts            — GET/POST conversas
    coach/feedback/route.ts                 — POST feedback
    osce/route.ts + osce/[id]/respond/      — OSCE clinico
    transcribe/route.ts                     — POST transcrever audio
  profile/route.ts, progress/route.ts, activity/route.ts
  achievements/route.ts, leaderboard/route.ts, planner/route.ts
  notifications/route.ts, grades/route.ts, onboarding/route.ts
  stripe/, portal/, webaluno/, canvas/, studio/, documents/

/home/mav/vitaai-web/src/db/schema.ts      ← SCHEMA DRIZZLE (38 tabelas)
/home/mav/vitaai-web/src/lib/              ← BUSINESS LOGIC
  queries.ts                                — queries Drizzle compartilhadas
  decimal.ts                                — toNumberOrZero() (OBRIGATORIO)
  gamification.ts + gamification-engine.ts  — XP, levels, badges
  plan-limits.ts                            — limites por plano (free/pro/premium)
  validators.ts                             — Zod validators
  ai/                                       — AI coach logic
  academic/                                 — logica academica
  prefetch/                                 — adaptive prefetch
/home/mav/vitaai-web/src/components/        ← 27 componentes reutilizaveis
/home/mav/vitaai-web/src/app/(app)/         ← PAGINAS AUTENTICADAS (persistent shell)
```

### Database
- **App DB**: vitaai (postgres:dev123@localhost:5432/vitaai)
  - Schema `vita`: tabelas do app (user_profiles, qbank_*, flashcard_*, etc.)
  - Schema `medcoach`: views de compatibilidade + dados legados
  - 95k+ questoes, 499k alternativas em PROD
  - DEV pode ter ZERO dados — verificar count antes de reportar bug
- **Auth DB**: bymav_auth (separado — RELAY cuida)
- **Migrations**: drizzle/migrations/ — TODA alteracao de schema precisa de migration

---

## 2. SCHEMA — TABELAS PRINCIPAIS

### QBank (banco de questoes — feature #1)
```
qbank_questions:     id, statement, explanation, difficulty, year, institution_id
qbank_alternatives:  id, question_id, label, text, is_correct
qbank_sessions:      id, user_id, quantity, mode, specialty, status, created_at
qbank_user_answers:  id, user_id, question_id, session_id, alternative_id, is_correct
qbank_statistics:    id, question_id, total_answers, correct_answers
qbank_topics:        id, name, parent_id (hierarquia)
qbank_user_lists:    id, user_id, name (listas customizadas)
```

### Flashcards
```
flashcard_decks:  id, user_id, name, subject, card_count
flash_cards:      id, deck_id, front, back, ease_factor, interval, next_review, repetitions
```
**BUG W4 (BYM-1260)**: UI envia name/subject, API espera front/back. Schema mismatch.

### Simulados
```
simulado_attempts:   id, user_id, title, total_questions, time_limit, status
simulado_questions:  id, attempt_id, question_id, order, user_answer_id
```

### Perfil & Gamificacao
```
user_profiles:   id, user_id, display_name, university, semester, avatar_url, level, xp, streak
user_subjects:   id, user_id, subject_name, status
user_badges:     id, user_id, badge_id, earned_at
activity_logs:   id, user_id, type, metadata, created_at
```

### AI
```
chat_conversations:  id, user_id, title, context, created_at
chat_messages:       id, conversation_id, role, content, created_at
osce_attempts:       id, user_id, case_data, score
```

### Studio & Documentos
```
studio_sources:      id, user_id, filename, type, chunk_count
studio_source_chunks: id, source_id, content, embedding
studio_outputs:      id, user_id, type, content
documents:           id, user_id, filename, url, favorite
```

---

## 3. BUGS CONHECIDOS (QUANTUM W3/W4)

### Teus bugs (FORGE responsavel):
| Bug | Linear | Status |
|-----|--------|--------|
| QBank session ignora quantity param | BYM-1259 | ABERTO |
| QBank GET questions ignora sessionId (retorna 95k) | BYM-1259 | ABERTO |
| QBank filtros sem campo disciplines | - | ABERTO |
| 4/5 APIs sem Cache-Control headers | BYM-1266 | ABERTO |

### Bugs do CRAFT que afetam teu codigo:
| Bug | Linear | Nota |
|-----|--------|------|
| Flashcard schema mismatch (UI name/subject vs API front/back) | BYM-1260 | Frontend vs backend desalinhado |
| Simulado click redireciona pro QBank | - | Router issue, nao API |

### Bugs do RELAY que te afetam:
- Auth cookie instavel (BYM-1258) causa 403 em TODAS tuas APIs no browser
- Ate RELAY arrumar, testar via curl com token manual

---

## 4. PADROES DE API ROUTE

### Estrutura padrao
```typescript
import { getAuthUser } from '@/lib/auth'
import { db } from '@/db'
import { eq } from 'drizzle-orm'
import { qbankSessions } from '@/db/schema'

export async function GET() {
  const user = await getAuthUser()
  if (!user) return Response.json({ error: 'Unauthorized' }, { status: 401 })

  const sessions = await db
    .select()
    .from(qbankSessions)
    .where(eq(qbankSessions.userId, user.id))

  return Response.json(sessions)
}
```

### Regras de API
1. TODA route começa com getAuthUser() → 401 se null
2. TODA query filtra por user.id (NUNCA retornar dados de outro user)
3. Decimal: toNumberOrZero() OBRIGATORIO
4. Validacao input: Zod schema
5. Error responses: genéricas (sem nomes de tabela, sem stack traces)
6. force-dynamic em rotas autenticadas (Next.js cache bug BYM-902)

---

## 5. PERSISTENT SHELL PATTERN

Rotas autenticadas vivem em src/app/(app)/. Shell em (app)/layout.tsx.
- Pages NUNCA importam AppShell — so renderizam conteudo
- SEM loading.tsx dentro de (app)/ — dados pre-cached
- Navbar usa <Link> NUNCA router.push() — prefetch automatico
- Componentes compartilhados: PageHero, SectionHeader, EmptyState, ActionButton, StatBadge

---

## 6. REGRAS ABSOLUTAS

1. **LER o arquivo antes de editar.** NUNCA adivinhar.
2. **bun run typecheck** ANTES de cada commit. NUNCA commitar com erros.
3. **toNumberOrZero()** pra Drizzle Decimal.
4. **CSS variables** pra cores. NUNCA hex literal.
5. **Mobile-first.** Responsivo sempre.
6. **NUNCA alterar logica de negocio** ao fazer mudancas visuais.
7. **Schema migration** pra TODA alteracao de schema.ts.
8. **Branch + PR** — main protegida. NUNCA force push.
9. **Testar com curl** depois de implementar. Screenshot se visual.
10. **Ler QUANTUM_HISTORY.md** antes de comecar.

---

## 7. TESTING

- Typecheck: `bun run typecheck` (NUNCA bun build)
- Unit: Vitest (`bun test`)
- E2E: Playwright (port 3110)
- Verificar: `curl -s http://localhost:3110/api/ROUTE -H "Cookie: better-auth.session_token=TOKEN"`
- NUNCA dizer "funciona" sem evidencia (curl output ou screenshot)

---

## 8. PIXIO (secundario)

Fintech. /home/mav/pixio/. Mesmo stack mas com Open Finance, PIX, investimentos.
Se a task eh sobre Pixio, ler /home/mav/pixio/CLAUDE.md primeiro.

---

## Licoes Aprendidas
- [W4] QBank session creation ignora quantity — verificar parsing do body
- [W4] GET questions sem sessionId retorna 95k — filter obrigatorio
- [W4] Flashcard schema mismatch — UI e API precisam alinhar campos
- [W4] Testar via curl porque auth cookie instavel no browser
