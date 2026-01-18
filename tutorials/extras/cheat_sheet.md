# GoRouter Cheat Sheet

## Navigation Methods

```dart
// الذهاب (يمسح الـ stack)
context.go('/path');
context.goNamed('name');

// Push (يضيف للـ stack)
context.push('/path');
context.pushNamed('name');

// Pop (رجوع)
context.pop();
context.pop(result);  // مع return value

// استبدال
context.replace('/path');
context.pushReplacement('/path');

// التحقق
context.canPop();
```

---

## Route Definition

```dart
GoRoute(
  path: '/path/:id',          // المسار
  name: 'route-name',         // الاسم (اختياري)
  builder: (context, state) => Screen(),
  pageBuilder: (context, state) => CustomPage(),
  redirect: (context, state) => '/other',
  onExit: (context, state) => showConfirm(),
  routes: [...],              // Sub-routes
)
```

---

## Parameters

```dart
// Path Parameters
path: '/user/:userId/post/:postId'
state.pathParameters['userId']
state.pathParameters['postId']

// Query Parameters
// /search?q=flutter&page=2
state.uri.queryParameters['q']
state.uri.queryParameters['page']

// Extra
context.go('/path', extra: myObject);
state.extra as MyType;
```

---

## GoRouter Configuration

```dart
GoRouter(
  initialLocation: '/',
  routes: [...],

  // Redirect
  redirect: (context, state) => null,
  redirectLimit: 5,

  // Error
  errorBuilder: (context, state) => ErrorScreen(),
  onException: (context, state, router) {},

  // Refresh
  refreshListenable: authNotifier,

  // Debug
  debugLogDiagnostics: true,

  // Navigator Key
  navigatorKey: GlobalKey<NavigatorState>(),
)
```

---

## ShellRoute

```dart
ShellRoute(
  builder: (context, state, child) => Shell(child: child),
  routes: [...],
)

StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) => Shell(
    navigationShell: navigationShell,
  ),
  branches: [
    StatefulShellBranch(routes: [...]),
  ],
)

// التنقل بين الـ branches
navigationShell.goBranch(index);
navigationShell.goBranch(index, initialLocation: true);
```

---

## GoRouter Builder

```dart
// التعريف
@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(context, state) => HomeScreen();
}

// الاستخدام
const HomeRoute().go(context);
HomeRoute().push(context);
```

---

## Parameters في Builder

```dart
@TypedGoRoute<ProductRoute>(path: '/product/:id')
class ProductRoute extends GoRouteData {
  const ProductRoute({
    required this.id,     // Path parameter
    this.sort,            // Query parameter
    this.showReviews = false,
    this.$extra,          // Extra data
  });

  final int id;
  final String? sort;
  final bool showReviews;
  final Product? $extra;
}

ProductRoute(id: 123, sort: 'price', $extra: product).go(context);
```

---

## Custom Transitions

```dart
pageBuilder: (context, state) => CustomTransitionPage(
  key: state.pageKey,
  child: MyScreen(),
  transitionsBuilder: (context, animation, secondaryAnimation, child) {
    return FadeTransition(opacity: animation, child: child);
  },
)

// بدون transition
NoTransitionPage(key: state.pageKey, child: MyScreen())
```

---

## Redirect Patterns

```dart
// Top-level
redirect: (context, state) {
  if (!isLoggedIn) return '/login';
  return null;
}

// Route-level
GoRoute(
  path: '/admin',
  redirect: (context, state) {
    if (!isAdmin) return '/';
    return null;
  },
)

// مع refreshListenable
GoRouter(
  refreshListenable: authNotifier,  // ChangeNotifier
  redirect: ...
)
```

---

## Quick Reference

| الوظيفة | الكود |
|---------|-------|
| Navigate | `context.go('/path')` |
| Push | `context.push('/path')` |
| Pop | `context.pop()` |
| Named | `context.goNamed('name')` |
| Path param | `state.pathParameters['id']` |
| Query param | `state.uri.queryParameters['q']` |
| Current path | `GoRouterState.of(context).uri.path` |
| Router | `GoRouter.of(context)` |

---

## Commands

```bash
# GoRouter Builder
dart run build_runner build
dart run build_runner watch
dart run build_runner clean
```
