# الدرس 9: Async Redirection

## المشكلة مع Sync Redirect

الـ redirect العادي **synchronous**، يعني مش بتقدر تعمل فيه `await`:

```dart
// Wrong ❌ This won't work
redirect: (context, state) async {  // Error!
  final user = await AuthService.getUser();
  if (user == null) return '/login';
  return null;
}
```

طيب لو محتاج تتحقق من API أو Database قبل الـ redirect؟

---

## الحلول المتاحة

### الحل 1: استخدام Stream مع refreshListenable

ده الحل الأكثر شيوعاً - بتستخدم Stream للـ auth state:

```dart
// Auth Service with Stream
class StreamAuth extends ChangeNotifier {
  // Stream controller for auth events
  final StreamController<bool> _controller = StreamController<bool>.broadcast();

  bool _isSignedIn = false;

  StreamAuth() {
    // Initialize
    _checkAuthStatus();
  }

  bool get isSignedIn => _isSignedIn;
  Stream<bool> get stream => _controller.stream;

  Future<void> _checkAuthStatus() async {
    // Simulate async check
    await Future.delayed(const Duration(milliseconds: 500));
    _isSignedIn = false;
    _controller.add(_isSignedIn);
    notifyListeners();
  }

  Future<void> signIn(String email, String password) async {
    // Simulate API call
    await Future.delayed(const Duration(seconds: 2));

    _isSignedIn = true;
    _controller.add(_isSignedIn);
    notifyListeners();
  }

  Future<void> signOut() async {
    await Future.delayed(const Duration(milliseconds: 500));

    _isSignedIn = false;
    _controller.add(_isSignedIn);
    notifyListeners();
  }

  void dispose() {
    _controller.close();
  }
}
```

```dart
// Using the Stream in the Router
final streamAuth = StreamAuth();

final appRouter = GoRouter(
  refreshListenable: streamAuth,

  redirect: (context, state) {
    // The state is ready because the Stream updates it
    final isSignedIn = streamAuth.isSignedIn;
    final isSigningIn = state.uri.path == '/login';

    if (!isSignedIn && !isSigningIn) {
      return '/login';
    }

    if (isSignedIn && isSigningIn) {
      return '/';
    }

    return null;
  },

  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginScreen(auth: streamAuth),
    ),
  ],
);
```

### مثال Login Screen

```dart
class LoginScreen extends StatefulWidget {
  final StreamAuth auth;

  const LoginScreen({super.key, required this.auth});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  bool _isLoading = false;

  Future<void> _signIn() async {
    setState(() => _isLoading = true);

    try {
      await widget.auth.signIn('user@example.com', 'password');
      // The router will redirect automatically due to the Stream
    } catch (e) {
      if (mounted) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(content: Text('Error: $e')),
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
      body: Center(
        child: _isLoading
            ? const Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  CircularProgressIndicator(),
                  SizedBox(height: 16),
                  Text('جاري تسجيل الدخول...'),
                ],
              )
            : ElevatedButton(
                onPressed: _signIn,
                child: const Text('دخول'),
              ),
      ),
    );
  }
}
```

---

### الحل 2: استخدام GoRouterRefreshStream

لو عندك Stream جاهز، تقدر تحوله لـ Listenable:

```dart
// Custom class to convert Stream to Listenable
class GoRouterRefreshStream extends ChangeNotifier {
  late final StreamSubscription<dynamic> _subscription;

  GoRouterRefreshStream(Stream<dynamic> stream) {
    _subscription = stream.asBroadcastStream().listen((_) {
      notifyListeners();
    });
  }

  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}

// Usage
final authStateStream = FirebaseAuth.instance.authStateChanges();

final appRouter = GoRouter(
  refreshListenable: GoRouterRefreshStream(authStateStream),

  redirect: (context, state) {
    final user = FirebaseAuth.instance.currentUser;
    final isLoggedIn = user != null;

    // ...
  },

  routes: [...],
);
```

---

### الحل 3: Splash Screen للتحميل الأولي

لو محتاج تعمل async initialization قبل ما التطبيق يبدأ:

