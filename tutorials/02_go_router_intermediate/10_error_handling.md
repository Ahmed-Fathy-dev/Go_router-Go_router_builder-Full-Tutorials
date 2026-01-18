# الدرس 10: Error Handling (معالجة الأخطاء)

## أنواع الأخطاء في GoRouter

| النوع | السبب | الحل |
|-------|-------|------|
| Route غير موجود | المستخدم راح لـ URL مش معرف | `errorBuilder` أو `onException` |
| Parameter غلط | الـ path parameter مش صالح | Validation + redirect |
| Exception أثناء الـ build | خطأ في الـ builder function | `onException` |

---

## errorBuilder

أبسط طريقة لعرض صفحة خطأ مخصصة:

```dart
final appRouter = GoRouter(
  routes: [...],

  errorBuilder: (context, state) {
    return Scaffold(
      appBar: AppBar(title: const Text('خطأ')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(
              Icons.error_outline,
              size: 64,
              color: Colors.red,
            ),
            const SizedBox(height: 16),
            Text(
              'الصفحة غير موجودة',
              style: Theme.of(context).textTheme.headlineSmall,
            ),
            const SizedBox(height: 8),
            Text(
              'المسار: ${state.uri}',
              style: Theme.of(context).textTheme.bodyMedium,
            ),
            const SizedBox(height: 24),
            ElevatedButton.icon(
              onPressed: () => context.go('/'),
              icon: const Icon(Icons.home),
              label: const Text('الرجوع للرئيسية'),
            ),
          ],
        ),
      ),
    );
  },
);
```

---

## errorPageBuilder

لو عايز تتحكم في الـ Page نفسها (للـ transitions مثلاً):

```dart
GoRouter(
  errorPageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: ErrorScreen(error: state.error),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return FadeTransition(
          opacity: animation,
          child: child,
        );
      },
    );
  },
  routes: [...],
)
```

---

## onException (الأقوى)

في الإصدارات الحديثة، `onException` بيوفرلك تحكم أكبر:

```dart
GoRouter(
  onException: (context, state, router) {
    // تسجيل الخطأ
    print('Navigation error: ${state.uri}');
    print('Exception: ${state.error}');

    // ممكن تعمل redirect
    router.go('/404', extra: state.uri.toString());
  },

  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/404',
      builder: (context, state) {
        final originalPath = state.extra as String?;
        return NotFoundScreen(attemptedPath: originalPath);
      },
    ),
  ],
)
```

### الفرق بين errorBuilder و onException

| الخاصية | errorBuilder | onException |
|---------|--------------|-------------|
| **الـ Return** | Widget | void |
| **التحكم** | بيعرض widget | بيديك الـ router |
| **Redirect** | لا | نعم |
| **الـ Context** | محدود | كامل |

---

## مثال شامل: صفحة 404 احترافية

### الـ Router

```dart
final appRouter = GoRouter(
  initialLocation: '/',

  onException: (context, state, router) {
    // Log the error
    debugPrint('404: ${state.uri}');

    // Redirect to 404 page with original path
    router.go(
      '/error',
      extra: ErrorDetails(
        path: state.uri.toString(),
        error: state.error?.toString(),
      ),
    );
  },

  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/products',
      builder: (context, state) => const ProductsScreen(),
    ),
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        // Validation
        if (int.tryParse(id) == null) {
          throw Exception('Invalid product ID: $id');
        }
        return ProductScreen(id: int.parse(id));
      },
    ),
    GoRoute(
      path: '/error',
      builder: (context, state) {
        final details = state.extra as ErrorDetails?;
        return ErrorScreen(details: details);
      },
    ),
  ],
);

class ErrorDetails {
  final String path;
  final String? error;

  ErrorDetails({required this.path, this.error});
}
```

### شاشة الخطأ

