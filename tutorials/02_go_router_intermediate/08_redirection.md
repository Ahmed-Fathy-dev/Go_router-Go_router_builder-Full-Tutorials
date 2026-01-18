# الدرس 8: Redirection (إعادة التوجيه)

## يعني إيه Redirection؟

الـ Redirection هي إعادة توجيه المستخدم لصفحة تانية بناءً على شرط معين. أشهر استخدام ليها هو حماية الصفحات من الـ users الغير مسجلين.

---

## أنواع الـ Redirection

### 1. Top-level Redirect (عام)
بيتنفذ على **كل** عمليات الـ navigation:

```dart
GoRouter(
  redirect: (context, state) {
    // بيتنفذ قبل أي navigation
  },
  routes: [...],
)
```

### 2. Route-level Redirect (خاص)
بيتنفذ بس لما تروح للـ route ده:

```dart
GoRoute(
  path: '/admin',
  redirect: (context, state) {
    // بيتنفذ بس لما حد يحاول يدخل /admin
  },
  builder: (context, state) => const AdminScreen(),
)
```

---

## Top-level Redirect

### مثال: حماية التطبيق كله

```dart
final appRouter = GoRouter(
  initialLocation: '/',

  redirect: (BuildContext context, GoRouterState state) {
    // التحقق من حالة تسجيل الدخول
    final isLoggedIn = AuthService.instance.isLoggedIn;

    // الصفحات اللي مش محتاجة login
    final publicPaths = ['/login', '/register', '/forgot-password'];
    final isPublicPath = publicPaths.contains(state.uri.path);

    // لو مش مسجل دخول وبيحاول يدخل صفحة محمية
    if (!isLoggedIn && !isPublicPath) {
      // احفظ الصفحة اللي كان عايز يروحها
      return '/login?redirect=${state.uri.path}';
    }

    // لو مسجل دخول وبيحاول يدخل Login
    if (isLoggedIn && state.uri.path == '/login') {
      return '/';  // وديه للـ Home
    }

    // متعملش redirect
    return null;
  },

  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfileScreen(),
    ),
  ],
);
```

---

## Route-level Redirect

### مثال: حماية صفحة معينة

```dart
GoRoute(
  path: '/admin',
  redirect: (context, state) {
    final user = AuthService.instance.currentUser;

    // لو مش admin، ارجعه للـ Home
    if (user == null || !user.isAdmin) {
      return '/';
    }

    return null;
  },
  builder: (context, state) => const AdminDashboard(),
)
```

### مثال: التحقق من صلاحيات مختلفة

```dart
GoRoute(
  path: '/admin',
  builder: (context, state) => const AdminDashboard(),
  routes: [
    GoRoute(
      path: 'users',
      redirect: (context, state) {
        if (!hasPermission('manage_users')) {
          return '/admin';  // ارجعه للـ dashboard
        }
        return null;
      },
      builder: (context, state) => const ManageUsersScreen(),
    ),
    GoRoute(
      path: 'settings',
      redirect: (context, state) {
        if (!hasPermission('admin_settings')) {
          return '/admin';
        }
        return null;
      },
      builder: (context, state) => const AdminSettingsScreen(),
    ),
  ],
)
```

---

## refreshListenable

لما حالة الـ authentication تتغير، محتاج الـ router يعيد تقييم الـ redirect. استخدم `refreshListenable`:

```dart
// الـ Auth Notifier
class AuthNotifier extends ChangeNotifier {
  bool _isLoggedIn = false;

  bool get isLoggedIn => _isLoggedIn;

  void login() {
    _isLoggedIn = true;
    notifyListeners();  // 👈 ده بيخلي الـ router يعيد التقييم
  }

  void logout() {
    _isLoggedIn = false;
    notifyListeners();
  }
}

// إنشاء instance
final authNotifier = AuthNotifier();

// الـ Router
final appRouter = GoRouter(
  refreshListenable: authNotifier,  // 👈 ربط الـ notifier

  redirect: (context, state) {
    final isLoggedIn = authNotifier.isLoggedIn;

    if (!isLoggedIn && state.uri.path != '/login') {
      return '/login';
    }

    if (isLoggedIn && state.uri.path == '/login') {
      return '/';
    }

    return null;
  },

  routes: [...],
);
```

