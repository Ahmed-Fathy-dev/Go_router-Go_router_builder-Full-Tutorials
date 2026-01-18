# الدرس 0: قصة Navigation في Flutter

## المقدمة

قبل ما نبدأ نتعلم GoRouter، لازم نفهم الأول ليه هو موجود أصلاً. القصة بدأت من الـ Navigation system في Flutter.

---

## Navigator 1.0 (الطريقة القديمة)

### إيه هو Navigator 1.0؟

ده الـ navigation system الأصلي في Flutter. بيشتغل بطريقة **Imperative** (أوامر مباشرة).

```dart
// Push a new screen
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => DetailsScreen()),
);

// Go back
Navigator.pop(context);

// Replace the current screen
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => HomeScreen()),
);

// Clear everything and go to new screen
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => LoginScreen()),
  (route) => false,
);
```

### مميزات Navigator 1.0

| الميزة | الوصف |
|--------|-------|
| ✅ سهل التعلم | Push و Pop بس |
| ✅ مباشر | تكتب الأمر ويتنفذ |
| ✅ مناسب للـ Mobile | معظم التطبيقات بسيطة |

### عيوب Navigator 1.0

| المشكلة | التأثير |
|---------|---------|
| ❌ مفيش URL حقيقي | الويب مش بيشتغل صح |
| ❌ Deep Linking صعب | لينكات خارجية معقدة |
| ❌ Browser Back/Forward | مش بيشتغلوا |
| ❌ State Management | صعب تتحكم في الـ stack |
| ❌ مفيش Route Hierarchy | كل route مستقل |

### مثال على المشكلة

```dart
// You want: /users/123/posts/456
// But how you do it in Navigator 1.0?

Navigator.push(context, MaterialPageRoute(
  builder: (_) => UserScreen(userId: '123'),
));

// Then inside UserScreen:
Navigator.push(context, MaterialPageRoute(
  builder: (_) => PostScreen(userId: '123', postId: '456'),
));

// Problems:
// 1. The URL doesn't change in the browser
// 2. Refresh = back to home 😱
// 3. Can't share the link
// 4. Browser back doesn't work as expected
```

---

## ليه Flutter عملت Navigator 2.0؟

في 2020، Flutter كانت عايزة تدعم الويب بشكل أفضل. المشاكل الرئيسية كانت:

### 1. الويب محتاج URLs حقيقية
```
https://myapp.com/products/123
https://myapp.com/users/ahmed/posts
```

### 2. Deep Linking للموبايل
```
myapp://product/123
```

### 3. Browser Navigation
- زرار Back و Forward
- الـ History
- الـ Bookmarks

### 4. تحكم أفضل في الـ Navigation Stack
```dart
// I want this stack:
// [Home] -> [Category] -> [Product]
// How to build it programmatically?
```

---

## Navigator 2.0 (الطريقة الجديدة)

### إيه هو Navigator 2.0؟

نظام navigation جديد بيشتغل بطريقة **Declarative** (وصفية). بدل ما تقول "روح للصفحة دي"، بتوصف "الصفحات دي هي اللي المفروض تكون موجودة".

### المكونات الأساسية

```
┌─────────────────────────────────────────────────────────┐
│                    Navigator 2.0                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┐    ┌─────────────────────────┐    │
│  │ RouteInformation │───▶│ RouteInformationParser │    │
│  │    Provider      │    │   (URL ↔ State)        │    │
│  └─────────────────┘    └─────────────────────────┘    │
│           │                         │                   │
│           ▼                         ▼                   │
│  ┌─────────────────────────────────────────────────┐   │
│  │              RouterDelegate                      │   │
│  │         (Builds the Navigator)                   │   │
│  └─────────────────────────────────────────────────┘   │
│                         │                               │
│                         ▼                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │                  Navigator                       │   │
│  │              (Shows Pages)                       │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### مثال Navigator 2.0 (الكود الكامل!)

```dart
// 1. The App State
class AppState extends ChangeNotifier {
  String? _selectedProductId;
  bool _showProducts = false;

  String? get selectedProductId => _selectedProductId;
  bool get showProducts => _showProducts;

  set selectedProductId(String? id) {
    _selectedProductId = id;
    notifyListeners();
  }

  set showProducts(bool value) {
    _showProducts = value;
    notifyListeners();
  }
}

// 2. The Route Information Parser
class MyRouteInformationParser extends RouteInformationParser<AppState> {
  @override
  Future<AppState> parseRouteInformation(RouteInformation routeInformation) async {
    final uri = Uri.parse(routeInformation.location ?? '/');
    final state = AppState();

    // Handle: /
    if (uri.pathSegments.isEmpty) {
      return state;
    }

    // Handle: /products
    if (uri.pathSegments[0] == 'products') {
      state.showProducts = true;

      // Handle: /products/:id
      if (uri.pathSegments.length == 2) {
        state.selectedProductId = uri.pathSegments[1];
      }
    }

    return state;
  }

