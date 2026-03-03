---
name: Testing Legacy Strategies
description: 为无测试的旧代码创建安全网的策略
---

# Testing Legacy Strategies (遗留代码测试)

## Instructions
- 仅在缺乏测试的既有代码上使用
- 先填写 Required Inputs（高风险路径、测试栈、阻挡策略）
- 依照下方章节顺序创建安全网
- 一次只锁定一类行为或输出
- 完成后对照 Quick Checklist

## When to Use
- Scenario C：旧项目现代化前的安全网
- Scenario F：共享逻辑需要测试保障

## Example Prompts
- "请依照 Characterization Tests 章节，替这个类别创建现状测试"
- "用 Robolectric 章节，为依赖 Framework 的 Activity 写测试"
- "请用 Detekt/Lint Baseline 章节创建技术债控管"

## Workflow
1. 先确认 Required Inputs（风险路径、测试优先级、CI 门槛）
2. 创建 Characterization Tests 锁定行为
3. 补齐 Framework 测试与 MockK 策略
4. 建立 Baseline 与 CI 门禁，避免技术债回流
5. 执行 Legacy Test Gate 并用 Quick Checklist 验收

## Practical Notes (2026)
- 测试输出必须可重复与可比对
- 每次只锁定一类行为，避免测试爆量
- Baseline 逐步收敛，避免一次性大改
- 先覆盖高风险业务，再扩展到普通路径
- flaky case 必须标记 owner 与修复截止时间

## Environment & Compatibility (先确认)
- 在开始前先记录测试栈：`AGP` / `Kotlin` / `JUnit4 or JUnit5` / `Robolectric` / `MockK` / `kotlinx-coroutines-test`
- 同一模块避免混搭两套测试 runner；若必须混搭，先确认执行入口与依赖冲突
- Robolectric 版本需与项目 `compileSdk` 与 AGP 组合先做 smoke test
- 所有示例版本号以项目 `libs.versions.toml` 或既有依赖锁定为准
- 若版本未定，请先完成 1 个最小可执行测试再批量铺开

## Minimal Template
```
目标: 
测试范围: 
高风险路径:
CI 阻挡门槛:
行为锁定: 
回归方式: 
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `高风险路径`（金流/登录/权限/写入）
- `测试栈版本`（JUnit/MockK/Robolectric/coroutines-test）
- `CI 阻挡策略`（哪些失败阻挡合并）
- `Baseline 策略`（是否允许、收敛计划）
- `责任人`（测试 owner）

## Deliverables (完成后交付物)

- `Characterization tests` 列表
- `Framework 相关测试`（Robolectric/Instrumented）
- `Baseline` 文件与收敛计划
- `CI 测试门禁` 配置
- `Legacy Test Gate` 验收记录

## Legacy Test Gate (验收门槛)

```bash
./gradlew test
./gradlew connectedDebugAndroidTest
```

> PR 需要说明新增测试覆盖了哪些高风险行为。

---

## Characterization Tests (现状测试)

为没有测试的旧代码撰写「现状测试」，不管对错，先锁定行为。

### 样本选择顺序 (避免盲目铺量)
1. 先测高风险路径：金流、登录、权限、数据写入
2. 再测边界条件：`null`、空集合、最小/最大值、异常输入
3. 最后补常见主路径：最常被调用的 20% 场景
4. 每轮只新增一类行为，确保失败原因单一可诊断

### 策略

```kotlin
// 1. 先写一个会失败的测试
@Test
fun `calculateDiscount returns unknown value`() {
    val result = legacyCalculator.calculateDiscount(100.0, "VIP")
    assertEquals(0.0, result)  // 故意用错误的预期值
}

// 2. 运行测试，记录实际回传值
// AssertionError: expected 0.0 but was 15.0

// 3. 更新测试为实际值
@Test
fun `calculateDiscount returns 15 percent for VIP`() {
    val result = legacyCalculator.calculateDiscount(100.0, "VIP")
    assertEquals(15.0, result)  // 锁定现有行为
}
```

### 批量生成

```kotlin
@ParameterizedTest
@CsvSource(
    "100.0, VIP, 15.0",
    "100.0, REGULAR, 5.0",
    "50.0, VIP, 7.5"
)
fun `calculateDiscount characterization`(
    price: Double,
    tier: String,
    expected: Double
) {
    assertEquals(expected, legacyCalculator.calculateDiscount(price, tier))
}
```

---

## Robolectric (Android Framework 测试)

处理高度依赖 Android Framework 的旧单元测试。

### 设置

```kotlin
// build.gradle.kts
testImplementation("org.robolectric:robolectric:<project-verified-version>")