### مع Riverpod

```dart
final authProvider = StateNotifierProvider<AuthNotifier, AuthState>((ref) {
  return AuthNotifier();
});

// في الـ router
GoRouter(
  refreshListenable: GoRouterRefreshStream(
    ref.watch(authProvider.notifier).stream,
  ),
  redirect: (context, state) {
    final authState = ref.read(authProvider);
    // ...
  },
  routes: [...],
)
```

---

## مثال كامل: Authentication Flow

```dart
// lib/services/auth_service.dart
class AuthService extends ChangeNotifier {
  static final AuthService instance = AuthService._();
  AuthService._();

  User? _currentUser;

  bool get isLoggedIn => _currentUser != null;
  User? get currentUser => _currentUser;

  Future<void> login(String email, String password) async {
    // محاكاة API call
    await Future.delayed(const Duration(seconds: 1));

    _currentUser = User(
      id: '1',
      email: email,
      name: 'أحمد',
      role: email.contains('admin') ? 'admin' : 'user',
    );

    notifyListeners();
  }

  Future<void> logout() async {
    _currentUser = null;
    notifyListeners();
  }
}

class User {
  final String id;
  final String email;
  final String name;
  final String role;

  User({
    required this.id,
    required this.email,
    required this.name,
    required this.role,
  });

  bool get isAdmin => role == 'admin';
}
```

```dart
// lib/router/app_router.dart
import 'package:go_router/go_router.dart';
import '../services/auth_service.dart';

final appRouter = GoRouter(
  initialLocation: '/',
  debugLogDiagnostics: true,
  refreshListenable: AuthService.instance,

  redirect: (context, state) {
    final auth = AuthService.instance;
    final isLoggedIn = auth.isLoggedIn;
    final isLoggingIn = state.uri.path == '/login';
    final isRegistering = state.uri.path == '/register';

    // الصفحات العامة
    if (!isLoggedIn && !isLoggingIn && !isRegistering) {
      // احفظ الـ path عشان نرجعله بعد Login
      final redirectTo = state.uri.toString();
      if (redirectTo != '/') {
        return '/login?from=$redirectTo';
      }
      return '/login';
    }

    // لو مسجل دخول ويحاول يدخل Login/Register
    if (isLoggedIn && (isLoggingIn || isRegistering)) {
      // شوف لو فيه redirect path
      final from = state.uri.queryParameters['from'];
      return from ?? '/';
    }

    return null;
  },

  routes: [
    // Home
    GoRoute(
      path: '/',
      name: 'home',
      builder: (context, state) => const HomeScreen(),
    ),

    // Auth Routes
    GoRoute(
      path: '/login',
      name: 'login',
      builder: (context, state) {
        final from = state.uri.queryParameters['from'];
        return LoginScreen(redirectTo: from);
      },
    ),
    GoRoute(
      path: '/register',
      name: 'register',
      builder: (context, state) => const RegisterScreen(),
    ),

    // Profile (محمي)
    GoRoute(
      path: '/profile',
      name: 'profile',
      builder: (context, state) => const ProfileScreen(),
    ),

    // Admin (محمي + يحتاج صلاحية)
    GoRoute(
      path: '/admin',
      name: 'admin',
      redirect: (context, state) {
        final user = AuthService.instance.currentUser;
        if (user == null || !user.isAdmin) {
          return '/';  // مش admin، ارجعه للـ Home
        }
        return null;
      },
      builder: (context, state) => const AdminScreen(),
    ),
  ],
);
```

