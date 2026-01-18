# الدرس 23: Common Patterns

## 1. Authentication Flow

### باستخدام Riverpod 3+ (الطريقة الموصى بها)

```dart
// lib/features/auth/domain/auth_state.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'auth_state.freezed.dart';

@freezed
class AuthState with _$AuthState {
  const factory AuthState.initial() = _Initial;
  const factory AuthState.loading() = _Loading;
  const factory AuthState.authenticated(User user) = _Authenticated;
  const factory AuthState.unauthenticated() = _Unauthenticated;
  const factory AuthState.error(String message) = _Error;
}

extension AuthStateX on AuthState {
  bool get isAuthenticated => this is _Authenticated;
  User? get user => mapOrNull(authenticated: (s) => s.user);
}
```

```dart
// lib/features/auth/providers/auth_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'auth_provider.g.dart';

@Riverpod(keepAlive: true)
class Auth extends _$Auth {
  @override
  AuthState build() => const AuthState.initial();

  Future<void> checkAuthStatus() async {
    state = const AuthState.loading();
    try {
      final user = await ref.read(authRepositoryProvider).getCurrentUser();
      state = user != null
          ? AuthState.authenticated(user)
          : const AuthState.unauthenticated();
    } catch (e) {
      state = AuthState.error(e.toString());
    }
  }

  Future<void> login(String email, String password) async {
    state = const AuthState.loading();
    try {
      final user = await ref.read(authRepositoryProvider).login(email, password);
      state = AuthState.authenticated(user);
    } catch (e) {
      state = AuthState.error(e.toString());
    }
  }

  Future<void> logout() async {
    await ref.read(authRepositoryProvider).logout();
    state = const AuthState.unauthenticated();
  }
}
```

```dart
// lib/core/router/auth_refresh_listenable.dart
import 'package:flutter/foundation.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class AuthRefreshListenable extends ChangeNotifier {
  AuthRefreshListenable(Ref ref) {
    ref.listen(authProvider, (_, __) => notifyListeners());
  }
}

@riverpod
Listenable authRefreshListenable(Ref ref) => AuthRefreshListenable(ref);
```

```dart
// lib/core/router/app_router.dart
@riverpod
GoRouter appRouter(Ref ref) {
  final authState = ref.watch(authProvider);

  return GoRouter(
    refreshListenable: ref.watch(authRefreshListenableProvider),
    initialLocation: '/',

    redirect: (context, state) {
      final isAuthenticated = authState.isAuthenticated;
      final isAuthRoute = ['/login', '/register'].contains(state.uri.path);

      // Not authenticated & trying to access protected route
      if (!isAuthenticated && !isAuthRoute) {
        return '/login?redirect=${Uri.encodeComponent(state.uri.toString())}';
      }

      // Authenticated & trying to access auth routes
      if (isAuthenticated && isAuthRoute) {
        final redirect = state.uri.queryParameters['redirect'];
        return redirect != null ? Uri.decodeComponent(redirect) : '/';
      }

      return null;
    },

    routes: $appRoutes,
  );
}
```

```dart
// lib/main.dart
void main() {
  runApp(const ProviderScope(child: MyApp()));
}

class MyApp extends ConsumerWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return MaterialApp.router(
      routerConfig: ref.watch(appRouterProvider),
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
    );
  }
}
```

---

## 2. Splash Screen Pattern

```dart
// lib/features/app_init/providers/app_init_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'app_init_provider.g.dart';

enum AppInitStatus { initial, loading, ready, error }

@Riverpod(keepAlive: true)
class AppInit extends _$AppInit {
  @override
  AppInitStatus build() => AppInitStatus.initial;

  Future<void> initialize() async {
    if (state != AppInitStatus.initial) return;

    state = AppInitStatus.loading;
    try {
      // Initialize your services
      await Future.wait([
        ref.read(authProvider.notifier).checkAuthStatus(),
        ref.read(settingsProvider.notifier).load(),
        // Add more initialization tasks...
      ]);
      state = AppInitStatus.ready;
    } catch (e) {
      state = AppInitStatus.error;
    }
  }
}
```

