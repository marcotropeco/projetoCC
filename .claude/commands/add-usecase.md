# add-usecase

Add a new use case to an existing Copa 2026 feature.

Usage: /add-usecase <Feature> <UseCaseName>
Example: /add-usecase Matches GetLiveMatchesUseCase

---

Given `$ARGUMENTS` as `<Feature> <UseCaseName>`:

## 1. Use case file — `shared/src/commonMain/kotlin/com/copa2026/domain/usecase/<UseCaseName>.kt`

```kotlin
package com.copa2026.domain.usecase

import com.copa2026.domain.model.<Feature>  // adjust import to actual domain type
import com.copa2026.domain.repository.<Feature>Repository

class <UseCaseName>(private val repository: <Feature>Repository) {
    suspend operator fun invoke(): Result<List<<Feature>>> {
        // TODO: add filtering/transformation logic here
        return repository.get<Feature>List()
    }
}
```

## 2. Add to repository interface if a new data operation is needed

In `shared/.../domain/repository/<Feature>Repository.kt`, add the method signature before implementing it.

## 3. Register in Koin module — `shared/.../di/<feature>Module.kt`

```kotlin
factory { <UseCaseName>(get()) }
```

## 4. Inject into ViewModel

In `<Feature>ViewModel`, add as constructor parameter and call in the relevant coroutine scope.

Keep use cases focused on a single operation. If the use case only delegates to the repository with no transformation, skip it and call the repository directly from the ViewModel.
