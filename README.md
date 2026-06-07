# Copa do Mundo 2026

App Android nativo construído com **Kotlin Multiplatform (KMP)** para acompanhar a Copa do Mundo 2026: feed de notícias, tabelas de grupos, chaveamento eliminatório e agenda de partidas com placar ao vivo.

## Funcionalidades

| Feature | Descrição |
|---|---|
| **Notícias** | Feed paginado com cache offline e atualização automática |
| **Classificação** | Tabelas dos grupos A–H e chaveamento eliminatório |
| **Jogos** | Agenda completa com filtros, resultados e placar ao vivo |

## Arquitetura

Clean Architecture em camadas dentro do módulo `shared` (KMP), consumido pelo `androidApp` via Jetpack Compose.

```
projetoCC/
├── shared/                        # Módulo KMP — lógica de negócio e dados
│   ├── commonMain/
│   │   ├── data/                  # Repositórios, Ktor API client, DTOs
│   │   ├── domain/                # Use cases, modelos, interfaces de repositório
│   │   └── presentation/          # ViewModels compartilhados (StateFlow)
│   └── androidMain/               # Implementações actual para Android
├── androidApp/                    # UI Android com Jetpack Compose
│   └── src/main/
│       ├── ui/
│       │   ├── news/              # Tela de notícias
│       │   ├── standings/         # Tabelas e chaveamento
│       │   └── matches/           # Agenda e detalhes de partidas
│       └── navigation/            # NavHost + bottom navigation
├── build.gradle.kts
└── settings.gradle.kts
```

### Fluxo de dados

```
API REST (Ktor)
      ↓
  Repository  ←→  SQLDelight (cache local)
      ↓
  Use Case
      ↓
  ViewModel  (StateFlow<UiState>)
      ↓
  Compose Screen
```

## Stack de tecnologias

| Camada | Biblioteca | Versão |
|---|---|---|
| Networking | [Ktor Client](https://ktor.io/docs/client-create-multiplatform-application.html) | 2.x |
| Serialização | [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) | 1.x |
| Async | Coroutines + StateFlow | 1.x |
| Injeção de dependência | [Koin Multiplatform](https://insert-koin.io/docs/reference/koin-mp/kmp/) | 3.x |
| Cache local | [SQLDelight](https://cashapp.github.io/sqldelight/) | 2.x |
| UI | [Jetpack Compose](https://developer.android.com/develop/ui/compose) | BOM 2024.x |
| Carregamento de imagens | [Coil 3](https://coil-kt.github.io/coil/) | 3.x |
| Navegação | [Navigation Compose](https://developer.android.com/develop/ui/compose/navigation) | 2.x |
| Design system | Material Design 3 | — |

## Build & Execução

**Pré-requisitos:** Android Studio Hedgehog+ / IntelliJ IDEA, JDK 17+, Android SDK 34.

```bash
# Build debug APK
./gradlew androidApp:assembleDebug

# Instalar em dispositivo/emulador conectado
./gradlew androidApp:installDebug

# Todos os testes (shared + android)
./gradlew test

# Testes unitários do módulo shared
./gradlew shared:testDebugUnitTest

# Classe de teste específica
./gradlew shared:testDebugUnitTest --tests "com.copa2026.domain.StandingsUseCaseTest"

# Lint
./gradlew lint

# Gerar interface SQLDelight (rodar após alterar .sq)
./gradlew generateCommonMainDatabaseInterface
```

## Modelos de domínio

```kotlin
data class Match(val id: String, val homeTeam: Team, val awayTeam: Team,
                 val date: Instant, val venue: String, val score: Score?,
                 val stage: Stage, val status: MatchStatus)

data class Standing(val team: Team, val played: Int, val won: Int,
                    val drawn: Int, val lost: Int, val gf: Int,
                    val ga: Int, val gd: Int, val points: Int)

data class NewsArticle(val id: String, val title: String, val summary: String,
                       val imageUrl: String, val publishedAt: Instant,
                       val sourceUrl: String)

data class Group(val letter: String, val standings: List<Standing>)

enum class Stage { GROUP, R16, QF, SF, THIRD, FINAL }
```

## Features

### Notícias
- Lista paginada consumindo REST API
- **Offline-first**: cache SQLDelight com TTL de 5 minutos
- Pull-to-refresh e paginação infinita
- ViewModel expõe `StateFlow<NewsUiState>`

### Classificação
- Tab por grupo (A–H) + chaveamento eliminatório visual
- Ordenação: pontos → saldo de gols → gols pró → confronto direto
- Mesma fonte de dados: API + cache local

### Jogos
- Filtros: Todos / Data / Time / Grupo / Fase
- **Placar ao vivo**: polling a cada 60 s durante a janela de jogo
- Tela de detalhe com escalações quando disponíveis

## Convenções do projeto

### Dados
- Interfaces de repositório em `domain/`, implementações em `data/`
- DTOs mapeados para modelos de domínio **dentro** do repositório — nenhum DTO vaza para `domain/` ou `presentation/`
- `Result<T>` (stdlib Kotlin) para propagação de erros; nunca lançar exceções além da fronteira de domínio

### UI
- Um composable `Screen` por feature, com um ViewModel correspondente
- ViewModel expõe `uiState: StateFlow<UiState>` — sealed class com `Loading`, `Success(data)`, `Error(message)`
- Rotas tipadas com Navigation Compose type-safe APIs
- `@Preview` obrigatório em todos os composables folha

## Comandos Claude Code (slash commands)

| Comando | Exemplo | Descrição |
|---|---|---|
| `/scaffold-project` | `/scaffold-project` | Gera estrutura inicial: Gradle, DI, navegação, MainActivity |
| `/new-feature` | `/new-feature standings` | Scaffolda feature completa: domain → data → ViewModel → Screen → Koin |
| `/add-screen` | `/add-screen matches MatchDetailScreen` | Adiciona tela Compose + rota de navegação |
| `/add-usecase` | `/add-usecase Matches GetLiveMatchesUseCase` | Adiciona use case com registro no Koin |
| `/add-sqldelight-table` | `/add-sqldelight-table Match` | Cria tabela `.sq` + data source com TTL |

## Issues

O backlog completo de desenvolvimento está disponível nas [Issues do repositório](https://github.com/marcotropeco/projetoCC/issues), organizadas por camada:

- **Setup & Infra**: scaffold KMP, Koin, navegação, tema
- **Domain**: modelos, interfaces, use cases
- **Data**: Ktor, SQLDelight, repositórios
- **Features**: News, Standings, Matches
- **Testes**: cobertura mínima 80% em domain e presentation

## Licença

Distribuído sob a licença MIT.
