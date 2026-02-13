---
name: Testing Legacy Strategies
description: 为无测试的旧代码创建安全网的策略
---

# Testing Legacy Strategies (遗留代码测试)

## Instructions
- 仅在缺乏测试的既有代码上使用
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
1. 先创建 Characterization Tests 锁定行为
2. 再补齐 Framework 测试与 MockK 策略
3. 最后用 Quick Checklist 验收

## Practical Notes (2026)
- 测试输出必须可重复与可比对
- 每次只锁定一类行为，避免测试爆量
- Baseline 逐步收敛，避免一次性大改

## Minimal Template
```
目标: 
测试范围: 
行为锁定: 
回归方式: 
验收: Quick Checklist
```

---

## Characterization Tests (现状测试)

为没有测试的旧代码撰写「现状测试」，不管对错，先锁定行为。

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
testImplementation("org.robolectric:robolectric:4.11.1")

// 测试类别
@RunWith(RobolectricTestRunner::class)
@Config(sdk = [33])
class LegacyActivityTest {
    
    @Test
    fun `activity displays correct title`() {
        val activity = Robolectric.buildActivity(LegacyActivity::class.java)
            .create()
            .resume()
            .get()
        
        assertEquals("Expected Title", activity.title)
    }
}
```

### Shadow 使用

```kotlin
@Test
fun `shows toast on error`() {
    activity.showError("Network failed")
    
    val toast = ShadowToast.getLatestToast()
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
    coEvery { api.fetchUser("123") } returns User(id = "123")
    
    val result = api.fetchUser("123")
    
    coVerify { api.fetchUser("123") }
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

---

## Quick Checklist

- [ ] 重构前先撰写 Characterization Tests
- [ ] 使用 Robolectric 处理 Android 依赖
- [ ] MockK 的 relaxed mock 加速旧代码测试
- [ ] Detekt/Lint Baseline 控制技术债
- [ ] 每个 Sprint 减少 Baseline 项目
