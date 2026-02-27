---
name: Observability First
description: Crash/ANR/Logs/Performance 指标与回馈闭环
---

# Observability First (可观测性优先)

## Instructions
- 先定义要观测的关键流程与指标
- 依序创建 Crash/ANR、结构化日志、性能指标
- 一次只补强一类信号，避免噪音扩散
- 完成后对照 Quick Checklist

## When to Use
- 发布前需要稳定监控与回馈闭环
- 事故频繁但缺乏可定位信息
- 需要把性能与稳定性纳入日常决策

## Example Prompts
- "请创建支付流程的可观测性指标与事件"
- "请设计 Crash/ANR 的告警门槛"
- "帮我创建结构化日志的字段规格"
- "请用 OpenTelemetry 追踪关键 API 调用链"

## Workflow
1. 定义关键流程与 SLO
2. 创建 Crash/ANR 与结构化事件
3. 加入性能指标与告警门槛
4. 创建回馈回路（分析 -> 修复 -> 验证）

## Practical Notes (2026)
- 先有指标再谈优化，避免主观调整
- 事件字段需一致，便于查找与汇整
- 告警门槛要可运行且可回溯

## Minimal Template
```
目标:
关键流程:
指标/事件:
告警门槛:
验收: Quick Checklist
```

---

## Signals & SLOs

### 关键流程清单

| 流程 | P0 指标 | SLO 目标 |
|------|---------|----------|
| 启动 | Cold Start 时间 | P95 < 1.5s |
| 登录 | 成功率 | > 99.5% |
| 核心交易 | 完成率、延迟 | 成功率 > 99.9%, P95 < 3s |
| 列表滚动 | 掉帧率 | Jank < 1% |
| 网络请求 | 成功率、延迟 | 成功率 > 99%, P95 < 500ms |

### 指标分级

```kotlin
enum class MetricPriority {
    P0,  // Crash/ANR 率、关键流程失败率 — 立即告警
    P1,  // 首次渲染时间、列表滚动流畅度 — 每日检视
    P2   // 特定功能转换率或完成率 — 每周检视
}
```

---

## Firebase Performance Monitoring

### 自定义 Trace

```kotlin
class CheckoutTracer @Inject constructor() {

    fun <T> traceCheckout(block: () -> T): T {
        val trace = Firebase.performance.newTrace("checkout_flow")
        trace.start()
        return try {
            val result = block()
            trace.putAttribute("result", "success")
            result
        } catch (e: Exception) {
            trace.putAttribute("result", "failure")
            trace.putAttribute("error", e.javaClass.simpleName)
            throw e
        } finally {
            trace.stop()
        }
    }
}

// 使用
class CheckoutUseCase @Inject constructor(
    private val tracer: CheckoutTracer,
    private val repository: OrderRepository
) {
    suspend fun execute(order: Order): OrderResult {
        return tracer.traceCheckout {
            repository.submitOrder(order)
        }
    }
}
```

### 网络请求自动追踪

```kotlin
class PerformanceInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        val metric = Firebase.performance.newHttpMetric(
            request.url.toString(),
            request.method
        )
        metric.start()

        val response = chain.proceed(request)

        metric.setResponseContentType(response.header("Content-Type"))
        metric.setHttpResponseCode(response.code)
        metric.setResponsePayloadSize(response.body?.contentLength() ?: 0)
        metric.stop()

        return response
    }
}
```

---

## Structured Events

### 统一事件接口

```kotlin
interface AnalyticsTracker {
    fun track(event: AnalyticsEvent)
}

data class AnalyticsEvent(
    val name: String,
    val params: Map<String, Any> = emptyMap()
)

class CompositeTracker @Inject constructor(
    private val trackers: Set<@JvmSuppressWildcards AnalyticsTracker>
) : AnalyticsTracker {
    override fun track(event: AnalyticsEvent) {
        trackers.forEach { it.track(event) }
    }
}

class FirebaseTracker @Inject constructor() : AnalyticsTracker {
    override fun track(event: AnalyticsEvent) {
        Firebase.analytics.logEvent(event.name) {
            event.params.forEach { (key, value) ->
                when (value) {
                    is String -> param(key, value)
                    is Long -> param(key, value)
                    is Double -> param(key, value)
                    is Bundle -> param(key, value)
                }
            }
        }
    }
}
```

