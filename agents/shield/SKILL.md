# SHIELD BRAIN — VP Security Knowledge Base

> Dense reference for SHIELD. Real authors, real CVEs, actionable patterns.
> Max 200 lines. No fluff. Cross-ref: SECURITY_PATTERNS.md (code), SENTINEL_BRAIN.md (infra).

---

## 1. GOLD STANDARD — Real References

### Books & Frameworks
| Ref | Author(s) | Core Idea | Apply at BYMAV |
|-----|-----------|-----------|----------------|
| **OWASP Top 10 (2021)** | OWASP Foundation | 10 critical web risks. 94% of apps have A01 (Broken Access Control). | Every PR reviewed against. Section 2 below. |
| **OWASP API Security Top 10 (2023)** | OWASP | API1=BOLA (~40% of API attacks), API2=Broken Auth. | Section 4 below. Real examples. |
| **Threat Modeling** (2014) | Adam Shostack (Microsoft) | STRIDE: Spoofing/Tampering/Repudiation/Info Disclosure/DoS/Elevation. Before code. | New feature = 10min STRIDE. |
| **NIST CSF 2.0** (2024) | NIST | Govern, Identify, Protect, Detect, Respond, Recover. Risk-based. | Align incident response to NIST. |
| **Web App Hacker's Handbook** 2nd ed | Stuttard, Pinto (PortSwigger) | Systematic: map, discover, exploit. Auth/sessions/access/injection/logic. | PR review: what can attacker control? |

### Key Practitioners
- **Troy Hunt** — haveibeenpwned.com. K-anonymity API for breached password checks. Password storage: bcrypt/argon2, NEVER SHA-256. Breach notification best practices.
- **Scott Helme** — securityheaders.com. Security headers grading (A+ to F). CSP, HSTS preload, X-Frame-Options. Free scanner for auditing.
- **Dafydd Stuttard** — Burp Suite creator. PortSwigger Web Security Academy = best free hands-on web security training.
- **Bruce Schneier** — "Amateurs hack systems, professionals hack people." Attack trees. Never roll your own crypto.
- **Adam Shostack** — "If you can draw it on a whiteboard, you can threat model it."

---

## 2. OWASP TOP 10 (2021) — BYMAV-Specific Fixes

| # | Risk | BYMAV Pattern | Fix |
|---|------|---------------|-----|
| **A01** | Broken Access Control | `prisma.update({ where: { id } })` sem userId | userId no WHERE. `getAuthUserId()` obrigatorio. |
| **A02** | Crypto Failures | Secrets in git, HTTP | bcrypt/argon2 (better-auth). TLS via Cloudflare. `pass` (GPG). |
| **A03** | Injection | `$queryRawUnsafe`, `innerHTML` | `Prisma.sql` tagged templates. DOMPurify. CSP. |
| **A04** | Insecure Design | No threat model, no rate limit | STRIDE before code. Rate limit auth endpoints. |
| **A05** | Security Misconfig | Docker root, CSP unsafe-inline, CORS * | uid 1000. CSP strict. CORS allowlist. |
| **A06** | Vulnerable Components | npm CVEs | `npm audit --production`. Trivy on images. Socket.dev. |
| **A07** | Auth Failures | OAuth `checks:['none']`, no session timeout | `checks:['state','pkce']`. Session 30d max. MFA admin. |
| **A08** | Integrity Failures | Unsigned deploys | Git-signed commits. Image pinning. Lockfile integrity. |
| **A09** | Logging Gaps | Failed logins unaudited | AuditActions for auth. Alertmanager. Structured logs. |
| **A10** | SSRF | `fetch(userUrl)` unvalidated | Host allowlist + `isPrivateIP()` block. |

---

## 3. REAL CVEs — Know Your Stack

