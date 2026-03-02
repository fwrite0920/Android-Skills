---
name: Supply Chain Security
description: 依赖治理、SCA、签章与密钥管理
---

# Supply Chain Security (供应链安全)

## Instructions
- 先盘点依赖来源与版本策略
- 创建 SCA 扫描与审核流程
- 一次只强化一个供应链节点
- 完成后对照 Quick Checklist

## When to Use
- 项目依赖多、更新频繁
- 发布前需要风险检查
- 需要创建依赖治理标准

## Example Prompts
- "请设计依赖版本锁定与更新策略"
- "帮我加上 SCA 扫描与风险门槛"
- "请创建密钥与签章管理规范"
- "帮我配置 Gradle Dependency Verification"

## Workflow
1. 创建依赖来源与版本锁定策略
2. 加入 SCA 扫描与审核流程
3. 设置签章与密钥管理规范
4. 将风险门槛纳入 CI Gate

## Practical Notes (2026)
- 版本锁定与审核是最小安全基线
- 依赖更新与安全修补分开处理
- SCA 结果必须有处置规则

## Minimal Template
```
目标:
依赖来源:
版本策略:
SCA 门槛:
验收: Quick Checklist
```

---

## Dependency Governance

### Gradle Dependency Verification

```bash
# 生成 verification-metadata.xml
./gradlew --write-verification-metadata sha256,pgp help
```

```xml
<!-- gradle/verification-metadata.xml -->
<verification-metadata>
    <configuration>
        <verify-metadata>true</verify-metadata>
        <verify-signatures>true</verify-signatures>
    </configuration>
    <components>
        <component group="com.google.dagger" name="hilt-android" version="<project-verified-version>">
            <artifact name="hilt-android-<project-verified-version>.aar">
                <sha256 value="abc123..." />
            </artifact>
        </component>
    </components>
</verification-metadata>
```

### Version Catalog 作为单一来源

```toml
# gradle/libs.versions.toml — 所有依赖版本集中管理
[versions]
kotlin = "<project-verified-version>"
hilt = "<project-verified-version>"
retrofit = "<project-verified-version>"

[libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
```

### 依赖来源白名单

```kotlin
// settings.gradle.kts — 限制仓库来源
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        // 禁止 jcenter()、jitpack 等未审核来源
    }
}
```

---

## SCA / Vulnerability Scanning

### OWASP Dependency-Check

```kotlin
// build.gradle.kts
plugins {
    id("org.owasp.dependencycheck") version "<project-verified-version>"
}

dependencyCheck {
    failBuildOnCVSS = 7.0f  // CVSS >= 7 阻挡构建
    formats = listOf("HTML", "JSON")
    suppressionFile = "config/owasp-suppressions.xml"
}
```

```bash
# 运行扫描
./gradlew dependencyCheckAnalyze
```

### Suppressions 配置

```xml
<!-- config/owasp-suppressions.xml -->
<suppressions xmlns="https://jeremylong.github.io/DependencyCheck/dependency-suppression.1.3.xsd">
    <suppress>
        <notes>误报：此 CVE 不影响 Android 使用场景</notes>
        <cve>CVE-2023-XXXXX</cve>
    </suppress>
</suppressions>
```

### Renovate / Dependabot 自动更新

文件名：`renovate.json`

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:base"],
  "packageRules": [
    {
      "matchUpdateTypes": ["major"],
      "labels": ["major-update"],
      "automerge": false
    },
    {
      "matchUpdateTypes": ["minor", "patch"],
      "matchPackagePatterns": ["androidx.*", "com.google.*"],
      "automerge": true,
      "automergeType": "pr"
    }
  ],
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security"]
  }
}
```

```yaml
# .github/dependabot.yml (替代方案)
version: 2
updates:
  - package-ecosystem: "gradle"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
```

---

## Signing & Secrets Management

### Keystore 安全管理

```kotlin
// build.gradle.kts — 从环境变量读取签章信息
android {
    signingConfigs {
        create("release") {
            storeFile = file(System.getenv("KEYSTORE_PATH") ?: "release.keystore")
            storePassword = System.getenv("KEYSTORE_PASSWORD") ?: ""
            keyAlias = System.getenv("KEY_ALIAS") ?: ""
            keyPassword = System.getenv("KEY_PASSWORD") ?: ""
        }
    }

    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

### GitHub Actions Secrets

```yaml
# .github/workflows/release.yml
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Decode Keystore
        run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > release.keystore

      - name: Build Release
        env:
          KEYSTORE_PATH: release.keystore
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
        run: ./gradlew assembleRelease

      - name: Clean Keystore
        if: always()
        run: rm -f release.keystore
```

### .gitignore 安全规则

```gitignore
# 密钥与敏感文件
*.keystore
*.jks
local.properties
google-services.json
secrets.properties
```

---

## CI Gate Integration

### 完整安全 Pipeline

```yaml
# .github/workflows/security-gate.yml
name: Security Gate
on:
  pull_request:
    branches: [main]

jobs:
  dependency-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: OWASP Dependency Check
        run: ./gradlew dependencyCheckAnalyze

      - name: Check for High Vulnerabilities
        run: |
          HIGH=$(jq '[.dependencies[].vulnerabilities[]? | select(.cvssv3?.baseScore >= 7)] | length' build/reports/dependency-check-report.json)
          if [ "$HIGH" -gt 0 ]; then
            echo "Found $HIGH high/critical vulnerabilities"
            exit 1
          fi

      - name: Verify Dependencies
        run: ./gradlew --dependency-verification strict help

      - name: Upload Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: build/reports/dependency-check-report.html
```

### 风险处置规则

| CVSS 分数 | 等级 | 处置 |
|-----------|------|------|
| 9.0 - 10.0 | Critical | 立即修补，阻挡合并 |
| 7.0 - 8.9 | High | 48 小时内修补，阻挡合并 |
| 4.0 - 6.9 | Medium | 标注 Issue，本 Sprint 修补 |
| 0.1 - 3.9 | Low | 标注 Issue，下次更新时处理 |

---

## Quick Checklist

- [ ] Version Catalog 作为依赖单一来源
- [ ] Dependency Verification 启用（sha256 + pgp）
- [ ] 仓库来源白名单（禁止未审核来源）
- [ ] OWASP Dependency-Check 纳入 CI（CVSS >= 7 阻挡）
- [ ] Renovate/Dependabot 自动更新配置
- [ ] Keystore 与 Secrets 不进版控
- [ ] 签章流程可追踪（CI 环境变量注入）
- [ ] 风险处置规则明确且有 SLA
