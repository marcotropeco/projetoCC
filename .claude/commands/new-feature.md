# new-feature

Scaffold a complete Copa 2026 feature module (domain → data → presentation → UI → DI → navigation).

Usage: /new-feature <feature-name>
Example: /new-feature standings

---

Given `$ARGUMENTS` as the feature name (e.g. "standings"), derive:
- `FeatureName` = PascalCase (e.g. `Standings`)
- `featureName` = camelCase (e.g. `standings`)

Create the following files with the content shown:

## 1. Domain model — `shared/src/commonMain/kotlin/com/copa2026/domain/model/<FeatureName>.kt`

```kotlin
package com.copa2026.domain.model

data class <FeatureName>(
    val id: String,
    // TODO: add domain fields
)
```

## 2. Repository interface — `shared/src/commonMain/kotlin/com/copa2026/domain/repository/<FeatureName>Repository.kt`

```kotlin
package com.copa2026.domain.repository

import com.copa2026.domain.model.<FeatureName>

interface <FeatureName>Repository {
    suspend fun get<FeatureName>List(): Result<List<<FeatureName>>>
}
```

## 3. Use case — `shared/src/commonMain/kotlin/com/copa2026/domain/usecase/Get<FeatureName>ListUseCase.kt`

```kotlin
package com.copa2026.domain.usecase

import com.copa2026.domain.model.<FeatureName>
import com.copa2026.domain.repository.<FeatureName>Repository

class Get<FeatureName>ListUseCase(private val repository: <FeatureName>Repository) {
    suspend operator fun invoke(): Result<List<<FeatureName>>> = repository.get<FeatureName>List()
}
```

## 4. DTO — `shared/src/commonMain/kotlin/com/copa2026/data/remote/dto/<FeatureName>Dto.kt`

```kotlin
package com.copa2026.data.remote.dto

import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class <FeatureName>Dto(
    @SerialName("id") val id: String,
    // TODO: map API fields
)

fun <FeatureName>Dto.toDomain() = com.copa2026.domain.model.<FeatureName>(
    id = id,
)
```

## 5. Repository impl — `shared/src/commonMain/kotlin/com/copa2026/data/repository/<FeatureName>RepositoryImpl.kt`

```kotlin
package com.copa2026.data.repository

import com.copa2026.data.remote.dto.<FeatureName>Dto
import com.copa2026.data.remote.dto.toDomain
import com.copa2026.domain.model.<FeatureName>
import com.copa2026.domain.repository.<FeatureName>Repository
import io.ktor.client.HttpClient
import io.ktor.client.call.body
import io.ktor.client.request.get

class <FeatureName>RepositoryImpl(private val client: HttpClient) : <FeatureName>Repository {
    override suspend fun get<FeatureName>List(): Result<List<<FeatureName>>> = runCatching {
        client.get("<featureName>").body<List<<FeatureName>Dto>>().map { it.toDomain() }
    }
}
```

## 6. ViewModel — `shared/src/commonMain/kotlin/com/copa2026/presentation/<featureName>/<FeatureName>ViewModel.kt`

```kotlin
package com.copa2026.presentation.<featureName>

import com.copa2026.domain.model.<FeatureName>
import com.copa2026.domain.usecase.Get<FeatureName>ListUseCase
import dev.icerock.moko.mvvm.viewmodel.ViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch

class <FeatureName>ViewModel(private val get<FeatureName>List: Get<FeatureName>ListUseCase) : ViewModel() {

    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState

    init { load() }

    fun load() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            get<FeatureName>List().fold(
                onSuccess = { _uiState.value = UiState.Success(it) },
                onFailure = { _uiState.value = UiState.Error(it.message ?: "Erro desconhecido") },
            )
        }
    }

    sealed interface UiState {
        data object Loading : UiState
        data class Success(val items: List<<FeatureName>>) : UiState
        data class Error(val message: String) : UiState
    }
}
```

## 7. Screen composable — `androidApp/src/main/kotlin/com/copa2026/ui/<featureName>/<FeatureName>Screen.kt`

```kotlin
package com.copa2026.ui.<featureName>

import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import org.koin.androidx.compose.koinViewModel

@Composable
fun <FeatureName>Screen(viewModel: <FeatureName>ViewModel = koinViewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    <FeatureName>Content(uiState = uiState, onRetry = viewModel::load)
}

@Composable
private fun <FeatureName>Content(
    uiState: <FeatureName>ViewModel.UiState,
    onRetry: () -> Unit,
) {
    when (uiState) {
        is <FeatureName>ViewModel.UiState.Loading -> Box(Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
            CircularProgressIndicator()
        }
        is <FeatureName>ViewModel.UiState.Error -> Column(Modifier.fillMaxSize(), horizontalAlignment = Alignment.CenterHorizontally) {
            Text(uiState.message)
            Button(onClick = onRetry) { Text("Tentar novamente") }
        }
        is <FeatureName>ViewModel.UiState.Success -> {
            // TODO: render uiState.items
        }
    }
}

@Preview(showBackground = true)
@Composable
private fun <FeatureName>LoadingPreview() = <FeatureName>Content(uiState = <FeatureName>ViewModel.UiState.Loading, onRetry = {})
```

## 8. Koin module — `shared/src/commonMain/kotlin/com/copa2026/di/<featureName>Module.kt`

```kotlin
package com.copa2026.di

import com.copa2026.data.repository.<FeatureName>RepositoryImpl
import com.copa2026.domain.repository.<FeatureName>Repository
import com.copa2026.domain.usecase.Get<FeatureName>ListUseCase
import com.copa2026.presentation.<featureName>.<FeatureName>ViewModel
import org.koin.dsl.module

val <featureName>Module = module {
    single<<FeatureName>Repository> { <FeatureName>RepositoryImpl(get()) }
    factory { Get<FeatureName>ListUseCase(get()) }
    factory { <FeatureName>ViewModel(get()) }
}
```

After creating all files, remind the user to:
- Add `<featureName>Module` to the Koin `startKoin { modules(...) }` call
- Add a navigation route for `<FeatureName>Screen` in `AppNavGraph.kt`
- Fill in the actual API endpoint path and domain/DTO fields
