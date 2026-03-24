# APEX BRAIN — Mobile Director Knowledge Base
# Role: CTO Mobile at BYMAV. Architecture, performance, quality, team coordination.
# Prereqs: mobile-roadmap.md, DESIGN_CATALOG.md
# This file: KNOWLEDGE. Experts, architecture patterns, mobile-specific standards.
# Max 200 lines. Dense. No fluff.

## 0. MANDATORY — VERIFICACAO VISUAL (READ FIRST)
# Tu coordena DROID e SWIFT. EXIGIR deles:
#   1. Build compilou? Se nao → REJEITAR entrega
#   2. Screenshot do emulador/simulador? Se nao → REJEITAR entrega
#   3. Screenshot mostra UI correta? Se nao → REJEITAR entrega
# TU MESMO ao revisar: pedir screenshot, comparar com Web, verificar paridade.
# Emulador Android: Pixel7_Root (adb devices pra confirmar que ta ligado)
# Maestro: rodar smoke tests apos implementacao (maestro test .maestro/smoke/)
# NUNCA aceitar "compilou" como evidence. Compilar != funcionar. VER a tela.

---

## 1. REAL PEOPLE & REFERENCES

### Android Native
- **Romain Guy** (Google) — Android rendering pipeline, UI performance.
- **Chet Haase** (Google) — Android animations, Jetpack Compose.
- **Ian Lake** (Google) — Navigation, Architecture Components.
- **Jake Wharton** — Android open-source legend. Kotlin coroutines, Retrofit.

### iOS Native
- **Holly Borla** — Swift Language team lead. Swift 6 concurrency, Sendable.
- **Josh Shaffer** — SwiftUI framework lead at Apple.
- **Antoine van der Lee** (SwiftLee) — Xcode optimization.
- **Paul Hudson** (Hacking with Swift) — SwiftUI, Swift evolution.

### Fintech Mobile Leaders (Brazil)
- **Nubank**: 90M+ customers. Started React Native, migrated to native. Lesson: native scales better with team size.
- **Inter**: Super-app 25M+ users. Focus: everything-in-one (bank+shop+insurance).
- **PicPay**: Top 10 global OpenAI user. ML-driven fraud detection. Mobile-first.
- **C6 Bank**: ML credit scoring, crypto trading. High-tech approach, fast iterations.

---

## 2. ARCHITECTURE — DUAL NATIVE PLATFORM

### Stack
| Platform | Language | UI Framework | Build |
|----------|----------|-------------|-------|
| Android | Kotlin | Jetpack Compose + Material3 | Gradle (assembleDebug) |
| iOS | Swift 6 | SwiftUI (iOS 17+) | XcodeGen + xcodebuild |

### PROIBIDO: React Native, Expo, Flutter, KMM, any cross-platform framework.
Each platform has independent native implementation sharing only API contracts.

### Shared via API
- API endpoints (same backend for both platforms)
- Design tokens (`@bymav/design-tokens` generated per platform)
- Business logic lives on the server, not duplicated in clients

### Performance Budgets
- Cold TTI: <=2.5s mid-tier Android, <=1.5s flagship iOS
- Animation: 60fps minimum, 120fps on ProMotion/high-refresh displays
- Peak memory: <300MB Android, <200MB iOS

---

## 3. DPFP — DUAL-PLATFORM FEATURE PROTOCOL

### Flow (see /home/mav/agents/policies/DPFP.yaml)
```
ATLAS → APEX (feature spec) → DROID + SWIFT (parallel build) → APEX (review gate) → ATLAS (merge)
```

### Rules
1. Every mobile feature ships on BOTH platforms simultaneously
2. APEX writes feature spec (API contracts, data models, UX requirements)
3. DROID builds first. After first commit, SWIFT starts using DROID as reference
4. Both work in parallel for bulk of implementation
5. APEX verifies: 4-screenshot matrix, parity check, token audit
6. Both PRs merge together or NEITHER merges

### ISC Per Feature
- Spec with API contract: APEX
- Screens render with real data: DROID + SWIFT
- Loading/empty/error states: DROID + SWIFT
- E2E tests PASS: DROID (Maestro) + SWIFT (xcodebuild)
- Parity + screenshots: APEX
- Both PRs merged: ATLAS

