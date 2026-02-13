---
name: Kotlin Multiplatform
description: KMP 跨平台架构、共享逻辑与平台集成
---

# Kotlin Multiplatform (KMP 跨平台)

## Instructions
- 仅在需要跨平台共享逻辑时使用
- 依照下方章节顺序套用
- 一次只扩充一个共享模块或平台端集成
- 完成后对照 Quick Checklist

## When to Use
- Scenario F：跨平台共享逻辑

## Example Prompts
- "请依照 Architecture Decision，界定可共享与不可共享范围"
- "用 Project Setup 章节创建 shared 模块"
- "请参考 Ktor Client 与 SQLDelight，规划跨平台数据层"

## Workflow
1. 先定义共享边界与项目结构
2. 再落实 Network/Database 的 expect/actual
3. 最后用 Testing Shared Code 与 Quick Checklist 验收

## Practical Notes (2026)
- 共享逻辑必限制在 Domain/Data，避免 UI 共享
- commonTest 必覆盖内核业务流程
- 平台特性变更需回写到共享边界文档

## Minimal Template
```
目标: 
共享边界: 
平台差异: 
测试策略: 
验收: Quick Checklist
```

---

## Architecture Decision

### 共享边界定义

```
┌─────────────────────────────────────────────────────────┐
│                      Shared (KMP)                       │
├─────────────────────────────────────────────────────────┤
│  Domain Layer    │ Use Cases, Entities, Repository API │
│  Data Layer      │ Repository Impl, API Client, DTOs   │
│  Utilities       │ Date/Time, Validation, Extensions   │
└─────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   Android App   │  │     iOS App     │  │    Web/Desktop  │
│   (Compose UI)  │  │   (SwiftUI)     │  │                 │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### 不共享的内容

- UI Layer (Compose / SwiftUI)
- Platform-specific APIs (Notifications, Sensors)
- Platform-specific Libraries (WorkManager, HealthKit)

---

## Project Setup

### 目录结构

```
my-kmp-project/
├── androidApp/
│   ├── build.gradle.kts
│   └── src/main/kotlin/
├── iosApp/
│   └── iosApp.xcodeproj
├── shared/
│   ├── build.gradle.kts
│   └── src/
│       ├── commonMain/kotlin/
│       ├── commonTest/kotlin/
│       ├── androidMain/kotlin/
│       └── iosMain/kotlin/
└── build.gradle.kts
```

### shared/build.gradle.kts

```kotlin
plugins {
    kotlin("multiplatform")
    kotlin("plugin.serialization")
    id("com.android.library")
}

kotlin {
    androidTarget()
    
    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64()
    ).forEach {
        it.binaries.framework {
            baseName = "shared"
            isStatic = true
        }
    }
    
    sourceSets {
        commonMain.dependencies {
            implementation("io.ktor:ktor-client-core:2.3.7")
            implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
            implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.2")
        }
        
        androidMain.dependencies {
            implementation("io.ktor:ktor-client-okhttp:2.3.7")
        }
        
        iosMain.dependencies {
            implementation("io.ktor:ktor-client-darwin:2.3.7")
        }
    }
}
```

---

## Ktor Client (跨平台 Network)

### Common API

```kotlin
// commonMain
class ApiClient(private val httpClient: HttpClient) {
    
    suspend fun getUser(id: String): User {
        return httpClient.get("https://api.example.com/users/$id").body()
    }
}

// expect/actual for HttpClient
expect fun createHttpClient(): HttpClient

// androidMain
actual fun createHttpClient(): HttpClient = HttpClient(OkHttp) {
    install(ContentNegotiation) {
        json()
    }
}

// iosMain
actual fun createHttpClient(): HttpClient = HttpClient(Darwin) {
    install(ContentNegotiation) {
        json()
    }
}
```

---

## SQLDelight (跨平台 Database)

### 设置

```kotlin
// build.gradle.kts
plugins {
    id("app.cash.sqldelight") version "2.0.1"
}

sqldelight {
    databases {
        create("AppDatabase") {
            packageName.set("com.example.db")
        }
    }
}
```

### Schema

```sql
-- src/commonMain/sqldelight/com/example/db/User.sq
CREATE TABLE user (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);

selectAll:
SELECT * FROM user;

insert:
INSERT INTO user(id, name, email) VALUES (?, ?, ?);
```

### expect/actual for Driver

```kotlin
// commonMain
expect class DriverFactory {
    fun createDriver(): SqlDriver
}

// androidMain
actual class DriverFactory(private val context: Context) {
    actual fun createDriver(): SqlDriver {
        return AndroidSqliteDriver(AppDatabase.Schema, context, "app.db")
    }
}

// iosMain
actual class DriverFactory {
    actual fun createDriver(): SqlDriver {
        return NativeSqliteDriver(AppDatabase.Schema, "app.db")
    }
}
```

---

## Testing Shared Code

```kotlin
// commonTest
class UserRepositoryTest {
    
    private val testDriver = JdbcSqliteDriver(JdbcSqliteDriver.IN_MEMORY)
    private val database = AppDatabase(testDriver)
    private val repository = UserRepository(database)
    
    @Test
    fun `insert and retrieve user`() = runTest {
        repository.insertUser(User("1", "Test", "test@example.com"))
        
        val users = repository.getAllUsers()
        
        assertEquals(1, users.size)
        assertEquals("Test", users.first().name)
    }
}
```

---

## Quick Checklist

- [ ] 共享边界明确 (Domain + Data)
- [ ] UI 保持平台原生
- [ ] Ktor Client 配置 expect/actual
- [ ] SQLDelight 取代 Room (跨平台)
- [ ] commonTest 测试共享逻辑
