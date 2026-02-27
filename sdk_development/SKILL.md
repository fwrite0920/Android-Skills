---
name: SDK Development
description: Android SDK/Library 开发全生命周期：API 设计、发布、版本策略与文档
---

# SDK Development (SDK/Library 开发)

## Instructions
- 确认需求属于 SDK/Library 开发（非 App 开发）
- 依照下方章节顺序套用
- 一次只处理一个面向（API 设计、发布、文档）
- 完成后对照 Quick Checklist

## When to Use
- 开发供第三方使用的 Android SDK/Library
- 需要发布 AAR/KLib 到 Maven Central 或私有仓库
- 需要设计稳定的 Public API 与版本策略

## Example Prompts
- "请依照 API Design 章节设计 SDK 的公开接口"
- "帮我配置 Maven Central 发布流程"
- "请用 Binary Compatibility 章节检查 API 变更"
- "帮我创建 SDK 的 Sample App"

## Workflow
1. 先设计 Public API 与模块结构
2. 配置 Consumer Proguard 与依赖传递策略
3. 设置发布流程（Maven Central / 私有仓库）
4. 创建文档与 Sample App
5. 用 Quick Checklist 验收

## Practical Notes (2026)
- Public API 必须最小化，只暴露必要接口
- 二进制兼容性检查纳入 CI Gate
- SDK 依赖尽量用 compileOnly 避免传递冲突
- Sample App 是最好的 API 验证工具

## Minimal Template
```
目标:
SDK 名称:
目标用户:
Public API 范围:
发布目标:
验收: Quick Checklist
```

---

## API Design & Visibility Control

### 最小化 Public API

```kotlin
// ❌ 错误：暴露过多实现细节
class MySDK {
    val internalCache: MutableMap<String, Any> = mutableMapOf()  // 不应该 public
    fun processInternal(data: String) { }  // 不应该 public
}

// ✅ 正确：只暴露必要接口
class MySDK {
    private val cache: MutableMap<String, Any> = mutableMapOf()

    fun initialize(context: Context, config: Config) { }
    fun performAction(input: String): Result<Output>

    internal fun processInternal(data: String) { }  // 仅供内部模块使用
}
```

### Visibility Modifiers 策略

```kotlin
// Public API — 供外部使用
public class MySDK { }
public interface Callback { }

// Internal — 跨模块共享但不对外
internal class NetworkClient { }
internal object ConfigValidator { }

// Private — 模块内部
private class CacheManager { }
```

### Sealed Interface 限制实现

```kotlin
// 防止外部实现接口
sealed interface Result<out T> {
    data class Success<T>(val data: T) : Result<T>
    data class Error(val exception: Exception) : Result<Nothing>
}

// 用户只能消费，不能创建新的 Result 子类
```

---

## Module Structure

### 多模块分层

```
my-sdk/
├── sdk-api/          # Public API 定义（纯接口）
├── sdk-core/         # 核心实现
├── sdk-network/      # 网络模块（可选依赖）
├── sdk-storage/      # 存储模块（可选依赖）
└── sample-app/       # 示例应用
```

### sdk-api Module

```kotlin
// sdk-api/build.gradle.kts — 纯接口，无依赖
plugins {
    id("com.android.library")
    kotlin("android")
}

dependencies {
    // 无外部依赖，只有 Kotlin stdlib
}
```

```kotlin
// sdk-api/src/main/kotlin/MySDK.kt
interface MySDK {
    fun initialize(context: Context, config: Config)
    fun performAction(input: String): Result<Output>
}

data class Config(
    val apiKey: String,
    val enableLogging: Boolean = false
)

### sdk-core Module

```kotlin
// sdk-core/build.gradle.kts
plugins {
    id("com.android.library")
    kotlin("android")
}

dependencies {
    api(project(":sdk-api"))  // 暴露 API 定义
    implementation(libs.okhttp)  // 不传递给消费者
    implementation(libs.kotlinx.coroutines.android)
}
```

```kotlin
// sdk-core/src/main/kotlin/MySDKImpl.kt
internal class MySDKImpl(
    private val context: Context,
    private val config: Config
) : MySDK {

    override fun initialize(context: Context, config: Config) {
        // 初始化逻辑
    }

    override fun performAction(input: String): Result<Output> {
        // 实现逻辑
        return Result.Success(Output(input))
    }
}

