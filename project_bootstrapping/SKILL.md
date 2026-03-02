---
name: Project Bootstrapping
description: 快速创建项目骨架、Gradle Convention Plugins 与标准化架构
---

# Project Bootstrapping (项目快速建置)

## Instructions
- 仅在新项目或新模块起步时使用
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
1. 先创建 Template 与目录结构
2. 再落实 Convention Plugins 与 Version Catalog
3. 最后用 Quick Checklist 验收

## Practical Notes (2026)
- 默认创建 CI Gate：lint、detekt、unit test、assemble
- 新项目先创建 Baseline Profile 量测框架
- Version Catalog 作为单一依赖来源

## Minimal Template
```
目标: 
模块范围: 
Convention Plugins: 
CI Gate: 
验收: Quick Checklist
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
class AndroidLibraryConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.android.library")
                apply("org.jetbrains.kotlin.android")
            }
            
            extensions.configure<LibraryExtension> {
                compileSdk = 34
                defaultConfig.minSdk = 24
                
                compileOptions {
                    sourceCompatibility = JavaVersion.VERSION_17
                    targetCompatibility = JavaVersion.VERSION_17
                }
            }
        }
    }
}
```

### 使用方式 (feature module)

```kotlin
// feature/login/build.gradle.kts
plugins {
    id("mycompany.android.feature")  // 一行搞定！
}

dependencies {
    implementation(projects.core.domain)
}
```

---

## Version Catalog (libs.versions.toml)

```toml
[versions]
kotlin = "<project-verified-version>"
compose-bom = "<project-verified-version>"
hilt = "<project-verified-version>"
room = "<project-verified-version>"
retrofit = "<project-verified-version>"

[libraries]
# Compose
compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
compose-ui = { group = "androidx.compose.ui", name = "ui" }
compose-material3 = { group = "androidx.compose.material3", name = "material3" }

# DI
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }

[bundles]
compose = ["compose-ui", "compose-material3"]

[plugins]
android-application = { id = "com.android.application", version = "<project-verified-version>" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

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
- [ ] 使用 Template Repository
- [ ] Convention Plugins 设置完成
- [ ] Version Catalog 配置
- [ ] Detekt/Ktlint 集成
- [ ] CI/CD 基础 Pipeline

### 新模块创建
- [ ] 使用正确的 Convention Plugin
- [ ] 遵循 Package Structure
- [ ] 加入 Navigation Graph (如需要)
