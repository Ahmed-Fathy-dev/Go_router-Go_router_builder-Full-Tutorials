# الدرس 6: Named Routes

## يعني إيه Named Routes؟

بدل ما تكتب الـ path كـ string في كل مكان، تقدر تدي الـ route اسم وتستخدم الاسم ده في الـ navigation. ده بيخلي الكود:
- أسهل في الصيانة
- أقل عرضة للأخطاء
- أوضح في القراءة

---

## تعريف Named Routes

### إضافة اسم للـ Route

```dart
GoRoute(
  path: '/user/:id',
  name: 'user-details',  // 👈 الاسم
  builder: (context, state) {
    final id = state.pathParameters['id']!;
    return UserDetailsScreen(id: id);
  },
)
```

### مثال كامل

```dart
final appRouter = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      name: 'home',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/products',
      name: 'products',
      builder: (context, state) => const ProductsScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      name: 'product-details',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductDetailsScreen(id: id);
      },
    ),
    GoRoute(
      path: '/user/:userId/orders',
      name: 'user-orders',
      builder: (context, state) {
        final userId = state.pathParameters['userId']!;
        return UserOrdersScreen(userId: userId);
      },
    ),
    GoRoute(
      path: '/search',
      name: 'search',
      builder: (context, state) {
        final query = state.uri.queryParameters['q'] ?? '';
        return SearchScreen(query: query);
      },
    ),
  ],
);
```

---

## التنقل باستخدام الأسماء

### goNamed()

```dart
// بدل كده
context.go('/products');

// استخدم كده
context.goNamed('products');
```

### مع Path Parameters

```dart
// بدل كده
context.go('/product/123');

// استخدم كده
context.goNamed(
  'product-details',
  pathParameters: {'id': '123'},
);
```

### مع Query Parameters

```dart
// بدل كده
context.go('/search?q=flutter&sort=recent');

// استخدم كده
context.goNamed(
  'search',
  queryParameters: {
    'q': 'flutter',
    'sort': 'recent',
  },
);
```

### مع Path و Query Parameters

```dart
context.goNamed(
  'user-orders',
  pathParameters: {'userId': '456'},
  queryParameters: {
    'status': 'pending',
    'page': '1',
  },
);
// النتيجة: /user/456/orders?status=pending&page=1
```

---

## pushNamed()

نفس الفكرة بس مع push بدل go:

```dart
// فتح صفحة التفاصيل
context.pushNamed(
  'product-details',
  pathParameters: {'id': productId},
);

// فتح صفحة البحث
context.pushNamed(
  'search',
  queryParameters: {'q': searchQuery},
);
```

---

## استخدام extra مع Named Routes

الـ `extra` بيسمحلك تمرر أي object:

```dart
// تمرير object كامل
context.goNamed(
  'product-details',
  pathParameters: {'id': '123'},
  extra: product,  // الـ Product object
);

// استقبال الـ extra
GoRoute(
  path: '/product/:id',
  name: 'product-details',
  builder: (context, state) {
    final product = state.extra as Product?;
    final id = state.pathParameters['id']!;

    // لو الـ extra موجود استخدمه، لو لأ اجلب البيانات
    if (product != null) {
      return ProductDetailsScreen(product: product);
    }

    return ProductDetailsScreen(id: id);
  },
)
```

---

## أفضل طريقة: Route Names كـ Constants

### إنشاء ملف للأسماء

📁 `lib/router/route_names.dart`

```dart
abstract class RouteNames {
  static const home = 'home';
  static const products = 'products';
  static const productDetails = 'product-details';
  static const cart = 'cart';
  static const checkout = 'checkout';
  static const userProfile = 'user-profile';
  static const userOrders = 'user-orders';
  static const settings = 'settings';
  static const search = 'search';
  static const login = 'login';
  static const register = 'register';
}
```

### استخدامها في الـ Router

```dart
import 'route_names.dart';

final appRouter = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      name: RouteNames.home,
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/products',
      name: RouteNames.products,
      builder: (context, state) => const ProductsScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      name: RouteNames.productDetails,
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return ProductDetailsScreen(id: id);
      },
    ),
    // ... باقي الـ routes
  ],
);
```

### استخدامها في الـ Navigation

```dart
import '../router/route_names.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () => context.goNamed(RouteNames.products),
            child: const Text('المنتجات'),
          ),
          ElevatedButton(
            onPressed: () => context.goNamed(
              RouteNames.productDetails,
              pathParameters: {'id': '123'},
            ),
            child: const Text('تفاصيل منتج'),
          ),
        ],
      ),
    );
  }
}
```

---

## طريقة أفضل: Navigation Helper Class

### إنشاء Helper Class

📁 `lib/router/app_navigator.dart`

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'route_names.dart';

class AppNavigator {
  // Home
  static void goHome(BuildContext context) {
    context.goNamed(RouteNames.home);
  }

  // Products
  static void goProducts(BuildContext context) {
    context.goNamed(RouteNames.products);
  }

