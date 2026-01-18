# الدرس 18: TypedGoRoute

## الـ Annotation الأساسي

`@TypedGoRoute<T>` هو الـ annotation الأساسي لتعريف الـ routes:

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

---

## خصائص TypedGoRoute

```dart
@TypedGoRoute<MyRoute>(
  path: '/my-path',         // The path (required)
  name: 'my-route',         // The route name (optional)
  routes: [...],            // Sub-routes (optional)
)
```

---

## GoRouteData

كل route لازم يمتد من `GoRouteData`:

```dart
class MyRoute extends GoRouteData {
  const MyRoute();

  // The required method
  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const MyScreen();
  }
}
```

### Methods المتاحة في GoRouteData

| Method | الوظيفة |
|--------|---------|
| `build()` | بناء الـ Widget (إجباري) |
| `buildPage()` | بناء Page مخصصة (للـ transitions) |
| `redirect()` | إعادة التوجيه |
| `onExit()` | callback قبل مغادرة الصفحة |

---

## Sub-Routes

### تعريف routes متداخلة

```dart
@TypedGoRoute<ProductsRoute>(
  path: '/products',
  routes: [
    TypedGoRoute<ProductRoute>(path: ':id'),
    TypedGoRoute<ProductReviewsRoute>(path: ':id/reviews'),
  ],
)
class ProductsRoute extends GoRouteData {
  const ProductsRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const ProductsScreen();
  }
}

class ProductRoute extends GoRouteData {
  const ProductRoute({required this.id});

  final int id;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductScreen(id: id);
  }
}

class ProductReviewsRoute extends GoRouteData {
  const ProductReviewsRoute({required this.id});

  final int id;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductReviewsScreen(productId: id);
  }
}
```

### النتيجة:
```
/products              -> ProductsScreen
/products/123          -> ProductScreen(id: 123)
/products/123/reviews  -> ProductReviewsScreen(productId: 123)
```

---

## Nesting عميق

```dart
@TypedGoRoute<ShopRoute>(
  path: '/shop',
  routes: [
    TypedGoRoute<CategoryRoute>(
      path: 'category/:categoryId',
      routes: [
        TypedGoRoute<SubcategoryRoute>(
          path: 'subcategory/:subId',
          routes: [
            TypedGoRoute<ItemRoute>(path: 'item/:itemId'),
          ],
        ),
      ],
    ),
  ],
)
class ShopRoute extends GoRouteData { ... }
class CategoryRoute extends GoRouteData { ... }
class SubcategoryRoute extends GoRouteData { ... }
class ItemRoute extends GoRouteData { ... }

// Result:
// /shop
// /shop/category/electronics
// /shop/category/electronics/subcategory/phones
// /shop/category/electronics/subcategory/phones/item/iphone-15
```

---

## الـ Methods المولدة

بعد تشغيل الـ builder، كل route بيحصل على methods جاهزة:

```dart
extension $ProductRouteExtension on ProductRoute {
  // The location string
  String get location => '/product/$id';

  // Navigation methods
  void go(BuildContext context);
  Future<T?> push<T>(BuildContext context);
  void pushReplacement(BuildContext context);
  void replace(BuildContext context);
}

// Usage
ProductRoute(id: 123).go(context);
ProductRoute(id: 456).push(context);
print(ProductRoute(id: 789).location);  // '/product/789'
```

---

## Named Routes

```dart
@TypedGoRoute<SettingsRoute>(
  path: '/settings',
  name: 'settings',  // 👈 The route name
)
class SettingsRoute extends GoRouteData {
  const SettingsRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const SettingsScreen();
  }
}

// The name is used in debugging and analytics
```

---

## redirect في Route

```dart
@TypedGoRoute<AdminRoute>(path: '/admin')
class AdminRoute extends GoRouteData {
  const AdminRoute();

  @override
  String? redirect(BuildContext context, GoRouterState state) {
    final user = AuthService.currentUser;

    // If not admin
    if (user == null || !user.isAdmin) {
      return const HomeRoute().location;
    }

    return null;  // Don't redirect
  }

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const AdminScreen();
  }
}
```

