---
name: UI/UX Engineering
description: Design System 实作、复杂 UI 模式与 Accessibility
---

# UI/UX Engineering (UI/UX 工程化)

## Instructions
- 确认需求属于 UI 架构、交互或可近用性
- 先填写 Required Inputs（屏幕范围、设计 tokens、a11y 门槛）
- 依照下方章节顺序套用
- 一次只调整一种 UI 模式或设计 token
- 完成后对照 Quick Checklist

## When to Use
- Scenario A：新项目设计系统创建
- Scenario B：旧项目添加 UI 功能
- Scenario D：UI 性能与体验问题

## Example Prompts
- "请依照 Design System Implementation 章节，创建主题与 spacing"
- "请参考 Complex UI Patterns，实作 Collapsing Toolbar"
- "用 Accessibility 章节查看主要流程是否符合 a11y"

## Workflow
1. 先确认 Required Inputs（范围、断点、a11y 目标）
2. 创建 Design System 的基础 tokens
3. 套用 Complex UI Patterns 与 Adaptive Layouts
4. 执行 UI Gate（性能 + a11y）并记录结果
5. 用 Accessibility 与 Quick Checklist 验收

## Practical Notes (2026)
- Compose-first，但保留 View/Fragment 互通规范
- a11y 验收必包含 TalkBack 走查与触摸目标
- UI 性能以重组与列表滚动为优先检查点
- 动效与布局变更必须考虑低端机表现
- 设计 token 变更应集中在主题层，避免散落硬编码

## Minimal Template
```
目标: 
画面范围: 
断点策略:
性能目标:
设计规范: 
a11y 要求: 
验收: Quick Checklist
```

---

## Required Inputs (执行前输入)

- `页面范围`（核心流程与非核心流程）
- `设计 tokens`（颜色/字体/间距）
- `断点策略`（手机/平板/折叠）
- `a11y 阈值`（对比度、触控面积、读屏）
- `性能目标`（重组次数、滚动掉帧）

## Deliverables (完成后交付物)

- `Theme + tokens` 定义
- `关键组件`（可复用、可测试）
- `a11y 校验` 结果（TalkBack/对比度/触控）
- `UI 性能` 量测记录
- `UI Gate` 验收记录

## UI Gate (验收门槛)

```bash
./gradlew test
./gradlew connectedDebugAndroidTest
```

> PR 需附关键页面截图（至少手机与一个大屏断点）。

---

## Design System Implementation

### Theme 架构

```kotlin
// ui/theme/Theme.kt
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme
    
    CompositionLocalProvider(
        LocalSpacing provides Spacing(),
        LocalTypography provides AppTypography,
    ) {
        MaterialTheme(
            colorScheme = colorScheme,
            typography = AppTypography,
            content = content
        )
    }
}
```

### Spacing System

```kotlin
data class Spacing(
    val xs: Dp = 4.dp,
    val sm: Dp = 8.dp,
    val md: Dp = 16.dp,
    val lg: Dp = 24.dp,
    val xl: Dp = 32.dp,
)

val LocalSpacing = staticCompositionLocalOf { Spacing() }

// 使用
val spacing = LocalSpacing.current
Spacer(modifier = Modifier.height(spacing.md))
```

### Figma Token Sync

```kotlin
// 从 Figma 导出的 Tokens
object DesignTokens {
    object Colors {
        val Primary = Color(0xFF6200EE)
        val OnPrimary = Color(0xFFFFFFFF)
        val Surface = Color(0xFFFFFBFE)
    }
    
    object Typography {
        val HeadlineLarge = TextStyle(
            fontFamily = FontFamily.Default,
            fontWeight = FontWeight.Bold,
            fontSize = 32.sp,
            lineHeight = 40.sp
        )
    }
}
```

---

## Complex UI Patterns

### Collapsing Toolbar

