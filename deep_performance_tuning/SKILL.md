---
name: Deep Performance Tuning
description: Systrace, Memory Analysis, R8 优化与 App Startup 调校
---

# Deep Performance Tuning (深度性能优化)

## Instructions
- 仅在有量测数据或明确瓶颈时使用
- 依照下方章节顺序套用
- 一次只施作一种优化并验证效果
- 完成后对照 Quick Checklist

## When to Use
- Scenario D：性能问题排查
- Scenario E：发布前性能验证

## Example Prompts
- "请参考 App Startup Optimization，创建 Macrobenchmark 测量"
- "用 Memory Analysis 章节，规划内存分析流程"
- "请依照 R8/Proguard Optimization，检查规则是否完整"

## Workflow
1. 先创建量测基准（Startup/Memory/UI）
2. 再逐一套用对应优化手段
3. 最后用 Quick Checklist 验收

## Practical Notes (2026)
- Baseline Profile + Macrobenchmark 作为默认流程
- 每次只做一项优化并量测差异
- 性能门槛要进 CI Gate

## Minimal Template
```
目标: 
量测基准: 
优化范围: 
回归指标: 
验收: Quick Checklist
```

---

## App Startup Optimization

### Macrobenchmark 测量

```kotlin
@LargeTest
@RunWith(AndroidJUnit4::class)
class StartupBenchmark {
    
    @get:Rule
    val benchmarkRule = MacrobenchmarkRule()
    
    @Test
    fun startupCompilationNone() = startup(CompilationMode.None())
    
    @Test
    fun startupCompilationPartial() = startup(CompilationMode.Partial())
    
    private fun startup(compilationMode: CompilationMode) {
        benchmarkRule.measureRepeated(
            packageName = "com.example.app",
            metrics = listOf(StartupTimingMetric()),
            compilationMode = compilationMode,
            iterations = 5,
            startupMode = StartupMode.COLD
        ) {
            pressHome()
            startActivityAndWait()
        }
    }
}
```

### Baseline Profiles

```kotlin
// 生成 Baseline Profile
@RunWith(AndroidJUnit4::class)
class BaselineProfileGenerator {
    
    @get:Rule
    val rule = BaselineProfileRule()
    
    @Test
    fun generate() {
        rule.collect("com.example.app") {
            pressHome()
            startActivityAndWait()
            
            // 关键路径
            device.findObject(By.text("Login")).click()
            device.wait(Until.hasObject(By.text("Home")), 5000)
        }
    }
}
```

### Application onCreate 优化

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        
        // ❌ 同步初始化 (Block UI)
        // Firebase.initialize(this)
        // Timber.plant(DebugTree())
        
        // ✅ 延迟初始化
        AppInitializer.getInstance(this)
            .initializeComponent(FirebaseInitializer::class.java)
        
        // ✅ 背景初始化
        ProcessLifecycleOwner.get().lifecycle.addObserver(
            object : DefaultLifecycleObserver {
                override fun onCreate(owner: LifecycleOwner) {
                    owner.lifecycleScope.launch(Dispatchers.Default) {
                        // 非关键初始化
                    }
                }
            }
        )
    }
}
```

---

## Memory Analysis

### Heap Dump 分析

```bash
# 取得 Heap Dump
adb shell am dumpheap com.example.app /data/local/tmp/heap.hprof
adb pull /data/local/tmp/heap.hprof

# 使用 Android Studio Profiler 分析
# 或使用 MAT (Memory Analyzer Tool)
```

### LeakCanary 设置

```kotlin
// build.gradle.kts
debugImplementation("com.squareup.leakcanary:leakcanary-android:2.12")

// 自定义报告
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        LeakCanary.config = LeakCanary.config.copy(
            retainedVisibleThreshold = 3,
            objectInspectors = AndroidObjectInspectors.appDefaults
        )
    }
}
```

### Bitmap 内存管理

```kotlin
// 使用 Coil 自动管理
AsyncImage(
    model = ImageRequest.Builder(LocalContext.current)
        .data(imageUrl)
        .size(Size.ORIGINAL)  // 根据需求调整
        .memoryCachePolicy(CachePolicy.ENABLED)
        .build(),
    contentDescription = null
)
```

---

## R8/Proguard Optimization

### 自定义规则

```proguard
# 保留 Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
}

# 保留 Retrofit Interface
-keep,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}

# 保留 Compose stability
-keep class * {
    @androidx.compose.runtime.Stable *;
    @androidx.compose.runtime.Immutable *;
}
```

### APK Size 分析

```bash
# 使用 bundletool
bundletool build-apks --bundle=app.aab --output=app.apks
bundletool get-size total --apks=app.apks

# Android Studio: Build > Analyze APK
```

---

## UI Performance

### JankStats

```kotlin
class MainActivity : ComponentActivity() {
    private lateinit var jankStats: JankStats
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        jankStats = JankStats.createAndTrack(window) { frameData ->
            if (frameData.isJank) {
                Log.w("Jank", "Frame took ${frameData.frameDurationUiNanos / 1_000_000}ms")
            }
        }
    }
}
```

### Compose Recomposition Tracking

```kotlin
// 打开 Composition Tracking
@Composable
fun DebugComposable() {
    SideEffect {
        Log.d("Recomposition", "DebugComposable recomposed")
    }
}

// 使用 Layout Inspector 检查 recomposition count
```

---

## Quick Checklist

- [ ] Startup: Baseline Profiles 生成
- [ ] Startup: Application onCreate 延迟初始化
- [ ] Memory: LeakCanary 无泄漏
- [ ] APK: R8 规则完善
- [ ] UI: JankStats 无严重掉帧