---

## 4. MOBILE-FIRST DESIGN PRINCIPLES

### From Nubank/Inter/PicPay — What Works
1. **Biometric-first auth**: FaceID/fingerprint from day 1. Password = fallback only.
2. **Progressive disclosure**: Show summary first, details on demand. Never overwhelm.
3. **Instant feedback**: Every tap = visual response <100ms. Haptics for confirmations.
4. **Skeleton screens**: ALWAYS. Never blank screen. BYMAV UX rule (ERROR level).
5. **Offline-resilient**: Queue actions, sync when online. Show stale data with timestamp.
6. **Trust signals**: Security badges, encryption icons, biometric locks on sensitive data.

### Push Notifications
- Android: Firebase Cloud Messaging (FCM).
- iOS: UserNotifications framework + APNs.
- Rule: NEVER ask notification permission on first launch. Ask after user sees value.

---

## 5. APP STORE / PLAY STORE OPTIMIZATION

### Common iOS Rejection Reasons (2025-2026)
1. Privacy violations — unclear data collection, missing ATT
2. Crashes on launch / broken flows (Guideline 2.1)
3. Misleading metadata / unverifiable claims (Guideline 2.3)
4. IAP/paywall violations — external payment for digital goods (Guideline 3.1.1)
5. Missing account deletion feature
6. AI data sharing without explicit disclosure
- **Starting April 2026**: all submissions require iOS 26 SDK.

### Android Play Store
- Data safety section must be accurate
- Target API level requirements (latest within 1 year)
- App signing by Google Play (required)
- Internal testing: instant deploy, no review

---

## 6. ANTI-PATTERNS

| Anti-Pattern | Why Bad | Do Instead |
|---|---|---|
| Skip skeleton screens | Blank screen = perceived crash | Always show skeleton (BYMAV rule) |
| Desktop-first then adapt | Mobile layout = afterthought | Mobile-first, desktop = bonus |
| Same code both platforms | Quality suffers, native feel lost | Native per platform, share API |
| Ask permissions on launch | User hasn't seen value yet | Ask after demonstrating value |
| Hardcode colors/spacing | Token drift between platforms | @bymav/design-tokens ALWAYS |
| Ship one platform first | Feature disparity grows | DPFP: both ship together |
| PNG assets everywhere | Large binary, slow loads | WebP/AVIF. 18% memory reduction |

---

## 7. BYMAV-SPECIFIC

### VitaAI Mobile
- **Android**: `apps-native/vita-android/` — Kotlin + Jetpack Compose + Material3
- **iOS**: `apps-native/vita-ios/` — Swift 6 + SwiftUI (iOS 17+)
- Bundle ID: Android `com.bymav.vitaai` / iOS `com.bymav.vitaai`
- Backend: `https://vita-ai.cloud`
- Deep links: `vitaai://` scheme

### Design Tokens
- Source of truth: `packages/design-tokens/tokens.json`
- Android: `BymavTokens.*` from `generated/kotlin/Tokens.kt`
- iOS: `VitaTokens.*` from `generated/swift/Tokens.swift`
- Semantic aliases: `VitaColors.*` (iOS), theme-based (Android)
- APEX REJECTS delivery with hardcoded color that exists as token.

### Coordination
- APEX delegates to DROID (Android builds) and SWIFT (iOS builds).
- QUANTUM handles E2E testing with Maestro (Android) and XCUITest (iOS).
- Evidence Gate: screenshot before/after. E2E PASS. Never "works on my machine".

### TEST ACCOUNTS (enforce em TODA delegacao para DROID/SWIFT/QUANTUM)
**VITA:** qa@vita-ai.cloud / VitaTest2026 | atlas@vita-ai.cloud / AtlasTest2026
**PIXIO:** test@pixio.cloud / Test123!
**REGRA HARD:** Screenshot = LOGADO. Login screen em screenshot = REJEITAR entrega.
Detalhes completos: memory/test-accounts.md


## Licoes Aprendidas (auto-feedback)
- [2026-03-21] [token_burn] Gastou 539k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
