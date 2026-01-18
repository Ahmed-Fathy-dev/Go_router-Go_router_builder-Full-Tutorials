# الدرس 23: Common Patterns

## 1. Authentication Flow

```dart
class AuthNotifier extends ChangeNotifier {
  bool _isLoggedIn = false;
  bool get isLoggedIn => _isLoggedIn;

  void login() {
    _isLoggedIn = true;
    notifyListeners();
  }

  void logout() {
    _isLoggedIn = false;
    notifyListeners();
  }
}

final authNotifier = AuthNotifier();

final appRouter = GoRouter(
  refreshListenable: authNotifier,
  initialLocation: '/',

  redirect: (context, state) {
    final isLoggedIn = authNotifier.isLoggedIn;
    final isAuthRoute = state.uri.path == '/login' ||
                        state.uri.path == '/register';

    // مش مسجل + مش في صفحة auth
    if (!isLoggedIn && !isAuthRoute) {
      return '/login?redirect=${state.uri}';
    }

    // مسجل + في صفحة auth
    if (isLoggedIn && isAuthRoute) {
      final redirect = state.uri.queryParameters['redirect'];
      return redirect ?? '/';
    }

    return null;
  },

  routes: [
    GoRoute(path: '/', builder: ...),
    GoRoute(path: '/login', builder: ...),
    GoRoute(path: '/register', builder: ...),
    GoRoute(path: '/profile', builder: ...),
  ],
);
```

---

## 2. Splash Screen Pattern

```dart
final appRouter = GoRouter(
  initialLocation: '/splash',

  redirect: (context, state) {
    final isInitialized = AppInitService.isInitialized;
    final isSplash = state.uri.path == '/splash';

    // لو مش initialized، ابقى في splash
    if (!isInitialized && !isSplash) {
      return '/splash';
    }

    // لو initialized ولسه في splash
    if (isInitialized && isSplash) {
      return AuthService.isLoggedIn ? '/' : '/onboarding';
    }

    return null;
  },

  routes: [
    GoRoute(
      path: '/splash',
      builder: (context, state) => const SplashScreen(),
    ),
    // باقي الـ routes
  ],
);

class SplashScreen extends StatefulWidget {
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
    await AppInitService.initialize();
    if (mounted) {
      // الـ router هيعمل redirect تلقائي
      context.go('/');
    }
  }

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      body: Center(child: CircularProgressIndicator()),
    );
  }
}
```

---

## 3. Tab Navigation Pattern

```dart
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) {
    return TabScaffold(navigationShell: navigationShell);
  },
  branches: [
    StatefulShellBranch(routes: [
      GoRoute(path: '/home', builder: ...),
    ]),
    StatefulShellBranch(routes: [
      GoRoute(path: '/search', builder: ...),
    ]),
    StatefulShellBranch(routes: [
      GoRoute(path: '/profile', builder: ...),
    ]),
  ],
)

class TabScaffold extends StatelessWidget {
  final StatefulNavigationShell navigationShell;

  const TabScaffold({super.key, required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: navigationShell.currentIndex,
        onTap: (index) => navigationShell.goBranch(
          index,
          initialLocation: index == navigationShell.currentIndex,
        ),
        items: const [...],
      ),
    );
  }
}
```

---

## 4. Master-Detail Pattern

```dart
final appRouter = GoRouter(
  routes: [
    GoRoute(
      path: '/items',
      builder: (context, state) => const ItemsListScreen(),
      routes: [
        GoRoute(
          path: ':id',
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return ItemDetailScreen(id: id);
          },
        ),
      ],
    ),
  ],
);

// List Screen
class ItemsListScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('العناصر')),
      body: ListView.builder(
        itemCount: items.length,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text(items[index].title),
            onTap: () => context.push('/items/${items[index].id}'),
          );
        },
      ),
    );
  }
}
```

---

## 5. Modal/Dialog Pattern