```dart
class AppInitializer {
  static bool isInitialized = false;
  static User? currentUser;

  static Future<void> initialize() async {
    if (isInitialized) return;

    // Load user from storage
    currentUser = await SecureStorage.getUser();

    // Any other initialization
    await Analytics.initialize();
    await RemoteConfig.fetch();

    isInitialized = true;
  }
}

final appRouter = GoRouter(
  initialLocation: '/splash',

  redirect: (context, state) {
    // If not initialized, keep in splash
    if (!AppInitializer.isInitialized && state.uri.path != '/splash') {
      return '/splash';
    }

    // If initialized and still in splash
    if (AppInitializer.isInitialized && state.uri.path == '/splash') {
      return AppInitializer.currentUser != null ? '/' : '/login';
    }

    // Normal auth redirect
    if (AppInitializer.isInitialized) {
      final isLoggedIn = AppInitializer.currentUser != null;
      if (!isLoggedIn && state.uri.path != '/login') {
        return '/login';
      }
    }

    return null;
  },

  routes: [
    GoRoute(
      path: '/splash',
      builder: (context, state) => const SplashScreen(),
    ),
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
  ],
);

class SplashScreen extends StatefulWidget {
  const SplashScreen({super.key});

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();
    _initialize();
  }

  Future<void> _initialize() async {
    await AppInitializer.initialize();

    if (mounted) {
      // Go to the appropriate screen
      if (AppInitializer.currentUser != null) {
        context.go('/');
      } else {
        context.go('/login');
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            FlutterLogo(size: 100),
            SizedBox(height: 24),
            CircularProgressIndicator(),
          ],
        ),
      ),
    );
  }
}
```

---

### الحل 4: FutureProvider مع Riverpod

لو بتستخدم Riverpod:

```dart
// Auth provider
final authProvider = StreamProvider<User?>((ref) {
  return FirebaseAuth.instance.authStateChanges();
});

// Router provider
final routerProvider = Provider<GoRouter>((ref) {
  final authState = ref.watch(authProvider);

  return GoRouter(
    refreshListenable: RouterNotifier(ref),

    redirect: (context, state) {
      final isLoading = authState.isLoading;
      final user = authState.valueOrNull;

      // If still loading, don't redirect
      if (isLoading) return null;

      final isLoggedIn = user != null;
      final isAuthRoute = state.uri.path == '/login' ||
                          state.uri.path == '/register';

      if (!isLoggedIn && !isAuthRoute) return '/login';
      if (isLoggedIn && isAuthRoute) return '/';

      return null;
    },

    routes: [...],
  );
});

// Router notifier
class RouterNotifier extends ChangeNotifier {
  final Ref _ref;
  late final StreamSubscription<AsyncValue<User?>> _subscription;

  RouterNotifier(this._ref) {
    _subscription = _ref.listen(
      authProvider,
      (_, __) => notifyListeners(),
    ) as StreamSubscription<AsyncValue<User?>>;
  }

  @override
  void dispose() {
    _subscription.cancel();
    super.dispose();
  }
}
```

---

## Session Timeout مع Stream

مثال من الـ official examples - Session timeout تلقائي:

```dart
class StreamAuthWithTimeout extends ChangeNotifier {
  final StreamController<bool> _controller = StreamController<bool>.broadcast();
  Timer? _sessionTimer;
  bool _isSignedIn = false;

  // Session duration (20 seconds for testing)
  static const sessionDuration = Duration(seconds: 20);

  bool get isSignedIn => _isSignedIn;
  Stream<bool> get stream => _controller.stream;

  Future<void> signIn(String email, String password) async {
    await Future.delayed(const Duration(seconds: 2));

    _isSignedIn = true;
    _controller.add(_isSignedIn);
    notifyListeners();

    // Start the session timer
    _startSessionTimer();
  }

  void _startSessionTimer() {
    _sessionTimer?.cancel();
    _sessionTimer = Timer(sessionDuration, () {
      // Session expired
      _signOut();
    });
  }

  Future<void> _signOut() async {
    _sessionTimer?.cancel();
    _isSignedIn = false;
    _controller.add(_isSignedIn);
    notifyListeners();
  }

  Future<void> signOut() async {
    await _signOut();
  }

  // Refresh the session when the user interacts
  void refreshSession() {
    if (_isSignedIn) {
      _startSessionTimer();
    }
  }

  void dispose() {
    _sessionTimer?.cancel();
    _controller.close();
  }
}
```

---

## نصائح مهمة

### 1. استخدم Loading State

```dart
// During loading, show loading indicator
if (authState.isLoading) {
  return '/loading';  // Or null to stay in the same place
}
```

### 2. Handle Errors

```dart
final authState = ref.watch(authProvider);

authState.when(
  data: (user) {
    // Normal redirect logic
  },
  loading: () {
    // Show loading
  },
  error: (error, stack) {
    // Go to error page or login
    return '/error';
  },
);
```

### 3. Debounce الـ Stream

لو الـ Stream بيبعت events كتير:

```dart
Stream<bool> get debouncedStream =>
    stream.debounceTime(const Duration(milliseconds: 300));
```

---

## ملخص

| الحل | الاستخدام |
|------|-----------|
| **Stream + refreshListenable** | الأكثر شيوعاً للـ auth |
| **GoRouterRefreshStream** | لتحويل أي Stream |
| **Splash Screen** | للـ initialization الأولي |
| **Riverpod** | لو بتستخدم state management |

---

[⬅️ الدرس السابق: Redirection](08_redirection.md) | [➡️ الدرس التالي: Error Handling](10_error_handling.md)
