---
name: Coding Style Conventions
description: Kotlin 代码规范、Linter 配置与 Code Review 检核标准
---

# Coding Style Conventions (代码规范)

## Instructions
- 确认需求属于本技能范围（命名、格式、检核）
- 先填写 Required Inputs（规则范围、模块、门槛）并冻结
- 依照下方章节顺序套用
- 一次只调整一类规范，避免混杂变更
- 完成后对照 Quick Checklist

## When to Use
- Scenario A：新项目创建规范
- Scenario C：旧项目现代化基准

## Example Prompts
- "请参考本技能的 Naming Conventions，检查这段 Kotlin 代码命名是否一致"
- "依照 Detekt 与 Ktlint 配置章节，帮我创建项目规范"
- "请用 Quick Checklist 审视这个 PR 的风格问题"

## Workflow
1. 先确认 Required Inputs（适用模块、规则级别、阻挡条件）
2. 对照 Naming Conventions 设置命名规则
3. 依 Detekt / Ktlint 配置落实到项目
4. 执行 Style Gate，再用 Quick Checklist 验收

## Practical Notes (2026)
- CI Gate 仅针对变更文件运行 Lint/Detekt/Ktlint
- 规范调整与功能变更分开提交，方便回溯
- Code Review 以 Checklist 作为硬性验收
- 禁止在功能 PR 临时放宽规则，规则变更需独立 PR
- Baseline 只减不增，新增违规必须当次修复

## Minimal Template
```
目标: 
适用范围: 
阻挡门槛:
Baseline 策略:
规范重点: 
检核方式: 
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `适用范围`（全仓 / 模块 / feature）
- `规则门槛`（warning / error / 阻挡合并）
- `Baseline 策略`（是否允许、收敛节奏）
- `CI 执行范围`（全量或仅变更文件）
- `代码审查口径`（review 必查项）

## Deliverables (完成后交付物)

- `.editorconfig` 与命名规范文档
- `detekt.yml` / `ktlint` 配置与 baseline
- `CI style gate`（lint/detekt/ktlint）
- `PR 模板`（包含风格验收条目）
- `规则变更记录`（说明原因与影响）

## Style Gate (验收门槛)

```bash
./gradlew lint
./gradlew detekt
./gradlew ktlintCheck
```

> 旧项目使用 baseline 时，必须在 PR 说明中写明本次减少了哪些 baseline 项目。

---

## Naming Conventions (命名规则)

| 类型 | 规则 | 范例 |
|------|------|------|
| Class / Interface | `PascalCase` | `UserRepository`, `Drawable` |
| Function / Method | `camelCase` | `getUserById()`, `onClick()` |
| Variable / Property | `camelCase` | `userName`, `isLoading` |
| Constant (top-level/object) | `SCREAMING_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Package | `lowercase` (no underscores) | `com.example.feature.auth` |
| @Composable Function | `PascalCase` | `LoginScreen()`, `UserCard()` |
| Backing Property | `_camelCase` | `private val _uiState` |

### Compose Specific

```kotlin
// ✅ Composable 函数用 PascalCase (像 Class)
@Composable
fun UserProfileCard(user: User, modifier: Modifier = Modifier) { }

// ✅ State holder 用 remember + camelCase
val scrollState = rememberScrollState()

// ✅ Event callback 用 on 前缀
onUserClick: (User) -> Unit
```

---

## Detekt Configuration

### 安装与设置

```kotlin
// build.gradle.kts (project-level)
plugins {
    id("io.gitlab.arturbosch.detekt") version "<project-verified-version>"
}

// build.gradle.kts (app-level)
detekt {
    buildUponDefaultConfig = true
    config.setFrom("$rootDir/config/detekt/detekt.yml")
    baseline = file("$rootDir/config/detekt/baseline.xml")
}
```

### 建议规则集 (detekt.yml)

```yaml
complexity:
  LongMethod:
    threshold: 30
  LongParameterList:
    functionThreshold: 6
    constructorThreshold: 8
  
naming:
  FunctionNaming:
    excludes: ['**/composables/**']  # Compose 用 PascalCase
  
style:
  MaxLineLength:
    maxLineLength: 120
  WildcardImport:
    active: true
  MagicNumber:
    ignorePropertyDeclaration: true
    ignoreCompanionObjectPropertyDeclaration: true
```

### Baseline 机制 (旧项目适用)

```bash
# 生成 baseline，忽略现有问题
./gradlew detektBaseline

# 之后只检查新代码的违规
./gradlew detekt
```

---

## Ktlint Configuration

### 安装

```kotlin
// build.gradle.kts
plugins {
    id("org.jlleitschuh.gradle.ktlint") version "<project-verified-version>"
}

ktlint {
    android.set(true)
    outputColorName.set("RED")
}
```

### .editorconfig (与 IDE 同步)

```ini
[*.{kt,kts}]
max_line_length = 120
indent_size = 4
insert_final_newline = true

# Ktlint specific
ktlint_standard_no-wildcard-imports = enabled
ktlint_standard_trailing-comma-on-call-site = enabled
ktlint_standard_trailing-comma-on-declaration-site = enabled
```

---

## Documentation Standards (KDoc)

### 何时该写

- ✅ Public API (SDK, Library)
- ✅ 复杂的业务逻辑
- ✅ 非直观的参数或回传值
- ✅ 重要的设计决策

### 何时不该写

- ❌ 自解释的代码 (e.g., `fun getUserName(): String`)
- ❌ 覆写的方法 (继承父类文档)
- ❌ 简单的 CRUD 操作

### 范例

```kotlin
/**
 * 根据优先级排序并过滤过期的任务。
 *
 * @param tasks 待处理的任务列表
 * @param now 用于判断过期的时间点，默认为当前时间
 * @return 依优先级排序的有效任务，过期任务会被过滤
 * @throws IllegalArgumentException 如果 tasks 包含 null 元素
 */
fun filterAndSort(tasks: List<Task>, now: Instant = Instant.now()): List<Task>
```

---

## Quick Checklist

- [ ] Required Inputs 已填写并冻结（范围/门槛/baseline）

### Naming & Style
- [ ] 命名是否遵循上述规则？
- [ ] Compose 函数是否用 PascalCase？
- [ ] 是否有 Magic Number？

### Structure
- [ ] 函数是否过长 (> 30 行)？
- [ ] 参数是否过多 (> 6 个)？
- [ ] 是否有 God Class 倾向？

### Safety
- [ ] Nullable 处理是否安全？
- [ ] 是否有潜在的 NPE？
- [ ] 异常处理是否完善？

### Compose Specific
- [ ] Modifier 是否为第一个可选参数？
- [ ] State 是否正确 hoist？
- [ ] 是否有 unstable 的参数导致不必要重组？
- [ ] Style Gate（lint/detekt/ktlintCheck）已执行并通过
