# الدرس 15: Extra Data & Codec + onEnter

## يعني إيه Extra؟

الـ `extra` هو طريقة لتمرير أي object للصفحة التالية بدون ما يظهر في الـ URL.

```dart
// Pass object
context.go('/product/123', extra: productObject);

// Receive it
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final product = state.extra as Product?;
    final id = state.pathParameters['id']!;

    // If extra exists, use it
    // If not, fetch data from API
    return ProductScreen(
      product: product,
      id: id,
    );
  },
)
```

---

## استخدامات الـ Extra

| الحالة | استخدم extra |
|--------|--------------|
| عندك الـ object جاهز | ✅ |
| تجنب API call إضافي | ✅ |
| بيانات كبيرة أو معقدة | ✅ |
| محتاج الـ data تتحفظ في URL | ❌ |
| Deep linking | ❌ |

---

## المشكلة: الـ Extra والـ Web

على الويب، لما المستخدم يعمل refresh أو يستخدم browser back/forward، الـ `extra` بيضيع لأنه مش جزء من الـ URL.

### الحل: extraCodec

الـ `extraCodec` بيسمحلك تحول الـ extra لـ format قابل للتخزين والاسترجاع.

```dart
import 'dart:convert';

class MyExtraCodec extends Codec<Object?, Object?> {
  const MyExtraCodec();

  @override
  Converter<Object?, Object?> get decoder => const _Decoder();

  @override
  Converter<Object?, Object?> get encoder => const _Encoder();
}

class _Encoder extends Converter<Object?, Object?> {
  const _Encoder();

  @override
  Object? convert(Object? input) {
    if (input is Product) {
      return ['Product', input.toJson()];
    }
    if (input is User) {
      return ['User', input.toJson()];
    }
    return input;
  }
}

class _Decoder extends Converter<Object?, Object?> {
  const _Decoder();

  @override
  Object? convert(Object? input) {
    if (input is List && input.length == 2) {
      final type = input[0] as String;
      final data = input[1];

      switch (type) {
        case 'Product':
          return Product.fromJson(data as Map<String, dynamic>);
        case 'User':
          return User.fromJson(data as Map<String, dynamic>);
      }
    }
    return input;
  }
}
```

### استخدامه

```dart
final appRouter = GoRouter(
  extraCodec: const MyExtraCodec(),
  routes: [...],
);
```

---

## مثال كامل للـ extraCodec

```dart
// Models
class Product {
  final String id;
  final String name;
  final double price;

  Product({required this.id, required this.name, required this.price});

  Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'price': price,
  };

  factory Product.fromJson(Map<String, dynamic> json) => Product(
    id: json['id'] as String,
    name: json['name'] as String,
    price: (json['price'] as num).toDouble(),
  );
}

class CartItem {
  final Product product;
  final int quantity;

  CartItem({required this.product, required this.quantity});

  Map<String, dynamic> toJson() => {
    'product': product.toJson(),
    'quantity': quantity,
  };

  factory CartItem.fromJson(Map<String, dynamic> json) => CartItem(
    product: Product.fromJson(json['product'] as Map<String, dynamic>),
    quantity: json['quantity'] as int,
  );
}

// Codec
class AppExtraCodec extends Codec<Object?, Object?> {
  const AppExtraCodec();

  @override
  Converter<Object?, Object?> get decoder => const _AppDecoder();

  @override
  Converter<Object?, Object?> get encoder => const _AppEncoder();
}

class _AppEncoder extends Converter<Object?, Object?> {
  const _AppEncoder();

  @override
  Object? convert(Object? input) {
    if (input == null) return null;

    if (input is Product) {
      return {'__type': 'Product', 'data': input.toJson()};
    }
    if (input is CartItem) {
      return {'__type': 'CartItem', 'data': input.toJson()};
    }
    if (input is List<CartItem>) {
      return {
        '__type': 'List<CartItem>',
        'data': input.map((e) => e.toJson()).toList(),
      };
    }

    // For simple types
    return input;
  }
}

class _AppDecoder extends Converter<Object?, Object?> {
  const _AppDecoder();

  @override
  Object? convert(Object? input) {
    if (input == null) return null;

    if (input is Map<String, dynamic> && input.containsKey('__type')) {
      final type = input['__type'] as String;
      final data = input['data'];

      switch (type) {
        case 'Product':
          return Product.fromJson(data as Map<String, dynamic>);
        case 'CartItem':
          return CartItem.fromJson(data as Map<String, dynamic>);
        case 'List<CartItem>':
          return (data as List)
              .map((e) => CartItem.fromJson(e as Map<String, dynamic>))
              .toList();
      }
    }

    return input;
  }
}

// The Router
final appRouter = GoRouter(
  extraCodec: const AppExtraCodec(),
  routes: [
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final product = state.extra as Product?;
        return ProductScreen(
          id: state.pathParameters['id']!,
          product: product,
        );
      },
    ),
    GoRoute(
      path: '/checkout',
      builder: (context, state) {
        final items = state.extra as List<CartItem>?;
        return CheckoutScreen(items: items ?? []);
      },
    ),
  ],
);
```

---

## onEnter Callback (v16.3+)

