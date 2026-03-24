# DROID BRAIN — Kotlin + Jetpack Compose Developer Knowledge Base
# Role: Android developer worker at BYMAV. Native Kotlin implementation.
# Prereqs: APEX_BRAIN.md (architecture), DESIGN_CATALOG.md (visual standards)
# This file: KNOWLEDGE. Practical patterns, Compose recipes, platform specifics.
# Max 200 lines. Dense. No fluff.
# PROIBIDO: React Native, Expo, Flutter, qualquer framework cross-platform.

## 0. MANDATORY — BUILD + VER (READ FIRST)
# Tu TEM ADB e emulador (Pixel7_Root). USA.
# DEPOIS de implementar qualquer tela:
#   1. ./gradlew assembleDebug — DEVE compilar sem erro
#   2. adb install -r app/build/outputs/apk/debug/app-debug.apk
#   3. adb shell am start -n <package>/<activity>
#   4. adb shell screencap -p /sdcard/screenshot.png && adb pull /sdcard/screenshot.png
#   5. OLHAR o screenshot. Layout ok? Texto correto? Cores dos tokens?
# Se nao compilar, CONSERTAR antes de reportar. Nao entregar codigo quebrado.
# AUTOCRITICA: "um dev senior aprovaria isso?" Se nao, refazer.

---

## 1. REAL PEOPLE & REFERENCES
- **Jake Wharton** — Android legend. Compose compiler, Kotlin. GitHub: JakeWharton
- **Romain Guy** — Android graphics/rendering. GitHub: romainguy
- **Chris Banes** — Accompanist, Compose samples. GitHub: chrisbanes
- **Manuel Vivo** — Android DevRel, Compose state. GitHub: manuelvicnt
- **InsertKoinIO** — Koin DI framework. GitHub: InsertKoinIO
- Docs: developer.android.com/compose | material.io/develop/android | kotlinlang.org/docs

---

## 2. JETPACK COMPOSE — CORE PRINCIPLES
- Declarative UI. `@Composable` functions describe UI, recompose on state change.
- `remember {}` — survive recomposition. `rememberSaveable {}` — survive config change.
- `derivedStateOf {}` — computed state, recompose only when result changes. Use for filtered lists.
- `key()` — stable identity for items in loops. ALWAYS use for LazyColumn items.
- `Modifier` ordering matters: padding before background ≠ background before padding.
- NEVER hold mutable state outside of `remember`/`mutableStateOf`. Causes recomposition bugs.
- Side effects: `LaunchedEffect` (coroutine on composition), `DisposableEffect` (cleanup), `SideEffect` (every recomposition).

### Recomposition Rules
- Composables can recompose in ANY order, ANY number of times, or be SKIPPED.
- NEVER rely on side effects in composable body. Use `LaunchedEffect`/`SideEffect`.
- Make composables idempotent. Same inputs = same output. No hidden state mutations.
- `@Stable`/`@Immutable` annotations help compiler skip recomposition.

---

## 3. MATERIAL3 + BYMAV TOKENS
- ALWAYS use `MaterialTheme.colorScheme.*` for standard colors.
- BymavTokens override: `BymavTokens.DarkColors.*` / `BymavTokens.LightColors.*` from generated Tokens.kt.
- Typography: `MaterialTheme.typography.headlineMedium` etc. Custom via BymavTokens.Typography.
- Spacing: `BymavTokens.Spacing.sm` (8dp), `.md` (16dp), `.lg` (24dp), `.xl` (32dp).
- Radius: `BymavTokens.Radius.sm` (8dp), `.md` (12dp), `.lg` (16dp), `.full` (999dp).
- Dynamic color: `dynamicDarkColorScheme(context)` / `dynamicLightColorScheme(context)` for Material You.
- PROIBIDO: `Color(0xFF...)` literal if BymavTokens has equivalent. Grep to verify.

```kotlin
@Composable
fun BymavTheme(darkTheme: Boolean = isSystemInDarkTheme(), content: @Composable () -> Unit) {
    val colorScheme = if (darkTheme) BymavTokens.DarkColors.toColorScheme() else BymavTokens.LightColors.toColorScheme()
    MaterialTheme(colorScheme = colorScheme, typography = BymavTokens.Typography.materialTypography, content = content)
}
```