  static void goProductDetails(
    BuildContext context, {
    required String productId,
    Product? product,
  }) {
    context.goNamed(
      RouteNames.productDetails,
      pathParameters: {'id': productId},
      extra: product,
    );
  }

  static void pushProductDetails(
    BuildContext context, {
    required String productId,
    Product? product,
  }) {
    context.pushNamed(
      RouteNames.productDetails,
      pathParameters: {'id': productId},
      extra: product,
    );
  }

  // Search
  static void goSearch(
    BuildContext context, {
    String? query,
    String? category,
    String sortBy = 'relevance',
  }) {
    context.goNamed(
      RouteNames.search,
      queryParameters: {
        if (query != null) 'q': query,
        if (category != null) 'category': category,
        'sort': sortBy,
      },
    );
  }

  // User
  static void goUserOrders(
    BuildContext context, {
    required String userId,
    String? status,
  }) {
    context.goNamed(
      RouteNames.userOrders,
      pathParameters: {'userId': userId},
      queryParameters: {
        if (status != null) 'status': status,
      },
    );
  }

  // Cart & Checkout
  static void goCart(BuildContext context) {
    context.goNamed(RouteNames.cart);
  }

  static void goCheckout(BuildContext context) {
    context.goNamed(RouteNames.checkout);
  }

  // Auth
  static void goLogin(BuildContext context) {
    context.goNamed(RouteNames.login);
  }

  static void goRegister(BuildContext context) {
    context.goNamed(RouteNames.register);
  }
}
```

### استخدام الـ Helper

```dart
import '../router/app_navigator.dart';

class ProductCard extends StatelessWidget {
  final Product product;

  const ProductCard({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        title: Text(product.name),
        subtitle: Text('${product.price} ج.م'),
        onTap: () {
          // استخدام الـ helper - واضح ونظيف
          AppNavigator.pushProductDetails(
            context,
            productId: product.id,
            product: product,
          );
        },
      ),
    );
  }
}

class SearchBar extends StatelessWidget {
  final TextEditingController controller;

  const SearchBar({super.key, required this.controller});

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: controller,
      onSubmitted: (query) {
        AppNavigator.goSearch(context, query: query);
      },
      decoration: InputDecoration(
        hintText: 'ابحث...',
        suffixIcon: IconButton(
          icon: const Icon(Icons.search),
          onPressed: () {
            AppNavigator.goSearch(context, query: controller.text);
          },
        ),
      ),
    );
  }
}
```

---

## الحصول على معلومات الـ Route الحالي

```dart
// الحصول على اسم الـ route الحالي
final currentRouteName = GoRouterState.of(context).name;
print(currentRouteName);  // 'product-details'

// التحقق من الـ route الحالي
if (currentRouteName == RouteNames.home) {
  // أنت في الصفحة الرئيسية
}
```

---

## نصائح مهمة

### 1. الأسماء لازم تكون Unique

```dart
// ❌ غلط - اسمين متشابهين
GoRoute(path: '/products', name: 'list', ...),
GoRoute(path: '/users', name: 'list', ...),

// ✅ صح - أسماء مميزة
GoRoute(path: '/products', name: 'products-list', ...),
GoRoute(path: '/users', name: 'users-list', ...),
```

### 2. استخدم naming convention واضح

```dart
// ✅ أسماء واضحة ومتسقة
name: 'user-profile'
name: 'user-orders'
name: 'product-details'
name: 'cart-items'

// ❌ أسماء غير متسقة
name: 'userProfile'
name: 'orders_list'
name: 'ProductDetails'
```

### 3. Path Parameters لازم تتطابق

```dart
// الـ Route
GoRoute(
  path: '/user/:userId/order/:orderId',
  name: 'order-details',
  ...
)

// ✅ صح - كل الـ parameters موجودة
context.goNamed(
  'order-details',
  pathParameters: {
    'userId': '123',
    'orderId': '456',
  },
);

// ❌ غلط - parameter ناقص
context.goNamed(
  'order-details',
  pathParameters: {'userId': '123'},  // orderId ناقص!
);
```

---

## مقارنة: Path vs Named Navigation

| الخاصية | Path-based | Named-based |
|---------|------------|-------------|
| **البساطة** | أبسط للـ routes البسيطة | أفضل للـ routes المعقدة |
| **الصيانة** | صعب لو غيرت الـ path | سهل - غير الـ path في مكان واحد |
| **Type Safety** | لا | نسبياً أفضل |
| **Autocomplete** | لا | نعم (مع constants) |

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **التعريف** | `name: 'route-name'` في GoRoute |
| **الـ Navigation** | `goNamed()`, `pushNamed()` |
| **Path Params** | `pathParameters: {'key': 'value'}` |
| **Query Params** | `queryParameters: {'key': 'value'}` |
| **Extra Data** | `extra: anyObject` |
| **Best Practice** | استخدم constants أو helper class |

---

[⬅️ الدرس السابق: Query Parameters](05_query_parameters.md) | [➡️ الدرس التالي: Sub-Routes](../02_go_router_intermediate/07_sub_routes.md)