```dart
// lib/screens/login_screen.dart
class LoginScreen extends StatefulWidget {
  final String? redirectTo;

  const LoginScreen({super.key, this.redirectTo});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _isLoading = false;

  Future<void> _login() async {
    setState(() => _isLoading = true);

    try {
      await AuthService.instance.login(
        _emailController.text,
        _passwordController.text,
      );

      if (mounted) {
        // الـ router هيعمل redirect تلقائي بسبب refreshListenable
        // بس لو فيه path معين، روحله
        if (widget.redirectTo != null) {
          context.go(widget.redirectTo!);
        }
      }
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('خطأ: $e')),
        );
      }
    } finally {
      if (mounted) {
        setState(() => _isLoading = false);
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('تسجيل الدخول')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _emailController,
              decoration: const InputDecoration(
                labelText: 'البريد الإلكتروني',
                prefixIcon: Icon(Icons.email),
              ),
            ),
            const SizedBox(height: 16),
            TextField(
              controller: _passwordController,
              obscureText: true,
              decoration: const InputDecoration(
                labelText: 'كلمة المرور',
                prefixIcon: Icon(Icons.lock),
              ),
            ),
            const SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: _isLoading ? null : _login,
                child: _isLoading
                    ? const CircularProgressIndicator()
                    : const Text('دخول'),
              ),
            ),
            const SizedBox(height: 16),
            TextButton(
              onPressed: () => context.go('/register'),
              child: const Text('إنشاء حساب جديد'),
            ),
          ],
        ),
      ),
    );
  }
}
```

---

## redirectLimit

لو حصل infinite redirect loop، الـ GoRouter بيوقف بعد عدد معين (default: 5):

```dart
GoRouter(
  redirectLimit: 10,  // زود الحد لو محتاج
  redirect: (context, state) {
    // ...
  },
  routes: [...],
)
```

> ⚠️ لو وصلت للـ limit، هيظهر error screen

---

## ترتيب تنفيذ الـ Redirects

1. **Top-level redirect** (الأول)
2. **Route-level redirect** للـ route المطلوب
3. **Route-level redirect** للـ sub-routes

```dart
GoRouter(
  redirect: (context, state) {
    print('1. Top-level');
    return null;
  },
  routes: [
    GoRoute(
      path: '/parent',
      redirect: (context, state) {
        print('2. Parent route');
        return null;
      },
      routes: [
        GoRoute(
          path: 'child',
          redirect: (context, state) {
            print('3. Child route');
            return null;
          },
          builder: (context, state) => const ChildScreen(),
        ),
      ],
    ),
  ],
)

// لما تروح لـ /parent/child:
// Output:
// 1. Top-level
// 2. Parent route
// 3. Child route
```

---

## نصائح مهمة

### 1. متعملش redirect loop

```dart
// ❌ Infinite loop!
redirect: (context, state) {
  if (state.uri.path == '/a') return '/b';
  if (state.uri.path == '/b') return '/a';
  return null;
}
```

### 2. تحقق من الـ path قبل ما تعمل redirect

```dart
// ✅ تجنب redirect للنفس الصفحة
redirect: (context, state) {
  if (!isLoggedIn && state.uri.path != '/login') {
    return '/login';
  }
  return null;
}
```

### 3. استخدم refreshListenable

```dart
// ✅ عشان الـ router يعرف لما الـ auth state تتغير
GoRouter(
  refreshListenable: authNotifier,
  redirect: ...
)
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **Top-level** | بيتنفذ على كل navigation |
| **Route-level** | بيتنفذ على route معين |
| **Return value** | `null` = متعملش redirect، `String` = الـ path الجديد |
| **refreshListenable** | لتحديث الـ redirect لما الحالة تتغير |
| **redirectLimit** | الحد الأقصى للـ redirects (default: 5) |

---

[⬅️ الدرس السابق: Sub-Routes](07_sub_routes.md) | [➡️ الدرس التالي: Async Redirection](09_async_redirection.md)
