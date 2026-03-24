# SWIFT BRAIN — iOS Specialist Knowledge Base
# Role: Native iOS developer at BYMAV. 100% SwiftUI. No cross-platform.
# Prereqs: APEX_BRAIN.md (mobile architecture), Apple HIG
# This file: KNOWLEDGE. Swift/iOS patterns, Apple conventions, VitaAI architecture.
# Max 200 lines. Dense. No fluff.

## 0. MANDATORY — BUILD + VER (READ FIRST)
# DEPOIS de implementar qualquer tela:
#   1. xcodebuild (ou xcodegen + xcodebuild) — DEVE compilar sem erro
#   2. Quando Mac Mini chegar: xcrun simctl io booted screenshot → OLHAR
#   3. Ate la: SwiftUI Preview mental check — structs conformam a View?
#   4. Se nao compilar, CONSERTAR antes de reportar
# AUTOCRITICA: "segue Apple HIG? Um reviewer da App Store aprovaria?"

---

## 1. REAL PEOPLE & REFERENCES
- **Holly Borla** — Swift Language team lead. Swift 6 concurrency, Sendable.
- **Josh Shaffer** — SwiftUI framework lead at Apple.
- **Antoine van der Lee** (SwiftLee) — Xcode optimization. avanderlee.com
- **Paul Hudson** (Hacking with Swift) — SwiftUI, Swift evolution.
- Refs: developer.apple.com/design/human-interface-guidelines | developer.apple.com/app-store/review/guidelines
- Build times: github.com/fastred/Optimizing-Swift-Build-Times

---

## 2. SWIFT 6 + ASYNC/AWAIT
- **Strict concurrency**: Sendable checking enforced. Data crossing boundaries must be Sendable.
- **Actor isolation**: MainActor for UI. Custom actors for data isolation.
- **Structured concurrency**: TaskGroup, async let, withThrowingTaskGroup.
- **Observation** (iOS 17+): `@Observable` macro replaces ObservableObject. Use @State for VM.
- **async let**: parallel API calls in ViewModels (e.g., `async let progressTask`, `async let examsTask`).

---

## 3. VITAAI ARCHITECTURE PATTERNS
| Pattern | Implementation |
|---------|---------------|
| ViewModels | `@Observable class XViewModel` — @State in View |
| API calls | `APIClient.shared.get/post/stream` with async/await |
| Local persistence | SwiftData `@Model` + `@Query` in views |
| Auth state | `AuthManager` with loading/authenticated/unauthenticated/onboarding |
| Navigation | `NavigationStack` within each tab (no global Router) |
| Tab bar | `VitaTab` enum, custom tab bar in `ContentView` |
| Streaming | `APIClient.shared.stream()` for SSE (AI chat) |
| Design tokens | `VitaColors.*` for colors, `VitaTypography.*` for fonts |
| Components | `VitaComponents.swift`: glassCard, cardStyle, skeleton, chipStyle |
| Secrets | Keychain via `KeychainHelper` — NEVER UserDefaults |
| Build | XcodeGen (`project.yml`) — auto-includes all files under VitaAI/ |

### File Organization
```
VitaAI/
  App/                    # VitaAIApp.swift, ContentView.swift, Config.swift
  Core/
    Design/               # VitaColors, VitaTypography, VitaComponents, VitaTokens
    Models/               # Codable structs (UserProfile, Conversation, Simulado, etc.)
    Network/              # APIClient, Endpoints, AuthManager, KeychainHelper
  Features/
    Dashboard/            # DashboardView, ProgressView
    Chat/                 # ChatView (inline VM, sidebar, attachments, feedback)
    Estudos/              # EstudosView (tabs: Disciplinas, Notebooks, Flashcards, PDFs)
    Flashcards/           # FlashcardReviewView, FlashcardStatsView
    Simulados/            # SimuladoListView, SimuladoView, SimuladoResultView, DiagnosticsView
    MindMap/              # MindMapListView, MindMapEditorView, MindMapModels
    Trabalhos/            # TrabalhosView, Editor/AssignmentEditorView
    Agenda/               # AgendaView
    Profile/              # ProfileView, SettingsView
    Widgets/              # StudyWidget (WidgetKit)
```

---

## 4. APP STORE REVIEW — REJECTION AVOIDANCE

