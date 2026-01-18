# الدرس 20: Shell Routes في GoRouter Builder

## TypedShellRoute

لإنشاء ShellRoute مع الـ Builder:

```dart
@TypedShellRoute<MainShellRoute>(
  routes: [
    TypedGoRoute<HomeRoute>(path: '/home'),
    TypedGoRoute<SearchRoute>(path: '/search'),
    TypedGoRoute<ProfileRoute>(path: '/profile'),
  ],
)
class MainShellRoute extends ShellRouteData {
  const MainShellRoute();

  @override
  Widget builder(BuildContext context, GoRouterState state, Widget navigator) {
    return MainShell(child: navigator);
  }
}
```

---

## ShellRouteData

الـ base class للـ ShellRoute:

```dart
class MainShellRoute extends ShellRouteData {
  const MainShellRoute();

  @override
  Widget builder(
    BuildContext context,
    GoRouterState state,
    Widget navigator,  // 👈 The child route
  ) {
    return Scaffold(
      body: navigator,
      bottomNavigationBar: const MyBottomNav(),
    );
  }
}
```

---

## مثال كامل: Bottom Navigation

```dart
part 'routes.g.dart';

// ==================== Shell ====================

@TypedShellRoute<MainShellRoute>(
  routes: [
    TypedGoRoute<HomeRoute>(path: '/home'),
    TypedGoRoute<ExploreRoute>(path: '/explore'),
    TypedGoRoute<CartRoute>(path: '/cart'),
    TypedGoRoute<ProfileRoute>(path: '/profile'),
  ],
)
class MainShellRoute extends ShellRouteData {
  const MainShellRoute();

  @override
  Widget builder(BuildContext context, GoRouterState state, Widget navigator) {
    return ShellScaffold(
      currentPath: state.uri.path,
      child: navigator,
    );
  }
}

// ==================== Tab Routes ====================

class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const HomeScreen();
  }
}

class ExploreRoute extends GoRouteData {
  const ExploreRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const ExploreScreen();
  }
}

class CartRoute extends GoRouteData {
  const CartRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const CartScreen();
  }
}

class ProfileRoute extends GoRouteData {
  const ProfileRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const ProfileScreen();
  }
}

// ==================== Shell Widget ====================

class ShellScaffold extends StatelessWidget {
  const ShellScaffold({
    super.key,
    required this.currentPath,
    required this.child,
  });

  final String currentPath;
  final Widget child;

  static const _tabs = [
    (path: '/home', icon: Icons.home_outlined, selectedIcon: Icons.home, label: 'الرئيسية'),
    (path: '/explore', icon: Icons.explore_outlined, selectedIcon: Icons.explore, label: 'استكشف'),
    (path: '/cart', icon: Icons.shopping_cart_outlined, selectedIcon: Icons.shopping_cart, label: 'السلة'),
    (path: '/profile', icon: Icons.person_outline, selectedIcon: Icons.person, label: 'حسابي'),
  ];

  static const _routes = [HomeRoute(), ExploreRoute(), CartRoute(), ProfileRoute()];

  int get _selectedIndex {
    for (var i = 0; i < _tabs.length; i++) {
      if (currentPath.startsWith(_tabs[i].path)) return i;
    }
    return 0;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: NavigationBar(
        selectedIndex: _selectedIndex,
        onDestinationSelected: (index) => _routes[index].go(context),
        destinations: [
          for (final tab in _tabs)
            NavigationDestination(
              icon: Icon(tab.icon),
              selectedIcon: Icon(tab.selectedIcon),
              label: tab.label,
            ),
        ],
      ),
    );
  }
}
```

---

## TypedStatefulShellRoute

لحفظ حالة كل tab:

```dart
@TypedStatefulShellRoute<MainShellRoute>(
  branches: [
    TypedStatefulShellBranch<HomeBranch>(
      routes: [
        TypedGoRoute<HomeRoute>(path: '/home'),
      ],
    ),
    TypedStatefulShellBranch<SearchBranch>(
      routes: [
        TypedGoRoute<SearchRoute>(path: '/search'),
      ],
    ),
    TypedStatefulShellBranch<ProfileBranch>(
      routes: [
        TypedGoRoute<ProfileRoute>(path: '/profile'),
      ],
    ),
  ],
)
class MainShellRoute extends StatefulShellRouteData {
  const MainShellRoute();

  @override
  Widget builder(
    BuildContext context,
    GoRouterState state,
    StatefulNavigationShell navigationShell,
  ) {
    return MainShellWidget(navigationShell: navigationShell);
  }
}

// Branch classes
class HomeBranch extends StatefulShellBranchData {
  const HomeBranch();
}

class SearchBranch extends StatefulShellBranchData {
  const SearchBranch();
}

class ProfileBranch extends StatefulShellBranchData {
  const ProfileBranch();
}
```

---

## مثال كامل: StatefulShellRoute