```dart
class ErrorScreen extends StatelessWidget {
  final ErrorDetails? details;

  const ErrorScreen({super.key, this.details});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              // Illustration
              Image.asset(
                'assets/images/404.png',
                height: 200,
                errorBuilder: (context, error, stackTrace) {
                  return const Icon(
                    Icons.error_outline,
                    size: 120,
                    color: Colors.grey,
                  );
                },
              ),

              const SizedBox(height: 32),

              // Title
              Text(
                'عذراً!',
                style: Theme.of(context).textTheme.headlineLarge?.copyWith(
                  fontWeight: FontWeight.bold,
                ),
              ),

              const SizedBox(height: 16),

              // Message
              Text(
                'الصفحة اللي بتدور عليها مش موجودة',
                style: Theme.of(context).textTheme.bodyLarge,
                textAlign: TextAlign.center,
              ),

              if (details != null) ...[
                const SizedBox(height: 16),
                Container(
                  padding: const EdgeInsets.all(12),
                  decoration: BoxDecoration(
                    color: Colors.grey.shade100,
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Text(
                    details!.path,
                    style: TextStyle(
                      fontFamily: 'monospace',
                      color: Colors.grey.shade700,
                    ),
                  ),
                ),
              ],

              const SizedBox(height: 32),

              // Actions
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  OutlinedButton.icon(
                    onPressed: () {
                      if (context.canPop()) {
                        context.pop();
                      } else {
                        context.go('/');
                      }
                    },
                    icon: const Icon(Icons.arrow_back),
                    label: const Text('رجوع'),
                  ),
                  const SizedBox(width: 16),
                  ElevatedButton.icon(
                    onPressed: () => context.go('/'),
                    icon: const Icon(Icons.home),
                    label: const Text('الرئيسية'),
                  ),
                ],
              ),

              const SizedBox(height: 24),

              // Search suggestion
              Text(
                'جرب تبحث عن اللي بتدور عليه:',
                style: Theme.of(context).textTheme.bodyMedium,
              ),
              const SizedBox(height: 12),
              SizedBox(
                width: 300,
                child: TextField(
                  decoration: InputDecoration(
                    hintText: 'ابحث...',
                    prefixIcon: const Icon(Icons.search),
                    border: OutlineInputBorder(
                      borderRadius: BorderRadius.circular(30),
                    ),
                  ),
                  onSubmitted: (query) {
                    context.go('/search?q=$query');
                  },
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## التعامل مع أخطاء الـ Parameters

### Validation في الـ redirect

```dart
GoRoute(
  path: '/user/:id',
  redirect: (context, state) {
    final id = state.pathParameters['id'];

    // تحقق من صحة الـ ID
    if (id == null || int.tryParse(id) == null) {
      return '/error?message=Invalid user ID';
    }

    // تحقق من وجود الـ user (sync فقط)
    // لو محتاج async، استخدم approach تاني

    return null;
  },
  builder: (context, state) {
    final id = int.parse(state.pathParameters['id']!);
    return UserScreen(id: id);
  },
)
```

### Validation في الـ builder

```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final idString = state.pathParameters['id'];
    final id = int.tryParse(idString ?? '');

    if (id == null) {
      return const ErrorScreen(
        message: 'Invalid product ID',
      );
    }

    // تحقق إضافي
    if (id < 0) {
      return const ErrorScreen(
        message: 'Product ID must be positive',
      );
    }

    return ProductScreen(id: id);
  },
)
```

---

## FutureBuilder للـ Async Validation

```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = state.pathParameters['id']!;

    return FutureBuilder<Product?>(
      future: ProductService.getProduct(id),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }

        if (snapshot.hasError) {
          return ErrorScreen(
            message: 'Error loading product: ${snapshot.error}',
          );
        }

        final product = snapshot.data;
        if (product == null) {
          return const ErrorScreen(
            message: 'Product not found',
          );
        }

        return ProductScreen(product: product);
      },
    );
  },
)
```

---

## Custom Error Widget مع Animation

```dart
class AnimatedErrorScreen extends StatefulWidget {
  final String? path;

  const AnimatedErrorScreen({super.key, this.path});

  @override
  State<AnimatedErrorScreen> createState() => _AnimatedErrorScreenState();
}

class _AnimatedErrorScreenState extends State<AnimatedErrorScreen>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  late Animation<double> _fadeAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 600),
      vsync: this,
    );

    _scaleAnimation = Tween<double>(begin: 0.5, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.elasticOut),
    );

    _fadeAnimation = Tween<double>(begin: 0.0, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeIn),
    );

    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: FadeTransition(
          opacity: _fadeAnimation,
          child: ScaleTransition(
            scale: _scaleAnimation,
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Text(
                  '404',
                  style: TextStyle(
                    fontSize: 100,
                    fontWeight: FontWeight.bold,
                    color: Colors.grey,
                  ),
                ),
                const SizedBox(height: 16),
                const Text(
                  'الصفحة غير موجودة',
                  style: TextStyle(fontSize: 24),
                ),
                const SizedBox(height: 32),
                ElevatedButton(
                  onPressed: () => context.go('/'),
                  child: const Text('الرجوع للرئيسية'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## Logging الأخطاء

```dart
GoRouter(
  debugLogDiagnostics: true,  // في الـ development

  onException: (context, state, router) {
    // Log to analytics
    Analytics.logEvent(
      name: 'navigation_error',
      parameters: {
        'path': state.uri.toString(),
        'error': state.error?.toString() ?? 'Unknown',
      },
    );

    // Log to crash reporting
    Crashlytics.recordError(
      state.error ?? Exception('Unknown navigation error'),
      StackTrace.current,
      reason: 'Navigation to ${state.uri} failed',
    );

    router.go('/error');
  },

  routes: [...],
)
```

---

## ملخص

| الخاصية | الاستخدام |
|---------|-----------|
| **errorBuilder** | عرض widget بسيط للخطأ |
| **errorPageBuilder** | تحكم في الـ Page والـ transitions |
| **onException** | تحكم كامل + redirect + logging |
| **Validation** | في redirect أو builder |
| **Async** | استخدم FutureBuilder |

---

[⬅️ الدرس السابق: Async Redirection](09_async_redirection.md) | [➡️ الدرس التالي: ShellRoute](../03_go_router_advanced/11_shell_routes.md)
