# scaffold-project

Generate the complete initial project structure for the Copa 2026 KMP Android app.

Usage: /scaffold-project

---

Create all files below. Package root is `com.copa2026`.

## Root `settings.gradle.kts`

```kotlin
rootProject.name = "Copa2026"
include(":androidApp", ":shared")

pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}
```

## Root `build.gradle.kts`

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.android.library) apply false
    alias(libs.plugins.kotlin.multiplatform) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.serialization) apply false
    alias(libs.plugins.sqldelight) apply false
}
```

## `gradle/libs.versions.toml`

```toml
[versions]
kotlin = "2.0.21"
agp = "8.5.2"
compose-bom = "2024.10.00"
ktor = "2.3.12"
koin = "3.5.6"
sqldelight = "2.0.2"
coil = "3.0.4"
navigation-compose = "2.8.3"
lifecycle = "2.8.7"

[libraries]
# Compose
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }
compose-material-icons = { group = "androidx.compose.material", name = "material-icons-extended" }
# Navigation
navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "navigation-compose" }
# Lifecycle
lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycle" }
lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycle" }
# Ktor
ktor-client-core = { group = "io.ktor", name = "ktor-client-core", version.ref = "ktor" }
ktor-client-android = { group = "io.ktor", name = "ktor-client-android", version.ref = "ktor" }
ktor-client-content-negotiation = { group = "io.ktor", name = "ktor-client-content-negotiation", version.ref = "ktor" }
ktor-serialization-kotlinx-json = { group = "io.ktor", name = "ktor-serialization-kotlinx-json", version.ref = "ktor" }
ktor-client-logging = { group = "io.ktor", name = "ktor-client-logging", version.ref = "ktor" }
# Koin
koin-android = { group = "io.insert-koin", name = "koin-android", version.ref = "koin" }
koin-androidx-compose = { group = "io.insert-koin", name = "koin-androidx-compose", version.ref = "koin" }
koin-core = { group = "io.insert-koin", name = "koin-core", version.ref = "koin" }
# SQLDelight
sqldelight-android-driver = { group = "app.cash.sqldelight", name = "android-driver", version.ref = "sqldelight" }
sqldelight-native-driver = { group = "app.cash.sqldelight", name = "native-driver", version.ref = "sqldelight" }
sqldelight-coroutines = { group = "app.cash.sqldelight", name = "coroutines-extensions", version.ref = "sqldelight" }
# Coil
coil-compose = { group = "io.coil-kt.coil3", name = "coil-compose", version.ref = "coil" }
coil-network-ktor = { group = "io.coil-kt.coil3", name = "coil-network-ktor3", version.ref = "coil" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
android-library = { id = "com.android.library", version.ref = "agp" }
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
sqldelight = { id = "app.cash.sqldelight", version.ref = "sqldelight" }
```

## `shared/build.gradle.kts`

```kotlin
plugins {
    alias(libs.plugins.kotlin.multiplatform)
    alias(libs.plugins.android.library)
    alias(libs.plugins.kotlin.serialization)
    alias(libs.plugins.sqldelight)
}

kotlin {
    androidTarget()

    sourceSets {
        commonMain.dependencies {
            implementation(libs.ktor.client.core)
            implementation(libs.ktor.client.content.negotiation)
            implementation(libs.ktor.serialization.kotlinx.json)
            implementation(libs.ktor.client.logging)
            implementation(libs.koin.core)
            implementation(libs.sqldelight.coroutines)
        }
        androidMain.dependencies {
            implementation(libs.ktor.client.android)
            implementation(libs.sqldelight.android.driver)
        }
        commonTest.dependencies {
            implementation(kotlin("test"))
        }
    }
}

android {
    namespace = "com.copa2026.shared"
    compileSdk = 35
    defaultConfig { minSdk = 26 }
}

sqldelight {
    databases {
        create("Copa2026Database") {
            packageName.set("com.copa2026.cache")
        }
    }
}
```

## `androidApp/build.gradle.kts`

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}

android {
    namespace = "com.copa2026"
    compileSdk = 35
    defaultConfig {
        applicationId = "com.copa2026.app"
        minSdk = 26
        targetSdk = 35
        versionCode = 1
        versionName = "1.0"
    }
    buildFeatures { compose = true }
}

dependencies {
    implementation(project(":shared"))
    implementation(platform(libs.compose.bom))
    implementation(libs.compose.ui)
    implementation(libs.compose.ui.tooling.preview)
    implementation(libs.compose.material3)
    implementation(libs.compose.material.icons)
    implementation(libs.navigation.compose)
    implementation(libs.lifecycle.viewmodel.compose)
    implementation(libs.lifecycle.runtime.compose)
    implementation(libs.koin.android)
    implementation(libs.koin.androidx.compose)
    implementation(libs.coil.compose)
    implementation(libs.coil.network.ktor)
}
```

## `androidApp/src/main/kotlin/com/copa2026/MainActivity.kt`

```kotlin
package com.copa2026

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import com.copa2026.ui.theme.Copa2026Theme
import com.copa2026.navigation.AppNavGraph

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Copa2026Theme {
                AppNavGraph()
            }
        }
    }
}
```

## `androidApp/src/main/kotlin/com/copa2026/App.kt`

```kotlin
package com.copa2026

import android.app.Application
import com.copa2026.di.appModule
import com.copa2026.di.matchesModule
import com.copa2026.di.newsModule
import com.copa2026.di.standingsModule
import org.koin.android.ext.koin.androidContext
import org.koin.core.context.startKoin

class App : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@App)
            modules(appModule, newsModule, standingsModule, matchesModule)
        }
    }
}
```

## `shared/src/commonMain/kotlin/com/copa2026/di/AppModule.kt`

```kotlin
package com.copa2026.di

import io.ktor.client.HttpClient
import io.ktor.client.plugins.contentnegotiation.ContentNegotiation
import io.ktor.client.plugins.logging.LogLevel
import io.ktor.client.plugins.logging.Logging
import io.ktor.serialization.kotlinx.json.json
import kotlinx.serialization.json.Json
import org.koin.dsl.module

val appModule = module {
    single {
        HttpClient {
            install(ContentNegotiation) {
                json(Json { ignoreUnknownKeys = true })
            }
            install(Logging) { level = LogLevel.INFO }
        }
    }
}
```

## `androidApp/src/main/kotlin/com/copa2026/navigation/AppNavGraph.kt`

```kotlin
package com.copa2026.navigation

import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.DateRange
import androidx.compose.material.icons.filled.List
import androidx.compose.material.icons.filled.Newspaper
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.navigation.NavDestination.Companion.hierarchy
import androidx.navigation.NavGraph.Companion.findStartDestination
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.navigation.compose.rememberNavController
import com.copa2026.ui.matches.MatchesScreen
import com.copa2026.ui.news.NewsScreen
import com.copa2026.ui.standings.StandingsScreen

sealed class Screen(val route: String, val label: String) {
    data object News : Screen("news", "Notícias")
    data object Standings : Screen("standings", "Tabelas")
    data object Matches : Screen("matches", "Jogos")
}

@Composable
fun AppNavGraph() {
    val navController = rememberNavController()
    val tabs = listOf(Screen.News, Screen.Standings, Screen.Matches)
    val icons = mapOf(
        Screen.News.route to Icons.Default.Newspaper,
        Screen.Standings.route to Icons.Default.List,
        Screen.Matches.route to Icons.Default.DateRange,
    )
    Scaffold(
        bottomBar = {
            NavigationBar {
                val navBackStackEntry by navController.currentBackStackEntryAsState()
                val currentDestination = navBackStackEntry?.destination
                tabs.forEach { screen ->
                    NavigationBarItem(
                        icon = { Icon(icons[screen.route]!!, contentDescription = screen.label) },
                        label = { Text(screen.label) },
                        selected = currentDestination?.hierarchy?.any { it.route == screen.route } == true,
                        onClick = {
                            navController.navigate(screen.route) {
                                popUpTo(navController.graph.findStartDestination().id) { saveState = true }
                                launchSingleTop = true
                                restoreState = true
                            }
                        },
                    )
                }
            }
        }
    ) { padding ->
        NavHost(navController, startDestination = Screen.News.route) {
            composable(Screen.News.route) { NewsScreen() }
            composable(Screen.Standings.route) { StandingsScreen() }
            composable(Screen.Matches.route) { MatchesScreen() }
        }
    }
}
```

After generating all files, run `/new-feature news`, `/new-feature standings`, and `/new-feature matches` to scaffold each feature's layers.
