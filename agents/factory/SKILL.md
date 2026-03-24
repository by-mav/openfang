# FACTORY BRAIN — SaaS Builder Knowledge Base
# Role: SaaS scaffolding worker at BYMAV. New app creation, monorepo management.
# Prereqs: saas-factory.md, BYMAV CLAUDE.md, factory-stack.md
# This file: KNOWLEDGE. Monorepo patterns, template architecture, tooling.
# Max 200 lines. Dense. No fluff.
# Sources: Turborepo docs, Biome docs, Drizzle docs, create-t3-app, SaaS Yacht Club, MakerKit.

---

## 1. GOLD STANDARD REFERENCES

### Jared Palmer — Turborepo Creator (Vercel)
- **"Monorepos make code sharing trivial."** Shared packages = write once, use everywhere.
- **Task pipeline:** Build depends on typecheck. Turborepo resolves graph automatically.
- **Remote caching:** Hash inputs -> cache outputs. Skip if unchanged. 70-85% faster builds.
- **Workspace protocol:** `"@bymav/ui": "workspace:*"` — resolved at install, not published to npm.
- **Pruned installs:** `turbo prune --scope=app-name` — install only app's dependencies.
- **Used by:** Netflix, Airbnb, Microsoft in production monorepos.

### Biome — Unified Linter/Formatter (biomejs.dev)
- **10-25x faster than ESLint + Prettier.** Rust binary. 10k-line monorepo: ~200ms vs 3-5s.
- **One config file** instead of 4 (.eslintrc, .prettierrc, .editorconfig, .eslintignore).
- **Single binary** instead of 127+ npm packages.
- **97% Prettier-compatible** formatting. 423+ lint rules. Type-aware linting in v2.3.
- **Migration:** `biome migrate eslint --write` + `biome migrate prettier --write`. Automated.
- **camelCase rule names** (not kebab-case like ESLint). Tabs by default (configurable).
- **Limitation:** Less mature for Vue, Markdown, YAML. Fine for Next.js/React.

