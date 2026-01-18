# الدرس 21: ميزات متقدمة في GoRouter Builder

## Custom Transitions (buildPage)

بدل `build()` استخدم `buildPage()` للتحكم في الـ transition:

```dart
@TypedGoRoute<ModalRoute>(path: '/modal')
class ModalRoute extends GoRouteData {
  const ModalRoute();

  @override
  Page<void> buildPage(BuildContext context, GoRouterState state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: const ModalScreen(),
      transitionDuration: const Duration(milliseconds: 300),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return SlideTransition(
          position: Tween<Offset>(
            begin: const Offset(0, 1),
            end: Offset.zero,
          ).animate(CurvedAnimation(
            parent: animation,
            curve: Curves.easeOutCubic,
          )),
          child: child,
        );
      },
    );
  }
}
```

---

## أنواع Pages المختلفة

### FadeTransition

```dart
@override
Page<void> buildPage(BuildContext context, GoRouterState state) {
  return CustomTransitionPage(
    key: state.pageKey,
    child: const MyScreen(),
    transitionsBuilder: (context, animation, _, child) {
      return FadeTransition(opacity: animation, child: child);
    },
  );
}
```

### NoTransition

```dart
@override
Page<void> buildPage(BuildContext context, GoRouterState state) {
  return NoTransitionPage(
    key: state.pageKey,
    child: const MyScreen(),
  );
}
```

### MaterialPage

```dart
@override
Page<void> buildPage(BuildContext context, GoRouterState state) {
  return MaterialPage(
    key: state.pageKey,
    child: const MyScreen(),
    fullscreenDialog: true,  // For modals
  );
}
```

---

## Return Values من push

### تعريف Route بـ Return Type

```dart
@TypedGoRoute<ConfirmRoute>(path: '/confirm')
class ConfirmRoute extends GoRouteData {
  const ConfirmRoute({required this.message});

  final String message;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ConfirmDialog(
      message: message,
      onConfirm: () => context.pop(true),
      onCancel: () => context.pop(false),
    );
  }
}
```

### استخدام push مع await

```dart
// In the original screen
Future<void> _showConfirmation() async {
  final result = await ConfirmRoute(
    message: 'هل أنت متأكد؟',
  ).push<bool>(context);

  if (result == true) {
    // Confirmed
    _performAction();
  }
}
```

---

## TypedRelativeGoRoute

لإعادة استخدام route في أماكن مختلفة:

```dart
// Reusable route
class DetailsRoute extends GoRouteData {
  const DetailsRoute({required this.id});

  final int id;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return DetailsScreen(id: id);
  }
}

// Using it in different places
@TypedGoRoute<ProductsRoute>(
  path: '/products',
  routes: [
    TypedRelativeGoRoute<DetailsRoute>(path: ':id'),  // /products/:id
  ],
)
class ProductsRoute extends GoRouteData { ... }

@TypedGoRoute<OrdersRoute>(
  path: '/orders',
  routes: [
    TypedRelativeGoRoute<DetailsRoute>(path: ':id'),  // /orders/:id
  ],
)
class OrdersRoute extends GoRouteData { ... }
```

---

## Redirect في Route

```dart
@TypedGoRoute<ProtectedRoute>(path: '/protected')
class ProtectedRoute extends GoRouteData {
  const ProtectedRoute();

  @override
  String? redirect(BuildContext context, GoRouterState state) {
    // Check authentication
    if (!AuthService.isLoggedIn) {
      return const LoginRoute(
        redirectTo: '/protected',
      ).location;
    }

    // Check permissions
    if (!AuthService.hasPermission('view_protected')) {
      return const HomeRoute().location;
    }

    return null;  // Allow access
  }

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const ProtectedScreen();
  }
}
```

---

## onExit للتأكيد قبل الخروج

```dart
@TypedGoRoute<FormRoute>(path: '/form')
class FormRoute extends GoRouteData {
  const FormRoute();

  @override
  Future<bool> onExit(BuildContext context, GoRouterState state) async {
    // If there are unsaved changes
    final formState = FormController.of(context);

    if (formState.hasUnsavedChanges) {
      return await showDialog<bool>(
        context: context,
        builder: (context) => AlertDialog(
          title: const Text('تأكيد'),
          content: const Text('فيه تغييرات مش محفوظة. تريد الخروج؟'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context, false),
              child: const Text('إلغاء'),
            ),
            TextButton(
              onPressed: () => Navigator.pop(context, true),
              child: const Text('خروج'),
            ),
          ],
        ),
      ) ?? false;
    }

    return true;
  }

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return const FormScreen();
  }
}
```

---

## Factory Constructor للـ Route

```dart
@TypedGoRoute<ProductRoute>(path: '/product/:id')
class ProductRoute extends GoRouteData {
  const ProductRoute._({
    required this.id,
    this.product,
  });

  final int id;
  final Product? product;

  // Factory to create from a Product directly
  factory ProductRoute.fromProduct(Product product) {
    return ProductRoute._(
      id: product.id,
      product: product,
    );
  }

  // Factory to create from ID only
  factory ProductRoute.fromId(int id) {
    return ProductRoute._(id: id);
  }

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductScreen(id: id, product: product);
  }
}

// Usage
ProductRoute.fromProduct(myProduct).push(context);
ProductRoute.fromId(123).go(context);
```

