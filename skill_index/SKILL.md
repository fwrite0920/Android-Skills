---
name: Android Skill Index
description: 资深 Android 工程师技能导航中心，根据场景推荐适合的技能组合
---

# Android Skill Index (技能索引导航)

此技能作为所有 Android 技能的入口点与导航器。

## Instructions
- 先描述你的目标与现况（新项目、旧项目、性能问题等）
- 使用 Scenario Router 选择技能组合
- 只加载当下需要的 2-3 个技能
- 完成后回到这里确认是否漏掉关键能力

## When to Use
- 不确定该加载哪些技能时
- 任务涵盖多个领域，需要组合技能时
- 想依情境快速创建操作流程时

## Example Prompts
- "我在做旧项目现代化，请推荐需要的技能组合"
- "性能很差，请告诉我应该先用哪些技能排查"
- "我要创建新项目，请给我最短的技能路径"

## Workflow
1. 先在 Scenario Router 找到最接近的情境
2. 依顺序加载对应技能并运行
3. 任务完成后回到 Skill Dependency Graph 检查漏项

## Minimal Template
```
目标: 
现况: 
情境: 
建议技能: 
验收: 依 Quick Checklist
```

## Quick Reference (快速索引)

| Skill | 一句话描述 | 适用场景 |
|-------|-----------|---------|
| `coding_style_conventions` | Kotlin 代码规范与 Linter 配置 | 所有项目 |
| `project_bootstrapping` | 快速创建项目骨架与 Gradle Plugins | 新项目 |
| `ui_ux_engineering` | Design System 与复杂 UI 实作 | UI 开发 |
| `dependency_injection_mastery` | Hilt 进阶用法与模块化 | 架构设计 |
| `data_layer_mastery` | Room, Retrofit, Offline-First | 数据层 |
| `navigation_patterns` | Deep Links 与跨模块导航 | 导航设计 |
| `legacy_rapid_expansion` | 在旧架构中快速创建新功能 | 旧项目扩充 |
| `tech_stack_migration` | Rx→Flow, View→Compose 迁移 | 技术升级 |
| `testing_legacy_strategies` | 为无测试代码创建安全网 | 旧项目重构 |
| `deep_performance_tuning` | Systrace, Memory, R8 深度优化 | 性能问题 |
| `devops_and_security` | CI/CD, Fastlane, 资安加固 | 发布准备 |
| `crash_monitoring` | Crashlytics, ANR 分析 | 在线监控 |
| `kotlin_multiplatform` | KMP 跨平台架构 | 跨平台准备 |
| `observability_first` | 可观测性优先与指标闭环 | 监控体系 |
| `supply_chain_security` | 依赖治理与供应链安全 | 发布安全 |
| `sdk_development` | SDK/Library 开发全生命周期 | SDK 开发 |

---

## Scenario Router (场景路由)

根据您的情境，选择对应的技能组合：

### 🚀 场景 A：从零创建新项目
```
1. project_bootstrapping    → 创建骨架
2. coding_style_conventions → 设置规范
3. dependency_injection_mastery → DI 架构
4. ui_ux_engineering        → Design System
5. data_layer_mastery       → 数据层
6. navigation_patterns      → 导航设计
```

### 🔧 场景 B：旧项目加入新功能模块
```
1. legacy_rapid_expansion   → Islanding 策略
2. ui_ux_engineering        → Hybrid Theming
3. navigation_patterns      → 导航桥接
4. testing_legacy_strategies → 边界测试
```

### 🔄 场景 C：旧项目全面现代化
```
1. testing_legacy_strategies → Characterization Tests
2. coding_style_conventions  → Baseline 规范
3. tech_stack_migration      → 技术迁移
4. dependency_injection_mastery → 模块拆分
5. deep_performance_tuning   → 性能验证
```

### ⚡ 场景 D：性能问题排查
```
1. crash_monitoring         → 监控数据收集
2. deep_performance_tuning  → 深度分析
3. ui_ux_engineering        → UI 优化
4. data_layer_mastery       → 数据层优化
```

### 📦 场景 E：App 发布准备
```
1. devops_and_security      → 品质检查
2. deep_performance_tuning  → 性能验证
3. devops_and_security      → 安全加固
4. crash_monitoring         → 监控准备
5. devops_and_security      → 自动发布
```

### 🌐 场景 F：跨平台共享逻辑
```
1. kotlin_multiplatform     → 架构评估
2. data_layer_mastery       → 数据层抽取
3. dependency_injection_mastery → DI 调整
4. testing_legacy_strategies → 共享测试
```

### 🧭 场景 G：AI-assisted CI / Quality Gates
```
1. coding_style_conventions → 规范与检核
2. devops_and_security      → CI Gate 与自动化
3. testing_legacy_strategies → 测试安全网
```

### 📈 场景 H：Performance-by-default
```
1. deep_performance_tuning  → 基准量测
2. project_bootstrapping    → 默认性能配置
3. devops_and_security      → CI 量测与门槛
```

### 🔍 场景 I：Observability-first
```
1. observability_first      → 指标与回馈闭环
2. crash_monitoring         → Crash/ANR/Logs
3. deep_performance_tuning  → 性能指标
```

### 🧱 场景 J：Supply Chain Security
```
1. supply_chain_security    → 依赖与密钥治理
2. devops_and_security      → CI 与发版安全
3. coding_style_conventions → 规范与审查标准
```

### 🧩 场景 K：Compose-first + Legacy Interop
```
1. tech_stack_migration     → Compose/View 共存
2. ui_ux_engineering        → Design System 与 a11y
3. legacy_rapid_expansion   → 旧项目桥接
```

