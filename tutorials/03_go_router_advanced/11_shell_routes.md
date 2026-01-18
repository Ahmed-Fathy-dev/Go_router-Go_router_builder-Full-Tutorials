# الدرس 11: ShellRoute

## يعني إيه ShellRoute؟

الـ ShellRoute بيسمحلك تعمل **layout مشترك** (shell) يفضل ثابت وبس الـ content جواه يتغير. أشهر استخدام ليها هو الـ Bottom Navigation Bar أو الـ Drawer.

### المشكلة بدون ShellRoute

```dart
// Wrong ❌ Each page must rebuild the Scaffold and BottomNav
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: const HomeContent(),
      bottomNavigationBar: MyBottomNav(currentIndex: 0),
    );
  }
}

class SearchScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: const SearchContent(),
      bottomNavigationBar: MyBottomNav(currentIndex: 1),  // Repetition!
    );
  }
}
```

### الحل مع ShellRoute

```dart
// Correct ✅ The Shell wrapper is single and only the child changes
ShellRoute(
  builder: (context, state, child) {
    return Scaffold(
      body: child,  // 👈 This changes
      bottomNavigationBar: const MyBottomNav(),  // 👈 This stays fixed
    );
  },
  routes: [
    GoRoute(path: '/home', builder: ...),
    GoRoute(path: '/search', builder: ...),
    GoRoute(path: '/profile', builder: ...),
  ],
)
```

---

## الـ Syntax الأساسي

```dart
ShellRoute(
  // The builder takes 3 parameters
  builder: (
    BuildContext context,
    GoRouterState state,
    Widget child,  // 👈 The current route
  ) {
    return MyShellWidget(child: child);
  },

  // The routes inside the shell
  routes: [
    GoRoute(
      path: '/tab1',
      builder: (context, state) => const Tab1Screen(),
    ),
    GoRoute(
      path: '/tab2',
      builder: (context, state) => const Tab2Screen(),
    ),
  ],
)
```

---

## مثال كامل: Bottom Navigation

```dart
final appRouter = GoRouter(
  initialLocation: '/home',
  routes: [
    // The ShellRoute for Bottom Navigation
    ShellRoute(
      builder: (context, state, child) {
        return ScaffoldWithBottomNav(child: child);
      },
      routes: [
        GoRoute(
          path: '/home',
          builder: (context, state) => const HomeScreen(),
        ),
        GoRoute(
          path: '/search',
          builder: (context, state) => const SearchScreen(),
        ),
        GoRoute(
          path: '/favorites',
          builder: (context, state) => const FavoritesScreen(),
        ),
        GoRoute(
          path: '/profile',
          builder: (context, state) => const ProfileScreen(),
        ),
      ],
    ),

    // Route outside the Shell (no Bottom Nav)
    GoRoute(
      path: '/login',
      builder: (context, state) => const LoginScreen(),
    ),
  ],
);

// The Shell Widget (Material 3)
class ScaffoldWithBottomNav extends StatelessWidget {
  const ScaffoldWithBottomNav({super.key, required this.child});

  final Widget child;

  static const _destinations = [
    NavigationDestination(icon: Icon(Icons.home_outlined), selectedIcon: Icon(Icons.home), label: 'الرئيسية'),
    NavigationDestination(icon: Icon(Icons.search_outlined), selectedIcon: Icon(Icons.search), label: 'بحث'),
    NavigationDestination(icon: Icon(Icons.favorite_outline), selectedIcon: Icon(Icons.favorite), label: 'المفضلة'),
    NavigationDestination(icon: Icon(Icons.person_outline), selectedIcon: Icon(Icons.person), label: 'حسابي'),
  ];

  static const _paths = ['/home', '/search', '/favorites', '/profile'];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: NavigationBar(
        selectedIndex: _calculateSelectedIndex(context),
        onDestinationSelected: (index) => context.go(_paths[index]),
        destinations: _destinations,
      ),
    );
  }

  int _calculateSelectedIndex(BuildContext context) {
    final location = GoRouterState.of(context).uri.path;

    for (var i = 0; i < _paths.length; i++) {
      if (location.startsWith(_paths[i])) return i;
    }

    return 0;
  }
}
```

---

## Sub-Routes داخل ShellRoute

```dart
ShellRoute(
  builder: (context, state, child) => MainShell(child: child),
  routes: [
    GoRoute(
      path: '/home',
      builder: (context, state) => const HomeScreen(),
      routes: [
        // Sub-route - opens inside the shell
        GoRoute(
          path: 'notifications',  // /home/notifications
          builder: (context, state) => const NotificationsScreen(),
        ),
      ],
    ),
    GoRoute(
      path: '/profile',
      builder: (context, state) => const ProfileScreen(),
      routes: [
        GoRoute(
          path: 'settings',  // /profile/settings
          builder: (context, state) => const SettingsScreen(),
        ),
      ],
    ),
  ],
)
```

---

## parentNavigatorKey - الخروج من الـ Shell

أحياناً عايز route معين يفتح **خارج** الـ Shell (بدون Bottom Nav مثلاً):