### CVE-2025-29927: Next.js Middleware Authorization Bypass (CVSS 9.1)
Source: [ProjectDiscovery](https://projectdiscovery.io/blog/nextjs-middleware-authorization-bypass), [Datadog](https://securitylabs.datadoghq.com/articles/nextjs-middleware-auth-bypass/)
- **How**: `x-middleware-subrequest` header skips ALL middleware. Attacker adds header -> auth bypassed.
- **Payload (13.2+)**: `x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware`
- **Affected**: Next.js <14.2.25, <15.2.3. Vercel auto-patched; self-hosted = VULNERABLE.
- **BYMAV fix**: Strip header at Cloudflare level: `proxy_set_header x-middleware-subrequest "";`
- **Lesson**: NEVER rely solely on middleware for auth. Server-side checks in route handlers too.

### CVE-2025-66478: Next.js (additional)
Second Next.js CVE in same period. Pattern: framework internals exposed as attack surface.

### Supabase RLS Mass Exposure (Jan 2025)
Source: [byteiota](https://byteiota.com/supabase-security-flaw-170-apps-exposed-by-missing-rls/)
- **170+ apps** built with AI tools (Lovable) had databases fully exposed. RLS not enabled.
- **83% of Supabase breaches** = RLS misconfiguration. Tables default to RLS OFF.
- **BYMAV rule**: EVERY new table = `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` + policy.

### npm Supply Chain: SHA1-Hulud Worm (2024-2025)
Source: [Snyk](https://snyk.io/blog/sha1-hulud-npm-supply-chain-incident/), [TrendMicro](https://www.trendmicro.com/en_us/research/25/i/npm-supply-chain-attack.html)
- Trojanized packages (chalk, debug, ansi-styles) = 2.6B weekly downloads compromised.
- `preinstall` scripts deploy payloads turning machines into attacker-controlled runners.
- **600+ packages** impacted including Zapier, PostHog, Postman dependencies.
- **BYMAV defense**: `npm audit`, Socket.dev, lockfile integrity checks, `--ignore-scripts` for untrusted.

---

## 4. API SECURITY — OWASP API Top 10 (2023)

### BOLA/IDOR (API1 — 40% of all API attacks)
Real examples: Dell portal = 49M records. Automobile API = any VIN controls any car.
Attack: change `id` parameter. `GET /api/users/UUID-other-user` without ownership check.
**Fix**: `WHERE user_id = auth.uid()` on EVERY query. UUIDs (not sequential IDs). Rate limit enumeration.

### Supabase RLS Bypass Patterns
Source: [Precursor Security](https://www.precursorsecurity.com/security-blog/row-level-recklessness-testing-supabase-security)
1. **UUID enumeration**: `id=gt.00000000-0000-0000-0000-000000000000` returns ALL rows via PostgREST
2. **Missing WITH CHECK**: INSERT policy without WITH CHECK = insert rows as any user_id
3. **user_metadata in JWT**: Modifiable by end users. NEVER use in RLS policies. Use `auth.uid()` only.
4. **SQL Editor false confidence**: Runs as postgres superuser, bypasses ALL RLS. Test with anon/authenticated.
5. **service_role in client**: service_role key = god mode. NEVER in frontend. NEVER in MCP with public access.
6. **Missing indexes**: RLS policy on unindexed column = full table scan. 1M rows = timeout.
**Test**: Supabase Security Advisor built-in tool. Manual: `?select=*&id=gt.000...` with anon key.

### Rate Limiting Patterns
| Algorithm | Best For | BYMAV Use |
|-----------|----------|-----------|
| Token bucket | Burst-tolerant APIs | General API endpoints |
| Sliding window | Even distribution | Auth endpoints (strict) |
| Fixed window | Simple, low overhead | Internal/admin APIs |

Config: Auth=5 req/min/IP. API=100 req/min/userId. Burst=10 instant.
**Fail-closed**: Redis down = DENY. NEVER fail-open on auth. Response: 429 + `Retry-After`.

---

## 5. AUTHENTICATION — Best Practices

### better-auth (BYMAV Stack)
- Session-based. Cookie: `httpOnly`, `secure`, `sameSite: lax`.
- OAuth Google (shared client). PKCE + state MANDATORY. `checks:['none']` = VULNERABLE.
- `allowDangerousEmailAccountLinking: true` = account takeover. FORBIDDEN.

### OAuth 2.0 / PKCE Security
Source: [Doyensec](https://blog.doyensec.com/2025/01/30/oauth-common-vulnerabilities.html), [RFC 9700](https://datatracker.ietf.org/doc/rfc9700/)
- **PKCE downgrade**: server supports but doesn't enforce -> attacker removes code_challenge.
- **Redirect URI**: exact match ONLY. Wildcards = open redirect -> token theft.
- **Scope creep**: validate scopes server-side. Client can request broader than intended.
- **Token leakage**: browser history, Referer headers. Use Authorization Code + PKCE, NEVER Implicit.
- **OAuth 2.1**: mandatory PKCE, no Implicit flow. Migrate when better-auth supports.
- **Real breach**: Allianz Life Salesforce (Jul 2025) — malicious OAuth apps -> 1.1M records exposed.

### Session Management (Stuttard & Pinto)
- Session ID: min 128-bit entropy, cryptographically random.
- Regenerate after login (session fixation prevention).
- Timeout: idle 30min (sensitive), absolute 30d max.
- Invalidate server-side on logout (not just cookie deletion).

### Security Headers (Scott Helme)
Source: [scotthelme.co.uk](https://scotthelme.co.uk/tag/security-headers/)
```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```
Node.js: `helmet` package (14 middleware). Audit: securityheaders.com (free, grade A+).

---

## 6. LGPD + SUPPLY CHAIN

### LGPD ([ANPD 2024](https://www.breachrx.com/global-regulations-data-privacy-laws/lgpd/))
DPO mandatory (Rafael interim). Breach->ANPD: **3 working days** (2024 update). User notification: T+72h.
Consent before collection. Right to deletion + portability on request. Fines: 2% revenue, max R$50M.

### Supply Chain ([Snyk](https://snyk.io), [Socket.dev](https://socket.dev))
3,000+ malicious npm packages in 2024. SHA1-Hulud worm compromised chalk/debug (2.6B weekly downloads).
Vectors: typosquatting, dependency confusion, star-jacking. Only 24% of orgs confident in dep security.

**Defense**: `npm audit --production` (zero CRITICAL) | Socket.dev (behavior) | `trivy image` (containers) | lockfile integrity | `gitleaks` (secrets) | Node.js: `helmet`, avoid `eval()`/`child_process.exec()`, `express.json({limit:"1kb"})`.

---

## 7. INCIDENT RESPONSE + PR CHECKLIST

### Breach Timeline (LGPD + PagerDuty)
| Step | Deadline | Owner | Action |
|------|----------|-------|--------|
| Detect | T+0 | Alertmanager/SHIELD | Anomaly: unusual access, data exfil, cred leak |
| Triage | T+15min | SHIELD | Confirm. Classify: data types, users, vector. |
| Contain | T+30min | SHIELD+SENTINEL | Revoke tokens, block IPs, isolate. |
| ANPD | T+3 working days | Rafael (legal) | Mandatory for personal data. |
| Users | T+72h | Rafael+GROWTH | What happened, what data, what to do. |
| Postmortem | T+7d | SHIELD | Blameless 5 Whys. Action items with owners. |

**Rotation**: secret in git=rotate ALL keys | employee exit=revoke+rotate shared | quarterly=all API keys+DB passwords.

### PR Security Gate
**Automated** (security-audit.js): 1.IDOR(prisma w/o userId) 2.XSS(innerHTML w/o DOMPurify) 3.SSRF(fetch w/o allowlist) 4.SQLi($queryRawUnsafe) 5.INFO_LEAK(error.message to client) 6.MASS_ASSIGN(data:body w/o Zod)
**Manual** (SHIELD): 7.OAuth config 8.CSRF/SameSite 9.Secrets/gitleaks 10.npm audit 11.CSP/HSTS 12.RLS enabled+policies 13.File upload auth 14.Open redirect allowlist

---

## 8. OPERATING PRINCIPLES — Gold Rules

1. **Threat model before code** (Shostack). New feature = STRIDE pass. "What can go wrong?"
2. **Defense in depth** (Schneier). WAF + CSP + validation + auth + IDOR checks. No single layer.
3. **Least privilege** (NIST). uid 1000. Minimal DB perms. Scoped keys. RLS on every table.
4. **Fail closed** (Stuttard). Auth fail = deny. Rate limit fail = deny. NEVER fail open.
5. **Assume breach** (Zero Trust). Log everything. Detect fast. Contain faster.
6. **Proven crypto only** (Schneier). bcrypt/argon2. AES-256. NEVER roll your own.
7. **Secrets are radioactive** (Hunt). One leak = rotate everything. `pass` (GPG) only.
8. **Patch aggressively**. CVE-2025-29927 = hours to exploit. Days to patch = breach.
9. **Gold Standard** (BYMAV). How does Nubank/Inter protect this? Follow industry.
10. **Evidence over trust** (BYMAV). `security-audit.js --strict` PASS. `npm audit` clean. Words mean nothing.
