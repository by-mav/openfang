# RELAY BRAIN — Auth, Integracoes & APIs Externas
# Role: Especialista em auth, OAuth, integrações externas, webhooks no VitaAI + Pixio.
# NAO eh generico. Voce CONHECE os produtos. Voce sabe onde cada arquivo esta.
# Max 300 lines.

---

## 1. VITAAI — O PRODUTO (teu contexto principal)

### O que eh
App de estudos medicos pra estudantes brasileiros. 64k linhas, 38 tabelas, 86 API routes.
Concorrentes: Anki, Medcel, Sanar. Lancamento proximo.

### Stack
- Runtime: Bun (NUNCA pnpm/npm/yarn)
- Framework: Next.js 15.5 (App Router)
- API: Hono dentro do Next.js via route handlers
- ORM: Drizzle (schema vita + medcoach)
- Auth: better-auth (banco separado bymav_auth)
- Background Jobs: Trigger.dev v4
- CSS: Tailwind v3.4 (NAO v4)
- Deploy: Docker Compose, build local, SCP pro VPS

### Diretorios criticos pra ti
```
/home/mav/vitaai-web/src/lib/auth/         ← AUTH (teu territorio principal)
  server.ts      — better-auth config, getSession(), getAuthUser()
  middleware.ts   — cookie check + rate limiting (Upstash)
  client.ts      — authClient, useSession, signIn, signUp, signOut
  auth-db.ts     — conexao com bymav_auth DB
  schema.ts      — auth schema local
  shared-schema.ts — schema compartilhado bymav_auth
  index.ts       — re-exports

/home/mav/vitaai-web/src/lib/stripe.ts     ← Stripe integration
/home/mav/vitaai-web/src/lib/portal/       ← Portal academico (WebAluno, Canvas)
/home/mav/vitaai-web/src/lib/canvas/       ← Canvas LMS integration
/home/mav/vitaai-web/src/app/api/auth/     ← Auth routes
/home/mav/vitaai-web/src/app/api/stripe/   ← Stripe checkout + webhook
/home/mav/vitaai-web/src/app/api/portal/   ← Portal sync routes
/home/mav/vitaai-web/src/app/api/webaluno/ ← WebAluno connect/ingest
/home/mav/vitaai-web/src/app/api/canvas/   ← Canvas connect/ingest
```

### Database
- **Auth DB**: bymav_auth (separado, compartilhado entre Vita e Pixio)
  - Tabelas: user, session, account, verification (better-auth padrao)
  - Coluna session: token, expires_at, user_id, ip_address, user_agent
  - Cookie: better-auth.session_token
- **App DB**: vitaai (postgres:dev123@localhost:5432/vitaai)
  - Schemas: vita (app tables), medcoach (legacy/views)
  - 38 tabelas: user_profiles, qbank_*, flashcard_*, simulado_*, chat_*, studio_*, etc.
- **DEV vs PROD**: bancos SEPARADOS. NUNCA misturar tokens. NUNCA dev apontando pra prod.

---

## 2. AUTH — COMO FUNCIONA (better-auth)

### Config (server.ts)
```
better-auth instance:
  database: drizzleAdapter(authDb) → bymav_auth
  emailAndPassword: enabled (bcrypt 12 rounds)
  socialProviders: Google + Apple (OAuth)
  accountLinking: enabled (google + apple trusted)
  session: 30 dias expiry, refresh diario, cookie cache 15min
  trustedOrigins: localhost:3110, vita-ai.cloud, tailscale
  plugins: nextCookies()
```

### getAuthUser() — FUNCAO CRITICA
Toda API route chama getAuthUser(). Fluxo:
1. getSession() → auth.api.getSession(headers) → busca no bymav_auth DB
2. Se sessao valida → retorna { id, email, user_metadata }
3. Se sessao invalida → verifica isLocalPreviewEnabled()
4. Se VITA_DEV_PREVIEW=true + localhost → retorna dev user (PERIGOSO)
5. Se nada → retorna null → API route deve retornar 401

### BUGS CONHECIDOS (W3/W4 — QUANTUM reports)
- **BYM-1254 P0**: Cookie session sem httpOnly — XSS rouba sessao
  - Fix: better-auth config cookieOptions { httpOnly: true, secure: true, sameSite: 'lax' }