// Factory 作为唯一入口
object MySDKFactory {
    fun create(context: Context, config: Config): MySDK {
        return MySDKImpl(context, config)
    }
}
```

---

## Consumer Proguard Rules

### 创建 consumer-rules.pro

```proguard
# sdk-core/consumer-rules.pro — 自动应用到消费者的 Proguard 规则

# 保留 Public API
-keep public class com.example.sdk.MySDK { *; }
-keep public class com.example.sdk.Config { *; }
-keep public class com.example.sdk.Result { *; }

# 保留 Kotlin metadata（用于反射）
-keep class kotlin.Metadata { *; }

# 保留 Coroutines
-keepnames class kotlinx.coroutines.internal.MainDispatcherFactory {}
-keepnames class kotlinx.coroutines.CoroutineExceptionHandler {}

# 如果使用 Retrofit
-keepattributes Signature, InnerClasses, EnclosingMethod
-keepattributes RuntimeVisibleAnnotations, RuntimeVisibleParameterAnnotations
-keepclassmembers,allowsetter,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}
```

### 在 build.gradle.kts 中引用

```kotlin
android {
    defaultConfig {
        consumerProguardFiles("consumer-rules.pro")
    }
}
```

---

## Dependency Strategy

### api vs implementation vs compileOnly

```kotlin
dependencies {
    // api — 暴露给消费者（慎用）
    api(project(":sdk-api"))  // 消费者需要直接使用 API 接口
    api(libs.kotlinx.coroutines.core)  // 返回值中有 Flow/suspend

    // implementation — 不传递（推荐）
    implementation(libs.okhttp)  // 内部使用，不暴露
    implementation(libs.gson)

    // compileOnly — 编译时可用，运行时由消费者提供（避免冲突）
    compileOnly(libs.androidx.annotation)  // 消费者通常已有
}
```

### 依赖冲突处理

```kotlin
// 如果 SDK 依赖的库与消费者冲突，用 compileOnly + 文档说明
dependencies {
    compileOnly(libs.okhttp) {
        because("Consumers should provide their own OkHttp version")
    }
}
```

```markdown
<!-- README.md -->
## Dependencies

This SDK requires the following dependencies in your app:

```gradle
implementation("com.squareup.okhttp3:okhttp:4.12.0")
```
```

---

## Maven Central Publishing

### 配置 maven-publish Plugin

```kotlin
// sdk-core/build.gradle.kts
plugins {
    id("com.android.library")
    kotlin("android")
    id("maven-publish")
    id("signing")
}

android {
    publishing {
        singleVariant("release") {
            withSourcesJar()
            withJavadocJar()
        }
    }
}

publishing {
    publications {
        create<MavenPublication>("release") {
            from(components["release"])

            groupId = "com.example"
            artifactId = "my-sdk"
            version = "1.0.0"

            pom {
                name.set("My SDK")
                description.set("A powerful Android SDK")
                url.set("https://github.com/example/my-sdk")

                licenses {
                    license {
                        name.set("The Apache License, Version 2.0")
                        url.set("http://www.apache.org/licenses/LICENSE-2.0.txt")
                    }
                }

                developers {
                    developer {
                        id.set("example")
                        name.set("Example Developer")
                        email.set("dev@example.com")
                    }
                }

                scm {
                    connection.set("scm:git:git://github.com/example/my-sdk.git")
                    developerConnection.set("scm:git:ssh://github.com/example/my-sdk.git")
                    url.set("https://github.com/example/my-sdk")
                }
            }
        }
    }

    repositories {
        maven {
            name = "sonatype"
            url = uri("https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/")
            credentials {
                username = System.getenv("OSSRH_USERNAME")
                password = System.getenv("OSSRH_PASSWORD")
            }
        }
    }
}

