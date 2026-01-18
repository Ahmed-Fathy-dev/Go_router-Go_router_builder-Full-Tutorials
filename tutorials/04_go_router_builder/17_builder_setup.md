# الدرس 17: إعداد GoRouter Builder

## إضافة الـ Dependencies

### في pubspec.yaml

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^17.0.1

dev_dependencies:
  build_runner: ^2.4.13
  go_router_builder: ^4.1.3
```

### تثبيت الـ Packages

```bash
flutter pub get
```

---

## هيكل الملفات

```
lib/
├── main.dart
├── router/
│   ├── app_router.dart      # الـ GoRouter configuration
│   └── routes/
│       ├── routes.dart       # الـ route classes
│       └── routes.g.dart     # Generated file
└── screens/
    ├── home_screen.dart
    ├── product_screen.dart
    └── ...
```

---

## إنشاء الـ Route الأول

### الخطوة 1: إنشاء Route Class

📁 `lib/router/routes/routes.dart`

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

// ⚠️ مهم جداً: أضف الـ part directive
part 'routes.g.dart';

// الـ Home Route
@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomeScreen();
  }
}

// الـ Products Route
@TypedGoRoute<ProductsRoute>(path: '/products')
class ProductsRoute extends GoRouteData {
  const ProductsRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const ProductsScreen();
  }
}

// Product Details مع path parameter
@TypedGoRoute<ProductRoute>(path: '/product/:id')
class ProductRoute extends GoRouteData {
  const ProductRoute({required this.id});

  final int id;  // Path parameter

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductScreen(id: id);
  }
}
```

### الخطوة 2: تشغيل الـ Code Generator

```bash
# تشغيل مرة واحدة
dart run build_runner build

# أو تشغيل مستمر (يراقب التغييرات)
dart run build_runner watch
```

### الخطوة 3: الملف المولد

بعد تشغيل الـ builder، هيتولد ملف `routes.g.dart`:

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'routes.dart';

// **************************************************************************
// GoRouterGenerator
// **************************************************************************

List<RouteBase> get $appRoutes => [
      $homeRoute,
      $productsRoute,
      $productRoute,
    ];

RouteBase get $homeRoute => GoRouteData.$route(
      path: '/',
      factory: $HomeRouteExtension._fromState,
    );

extension $HomeRouteExtension on HomeRoute {
  static HomeRoute _fromState(GoRouterState state) => const HomeRoute();

  String get location => GoRouteData.$location('/');

  void go(BuildContext context) => context.go(location);

  Future<T?> push<T>(BuildContext context) => context.push<T>(location);

  void pushReplacement(BuildContext context) =>
      context.pushReplacement(location);

  void replace(BuildContext context) => context.replace(location);
}

// ... المزيد من الـ extensions
```

---

## إعداد الـ Router

📁 `lib/router/app_router.dart`

```dart
import 'package:go_router/go_router.dart';
import 'routes/routes.dart';

final appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,

  // استخدم الـ $appRoutes المولد
  routes: $appRoutes,
);
```

---

## استخدامه في main.dart

```dart
import 'package:flutter/material.dart';
import 'router/app_router.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'GoRouter Builder Demo',
      routerConfig: appRouter,
    );
  }
}
```

---

## التنقل باستخدام الـ Routes

```dart
// بدل context.go('/products')
const ProductsRoute().go(context);

// بدل context.push('/product/123')
ProductRoute(id: 123).push(context);

// بدل context.go('/product/456')
const ProductRoute(id: 456).go(context);
```

---

## مثال كامل

### routes.dart

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import '../../screens/home_screen.dart';
import '../../screens/products_screen.dart';
import '../../screens/product_screen.dart';
import '../../screens/cart_screen.dart';
import '../../screens/search_screen.dart';

part 'routes.g.dart';

// Home
@TypedGoRoute<HomeRoute>(path: '/')
class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomeScreen();
  }
}

// Products
@TypedGoRoute<ProductsRoute>(
  path: '/products',
  routes: [
    // Sub-route
    TypedGoRoute<ProductRoute>(path: ':id'),
  ],
)
class ProductsRoute extends GoRouteData {
  const ProductsRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const ProductsScreen();
  }
}

// Product Details
class ProductRoute extends GoRouteData {
  const ProductRoute({required this.id});

  final int id;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductScreen(id: id);
  }
}

// Cart
@TypedGoRoute<CartRoute>(path: '/cart')
class CartRoute extends GoRouteData {
  const CartRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const CartScreen();
  }
}

// Search with query parameters
@TypedGoRoute<SearchRoute>(path: '/search')
class SearchRoute extends GoRouteData {
  const SearchRoute({
    this.query,
    this.category,
    this.sortBy = 'relevance',
  });

  final String? query;
  final String? category;
  final String sortBy;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return SearchScreen(
      query: query,
      category: category,
      sortBy: sortBy,
    );
  }
}
```

### استخدامها في الـ Screens

```dart
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('الرئيسية')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () => const ProductsRoute().go(context),
              child: const Text('المنتجات'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => const CartRoute().go(context),
              child: const Text('السلة'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => const SearchRoute(
                query: 'flutter',
                category: 'books',
              ).go(context),
              child: const Text('بحث'),
            ),
          ],
        ),
      ),
    );
  }
}

class ProductsScreen extends StatelessWidget {
  const ProductsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('المنتجات')),
      body: ListView.builder(
        itemCount: 10,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text('منتج ${index + 1}'),
            onTap: () => ProductRoute(id: index + 1).push(context),
          );
        },
      ),
    );
  }
}
```

---

## أوامر build_runner المفيدة

```bash
# تشغيل مرة واحدة
dart run build_runner build

# تشغيل مع مسح الملفات القديمة
dart run build_runner build --delete-conflicting-outputs

# تشغيل مستمر (يراقب التغييرات)
dart run build_runner watch

# مسح الملفات المولدة
dart run build_runner clean
```

---

## نصائح

### 1. لا تنسى الـ part directive

```dart
// ⚠️ لازم تكون موجودة
part 'routes.g.dart';
```

### 2. شغل watch أثناء التطوير

```bash
dart run build_runner watch
```

### 3. أضف .g.dart للـ .gitignore (اختياري)

```gitignore
# Generated files
*.g.dart
```

### 4. تعامل مع أخطاء الـ Generation

```bash
# لو حصل error، جرب
dart run build_runner clean
dart run build_runner build --delete-conflicting-outputs
```

---

## ملخص

| الخطوة | الوصف |
|--------|-------|
| 1 | أضف `go_router_builder` و `build_runner` |
| 2 | أنشئ Route classes مع `@TypedGoRoute` |
| 3 | أضف `part 'filename.g.dart'` |
| 4 | شغل `dart run build_runner build` |
| 5 | استخدم `$appRoutes` في الـ router |
| 6 | انتقل باستخدام `RouteClass().go(context)` |

---

[⬅️ الدرس السابق: المقدمة](16_builder_introduction.md) | [➡️ الدرس التالي: TypedGoRoute](18_typed_routes.md)