### 事件字段规格

```kotlin
object EventKeys {
    const val FLOW_ID = "flow_id"
    const val USER_TIER = "user_tier"
    const val BUILD_VERSION = "build_version"
    const val LATENCY_MS = "latency_ms"
    const val RESULT = "result"
    const val ERROR_CODE = "error_code"
    const val SCREEN_NAME = "screen_name"
}

// 使用
tracker.track(AnalyticsEvent(
    name = "checkout_completed",
    params = mapOf(
        EventKeys.FLOW_ID to flowId,
        EventKeys.LATENCY_MS to duration,
        EventKeys.RESULT to "success",
        EventKeys.USER_TIER to "premium"
    )
))
```

---

## Crash / ANR Strategy

### Crash 上下文增强

```kotlin
class CrashContextManager @Inject constructor() {

    fun setFlowContext(flowName: String, params: Map<String, String> = emptyMap()) {
        Firebase.crashlytics.apply {
            setCustomKey("current_flow", flowName)
            setCustomKey("flow_timestamp", System.currentTimeMillis().toString())
            params.forEach { (k, v) -> setCustomKey(k, v) }
        }
    }

    fun clearFlowContext() {
        Firebase.crashlytics.setCustomKey("current_flow", "none")
    }
}
```

### Non-Fatal 分级策略

```kotlin
enum class NonFatalSeverity { LOW, MEDIUM, HIGH }

class NonFatalReporter @Inject constructor() {

    fun report(
        exception: Exception,
        severity: NonFatalSeverity,
        context: Map<String, String> = emptyMap()
    ) {
        if (severity == NonFatalSeverity.LOW) return

        Firebase.crashlytics.apply {
            setCustomKey("severity", severity.name)
            context.forEach { (k, v) -> setCustomKey(k, v) }
            recordException(exception)
        }
    }
}
```

---

## Performance Signals

### Startup 量测上报

```kotlin
class StartupMetricReporter @Inject constructor(
    private val tracker: AnalyticsTracker
) {
    private var processStartTime: Long = 0L

    fun onProcessStart() {
        processStartTime = SystemClock.elapsedRealtime()
    }

    fun onFirstFrameRendered() {
        val duration = SystemClock.elapsedRealtime() - processStartTime
        tracker.track(AnalyticsEvent(
            name = "app_startup",
            params = mapOf(
                EventKeys.LATENCY_MS to duration,
                "startup_type" to "cold"
            )
        ))
    }
}
```

### CI Gate 性能门槛

```yaml
# .github/workflows/performance-gate.yml
name: Performance Gate
on:
  pull_request:
    branches: [main]
jobs:
  benchmark:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Macrobenchmark
        uses: reactivecircus/android-emulator-runner@v2
        with:
          api-level: 34
          script: ./gradlew :benchmark:connectedBenchmarkAndroidTest
      - name: Check Startup Threshold
        run: |
          STARTUP_MS=$(jq '.benchmarks[0].metrics.timeToInitialDisplayMs.median' benchmark/build/outputs/connected_android_test_additional_output/benchmarkData.json)
          if (( $(echo "$STARTUP_MS > 1500" | bc -l) )); then
            echo "Startup ${STARTUP_MS}ms exceeds 1500ms threshold"
            exit 1
          fi
```

---

## Alerting & Feedback Loop

### 告警门槛

| 指标 | 告警门槛 | 动作 |
|------|----------|------|
| Crash-free rate | < 99.5% | P0 立即通知 |
| ANR rate | > 0.5% | P0 立即通知 |
| API 成功率 | < 99% | P1 当日处理 |
| Cold Start P95 | > 2s | P1 当日处理 |
| Jank rate | > 5% | P2 本周处理 |

### 回馈回路

```
发现问题 → 定位根因 → 修复 → 验证指标回归 → 更新 SLO
    ↑                                              │
    └──────────────────────────────────────────────┘
```

---

## Quick Checklist

- [ ] 关键流程与 SLO 定义完成
- [ ] Firebase Performance 自定义 Trace 覆盖核心流程
- [ ] 事件字段统一且可查找
- [ ] Crash/ANR 上下文增强（Custom Keys）
- [ ] Non-Fatal 分级策略避免噪音
- [ ] 性能指标有量测与 CI Gate 门槛
- [ ] 告警门槛与回馈回路已创建