signing {
    sign(publishing.publications["release"])
}
```

### GitHub Actions 发布流程

```yaml
# .github/workflows/publish.yml
name: Publish to Maven Central

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Decode GPG Key
        run: |
          echo "${{ secrets.GPG_PRIVATE_KEY }}" | base64 -d > private.gpg
          gpg --import --batch private.gpg

      - name: Publish to Maven Central
        env:
          OSSRH_USERNAME: ${{ secrets.OSSRH_USERNAME }}
          OSSRH_PASSWORD: ${{ secrets.OSSRH_PASSWORD }}
          SIGNING_KEY_ID: ${{ secrets.SIGNING_KEY_ID }}
          SIGNING_PASSWORD: ${{ secrets.SIGNING_PASSWORD }}
        run: ./gradlew publish

      - name: Clean GPG Key
        if: always()
        run: rm -f private.gpg
```

---

## Binary Compatibility Validation

### Binary Compatibility Validator (BCV)

```kotlin
// build.gradle.kts (root)
plugins {
    id("org.jetbrains.kotlinx.binary-compatibility-validator") version "0.14.0"
}

apiValidation {
    ignoredProjects.addAll(listOf("sample-app"))
    nonPublicMarkers.add("com.example.sdk.InternalApi")
}
```

```bash
# 生成 API dump
./gradlew apiDump

# 检查 API 变更
./gradlew apiCheck
```

```
# sdk-api/api/sdk-api.api — 自动生成的 API 签名
public final class com/example/sdk/MySDK {
    public fun initialize(Landroid/content/Context;Lcom/example/sdk/Config;)V
    public fun performAction(Ljava/lang/String;)Lcom/example/sdk/Result;
}
```

### Metalava (Android 官方)

```kotlin
// build.gradle.kts
plugins {
    id("me.tylerbwong.gradle.metalava") version "0.3.4"
}

metalava {
    filename.set("api/current.txt")
    reportLintsAsErrors.set(true)
}
```

### CI Gate 集成

```yaml
# .github/workflows/api-check.yml
name: API Compatibility Check

on:
  pull_request:
    branches: [main]

jobs:
  api-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check API Compatibility
        run: ./gradlew apiCheck

      - name: Fail on Breaking Changes
        if: failure()
        run: |
          echo "API breaking changes detected!"
          echo "Run './gradlew apiDump' to update API signatures"
          exit 1
```

---

## Versioning & Deprecation Strategy

### Semantic Versioning

```
MAJOR.MINOR.PATCH

MAJOR: 不兼容的 API 变更
MINOR: 向后兼容的新功能
PATCH: 向后兼容的 Bug 修复
```

### Deprecation 流程

```kotlin
// 1. 标记 @Deprecated，提供替代方案
@Deprecated(
    message = "Use performActionAsync instead",
    replaceWith = ReplaceWith("performActionAsync(input)"),
    level = DeprecationLevel.WARNING
)
fun performAction(input: String): Result<Output>

// 2. 提供新 API
suspend fun performActionAsync(input: String): Result<Output>
```

### Deprecation 时间表

| 版本 | 动作 |
|------|------|
| 1.0.0 | 发布 API |
| 1.1.0 | 标记 @Deprecated (WARNING) |
| 1.2.0 | 升级为 ERROR |
| 2.0.0 | 移除 API |

### CHANGELOG.md

```markdown
# Changelog

## [1.2.0] - 2026-02-15

### Added
- New `performActionAsync` API with coroutine support

### Deprecated
- `performAction` is now ERROR level, will be removed in 2.0.0

### Fixed
- Memory leak in NetworkClient

## [1.1.0] - 2026-01-10

### Added
- Support for custom timeout configuration

### Deprecated
- `performAction` (use `performActionAsync` instead)
```

---

## Dokka Documentation

### 配置 Dokka Plugin

```kotlin
// build.gradle.kts
plugins {
    id("org.jetbrains.dokka") version "1.9.10"
}

