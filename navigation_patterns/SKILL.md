---
name: Navigation Patterns
description: Deep Links、跨模块导航与复杂 Back Stack 管理
---

# Navigation Patterns (导航模式)

## Instructions
- 确认需求属于导览与 Back Stack 管理
- 依照下方章节顺序套用
- 一次只处理一个导航面向（Deep Link、跨模块、Back Stack）
- 完成后对照 Quick Checklist

## When to Use
- Scenario A：新项目导航设计
- Scenario B：旧项目扩充与导航桥接

## Example Prompts
- "请参考 Type-Safe Args，更新我的 NavHost 写法"
- "依照 Deep Links 章节，帮我加入 App Links"
- "请用 Multi-Module Navigation 设计跨模块导航接口"

## Workflow
1. 先创建 Compose Navigation 基础路由
2. 再加入 Deep Links 与跨模块导航
3. 最后用 Back Stack 管理与 Quick Checklist 验收

## Practical Notes (2026)
- 默认采用 type-safe args，避免字符串路由散落
- Deep Link 必须有验证与回归测试流程
- Back Stack 规则统一化，避免各模块自订逻辑

## Minimal Template
```
目标: 
路由范围: 
Deep Link: 
Back Stack 规则: 
验收: Quick Checklist
```

---

## Compose Navigation Basics

### Type-Safe Args (Navigation 2.8+)

```kotlin
// 定义路由
@Serializable
data class DetailRoute(val productId: String)

@Serializable
object HomeRoute

// NavHost
NavHost(navController, startDestination = HomeRoute) {
    composable<HomeRoute> { 
        HomeScreen(onProductClick = { id ->
            navController.navigate(DetailRoute(id))
        })
    }
    
    composable<DetailRoute> { backStackEntry ->
        val route = backStackEntry.toRoute<DetailRoute>()
        DetailScreen(productId = route.productId)
    }
}
```

### Nested Graphs

```kotlin
NavHost(navController, startDestination = "main") {
    navigation(startDestination = "home", route = "main") {
        composable("home") { HomeScreen() }
        composable("profile") { ProfileScreen() }
    }
    
    navigation(startDestination = "login", route = "auth") {
        composable("login") { LoginScreen() }
        composable("register") { RegisterScreen() }
    }
}
```

---

## Deep Links

### App Links 设置

```xml
<!-- AndroidManifest.xml -->
<activity android:name=".MainActivity">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:scheme="https"
            android:host="example.com"
            android:pathPrefix="/product" />
    </intent-filter>
</activity>
```

### assetlinks.json (Host 验证)

部署位置：`https://example.com/.well-known/assetlinks.json`

```json
[{
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
        "namespace": "android_app",
        "package_name": "com.example.app",
        "sha256_cert_fingerprints": ["..."]
    }
}]
```

### Navigation 集成

```kotlin
composable(
    route = "product/{id}",
    deepLinks = listOf(
        navDeepLink { uriPattern = "https://example.com/product/{id}" }
    )
) { backStackEntry ->
    val id = backStackEntry.arguments?.getString("id")
    ProductScreen(id)
}
```

---

## Multi-Module Navigation

### API Module Pattern

```kotlin
// :feature:product:api
interface ProductNavigator {
    fun navigateToProduct(productId: String)
    fun navigateToProductList()
}

// :feature:product:impl
class ProductNavigatorImpl @Inject constructor(
    private val navController: NavController
) : ProductNavigator {
    override fun navigateToProduct(productId: String) {
        navController.navigate("product/$productId")
    }
}

// 其他模块使用
class HomeViewModel @Inject constructor(
    private val productNavigator: ProductNavigator
) {
    fun onProductClick(id: String) {
        productNavigator.navigateToProduct(id)
    }
}
```

### Navigation Events (Single Event)

```kotlin
// ViewModel
sealed class NavigationEvent {
    data class ToDetail(val id: String) : NavigationEvent()
    object Back : NavigationEvent()
}

private val _navigationEvent = Channel<NavigationEvent>()
val navigationEvent = _navigationEvent.receiveAsFlow()

// Composable
LaunchedEffect(Unit) {
    viewModel.navigationEvent.collect { event ->
        when (event) {
            is NavigationEvent.ToDetail -> navController.navigate("detail/${event.id}")
            NavigationEvent.Back -> navController.popBackStack()
        }
    }
}
```

---

## Complex Back Stack Management

### Auth Flow (Clear Stack)

```kotlin
fun navigateToHome() {
    navController.navigate("home") {
        popUpTo("auth") { inclusive = true }  // 清除 auth 流程
        launchSingleTop = true
    }
}
```

### Bottom Nav with Separate Stacks

```kotlin
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    
    Scaffold(
        bottomBar = {
            NavigationBar {
                items.forEach { item ->
                    NavigationBarItem(
                        selected = currentRoute == item.route,
                        onClick = {
                            navController.navigate(item.route) {
                                popUpTo(navController.graph.findStartDestination().id) {
                                    saveState = true
                                }
                                launchSingleTop = true
                                restoreState = true
                            }
                        }
                    )
                }
            }
        }
    ) { /* NavHost */ }
}
```

---

## Quick Checklist

- [ ] 使用 Type-Safe Args (Navigation 2.8+)
- [ ] Deep Links 配置 assetlinks.json
- [ ] 跨模块使用 Navigator interface
- [ ] Navigation Events 作为 Single Event 处理
- [ ] Bottom Nav 正确保存/恢复 State
