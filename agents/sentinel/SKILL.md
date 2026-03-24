# SENTINEL BRAIN — DevOps & Infrastructure Knowledge Base

> Dense reference for SENTINEL. Real people, real principles, actionable patterns.
> Max 200 lines. No fluff. English terms, PT-BR headers onde faz sentido.

---

## 1. GOLD STANDARD — Real References

### Books & Authors
| Ref | Author(s) | Core Idea | Apply at BYMAV |
|-----|-----------|-----------|----------------|
| **Site Reliability Engineering** (2016) | Beyer, Jones, Petoff, Murphy (Google) | SLIs/SLOs/error budgets, toil <=50%, blameless postmortems | SLIs for pixio.cloud (p99 <500ms, avail 99.9%). Track toil. |
| **The Phoenix Project** (2013) | Gene Kim et al. | Three Ways: flow, feedback, continual learning. WIP limits. | Limit concurrent infra changes. Every incident = postmortem. |
| **Release It!** 2nd ed (2018) | Michael Nygard | Circuit breakers, bulkheads, timeouts, steady state. Anti-patterns: cascading failures. | Timeouts on PostgreSQL/Stripe/Pluggy. Circuit breaker on flaky APIs. |
| **Systems Performance** 2nd ed (2020) | Brendan Gregg | USE method (Utilization/Saturation/Errors), flamegraphs. | USE for VPS. `docker stats` + /proc for containers. |
| **Observability Engineering** (2022) | Charity Majors, Fong-Jones, Miranda | High cardinality > averages. Wide events. Ask new questions without deploying. | Structured JSON logs: request_id, user_id, duration. |

### Practitioners
- **Kelsey Hightower** — Declarative config, health checks, graceful shutdown, immutable deploys. Minimalist.
- **Brendan Gregg** — USE: CPU(`mpstat`), Mem(`vmstat`), Disk(`iostat`), Net(`sar`). Measure FIRST.
- **Julia Evans** (b0rk) — Practical debugging. Measure first, hypothesize second.
- **Jessie Frazelle** — Container security. Minimal images, non-root, read-only fs, seccomp.

---

## 2. DORA METRICS — Real Benchmarks (2024 Report)