  @override
  RouteInformation? restoreRouteInformation(AppState configuration) {
    if (configuration.selectedProductId != null) {
      return RouteInformation(
        location: '/products/${configuration.selectedProductId}',
      );
    }
    if (configuration.showProducts) {
      return const RouteInformation(location: '/products');
    }
    return const RouteInformation(location: '/');
  }
}

// 3. The Router Delegate
class MyRouterDelegate extends RouterDelegate<AppState>
    with ChangeNotifier, PopNavigatorRouterDelegateMixin<AppState> {

  @override
  final GlobalKey<NavigatorState> navigatorKey = GlobalKey<NavigatorState>();

  AppState _state = AppState();

  @override
  AppState get currentConfiguration => _state;

  @override
  Future<void> setNewRoutePath(AppState configuration) async {
    _state = configuration;
    notifyListeners();
  }

  @override
  Widget build(BuildContext context) {
    return Navigator(
      key: navigatorKey,
      pages: [
        // Home is always present
        const MaterialPage(
          key: ValueKey('home'),
          child: HomeScreen(),
        ),

        // Products page
        if (_state.showProducts)
          const MaterialPage(
            key: ValueKey('products'),
            child: ProductsScreen(),
          ),

        // Product details
        if (_state.selectedProductId != null)
          MaterialPage(
            key: ValueKey('product-${_state.selectedProductId}'),
            child: ProductDetailsScreen(id: _state.selectedProductId!),
          ),
      ],
      onPopPage: (route, result) {
        if (!route.didPop(result)) return false;

        // Handle back button
        if (_state.selectedProductId != null) {
          _state.selectedProductId = null;
        } else if (_state.showProducts) {
          _state.showProducts = false;
        }

        notifyListeners();
        return true;
      },
    );
  }
}

// 4. The App
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routeInformationParser: MyRouteInformationParser(),
      routerDelegate: MyRouterDelegate(),
    );
  }
}
```

### 😱 شفت الكود ده؟!

**150+ سطر** عشان تعمل 3 صفحات بس!

وده بدون:
- ❌ Authentication & Guards
- ❌ Nested Routes
- ❌ Query Parameters
- ❌ Transitions
- ❌ Error Handling

---

## المقارنة المباشرة

### Navigator 1.0
```dart
// Go to product details
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (_) => ProductDetails(id: '123'),
  ),
);
```
**3 سطور، سهل، مفهوم**

### Navigator 2.0 (Pure)
```dart
// Go to product details
appState.selectedProductId = '123';
appState.showProducts = true;
// And hope the RouterDelegate rebuilds correctly...
```
**بتغير state وتستنى الـ delegate يعمل rebuild**

### GoRouter
```dart
// Go to product details
context.go('/products/123');
// Or
context.push('/products/123');
```
**سطر واحد، واضح، وبيشتغل صح!**

---

## جدول المقارنة

| الميزة | Navigator 1.0 | Navigator 2.0 | GoRouter |
|--------|---------------|---------------|----------|
| **سهولة التعلم** | ⭐⭐⭐⭐⭐ | ⭐ | ⭐⭐⭐⭐ |
| **URL Support** | ❌ | ✅ | ✅ |
| **Deep Linking** | ❌ | ✅ | ✅ |
| **Web Support** | ❌ | ✅ | ✅ |
| **Type Safety** | ❌ | ❌ | ✅ (with Builder) |
| **Boilerplate** | قليل | كتير جداً | قليل |
| **Maintenance** | سهل | صعب | سهل |
| **Documentation** | جيد | معقد | ممتاز |

---

## الخلاصة

### Navigator 1.0
> سهل بس محدود. مناسب للتطبيقات البسيطة اللي مش محتاجة URLs.

### Navigator 2.0
> قوي بس معقد جداً. محدش بيستخدمه directly.

### GoRouter
> الحل الوسط المثالي. سهولة Navigator 1.0 + قوة Navigator 2.0.

```
   Ease of Use                              Power
        │                                     │
        │  Nav 1.0          GoRouter    Nav 2.0
        │    ●────────────────●────────────●
        │   Easy            Sweet         Complex
        │   Limited         Spot          Powerful
        │                                     │
```

---

## في الدرس الجاي

هنتعلم إيه هو GoRouter بالتفصيل وإزاي بيحل كل المشاكل دي بطريقة سهلة وأنيقة.

---

[➡️ الدرس التالي: مقدمة GoRouter](01_introduction.md)