### 📚 场景 L：SDK/Library 开发
```
1. sdk_development          → API 设计与发布
2. coding_style_conventions → 代码规范
3. supply_chain_security    → 依赖与密钥管理
4. devops_and_security      → CI/CD 与发布流程
```

### 🧭 场景 M：多模块扩展与导航治理
```
1. project_bootstrapping    → 模块与插件
2. dependency_injection_mastery → 模块边界
3. navigation_patterns      → 跨模块导航
```

---

## Skill Dependency Graph

```
┌─────────────────────────────────────────────────────────────┐
│                    coding_style_conventions                  │
│                     (所有技能的基础)                          │
└─────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│project_bootstrap│  │legacy_rapid_    │  │deep_performance │
│      ing        │  │   expansion     │  │    _tuning      │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                    │
    ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
    ▼         ▼          ▼         ▼          ▼         ▼
┌───────┐ ┌───────┐  ┌───────┐ ┌───────┐  ┌───────┐ ┌───────┐
│ui_ux_ │ │depend-│  │tech_  │ │testing│  │devops_│ │crash_ │
│engine-│ │ency_  │  │stack_ │ │legacy_│  │and_   │ │monito-│
│ering  │ │inject-│  │migrat-│ │strate-│  │securi-│ │ring   │
│       │ │ion    │  │ion    │ │gies   │  │ty     │ │       │
└───────┘ └───────┘  └───────┘ └───────┘  └───────┘ └───────┘
    │         │
    └────┬────┘
         ▼
┌─────────────────┐  ┌─────────────────┐
│data_layer_      │  │kotlin_          │
│   mastery       │  │ multiplatform   │
└────────┬────────┘  └─────────────────┘
         │
         ▼
┌─────────────────┐
│navigation_      │
│   patterns      │
└─────────────────┘
                          │                 │                 │
                          ▼                 ▼                 ▼
                ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
                │observability_   │ │supply_chain_    │ │sdk_development  │
                │   first         │ │  security       │ │                 │
                └─────────────────┘ └─────────────────┘ └─────────────────┘
```

## Notes
- `observability_first`, `supply_chain_security`, `sdk_development` 为 2026 追加技能

---

## How to Use (使用方式)

1. **确认您的场景**：从上方的 Scenario Router 选择最接近的情境
2. **依序运行技能**：按照建议顺序阅读并应用各技能
3. **参考 Checklist**：每个技能都附有 Quick Checklist 供 Code Review 使用

---

## AI Tool Compatibility & Usage Guide (AI 工具兼容性指南)

这些技能文档是为了让 **任何支持上下文 (Context) 的 AI 工具** 都能使用而设计的。

### 1. 通用性 (Universality)
所有的 SKILL.md 均采用 **标准 Markdown** 格式编写，这是目前所有 LLM (ChatGPT, Claude, Gemini) 最能精确理解的格式。
- **结构化**：使用标题与清单，AI 容易解析逻辑。
- **代码块**：包含实际可运行的 Kotlin/Groovy 范例。
- **Checklist**：AI 可用来进行 Self-Correction (自我修正)。

### 2. 各类工具使用方式

#### A. Agentic IDEs (Cursor, Windsurf, Roo Code)
**契合度：⭐⭐⭐⭐⭐ (完美)**
这些工具可以直接索引并读取本地文件，体验最好。
- **用法**：
  1. 在 Chat 中输入 `@` (或是引用符号) 选择对应的 `SKILL.md`。
     - *例：`@project_bootstrapping`*
  2. 输入指令：
     - *「请根据 @project_bootstrapping 的架构，帮我创建 Auth 模块。」*
- **技巧**：同时引用你的代码与 SKILL，让 AI 进行「对照实作」。

#### B. CLI Tools (Aider, OpenInterpreter)
**契合度：⭐⭐⭐⭐⭐ (高效)**
适合且习惯使用 Terminal 的开发者。
- **用法 (Aider)**：
  1. `/add skills/data_layer_mastery/SKILL.md` (将技能加入 Context Window)
  2. `/code "依照刚才加入的技能标准，帮我重构 UserRepository，加入 Offline-first 支持"`
- **用法 (一般 CLI)**：
  1. 利用 Pipe 将内容送给 LLM：
     - `cat skills/coding_style/SKILL.md | llm prompt "Review this file"`

#### C. GitHub Copilot / Standard IDE Assistants
**契合度：⭐⭐⭐⭐ (良好)**
Copilot 通常会读取「目前打开的文件」作为上下文。
- **用法**：
  1. 在 IDE 中**打开** (Open Tab) 你想参考的 `SKILL.md`。
  2. 切换回你的代码文件进行编辑。
  3. 在 Chat 或 Inline Chat 中询问：「参考打开的技能文档，这段代码符合规范吗？」

#### D. Web Chat (ChatGPT, Claude.ai)
**契合度：⭐⭐⭐ (手动)**
- **用法**：
  1. 复制 `SKILL.md` 的内容。
  2. 粘贴并加上 Prompt：「你是一位资深工程师，请依照以下指南...」。

### 3. 如何达成「完美契合」 (Best Practices)

为了在有限的 Token 与 Attention 下达到最佳效果：

1. **Context Management (不浪费 Token)**：
   - 🚫 **Don't**: 一次把 14 个文件全部丢给 AI。
   - ✅ **Do**: 根据 `Scenario Router`，只加载当下任务需要的 2-3 个技能。
   
2. **Focusing (聚焦指令)**：
   - 指令要明确引用章节。
   - *例：「请参考 @ui_ux_engineering 中的 **Accessibility** 章节，检查这个按钮。」*

3. **Enforce Checklists (强制验收)**：
   - 在任务结束前，要求 AI：
   - *「请逐一检查 @SKILL.md 中的 Quick Checklist，确认我们是否遗漏了什么？」*

