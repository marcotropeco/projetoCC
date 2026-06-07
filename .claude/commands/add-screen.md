# add-screen

Add a new Compose screen to an existing Copa 2026 feature, wired to an existing ViewModel.

Usage: /add-screen <feature> <ScreenName>
Example: /add-screen matches MatchDetailScreen

---

Given `$ARGUMENTS` as `<feature> <ScreenName>`:
- `feature` = existing feature folder name (e.g. `matches`)
- `ScreenName` = PascalCase screen name (e.g. `MatchDetailScreen`)
- Infer the ViewModel name as `<Feature>ViewModel` (e.g. `MatchesViewModel`)

## 1. Screen file — `androidApp/src/main/kotlin/com/copa2026/ui/<feature>/<ScreenName>.kt`

```kotlin
package com.copa2026.ui.<feature>

import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import org.koin.androidx.compose.koinViewModel

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun <ScreenName>(
    onNavigateBack: () -> Unit,
    viewModel: <Feature>ViewModel = koinViewModel(),
) {
    val uiState by viewModel.uiState.collectAsState()
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("<ScreenName>") },
                navigationIcon = {
                    IconButton(onClick = onNavigateBack) {
                        Icon(Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "Voltar")
                    }
                },
            )
        }
    ) { padding ->
        <ScreenName>Content(uiState = uiState, modifier = Modifier.padding(padding))
    }
}

@Composable
private fun <ScreenName>Content(
    uiState: <Feature>ViewModel.UiState,
    modifier: Modifier = Modifier,
) {
    // TODO: implement UI for this screen
}

@Preview(showBackground = true)
@Composable
private fun <ScreenName>Preview() {
    // TODO: pass a stub uiState for preview
}
```

## 2. Register route in `androidApp/src/main/kotlin/com/copa2026/navigation/AppNavGraph.kt`

Add inside the `NavHost` block:
```kotlin
composable("<screenNameLowerCase>/{id}") { backStackEntry ->
    val id = backStackEntry.arguments?.getString("id") ?: return@composable
    <ScreenName>(onNavigateBack = { navController.popBackStack() })
}
```

And add a route constant to the sealed/object routes file if one exists.

After creating the file, remind the user to replace `<Feature>ViewModel.UiState` with the correct state type for this screen (may need a new detail-specific ViewModel if data is different from the list ViewModel).