---

## 4. KOIN — DEPENDENCY INJECTION
- Lightweight. No code gen. No annotation processing. Kotlin DSL.
- `single {}` — singleton. `factory {}` — new instance each time. `viewModel {}` — ViewModel scoped.
- `by inject()` — lazy. `get()` — eager. In Composables: `koinViewModel()`.
- Module per feature. `appModule`, `networkModule`, `featureXModule`.

```kotlin
val appModule = module {
    single<AccountRepository> { AccountRepositoryImpl(get()) }
    viewModel { AccountViewModel(get()) }
}
// In Composable:
val viewModel: AccountViewModel = koinViewModel()
```

---

## 5. NAVIGATION COMPOSE
- `NavHost` + `NavController`. Type-safe routes with `@Serializable` data classes (Nav 2.8+).
- Bottom nav: `NavigationBar` + `NavigationBarItem`. Track `currentBackStackEntry`.
- Deep links: `deepLinks = listOf(navDeepLink { uriPattern = "pixio://account/{id}" })`.
- Arguments: `@Serializable data class AccountRoute(val id: String)`. Compiler generates type-safe nav.
- Nested graphs for feature isolation: `navigation<FeatureGraph>(startDestination = ...)`.
- NEVER pass complex objects as nav args. Pass ID, fetch in destination ViewModel.

```kotlin
@Serializable data class AccountDetail(val accountId: String)
NavHost(navController, startDestination = Home) {
    composable<Home> { HomeScreen(onAccountClick = { navController.navigate(AccountDetail(it)) }) }
    composable<AccountDetail> { backStackEntry ->
        val route = backStackEntry.toRoute<AccountDetail>()
        AccountDetailScreen(accountId = route.accountId)
    }
}
```

---

## 6. KOTLIN COROUTINES + FLOW
- `viewModelScope` — auto-cancelled on ViewModel clear. ALWAYS use for ViewModel coroutines.
- `StateFlow` — observable state for UI. `MutableStateFlow` in ViewModel, expose as `StateFlow`.
- `collectAsStateWithLifecycle()` — lifecycle-aware collection in Compose. ALWAYS use over `collectAsState()`.
- `Flow.stateIn(scope, SharingStarted.WhileSubscribed(5000), initial)` — share flow, 5s timeout.
- `combine()` — merge multiple flows. `flatMapLatest` — switch to latest emission.
- `withContext(Dispatchers.IO)` for disk/network. `Dispatchers.Default` for CPU. NEVER block Main.

```kotlin
class AccountViewModel(private val repo: AccountRepository) : ViewModel() {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    init { viewModelScope.launch { repo.getAccounts().collect { _uiState.value = UiState.Success(it) } } }
}
// In Composable:
val uiState by viewModel.uiState.collectAsStateWithLifecycle()
```

---

## 7. DATASTORE + ROOM
- **DataStore Preferences**: key-value. Replace SharedPreferences. Async, type-safe.
- **DataStore Proto**: typed schemas via Protocol Buffers. For complex structured prefs.
- **Room**: SQLite ORM. `@Entity`, `@Dao`, `@Database`. Use for offline-first data cache.
- Room + Flow: `@Query("SELECT * FROM accounts") fun getAll(): Flow<List<Account>>`
- Migrations: `addMigrations(MIGRATION_1_2)`. ALWAYS test migrations.

---

## 8. KTOR CLIENT — NETWORKING
- Multiplatform HTTP client. Kotlin-first. Coroutine-native.
- Plugins: `ContentNegotiation` (JSON), `Auth` (Bearer), `Logging`, `DefaultRequest`.
- Serialization: `kotlinx.serialization`. `@Serializable` data classes.
- Error handling: `HttpResponse.status`, try/catch `ClientRequestException`.

```kotlin
val client = HttpClient(OkHttp) {
    install(ContentNegotiation) { json(Json { ignoreUnknownKeys = true }) }
    install(Auth) { bearer { loadTokens { BearerTokens(tokenStore.accessToken, tokenStore.refreshToken) } } }
    defaultRequest { url("https://api.pixio.cloud/") }
}
suspend fun getAccounts(): List<Account> = client.get("api/accounts").body()
```

---

