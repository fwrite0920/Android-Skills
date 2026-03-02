---
name: Data Layer Mastery
description: Room 进阶用法、Retrofit 集成与 Offline-First 架构
---

# Data Layer Mastery (数据层专精)

## Instructions
- 确认需求属于数据层（Room、网络、脱机策略）
- 依照下方章节顺序套用
- 一次只调整一个数据流或责任边界
- 完成后对照 Quick Checklist

## When to Use
- Scenario A：新项目数据层创建
- Scenario D：性能问题的数据层瓶颈
- Scenario F：KMP 共享数据层设计

## Example Prompts
- "请参考 Room Advanced，帮我设计 Migration 策略"
- "依照 Network Layer 章节，创建统一的错误处理"
- "请用 Offline-First 章节查看目前 Repository 是否符合 SSOT"

## Workflow
1. 先检查 Room / Network 的基础设计
2. 再确立 Offline-First 与数据同步策略
3. 最后用 Quick Checklist 验收

## Practical Notes (2026)
- Offline-first 只在不稳网络或高一致性需求时激活
- Repository 必须是 SSOT，避免多处来源竞争
- 错误处理统一化，避免每层自行判断

## Minimal Template
```
目标: 
数据源: 
缓存策略: 
错误处理: 
验收: Quick Checklist
```

---

## Room Advanced

### Migration 策略

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE users ADD COLUMN avatar_url TEXT")
    }
}

val MIGRATION_2_3 = object : Migration(2, 3) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // 复杂迁移：创建新表、复制数据、删除旧表
        database.execSQL("CREATE TABLE users_new (...)")
        database.execSQL("INSERT INTO users_new SELECT ... FROM users")
        database.execSQL("DROP TABLE users")
        database.execSQL("ALTER TABLE users_new RENAME TO users")
    }
}

Room.databaseBuilder(context, AppDatabase::class.java, "app.db")
    .addMigrations(MIGRATION_1_2, MIGRATION_2_3)
    .build()
```

### Paging 3 集成

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users ORDER BY name")
    fun pagingSource(): PagingSource<Int, User>
}

// Repository
class UserRepository(private val dao: UserDao) {
    fun getUsers(): Flow<PagingData<User>> = Pager(
        config = PagingConfig(pageSize = 20, prefetchDistance = 5),
        pagingSourceFactory = { dao.pagingSource() }
    ).flow
}

// ViewModel
val users = repository.getUsers().cachedIn(viewModelScope)
```

### Full-Text Search (FTS)

```kotlin
@Fts4(contentEntity = Article::class)
@Entity(tableName = "articles_fts")
data class ArticleFts(
    @ColumnInfo(name = "title") val title: String,
    @ColumnInfo(name = "content") val content: String
)

@Dao
interface ArticleDao {
    @Query("SELECT * FROM articles WHERE rowid IN (SELECT rowid FROM articles_fts WHERE articles_fts MATCH :query)")
    fun search(query: String): Flow<List<Article>>
}
```

---

## Network Layer (Retrofit + OkHttp)

### Error Handling Strategy

```kotlin
sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error(val code: Int, val message: String) : NetworkResult<Nothing>()
    data object NetworkError : NetworkResult<Nothing>()
}

suspend fun <T> safeApiCall(apiCall: suspend () -> Response<T>): NetworkResult<T> {
    return try {
        val response = apiCall()
        if (response.isSuccessful) {
            val body = response.body()
            if (body != null) {
                NetworkResult.Success(body)
            } else {
                NetworkResult.Error(response.code(), "Empty response body")
            }
        } else {
            NetworkResult.Error(response.code(), response.message())
        }
    } catch (e: IOException) {
        NetworkResult.NetworkError
    }
}
```

### Interceptors

```kotlin
// Logging
val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = if (BuildConfig.DEBUG) BODY else NONE
}

// Auth Token
class AuthInterceptor(private val tokenProvider: TokenProvider) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer ${tokenProvider.token}")
            .build()
        return chain.proceed(request)
    }
}

// Retry
class RetryInterceptor(
    private val maxRetries: Int = 3,
    private val baseDelayMs: Long = 200
) : Interceptor {
    private val retryableMethods = setOf("GET", "HEAD", "OPTIONS")
    private val retryableStatusCodes = setOf(408, 429, 500, 502, 503, 504)

    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()

        // 仅重试幂等请求，避免 POST/PUT 重复提交副作用
        if (request.method !in retryableMethods) {
            return chain.proceed(request)
        }

        var attempt = 0
        while (true) {
            try {
                val response = chain.proceed(request)
                val shouldRetry = response.code in retryableStatusCodes && attempt < maxRetries - 1
                if (!shouldRetry) {
                    return response
                }
                response.close()
            } catch (e: IOException) {
                if (attempt >= maxRetries - 1) throw e
            }

            attempt++
            Thread.sleep(baseDelayMs * attempt) // 简单 backoff，降低瞬时失败抖动
        }
    }
}
```

---

## Offline-First Architecture

### Repository Pattern (SSOT)

```kotlin
class UserRepository(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource
) {
    fun getUser(id: String): Flow<User> = flow {
        // 1. 先从 Local 发射
        localDataSource.getUser(id)?.let { emit(it) }
        
        // 2. 从 Remote 取得最新
        val remote = remoteDataSource.fetchUser(id)
        
        // 3. 存入 Local
        localDataSource.saveUser(remote)
        
        // 4. 发射更新后的数据
        emit(remote)
    }
    
    // 或使用 NetworkBoundResource pattern
    fun getUserWithCache(id: String): Flow<Resource<User>> = networkBoundResource(
        query = { localDataSource.getUserFlow(id) },
        fetch = { remoteDataSource.fetchUser(id) },
        saveFetchResult = { localDataSource.saveUser(it) },
        shouldFetch = { it == null || it.isStale() }
    )
}
```

---

## DataStore Migration

### SharedPreferences → Preferences DataStore

```kotlin
val Context.dataStore by preferencesDataStore(
    name = "settings",
    produceMigrations = { context ->
        listOf(SharedPreferencesMigration(context, "old_prefs"))
    }
)

// 使用
val themeKey = booleanPreferencesKey("dark_theme")

suspend fun setDarkTheme(enabled: Boolean) {
    context.dataStore.edit { prefs ->
        prefs[themeKey] = enabled
    }
}

val darkThemeFlow: Flow<Boolean> = context.dataStore.data
    .map { it[themeKey] ?: false }
```

---

## Quick Checklist

- [ ] Room Migration 测试通过
- [ ] Network Error 统一处理
- [ ] Repository 实作 SSOT
- [ ] DataStore 取代 SharedPreferences
- [ ] Paging 用于大量数据列表
