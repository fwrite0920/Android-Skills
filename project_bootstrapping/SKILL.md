---
name: Project Bootstrapping
description: 快速创建项目骨架、Gradle Convention Plugins 与标准化架构
---

# Project Bootstrapping (项目快速建置)

## Instructions
- 仅在新项目或新模块起步时使用
- 先填写 Required Inputs，避免边做边改
- 依照下方章节顺序创建骨架
- 一次只处理一个子系统（插件、版本、结构）
- 完成后对照 Quick Checklist

## When to Use
- Scenario A：从零创建新项目

## Example Prompts
- "请依照 One-Command Setup，创建公司模板的项目骨架"
- "依照 Gradle Convention Plugins 章节，创建 feature module 插件"
- "请根据 Package Structure 规划模块与套件配置"

## Workflow
1. 先确认 Required Inputs（包名、SDK、JDK、CI）
2. 创建 Template 与目录结构
3. 落实 Convention Plugins 与 Version Catalog（禁止硬编码版本）
4. 生成 CI Gate 与验收指令，再用 Quick Checklist 收尾

## Practical Notes (2026)
- 默认创建 CI Gate：lint、detekt、unit test、assemble
- 新项目先创建 Baseline Profile 量测框架
- Version Catalog 作为单一依赖来源
- SDK/JVM 版本集中在 `libs.versions.toml`，Convention Plugin 只读取不写死
- 每次脚手架完成后至少执行一次 `./gradlew help` 与 `./gradlew :app:assembleDebug`

## Minimal Template
```
目标: 
packageName:
compileSdk/minSdk/targetSdk:
JDK/JVM Target:
模块范围: 
Convention Plugins: 
CI Gate: 
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `packageName`（例如：`com.company.app`）
- `compileSdk / targetSdk / minSdk`
- `jdkVersion / jvmTarget`
- `CI provider`（GitHub Actions / GitLab CI / Jenkins）
- `模块清单`（`app`、`core/*`、`feature/*`）

## Deliverables (完成后交付物)

- `settings.gradle.kts`：含模块声明与 `includeBuild("build-logic")`
- `gradle/libs.versions.toml`：集中管理版本与 plugins
- `build-logic/convention/...`：至少含 app/library/feature plugin
- `app/build.gradle.kts`：不直接写版本号，全部走 Version Catalog
- `CI workflow`：至少包含 lint、detekt、test、assemble
- 可执行验收命令：`./gradlew help`、`./gradlew :app:assembleDebug`

## Project Gate (验收门槛)

```bash
./gradlew help
./gradlew :app:assembleDebug
```

---

## One-Command Setup

### GitHub Template Repository

创建公司内部的 Template Repository，包含：

```
my-company-android-template/
├── app/
├── build-logic/
│   └── convention/           # Convention Plugins
├── core/
│   ├── common/
│   ├── data/
│   ├── domain/
│   ├── network/
│   └── ui/
├── feature/
│   └── sample/
├── gradle/
│   └── libs.versions.toml    # Version Catalog
├── .editorconfig
├── detekt.yml
└── README.md
```

### 使用方式

```bash
# GitHub Template → Use this template
# 或使用 gh cli
gh repo create my-new-app --template my-company/android-template
```

### 在现有仓库初始化（不走 GitHub Template）

```bash
mkdir -p build-logic/convention/src/main/kotlin
mkdir -p gradle app core/{common,data,domain,network,ui} feature
```

---

## Gradle Convention Plugins

### 目录结构

```
build-logic/
├── convention/
│   ├── build.gradle.kts
│   └── src/main/kotlin/
│       ├── AndroidApplicationConventionPlugin.kt
│       ├── AndroidLibraryConventionPlugin.kt
│       ├── AndroidComposeConventionPlugin.kt
│       └── AndroidFeatureConventionPlugin.kt
└── settings.gradle.kts
```

### settings.gradle.kts (root)

```kotlin
pluginManagement {
    includeBuild("build-logic")
}
```

### AndroidLibraryConventionPlugin.kt

```kotlin
import com.android.build.api.dsl.LibraryExtension
import org.gradle.api.JavaVersion
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.api.artifacts.VersionCatalogsExtension
import org.gradle.kotlin.dsl.configure
import org.gradle.kotlin.dsl.getByType

class AndroidLibraryConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) = with(target) {
        pluginManager.apply("com.android.library")
        pluginManager.apply("org.jetbrains.kotlin.android")

        val libs = extensions.getByType<VersionCatalogsExtension>().named("libs")
        val compileSdk = libs.findVersion("compileSdk").get().requiredVersion.toInt()
        val minSdk = libs.findVersion("minSdk").get().requiredVersion.toInt()
        val jvmTarget = libs.findVersion("jvmTarget").get().requiredVersion

        extensions.configure<LibraryExtension> {
            this.compileSdk = compileSdk

            defaultConfig {
                this.minSdk = minSdk
            }

            compileOptions {
                sourceCompatibility = JavaVersion.toVersion(jvmTarget)
                targetCompatibility = JavaVersion.toVersion(jvmTarget)
            }
        }
    }
}
```

### 避免版本散落

```kotlin
// ❌ 不要在 module build.gradle.kts 中写死版本
dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
}

// ✅ 统一从 Version Catalog 读取
dependencies {
    implementation(libs.retrofit)
}
```

### 使用方式 (feature module)

```kotlin
// feature/login/build.gradle.kts
plugins {
    id("mycompany.android.feature")
}

dependencies {
    implementation(projects.core.domain)
}
```

---

## Version Catalog (libs.versions.toml)

```toml
[versions]
# 以下请替换为团队已验证组合
agp = "<agp-verified>"
kotlin = "<kotlin-verified>"
compileSdk = "<compile-sdk>"
targetSdk = "<target-sdk>"
minSdk = "<min-sdk>"
jvmTarget = "<jvm-target>"
composeBom = "<compose-bom-verified>"
hilt = "<hilt-verified>"
room = "<room-verified>"
retrofit = "<retrofit-verified>"

[libraries]
# Compose
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }

# DI
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }

# Network
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }

[bundles]
compose = ["compose-ui", "compose-material3"]

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
android-library = { id = "com.android.library", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

> 建议：依赖升级在固定节奏任务中统一进行，不在功能 PR 临时改版本。

---

## Package Structure (Feature-based)

```
com.example.app/
├── core/
│   ├── common/          # 共用工具 (Extensions, Utils)
│   ├── data/            # Repository 实作
│   ├── domain/          # UseCase, Entity
│   ├── network/         # Retrofit, API
│   └── ui/              # Design System, Theme
├── feature/
│   ├── auth/
│   │   ├── data/        # Feature-specific data
│   │   ├── domain/      # Feature-specific use cases
│   │   └── ui/          # Screens, ViewModels
│   └── home/
└── app/                 # Application, DI, Navigation
```

---

## Quick Checklist

### 新项目创建
- [ ] Required Inputs 已填写并冻结
- [ ] 使用 Template Repository 或完成本地初始化
- [ ] Convention Plugins 设置完成
- [ ] Version Catalog 配置且模块内无硬编码版本
- [ ] Detekt/Ktlint 集成
- [ ] CI/CD 基础 Pipeline（lint/detekt/test/assemble）
- [ ] `./gradlew help` 与 `./gradlew :app:assembleDebug` 可通过

### 新模块创建
- [ ] 使用正确的 Convention Plugin
- [ ] 遵循 Package Structure
- [ ] 加入 Navigation Graph (如需要)
- [ ] 不在 module 内直接写依赖版本