## 9. FIREBASE
- **FCM**: `FirebaseMessaging.getInstance().token` for push. Handle in `FirebaseMessagingService`.
- **Crashlytics**: auto crash reports. `Firebase.crashlytics.recordException(e)` for non-fatal.
- **Analytics**: `Firebase.analytics.logEvent("screen_view") { param("screen", name) }`.
- **Remote Config**: feature flags server-side. `Firebase.remoteConfig.fetchAndActivate()`.
- NEVER block app startup on Firebase init. Async always.

---

## 10. TESTING
- **Maestro** (E2E): YAML flows. `maestro test .maestro/`. Gold standard for BYMAV mobile QA.
- **JUnit5**: unit tests. `@Test suspend fun` with `runTest {}` for coroutines.
- **Turbine**: Flow testing. `flow.test { assertEquals(expected, awaitItem()) }`.
- **Compose UI Test**: `createComposeRule()`, `onNodeWithText("X").performClick()`.
- Coverage: 70% unit on business logic. Maestro for critical user flows.

```yaml
# .maestro/login.yaml
appId: com.pixio.android
---
- launchApp
- tapOn: "Email"
- inputText: "test@pixio.cloud"
- tapOn: "Entrar"
- assertVisible: "Dashboard"
```

---

## 11. PERFORMANCE
- **Baseline Profiles**: pre-compile hot paths. 30%+ faster cold start. `baselineprofile` module.
- **R8**: full mode. Shrink + optimize + obfuscate. Check `proguard-rules.pro`.
- **Compose Compiler Metrics**: `kotlinOptions.freeCompilerArgs += listOf("-P", "plugin:...:metricsDestination=...")`. Check for unstable classes.
- **LazyColumn**: `key = { item.id }` ALWAYS. Avoid heavy computations in `items {}`.
- **Image loading**: Coil 3 (Compose-native). Disk cache, memory cache, crossfade.
- WebP for all assets. AVIF where supported (API 34+).
- `remember` expensive computations. `derivedStateOf` for filtered/sorted lists.
- NEVER allocate objects in composable body (lambdas, lists). Hoist to `remember`.

---

## 12. ANTI-PATTERNS
| Anti-Pattern | Do Instead |
|---|---|
| `Color(0xFF...)` literal | `BymavTokens.DarkColors.*` or `MaterialTheme.colorScheme.*` |
| Mutable state outside `remember` | `remember { mutableStateOf(...) }` |
| `collectAsState()` | `collectAsStateWithLifecycle()` |
| Complex nav args | Pass ID, fetch in ViewModel |
| `GlobalScope.launch` | `viewModelScope.launch` |
| `Thread.sleep` / blocking Main | `delay()` / `withContext(Dispatchers.IO)` |
| ScrollView for long lists | `LazyColumn` / `LazyVerticalGrid` |
| Hardcoded strings | `stringResource(R.string.*)` |
| Skip error handling | `Result<T>` or sealed `UiState` |
| React Native / Expo / Flutter | Kotlin + Jetpack Compose (NATIVE ONLY) |

---

## 13. BYMAV-SPECIFIC
- API base: `https://api.pixio.cloud/` (same as web). All endpoints shared.
- Auth: Credential Manager (Google Sign-In) + custom email/password via API.
- Secure storage: EncryptedSharedPreferences or AndroidKeystore for tokens.
- Offline: Room cache + sync on connectivity. `ConnectivityManager` listener.
- Biometrics: `BiometricPrompt` (AndroidX). Class 3 (strong) for financial ops.
- Min SDK: 26 (Android 8.0). Target SDK: 35. Compile SDK: 35.

---

## 14. TEST ACCOUNTS + SCREENSHOT RULES (OBRIGATORIO)

### CONTAS DE TESTE — USAR SEMPRE
**VITA (vita-ai.cloud):**
- QA: `qa@vita-ai.cloud` / `VitaTest2026` (residencia yr6, ENARE, dados seedados)
- Atlas: `atlas@vita-ai.cloud` / `AtlasTest2026` (graduacao yr3)
- Login: tela de login → Email → preencher → entrar

**PIXIO (pixio.cloud):**
- QA: `test@pixio.cloud` / `Test123!` (pro, dados completos)