### Top Reasons (2025-2026)
1. **Privacy violations** (#1) — missing/vague privacy policy, ATT non-compliance
2. **Crashes/broken flows** (2.1) — test clean install + slow network
3. **Misleading metadata** (2.3) — screenshots must match real app
4. **IAP violations** (3.1.1) — digital goods MUST use Apple IAP
5. **Missing account deletion** — required if accounts exist
6. **AI data sharing** — explicit disclosure + user permission required
7. **Incomplete app** — placeholder text = instant reject

### VitaAI Health App Requirements
- HealthKit entitlement in provisioning profile
- Purpose strings for EVERY health data type in Info.plist
- Privacy policy: what data, why, how stored, who sees it
- Demo account in Review Notes for gated features
- Starting **April 2026**: iOS 26 SDK required for all submissions

---

## 5. KEYCHAIN / SECURE STORAGE
- `KeychainHelper` wrapper around Security framework (`kSecClassGenericPassword`).
- Persists across reinstalls (same bundle ID). Design for stale data.
- `kSecAttrAccessibleWhenUnlocked` (default). `AfterFirstUnlock` for background tasks.
- Biometric: `kSecAccessControlBiometryCurrentSet`.
- Keychain sharing via app groups (main app + widget extension).
- NEVER store auth tokens in UserDefaults.

---

## 6. HEALTHKIT (VitaAI)
- Native `HKHealthStore` — no third-party wrappers needed.
- Request permissions progressively (when user navigates to relevant screen).
- Background delivery: `enableBackgroundDelivery(for:frequency:)`.
- Observer queries: real-time health data via `HKObserverQuery`.
- 100+ data types: steps, heart rate, sleep, nutrition, workouts, lab results.
- Never cache health data longer than necessary. Privacy-first.

---

## 7. SWIFTDATA (iOS 17+)
- `@Model` macro for entity classes. Automatic schema generation.
- `@Query` in views for automatic list updates with sort/filter.
- `ModelContainer` configured in `VitaAIApp.swift`.
- VitaAI entities: `MindMapEntity`, `LocalAssignmentEntity`.
- JSON-serialized complex fields: `nodesJson: String` with computed decode/encode.
- Context: `@Environment(\.modelContext)` for CRUD operations.

---

## 8. SWIFT CHARTS (iOS 16+)
- `BarMark`, `LineMark`, `AreaMark`, `SectorMark`, `PointMark`.
- Interpolation: `.catmullRom` for smooth curves.
- `foregroundStyle(by:)` for color-coded series.
- Custom `chartXAxis`/`chartYAxis` for label formatting.
- Used in: SimuladoDiagnosticsView, FlashcardStatsView.

---

## 9. STOREKIT 2 — IN-APP PURCHASES
- `Product.products(for: ids)` — fetch. `product.purchase()` — async + built-in verification.
- `Transaction.currentEntitlements` — check subscriptions.
- `Transaction.updates` — listen for state changes.
- StoreKit Testing in Xcode: local testing without TestFlight.
- Never hardcode prices. Use `product.displayPrice`.

---

## 10. XCODE BUILD OPTIMIZATION
- **Build Active Architecture Only = Yes** (Debug). Single arch.
- **XcodeGen**: `project.yml` generates `.xcodeproj`. Auto-includes sources.
- **Xcode 26 compilation caching** (opt-in): **30% faster builds**.
- Break complex SwiftUI views into subviews (compiler bottleneck).
- Flag: `-Xfrontend -warn-long-expression-type-checking=100` to find slow code.

---

## 11. ANTI-PATTERNS
| Anti-Pattern | Do Instead |
|---|---|
| Auth tokens in UserDefaults | KeychainHelper (Security framework) |
| Color(red:) / Color(hex:) literals | VitaColors.* semantic tokens |
| .font(.system(size:)) everywhere | VitaTypography.body/display/headline |
| All permissions on launch | Progressive, at point of use |
| Hardcode IAP prices | product.displayPrice from StoreKit |
| Cache health data long | Minimal retention, privacy-first |
| Giant SwiftUI view bodies | Small subviews (compiler perf) |
| ObservableObject + @Published | @Observable macro (iOS 17+) |
| Global Router with enum | NavigationStack per tab |
| Separate ViewModel files for small features | Inline @Observable in same file |

---

## 12. BYMAV-SPECIFIC
### VitaAI iOS (com.bymav.vitaai)
- Bundle ID: `com.bymav.vitaai`. Deep link: `vitaai://`.
- Backend: `https://vita-ai.cloud` (production).
- iOS 17+ minimum. SwiftUI only. Zero UIKit (except ASWebAuthenticationSession, PDFKit).
- HealthKit: steps, heart rate, sleep, nutrition.
- Biometric: FaceID + Keychain-backed secure storage.
- Push: native UserNotifications framework.
- Offline: queue entries, sync online.
- ATT: implement, don't gate. Expect 70% decline.

### DPFP (Dual-Platform Feature Protocol)
- Every mobile feature ships on BOTH platforms simultaneously.
- SWIFT reads DROID's code as reference, implements iOS-native equivalent.
- Deliverables: xcodebuild PASS + screenshots (light/dark) + PR.
- See: /home/mav/agents/policies/DPFP.yaml

---

## 13. TEST ACCOUNTS + SCREENSHOT RULES (OBRIGATORIO)

### CONTAS DE TESTE — USAR SEMPRE
**VITA (vita-ai.cloud):**
- QA: `qa@vita-ai.cloud` / `VitaTest2026` (residencia yr6, ENARE, dados seedados)
- Atlas: `atlas@vita-ai.cloud` / `AtlasTest2026` (graduacao yr3)
- Login: tela de login → Email → preencher email+senha → entrar

**PIXIO (pixio.cloud):**
- QA: `test@pixio.cloud` / `Test123!` (pro, dados completos)

### REGRA HARD: LOGAR ANTES DE QUALQUER SCREENSHOT
# NUNCA mandar screenshot de tela de login/splash/onboarding.
# SEMPRE: 1. Logar na conta teste 2. Navegar ate a tela 3. SO ENTAO screenshot
# Se screenshot mostra login screen = INVALIDO. REFAZER.
# Simulador iOS: xcrun simctl → abrir app → preencher login → navegar → screenshot
# Quando Mac Mini chegar, automatizar com XCTest UI.


## Licoes Aprendidas (auto-feedback)
- [2026-03-21] [token_burn] Gastou 936k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 1249k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 7298k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 1080k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 4445k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 6679k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 2528k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 7733k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 6222k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 3004k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 2487k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 2870k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [edit_without_read] SEMPRE ler o arquivo antes de editar. Edit sem Read = edit cego = erros evitaveis.
- [2026-03-12] [exploration_no_output] Leitura excessiva sem editar = tunnel vision de exploração. Max 5 reads antes de produzir algo. Faca plano curto e execute.
