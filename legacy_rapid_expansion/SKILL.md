---
name: Legacy Rapid Expansion
description: 在旧架构中快速创建新功能的 Islanding 策略
---

# Legacy Rapid Expansion (旧项目快速扩充)

## Instructions
- 仅在旧项目加入新功能时使用
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
1. 先创建 Islanding Architecture 与 Bridge Pattern
2. 再处理 Hybrid Theming 与 Wrapper Activities
3. 最后用 Feature Toggle 与 Quick Checklist 验收

## Practical Notes (2026)
- 新功能必须独立于 legacy 内部实作
- Bridge API 需稳定，避免频繁变更造成扩散
- Feature Toggle 作为上线与回退的唯一入口

## Minimal Template
```
目标: 
隔离范围: 
Bridge 契约: 
Toggle 策略: 
验收: Quick Checklist
```

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
    implementation("com.google.android.material:compose-theme-adapter:1.2.1")
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
    private val featureFlags: FeatureFlags
) {
    fun navigateToCheckout() {
        if (featureFlags.useNewCheckout) {
            ModernEntryPoint.startCheckoutCompose()
        } else {
            startActivity(LegacyCheckoutActivity::class)
        }
    }
}
```

---

## Quick Checklist

- [ ] 新功能放在 `modern/` 目录
- [ ] 桥接层清晰定义 (bridge/)
- [ ] Hybrid Theming 确保视觉一致
- [ ] Feature Toggle 控制上线
- [ ] 避免新代码依赖旧代码的内部实作
