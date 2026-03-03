# Android Skills 完整使用教学

这份指南详细说明如何在各种 AI 工具中使用这 17 个 Android 技能。

---

## 📚 目录

1. [Skills 总览](#skills-总览)
2. [通用使用原则](#通用使用原则)
3. [各 AI 工具详细教学](#各-ai-工具详细教学)
   - [Antigravity (Gemini CLI/VS Code)](#a-antigravity)
   - [Cursor](#b-cursor)
   - [Windsurf](#c-windsurf)
   - [Roo Code (VS Code Extension)](#d-roo-code)
   - [Cline (VS Code Extension)](#e-cline)
   - [Aider (CLI)](#f-aider-cli)
   - [Claude Code (CLI)](#g-claude-code-cli)
   - [GitHub Copilot](#h-github-copilot)
   - [Amazon Q Developer](#i-amazon-q-developer)
   - [JetBrains AI Assistant](#j-jetbrains-ai-assistant)
   - [Claude Web / Claude Projects](#k-claude-web)
   - [ChatGPT / Custom GPT](#l-chatgpt)
   - [Google AI Studio / Gemini](#m-google-ai-studio)
   - [LLM CLI](#n-llm-cli)
   - [Codex CLI (OpenAI)](#o-codex-cli-openai)
   - [Gemini CLI (Google)](#p-gemini-cli-google)
   - [OpenInterpreter](#q-openinterpreter)
   - [Fabric](#r-fabric)
   - [Ollama + Open WebUI](#s-ollama--open-webui)
   - [OpenCode CLI (opencode.ai)](#t-opencode-cli-opencodeai)
4. [场景导向完整范例](#场景导向完整范例)
5. [进阶集成技巧](#进阶集成技巧)
6. [常见问题](#常见问题)

---

## Skills 总览

| # | Skill 名称 | 用途简述 | 文件路径 |
|---|-----------|---------|---------|
| 1 | `skill_index` | 技能导航中心 | `skill_index/SKILL.md` |
| 2 | `coding_style_conventions` | 代码规范、Detekt/Ktlint | `coding_style_conventions/SKILL.md` |
| 3 | `project_bootstrapping` | 项目快速建置、Convention Plugins | `project_bootstrapping/SKILL.md` |
| 4 | `ui_ux_engineering` | Design System、Accessibility | `ui_ux_engineering/SKILL.md` |
| 5 | `dependency_injection_mastery` | Hilt 进阶、Multibinding | `dependency_injection_mastery/SKILL.md` |
| 6 | `data_layer_mastery` | Room、Retrofit、Offline-First | `data_layer_mastery/SKILL.md` |
| 7 | `navigation_patterns` | Deep Links、跨模块导航 | `navigation_patterns/SKILL.md` |
| 8 | `legacy_rapid_expansion` | 旧项目快速扩充、Islanding | `legacy_rapid_expansion/SKILL.md` |
| 9 | `tech_stack_migration` | Rx→Flow、View→Compose | `tech_stack_migration/SKILL.md` |
| 10 | `testing_legacy_strategies` | 遗留代码测试策略 | `testing_legacy_strategies/SKILL.md` |
| 11 | `deep_performance_tuning` | 性能深度优化 | `deep_performance_tuning/SKILL.md` |
| 12 | `devops_and_security` | CI/CD、安全加固 | `devops_and_security/SKILL.md` |
| 13 | `crash_monitoring` | Crashlytics、ANR 分析 | `crash_monitoring/SKILL.md` |
| 14 | `kotlin_multiplatform` | KMP 跨平台架构 | `kotlin_multiplatform/SKILL.md` |
| 15 | `observability_first` | 可观测性优先与指标闭环 | `observability_first/SKILL.md` |
| 16 | `supply_chain_security` | 依赖治理与供应链安全 | `supply_chain_security/SKILL.md` |
| 17 | `sdk_development` | SDK/Library 开发全生命周期 | `sdk_development/SKILL.md` |

---

## 通用使用原则

### 原则 1：Context 管理 (不浪费 Token)

```
❌ 错误做法：把 17 个文件全部丢给 AI
   → Token 浪费、AI 注意力分散、回应品质下降

✅ 正确做法：根据任务只加载 2-3 个相关技能
   → Token 节省、AI 专注、回应精准
```

### 原则 2：使用场景路由 (Scenario Router)

先参考 `skill_index/SKILL.md` 选择技能组合：

| 场景 | 描述 | 建议加载的技能 |
|------|------|--------------|
| A | 从零创建新项目 | `project_bootstrapping` + `coding_style_conventions` + `ui_ux_engineering` |
| B | 旧项目加新功能 | `legacy_rapid_expansion` + `navigation_patterns` |
| C | 旧项目全面现代化 | `testing_legacy_strategies` + `tech_stack_migration` |
| D | 性能问题排查 | `deep_performance_tuning` + `crash_monitoring` |
| E | App 发布准备 | `devops_and_security` + `deep_performance_tuning` |
| F | 跨平台共享逻辑 | `kotlin_multiplatform` + `data_layer_mastery` |
| G | AI-assisted CI / Quality Gates | `coding_style_conventions` + `devops_and_security` |
| H | Performance-by-default | `deep_performance_tuning` + `project_bootstrapping` |
| I | Observability-first | `observability_first` + `crash_monitoring`（先定义 SLO/Owner/告警渠道） |
| J | Supply Chain Security | `supply_chain_security` + `devops_and_security` |
| K | Compose-first + Interop | `tech_stack_migration` + `ui_ux_engineering` |
| L | 多模块治理 | `project_bootstrapping` + `dependency_injection_mastery` |

> 所有场景都建议执行：`Required Inputs` → `Deliverables` → `*Gate` → `Quick Checklist`。

### 原则 3：明确引用章节

```
❌ 模糊指令：
   「帮我优化这段代码」

✅ 明确指令：
   「请参考 @ui_ux_engineering 中的【Accessibility】章节，
    检查这个按钮的 contentDescription 和触摸目标大小」
```

### 原则 4：强制 Checklist 验收

```
「任务完成后，请逐一对照 @coding_style_conventions 中的 Quick Checklist，
 确认每一项都符合规范」
```

### 原则 5：执行契约 (Required Inputs → Deliverables → *Gate)

从当前版本开始，技能默认采用统一执行契约：

1. 先填写 `Required Inputs`（范围、阈值、Owner、回退策略）
2. 明确 `Deliverables`（本次要产出的文件、配置、报告）
3. 执行对应 `* Gate`（例如 `Style Gate`、`Release Gate`、`Monitoring Gate`）
4. 最后再做 `Quick Checklist` 收尾

推荐在提示词里直接加上：

```
请先填写该技能的 Required Inputs，再输出 Deliverables，
执行对应 Gate 后，逐条完成 Quick Checklist。
```

---

## 各 AI 工具详细教学

---

### A. Antigravity (Gemini CLI / VS Code)

**契合度：⭐⭐⭐⭐⭐ (高)**

**注意**：以下流程请以该工具在你使用当日的官方文档为准；本文仅提供技能接入思路与示例。

#### 安装方式

```bash
# CLI 安装
npm install -g @anthropic-ai/antigravity

# VS Code Extension
# 在 Extensions 中搜索 "Antigravity" 并安装
```

#### 技能存放位置

```
~/.gemini/antigravity/skills/
├── skill_index/SKILL.md
├── coding_style_conventions/SKILL.md
├── project_bootstrapping/SKILL.md
└── ... (共 17 个)
```

#### 使用方式 1：自动识别

Antigravity 会自动扫描 `~/.gemini/antigravity/skills/` 目录下的技能。
在对话中直接提到技能名称即可：

```
请根据 coding_style_conventions 技能，检查这个文件的命名规范
```

#### 使用方式 2：使用 `@` 符号引用

```
@coding_style_conventions 请帮我检查目前打开的文件

# 组合多个技能
请同时参考：
@coding_style_conventions
@dependency_injection_mastery

帮我创建一个 UserRepository
```

#### 使用方式 3：引用技能与代码

```
请对照 @data_layer_mastery 的 Offline-First 架构，
查看目前打开的 UserRepository.kt 是否符合规范
```

#### 使用方式 4：场景导向

```
# 先咨询 skill_index
@skill_index 我要进行旧项目现代化，请推荐适合的技能组合

# AI 会推荐：
# - testing_legacy_strategies
# - tech_stack_migration
# - coding_style_conventions

# 然后引用推荐的技能
@testing_legacy_strategies @tech_stack_migration 
请按照这两份技能的指南，帮我重构 PaymentManager
```

#### 进阶：Planning Mode

Antigravity 支持 Planning Mode，适合复杂任务：

```
# 打开 Planning Mode (在 VS Code 中)
# 使用 task.md 追踪进度

请根据 @project_bootstrapping 创建一个完整的项目架构，
使用 Planning Mode 先规划再运行
```

#### 进阶：使用 Workflows

可以将常用流程存为 Workflow：

```markdown
<!-- .agent/workflows/android-review.md -->
---
description: Android Code Review 流程
---

1. 读取 @coding_style_conventions 技能
2. 检查命名规范
3. 检查 Compose 相关规范
4. 对照 Quick Checklist
5. 生成报告
```

使用：
```
/android-review
```

#### 最佳实践

1. **技能已内置**：这些技能放在 `~/.gemini/antigravity/skills/`，Antigravity 会自动识别
2. **使用场景路由**：先 `@skill_index` 获取建议
3. **组合使用**：同时引用 2-3 个相关技能
4. **Checklist 验收**：任务结束前要求使用 Quick Checklist

---

### B. Cursor

**契合度：⭐⭐⭐⭐⭐ (完美)**

官方文档：
- https://geminicli.com/docs/
- https://github.com/google-gemini/gemini-cli

Cursor 是目前最适合使用这些技能的工具之一。

官方下载与文档：
- 下载：https://cursor.com/downloads
- 文档：https://cursor.com/docs

#### 环境设置

1. **打开 Cursor Settings** (`Ctrl/Cmd + ,`)
2. **进入 Features > Rules**
3. **添加 Project Rules**（可选）

#### 方法 1：使用 `@` 符号直接引用

```
# 在 Chat 中输入
@coding_style_conventions

# 然后输入指令
请依照这份规范，检查目前打开的文件
```

#### 方法 2：引用多个技能

```
请同时参考以下规范：
@coding_style_conventions
@dependency_injection_mastery

帮我创建一个符合规范的 UserRepository，使用 Hilt 注入
```

#### 方法 3：同时引用技能与代码

```
请对照 @data_layer_mastery 的 Offline-First 架构，
查看 @src/main/kotlin/data/UserRepository.kt 是否符合规范，
并列出需要修改的地方
```

#### 方法 4：使用 Cursor Rules (自动套用)

在项目根目录创建 `.cursor/rules/android.mdc`：

```markdown
---
description: Android 开发规范
globs: ["**/*.kt", "**/*.kts"]
---

这是 Android 项目，请遵循以下规范：

1. 命名规则：参考 @coding_style_conventions
2. 架构设计：参考 @dependency_injection_mastery
3. UI 开发：参考 @ui_ux_engineering
```

#### Cursor Composer 模式

```
# 适合大规模重构
/composer

请根据 @project_bootstrapping 的架构，
帮我将这个 monolithic app 拆分成以下模块：
- :core:common
- :core:data
- :core:ui
- :feature:auth
- :feature:home
```

---

### C. Windsurf

**契合度：⭐⭐⭐⭐⭐ (完美)**

Windsurf 的 Cascade 功能非常适合多步骤任务。

官方下载与文档：
- 下载：https://windsurf.com/download/editor
- 入门：https://docs.windsurf.com/windsurf/getting-started.md

补充：若使用插件版，官方注记维护模式，优先使用 Windsurf Editor 或官方 JetBrains 插件。

#### 基本使用

```
# 打开 Cascade (Cmd/Ctrl + L)

@skill_index 我要进行旧项目现代化，请告诉我步骤

# Cascade 会自动规划多步骤任务
```

#### 引用技能进行任务

```
请根据 @testing_legacy_strategies：

1. 分析 @src/main/kotlin/legacy/PaymentManager.kt
2. 列出所有公开方法
3. 为每个方法生成 Characterization Test
4. 将测试文件放在对应的 test 目录
```

#### 使用 Windsurf Rules

在 `.windsurf/rules.md` 中：

```markdown
# Android Project Rules

当处理 Kotlin 文件时：
- 遵循 coding_style_conventions 中的命名规则
- Compose 函数使用 PascalCase
- ViewModel 使用 Hilt 注入

当创建新模块时：
- 参考 project_bootstrapping 的 Package Structure
- 使用 Convention Plugins
```

---

### D. Roo Code

**契合度：⭐⭐⭐⭐⭐ (完美)**

VS Code Extension，支持多种 AI Provider。

官方文档与安装：
- 文档：https://docs.roocode.com
- 安装：https://docs.roocode.com/getting-started/installing

VS Code 扩展 ID：`RooVeterinaryInc.roo-cline`（官方名称为 Roo Code）

#### 安装

1. VS Code Extensions 搜索 "Roo Code"
2. 安装并设置 API Key

#### 使用方式

```
# 打开 Roo Code Panel (Ctrl + Shift + P > Roo Code)

# 引用技能
请参考 @~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md

帮我重构这个文件
```

#### 使用 Custom Instructions

在 Roo Code 设置中加入：

```
当处理 Android/Kotlin 代码时，请参考以下技能文件：
- ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md
- ~/.gemini/antigravity/skills/ui_ux_engineering/SKILL.md
```

---

### E. Cline

**契合度：⭐⭐⭐⭐⭐ (完美)**

VS Code Extension，支持自主运行任务。

支持：VS Code、Cursor、JetBrains、VSCodium、Windsurf

官方文档与安装：
- 文档：https://docs.cline.bot
- 安装：https://docs.cline.bot/getting-started/installing-cline

安装后需登录：https://app.cline.bot

#### 安装

1. VS Code Extensions 搜索 "Cline"
2. 设置 API Key (支持 Claude, OpenAI, Gemini 等)

#### 使用方式

```
# 打开 Cline (Ctrl + Shift + P > Cline: Open)

请读取 ~/.gemini/antigravity/skills/project_bootstrapping/SKILL.md
然后根据内容，在当前目录创建一个新的 Android 模块骨架
```

#### 使用 .clinerules

在项目根目录创建 `.clinerules`：

```
# Android Development Rules

When working with Kotlin files:
1. Read and follow: ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md
2. For UI: ~/.gemini/antigravity/skills/ui_ux_engineering/SKILL.md
3. For DI: ~/.gemini/antigravity/skills/dependency_injection_mastery/SKILL.md

Always run Quick Checklist before completing a task.
```

---

### F. Aider (CLI)

**契合度：⭐⭐⭐⭐⭐ (高效)**

Terminal 爱好者的首选，功能强大。

官方文档：
- 安装：https://aider.chat/docs/install.html
- 使用：https://aider.chat/docs/usage.html

#### 安装

```bash
# 官方推荐（aider-install）
python -m pip install aider-install
aider-install

# 或使用 pipx
pipx install aider-chat
```

#### 设置环境变量

```bash
# ~/.bashrc 或 ~/.zshrc

# Claude (推荐)
export ANTHROPIC_API_KEY="your-key"

# OpenAI
export OPENAI_API_KEY="your-key"

# Gemini
export GEMINI_API_KEY="your-key"
```

#### 启动方式

```bash
# 使用 Claude Sonnet (推荐，平衡性能与成本)
aider --model claude-3-5-sonnet-20241022

# 使用 Claude Opus (最强，成本高)
aider --model claude-3-opus-20240229

# 使用 GPT-4 Turbo
aider --model gpt-4-turbo

# 使用 GPT-4o
aider --model gpt-4o

# 使用 Gemini Pro
aider --model gemini/gemini-1.5-pro-latest

# 使用 DeepSeek (成本低)
aider --model deepseek/deepseek-chat
```

#### 基本操作流程

```bash
# Step 1: 启动 aider
aider --model claude-3-5-sonnet-20241022

# Step 2: 加入技能文件到 Context (唯读)
/read ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md

# Step 3: 加入要修改的代码 (可写)
/add app/src/main/kotlin/data/UserRepository.kt

# Step 4: 下达指令
请依照 coding_style_conventions 的规范重构 UserRepository，
特别注意命名规则和 Compose 相关的命名
```

#### 重要指令大全

```bash
# 文件管理
/add <file>          # 加入可编辑的文件
/read <file>         # 加入唯读文件 (适合技能)
/drop <file>         # 移除文件
/ls                  # 列出目前加载的文件

# 模式切换
/architect           # 切换 Architect 模式 (先规划再运行)
/code                # 切换回 Code 模式

# 运行控制
/undo                # 撤销上一次修改
/diff                # 显示差异
/commit              # 提交变更
/clear               # 清除对话

# 设置
/model <name>        # 切换模型
/tokens              # 显示 Token 使用量
```

#### 进阶：场景 C 完整流程

```bash
# Step 1: 启动并加载技能
aider --model claude-3-5-sonnet-20241022 \
  --read ~/.gemini/antigravity/skills/testing_legacy_strategies/SKILL.md \
  --read ~/.gemini/antigravity/skills/tech_stack_migration/SKILL.md

# Step 2: 加入目标文件
/add app/src/main/kotlin/legacy/PaymentManager.kt
/add app/src/test/kotlin/legacy/PaymentManagerTest.kt

# Step 3: 创建 Characterization Tests
依照 testing_legacy_strategies 的指南，
为 PaymentManager 的所有公开方法创建 Characterization Tests

# Step 4: 运行测试确认
/run ./gradlew test --tests "PaymentManagerTest"

# Step 5: 进行迁移
测试全部通过了。
现在依照 tech_stack_migration 的 RxJava → Flow 对照表，
将 PaymentManager 中的 Observable 改为 Flow

# Step 6: 再次测试
/run ./gradlew test --tests "PaymentManagerTest"

# Step 7: 使用 Checklist 验收
请对照 coding_style_conventions 的 Quick Checklist，
确认修改后的代码符合所有规范
```

#### Aider 设置档

创建 `~/.aider.conf.yml`：

```yaml
# 默认模型
model: claude-3-5-sonnet-20241022

# 自动加载规范
read:
  - ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md

# Git 设置
auto-commits: true
dirty-commits: true

# 编辑器设置
editor: code --wait
```

#### 创建 Shell Alias

```bash
# ~/.bashrc 或 ~/.zshrc

# 快速启动 + 加载 Android 技能
alias aider-android='aider --model claude-3-5-sonnet-20241022 \
  --read ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md \
  --read ~/.gemini/antigravity/skills/dependency_injection_mastery/SKILL.md'

# 专门用于 Code Review
alias aider-review='aider --model claude-3-5-sonnet-20241022 \
  --read ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md \
  --read ~/.gemini/antigravity/skills/deep_performance_tuning/SKILL.md'

# 专门用于旧项目
alias aider-legacy='aider --model claude-3-5-sonnet-20241022 \
  --read ~/.gemini/antigravity/skills/legacy_rapid_expansion/SKILL.md \
  --read ~/.gemini/antigravity/skills/tech_stack_migration/SKILL.md'
```

---

### G. Claude Code (CLI)

**契合度：⭐⭐⭐⭐⭐ (完美)**

Anthropic 官方的 Agentic CLI 工具，支持 Skills 系统。

#### 安装

```bash
# 官方安装指南
# https://code.claude.com/docs/en/setup

# 依官方安装指南进行安装
```

注：Homebrew 套件名为 `fabric-ai`，如需 `fabric` 指令可自行设 alias。

注：官方文档标示 npm 安装方式已 deprecated，请以官方安装指南为主。

#### Skills 位置

```
~/.claude/skills/
├── coding_style_conventions/
│   └── SKILL.md
├── data_layer_mastery/
│   └── SKILL.md
└── ... (共 17 个 Android skills)
```

#### 使用方式 1：Slash Command（直接调用）

Skill 的 `name` 会自动成为 slash command：

```bash
# 启动
claude

# 使用 slash command 直接调用 skill
> /coding_style_conventions
> 请检查这段 Kotlin 代码的命名规范

# 多个 skill 组合
> /coding_style_conventions
> /dependency_injection_mastery
> 帮我创建一个 UserRepository
```

#### 使用方式 2：自动识别

Claude 会根据 skill 的 `description` 自动判断是否加载：

```bash
> 请检查这段 Kotlin 代码的命名规范
# Claude 自动识别并加载 coding_style_conventions skill
```

#### 使用方式 3：动态 Context（进阶）

使用 `! command` 语法注入动态数据：

```bash
# 在 SKILL.md 中可以使用
! gh pr diff        # 注入 GitHub PR 差异
! cat package.json  # 注入文件内容
```

#### 进阶：停用 Skills

```bash
# 启动时停用所有 skills
claude --disable-slash-commands
```

---

### H. GitHub Copilot

**契合度：⭐⭐⭐⭐⭐ (完美 - 2026 支持 Agent Skills)**

2026 年的 GitHub Copilot 支持 Agent Skills 和多层级 Instructions。

官方文档：
- 安装：https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-extension
- 入门：https://docs.github.com/en/copilot/get-started/quickstart

#### Skills 位置（2026 新功能）

Copilot 现在支持类似 Claude Code 的 Agent Skills：

```
.github/
├── copilot-instructions.md      # Workspace 全域指令
├── instructions/                # 文件特定指令
│   ├── kotlin.instructions.md   # Kotlin 文件适用
│   └── compose.instructions.md  # Compose 文件适用
└── skills/                      # Agent Skills (2026)
    ├── coding_style_conventions/
    │   └── SKILL.md
    └── ...
```

#### 方法 1：Workspace Instructions（自动套用）

创建 `.github/copilot-instructions.md`：

```markdown
# Android 项目规范

## Kotlin Coding Style
- Class/Interface: PascalCase
- Function/Variable: camelCase
- Constants: SCREAMING_SNAKE_CASE
- @Composable: PascalCase

## Architecture
- 使用 Clean Architecture
- ViewModel 使用 Hilt @HiltViewModel
- Repository 实作 SSOT 模式

## Compose
- Modifier 作为第一个可选参数
- 使用 collectAsStateWithLifecycle
```

#### 方法 2：File-Specific Instructions（2026）

创建 `.github/instructions/kotlin.instructions.md`：

```markdown
---
applyTo: "**/*.kt"
---
处理 Kotlin 文件时，请遵循 coding_style_conventions 技能的规范。
```

#### 方法 3：使用 @workspace 引用

```
@workspace 请根据 .github/skills/coding_style_conventions/SKILL.md，
检查目前文件的命名规范
```

#### 方法 4：Open Tab Context

1. **打开 SKILL.md 作为 Tab**
2. **使用 Copilot Chat**：
```
参考我打开的技能文档，检查这段代码
```

#### 方法 5：Copilot Edits

```
# 打开 Copilot Edits (Ctrl + Shift + I)
请根据 #file:SKILL.md 中的规范，
重构 #file:UserRepository.kt
```

---

### I. Amazon Q Developer

**契合度：⭐⭐⭐⭐ (良好)**

官方入口：
- AI Studio：https://aistudio.google.com/
- API Key：https://aistudio.google.com/apikey
- Gemini API Docs：https://ai.google.dev/gemini-api/docs

AWS 官方的 AI Coding Assistant。

官方入门：
- https://aws.amazon.com/q/developer/getting-started/

登录方式：Builder ID 或 IAM Identity Center（依官方入门页为准）

#### 使用方式

```
# 在 Chat 中使用 @file 引用
请参考 @file:.gemini/antigravity/skills/coding_style_conventions/SKILL.md
帮我检查这段代码
```

---

### J. JetBrains AI Assistant

**契合度：⭐⭐⭐⭐ (良好)**

Android Studio / IntelliJ IDEA 的原生 AI。

官方文档与插件：
- 使用说明：https://www.jetbrains.com/help/idea/ai-assistant.html
- 插件：https://plugins.jetbrains.com/plugin/22282-jetbrains-ai-assistant

#### 方法 1：通过 Chat

1. **打开 AI Assistant (Alt + Enter > AI)**
2. **在 Chat 中贴入技能内容**

```
请依照以下规范进行 Code Review：

[贴入 SKILL.md 内容]

现在请检查这段代码...
```

#### 方法 2：Custom Prompts

在 Settings > AI Assistant > Custom Prompts：

```
Name: Android Style Check
Prompt: 
依照以下 Kotlin 命名规范检查选中的代码：
- Class: PascalCase
- Function: camelCase
- Constants: SCREAMING_SNAKE_CASE
- Composable: PascalCase
列出所有违规项目。
```

---

### K. Claude Web

**契合度：⭐⭐⭐⭐ (良好)**

使用 Claude Projects 可大幅提升体验。

官方文档：
- Getting started：https://support.claude.com/en/articles/8114491-getting-started-with-claude
- Projects：https://support.claude.com/en/articles/9517075-what-are-projects

备注：Projects 为官方功能，免费帐号最多 5 个 Projects（依官方说明为准）。

#### 方法 1：Claude Projects (推荐)

1. **创建 Project** (claude.ai > Projects > New)
2. **命名为 "Android Development"**
3. **上传所有 SKILL.md 到 Project Knowledge**
4. **设置 Project Instructions**：

```
你是一位资深 Android 工程师。
在回答问题时，请优先参考 Project Knowledge 中的技能文档。
每次代码产出后，请对照相关技能的 Quick Checklist 验收。
```

5. **在对话中使用**：

```
请参考 coding_style_conventions 技能，
帮我审查以下 Kotlin 代码：

```kotlin
[粘贴代码]
```
```

#### 方法 2：一般对话

```
# System Prompt 区块
你是一位资深 Android 工程师。请依照以下规范进行开发：

---
[贴入 SKILL.md 完整内容]
---

# User Prompt
现在请帮我...
```

---

### L. ChatGPT

**契合度：⭐⭐⭐ (手动)**

官方文档：
- Projects：https://help.openai.com/en/articles/10169521-projects-in-chatgpt
- macOS App：https://help.openai.com/en/articles/9275200-downloading-the-chatgpt-macos-app
- Android App：https://help.openai.com/en/articles/8167604-how-to-find-and-install-the-chatgpt-android-app-on-the-google-play-store

#### 方法 1：Custom GPT (推荐)

1. **ChatGPT Plus > Explore GPTs > Create**
2. **配置**：
   - Name: Android Senior Engineer
   - Instructions: 贴入 `skill_index/SKILL.md` 内容
   - Knowledge: 上传所有 SKILL.md 文件

3. **使用**：
```
请参考 coding_style_conventions，检查以下代码...
```

注意：Android App 请确认发行者为 OpenAI。

#### 方法 2：一般对话

```
# 在对话开头设置 Context
请扮演资深 Android 工程师，依照以下规范：

[贴入 SKILL.md 内容]

---

现在请帮我...
```

---

### M. Google AI Studio / Gemini

**契合度：⭐⭐⭐⭐ (良好)**

#### 方法 1：System Instructions

1. **打开 AI Studio（网页版）**
2. **取得 API Key（需要登录）**
3. **设置 System Instructions**：

```
你是资深 Android 工程师。
请遵循以下开发规范：

[贴入技能内容]
```

#### 方法 2：上传文件

1. **点击 📎 图标**
2. **上传 SKILL.md 文件**
3. **下达指令**：

```
请依照上传的规范文档，检查以下代码...
```

---

### N. LLM CLI

**契合度：⭐⭐⭐⭐⭐ (高效)**

Simon Willison 开发的强大 CLI 工具。

#### 安装

```bash
pip install llm

# 或使用 pipx
pipx install llm

# 或使用 uv
uv tool install llm

# 或使用 Homebrew
brew install llm

# 安装 Claude plugin
llm install llm-claude-3

# 设置 API Key
llm keys set anthropic
```

#### 基本使用

```bash
# 单一技能
cat ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md | \
llm "根据这份规范，给我一个标准的 Kotlin ViewModel 范例"

# 多个技能组合
cat ~/.gemini/antigravity/skills/project_bootstrapping/SKILL.md \
    ~/.gemini/antigravity/skills/dependency_injection_mastery/SKILL.md | \
llm "根据这两份技能，帮我设计一个 Feature Module 的架构"
```

#### 搭配代码

```bash
# 读取规范 + 代码，进行 Review
cat ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md \
    ./app/src/main/kotlin/UserRepository.kt | \
llm "请依照第一份文档的规范，Review 第二份的代码"
```

#### 创建 Alias

```bash
# ~/.bashrc
alias android-check='function _ac() { 
  cat ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md "$1" | \
  llm "依照规范检查代码，列出问题"
}; _ac'

# 使用
android-check ./UserRepository.kt
```

---

### O. Codex CLI (OpenAI)

**契合度：⭐⭐⭐⭐⭐ (完美)**

OpenAI 官方的 Agentic Coding CLI，支持 Skills 系统。

官方文档：
- https://developers.openai.com/codex/cli
- https://github.com/openai/codex

#### 安装

```bash
# npm
npm install -g @openai/codex

# Homebrew
brew install --cask codex
```

#### Skills 位置

```
~/.codex/skills/
├── coding_style_conventions/
│   └── SKILL.md
├── data_layer_mastery/
│   └── SKILL.md
└── ... (共 17 个 Android skills)
```

#### 激活 Skills 功能

```bash
# 启动时激活 skills
codex --enable skills

# 或在 ~/.codex/config.toml 中设置
# [features]
# skills = true
```

#### 使用方式 1：$ 符号引用 Skill

```bash
# 启动
codex --enable skills

# 使用 $ 符号引用 skill
> $coding_style_conventions 请检查这段 Kotlin 代码

# 多个 skill
> $coding_style_conventions $dependency_injection_mastery 
> 帮我创建一个 UserRepository
```

#### 使用方式 2：/skills 指令

```bash
# 列出可用 skills
> /skills

# 查看特定 skill 详情
> /skills coding_style_conventions
```

#### 使用方式 3：自动识别

Codex 会根据 skill 的 `description` 自动判断：

```bash
> 请检查这段代码的命名规范
# Codex 自动识别并加载相关 skill
```

---

### P. Gemini CLI (Google)

**契合度：⭐⭐⭐⭐⭐ (完美)**

Google 官方的 Agentic CLI，支持 Agent Skills 系统。

#### 安装

```bash
# npx（免安装）
npx @google/gemini-cli

# npm
npm install -g @google/gemini-cli

# Homebrew
brew install gemini-cli

# MacPorts
sudo port install gemini-cli
```

#### Skills 位置

```
~/.gemini/skills/
├── coding_style_conventions/
│   └── SKILL.md
├── data_layer_mastery/
│   └── SKILL.md
└── ... (共 17 个 Android skills)
```

#### 激活 Agent Skills（首次需要）

```bash
# 启动 Gemini CLI
gemini

# 打开设置
> /settings

# 搜索 "Skills" → 打开 "Agent Skills" → 按 Esc 保存
```

#### 使用方式 1：/skills 管理指令

```bash
# 列出所有 skills
> /skills list

# 激活特定 skill
> /skills enable coding_style_conventions

# 停用特定 skill
> /skills disable coding_style_conventions

# 重新加载 skills（添加 skill 后使用）
> /skills reload
```

#### 使用方式 2：自动识别（需确认）

Gemini 会根据 skill 的 `description` 自动判断，但会询问确认：

```bash
> 请检查这段 Kotlin 代码的命名规范

# Gemini 提示：
# 「侦测到相关 skill: coding_style_conventions
#   是否要激活此 skill？(Y/n)」

> Y
# skill 加载后继续运行
```

#### 使用方式 3：明确引用

```bash
> 使用 coding_style_conventions skill 来检查代码
```

---

### Q. OpenInterpreter

**契合度：⭐⭐⭐⭐ (良好)**

可以自主运行任务的 AI。

官方文档：
- https://docs.openinterpreter.com/
- 安装：https://docs.openinterpreter.com/getting-started/setup

建议 Python 版本：3.10 / 3.11

#### 安装

```bash
pip install open-interpreter
```

#### 使用

```bash
# 启动
interpreter

# 加载技能
>>> 请读取并记住这个文件的规范：~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md

# 下达任务
>>> 现在请扫描 ./app/src/main/kotlin 下所有 .kt 文件，
    找出不符合命名规范的地方，生成报告
```

---

### R. Fabric

**契合度：⭐⭐⭐⭐ (良好)**

Daniel Miessler 开发的 AI Pattern 系统。

官方文档：
- https://github.com/danielmiessler/Fabric

#### 安装

```bash
# macOS/Linux
curl -fsSL https://raw.githubusercontent.com/danielmiessler/Fabric/main/scripts/install | bash

# Windows PowerShell
iwr https://raw.githubusercontent.com/danielmiessler/Fabric/main/scripts/install.ps1 -useb | iex

# Homebrew
brew install fabric-ai
```

#### 创建 Android Pattern

```bash
# 创建 pattern 目录
mkdir -p ~/.config/fabric/patterns/android_review

# 创建 system.md
cat > ~/.config/fabric/patterns/android_review/system.md << 'EOF'
# IDENTITY and PURPOSE

你是资深 Android 工程师，专门进行 Code Review。

# STEPS

1. 读取输入的 Kotlin 代码
2. 依照以下规范检查
3. 列出所有违规项目
4. 提供修正建议

# RULES

## Naming
- Class/Interface: PascalCase
- Function/Variable: camelCase  
- Constants: SCREAMING_SNAKE_CASE
- @Composable: PascalCase

## Compose
- Modifier 为第一个可选参数
- 使用 remember 管理状态

# OUTPUT

以 Markdown 格式输出报告
EOF
```

#### 使用

```bash
# Review 单一文件
cat ./UserRepository.kt | fabric --pattern android_review

# Review 多个文件
find ./app/src -name "*.kt" -exec cat {} \; | fabric --pattern android_review
```

---

### S. Ollama + Open WebUI

**契合度：⭐⭐⭐⭐ (本地运行)**

完全本地运行，适合敏感项目。

官方文档：
- Ollama Quickstart：https://docs.ollama.com/quickstart
- Open WebUI：https://docs.openwebui.com/

#### 安装 Ollama

```bash
# macOS/Linux
curl -fsSL https://ollama.com/install.sh | sh

# 下载模型
ollama run gemma3
```

#### 安装 Open WebUI

```bash
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  ghcr.io/open-webui/open-webui:main
```

#### 使用

1. **打开 http://localhost:3000**
2. **上传 SKILL.md 到 Documents**
3. **在对话中使用**：

```
请参考已上传的 coding_style_conventions 文档，
检查以下代码...
```

---

### T. OpenCode CLI (opencode.ai)

**契合度：⭐⭐⭐⭐⭐ (完美 - 强大的 Agentic CLI)**

OpenCode 是 opencode.ai 的官方 CLI 工具，提供 TUI 接口与本地模型支持。

官方文档：
- https://opencode.ai/
- https://opencode.ai/docs
- https://github.com/anomalyco/opencode

#### 安装

```bash
# macOS / Linux
curl -fsSL https://opencode.ai/install | bash
```

#### 基本操作

```bash
# 启动并初始化项目 (创建 AGENTS.md)
opencode
> /init
```

#### 技能使用

请依官方文档设置技能路径与使用方式，确保与 OpenCode 版本一致。

#### Plan Mode (规划模式)

OpenCode 的强项在于先规划再运行：

1. 按 `Tab` 切换到 **Plan Mode**
2. 输入指令：
```
请根据 @project_bootstrapping 的架构，
规划一个 MVVM + Clean Architecture 的模块结构
```
3. 确认计划后，切换回 **Build Mode** 让 AI 运行

#### 最佳实践

1. **利用 AGENTS.md**：可以在项目内定义专属 Agent，预加载常用 Skills。
2. **本地模型集成**：配合 @devops_and_security 中的隐私规范，可完全使用本地模型以避免敏感数据外泄。

---

## 场景导向完整范例

### 场景 A：从零创建新项目 (完整流程)

#### 使用 Cursor

```
Step 1: 规划
@skill_index 我要创建一个新的电商 App，
需要 Auth, Product, Cart, Checkout 四个功能，
请告诉我应该使用哪些技能和步骤

Step 2: 项目骨架
@project_bootstrapping 请创建完整的项目结构，包含：
- build-logic/ 目录
- Convention Plugins (android-library, compose-feature)
- Version Catalog
- 模块结构

Step 3: 规范设置
@coding_style_conventions 请设置：
- Detekt 配置
- Ktlint 配置
- .editorconfig

Step 4: Design System
@ui_ux_engineering 请创建基础 Design System：
- Color Scheme (Light/Dark)
- Typography
- Spacing Tokens
- 基础 Components

Step 5: 验收
请对照所有使用到的技能的 Quick Checklist，
确认项目设置完整
```

#### 使用 Aider

```bash
# 启动
aider --model claude-3-5-sonnet-20241022 \
  --read ~/.gemini/antigravity/skills/skill_index/SKILL.md \
  --read ~/.gemini/antigravity/skills/project_bootstrapping/SKILL.md \
  --read ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md \
  --read ~/.gemini/antigravity/skills/ui_ux_engineering/SKILL.md

# 运行
我要创建一个电商 App 项目，请依照加载的技能：
1. 先依照 project_bootstrapping 创建项目骨架
2. 再依照 coding_style_conventions 设置 linter
3. 最后依照 ui_ux_engineering 创建 Design System

请创建所有必要的文件
```

---

### 场景 C：旧项目现代化 (完整流程)

#### 使用 Aider

```bash
# Step 1: 启动
aider --model claude-3-5-sonnet-20241022 \
  --read ~/.gemini/antigravity/skills/testing_legacy_strategies/SKILL.md \
  --read ~/.gemini/antigravity/skills/tech_stack_migration/SKILL.md \
  --read ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md

# Step 2: 分析目标
/add app/src/main/kotlin/legacy/PaymentManager.kt

请分析这个类别：
1. 列出所有公开方法
2. 识别使用的 RxJava 操作
3. 找出潜在的问题点

# Step 3: 创建测试
/add app/src/test/kotlin/legacy/PaymentManagerTest.kt

依照 testing_legacy_strategies 的 Characterization Tests 指南，
为 PaymentManager 的每个公开方法创建测试

# Step 4: 运行测试
/run ./gradlew test --tests "*PaymentManagerTest*"

# Step 5: 迁移
测试通过了。现在依照 tech_stack_migration：
1. 将 Observable 改为 Flow
2. 将 subscribe 改为 collect
3. 处理错误使用 catch

# Step 6: 再次测试
/run ./gradlew test --tests "*PaymentManagerTest*"

# Step 7: 代码风格
依照 coding_style_conventions 的 Quick Checklist 检查最终代码
```

---

### 场景 G：AI-assisted CI / Quality Gates (完整流程)

#### 使用 Aider

```bash
# Step 1: 启动并加载技能
aider --model claude-3-5-sonnet-20241022 \
  --read ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md \
  --read ~/.gemini/antigravity/skills/devops_and_security/SKILL.md \
  --read ~/.gemini/antigravity/skills/testing_legacy_strategies/SKILL.md

# Step 2: 创建 CI Gate
请创建 CI Gate：lint、detekt、unit test、assemble

# Step 3: 验收
请对照三份技能的 Quick Checklist 验收
```

---

### 场景 H：Performance-by-default (完整流程)

#### 使用 Cursor

```
Step 1: 设置性能基准
@deep_performance_tuning 请创建 Baseline Profile 与 Macrobenchmark

Step 2: 新项目默认
@project_bootstrapping 请将性能量测流程纳入项目骨架

Step 3: CI Gate
@devops_and_security 将性能门槛纳入 CI Gate
```

---

### 场景 I：Observability-first (完整流程)

#### 使用 Cursor

```
Step 1: 指标与事件
@observability_first 请定义关键流程与事件字段

Step 2: Crash/ANR
@crash_monitoring 创建 Crashlytics 与 ANR 告警

Step 3: 性能指标
@deep_performance_tuning 加入性能量测与回归规则
```

---

### 场景 J：Supply Chain Security (完整流程)

#### 使用 Aider

```bash
# Step 1: 依赖治理
aider --model claude-3-5-sonnet-20241022 \
  --read ~/.gemini/antigravity/skills/supply_chain_security/SKILL.md \
  --read ~/.gemini/antigravity/skills/devops_and_security/SKILL.md

# Step 2: SCA + Gate
请创建依赖扫描与风险门槛，纳入 CI
```

---

### 场景 K：Compose-first + Legacy Interop (完整流程)

#### 使用 Cursor

```
Step 1: 互通策略
@tech_stack_migration 请设计 Compose/View 共存策略

Step 2: UI 规范
@ui_ux_engineering 创建 Design System 与 a11y 验收

Step 3: 旧项目桥接
@legacy_rapid_expansion 设计 Bridge 与 Feature Toggle
```

---

### 场景 L：多模块扩展与导航治理 (完整流程)

#### 使用 Cursor

```
Step 1: 模块骨架
@project_bootstrapping 请规划模块与 Convention Plugins

Step 2: DI 边界
@dependency_injection_mastery 创建 API/impl 分离

Step 3: 导航治理
@navigation_patterns 设计跨模块导航接口
```

---

## 进阶集成技巧

### 1. Git Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

# 取得 staged 的 Kotlin 文件
STAGED=$(git diff --cached --name-only --diff-filter=ACM | grep "\.kt$")

if [ -n "$STAGED" ]; then
  echo "Running AI Code Review..."
  
  for file in $STAGED; do
    cat ~/.gemini/antigravity/skills/coding_style_conventions/SKILL.md "$file" | \
    llm "依照规范快速检查，只回报严重问题" || exit 1
  done
fi
```

### 2. VS Code Task

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Android: Code Review",
      "type": "shell",
      "command": "cat ${workspaceFolder}/.gemini/antigravity/skills/coding_style_conventions/SKILL.md ${file} | llm '依照规范 Review 代码'",
      "problemMatcher": []
    },
    {
      "label": "Android: Generate Tests",
      "type": "shell", 
      "command": "aider --model claude-3-5-sonnet-20241022 --read ${workspaceFolder}/.gemini/antigravity/skills/testing_legacy_strategies/SKILL.md ${file} --message '为这个类别生成 Characterization Tests'",
      "problemMatcher": []
    }
  ]
}
```

### 3. PowerShell 函数 (Windows)

```powershell
# $PROFILE

function Invoke-AndroidReview {
    param([string]$FilePath)
    
    $skill = Get-Content "$env:USERPROFILE\.gemini\antigravity\skills\coding_style_conventions\SKILL.md" -Raw
    $code = Get-Content $FilePath -Raw
    
    "$skill`n---`n$code" | llm "依照规范 Review 代码"
}

Set-Alias -Name android-review -Value Invoke-AndroidReview
```

---

## 常见问题

### Q1: Token 不够怎么办？
**A:** 
- 只加载当下需要的 2-3 个技能
- 使用 `skill_index` 的场景路由选择组合
- 对于 CLI 工具，使用 `-read` 而非 `/add`

### Q2: AI 没有遵循规范怎么办？
**A:** 
- 在指令最后加上：「请逐一对照 Quick Checklist」
- 分步骤运行，每步骤验证
- 明确指定要参考的章节

### Q3: 如何更新技能内容？
**A:** 
- 直接编辑对应的 SKILL.md
- 所有工具会自动读取最新版本
- 建议使用 Git 追踪变更

### Q4: 哪个工具最推荐？
**A:** 
- **Gemini 用户**: Antigravity / Gemini CLI (技能已内置)
- **Claude 用户**: Claude Code CLI (使用 /skill_name)
- **OpenAI 用户**: Codex CLI (使用 $skill_name)
- **最佳体验**: Cursor / Windsurf
- **CLI 爱好者**: Aider / OpenCode CLI
- **完全本地**: Ollama + Open WebUI
- **团队共享**: Claude Projects

---

## 快速参考卡

```
┌───────────────────────────────────────────────────────────────────────┐
│                    Android Skills 快速使用 (2026)                      │
├───────────────────────────────────────────────────────────────────────┤
│                        🔥 CLI 工具 (Native Skills)                     │
├───────────────────────────────────────────────────────────────────────┤
│ CLAUDE CODE    │ /skill_name (Slash Command)                         │
│ CODEX CLI      │ $skill_name (需 --enable skills)                    │
│ GEMINI CLI     │ /skills list, /skills enable (需打开 Agent Skills)  │
├───────────────────────────────────────────────────────────────────────┤
│                        🖥️ Agentic IDE                                  │
├───────────────────────────────────────────────────────────────────────┤
│ ANTIGRAVITY    │ @skill_name (技能已内置，自动识别)                    │
│ CURSOR         │ @skill_name                                         │
│ WINDSURF       │ @skill_name 在 Cascade 中                            │
│ ROO CODE       │ @<skills-root>/xxx/SKILL.md                          │
│ CLINE          │ 请读取 <skills-root>/xxx/SKILL.md                     │
├───────────────────────────────────────────────────────────────────────┤
│                        🔧 其他 CLI 工具                                │
├───────────────────────────────────────────────────────────────────────┤
│ AIDER          │ /read <skills-root>/xxx/SKILL.md                     │
│ OPENCODE CLI   │ @skill_name (TUI)                                   │
│ LLM CLI        │ cat SKILL.md | llm "指令"                           │
│ FABRIC         │ cat code.kt | fabric --pattern android_review       │
├───────────────────────────────────────────────────────────────────────┤
│                        🌐 Web / IDE                                    │
├───────────────────────────────────────────────────────────────────────┤
│ COPILOT        │ Open Tab + @workspace                               │
│ CLAUDE WEB     │ Projects > Knowledge 上传                            │
│ CHATGPT        │ Custom GPT > Knowledge 上传                          │
│ OLLAMA         │ Open WebUI > Documents 上传                          │
└───────────────────────────────────────────────────────────────────────┘
```

注：`<skills-root>` 请替换为你实际放置目录（如 `~/.gemini/antigravity/skills`、`~/.gemini/skills`、`~/.claude/skills`、`~/.codex/skills`）。
各工具的技能加载指令以官方文档为准，若版本差异请依官方更新。

### Skills 安装位置对照

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                    2026 CLI Skills 标准位置                                    │
├───────────────────────────────────────────────────────────────────────────────┤
│ OpenCode CLI  │ Windows: %USERPROFILE%\AppData\Roaming\opencode\agents          │
│               │ macOS/Linux: ~/.opencode/agents                                 │
│ Claude Code   │ Windows: %USERPROFILE%\.claude\skills\<skill-name>\SKILL.md      │
│               │ macOS/Linux: ~/.claude/skills/<skill-name>/SKILL.md             │
│ Codex CLI     │ Windows: %USERPROFILE%\.codex\skills\<skill-name>\SKILL.md       │
│               │ macOS/Linux: ~/.codex/skills/<skill-name>/SKILL.md              │
│ Gemini CLI    │ Windows: %USERPROFILE%\.gemini\skills\<skill-name>\SKILL.md      │
│               │ macOS/Linux: ~/.gemini/skills/<skill-name>/SKILL.md             │
│ Antigravity   │ Windows: %USERPROFILE%\.gemini\antigravity\skills                │
│               │ macOS/Linux: ~/.gemini/antigravity/skills                        │
└───────────────────────────────────────────────────────────────────────────────┘
```

简短说明：若你的 CLI 已内置 skills 机制，请把对应的 skill 目录放到以上标准位置；若版本不同，请以官方文档为准并保留此表作为默认值。
