# FAQ - أسئلة شائعة

## أساسيات

### س: ما الفرق بين `go()` و `push()`؟

**`go()`**: بيمسح الـ navigation stack ويروح للصفحة المطلوبة.
```dart
// Stack قبل: [Home, Products, Details]
context.go('/settings');
// Stack بعد: [Settings]
```

**`push()`**: بيضيف الصفحة فوق الـ stack الحالي.
```dart
// Stack قبل: [Home, Products]
context.push('/details');
// Stack بعد: [Home, Products, Details]
```

---

### س: متى أستخدم GoRouter Builder؟

استخدمه لو:
- عندك routes كتير
- محتاج type safety
- فريق كبير
- مشروع متوسط أو كبير

ممكن تستغنى عنه لو:
- مشروع صغير (3-4 routes)
- prototype سريع

---

### س: هل GoRouter بيشتغل على الويب؟

**نعم!** GoRouter مصمم للويب:
- الـ URL بيتغير في المتصفح
- الـ browser back/forward بيشتغلوا
- Deep linking مدعوم

---

## Navigation

### س: إزاي أرجع قيمة من صفحة؟

```dart
// في الصفحة الأصلية
final result = await context.push<bool>('/confirm');
if (result == true) {
  // تم التأكيد
}

// في صفحة التأكيد
context.pop(true);   // تأكيد
context.pop(false);  // إلغاء
```

---

### س: إزاي أمنع المستخدم من الرجوع؟

```dart
GoRoute(
  path: '/form',
  onExit: (context, state) async {
    if (hasUnsavedChanges) {
      return await showDialog<bool>(
        context: context,
        builder: (context) => AlertDialog(
          title: Text('تأكيد'),
          content: Text('فيه تغييرات مش محفوظة'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context, false),
              child: Text('ابقى'),
            ),
            TextButton(
              onPressed: () => Navigator.pop(context, true),
              child: Text('اخرج'),
            ),
          ],
        ),
      ) ?? false;
    }
    return true;
  },
)
```

---

### س: إزاي أروح لصفحة خارج الـ Shell؟

```dart
final _rootNavigatorKey = GlobalKey<NavigatorState>();

GoRouter(
  navigatorKey: _rootNavigatorKey,
  routes: [
    StatefulShellRoute.indexedStack(...),

    // Route خارج الـ Shell
    GoRoute(
      path: '/full-screen',
      parentNavigatorKey: _rootNavigatorKey,
      builder: (context, state) => FullScreenPage(),
    ),
  ],
)
```

---

## Parameters

### س: إيه الفرق بين Path و Query Parameters؟

**Path Parameters**: جزء أساسي من الـ URL
```
/product/123
/user/ahmed/posts
```

**Query Parameters**: اختيارية بعد `?`
```
/products?category=phones&sort=price
/search?q=flutter
```

---

### س: إزاي أمرر object كامل؟

```dart
// باستخدام extra
context.go('/product/123', extra: productObject);

// استقباله
final product = state.extra as Product?;
```

---

## Authentication

### س: إزاي أحمي الصفحات؟

```dart
GoRouter(
  refreshListenable: authNotifier,  // للتحديث التلقائي
  redirect: (context, state) {
    final isLoggedIn = AuthService.isLoggedIn;
    final isAuthRoute = state.uri.path == '/login';

    if (!isLoggedIn && !isAuthRoute) {
      return '/login?redirect=${state.uri}';
    }

    if (isLoggedIn && isAuthRoute) {
      return '/';
    }

    return null;
  },
)
```

---

### س: إزاي أرجع المستخدم للصفحة اللي كان فيها بعد Login؟

```dart
// في redirect
if (!isLoggedIn) {
  return '/login?redirect=${Uri.encodeComponent(state.uri.toString())}';
}

// في Login Screen
class LoginScreen extends StatelessWidget {
  final String? redirectTo;

  void onLoginSuccess(BuildContext context) {
    if (redirectTo != null) {
      context.go(redirectTo!);
    } else {
      context.go('/');
    }
  }
}
```

---

## Performance

### س: ليه الـ tabs بتتحمل من الأول كل مرة؟

استخدم `StatefulShellRoute` بدل `ShellRoute`:

```dart
StatefulShellRoute.indexedStack(
  branches: [
    StatefulShellBranch(routes: [...]),
    StatefulShellBranch(routes: [...]),
  ],
)
```

---

### س: إزاي أحسن الأداء؟

1. استخدم `StatefulShellRoute` للـ tabs
2. استخدم `const` constructors
3. استخدم `extra` بدل API calls
4. استخدم `NoTransitionPage` للـ instant switching

---

## Troubleshooting

### س: الـ build_runner مش بيولد الملفات؟

```bash
# امسح الـ cache
dart run build_runner clean

# شغل تاني
dart run build_runner build --delete-conflicting-outputs
```

---

### س: إزاي أعرف الـ location الحالي؟

```dart
// الـ URI كامل
final uri = GoRouterState.of(context).uri;
print(uri.path);           // /products/123
print(uri.queryParameters); // {sort: price}

// أو
final router = GoRouter.of(context);
print(router.state.uri);
```

---

### س: إزاي أعمل debug للـ navigation؟

```dart
GoRouter(
  debugLogDiagnostics: true,  // هيطبع كل الـ navigation
  routes: [...],
)
```

---

## المزيد

### س: وين الـ documentation الرسمي؟

- [pub.dev/packages/go_router](https://pub.dev/packages/go_router)
- [pub.dev/documentation/go_router](https://pub.dev/documentation/go_router/latest/)
- [GitHub Repository](https://github.com/flutter/packages/tree/main/packages/go_router)

---

### س: هل فيه أمثلة رسمية؟

نعم! في الـ [GitHub Examples](https://github.com/flutter/packages/tree/main/packages/go_router/example/lib):
- async_redirection.dart
- stateful_shell_route.dart
- transition_animations.dart
- و كتير غيرهم
