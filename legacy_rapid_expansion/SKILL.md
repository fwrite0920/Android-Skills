---
name: Legacy Rapid Expansion
description: 在旧架构中快速创建新功能的 Islanding 策略
---

# Legacy Rapid Expansion (旧项目快速扩充)

## Instructions
- 仅在旧项目加入新功能时使用
- 先填写 Required Inputs（隔离边界、桥接契约、回退方案）
- 依照下方章节顺序套用
- 一次只创建一个隔离区与桥接点
- 完成后对照 Quick Checklist

## When to Use
- Scenario B：旧项目加新功能

## Example Prompts
- "请依照 Islanding Architecture 创建 modern/ 与 bridge/"
- "用 Hybrid Theming 章节让新 Compose UI 沿用旧 Theme"
- "请用 Feature Toggle 章节设计新功能开关"

## Workflow
1. 先确认 Required Inputs（隔离范围、bridge API、toggle 策略）
2. 创建 Islanding Architecture 与 Bridge Pattern
3. 处理 Hybrid Theming 与 Wrapper Activities
4. 配置 Feature Toggle 与回滚流程
5. 执行 Legacy Gate 并用 Quick Checklist 验收

## Practical Notes (2026)
- 新功能必须独立于 legacy 内部实作
- Bridge API 需稳定，避免频繁变更造成扩散
- Feature Toggle 作为上线与回退的唯一入口
- 新旧系统边界必须有 owner，避免“共享无人区”
- 迁移期间每个 bridge 调用都应有埋点便于回滚判断

## Minimal Template
```
目标: 
隔离范围: 
旧系统依赖:
回滚触发条件:
Bridge 契约: 
Toggle 策略: 
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `隔离范围`（新功能边界与 legacy 触点）
- `Bridge 契约`（输入/输出/错误码）
- `Feature Toggle 策略`（开关位置、默认值、回滚）
- `回滚触发条件`（指标阈值与负责人）
- `埋点字段`（新旧路径识别）

## Deliverables (完成后交付物)

- `modern/` 与 `bridge/` 目录落地
- `Bridge API` 文档与使用样例
- `Hybrid theme` 兼容策略
- `Toggle` 配置与灰度/回滚脚本
- `Legacy Gate` 验收记录

## Legacy Gate (验收门槛)

```bash
./gradlew test
./gradlew connectedDebugAndroidTest
```

> 需要在 PR 说明中附新旧路径对照与回滚步骤。

---

## Islanding Architecture (孤岛策略)

在旧架构中切出一块「净土」，新功能完全使用现代架构开发。

### 目录结构

```
app/
├── legacy/                # 旧代码 (不动)
│   ├── activities/
│   └── fragments/
├── modern/                # 新代码 (净土)
│   ├── core/
│   │   ├── data/
│   │   ├── domain/
│   │   └── ui/
│   └── feature/
│       └── newfeature/
└── bridge/                # 桥接层
    ├── LegacyNavigator.kt
    └── ModernEntryPoint.kt
```

### Bridge Pattern

```kotlin
// bridge/ModernEntryPoint.kt
object ModernEntryPoint {
    
    fun startNewFeatureActivity(context: Context, params: Bundle) {
        val intent = Intent(context, NewFeatureActivity::class.java).apply {
            putExtras(params)
        }
        context.startActivity(intent)
    }

    fun startCheckoutCompose(context: Context, orderId: String) {
        startNewFeatureActivity(
            context = context,
            params = bundleOf("order_id" to orderId, "entry" to "checkout")
        )
    }
    
    @Composable
    fun NewFeatureScreen(params: Map<String, Any>) {
        // 现代 Compose UI
    }
}

// 旧代码调用
class LegacyActivity : AppCompatActivity() {
    fun onButtonClick() {
        ModernEntryPoint.startNewFeatureActivity(this, bundleOf("id" to productId))
    }
}
```

---

## Hybrid Theming

让 Compose UI 沿用旧有的 XML Theme。

### MDC-Android Compose Theme Adapter

```kotlin
// build.gradle.kts
dependencies {
    implementation("com.google.android.material:compose-theme-adapter:<project-verified-version>")
}

// 使用
@Composable
fun NewFeatureScreen() {
    MdcTheme {  // 自动桥接 XML Theme
        Surface {
            // Compose UI 会使用 XML 定义的 colors/typography
        }
    }
}
```

### 渐进式迁移

```kotlin
// 1. 初期：完全沿用 XML Theme
MdcTheme { content() }

// 2. 中期：覆写部分 Token
MdcTheme(
    setTextColors = true,
    setDefaultFontFamily = true
) { content() }

// 3. 后期：完全使用 Compose Theme
AppTheme { content() }
```

---

## Wrapper Activities

快速将 Compose Screen 包装供旧有 `startActivity` 调用。

### 通用 Wrapper

```kotlin
@AndroidEntryPoint
class ComposeWrapperActivity : ComponentActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val screenType = intent.getStringExtra(EXTRA_SCREEN_TYPE)
        val params = intent.extras ?: Bundle.EMPTY
        
        setContent {
            AppTheme {
                when (screenType) {
                    "new_feature" -> NewFeatureScreen(params.toMap())
                    "settings" -> SettingsScreen()
                    else -> ErrorScreen()
                }
            }
        }
    }
    
    companion object {
        private const val EXTRA_SCREEN_TYPE = "screen_type"
        
        fun newIntent(context: Context, screenType: String, params: Bundle = Bundle.EMPTY): Intent {
            return Intent(context, ComposeWrapperActivity::class.java).apply {
                putExtra(EXTRA_SCREEN_TYPE, screenType)
                putExtras(params)
            }
        }
    }
}
```

---

## Feature Toggle

安全地在 Production 环境开关新功能。

```kotlin
interface FeatureFlags {
    val useNewCheckout: Boolean
    val enableComposeProfile: Boolean
}

class RemoteFeatureFlags @Inject constructor(
    private val remoteConfig: FirebaseRemoteConfig
) : FeatureFlags {
    override val useNewCheckout: Boolean
        get() = remoteConfig.getBoolean("use_new_checkout")
}

// 使用
class CheckoutNavigator @Inject constructor(
    private val context: Context,
    private val featureFlags: FeatureFlags
) {
    fun navigateToCheckout(orderId: String) {
        if (featureFlags.useNewCheckout) {
            ModernEntryPoint.startCheckoutCompose(context, orderId)
        } else {
            context.startActivity(Intent(context, LegacyCheckoutActivity::class.java))
        }
    }
}
```

---

## Quick Checklist

- [ ] Required Inputs 已填写并冻结（边界/契约/回滚）
- [ ] 新功能放在 `modern/` 目录
- [ ] 桥接层清晰定义 (bridge/)
- [ ] Hybrid Theming 确保视觉一致
- [ ] Feature Toggle 控制上线
- [ ] 避免新代码依赖旧代码的内部实作
- [ ] Legacy Gate 已执行并记录结果