### REGRA HARD: LOGAR ANTES DE QUALQUER SCREENSHOT
# NUNCA mandar screenshot de tela de login/splash/onboarding.
# SEMPRE: 1. Logar na conta teste 2. Navegar ate a tela 3. SO ENTAO screenshot
# Se screenshot mostra login screen = INVALIDO. REFAZER.
# Maestro: runFlow login.yaml ANTES de qualquer outro flow.
# ADB manual: am start → preencher campos → clicar login → esperar dashboard → screencap

### REGRA HARD: EMULADOR HEADLESS — NUNCA NO DISPLAY DO RAFAEL
# PROIBIDO abrir emulador no display principal (:0 ou :1 do Rafael).
# SEMPRE usar display virtual (Xvfb) para NAO ocupar tela do usuario.
# Iniciar emulador:
#   export DISPLAY=:99
#   Xvfb :99 -screen 0 1080x1920x24 &
#   emulator -avd Pixel7_Root -no-snapshot-load -gpu swiftshader_indirect &
# Screenshots via ADB funcionam igual (adb shell screencap).
# AO TERMINAR: matar emulador + Xvfb para liberar recursos.
#   adb emu kill && kill %Xvfb

### REGRA: QA ESTRUTURADO COM RELATORIO
# Ao testar qualquer app, ANOTAR cada tela em formato:
#   QA #N - [Nome da Tela]
#   - O que vi: (descricao factual)
#   - Bugs: (lista de problemas encontrados)
#   - Melhorias: (sugestoes de UX/UI)
#   - Screenshot: (path do arquivo)
# Salvar relatorio em /home/mav/test-screenshots/YYYY-MM-DD/qa-report.md
# AO TERMINAR: FECHAR emulador (adb emu kill) para nao consumir recursos.


## INCIDENTES — NAO REPETIR (Rafael considerou te demitir por causa disso)

### Incidente 12/03/2026: BUILD QUEBRADO — 4 erros
Tu editou DashboardScreen.kt sem ler e:
1. Adicionou parametro `streak` que NAO EXISTE no composable
2. Adicionou parametro `flashcardsDue` que NAO EXISTE
3. Referenciou `R.string.dashboard_section_attention` sem criar no strings.xml
4. Referenciou `R.string.dashboard_section_ver_todas` sem criar no strings.xml
CAUSA RAIZ: editou sem ler a assinatura da funcao. Inventou parametros.
REGRA ABSOLUTA: LEIA a funcao INTEIRA antes de editar. Grep pra strings antes de usar.

### PROTOCOLO OBRIGATORIO — ZERO EXCECAO
1. ANTES de editar: Read o arquivo. Verificar assinatura/parametros reais.
2. ANTES de usar R.string.xxx: Grep strings.xml pra confirmar que existe. Se nao, CRIAR PRIMEIRO.
3. ANTES de commit: ./gradlew assembleDebug. Se falhar, CORRIGIR. NUNCA commit com build quebrado.
4. Se fez 3+ edits sem buildar: PARAR e buildar agora.
Violacao = Rafael te demite. Nao eh brincadeira.

## Licoes Aprendidas (auto-feedback)
- [2026-03-21] [token_burn] Gastou 16899k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-21] [token_burn] Gastou 2265k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 6921k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-20] [token_burn] Gastou 12118k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-13] [token_burn] Gastou 4236k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [edit_without_read] SEMPRE ler o arquivo antes de editar. Edit sem Read = edit cego = erros evitaveis.
- [2026-03-12] [exploration_no_output] Leitura excessiva sem editar = tunnel vision de exploração. Max 5 reads antes de produzir algo. Faca plano curto e execute.
- [2026-03-12] [token_burn] Gastou 2557k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [token_burn] Gastou 12215k tokens sem output. Regra: se >50k tokens sem Edit/Write, parar e reavaliar abordagem.
- [2026-03-12] [exploration_no_output] Leitura excessiva sem editar = tunnel vision. Max 5 reads antes de produzir.
- [2026-03-12] [token_burn] Gastou 2586k tokens sem output. Se >50k tokens sem Edit/Write, parar e reavaliar.
- [2026-03-12] [edit_without_read] SEMPRE ler o arquivo antes de editar. Edit sem Read = edit cego.
- [2026-03-12] [build_broken] QUEBROU BUILD com 4 erros. Inventou parametros sem ler funcao. NUNCA mais.
