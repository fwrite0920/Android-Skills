---
name: Crash Monitoring
description: Crashlytics 设置、ANR 分析与结构化日志
---

# Crash Monitoring (当机监控)

## Instructions
- 确认需求属于当机/ANR/日志监控
- 依照下方章节顺序套用
- 一次只调整一种监控或警报设置
- 完成后对照 Quick Checklist

## When to Use
- Scenario D：性能问题排查
- Scenario E：发布前监控准备

## Example Prompts
- "请依照 Firebase Crashlytics Setup，完成 Crashlytics 集成"
- "用 ANR Analysis 章节加入 StrictMode 与 ANR Watchdog"
- "请依 Structured Logging 章节统一日志格式"

## Workflow
1. 先完成 Crashlytics 基本设置与 Custom Keys
2. 再加入 ANR 与 Structured Logging
3. 最后用 Dashboard & Alerting 与 Quick Checklist 验收

## Practical Notes (2026)
- Release 前先确立 Crash/ANR 的告警门槛
- 重要流程必有结构化事件与关键键值
- Non-Fatal 记录以高价值事件为主

## Minimal Template
```
目标: 
监控范围: 
告警门槛: 
事件字段: 
验收: Quick Checklist
```

---

## Firebase Crashlytics Setup

### 基本设置

```kotlin
// build.gradle.kts (app)
plugins {
    id("com.google.firebase.crashlytics")
}

dependencies {
    implementation(platform("com.google.firebase:firebase-bom:32.7.0"))
    implementation("com.google.firebase:firebase-crashlytics-ktx")
    implementation("com.google.firebase:firebase-analytics-ktx")
}
```

### Custom Keys

```kotlin
// 记录用户状态
Firebase.crashlytics.apply {
    setUserId("user_123")
    setCustomKey("subscription_tier", "premium")
    setCustomKey("feature_flags", "new_checkout:true")
}
```

### Non-Fatal Logging

```kotlin
// 记录非致命错误
try {
    riskyOperation()
} catch (e: Exception) {
    Firebase.crashlytics.recordException(e)
    // 继续正常流程
}

// 带上下文的 Non-Fatal
Firebase.crashlytics.log("Starting payment flow")
try {
    processPayment()
} catch (e: PaymentException) {
    Firebase.crashlytics.apply {
        setCustomKey("payment_amount", amount)
        setCustomKey("payment_method", method)
        recordException(e)
    }
}
```

---

## ANR Analysis

### StrictMode (Debug)

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        
        if (BuildConfig.DEBUG) {
            StrictMode.setThreadPolicy(
                StrictMode.ThreadPolicy.Builder()
                    .detectDiskReads()
                    .detectDiskWrites()
                    .detectNetwork()
                    .penaltyLog()
                    .penaltyDeath()  // 强制 Crash
                    .build()
            )
            
            StrictMode.setVmPolicy(
                StrictMode.VmPolicy.Builder()
                    .detectLeakedClosableObjects()
                    .detectActivityLeaks()
                    .penaltyLog()
                    .build()
            )
        }
    }
}
```

### ANR Watchdog

```kotlin
class ANRWatchDog(private val timeoutMs: Long = 5000) : Thread() {
    
    private val mainHandler = Handler(Looper.getMainLooper())
    private var tick = 0
    private var reported = false
    
    override fun run() {
        while (!isInterrupted) {
            val currentTick = tick
            mainHandler.post { tick++ }
            
            Thread.sleep(timeoutMs)
            
            if (currentTick == tick && !reported) {
                reported = true
                val stackTraces = Looper.getMainLooper().thread.stackTrace
                Firebase.crashlytics.log("ANR Detected")
                Firebase.crashlytics.recordException(ANRException(stackTraces))
            }
        }
    }
}
```

---

## Structured Logging

### Timber Setup

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        
        if (BuildConfig.DEBUG) {
            Timber.plant(Timber.DebugTree())
        } else {
            Timber.plant(CrashlyticsTree())
        }
    }
}

class CrashlyticsTree : Timber.Tree() {
    override fun log(priority: Int, tag: String?, message: String, t: Throwable?) {
        if (priority >= Log.INFO) {
            Firebase.crashlytics.log("[$tag] $message")
        }
        
        t?.let {
            Firebase.crashlytics.recordException(it)
        }
    }
}
```

### Log Levels 使用规范

| Level | 使用场景 | 例子 |
|-------|---------|------|
| VERBOSE | Debug 专用细节 | API Response body |
| DEBUG | 开发调试 | ViewModel state changes |
| INFO | 重要里程碑 | User login success |
| WARN | 潜在问题 | Retry attempt |
| ERROR | 可恢复错误 | Network timeout |

---

## Dashboard & Alerting

### Crashlytics Velocity Alerts

```
Firebase Console > Crashlytics > Settings
- Enable velocity alerts
- Set threshold: 1% of sessions
```

### Custom Metrics

```kotlin
// 追踪关键指标
Firebase.analytics.logEvent("checkout_started") {
    param("cart_value", cartValue)
}

Firebase.analytics.logEvent("checkout_completed") {
    param("payment_method", method)
    param("time_to_complete_ms", duration)
}
```

---

## Quick Checklist

- [ ] Crashlytics 集成完成
- [ ] Custom Keys 设置 (User tier, Feature flags)
- [ ] Non-Fatal 记录重要异常
- [ ] StrictMode 在 Debug 激活
- [ ] Timber 统一日志
- [ ] Velocity Alerts 设置