// 测试类别
@RunWith(RobolectricTestRunner::class)
@Config(sdk = [33])
class LegacyActivityTest {
    private lateinit var activity: LegacyActivity

    @Before
    fun setUp() {
        activity = Robolectric.buildActivity(LegacyActivity::class.java)
            .setup()
            .get()
    }
    
    @Test
    fun `activity displays correct title`() {
        assertEquals("Expected Title", activity.title)
    }
}
```

### Shadow 使用

```kotlin
@Test
fun `shows toast on error`() {
    activity.showError("Network failed")
    
    assertEquals("Network failed", ShadowToast.getTextOfLatestToast())
}
```

---

## MockK for Kotlin

### 基本用法

```kotlin
@Test
fun `repository returns cached data`() {
    val repository = mockk<UserRepository>()
    every { repository.getUser("123") } returns User(id = "123", name = "Test")
    
    val result = repository.getUser("123")
    
    assertEquals("Test", result.name)
    verify { repository.getUser("123") }
}
```

### Coroutines Support

```kotlin
@Test
fun `suspend function mocking`() = runTest {
    val api = mockk<UserApi>()
    coEvery { api.fetchUser("123") } returns User(id = "123", name = "Test")
    
    val result = api.fetchUser("123")
    
    assertEquals("123", result.id)
    assertEquals("Test", result.name)
    coVerify(exactly = 1) { api.fetchUser("123") }
}
```

### Dispatcher 控制 (降低 flaky)

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
class MainDispatcherRule(
    private val dispatcher: TestDispatcher = UnconfinedTestDispatcher()
) : TestWatcher() {
    override fun starting(description: Description) {
        Dispatchers.setMain(dispatcher)
    }

    override fun finished(description: Description) {
        Dispatchers.resetMain()
    }
}
```

### Relaxed Mocks

```kotlin
// 自动回传默认值，适合旧代码测试
val service = mockk<LegacyService>(relaxed = true)
```

---

## Detekt/Lint Baseline

渐进式收紧旧项目的品质标准。

### 生成 Baseline

```bash
# Detekt
./gradlew detektBaseline
# 产生 config/detekt/baseline.xml

# Lint
./gradlew lintDebug -Dlint.baselines.continue=true
# 产生 lint-baseline.xml
```

### 只检查新代码

```kotlin
// build.gradle.kts
android {
    lint {
        baseline = file("lint-baseline.xml")
    }
}

detekt {
    baseline = file("config/detekt/baseline.xml")
}
```

### 渐进式修复

```bash
# 每个 Sprint 减少 Baseline 中的项目
# 1. 修复一批问题
# 2. 重新生成 Baseline
./gradlew detektBaseline
```

### CI 门禁 (防止回流)

```bash
# PR 常规检查：不允许新增违规
./gradlew detekt lintDebug

# 建议策略：
# 1. Baseline 文件只允许在「技术债收敛任务」中变更
# 2. 业务 PR 若修改 baseline，需额外说明与审批
```

---

## Golden Master Testing

适合复杂输出 (HTML, JSON) 的旧代码。

```kotlin
@Test
fun `report generator produces expected output`() {
    val output = legacyReportGenerator.generate(testData)
    
    // 首次运行：保存为 golden file
    // File("src/test/resources/golden/report.html").writeText(output)
    
    // 后续运行：比对
    val golden = File("src/test/resources/golden/report.html").readText()
    assertEquals(golden, output)
}
```

实务建议：
- 先做标准化再比对（时间、时区、随机值、UUID、排序）
- Golden 文件纳入版本控制并在 PR 中审阅 diff
- 输出过大时改用结构化断言 + 关键片段 golden，降低维护成本

---

## Quick Checklist

- [ ] Required Inputs 已填写并冻结（高风险路径/测试栈/门槛）
- [ ] 重构前先撰写 Characterization Tests
- [ ] 每次只锁定一类行为，失败原因可单点定位
- [ ] 使用 Robolectric 处理 Android 依赖
- [ ] MockK/Coroutine 测试同时验证「调用次数 + 业务结果」
- [ ] 使用 `MainDispatcherRule` 或同等机制固定调度器
- [ ] Detekt/Lint Baseline 控制技术债
- [ ] CI 阻挡新增违规，不让 baseline 扩张
- [ ] 每个 Sprint 减少 Baseline 项目
- [ ] 固定时区、Locale、随机种子，确保测试可重复
- [ ] 对 flaky 测试标记原因与修复期限，不长期跳过
- [ ] Legacy Test Gate 已执行并记录结果
