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
    // Executes before any navigation
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
    // Executes only when someone tries to access /admin
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
    // Check login status
    final isLoggedIn = AuthService.instance.isLoggedIn;

    // Pages that don't require login
    final publicPaths = ['/login', '/register', '/forgot-password'];
    final isPublicPath = publicPaths.contains(state.uri.path);

    // If not logged in and trying to access a protected page
    if (!isLoggedIn && !isPublicPath) {
      // Save the page they wanted to go to
      return '/login?redirect=${state.uri.path}';
    }

    // If logged in and trying to access Login
    if (isLoggedIn && state.uri.path == '/login') {
      return '/';  // Send to Home
    }

    // Don't redirect
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

    // If not admin, redirect to Home
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
          return '/admin';  // Redirect to dashboard
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
// The Auth Notifier
class AuthNotifier extends ChangeNotifier {
  bool _isLoggedIn = false;

  bool get isLoggedIn => _isLoggedIn;

  void login() {
    _isLoggedIn = true;
    notifyListeners();  // 👈 This makes the router re-evaluate
  }

  void logout() {
    _isLoggedIn = false;
    notifyListeners();
  }
}

// Create instance
final authNotifier = AuthNotifier();

// The Router
final appRouter = GoRouter(
  refreshListenable: authNotifier,  // 👈 Connect the notifier

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

### مع Riverpod 3+

```dart
// lib/features/auth/providers/auth_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'auth_provider.g.dart';

/// Auth state
class AuthState {
  const AuthState({
    this.user,
    this.isLoading = false,
  });

  final User? user;
  final bool isLoading;

  bool get isAuthenticated => user != null;

  AuthState copyWith({User? user, bool? isLoading}) {
    return AuthState(
      user: user ?? this.user,
      isLoading: isLoading ?? this.isLoading,
    );
  }
}

/// Auth notifier using Riverpod 3+ syntax
@riverpod
class Auth extends _$Auth {
  @override
  AuthState build() => const AuthState();

  Future<void> login(String email, String password) async {
    state = state.copyWith(isLoading: true);
    try {
      final user = await AuthRepository.login(email, password);
      state = AuthState(user: user);
    } finally {
      state = state.copyWith(isLoading: false);
    }
  }

  void logout() {
    state = const AuthState();
  }
}

/// Listenable for GoRouter refresh
@riverpod
Listenable authRefreshListenable(Ref ref) {
  return _AuthRefreshNotifier(ref);
}

class _AuthRefreshNotifier extends ChangeNotifier {
  _AuthRefreshNotifier(this._ref) {
    _ref.listen(authProvider, (_, __) => notifyListeners());
  }

  final Ref _ref;
}
```

```dart
// lib/core/router/app_router.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:go_router/go_router.dart';

final routerProvider = Provider<GoRouter>((ref) {
  final authState = ref.watch(authProvider);
  final refreshListenable = ref.watch(authRefreshListenableProvider);

  return GoRouter(
    refreshListenable: refreshListenable,
    redirect: (context, state) {
      final isAuthenticated = authState.isAuthenticated;
      final isAuthRoute = state.uri.path == '/login';

      if (!isAuthenticated && !isAuthRoute) {
        return '/login?redirect=${state.uri}';
      }

      if (isAuthenticated && isAuthRoute) {
        return state.uri.queryParameters['redirect'] ?? '/';
      }

      return null;
    },
    routes: [...],
  );
});
```

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends ConsumerWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final router = ref.watch(routerProvider);

    return MaterialApp.router(
      routerConfig: router,
      // ...
    );
  }
}
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
    // Simulate API call
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

    // Public pages
    if (!isLoggedIn && !isLoggingIn && !isRegistering) {
      // Save the path to return to after Login
      final redirectTo = state.uri.toString();
      if (redirectTo != '/') {
        return '/login?from=$redirectTo';
      }
      return '/login';
    }

    // If logged in and trying to access Login/Register
    if (isLoggedIn && (isLoggingIn || isRegistering)) {
      // Check if there's a redirect path
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

    // Profile (protected)
    GoRoute(
      path: '/profile',
      name: 'profile',
      builder: (context, state) => const ProfileScreen(),
    ),

    // Admin (protected + requires permission)
    GoRoute(
      path: '/admin',
      name: 'admin',
      redirect: (context, state) {
        final user = AuthService.instance.currentUser;
        if (user == null || !user.isAdmin) {
          return '/';  // Not admin, redirect to Home
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
        // The router will redirect automatically due to refreshListenable
        // But if there's a specific path, go to it
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
  redirectLimit: 10,  // Increase the limit if needed
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

// When you go to /parent/child:
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
// Correct ✅ Avoid redirecting to the same page
redirect: (context, state) {
  if (!isLoggedIn && state.uri.path != '/login') {
    return '/login';
  }
  return null;
}
```

### 3. استخدم refreshListenable

```dart
// Correct ✅ So the router knows when auth state changes
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
