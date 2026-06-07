# add-sqldelight-table

Add a SQLDelight table and queries for offline caching of a Copa 2026 entity.

Usage: /add-sqldelight-table <TableName>
Example: /add-sqldelight-table Match

---

Given `$ARGUMENTS` as the table name (PascalCase, e.g. `Match`):
- `TableName` = PascalCase (e.g. `Match`)
- `tableName` = snake_case (e.g. `match`)

## 1. SQLDelight schema file — `shared/src/commonMain/sqldelight/com/copa2026/cache/<TableName>.sq`

```sql
CREATE TABLE <TableName> (
    id TEXT NOT NULL PRIMARY KEY,
    -- TODO: add columns matching the domain model
    cached_at INTEGER NOT NULL  -- epoch millis, used for TTL invalidation
);

selectAll:
SELECT * FROM <TableName> ORDER BY id;

selectById:
SELECT * FROM <TableName> WHERE id = :id;

upsert:
INSERT OR REPLACE INTO <TableName>(id, cached_at)
VALUES (?, ?);

deleteAll:
DELETE FROM <TableName>;

deleteExpired:
DELETE FROM <TableName> WHERE cached_at < :threshold;
```

## 2. Cache data source — `shared/src/commonMain/kotlin/com/copa2026/data/local/<TableName>LocalDataSource.kt`

```kotlin
package com.copa2026.data.local

import com.copa2026.cache.Copa2026Database
import com.copa2026.domain.model.<TableName>
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

private const val TTL_MS = 5 * 60 * 1_000L // 5 minutes

class <TableName>LocalDataSource(private val db: Copa2026Database) {

    suspend fun getAll(): List<<TableName>> = withContext(Dispatchers.Default) {
        db.<tableName>Queries.selectAll().executeAsList().map { it.toDomain() }
    }

    suspend fun upsertAll(items: List<<TableName>>) = withContext(Dispatchers.Default) {
        db.transaction {
            items.forEach { item ->
                db.<tableName>Queries.upsert(item.id, System.currentTimeMillis())
            }
        }
    }

    suspend fun isCacheValid(): Boolean = withContext(Dispatchers.Default) {
        val threshold = System.currentTimeMillis() - TTL_MS
        db.<tableName>Queries.selectAll().executeAsList()
            .any { it.cached_at > threshold }
    }

    suspend fun clear() = withContext(Dispatchers.Default) {
        db.<tableName>Queries.deleteAll()
    }
}

// Extension — fill in actual column mapping after adding columns to the .sq file
private fun com.copa2026.cache.<TableName>.toDomain() = <TableName>(id = id)
```

## 3. Update the repository impl to use the cache

In `<TableName>RepositoryImpl`, inject `<TableName>LocalDataSource` and apply offline-first pattern:

```kotlin
override suspend fun get<TableName>List(): Result<List<<TableName>>> = runCatching {
    if (localDataSource.isCacheValid()) return@runCatching localDataSource.getAll()
    val fresh = client.get("<tableName>").body<List<<TableName>Dto>>().map { it.toDomain() }
    localDataSource.upsertAll(fresh)
    fresh
}
```

## 4. Provide the database in Koin (`shared/.../di/AppModule.kt`)

```kotlin
// Android actual — add in androidMain
single { AndroidSqliteDriver(Copa2026Database.Schema, androidContext(), "copa2026.db") }
single { Copa2026Database(get()) }
```

Remind the user to add the actual column definitions to the `.sq` file and update the `toDomain()` extension accordingly.