tasks.dokkaHtml.configure {
    outputDirectory.set(buildDir.resolve("dokka"))

    dokkaSourceSets {
        named("main") {
            moduleName.set("My SDK")
            includes.from("Module.md")

            sourceLink {
                localDirectory.set(file("src/main/kotlin"))
                remoteUrl.set(URL("https://github.com/example/my-sdk/tree/main/src/main/kotlin"))
                remoteLineSuffix.set("#L")
            }
        }
    }
}
```

### KDoc 注释规范

```kotlin
/**
 * SDK 主入口，负责初始化与核心操作。
 *
 * ## 使用示例
 * ```kotlin
 * val sdk = MySDKFactory.create(context, Config(apiKey = "xxx"))
 * sdk.initialize(context, config)
 * val result = sdk.performAction("input")
 * ```
 *
 * @property context Android Context
 * @property config SDK 配置
 * @see Config
 * @since 1.0.0
 */
interface MySDK {

    /**
     * 初始化 SDK。
     *
     * 必须在使用其他 API 前调用。
     *
     * @param context Android Application Context
     * @param config SDK 配置对象
     * @throws IllegalStateException 如果已初始化
     */
    fun initialize(context: Context, config: Config)

    /**
     * 执行核心操作。
     *
     * @param input 输入字符串
     * @return [Result.Success] 包含 [Output]，或 [Result.Error] 包含异常
     * @sample com.example.sdk.samples.performActionSample
     */
    fun performAction(input: String): Result<Output>
}
```

### 生成文档

```bash
# 生成 HTML 文档
./gradlew dokkaHtml

# 生成 Javadoc JAR（用于 Maven Central）
./gradlew dokkaJavadocJar
```

---

## Sample App Design

### Sample App 模块结构

```
sample-app/
├── src/main/
│   ├── kotlin/
│   │   └── com/example/sample/
│   │       ├── MainActivity.kt
│   │       ├── BasicUsageFragment.kt
│   │       ├── AdvancedUsageFragment.kt
│   │       └── ErrorHandlingFragment.kt
│   └── res/
└── build.gradle.kts
```

### build.gradle.kts

```kotlin
plugins {
    id("com.android.application")
    kotlin("android")
}

dependencies {
    // 使用本地 SDK 模块
    implementation(project(":sdk-core"))

    // 或使用已发布版本
    // implementation("com.example:my-sdk:1.0.0")
}
```

### 示例代码组织

```kotlin
// BasicUsageFragment.kt — 基础用法
class BasicUsageFragment : Fragment() {

    private lateinit var sdk: MySDK

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 1. 创建 SDK 实例
        sdk = MySDKFactory.create(
            requireContext(),
            Config(apiKey = "demo_key")
        )

        // 2. 初始化
        sdk.initialize(requireContext(), Config(apiKey = "demo_key"))

        // 3. 执行操作
        lifecycleScope.launch {
            when (val result = sdk.performAction("test")) {
                is Result.Success -> showSuccess(result.data)
                is Result.Error -> showError(result.exception)
            }
        }
    }
}
```

```kotlin
// AdvancedUsageFragment.kt — 进阶用法
class AdvancedUsageFragment : Fragment() {

    private val sdk by lazy {
        MySDKFactory.create(
            requireContext(),
            Config(
                apiKey = "demo_key",
                enableLogging = true
            )
        )
    }

    // 展示自定义配置、错误处理、并发操作等
}
```

### README.md 示例

```markdown
# Sample App

演示 My SDK 的各种用法。

## 运行

```bash
./gradlew :sample-app:installDebug
```

## 示例清单

- **BasicUsageFragment**: 基础初始化与调用
- **AdvancedUsageFragment**: 自定义配置与并发操作
- **ErrorHandlingFragment**: 错误处理与重试策略
```

---

## Quick Checklist

- [ ] Public API 最小化，只暴露必要接口
- [ ] 使用 internal/private 隐藏实现细节
- [ ] 多模块分层（api + core + optional modules）
- [ ] Consumer Proguard Rules 配置完成
- [ ] 依赖策略明确（api vs implementation vs compileOnly）
- [ ] Maven Central 发布流程配置完成
- [ ] Binary Compatibility Validator 纳入 CI
- [ ] Semantic Versioning 与 Deprecation 策略明确
- [ ] Dokka 文档生成配置完成
- [ ] KDoc 注释覆盖所有 Public API
- [ ] Sample App 展示所有核心用法
- [ ] CHANGELOG.md 记录所有版本变更