```dart
GoRoute(
  path: '/confirm',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: ConfirmDialog(message: state.extra as String?),
      opaque: false,
      barrierDismissible: true,
      barrierColor: Colors.black54,
      transitionsBuilder: (context, animation, _, child) {
        return FadeTransition(
          opacity: animation,
          child: child,
        );
      },
    );
  },
)

// الاستخدام
final result = await context.push<bool>('/confirm', extra: 'هل أنت متأكد؟');
if (result == true) {
  // تم التأكيد
}
```

---

## 6. Wizard/Multi-step Pattern

```dart
final appRouter = GoRouter(
  routes: [
    GoRoute(
      path: '/checkout',
      redirect: (context, state) {
        // لو دخل /checkout مباشرة، وديه للخطوة الأولى
        if (state.uri.path == '/checkout') {
          return '/checkout/cart';
        }
        return null;
      },
      routes: [
        GoRoute(
          path: 'cart',
          builder: (context, state) => const CartStep(),
        ),
        GoRoute(
          path: 'shipping',
          builder: (context, state) => const ShippingStep(),
        ),
        GoRoute(
          path: 'payment',
          builder: (context, state) => const PaymentStep(),
        ),
        GoRoute(
          path: 'confirm',
          builder: (context, state) => const ConfirmStep(),
        ),
      ],
    ),
  ],
);

// Navigation في كل step
context.go('/checkout/shipping');  // للخطوة التالية
context.go('/checkout/cart');      // للخطوة السابقة
```

---

## 7. Search with History Pattern

```dart
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.uri.queryParameters['q'];
    return SearchScreen(initialQuery: query);
  },
)

class SearchScreen extends StatelessWidget {
  final String? initialQuery;

  void _search(BuildContext context, String query) {
    // بيحدث الـ URL بدون إضافة للـ history
    context.go('/search?q=${Uri.encodeComponent(query)}');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: SearchField(
          initialValue: initialQuery,
          onSubmitted: (query) => _search(context, query),
        ),
      ),
      body: initialQuery != null
          ? SearchResults(query: initialQuery!)
          : const SearchSuggestions(),
    );
  }
}
```

---

## 8. Deep Link Handler Pattern

```dart
GoRouter(
  onEnter: (context, state) {
    // Handle referral codes
    final ref = state.uri.queryParameters['ref'];
    if (ref != null) {
      ReferralService.track(ref);
    }

    // Handle OAuth callbacks
    if (state.uri.path == '/oauth/callback') {
      final code = state.uri.queryParameters['code'];
      if (code != null) {
        AuthService.handleOAuth(code);
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

## 9. Protected Route Pattern

```dart
// باستخدام extension
extension ProtectedRoute on GoRoute {
  GoRoute protected() {
    return GoRoute(
      path: path,
      name: name,
      redirect: (context, state) {
        if (!AuthService.isLoggedIn) {
          return '/login?redirect=${state.uri}';
        }
        return redirect?.call(context, state);
      },
      builder: builder,
      routes: routes,
    );
  }
}

// الاستخدام
GoRoute(
  path: '/profile',
  builder: (context, state) => const ProfileScreen(),
).protected()
```

---

## 10. Feature Flag Pattern

```dart
GoRouter(
  redirect: (context, state) {
    // تحقق من الـ feature flags
    if (state.uri.path.startsWith('/new-feature')) {
      if (!FeatureFlags.isEnabled('new_feature')) {
        return '/';  // الميزة مش مفعلة
      }
    }

    return null;
  },
  routes: [
    GoRoute(
      path: '/new-feature',
      builder: (context, state) => const NewFeatureScreen(),
    ),
  ],
)
```

---

## ملخص

| الـ Pattern | الاستخدام |
|-------------|----------|
| **Auth Flow** | حماية الصفحات + redirect |
| **Splash** | Initialization + redirect |
| **Tab Nav** | StatefulShellRoute |
| **Master-Detail** | Parent + child routes |
| **Modal** | CustomTransitionPage |
| **Wizard** | Sequential routes |
| **Search** | Query parameters |
| **Deep Link** | onEnter callback |

---

[⬅️ الدرس السابق: هيكلة المشروع](22_project_structure.md) | [➡️ الدرس التالي: Testing](24_testing.md)
