---
name: DevOps and Security
description: CI/CD 自动化、Gradle 优化与应用程序安全加固
---

# DevOps and Security (DevOps 与资安)

## Instructions
- 仅在发布准备或流程自动化需求时使用
- 先填写 Required Inputs（CI 平台、签章策略、密钥来源）
- 依照下方章节顺序套用
- 一次只处理一个 pipeline 或安全措施
- 完成后对照 Quick Checklist

## When to Use
- Scenario E：App 发布准备

## Example Prompts
- "请依照 Build Speed Optimization，调整 Gradle 设置"
- "用 CI Quality Gates 章节创建 GitHub Actions"
- "请参考 Security Hardening，检查 secrets 与网络安全"

## Workflow
1. 先确认 Required Inputs（CI provider、发布渠道、secret 管理）
2. 创建 Build Speed 与 CI Quality Gates
3. 导入 Fastlane 与 Security Hardening
4. 执行 Delivery Gate 并记录发布演练结果
5. 用 Quick Checklist 验收

## Practical Notes (2026)
- 依赖来源与版本必有审核与锁定策略
- Secrets 仅能通过环境变量或安全保存
- CI Gate 必含 Lint/Detekt/Test/Build
- Pipeline 失败要可重现，禁止人工临时绕过
- 发布流程至少每月 dry-run 一次，避免到发布日失效

## Minimal Template
```
目标: 
CI Gate: 
CI Provider:
Secrets 来源:
安全措施: 
发版流程: 
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `CI provider`（GitHub Actions / GitLab CI / Jenkins）
- `发布渠道`（Play Internal/Alpha/Production）
- `Secret 管理策略`（CI Secret / Vault / KMS）
- `签章策略`（keystore 来源、轮换周期）
- `阻挡条件`（哪些任务失败即阻挡发布）

## Deliverables (完成后交付物)

- 可执行 `CI pipeline`（lint/detekt/test/assemble）
- `发布流水线`（Fastlane 或等效脚本）
- `安全加固配置`（Network Security / Pinning / Secret policy）
- `告警与追踪`（失败通知、日志可追溯）
- `发布演练记录`（dry-run 结果）

## Delivery Gate (验收门槛)

```bash
./gradlew lint detekt test assemble
./gradlew bundleRelease
```

> 若使用 Fastlane，需补跑 `fastlane lane` 的 dry-run 或 internal track 发布演练。

---

## Build Speed Optimization

### Configuration Cache

```kotlin
// gradle.properties
org.gradle.configuration-cache=true
org.gradle.configuration-cache.problems=warn
```

### Build Cache

```kotlin
// settings.gradle.kts
buildCache {
    local {
        directory = File(rootDir, "build-cache")
        removeUnusedEntriesAfterDays = 7
    }
    
    // 企业级：Remote cache
    remote<HttpBuildCache> {
        url = uri("https://cache.example.com/")
        isPush = System.getenv("CI") != null
    }
}
```

### Parallel Execution

```kotlin
// gradle.properties
org.gradle.parallel=true
org.gradle.caching=true
org.gradle.jvmargs=-Xmx4g -XX:+HeapDumpOnOutOfMemoryError
```

---

## CI Quality Gates

### GitHub Actions 范例

```yaml
name: Android CI

on:
  pull_request:
    branches: [main, develop]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Lint
        run: ./gradlew lintDebug
      
      - name: Detekt
        run: ./gradlew detekt
      
      - name: Unit Tests
        run: ./gradlew testDebugUnitTest
      
      - name: Build
        run: ./gradlew assembleDebug
```

### Danger for PR Review

```ruby
# Dangerfile
# APK Size Check
apk_size = File.size("app/build/outputs/apk/debug/app-debug.apk") / 1024.0 / 1024.0
warn "APK size is #{apk_size.round(2)}MB" if apk_size > 50

# Kotlin Files changed
kotlin_files = git.modified_files.select { |f| f.end_with?(".kt") }
warn "Large PR with #{kotlin_files.count} Kotlin files" if kotlin_files.count > 20
```

---

## Fastlane Automation

### Fastfile

```ruby
default_platform(:android)

platform :android do
  
  desc "Deploy to Play Store Internal Track"
  lane :internal do
    gradle(task: "bundleRelease")
    upload_to_play_store(
      track: "internal",
      aab: "app/build/outputs/bundle/release/app-release.aab"
    )
  end
  
  desc "Promote Internal to Production"
  lane :promote do
    upload_to_play_store(
      track: "internal",
      track_promote_to: "production",
      skip_upload_apk: true,
      skip_upload_aab: true
    )
  end
end
```

---

## Security Hardening

### Secrets Management

```kotlin
// ❌ 不要把真实密钥写进 BuildConfig / strings.xml / assets
// ✅ 用 secrets-gradle-plugin 管理「可公开但需分环境」的配置
// build.gradle.kts
plugins {
    id("com.google.android.libraries.mapsplatform.secrets-gradle-plugin")
}

secrets {
    propertiesFileName = "secrets.properties"            // 本地/CI 注入，不进版控
    defaultPropertiesFileName = "local.defaults.properties" // 可提交的占位值
}
```

```kotlin
// 真正敏感凭证：由后端签发短时 token，不内置在 APK
interface TokenApi {
    suspend fun getEphemeralToken(): String
}
```

```gitignore
secrets.properties
local.properties
```

### Certificate Pinning

```kotlin
// OkHttp
val certificatePinner = CertificatePinner.Builder()
    .add("example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .build()

val client = OkHttpClient.Builder()
    .certificatePinner(certificatePinner)
    .build()
```

### Root Detection

```kotlin
class RootDetection {
    fun isDeviceRooted(): Boolean {
        return checkRootBinaries() || checkSuExists() || checkRootCloaking()
    }
    
    private fun checkRootBinaries(): Boolean {
        val paths = arrayOf("/system/bin/su", "/system/xbin/su", "/sbin/su")
        return paths.any { File(it).exists() }
    }
}
```

### Network Security Config

```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <pin-set>
            <pin digest="SHA-256">...</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

---

## Quick Checklist

- [ ] Required Inputs 已填写并冻结（CI/发布/密钥策略）
- [ ] Build Cache 激活 (Local + Remote)
- [ ] CI 包含 Lint, Detekt, Unit Test
- [ ] Fastlane 自动化部署
- [ ] Secrets 不进版控
- [ ] Certificate Pinning 激活
- [ ] Network Security Config 禁止 Cleartext
- [ ] Delivery Gate 已执行并通过