---

## مثال شامل

```dart
part 'routes.g.dart';

// ==================== Keys ====================
final rootNavigatorKey = GlobalKey<NavigatorState>();

// ==================== App Shell ====================

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
    TypedStatefulShellBranch<ProfileBranch>(
      routes: [
        TypedGoRoute<ProfileRoute>(
          path: '/profile',
          routes: [
            TypedGoRoute<EditProfileRoute>(path: 'edit'),
          ],
        ),
      ],
    ),
  ],
)
class AppShellRoute extends StatefulShellRouteData {
  const AppShellRoute();

  @override
  Widget builder(context, state, navigationShell) =>
      AppShell(navigationShell: navigationShell);
}

class HomeBranch extends StatefulShellBranchData {
  const HomeBranch();
}

class ProfileBranch extends StatefulShellBranchData {
  const ProfileBranch();
}

// ==================== Routes ====================

class HomeRoute extends GoRouteData {
  const HomeRoute();

  @override
  Widget build(context, state) => const HomeScreen();
}

class ProductRoute extends GoRouteData {
  const ProductRoute({required this.id, this.$extra});

  final int id;
  final Product? $extra;

  // Custom transition
  @override
  Page<void> buildPage(context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: ProductScreen(id: id, product: $extra),
      transitionsBuilder: (context, animation, _, child) {
        return FadeTransition(
          opacity: animation,
          child: SlideTransition(
            position: Tween(
              begin: const Offset(0, 0.1),
              end: Offset.zero,
            ).animate(animation),
            child: child,
          ),
        );
      },
    );
  }
}

class ProfileRoute extends GoRouteData {
  const ProfileRoute();

  // Redirect if not logged in
  @override
  String? redirect(context, state) {
    if (!AuthService.isLoggedIn) {
      return const LoginRoute().location;
    }
    return null;
  }

  @override
  Widget build(context, state) => const ProfileScreen();
}

class EditProfileRoute extends GoRouteData {
  const EditProfileRoute();

  // Confirm before exit
  @override
  Future<bool> onExit(context, state) async {
    if (hasUnsavedChanges) {
      return await showExitConfirmDialog(context);
    }
    return true;
  }

  @override
  Widget build(context, state) => const EditProfileScreen();
}

// ==================== Modal Routes ====================

@TypedGoRoute<ConfirmDialogRoute>(path: '/confirm')
class ConfirmDialogRoute extends GoRouteData {
  const ConfirmDialogRoute({required this.message});

  final String message;

  // Outside the shell
  static final GlobalKey<NavigatorState> $parentNavigatorKey = rootNavigatorKey;

  // Modal transition
  @override
  Page<bool> buildPage(context, state) {
    return CustomTransitionPage<bool>(
      key: state.pageKey,
      child: ConfirmDialog(
        message: message,
        onConfirm: () => context.pop(true),
        onCancel: () => context.pop(false),
      ),
      opaque: false,
      barrierColor: Colors.black54,
      transitionsBuilder: (context, animation, _, child) {
        return FadeTransition(
          opacity: animation,
          child: ScaleTransition(
            scale: Tween(begin: 0.8, end: 1.0).animate(
              CurvedAnimation(parent: animation, curve: Curves.easeOut),
            ),
            child: child,
          ),
        );
      },
    );
  }
}

// ==================== Login Route ====================

@TypedGoRoute<LoginRoute>(path: '/login')
class LoginRoute extends GoRouteData {
  const LoginRoute({this.redirectTo});

  final String? redirectTo;

  // Outside the shell
  static final GlobalKey<NavigatorState> $parentNavigatorKey = rootNavigatorKey;

  @override
  Widget build(context, state) => LoginScreen(redirectTo: redirectTo);
}

// ==================== Usage ====================

// Navigation with transition
ProductRoute(id: 123, $extra: product).push(context);

// Modal with return value
final confirmed = await const ConfirmDialogRoute(
  message: 'Do you want to delete?',
).push<bool>(context);

if (confirmed == true) {
  // Delete
}

// Protected route - redirect handled automatically
const ProfileRoute().go(context);
```

---

## ملخص

| الميزة | الـ Method/Property |
|--------|---------------------|
| **Custom Transition** | `buildPage()` |
| **Return Value** | `push<T>()` + `context.pop(value)` |
| **Redirect** | `redirect()` |
| **Exit Confirmation** | `onExit()` |
| **Parent Navigator** | `static $parentNavigatorKey` |
| **Reusable Route** | `TypedRelativeGoRoute` |

---

[⬅️ الدرس السابق: Shell Routes](20_builder_shell_routes.md) | [➡️ الدرس التالي: هيكلة المشروع](../05_best_practices/22_project_structure.md)