```dart
// lib/features/splash/presentation/splash_screen.dart
class SplashScreen extends ConsumerStatefulWidget {
  const SplashScreen({super.key});

  @override
  ConsumerState<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends ConsumerState<SplashScreen> {
  @override
  void initState() {
    super.initState();
    // Start initialization
    ref.read(appInitProvider.notifier).initialize();
  }

  @override
  Widget build(BuildContext context) {
    // Listen to init status and redirect when ready
    ref.listen(appInitProvider, (_, status) {
      if (status == AppInitStatus.ready) {
        final isAuthenticated = ref.read(authProvider).isAuthenticated;
        context.go(isAuthenticated ? '/' : '/onboarding');
      }
    });

    return const Scaffold(
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
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

```dart
// Router configuration
@riverpod
GoRouter appRouter(Ref ref) {
  final initStatus = ref.watch(appInitProvider);

  return GoRouter(
    initialLocation: '/splash',

    redirect: (context, state) {
      final isSplash = state.uri.path == '/splash';

      // If not ready, stay in splash
      if (initStatus != AppInitStatus.ready && !isSplash) {
        return '/splash';
      }

      return null;
    },

    routes: $appRoutes,
  );
}
```

---

## 3. Tab Navigation Pattern

```dart
// lib/core/router/routes.dart
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) {
    return MainShell(navigationShell: navigationShell);
  },
  branches: [
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/home',
          name: 'home',
          builder: (context, state) => const HomeScreen(),
          routes: [
            GoRoute(
              path: 'details/:id',
              name: 'home-details',
              builder: (context, state) => DetailsScreen(
                id: state.pathParameters['id']!,
              ),
            ),
          ],
        ),
      ],
    ),
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/search',
          name: 'search',
          builder: (context, state) => const SearchScreen(),
        ),
      ],
    ),
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/profile',
          name: 'profile',
          builder: (context, state) => const ProfileScreen(),
        ),
      ],
    ),
  ],
)
```

```dart
// lib/core/presentation/main_shell.dart
class MainShell extends StatelessWidget {
  const MainShell({super.key, required this.navigationShell});

  final StatefulNavigationShell navigationShell;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: _onTap,
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'الرئيسية',
          ),
          NavigationDestination(
            icon: Icon(Icons.search_outlined),
            selectedIcon: Icon(Icons.search),
            label: 'البحث',
          ),
          NavigationDestination(
            icon: Icon(Icons.person_outline),
            selectedIcon: Icon(Icons.person),
            label: 'حسابي',
          ),
        ],
      ),
    );
  }

  void _onTap(int index) {
    navigationShell.goBranch(
      index,
      // Go to initial location if tapping the current tab
      initialLocation: index == navigationShell.currentIndex,
    );
  }
}
```

---

## 4. Master-Detail Pattern

```dart
// lib/core/router/routes.dart
GoRoute(
  path: '/items',
  name: 'items',
  builder: (context, state) => const ItemsListScreen(),
  routes: [
    GoRoute(
      path: ':id',
      name: 'item-detail',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        final item = state.extra as Item?;
        return ItemDetailScreen(id: id, item: item);
      },
    ),
  ],
)
```

```dart
// lib/features/items/presentation/items_list_screen.dart
class ItemsListScreen extends ConsumerWidget {
  const ItemsListScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final itemsAsync = ref.watch(itemsProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('العناصر')),
      body: itemsAsync.when(
        data: (items) => _ItemsList(items: items),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, _) => Center(child: Text('خطأ: $error')),
      ),
    );
  }
}

class _ItemsList extends StatelessWidget {
  const _ItemsList({required this.items});

  final List<Item> items;

  @override
  Widget build(BuildContext context) {
    if (items.isEmpty) {
      return const Center(child: Text('لا توجد عناصر'));
    }

    return ListView.builder(
      itemCount: items.length,
      itemBuilder: (context, index) => _ItemTile(item: items[index]),
    );
  }
}

class _ItemTile extends StatelessWidget {
  const _ItemTile({required this.item});

  final Item item;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      leading: CircleAvatar(child: Text(item.title[0])),
      title: Text(item.title),
      subtitle: Text(item.description),
      trailing: const Icon(Icons.chevron_right),
      onTap: () => context.push(
        '/items/${item.id}',
        extra: item, // Pass item to avoid refetching
      ),
    );
  }
}
```

```dart
// lib/features/items/presentation/item_detail_screen.dart
class ItemDetailScreen extends ConsumerWidget {
  const ItemDetailScreen({
    super.key,
    required this.id,
    this.item,
  });

  final String id;
  final Item? item;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Use passed item or fetch from provider
    final itemAsync = item != null
        ? AsyncValue.data(item!)
        : ref.watch(itemProvider(id));

    return Scaffold(
      appBar: AppBar(title: Text(item?.title ?? 'تفاصيل')),
      body: itemAsync.when(
        data: (item) => _ItemDetails(item: item!),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (error, _) => Center(child: Text('خطأ: $error')),
      ),
    );
  }
}

class _ItemDetails extends StatelessWidget {
  const _ItemDetails({required this.item});

  final Item item;

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(item.title, style: Theme.of(context).textTheme.headlineMedium),
          const SizedBox(height: 16),
          Text(item.description),
        ],
      ),
    );
  }
}
```

---

## 5. Modal/Dialog Pattern

```dart
// lib/core/router/routes.dart
GoRoute(
  path: '/confirm',
  name: 'confirm',
  pageBuilder: (context, state) {
    final args = state.extra as ConfirmDialogArgs?;
    return DialogPage(
      child: ConfirmDialog(
        title: args?.title ?? 'تأكيد',
        message: args?.message ?? 'هل أنت متأكد؟',
        confirmText: args?.confirmText ?? 'تأكيد',
        cancelText: args?.cancelText ?? 'إلغاء',
      ),
    );
  },
)
```

```dart
// lib/core/presentation/dialog_page.dart
class DialogPage<T> extends Page<T> {
  const DialogPage({
    required this.child,
    super.key,
    super.name,
  });

  final Widget child;