Source: [dora.dev](https://dora.dev/guides/dora-metrics/), [Accelerate State of DevOps 2024](https://dora.dev)

| Metric | Elite | High | Medium | Low | BYMAV Target |
|--------|-------|------|--------|-----|-------------|
| **Deploy frequency** | On-demand (multi/day) | 1/day-1/week | 1/week-1/month | 1/month-6/month | 1/day |
| **Lead time for changes** | <1 hour | 1 day-1 week | 1 week-1 month | 1-6 months | <24h |
| **Change failure rate** | 0-15% | 16-30% | 16-30% | >30% | <15% |
| **Failed deploy recovery** | <1 hour | <1 day | 1 day-1 week | >6 months | <1h |

2024 finding: AI adoption improves throughput BUT increases delivery instability. Validate automated changes.

---

## 3. DOCKER COMPOSE — Production Patterns (BYMAV Docker-First)

### Build (Multi-stage, non-root uid 1000)
```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build

FROM node:22-alpine AS runner
RUN adduser -D -u 1000 appuser
USER appuser
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "server.js"]
```

### Compose Patterns
| Pattern | Implementation | Why |
|---------|---------------|-----|
| Health checks | `test: ["CMD","curl","-f","http://localhost:3000"], interval: 30s, retries: 3, start_period: 40s` | Auto-restart unhealthy |
| Dependency ordering | `depends_on: { redis: { condition: service_healthy } }` | Never start before deps ready |
| Resource limits | `mem_limit: 4g`, `cpus: 2` | Prevent noisy neighbors (Gregg) |
| Restart policy | `restart: unless-stopped` | Self-healing. NOT `always` (masks crash loops) |
| Log rotation | `logging: { driver: json-file, options: { max-size: "50m", max-file: "5" } }` | Prevent disk full |
| Non-root | `user: "1000:1000"` | Least privilege (Frazelle) |
| Image pinning | `node:22-alpine` NOT `:latest` | Reproducible builds |
| Read-only root | `read_only: true` + `tmpfs: [/tmp]` | Immutable container fs |
| Network isolation | Named networks, `internal: true` for backend | Only expose what needs external |
| Docker+UFW fix | `127.0.0.1:5432:5432` NOT `5432:5432` | Docker bypasses UFW iptables |

### Zero-Downtime ([docker-rollout](https://github.com/wowu/docker-rollout))
`docker compose up -d --no-deps --scale web=2 web` -> health check -> remove old. Cloudflare Tunnel absorbs the swap.

### Anti-Patterns (PROIBIDO)
`:latest` in production | `privileged: true` | root inside container | state in container fs | `docker exec` in prod

---

## 4. MONITORING & OBSERVABILITY — Real Stack

### Current BYMAV Stack
| Pillar | Tool | Port | Status |
|--------|------|------|--------|
| Metrics | Prometheus | :9090 | 5 targets, 16 rules |
| Dashboards | Grafana | :9091 | Provisioned |
| Alerting | Alertmanager | :9093 -> Telegram | Active |
| Errors | Sentry | Cloud (free) | pixio.cloud |
| Logs | Docker JSON + journald | Local | Structured where possible |

### Prometheus Best Practices (source: [BetterStack](https://betterstack.com/community/guides/monitoring/prometheus-best-practices/))
1. Naming: lowercase + underscore. Include units: `_seconds`, `_bytes`, `_total`. Prefix with domain.
2. Avoid high-cardinality labels. `product_id` on 1M products = cardinality explosion. Use patterns.
3. Track totals+failures, NOT success+failure separately. `rate(failures[5m])/rate(total[5m])`.
4. Scope PromQL: always use label matchers. `{service="pixio-web"}`.
5. `for` clause on alerts (10m+). Prevents transient spike pages.
6. Initialize metrics to zero at startup. Prevents `no data` false alarms.
7. Preserve labels in alert rules. `{{ $labels.instance }}` in messages.
8. Scaling: federated shards or Thanos/Cortex when single-node hits limits.

### Alert Design (source: [Grafana docs](https://grafana.com/docs/grafana/latest/alerting/guides/best-practices/))
- Symptom-based > component-based. Users care about latency/errors, not pod restarts.
- Infrastructure alerts -> low-severity channels (Slack/dashboard). NOT paging.
- Fine-tune thresholds iteratively. Elite teams spend 30% effort on alert quality.
- Every alert must answer: what's wrong, what's the impact, what to do next.

### SLIs/SLOs (Google SRE)
| Service | SLI | SLO | Error Budget/mo |
|---------|-----|-----|-----------------|
| pixio.cloud | Availability | 99.9% | 43.8 min downtime |
| pixio.cloud | Latency p99 | <500ms | 0.1% can exceed |
| API | Error rate | <0.1% | 4,320 errors at 100k req/day |

---

## 5. CLOUDFLARE TUNNEL — Architecture

How it works: `cloudflared` daemon creates outbound-only TLS WebSocket connections to Cloudflare edge.
No inbound ports needed. Firewall blocks ALL inbound; traffic routes through Cloudflare's network.
BYMAV: container `cloudflared` in VPS compose → exposes pixio.cloud. Zero exposed ports.

Best practices:
- Run replica `cloudflared` on second host for HA.
- `No TLS Verify` between cloudflared<->origin (both localhost). User<->Cloudflare = full TLS.
- Standardize public hostnames (subdomain pattern).
- Enable Cloudflare dashboard notifications for tunnel health.
- Strip `x-middleware-subrequest` header at Cloudflare level (CVE-2025-29927 mitigation).

---

## 6. TAILSCALE — Secure Access (VPS Admin)

BYMAV: SSH to VPS via Tailscale (`100.111.104.15`). Public IP firewalled. Zero Trust by default.
Source: [Tailscale security hardening](https://tailscale.com/kb/1196/security-hardening)

Hardening checklist:
- [x] MFA in identity provider (hardware tokens preferred)
- [x] Principle of least privilege: role-based ACLs, NOT IP-based
- [ ] Node key expiry (default 180d) — consider shorter for production nodes
- [ ] Device approval before joining tailnet
- [x] Offboard immediately on departure (revoke keys+sessions)
- [ ] SSH session recording to S3/dedicated node
- [ ] Network flow logs for audit
- [ ] GitOps for ACL policy changes (version-controlled, reviewed)

---

## 7. INCIDENT RESPONSE — PagerDuty Framework Adapted

Source: [response.pagerduty.com](https://response.pagerduty.com/before/severity_levels/)

| Level | Impact | Detect->Respond | Resolve | Notify |
|-------|--------|-----------------|---------|--------|
| **P1** | Site down, data loss, breach | 5 min | 30 min | Telegram Rafael IMMEDIATELY |
| **P2** | Core feature broken (auth, txns) | 15 min | 2h | Telegram ATLAS |
| **P3** | Perf degraded, non-core broken | 1h | 24h | Log + next standup |
| **P4** | Cosmetic, minor | 4h | 1 week | Linear backlog |

**Rule: If unsure, treat as higher severity. Reassess in postmortem.**

Timeline: `DETECT -> TRIAGE -> MITIGATE -> RESOLVE -> POSTMORTEM`
Mitigate = stop bleeding (rollback/restart). Postmortem = blameless 5 Whys + action items with owners.

### Quick Runbooks
| Scenario | First Action | Escalate If |
|----------|-------------|-------------|
| Container crash loop | `docker logs --tail 50` -> fix -> restart | 3+ restarts in 10 min |
| Disk >90% | `docker system prune -af` + `journalctl --vacuum-size=500M` | Still >85% after |
| High CPU >95% | `docker stats` -> identify offender -> restart | Sustained >15 min |
| Memory leak | Restart (mitigate) -> investigate heap | Recurs after restart |
| DB timeout | Verificar postgres container -> pool config -> restart app | docker logs postgres |

---

## 8. BACKUP & HARDENING

**3-2-1 Rule**: 3 copies, 2 media types, 1 offsite. Primary=VPS volumes. Local=`/backups/` hourly cron. Offsite=Rclone->GDrive daily.
PostgreSQL: pg_dump diario + WAL archiving (4 camadas). Ver database-topology.md. Quarterly restore drill MANDATORY.

**Linux Hardening** ([ServerMania](https://www.servermania.com/kb/articles/linux-server-hardening)):
- [x] SSH key-only, `PermitRootLogin no`, `MaxAuthTries 3` | [x] fail2ban (5 attempts/10min->ban)
- [x] UFW 22/80/443 only, Docker bound to 127.0.0.1 | [x] `unattended-upgrades` enabled
- [ ] `auditd` on sensitive paths | [ ] `sysctl` hardening (`rp_filter=1`)

**Cost** (stay <R$700/mo): Right-size containers weekly. Cloudflare cache static 1y. PgBouncer. Log rotation 50m/5 files. `docker system prune` weekly. Measure 2wk before upgrading.

---

## 9. OPERATING PRINCIPLES

1. **Measure, don't guess** (Gregg). `docker stats` before changing anything.
2. **Cattle, not pets** (Hightower). Broken = kill and recreate. Never SSH to "fix".
3. **Blast radius** (Nygard). Smallest scope per change. Bulkhead pattern.
4. **Error budgets** (SRE). 99.9% = 43.8 min/month allowed downtime.
5. **Toil elimination** (SRE). Done >2x = automate.
6. **2 Strikes** (BYMAV). Same problem twice = change approach.
7. **Docker-First** (BYMAV). NEVER PM2/systemd. Compose ALWAYS. uid 1000.
8. **main-only on prod** (BYMAV). NEVER feature branches in production.
9. **Fail fast, recover faster** (Nygard). Timeouts + circuit breakers everywhere.
10. **Defense in depth**. WAF + UFW + container isolation + app auth. No single layer.

## 10. TOPOLOGIA REAL — O QUE RODA ONDE (atualizado 05/03/2026)

### VPS Hostinger (EPYC 8vCPU, 32GB RAM, 387GB disco, SEM GPU, IP fixo 76.13.163.35)
**Papel: PRODUÇÃO. 24/7. Estático.**
| Serviço | Tipo | Container | Porta | RAM |
|---------|------|-----------|-------|-----|
| pixio-web | Web prod | Docker | 3000 | 350MB |
| vita-web | Web prod | Docker | 3110 | 115MB |
| postgres | Banco prod | Docker | 5432 | 115MB |
| redis | Cache | Docker | 6379 | 16MB |
| cloudflared | Tunnel | Docker | — | 44MB |
| auth-gateway | Auth | Docker | 8769 | 55MB |
| n8n | Automação | Docker | 5678 | 300MB |
| pixio-hub + worker | Orquestrador prod | Docker | 8767 | 870MB+135MB |
| terminal-server | Acesso remoto | Docker | — | 52MB |
| slimfy-bot | Bot e-commerce | Docker | — | 79MB |
| arena-tunnel | SSH Dublin bots | systemd | 18797-18798 | leve |
| mcp-proxy | Gateway MCP | systemd | 9500 | — |
| lab-scanner | Polymarket scan | systemd | 8780 | 1GB (TODO: mover pro Desktop) |
| lab-ingest | DuckDB ingest | systemd | — | 53MB (TODO: mover pro Desktop) |

**Load alvo: < 2.0. Disco alvo: < 70%.**

### Desktop (Ryzen 7800X3D 8C/16T, 32GB RAM, RTX 5070 12GB, 931GB NVMe)
**Papel: DEV + GPU + AGENTES. Tolera downtime.**
**Disco:** / (390GB, ~79GB livres) + /mnt/polygon-node (541GB LVM, ~214GB livres). **Total ~293GB livres.**
| Serviço | Tipo | Container/systemd | Notas |
|---------|------|-------------------|-------|
| 7 sessões Claude | Agentes (tmux) | tmux | ATLAS, JARVIS, KEEPER, APEX, DROID, etc |
| pixio-hub + worker | Orquestrador dev | Docker | Instância dev separada |
| postgres-dev | Banco dev | Docker | 5432 |
| redis | Cache dev | Docker | 6379 |
| telegram-bot | Bot Telegram | Docker | USA GPU (Whisper+TTS). TODO: avaliar mover texto pro VPS |
| MCP proxy | Gateway MCP | systemd | 9500 |
| brave-agents | Browser headless | systemd | GPU |
| memory-guardian | Monitor RAM | systemd | Kill antes de OOM |
| syncthing | Sync files | systemd | 22000 |
| emulador Android | Dev mobile | manual | GPU (QEMU) |
| Ollama | LLM local | systemd (DESABILITADO) | Desabilitado 05/03 — causa OOM loop |

**GPU livre: ~97% idle. Load alvo: < 4.0.**

### Dublin AWS (i-0eba7cf7442175435, Ubuntu, 54.73.37.74)
**Papel: TRADING BOTS. Gerenciado por KRAKEN/KEEPER.**
- raven_0x8_v1.service (dashboard :8797)
- Watchdog 24/7
- SENTINEL NÃO altera sem coordenar com KRAKEN

### REGRA: 1 LUGAR POR SERVIÇO
Se roda no VPS → NÃO roda no Desktop (exceção: Redis instâncias diferentes, Hub dev vs prod).

---

## 11. INFRA GOVERNANCE — REGRAS PERMANENTES (05/03/2026)
Source: /home/mav/agents/tasks/JARVIS-INFRA-CLEANUP.md | Full: memory/infra-governance.md

1. **RESOURCE LIMITS OBRIGATÓRIOS**: Todo container → mem_limit + cpus + restart: unless-stopped
2. **CAPACITY CHECK ANTES DE DEPLOY**: Sobra 30% RAM depois? Se não → NÃO SOBE
3. **BUDGET VPS**: 22/32 GB alocado. pixio-web(4GB), vita-web(2GB), hub(1GB), redis(512MB), runners(1GB)
4. **BUDGET LOCAL**: 17/32 GB alocado. KDE(2GB), Brave(4GB), Claude(2GB), pixio-dev(4GB), emulador(4GB)
5. **1 LUGAR POR SERVIÇO**: Se roda no VPS → NÃO roda local. Exceção: Redis (instâncias diferentes)
6. **MONITORAMENTO < MONITORADO**: cadvisor/prometheus/grafana/alertmanager BANIDOS. Usar docker stats + healthcheck.sh
7. **DUBLIN = KRAKEN**: SENTINEL NÃO altera nada em 54.73.37.74 sem coordenar com KRAKEN
8. **CONTAINERS MORTOS** (NÃO devem voltar): grafana, prometheus, alertmanager, exporters, n8n, tambo, wraith, igwa, dev-preview, uptime-kuma, cadvisor


## Licoes Aprendidas (auto-feedback)
- [2026-03-21] [token_burn] Gastou 3007k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [exploration_no_output] Leitura excessiva sem editar = tunnel vision de exploração. Max 5 reads antes de produzir algo. Faca plano curto e execute.
- [2026-03-12] [token_burn] Gastou 135k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 448k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [edit_without_read] SEMPRE ler o arquivo antes de editar. Edit sem Read = edit cego = erros evitaveis.