```dart
part 'routes.g.dart';

// ==================== Stateful Shell ====================

@TypedStatefulShellRoute<AppShellRoute>(
  branches: [
    TypedStatefulShellBranch<HomeBranch>(
      routes: [
        TypedGoRoute<HomeRoute>(
          path: '/home',
          routes: [
            TypedGoRoute<ProductRoute>(path: 'product/:id'),
          ],
        ),
      ],
    ),
    TypedStatefulShellBranch<CategoriesBranch>(
      routes: [
        TypedGoRoute<CategoriesRoute>(
          path: '/categories',
          routes: [
            TypedGoRoute<CategoryRoute>(path: ':categoryId'),
          ],
        ),
      ],
    ),
    TypedStatefulShellBranch<CartBranch>(
      routes: [
        TypedGoRoute<CartRoute>(path: '/cart'),
      ],
    ),
    TypedStatefulShellBranch<ProfileBranch>(
      routes: [
        TypedGoRoute<ProfileRoute>(
          path: '/profile',
          routes: [
            TypedGoRoute<SettingsRoute>(path: 'settings'),
            TypedGoRoute<OrdersRoute>(path: 'orders'),
          ],
        ),
      ],
    ),
  ],
)
class AppShellRoute extends StatefulShellRouteData {
  const AppShellRoute();

  @override
  Widget builder(
    BuildContext context,
    GoRouterState state,
    StatefulNavigationShell navigationShell,
  ) {
    return AppShellWidget(navigationShell: navigationShell);
  }
}

// ==================== Branches ====================

class HomeBranch extends StatefulShellBranchData {
  const HomeBranch();
}

class CategoriesBranch extends StatefulShellBranchData {
  const CategoriesBranch();
}

class CartBranch extends StatefulShellBranchData {
  const CartBranch();
}

class ProfileBranch extends StatefulShellBranchData {
  const ProfileBranch();
}

// ==================== Routes ====================

class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const HomeScreen();
}

class ProductRoute extends GoRouteData {
  const ProductRoute({required this.id, this.$extra});

  final int id;
  final Product? $extra;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      ProductScreen(id: id, product: $extra);
}

class CategoriesRoute extends GoRouteData {
  const CategoriesRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const CategoriesScreen();
}

class CategoryRoute extends GoRouteData {
  const CategoryRoute({required this.categoryId});

  final String categoryId;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      CategoryScreen(categoryId: categoryId);
}

class CartRoute extends GoRouteData {
  const CartRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const CartScreen();
}

class ProfileRoute extends GoRouteData {
  const ProfileRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const ProfileScreen();
}

class SettingsRoute extends GoRouteData {
  const SettingsRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const SettingsScreen();
}

class OrdersRoute extends GoRouteData {
  const OrdersRoute();

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const OrdersScreen();
}

// ==================== Shell Widget ====================

class AppShellWidget extends StatelessWidget {
  final StatefulNavigationShell navigationShell;

  const AppShellWidget({super.key, required this.navigationShell});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (index) {
          navigationShell.goBranch(
            index,
            initialLocation: index == navigationShell.currentIndex,
          );
        },
        destinations: const [
          NavigationDestination(
            icon: Icon(Icons.home_outlined),
            selectedIcon: Icon(Icons.home),
            label: 'الرئيسية',
          ),
          NavigationDestination(
            icon: Icon(Icons.category_outlined),
            selectedIcon: Icon(Icons.category),
            label: 'التصنيفات',
          ),
          NavigationDestination(
            icon: Icon(Icons.shopping_cart_outlined),
            selectedIcon: Icon(Icons.shopping_cart),
            label: 'السلة',
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
}
```

---

## parentNavigatorKey

للخروج من الـ Shell:

```dart
// Create GlobalKey
final rootNavigatorKey = GlobalKey<NavigatorState>();

// In the router
final router = GoRouter(
  navigatorKey: rootNavigatorKey,
  routes: $appRoutes,
);

// In the Route class
@TypedGoRoute<CheckoutRoute>(path: '/checkout')
class CheckoutRoute extends GoRouteData {
  const CheckoutRoute();

  // Specify parent navigator
  static final GlobalKey<NavigatorState> $parentNavigatorKey = rootNavigatorKey;

  @override
  Widget build(BuildContext context, GoRouterState state) =>
      const CheckoutScreen();
}
```

---

## routes خارج الـ Shell

```dart
// Routes inside the Shell
@TypedStatefulShellRoute<AppShellRoute>(
  branches: [
    TypedStatefulShellBranch<HomeBranch>(
      routes: [
        TypedGoRoute<HomeRoute>(path: '/home'),
      ],
    ),
  ],
)
class AppShellRoute extends StatefulShellRouteData { ... }

// Routes outside the Shell
@TypedGoRoute<LoginRoute>(path: '/login')
class LoginRoute extends GoRouteData { ... }

@TypedGoRoute<OnboardingRoute>(path: '/onboarding')
class OnboardingRoute extends GoRouteData { ... }

// All will be collected in $appRoutes
```

---

## ملخص

| العنصر | الـ Class |
|--------|----------|
| **ShellRoute** | `ShellRouteData` |
| **StatefulShellRoute** | `StatefulShellRouteData` |
| **Branch** | `StatefulShellBranchData` |

| الـ Annotation | الاستخدام |
|----------------|----------|
| `@TypedShellRoute` | Shell بسيط |
| `@TypedStatefulShellRoute` | Shell مع حفظ الحالة |
| `@TypedStatefulShellBranch` | Branch في الـ stateful shell |

---

[⬅️ الدرس السابق: Parameters](19_builder_parameters.md) | [➡️ الدرس التالي: ميزات متقدمة](21_builder_advanced.md)