  @override
  Route<T> createRoute(BuildContext context) {
    return DialogRoute<T>(
      context: context,
      settings: this,
      barrierDismissible: true,
      barrierColor: Colors.black54,
      builder: (context) => child,
    );
  }
}
```

```dart
// lib/features/confirm/presentation/confirm_dialog.dart
class ConfirmDialogArgs {
  const ConfirmDialogArgs({
    this.title,
    this.message,
    this.confirmText,
    this.cancelText,
    this.isDestructive = false,
  });

  final String? title;
  final String? message;
  final String? confirmText;
  final String? cancelText;
  final bool isDestructive;
}

class ConfirmDialog extends StatelessWidget {
  const ConfirmDialog({
    super.key,
    required this.title,
    required this.message,
    required this.confirmText,
    required this.cancelText,
    this.isDestructive = false,
  });

  final String title;
  final String message;
  final String confirmText;
  final String cancelText;
  final bool isDestructive;

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: Text(title),
      content: Text(message),
      actions: [
        TextButton(
          onPressed: () => context.pop(false),
          child: Text(cancelText),
        ),
        FilledButton(
          onPressed: () => context.pop(true),
          style: isDestructive
              ? FilledButton.styleFrom(
                  backgroundColor: Theme.of(context).colorScheme.error,
                )
              : null,
          child: Text(confirmText),
        ),
      ],
    );
  }
}
```

```dart
// Usage
Future<void> _deleteItem() async {
  final confirmed = await context.push<bool>(
    '/confirm',
    extra: const ConfirmDialogArgs(
      title: 'حذف العنصر',
      message: 'هل أنت متأكد من حذف هذا العنصر؟',
      confirmText: 'حذف',
      isDestructive: true,
    ),
  );

  if (confirmed == true) {
    await ref.read(itemsProvider.notifier).delete(item.id);
  }
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
        // If entering /checkout directly, redirect to first step
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

// Navigation in each step
context.go('/checkout/shipping');  // Next step
context.go('/checkout/cart');      // Previous step
```

---

## 7. Search with History Pattern

```dart
// lib/core/router/routes.dart
GoRoute(
  path: '/search',
  name: 'search',
  builder: (context, state) {
    return SearchScreen(
      query: state.uri.queryParameters['q'],
      category: state.uri.queryParameters['category'],
    );
  },
)
```

```dart
// lib/features/search/presentation/search_screen.dart
class SearchScreen extends ConsumerStatefulWidget {
  const SearchScreen({
    super.key,
    this.query,
    this.category,
  });

  final String? query;
  final String? category;

  @override
  ConsumerState<SearchScreen> createState() => _SearchScreenState();
}

class _SearchScreenState extends ConsumerState<SearchScreen> {
  late final TextEditingController _controller;

  @override
  void initState() {
    super.initState();
    _controller = TextEditingController(text: widget.query);
  }

  @override
  void didUpdateWidget(SearchScreen oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.query != oldWidget.query) {
      _controller.text = widget.query ?? '';
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _search(String query) {
    if (query.isEmpty) return;

    final uri = Uri(
      path: '/search',
      queryParameters: {
        'q': query,
        if (widget.category != null) 'category': widget.category!,
      },
    );
    context.go(uri.toString());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: _SearchField(
          controller: _controller,
          onSubmitted: _search,
        ),
      ),
      body: widget.query != null
          ? _SearchResults(
              query: widget.query!,
              category: widget.category,
            )
          : const _SearchSuggestions(),
    );
  }
}

class _SearchField extends StatelessWidget {
  const _SearchField({
    required this.controller,
    required this.onSubmitted,
  });

  final TextEditingController controller;
  final ValueChanged<String> onSubmitted;

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: controller,
      decoration: InputDecoration(
        hintText: 'ابحث...',
        border: InputBorder.none,
        suffixIcon: IconButton(
          icon: const Icon(Icons.clear),
          onPressed: () => controller.clear(),
        ),
      ),
      textInputAction: TextInputAction.search,
      onSubmitted: onSubmitted,
    );
  }
}

class _SearchResults extends ConsumerWidget {
  const _SearchResults({
    required this.query,
    this.category,
  });

  final String query;
  final String? category;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final resultsAsync = ref.watch(
      searchResultsProvider((query: query, category: category)),
    );

    return resultsAsync.when(
      data: (results) => ListView.builder(
        itemCount: results.length,
        itemBuilder: (context, index) => _SearchResultTile(
          result: results[index],
        ),
      ),
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (error, _) => Center(child: Text('خطأ: $error')),
    );
  }
}

class _SearchResultTile extends StatelessWidget {
  const _SearchResultTile({required this.result});

  final SearchResult result;

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text(result.title),
      subtitle: Text(result.description),
      onTap: () => context.push('/items/${result.id}'),
    );
  }
}

class _SearchSuggestions extends StatelessWidget {
  const _SearchSuggestions();

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Text('ابدأ البحث...'),
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
// Using extension
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

// Usage
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
    // Check feature flags
    if (state.uri.path.startsWith('/new-feature')) {
      if (!FeatureFlags.isEnabled('new_feature')) {
        return '/';  // Feature not enabled
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
