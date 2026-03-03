---
name: Tech Stack Migration
description: View→Compose, RxJava→Flow 等技术迁移指南
---

# Tech Stack Migration (技术迁移)

## Instructions
- 仅在需要渐进式迁移时使用
- 先填写 Required Inputs（迁移范围、回滚策略、验收指标）
- 依照下方章节顺序套用
- 一次只替换一个技术栈
- 完成后对照 Quick Checklist

## When to Use
- Scenario C：旧项目现代化

## Example Prompts
- "请依照 View → Compose 章节，帮我嵌入 ComposeView"
- "用 RxJava → Flow 对照表，改写这段 stream"
- "请用 LiveData → StateFlow 的步骤规划迁移"

## Workflow
1. 先确认 Required Inputs（迁移维度、回滚点、指标阈值）
2. 确认要迁移的技术栈与范围
3. 依序套用对应章节的范例与对照表
4. 执行 Migration Gate 并比较迁移前后指标
5. 用 Quick Checklist 验收

## Practical Notes (2026)
- 迁移必先有可验证的测试安全网
- 一次只迁移一个维度（UI 或 Data 或 DI）
- 指标回归后才能推进下一步
- 迁移期间必须保留双栈观测点，便于快速定位回归
- 功能开关应覆盖新旧实现切换

## Minimal Template
```
目标: 
迁移范围: 
回滚策略:
观测指标:
测试安全网: 
回归指标: 
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `迁移范围`（模块/页面/数据流）
- `回滚策略`（开关、版本、降级路径）
- `回归指标`（崩溃率、启动、耗时、成功率）
- `测试覆盖`（单元/UI/集成）
- `负责人`（迁移 owner 与 reviewer）

## Deliverables (完成后交付物)

- 迁移设计说明（旧 -> 新 对照）
- 双栈过渡实现与开关
- 回归测试清单与结果
- 指标对比报告（迁移前后）
- `Migration Gate` 验收记录

## Migration Gate (验收门槛)

```bash
./gradlew test
./gradlew connectedDebugAndroidTest
```

> 每个迁移 PR 需要附带回滚步骤与开关状态说明。

---

## View → Compose Interoperability

### Compose in XML

```xml
<!-- layout/activity_main.xml -->
<androidx.compose.ui.platform.ComposeView
    android:id="@+id/compose_view"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        findViewById<ComposeView>(R.id.compose_view).setContent {
            AppTheme {
                NewFeatureCard()
            }
        }
    }
}
```

### XML in Compose

```kotlin
@Composable
fun LegacyMapView(modifier: Modifier = Modifier) {
    AndroidView(
        modifier = modifier,
        factory = { context ->
            MapView(context).apply {
                onCreate(null)
            }
        },
        update = { mapView ->
            mapView.getMapAsync { /* configure */ }
        },
        onRelease = { mapView ->
            mapView.onDestroy()
        }
    )
}
```

### Fragment in Compose

```kotlin
@Composable
fun LegacyFragmentContainer() {
    AndroidViewBinding(LegacyFragmentContainerBinding::inflate) {
        val fragment = LegacyFragment()
        fragmentContainerView.getFragment<Fragment>() ?: run {
            (LocalContext.current as FragmentActivity)
                .supportFragmentManager
                .beginTransaction()
                .replace(fragmentContainerView.id, fragment)
                .commit()
        }
    }
}
```

---

## RxJava → Coroutines/Flow

### Operator Mapping

| RxJava | Coroutines/Flow | 备注 |
|--------|-----------------|------|
| `Observable` | `Flow` | Cold stream |
| `Single` | `suspend fun` | 单值 |
| `Completable` | `suspend fun` | 无回传 |
| `flatMap` | `flatMapLatest` | 取消前一个 |
| `flatMap` | `flatMapConcat` | 依序运行 |
| `switchMap` | `flatMapLatest` | 等同 |
| `debounce` | `debounce` | 相同 |
| `combineLatest` | `combine` | 相同 |
| `zip` | `zip` | 相同 |
| `observeOn(main)` | 在 Main scope `collect` | `flowOn` 不会切换下游 collector |
| `subscribeOn(io)` | `flowOn(Dispatchers.IO)` | 仅影响上游 |

### 范例：Search with Debounce

```kotlin
// Before (RxJava)
searchEditText.textChanges()
    .debounce(300, TimeUnit.MILLISECONDS)
    .switchMap { query -> api.search(query) }
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe { results -> updateUI(results) }

// After (Flow)
lifecycleScope.launch {
    searchFlow
        .debounce(300)
        .flatMapLatest { query ->
            flow { emit(api.search(query)) }
                .flowOn(Dispatchers.IO) // 对应 Rx 的 subscribeOn(IO)
        }
        .collect { results -> updateUI(results) } // 对应 Rx 的 observeOn(Main)
}
```

### Error Handling 差异

```kotlin
// RxJava: onError 终止 stream
observable
    .onErrorReturn { defaultValue }
    .subscribe()

// Flow: catch 会拦截上游异常；是否继续取决于是否 emit fallback
flow
    .catch { emit(defaultValue) }
    .collect()

// Flow: 重试
flow
    .retry(3) { e -> e is IOException }
    .collect()
```

---

## LiveData → StateFlow

### 渐进式替换

```kotlin
// Step 1: ViewModel 内部用 StateFlow
private val _uiState = MutableStateFlow(UiState())
val uiState: StateFlow<UiState> = _uiState.asStateFlow()

// Step 2: 暴露 LiveData 给旧 UI (过渡期)
val uiStateLiveData: LiveData<UiState> = uiState.asLiveData()

// Step 3: 新 UI 直接 collect StateFlow
@Composable
fun Screen(viewModel: MyViewModel) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
}
```

---

## Dagger → Hilt

### 渐进式迁移

```kotlin
// 1. 保留 Dagger Component，加入 Hilt
@HiltAndroidApp
class MyApp : Application() {
    // 保留旧的 Dagger component (过渡期)
    val legacyComponent by lazy { DaggerLegacyComponent.create() }
}

// 2. 新模块用 Hilt
@Module
@InstallIn(SingletonComponent::class)
object NewModule { }

// 3. 桥接旧模块
@Module
@InstallIn(SingletonComponent::class)
object LegacyBridgeModule {
    @Provides
    fun provideLegacyService(
        @ApplicationContext context: Context
    ): LegacyService {
        return (context.applicationContext as MyApp).legacyComponent.legacyService()
    }
}
```

---

## Quick Checklist

- [ ] Required Inputs 已填写并冻结（范围/回滚/指标）
- [ ] Compose 与 View 的生命周期对齐
- [ ] AndroidView 正确处理 onRelease
- [ ] Flow operator 对照 RxJava 正确
- [ ] StateFlow 使用 collectAsStateWithLifecycle
- [ ] Hilt 迁移维持向后兼容
- [ ] Migration Gate 已执行并记录结果
