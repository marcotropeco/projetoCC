# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Android app using **Kotlin Multiplatform (KMP)** to display Copa do Mundo 2026 content: news feed, group/knockout standings tables, and match schedule/results.

## Architecture

Layered Clean Architecture inside the KMP `shared` module, consumed by the `androidApp` module via Jetpack Compose UI.

```
projeto1/
├── shared/                  # KMP shared module (business logic + data)
│   ├── commonMain/
│   │   ├── data/            # Repository impls, API clients (Ktor), DTOs
│   │   ├── domain/          # Use cases, domain models, repository interfaces
│   │   └── presentation/    # ViewModels (shared, Compose Multiplatform-friendly)
│   └── androidMain/         # Android-specific actual implementations
├── androidApp/              # Android Compose UI
│   └── src/main/
│       ├── ui/
│       │   ├── news/        # News feed screen
│       │   ├── standings/   # Group + knockout tables screen
│       │   └── matches/     # Schedule/results screen
│       └── navigation/      # NavHost + bottom nav
└── build.gradle.kts / settings.gradle.kts
```

## Key Technology Stack

| Layer | Library |
|---|---|
| Networking | Ktor Client |
| Serialization | kotlinx.serialization |
| Async | Coroutines + StateFlow |
| DI | Koin Multiplatform |
| Local cache | SQLDelight |
| UI | Jetpack Compose (Android) |
| Image loading | Coil 3 (Compose) |
| Navigation | Navigation Compose |

## Build & Run

```bash
# Build debug APK
./gradlew androidApp:assembleDebug

# Install on connected device/emulator
./gradlew androidApp:installDebug

# Run all tests (shared + android)
./gradlew test

# Run shared module unit tests only
./gradlew shared:testDebugUnitTest

# Run a single test class
./gradlew shared:testDebugUnitTest --tests "com.copa2026.domain.StandingsUseCaseTest"

# Lint
./gradlew lint

# Generate SQLDelight schema/migrations
./gradlew generateCommonMainDatabaseInterface
```

## Domain Models

- `Match` — fixture with date, teams, venue, score, stage (group/round of 16/etc.)
- `Standing` — group table row: team, played, won, drawn, lost, GF, GA, GD, points
- `NewsArticle` — title, summary, imageUrl, publishedAt, sourceUrl
- `Group` — group letter + list of `Standing`
- `Stage` — enum: GROUP, R16, QF, SF, THIRD, FINAL

## Feature Modules

### News (`news/`)
- Paginated list from REST API
- Offline-first: SQLDelight cache, TTL-based refresh
- ViewModel exposes `StateFlow<NewsUiState>`

### Standings (`standings/`)
- Tab per group (A–H) + knockout bracket view
- Sorted by: points → GD → GF → head-to-head
- Data source: same REST API + local cache

### Matches (`matches/`)
- Filter by: All / Date / Team / Group / Stage
- Shows live scores (poll every 60 s during match window)
- Match detail screen with lineups if available

## Custom Slash Commands

| Command | Usage | Description |
|---|---|---|
| `/scaffold-project` | `/scaffold-project` | Gera estrutura inicial completa: gradle, DI, navegação, MainActivity |
| `/new-feature` | `/new-feature standings` | Scaffolda feature completa: domain → data → ViewModel → Screen → Koin |
| `/add-screen` | `/add-screen matches MatchDetailScreen` | Adiciona tela Compose a feature existente + rota de navegação |
| `/add-usecase` | `/add-usecase Matches GetLiveMatchesUseCase` | Adiciona caso de uso com registro no Koin |
| `/add-sqldelight-table` | `/add-sqldelight-table Match` | Cria tabela `.sq` + cache local data source com TTL |

## Data Layer Conventions

- Repository interfaces live in `domain/`, implementations in `data/`
- All network responses mapped to domain models inside the repository — no DTOs leak into `domain/` or `presentation/`
- Use `Result<T>` (kotlin stdlib) for error propagation; never throw across the domain boundary

## UI Conventions

- One `Screen` composable per feature, backed by one ViewModel
- ViewModel exposes a single `uiState: StateFlow<UiState>` sealed class with `Loading`, `Success(data)`, `Error(message)` states
- Navigation uses typed routes (Navigation Compose type-safe APIs)
- `@Preview` annotations required for all leaf composables