الـ `onEnter` هو callback جديد بيتنفذ **قبل** ما الصفحة تتعرض، وبيديك تحكم أكبر من `redirect`.

### الـ Syntax

```dart
GoRouter(
  onEnter: (BuildContext context, GoRouterState state) {
    // Return:
    // - EnterResult.allow() -> اسمح بالـ navigation
    // - EnterResult.block() -> امنع الـ navigation
    // - EnterResult.blockThen(() {}) -> امنع وقم بعمل حاجة

    return EnterResult.allow();
  },
  routes: [...],
);
```

---

## استخدامات onEnter

### 1. تتبع الـ Analytics

```dart
GoRouter(
  onEnter: (context, state) {
    // Track the page view
    Analytics.logPageView(
      pageName: state.name ?? state.uri.path,
      parameters: state.pathParameters,
    );

    return EnterResult.allow();
  },
  routes: [...],
)
```

### 2. معالجة Referral Codes

```dart
GoRouter(
  onEnter: (context, state) {
    final ref = state.uri.queryParameters['ref'];

    if (ref != null) {
      // Process the referral in background
      ReferralService.processCode(ref);
    }

    return EnterResult.allow();
  },
  routes: [...],
)
```

### 3. منع Navigation معين

```dart
GoRouter(
  onEnter: (context, state) {
    // Block entry during save operation
    if (SavingState.isSaving) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('جاري الحفظ...')),
      );
      return EnterResult.block();
    }

    return EnterResult.allow();
  },
  routes: [...],
)
```

### 4. Deep Link Processing

```dart
GoRouter(
  onEnter: (context, state) {
    // Handle the OAuth callback
    if (state.uri.path == '/oauth/callback') {
      final code = state.uri.queryParameters['code'];

      if (code != null) {
        // Process in background and redirect to home
        AuthService.processOAuthCode(code);

        return EnterResult.blockThen(() {
          GoRouter.of(context).go('/');
        });
      }
    }

    return EnterResult.allow();
  },
  routes: [...],
)
```

---

## onEnter vs redirect

| الخاصية | onEnter | redirect |
|---------|---------|----------|
| **التوقيت** | قبل redirect | بعد onEnter |
| **الوظيفة الرئيسية** | Side effects, blocking | تغيير المسار |
| **Return type** | EnterResult | String? |
| **Block بدون redirect** | ✅ نعم | ❌ لا |

### ترتيب التنفيذ:
1. `onEnter` (top-level)
2. `redirect` (top-level)
3. `redirect` (route-level)
4. `builder`

---

## onExit Callback

مشابه لـ `onEnter` بس بيتنفذ لما المستخدم يحاول يسيب الصفحة:

```dart
GoRoute(
  path: '/edit/:id',
  onExit: (context, state) async {
    // If there are unsaved changes
    if (hasUnsavedChanges) {
      final shouldLeave = await showDialog<bool>(
        context: context,
        builder: (context) => AlertDialog(
          title: const Text('تنبيه'),
          content: const Text('فيه تغييرات مش محفوظة. عايز تخرج؟'),
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
      );

      return shouldLeave ?? false;
    }

    return true;  // Allow exit
  },
  builder: (context, state) => EditScreen(id: state.pathParameters['id']!),
)
```

---

## مثال شامل

```dart
final appRouter = GoRouter(
  extraCodec: const AppExtraCodec(),

  // onEnter for analytics and deep link processing
  onEnter: (context, state) {
    // Log the navigation
    debugPrint('Entering: ${state.uri}');

    // Handle special deep links
    if (state.uri.path == '/verify-email') {
      final token = state.uri.queryParameters['token'];
      if (token != null) {
        // Process the verification
        AuthService.verifyEmail(token);

        // Show message and redirect
        WidgetsBinding.instance.addPostFrameCallback((_) {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(content: Text('تم تفعيل البريد!')),
          );
        });

        return EnterResult.blockThen(() {
          GoRouter.of(context).go('/');
        });
      }
    }

    return EnterResult.allow();
  },

  redirect: (context, state) {
    // Normal authentication redirect
    final isLoggedIn = AuthService.isLoggedIn;
    if (!isLoggedIn && state.uri.path.startsWith('/profile')) {
      return '/login';
    }
    return null;
  },

  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final product = state.extra as Product?;
        return ProductScreen(
          id: state.pathParameters['id']!,
          product: product,
        );
      },
    ),
    GoRoute(
      path: '/edit/:id',
      onExit: (context, state) async {
        // Confirm before leaving the page
        return await showExitConfirmDialog(context);
      },
      builder: (context, state) => EditScreen(
        id: state.pathParameters['id']!,
      ),
    ),
  ],
);
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **extra** | تمرير أي object للصفحة |
| **extraCodec** | لحفظ الـ extra على الويب |
| **onEnter** | callback قبل عرض الصفحة |
| **onExit** | callback قبل مغادرة الصفحة |
| **EnterResult** | allow, block, blockThen |

---

[⬅️ الدرس السابق: Deep Linking](14_deep_linking.md) | [➡️ الدرس التالي: مقدمة للـ Builder](../04_go_router_builder/16_builder_introduction.md)