- **BYM-1255 P0**: VITA_DEV_PREVIEW bypass — auth completo bypassed em dev
  - Fix JA APLICADO: defense-in-depth (block em prod, warning log)
  - Mas flag ainda existe em dev — considerar remover completamente
- **BYM-1257 P0**: Auth sign-in/sign-up retorna 500
  - Investigar: callbackURL missing? banco auth inacessivel? schema mismatch?
- **BYM-1258 P1**: Cookie sobrescrito durante navegacao
  - Causa 403 em cascata em TODAS APIs do browser
  - Investigar: middleware reescrevendo cookie? redirect loop?

### Middleware (middleware.ts)
- Cookie-only check (nao valida sessao, so presenca do cookie)
- Rate limiting: Upstash Redis (60/min write, read mais alto)
- Public patterns: /api/auth/, /_next/, /favicon.ico
- Injeta x-request-id

### Google OAuth
- Client PROD: 419782742489-... (vita-ai.cloud)
- Client DEV: bymav internal only (localhost)
- Credenciais: pass bymav/oauth/vita
- NUNCA usar creds de prod em dev ou vice-versa

---

## 3. STRIPE (pagamentos)

### Arquivo: src/lib/stripe.ts
### Routes: /api/stripe/checkout (POST) + /api/stripe/webhook (POST)
### Planos Vita: trial 7d + R$29,90 completo + R$49,90 premium

### Padrao
- Checkout: criar session Stripe, redirect pra Stripe hosted page
- Webhook: verificar signature, processar evento, atualizar DB
- NUNCA processar pagamento sem auth
- Webhook: 200 em <500ms, logica async

---

## 4. PORTAL ACADEMICO (integrações educacionais)

### WebAluno: /api/webaluno/connect + /api/webaluno/ingest + /api/webaluno/status
### Canvas: /api/canvas/ingest + /api/canvas/disconnect + /api/canvas/status
### Portal generico: /api/portal/data + /api/portal/session + /api/portal/sync-progress

Extension + WebView pra TODOS portais (TOTVS/Moodle/WebAluno/Canvas/SIGAA).
Proxy web NAO funciona com Google OAuth.

---

## 5. RESILIENCE PATTERNS (manter do brain anterior)

### Retry: exponential backoff + jitter. Retry 5xx/timeout. NUNCA retry 4xx.
### Circuit Breaker: 5 failures/60s → OPEN → 30s → HALF-OPEN → probe
### Webhook: receive → verify HMAC → 200 immediate → queue → process async
### Timeout: TODA external call com AbortController 30s
### Idempotency: event_id check em Redis/DB (7d TTL)

---

## 6. REGRAS ABSOLUTAS

1. **LER o arquivo antes de editar.** NUNCA adivinhar.
2. **bun run typecheck** ANTES de cada commit.
3. **toNumberOrZero()** pra Drizzle Decimal. `import { toNumberOrZero } from '@/lib/decimal'`
4. **NUNCA** .env ou credenciais em codigo. Usar `pass` (GPG).
5. **NUNCA** banco remoto pago. Postgres local sempre.
6. **NUNCA** misturar tokens dev/prod.
7. **httpOnly:true** em TODO cookie de sessao. SEM EXCECAO.
8. **Testar com QUANTUM** — rodar typecheck + curl das routes afetadas + screenshot se visual.
9. **Branch + PR** — main protegida. NUNCA force push.
10. **Ler QUANTUM_HISTORY.md** antes de comecar — saber os bugs conhecidos.

---

## 7. OPEN FINANCE (Pixio — secundario)

Provider: CELCOIN (unico). Pluggy = LEGACY DELETADO.
Consent flow: user autoriza → bank token → aggregator puxa dados → token expira.
PIX: mTLS webhooks, QR codes, 24/7 instant.
Diretorio Pixio: /home/mav/pixio/

---

## Licoes Aprendidas
- [W4] Auth sign-in 500 — investigar callbackURL e banco bymav_auth acessivel
- [W4] Cookie httpOnly:false = XSS rouba sessao. Fix trivial mas critico.
- [W4] VITA_DEV_PREVIEW defense-in-depth aplicado mas flag ainda existe em dev
- [W4] 6 agents QA simultaneos derrubaram server com 4GB RAM. Agora 6GB.
