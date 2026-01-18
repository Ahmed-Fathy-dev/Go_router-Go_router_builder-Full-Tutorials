# الدرس 16: مقدمة عن GoRouter Builder

## المشكلة مع GoRouter العادي

مع GoRouter العادي، الـ navigation بيعتمد على **strings**:

```dart
// ❌ Problems:

// 1. Typos
context.go('/prodcut/123');  // Typo - won't be detected until runtime

// 2. Forgetting parameters
context.go('/user');  // Should be /user/:id

// 3. Wrong types
context.go('/product/abc');  // id should be int

// 4. Changing the path
// If you change /products to /items
// You need to update every place that uses it
```

---

## الحل: GoRouter Builder

**go_router_builder** هو package بيولد كود type-safe للـ routes:

```dart
// ✅ Benefits:

// 1. Compile-time checking
ProductRoute(id: 123).go(context);  // The IDE will help you

// 2. Clear parameters
UserRoute(userId: '456').push(context);  // You won't forget a parameter

// 3. Correct types
ProductRoute(id: 123);  // The id must be int

// 4. Changing the path
// Change the path in one place only
@TypedGoRoute<ProductRoute>(path: '/items/:id')  // Instead of /products/:id
```

---

## كيف يعمل؟

### 1. تكتب Route class

```dart
@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomeScreen();
  }
}
```

### 2. تشغل الـ Code Generator

```bash
dart run build_runner build
```

### 3. الـ Generator بيولد كود

```dart
// Generated: home_route.g.dart

extension $HomeRouteExtension on HomeRoute {
  static HomeRoute _fromState(GoRouterState state) => const HomeRoute();

  String get location => '/';

  void go(BuildContext context) => context.go(location);

  void push(BuildContext context) => context.push(location);

  // ...
}
```

### 4. تستخدمه

```dart
// Instead of
context.go('/');

// Use
const HomeRoute().go(context);
```

---

## المقارنة

### بدون Builder

```dart
// Define the routes
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/user/:userId/order/:orderId',
      builder: (context, state) {
        final userId = state.pathParameters['userId']!;
        final orderId = state.pathParameters['orderId']!;
        final showDetails = state.uri.queryParameters['details'] == 'true';

        return OrderScreen(
          userId: userId,
          orderId: orderId,
          showDetails: showDetails,
        );
      },
    ),
  ],
);

// Usage
context.go('/user/123/order/456?details=true');
```

### مع Builder

```dart
// Define the route
@TypedGoRoute<OrderRoute>(path: '/user/:userId/order/:orderId')
class OrderRoute extends GoRouteData {
  const OrderRoute({
    required this.userId,
    required this.orderId,
    this.showDetails = false,
  });

  final String userId;
  final String orderId;
  final bool showDetails;  // Query parameter

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return OrderScreen(
      userId: userId,
      orderId: orderId,
      showDetails: showDetails,
    );
  }
}

// Usage
const OrderRoute(
  userId: '123',
  orderId: '456',
  showDetails: true,
).go(context);
```

---

## الـ Annotations المتاحة

| Annotation | الاستخدام |
|------------|----------|
| `@TypedGoRoute<T>` | Route عادي |
| `@TypedShellRoute<T>` | ShellRoute |
| `@TypedStatefulShellRoute<T>` | StatefulShellRoute |
| `@TypedStatefulShellBranch<T>` | Branch لـ StatefulShellRoute |

---

## الأنواع المدعومة للـ Parameters

الـ Builder بيدعم تحويل تلقائي لأنواع كتير:

| النوع | مثال |
|-------|------|
| `int` | `id: 123` |
| `double` | `price: 99.99` |
| `String` | `name: 'test'` |
| `bool` | `active: true` |
| `num` | `value: 42` |
| `BigInt` | `bigNumber: BigInt.from(...)` |
| `DateTime` | `date: DateTime.now()` |
| `enum` | `status: Status.active` |
| `Uri` | `link: Uri.parse(...)` |
| `Iterable<T>` | `ids: [1, 2, 3]` |
| `List<T>` | `tags: ['a', 'b']` |
| `Set<T>` | `categories: {'x', 'y'}` |

---

## متى تستخدم GoRouter Builder؟

### استخدمه لو:
- ✅ المشروع كبير أو متوسط
- ✅ فريق من أكتر من developer
- ✅ عندك routes كتير
- ✅ عايز type safety
- ✅ بتستخدم parameters كتير

### ممكن تستغنى عنه لو:
- المشروع صغير جداً (3-4 routes)
- Routes بسيطة بدون parameters
- Prototype سريع

---

## الخطوة الجاية

في الدرس الجاي هنتعلم إزاي نثبت ونعد الـ Builder.

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **المشكلة** | String-based routes = runtime errors |
| **الحل** | Code generation = compile-time checking |
| **الـ Package** | `go_router_builder` |
| **الميزة الرئيسية** | Type-safe navigation |

---

[⬅️ الدرس السابق: Extra Data](../03_go_router_advanced/15_extra_codec.md) | [➡️ الدرس التالي: الإعداد](17_builder_setup.md)