```dart
// Create GlobalKey for the root Navigator
final _rootNavigatorKey = GlobalKey<NavigatorState>();

final appRouter = GoRouter(
  navigatorKey: _rootNavigatorKey,  // The root navigator

  routes: [
    ShellRoute(
      builder: (context, state, child) => MainShell(child: child),
      routes: [
        GoRoute(
          path: '/home',
          builder: (context, state) => const HomeScreen(),
        ),
        GoRoute(
          path: '/profile',
          builder: (context, state) => const ProfileScreen(),
          routes: [
            // This opens inside the shell
            GoRoute(
              path: 'edit',
              builder: (context, state) => const EditProfileScreen(),
            ),
            // This opens outside the shell (full screen)
            GoRoute(
              path: 'photo',
              parentNavigatorKey: _rootNavigatorKey,  // 👈
              builder: (context, state) => const FullScreenPhoto(),
            ),
          ],
        ),
      ],
    ),
  ],
);
```

---

## navigatorKey للـ ShellRoute

لو محتاج تتحكم في الـ Navigator بتاع الـ Shell:

```dart
final _shellNavigatorKey = GlobalKey<NavigatorState>();

ShellRoute(
  navigatorKey: _shellNavigatorKey,
  builder: (context, state, child) => MainShell(child: child),
  routes: [...],
)

// Usage
_shellNavigatorKey.currentState?.pop();
```

---

## مثال: Drawer مع ShellRoute

```dart
ShellRoute(
  builder: (context, state, child) {
    return _DrawerShell(path: state.uri.path, child: child);
  },
  routes: [
    GoRoute(path: '/home', builder: (_, __) => const HomeScreen()),
    GoRoute(path: '/settings', builder: (_, __) => const SettingsScreen()),
    GoRoute(path: '/about', builder: (_, __) => const AboutScreen()),
  ],
)

class _DrawerShell extends StatelessWidget {
  const _DrawerShell({required this.path, required this.child});

  final String path;
  final Widget child;

  static const _pages = {
    '/home': ('الرئيسية', Icons.home),
    '/settings': ('الإعدادات', Icons.settings),
    '/about': ('عن التطبيق', Icons.info),
  };

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(_pages[path]?.$1 ?? 'التطبيق')),
      drawer: NavigationDrawer(
        selectedIndex: _pages.keys.toList().indexOf(path),
        onDestinationSelected: (index) {
          Navigator.pop(context);
          context.go(_pages.keys.elementAt(index));
        },
        children: [
          const DrawerHeader(child: Text('التطبيق')),
          ..._pages.entries.map((e) => NavigationDrawerDestination(
            icon: Icon(e.value.$2),
            label: Text(e.value.$1),
          )),
        ],
      ),
      body: child,
    );
  }
}
```

---

## observers

تقدر تضيف NavigatorObserver للـ ShellRoute:

```dart
class ShellRouteObserver extends NavigatorObserver {
  @override
  void didPush(Route<dynamic> route, Route<dynamic>? previousRoute) {
    print('Shell: Pushed ${route.settings.name}');
  }

  @override
  void didPop(Route<dynamic> route, Route<dynamic>? previousRoute) {
    print('Shell: Popped ${route.settings.name}');
  }
}

ShellRoute(
  observers: [ShellRouteObserver()],
  builder: (context, state, child) => MainShell(child: child),
  routes: [...],
)
```

---

## restorationScopeId

للحفاظ على الـ navigation state بعد restart:

```dart
ShellRoute(
  restorationScopeId: 'main-shell',
  builder: (context, state, child) => MainShell(child: child),
  routes: [...],
)
```

---

## المشكلة: الحالة مش بتتحفظ

مع `ShellRoute` العادي، لما تتنقل بين الـ tabs:
- كل tab بيتبني من الأول
- الـ scroll position بيضيع
- الـ state بيضيع

**الحل؟** استخدم `StatefulShellRoute` (الدرس الجاي).

---

## نصائح

### 1. استخدم `go()` للـ tabs

```dart
// Correct ✅ - clears the stack
context.go('/home');
context.go('/search');

// Warning ⚠️ - push adds to the stack
context.push('/search');  // May cause problems
```

### 2. حدد الـ currentIndex صح

```dart
int _calculateSelectedIndex(BuildContext context) {
  final location = GoRouterState.of(context).uri.path;

  // Use startsWith for sub-routes
  if (location.startsWith('/home')) return 0;
  if (location.startsWith('/search')) return 1;

  return 0;
}
```

### 3. Routes خارج الـ Shell

```dart
GoRouter(
  routes: [
    // Routes with Shell
    ShellRoute(
      builder: ...,
      routes: [
        GoRoute(path: '/home', ...),
        GoRoute(path: '/profile', ...),
      ],
    ),

    // Routes without Shell
    GoRoute(path: '/login', ...),
    GoRoute(path: '/onboarding', ...),
    GoRoute(path: '/full-screen-video', ...),
  ],
)
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **الغرض** | عمل layout مشترك (Bottom Nav, Drawer) |
| **الـ builder** | بياخد `child` وهو الـ route الحالي |
| **parentNavigatorKey** | للخروج من الـ shell |
| **المشكلة** | الحالة مش بتتحفظ بين الـ tabs |
| **الحل** | استخدم `StatefulShellRoute` |

---

[⬅️ الدرس السابق: Error Handling](../02_go_router_intermediate/10_error_handling.md) | [➡️ الدرس التالي: StatefulShellRoute](12_stateful_shell_route.md)