---

## onExit في Route

```dart
@TypedGoRoute<EditRoute>(path: '/edit/:id')
class EditRoute extends GoRouteData {
  const EditRoute({required this.id});

  final int id;

  @override
  Future<bool> onExit(BuildContext context, GoRouterState state) async {
    // If there are unsaved changes
    if (hasUnsavedChanges) {
      return await showDialog<bool>(
        context: context,
        builder: (context) => AlertDialog(
          title: const Text('تأكيد'),
          content: const Text('فيه تغييرات مش محفوظة. متأكد تريد الخروج؟'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context, false),
              child: const Text('لا'),
            ),
            TextButton(
              onPressed: () => Navigator.pop(context, true),
              child: const Text('نعم'),
            ),
          ],
        ),
      ) ?? false;
    }

    return true;  // Allow exit
  }

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return EditScreen(id: id);
  }
}
```

---

## Multiple Top-level Routes

```dart
part 'routes.g.dart';

// Route 1
@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData { ... }

// Route 2
@TypedGoRoute<LoginRoute>(path: '/login')
class LoginRoute extends GoRouteData { ... }

// Route 3
@TypedGoRoute<SettingsRoute>(path: '/settings')
class SettingsRoute extends GoRouteData { ... }

// All will be collected in $appRoutes
```

---

## تنظيم الـ Routes في ملفات متعددة

### طريقة 1: ملف واحد كبير

```
lib/router/routes/
└── routes.dart  # All routes
```

### طريقة 2: ملفات متعددة مع export

```
lib/router/routes/
├── routes.dart         # The main file
├── home_routes.dart
├── product_routes.dart
└── auth_routes.dart
```

```dart
// routes.dart
export 'home_routes.dart';
export 'product_routes.dart';
export 'auth_routes.dart';

// Here is the $appRoutes
part 'routes.g.dart';

// Combine routes
List<RouteBase> get appRoutes => [
  ...$homeRoutes,
  ...$productRoutes,
  ...$authRoutes,
];
```

---

## مثال شامل

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

part 'routes.g.dart';

// ==================== Home ====================
@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const HomeScreen();
}

// ==================== Products ====================
@TypedGoRoute<ProductsRoute>(
  path: '/products',
  name: 'products',
  routes: [
    TypedGoRoute<ProductRoute>(path: ':id'),
  ],
)
class ProductsRoute extends GoRouteData {
  const ProductsRoute({this.category, this.sortBy = 'popular'});

  final String? category;     // Query parameter
  final String sortBy;        // Query parameter with default

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      ProductsScreen(category: category, sortBy: sortBy);
}

class ProductRoute extends GoRouteData {
  const ProductRoute({required this.id, this.$extra});

  final int id;
  final Product? $extra;     // Extra data

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      ProductScreen(id: id, product: $extra);
}

// ==================== Auth ====================
@TypedGoRoute<LoginRoute>(path: '/login')
class LoginRoute extends GoRouteData {
  const LoginRoute({this.redirectTo});

  final String? redirectTo;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      LoginScreen(redirectTo: redirectTo);
}

// ==================== Protected Route ====================
@TypedGoRoute<ProfileRoute>(path: '/profile')
class ProfileRoute extends GoRouteData {
  const ProfileRoute();

  @override
  String? redirect(BuildContext context, GoRouterState state) {
    if (!AuthService.isLoggedIn) {
      return LoginRoute(redirectTo: location).location;
    }
    return null;
  }

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const ProfileScreen();
}
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| `@TypedGoRoute<T>` | الـ annotation لتعريف route |
| `GoRouteData` | الـ base class لكل route |
| `build()` | بناء الـ Widget |
| `redirect()` | إعادة التوجيه |
| `onExit()` | تأكيد قبل الخروج |
| `routes:` | للـ sub-routes |
| `$appRoutes` | القائمة المولدة |

---

[⬅️ الدرس السابق: الإعداد](17_builder_setup.md) | [➡️ الدرس التالي: Parameters](19_builder_parameters.md)
