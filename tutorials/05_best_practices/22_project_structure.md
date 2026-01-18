# الدرس 22: هيكلة المشروع

## هيكلة بسيطة (للمشاريع الصغيرة)

```
lib/
├── main.dart
├── router.dart           # All routing in one file
└── screens/
    ├── home_screen.dart
    ├── product_screen.dart
    └── ...
```

```dart
// router.dart
import 'package:go_router/go_router.dart';

final appRouter = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) => ProductScreen(
        id: state.pathParameters['id']!,
      ),
    ),
  ],
);
```

---

## هيكلة متوسطة (للمشاريع المتوسطة)

```
lib/
├── main.dart
├── router/
│   ├── app_router.dart       # GoRouter configuration
│   ├── route_names.dart      # Route names as constants
│   └── routes.dart           # Define routes
└── features/
    ├── home/
    │   └── home_screen.dart
    ├── products/
    │   ├── products_screen.dart
    │   └── product_details_screen.dart
    └── auth/
        ├── login_screen.dart
        └── register_screen.dart
```

```dart
// router/route_names.dart
abstract class RouteNames {
  static const home = 'home';
  static const products = 'products';
  static const productDetails = 'product-details';
  static const login = 'login';
  static const register = 'register';
}

// router/routes.dart
import 'package:go_router/go_router.dart';
import 'route_names.dart';

List<RouteBase> get appRoutes => [
  GoRoute(
    path: '/',
    name: RouteNames.home,
    builder: (context, state) => const HomeScreen(),
  ),
  GoRoute(
    path: '/products',
    name: RouteNames.products,
    builder: (context, state) => const ProductsScreen(),
    routes: [
      GoRoute(
        path: ':id',
        name: RouteNames.productDetails,
        builder: (context, state) => ProductDetailsScreen(
          id: state.pathParameters['id']!,
        ),
      ),
    ],
  ),
  GoRoute(
    path: '/login',
    name: RouteNames.login,
    builder: (context, state) => const LoginScreen(),
  ),
];

// router/app_router.dart
import 'package:go_router/go_router.dart';
import 'routes.dart';

final appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,
  routes: appRoutes,
);
```

---

## هيكلة متقدمة (للمشاريع الكبيرة)

```
lib/
├── main.dart
├── app/
│   └── app.dart
├── core/
│   ├── router/
│   │   ├── app_router.dart
│   │   ├── route_names.dart
│   │   ├── router_notifier.dart
│   │   └── routes/
│   │       ├── app_routes.dart
│   │       ├── auth_routes.dart
│   │       ├── home_routes.dart
│   │       └── product_routes.dart
│   └── guards/
│       ├── auth_guard.dart
│       └── admin_guard.dart
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── screens/
│   │       └── widgets/
│   ├── home/
│   │   └── ...
│   └── products/
│       └── ...
└── shared/
    ├── widgets/
    └── utils/
```

---

## هيكلة مع GoRouter Builder

```
lib/
├── main.dart
├── router/
│   ├── app_router.dart
│   └── routes/
│       ├── routes.dart         # Main file
│       ├── routes.g.dart       # Generated
│       ├── home_routes.dart
│       ├── product_routes.dart
│       ├── auth_routes.dart
│       └── shell_routes.dart
└── features/
    └── ...
```

```dart
// router/routes/routes.dart
import 'package:go_router/go_router.dart';

// Export all routes
export 'home_routes.dart';
export 'product_routes.dart';
export 'auth_routes.dart';
export 'shell_routes.dart';

part 'routes.g.dart';

// router/routes/home_routes.dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

part of 'routes.dart';

@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const HomeScreen();
}
```

---

## أفضل الممارسات للهيكلة

### 1. فصل الـ Router عن الـ Routes

