---
name: Kotlin Multiplatform
description: KMP 跨平台架构、共享逻辑与平台集成
---

# Kotlin Multiplatform (KMP 跨平台)

## Instructions
- 仅在需要跨平台共享逻辑时使用
- 先填写 Required Inputs（共享边界、平台差异、发布策略）
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
1. 先确认 Required Inputs（共享边界、平台 owner、测试基线）
2. 定义共享边界与项目结构
3. 落实 Network/Database 的 expect/actual
4. 执行 KMP Gate（common + platform tests）
5. 用 Testing Shared Code 与 Quick Checklist 验收

## Practical Notes (2026)
- 共享逻辑必限制在 Domain/Data，避免 UI 共享
- commonTest 必覆盖内核业务流程
- 平台特性变更需回写到共享边界文档
- 每次跨平台抽取后都要验证 API 稳定性与二进制兼容
- 平台特有 fallback 逻辑必须明确，避免 shared 层泄漏平台细节

## Minimal Template
```
目标: 
共享边界: 
平台 owner:
发布策略:
平台差异: 
测试策略: 
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `共享边界`（domain/data/util）
- `平台差异`（android/ios/web）
- `平台 owner`（责任人）
- `发布策略`（内部分发/外部发布）
- `测试基线`（commonTest + platform tests）

## Deliverables (完成后交付物)

- `shared` 模块结构与边界说明
- `expect/actual` 实现与注释
- `commonTest` 与平台测试结果
- `兼容性说明`（API/行为差异）
- `KMP Gate` 验收记录

## KMP Gate (验收门槛)

```bash
./gradlew :shared:allTests
./gradlew :androidApp:test
```

> 若接入 iOS，请补充 Xcode/CI 的 iOS 测试结果链接。

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
            implementation("io.ktor:ktor-client-core:<project-verified-version>")
            implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:<project-verified-version>")
            implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:<project-verified-version>")
        }
        
        androidMain.dependencies {
            implementation("io.ktor:ktor-client-okhttp:<project-verified-version>")
        }
        
        iosMain.dependencies {
            implementation("io.ktor:ktor-client-darwin:<project-verified-version>")
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
    id("app.cash.sqldelight") version "<project-verified-version>"
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

- [ ] Required Inputs 已填写并冻结（边界/差异/owner）
- [ ] 共享边界明确 (Domain + Data)
- [ ] UI 保持平台原生
- [ ] Ktor Client 配置 expect/actual
- [ ] SQLDelight 取代 Room (跨平台)
- [ ] commonTest 测试共享逻辑
- [ ] KMP Gate 已执行并记录结果
