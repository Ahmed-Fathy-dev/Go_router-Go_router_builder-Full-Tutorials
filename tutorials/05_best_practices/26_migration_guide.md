# الدرس 26: Migration Guide

## الترقية من v16 لـ v17

### التغييرات الرئيسية

1. **Minimum Flutter**: 3.32
2. **onEnter blocking fix**: تم إصلاح مشكلة فقدان navigation stack

### الخطوات

```yaml
# pubspec.yaml
dependencies:
  go_router: ^17.0.1
```

```bash
flutter pub upgrade go_router
```

---

## الترقية من v15 لـ v16

### التغييرات الرئيسية

1. **onEnter callback جديد**
2. **notifyRootObserver** parameter جديد

### إضافة onEnter

```dart
// v16+ new
GoRouter(
  onEnter: (context, state) {
    // Before redirect
    return EnterResult.allow();
  },
  redirect: (context, state) {
    // After onEnter
  },
  routes: [...],
)
```

---

## الترقية من v14 لـ v15

### التغيير الرئيسي: Case Sensitivity

```dart
// v14: Case insensitive by default
// /Product/123 = /product/123

// v15: Case SENSITIVE by default
// /Product/123 ≠ /product/123

// To keep the old behavior:
GoRouter(
  caseSensitive: false,  // 👈 Add this
  routes: [...],
)
```

---

## الترقية من Navigator 1.0 لـ GoRouter

### قبل (Navigator 1.0)

```dart
// Navigation
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => DetailsScreen()),
);

// Pop
Navigator.pop(context);

// Replace
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => HomeScreen()),
);
```

### بعد (GoRouter)

```dart
// Navigation
context.push('/details');
// Or
context.go('/details');

// Pop
context.pop();

// Replace
context.pushReplacement('/home');
// Or
context.go('/home');  // If you want to clear the stack
```

---

## الترقية من auto_route

### قبل (auto_route)

```dart
@MaterialAutoRouter(
  routes: [
    AutoRoute(page: HomeScreen, initial: true),
    AutoRoute(page: DetailsScreen, path: '/details/:id'),
  ],
)
class AppRouter extends _$AppRouter {}

// Navigation
context.router.push(DetailsRoute(id: 123));
```

### بعد (GoRouter Builder)

```dart
@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData {
  const HomeRoute();
  @override
  Widget build(context, state) => const HomeScreen();
}

@TypedGoRoute<DetailsRoute>(path: '/details/:id')
class DetailsRoute extends GoRouteData {
  const DetailsRoute({required this.id});
  final int id;
  @override
  Widget build(context, state) => DetailsScreen(id: id);
}

// Navigation
DetailsRoute(id: 123).push(context);
```

---

## Migration Checklist

### من Navigator 1.0

- [ ] أضف `go_router` dependency
- [ ] أنشئ `GoRouter` configuration
- [ ] غير `MaterialApp` لـ `MaterialApp.router`
- [ ] استبدل `Navigator.push` بـ `context.push`
- [ ] استبدل `Navigator.pop` بـ `context.pop`
- [ ] حول الـ named routes لـ GoRouter routes

### من v15 لـ v17

- [ ] حدث الـ dependency
- [ ] راجع الـ case sensitivity
- [ ] اختبر الـ onEnter لو بتستخدمه

### من GoRouter العادي للـ Builder

- [ ] أضف `go_router_builder` و `build_runner`
- [ ] أنشئ Route classes
- [ ] أضف `part` directive
- [ ] شغل `build_runner`
- [ ] استبدل `context.go('/path')` بـ `Route().go(context)`

---

## نصائح للـ Migration

### 1. اعملها تدريجياً

```dart
// Step 1: Add GoRouter while keeping old routes
final router = GoRouter(
  routes: [
    // New routes
    GoRoute(path: '/', ...),

    // Route using old Navigator (temporary)
    GoRoute(
      path: '/legacy/:screen',
      builder: (context, state) {
        return LegacyScreenWrapper(
          screenName: state.pathParameters['screen']!,
        );
      },
    ),
  ],
);

// Step 2: Convert one route at a time
// Step 3: Delete LegacyScreenWrapper when done
```

### 2. اختبر كل حاجة

```dart
// Write tests before migration
testWidgets('navigation works', (tester) async {
  // ...
});

// Run tests after each change
```

### 3. استخدم debugLogDiagnostics

```dart
GoRouter(
  debugLogDiagnostics: true,  // See navigation in console
  routes: [...],
)
```

---

## Breaking Changes History

| الإصدار | التغيير |
|---------|---------|
| v17.0.0 | onEnter blocking fix |
| v16.0.0 | notifyRootObserver parameter |
| v15.0.0 | Case sensitive by default |
| v14.0.0 | Removed deprecated APIs |

---

[⬅️ الدرس السابق: Performance](25_performance.md) | [🏠 الرئيسية](../../README.md)