```dart
// Wrong ❌ - everything in one big file
final router = GoRouter(
  redirect: (context, state) { ... },
  routes: [
    GoRoute(path: '/', builder: ...),
    GoRoute(path: '/products', builder: ...),
    // 50+ routes...
  ],
);

// Correct ✅ - separated
// app_router.dart
final appRouter = GoRouter(
  redirect: authRedirect,
  routes: appRoutes,
);

// routes.dart
final appRoutes = [
  ...homeRoutes,
  ...productRoutes,
  ...authRoutes,
];
```

### 2. استخدم Route Names كـ Constants

```dart
// Wrong ❌ - strings everywhere
context.goNamed('product-details');
context.goNamed('product_details');  // Oops, different spelling!

// Correct ✅ - constants
abstract class RouteNames {
  static const productDetails = 'product-details';
}
context.goNamed(RouteNames.productDetails);
```

### 3. أنشئ Navigation Helper

```dart
// navigation_helper.dart
class AppNavigator {
  static void goHome(BuildContext context) {
    context.goNamed(RouteNames.home);
  }

  static void goProductDetails(
    BuildContext context, {
    required String productId,
  }) {
    context.goNamed(
      RouteNames.productDetails,
      pathParameters: {'id': productId},
    );
  }

  static Future<bool?> showConfirmDialog(
    BuildContext context, {
    required String message,
  }) {
    return context.pushNamed<bool>(
      RouteNames.confirmDialog,
      extra: message,
    );
  }
}

// Usage
AppNavigator.goProductDetails(context, productId: '123');
```

### 4. فصل الـ Guards

```dart
// guards/auth_guard.dart
String? authGuard(BuildContext context, GoRouterState state) {
  final isLoggedIn = AuthService.isLoggedIn;
  final isAuthRoute = state.uri.path.startsWith('/login') ||
                       state.uri.path.startsWith('/register');

  if (!isLoggedIn && !isAuthRoute) {
    return '/login?redirect=${state.uri}';
  }

  if (isLoggedIn && isAuthRoute) {
    return '/';
  }

  return null;
}

// guards/admin_guard.dart
String? adminGuard(BuildContext context, GoRouterState state) {
  if (!AuthService.isAdmin) {
    return '/';
  }
  return null;
}

// app_router.dart
final appRouter = GoRouter(
  redirect: (context, state) {
    // Execute guards in order
    return authGuard(context, state) ??
           adminGuard(context, state);
  },
  routes: appRoutes,
);
```

---

## Feature-based Structure

```
lib/
├── features/
│   ├── products/
│   │   ├── routes.dart          # Routes for this feature
│   │   ├── screens/
│   │   │   ├── products_screen.dart
│   │   │   └── product_details_screen.dart
│   │   └── widgets/
│   │       └── product_card.dart
│   │
│   └── auth/
│       ├── routes.dart
│       ├── screens/
│       │   ├── login_screen.dart
│       │   └── register_screen.dart
│       └── services/
│           └── auth_service.dart
│
└── router/
    └── app_router.dart          # Combines all routes
```

```dart
// features/products/routes.dart
final productRoutes = [
  GoRoute(
    path: '/products',
    name: 'products',
    builder: (context, state) => const ProductsScreen(),
    routes: [
      GoRoute(
        path: ':id',
        name: 'product-details',
        builder: (context, state) => ProductDetailsScreen(
          id: state.pathParameters['id']!,
        ),
      ),
    ],
  ),
];

// router/app_router.dart
import '../features/products/routes.dart';
import '../features/auth/routes.dart';

final appRoutes = [
  ...productRoutes,
  ...authRoutes,
];
```

---

## ملخص

| حجم المشروع | الهيكل المقترح |
|-------------|---------------|
| **صغير** | ملف router.dart واحد |
| **متوسط** | مجلد router/ منفصل |
| **كبير** | Feature-based + router مركزي |

---

[⬅️ الدرس السابق: Builder Advanced](../04_go_router_builder/21_builder_advanced.md) | [➡️ الدرس التالي: Common Patterns](23_common_patterns.md)
