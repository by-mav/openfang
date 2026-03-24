# BYMAV × OpenFang Integration Plan

## Visão
Usar o OpenFang como motor (Rust backend) e manter nosso frontend visual (Cockpit/ReactFlow canvas com galáxias, cards arrastáveis, delegações). O resultado: performance de Rust + visual BYMAV + toda feature do OpenFang.

## Mapeamento: O que cada sistema tem

### OpenFang (motor Rust) — VAMOS USAR TUDO
| Componente | O que faz | Nosso equivalente |
|---|---|---|
| **openfang-kernel** | Lifecycle, registry, scheduler, RBAC, metering, event_bus, heartbeat, supervisor, workflows, triggers, cron | Hub `/api/agents` (básico) |
| **openfang-runtime** | Agent loop, LLM drivers (Anthropic+OpenAI+Gemini+7 mais), MCP, tools, sandbox, compactor, hooks, browser, web_search, shell_exec | Claude Code CLI + Codex CLI |
| **openfang-api** | 180 route handlers, REST+WS+SSE, OpenAI-compat, session auth, rate limiter | Hub Next.js API routes (~80) |
| **openfang-channels** | 40 adaptadores (Telegram, Discord, Slack, WhatsApp, etc) | Telegram bot (1 canal) |
| **openfang-memory** | SQLite + vector embeddings + decay | pass + JSONL + .md files |
| **openfang-cli** | TUI Ratatui com 18 telas (Dashboard, Agents, Chat, Sessions, etc) | Nada equivalente |
| **openfang-desktop** | Tauri 2.0 app nativo com tray, shortcuts, updater | Nada equivalente |
| **openfang-skills** | Runtime skill injection via SKILL.md | agents/knowledge/*_BRAIN.md |
| **openfang-types** | Shared types | TypeScript types |

### BYMAV Hub (frontend visual) — VAMOS MANTER
| Componente | O que faz | Manter? |
|---|---|---|
| **Cockpit.tsx** | ReactFlow canvas com galáxias, stars, edges animados | ✅ SIM |
| **BridgeVpNode** | Cards VP com glow, cor, estado, métricas | ✅ SIM |
| **BridgeDivisionNode** | Cards divisão | ✅ SIM |
| **BridgeAgentNode** | Cards agente worker | ✅ SIM |
| **AnimatedParticleEdge** | Partículas animadas nas conexões | ✅ SIM |
| **GoldenArcEdge/CurvedGlowEdge** | Edges brilhantes | ✅ SIM |
| **DraggablePanel** | Painéis arrastáveis | ✅ SIM |
| **DelegationsPanel** | Ver delegações subindo/descendo | ✅ SIM |
| **DelegationTimeline** | Timeline de delegações | ✅ SIM |
| **SystemMonitorPanel** | CPU/RAM/GPU monitor | ✅ SIM |
| **FloatingChat** | Chat popup com agente | ✅ SIM |
| **FloatingTerminal** | Terminal embutido | ✅ SIM |
| **TokenTracker** | Tracking de tokens/custo | ✅ SIM |
| **CeoStatsBar** | Barra de stats do Rafael | ✅ SIM |

## Plano de Integração (5 fases)

### Fase 1: Boot OpenFang com nossos agentes (Dia 1-2)
- [ ] Substituir `agents/` templates pelos nossos 48 agentes
- [ ] Cada agente → `agent.toml` com name, model, system_prompt, tools, capabilities
- [ ] Mapear hierarquia: orchestrator=ATLAS, capabilities.agent_message por VP→Dir→Worker
- [ ] Configurar `openfang.toml` com nossas API keys (Anthropic, OpenAI)
- [ ] Configurar MCP servers (os 33 que temos)
- [ ] `cargo build --release` e rodar: `./openfang start`
- [ ] Verificar: dashboard TUI funciona, agentes listam, chat funciona

### Fase 2: Conectar frontend BYMAV ao backend OpenFang (Dia 3-5)
- [ ] Trocar fetch do Hub Next.js → fetch do OpenFang API (:4200)
- [ ] Mapear endpoints: `/api/agents` → OpenFang `/v1/agents`
- [ ] WebSocket: nosso useEventStream → OpenFang SSE/WS events
- [ ] Delegações: `agent_send` do OpenFang = nosso `/api/agents/delegate`
- [ ] Status/heartbeat: OpenFang já tem, conectar nos cards
- [ ] Chat: FloatingChat → OpenFang `/v1/chat/completions` (OpenAI-compat)

### Fase 3: Canais (Dia 6-7)
- [ ] Telegram: configurar token no `openfang.toml` [telegram]
- [ ] WhatsApp: configurar gateway
- [ ] Cada canal mapeia pra um agente ou routing rule

### Fase 4: Desktop App (Dia 8-9)
- [ ] Tauri app carrega nosso frontend ReactFlow
- [ ] Tray icon com status dos agentes
- [ ] Global shortcuts (ex: Ctrl+Shift+A = abrir ATLAS)
- [ ] Auto-start com Windows/Linux

### Fase 5: Merge TUI + Visual (Dia 10)
- [ ] TUI do OpenFang (terminal) mostra nossos agentes com cores
- [ ] `openfang tui` = cmd-center turbinado
- [ ] Quick launcher integrado

## Drivers LLM já prontos no OpenFang
- `drivers/anthropic.rs` → Claude (Opus, Sonnet, Haiku)
- `drivers/openai.rs` → GPT-4o, o3
- `drivers/gemini.rs` → Gemini
- `drivers/claude_code.rs` → Claude Code CLI integration!
- `drivers/qwen_code.rs` → Qwen
- `drivers/copilot.rs` → GitHub Copilot
- `drivers/fallback.rs` → Auto-fallback entre providers

## O que NÃO precisamos recriar
- ❌ Sistema de delegação (OpenFang tem agent_send/agent_spawn)
- ❌ Metering/custos (OpenFang tem built-in)
- ❌ RBAC/permissões (OpenFang tem capabilities system)
- ❌ Memory/embeddings (OpenFang tem SQLite + vectors)
- ❌ MCP integration (OpenFang suporta nativamente)
- ❌ 40 canais de comunicação (prontos)
- ❌ TUI dashboard (pronto, 18 telas)
- ❌ Desktop app (Tauri pronto)
- ❌ Sandbox/segurança (16 sistemas de segurança)

## O que MANTEMOS nosso
- ✅ Frontend visual (ReactFlow canvas, galáxias, glow)
- ✅ Hierarquia 3-9-27 (policies, brains)
- ✅ Branding BYMAV (cores, nomes, identidade)
- ✅ Linear integration
- ✅ GitHub integration
- ✅ Hooks v2 system

## Resultado Final
Rafael abre o desktop app → vê o canvas com galáxias e todos 48 agentes.
Clica num agente → chat direto. Arrasta painéis. Vê delegações em tempo real.
Terminal: `openfang tui` → dashboard Ratatui com 18 telas.
Notebook: mesmo app via Tailscale.
Celular: Telegram conversa com qualquer agente.
