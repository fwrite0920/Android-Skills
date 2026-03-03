---
name: Observability First
description: Crash/ANR/Logs/Performance 指标与回馈闭环
---

# Observability First (可观测性优先)

## Instructions
- 先定义要观测的关键流程与指标
- 先填写 Required Inputs（流程、阈值、负责人）并冻结
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
1. 先确认 Required Inputs（关键流程、SLO、告警接收人）
2. 定义关键流程与 SLO，并建立事件命名规范
3. 创建 Crash/ANR 与结构化事件
4. 加入性能指标与告警门槛
5. 建立回馈回路（分析 -> 修复 -> 验证）与值班流程
6. 运行 Monitoring Gate 验收命令并记录结果

## Practical Notes (2026)
- 先有指标再谈优化，避免主观调整
- 事件字段需一致，便于查找与汇整
- 告警门槛要可运行且可回溯
- 同一个业务事件只定义一个 canonical name，避免跨系统同义词
- P0 告警必须有 owner 与响应时限（SLA），避免“看见但没人处理”
- 仪表板要区分 release/build version，支持回归比对

## Minimal Template
```
目标:
关键流程 owner:
告警接收渠道:
SLO 窗口(日/周):
关键流程:
指标/事件:
告警门槛:
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `关键流程清单`（启动/登录/支付/列表等）
- `SLO`（指标定义、统计窗口、目标值）
- `Owner & Oncall`（每个 P0/P1 指标对应负责人）
- `告警渠道`（PagerDuty/Slack/Email）
- `事件命名与字段字典`（event name、必填字段、枚举值）
- `发布维度`（build version、flavor、region）

## Deliverables (完成后交付物)

- `SLO 文档`（含阈值、owner、响应时限）
- `结构化事件字典`（字段说明 + 示例）
- `Crash/ANR 上下文策略`（custom keys 与分级）
- `性能仪表板`（启动、网络、滚动、关键交易）
- `告警规则`（P0/P1/P2）与升级路径
- `反馈闭环记录模板`（问题 -> 修复 -> 指标回归）

## Monitoring Gate (验收门槛)

```bash
# 1) 基础质量
./gradlew lint test assemble

# 2) 性能量测（若项目有 benchmark 模块）
./gradlew :benchmark:connectedBenchmarkAndroidTest

# 3) 关键信号验证（按项目脚本调整）
./gradlew :app:connectedDebugAndroidTest

# 4) 发布前手动核查
# - Crashlytics/Performance dashboard 有最近 24h 数据
# - P0 告警路由已验证（至少演练一次）
```

> 没有 benchmark 模块时，需在 PR 说明中记录替代量测方法与结果。

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

        return try {
            val response = chain.proceed(request)
            metric.setResponseContentType(response.header("Content-Type"))
            metric.setHttpResponseCode(response.code)
            metric.setResponsePayloadSize(response.body?.contentLength() ?: 0)
            response
        } catch (e: IOException) {
            metric.putAttribute("result", "io_exception")
            throw e
        } finally {
            metric.stop()
        }
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

- [ ] Required Inputs 已填写并冻结（流程/SLO/owner/告警渠道）
- [ ] 关键流程与 SLO 定义完成
- [ ] 关键事件命名与字段字典完成（含必填字段）
- [ ] Firebase Performance 自定义 Trace 覆盖核心流程
- [ ] 事件字段统一且可查找
- [ ] Crash/ANR 上下文增强（Custom Keys）
- [ ] Non-Fatal 分级策略避免噪音
- [ ] 性能指标有量测与 CI Gate 门槛
- [ ] 告警门槛与回馈回路已创建
- [ ] 每个 P0 指标有明确 owner 与响应时限
- [ ] Monitoring Gate 已执行并记录结果