### Drizzle ORM (orm.drizzle.team) — BYMAV Migration Target
- **Bundle:** 7.4KB min+gzip. Zero binary dependencies. Negligible cold start.
- **Code-first schema:** TypeScript = schema = types. No generation step. Instant type inference.
- **Dual API:** SQL-like Select API (precise control) + Query API (simpler patterns).
- **vs Prisma 7:** Prisma dropped Rust engine (late 2025). Now pure TS. 3.4x faster queries.
  - Prisma 7 bundle: ~1.6MB (90% reduction from v6's 14MB). Still 200x larger than Drizzle.
  - Prisma 7 serverless cold start: 9x better than v6. Drizzle still faster on edge.
- **Migration:** `drizzle-kit generate` -> `drizzle-kit migrate`. No generation step for types.
  - **CRITICAL:** Always `strict: true` in drizzle.config. Without it, renames = drop+add = DATA LOSS.
- **drizzle-seed:** Deterministic fake data via seedable pRNG. Consistent across runs. Great for testing.
- **Performance:** 4.6k reqs/s with ~100ms p95 on modest hardware (Drizzle benchmarks).

### better-auth — Authentication Library (better-auth.com)
- **Framework-agnostic TypeScript.** Plugin ecosystem for advanced features.
- **Multi-tenant OOTB:** Organization plugin = memberships, invitations, RBAC. No custom code.
- **Next.js integration:** `nextCookies` plugin for automatic cookie handling in Server Actions.
- **vs NextAuth:** better-auth has built-in org/team/RBAC. NextAuth needs custom layer.
- **Google OAuth:** Single shared client across all BYMAV apps (memory/google-oauth-config.md).

### SaaS Boilerplate Patterns (SaaS Yacht Club, Supastarter, create-t3-app)
- **create-t3-app:** Gold standard for TypeScript full-stack. tRPC + Prisma/Drizzle + NextAuth + Tailwind.
- **SaaS Yacht Club:** Next.js 15 + React 19 + better-auth + Stripe + multi-tenant.
- **Supastarter:** Organization switching, role-based access, per-org billing from day 1.
- **Common pattern:** `tenant_id` column in all tables. Bind once in data access layer.

---

## 2. BYMAV MONOREPO STRUCTURE

### Directory Layout
```
/home/mav/bymav/
  apps/
    medcoach/           # VitaAI web (Next.js)
    [new-app]/          # Created by pnpm factory:new
  apps-mobile/
    medcoach-template-mobile/  # DELETED — Expo ERRADICADO. Mobile = nativo only.
  packages/
    ui/                 # @bymav/ui — 17 shadcn components
    auth/               # @bymav/auth — better-auth (server/browser/middleware)
    config-ts/          # @bymav/config-ts — tsconfig base + nextjs
    config-tailwind/    # @bymav/config-tailwind — shared Tailwind theme
  infra/
    projects-registry.json  # Source of truth for ALL apps
  turbo.json            # Pipeline config
  pnpm-workspace.yaml   # Workspace definition
```

### Shared Packages
| Package | Purpose | Exports |
|---------|---------|---------|
| `@bymav/ui` | shadcn/ui components | Button, Card, Input, Dialog, etc. (17) |
| `@bymav/auth` | Authentication | `auth` (server), `authClient` (browser), middleware |
| `@bymav/config-ts` | TypeScript config | `base.json`, `nextjs.json` for extends |
| `@bymav/config-tailwind` | Tailwind theme | Colors, fonts, spacing tokens |

### turbo.json Pipeline (Turborepo best practice)
```json
{
  "pipeline": {
    "typecheck": {},
    "lint": {},
    "build": { "dependsOn": ["^build", "typecheck"] },
    "dev": { "cache": false, "persistent": true }
  }
}
```
- `^build` = build dependencies first (topological). `typecheck` before build.
- `dev` = never cached, persistent process. `lint` and `typecheck` = cacheable.

---

## 3. DOCKER PATTERNS (BYMAV Standard)

### Bun Multi-Stage Build (from Docker + Bun best practices)
```dockerfile
# Stage 1: Install dependencies
FROM oven/bun:latest AS deps
WORKDIR /app
COPY package.json bun.lock ./
RUN bun install --frozen-lockfile --production

# Stage 2: Build
FROM oven/bun:latest AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN bun run build

# Stage 3: Production (minimal)
FROM oven/bun:latest AS runner
WORKDIR /app
RUN adduser --system --uid 1000 appuser
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
USER appuser
ENV NODE_ENV=production
CMD ["bun", "run", "server.js"]
```
- **Always `--frozen-lockfile`** for reproducible installs.
- **Debian base for Bun** (not Alpine). Bun requires glibc, Alpine uses musl.
- **Non-root user (uid 1000).** Container Claude = non-root uid1000.
- **Multi-stage reduces image 70-90%.** Separate deps/build/runtime stages.
- **ENV NODE_ENV=production** before deps install to prune dev packages.

---

## 4. NEW APP CREATION WORKFLOW

### The Command
```bash
cd /home/mav/bymav && pnpm factory:new <app-name>
```

### What It Does
1. Creates `apps/<app-name>/` from template
2. Generates `package.json` with workspace deps (`@bymav/ui`, `@bymav/auth`, etc.)
3. Sets up `tsconfig.json` extending `@bymav/config-ts`
4. Configures Tailwind extending `@bymav/config-tailwind`
5. Sets up better-auth with Google OAuth (shared client)
6. Creates `.env.local` template
7. Registers in `infra/projects-registry.json`
8. Allocates unique port (3070+)

### Post-Creation Checklist
- [ ] `.env.local` filled with real values (DATABASE_URL, BETTER_AUTH_SECRET from `pass`)
- [ ] Google OAuth redirect URI added (ONE shared client)
- [ ] `pnpm install` from monorepo root
- [ ] `pnpm typecheck` passes
- [ ] App starts: `pnpm dev --filter=<app-name>`
- [ ] Login with Google + email/password both work
- [ ] Registered in projects-registry.json
- [ ] Multi-tenant org support configured (better-auth org plugin)

### Auth Setup: `betterAuth({ database: drizzleAdapter(db), socialProviders: { google: {...} }, emailAndPassword: { enabled: true }, plugins: [organization()] })`

---

## 5. MULTI-TENANT (WorkOS + SaaS boilerplate patterns)
- Shared tables with `organization_id` column. Bind ONCE in data access layer.
- RLS at DB level = double protection. Role hierarchy: `owner > admin > member > viewer`.
- better-auth `organization` plugin: membership, invitations, RBAC out of the box.
- Privilege escalation prevention: users can't assign roles higher than their own.

---

## 6. ANTI-PATTERNS

| Anti-Pattern | Why | Fix |
|-------------|-----|-----|
| `mkdir apps/x && npm init` | Not registered, breaks monorepo | `pnpm factory:new` ALWAYS |
| Duplicate types across apps | Drift, inconsistency | Shared types in packages/ |
| App-specific Tailwind config | Visual inconsistency | Extend `@bymav/config-tailwind` |
| Standalone repo outside bymav/ | Orphaned, no shared packages | VIOLATION |
| `pnpm build` in development | Corrupts .next/, slow | `pnpm typecheck` ONLY |
| Alpine base for Bun Docker | glibc required, musl breaks | Use Debian/oven-bun images |
| No `strict: true` in Drizzle Kit | Renames become drop+add | ALWAYS enable strict mode |

---

## 7. FACTORY DECISION FRAMEWORK

1. **Using `pnpm factory:new`?** Manual creation = FORBIDDEN.
2. **Registered in projects-registry.json?** Pre-commit hook validates.
3. **Using shared packages?** `@bymav/ui`, `@bymav/auth`, configs. No duplication.
4. **Auth configured?** Google OAuth (shared) + email/password. Both working.
5. **Env vars from `pass`?** NEVER hardcoded.
6. **Multi-tenant ready?** better-auth org plugin configured. `organization_id` on tables.
7. **Typecheck passes at root?** `pnpm typecheck` from `/home/mav/bymav/`.

If ANY answer is NO, the app is not ready.