```kotlin
@Composable
fun CollapsingToolbarScreen() {
    val scrollState = rememberLazyListState()
    val toolbarHeight = 200.dp
    val minToolbarHeight = 56.dp
    
    val toolbarHeightPx = with(LocalDensity.current) { toolbarHeight.toPx() }
    val minToolbarHeightPx = with(LocalDensity.current) { minToolbarHeight.toPx() }
    
    val toolbarOffsetHeightPx = remember { mutableFloatStateOf(0f) }
    
    val nestedScrollConnection = remember {
        object : NestedScrollConnection {
            override fun onPreScroll(available: Offset, source: NestedScrollSource): Offset {
                val delta = available.y
                val newOffset = toolbarOffsetHeightPx.floatValue + delta
                toolbarOffsetHeightPx.floatValue = newOffset.coerceIn(
                    minToolbarHeightPx - toolbarHeightPx, 0f
                )
                return Offset.Zero
            }
        }
    }
    
    Box(
        modifier = Modifier
            .fillMaxSize()
            .nestedScroll(nestedScrollConnection)
    ) {
        LazyColumn(state = scrollState) { /* content */ }
        
        Box(
            modifier = Modifier
                .height(with(LocalDensity.current) {
                    (toolbarHeightPx + toolbarOffsetHeightPx.floatValue).toDp()
                })
                .fillMaxWidth()
                .background(MaterialTheme.colorScheme.primary)
        )
    }
}
```

### Shared Element Transition

```kotlin
// Navigation 3.0+ with SharedTransitionLayout
SharedTransitionLayout {
    AnimatedContent(targetState = screen) { targetScreen ->
        when (targetScreen) {
            is Screen.List -> {
                ListItem(
                    modifier = Modifier.sharedElement(
                        state = rememberSharedContentState(key = "image-${item.id}"),
                        animatedVisibilityScope = this@AnimatedContent
                    )
                )
            }
            is Screen.Detail -> {
                DetailImage(
                    modifier = Modifier.sharedElement(
                        state = rememberSharedContentState(key = "image-${item.id}"),
                        animatedVisibilityScope = this@AnimatedContent
                    )
                )
            }
        }
    }
}
```

---

## Adaptive Layouts

### WindowSizeClass

```kotlin
@Composable
fun AdaptiveLayout() {
    val activity = LocalContext.current as Activity
    val windowSizeClass = calculateWindowSizeClass(activity)
    
    when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> {
            // Phone: Single column, bottom nav
            CompactLayout()
        }
        WindowWidthSizeClass.Medium -> {
            // Tablet portrait: Navigation rail
            MediumLayout()
        }
        WindowWidthSizeClass.Expanded -> {
            // Tablet landscape / Desktop: Permanent drawer
            ExpandedLayout()
        }
    }
}
```

---

## Accessibility (a11y)

### Semantic Properties

```kotlin
@Composable
fun AccessibleButton(
    onClick: () -> Unit,
    label: String
) {
    Button(
        onClick = onClick,
        modifier = Modifier.semantics {
            contentDescription = label
            role = Role.Button
        }
    )
}
```

### TalkBack Testing Checklist

- [ ] 所有可交互元素都有 `contentDescription`
- [ ] 图片有适当的 `contentDescription` 或标记为 decorative
- [ ] 触摸目标 >= 48dp
- [ ] 颜色对比度符合 WCAG 2.1 标准
- [ ] 使用 TalkBack 完整走过主要流程

### Decorative Images

```kotlin
Image(
    painter = painterResource(R.drawable.decoration),
    contentDescription = null,  // 装饰性图片
    modifier = Modifier.semantics { invisibleToUser() }
)
```

---

## Quick Checklist

- [ ] Required Inputs 已填写并冻结（范围/tokens/a11y 目标）

### Design System
- [ ] Color Scheme 定义 (Light/Dark)
- [ ] Typography Scale 定义
- [ ] Spacing System (Tokens)
- [ ] Common Components 封装

### Accessibility
- [ ] 所有按钮有 contentDescription
- [ ] 触摸目标 >= 48dp
- [ ] 颜色对比度检查
- [ ] TalkBack 测试通过
- [ ] UI Gate 已执行并记录结果
