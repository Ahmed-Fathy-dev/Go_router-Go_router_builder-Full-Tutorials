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

// The Shell Widget
class ScaffoldWithBottomNav extends StatelessWidget {
  final Widget child;

  const ScaffoldWithBottomNav({super.key, required this.child});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        type: BottomNavigationBarType.fixed,
        currentIndex: _calculateSelectedIndex(context),
        onTap: (index) => _onItemTapped(index, context),
        items: const [
          BottomNavigationBarItem(
            icon: Icon(Icons.home),
            label: 'الرئيسية',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.search),
            label: 'بحث',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.favorite),
            label: 'المفضلة',
          ),
          BottomNavigationBarItem(
            icon: Icon(Icons.person),
            label: 'حسابي',
          ),
        ],
      ),
    );
  }

  int _calculateSelectedIndex(BuildContext context) {
    final location = GoRouterState.of(context).uri.path;

    if (location.startsWith('/home')) return 0;
    if (location.startsWith('/search')) return 1;
    if (location.startsWith('/favorites')) return 2;
    if (location.startsWith('/profile')) return 3;

    return 0;
  }

  void _onItemTapped(int index, BuildContext context) {
    switch (index) {
      case 0:
        context.go('/home');
        break;
      case 1:
        context.go('/search');
        break;
      case 2:
        context.go('/favorites');
        break;
      case 3:
        context.go('/profile');
        break;
    }
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
    return Scaffold(
      appBar: AppBar(
        title: Text(_getTitle(state.uri.path)),
      ),
      drawer: Drawer(
        child: ListView(
          children: [
            const DrawerHeader(
              child: Text('التطبيق'),
            ),
            ListTile(
              leading: const Icon(Icons.home),
              title: const Text('الرئيسية'),
              selected: state.uri.path == '/home',
              onTap: () {
                Navigator.pop(context);  // Close the drawer
                context.go('/home');
              },
            ),
            ListTile(
              leading: const Icon(Icons.settings),
              title: const Text('الإعدادات'),
              selected: state.uri.path == '/settings',
              onTap: () {
                Navigator.pop(context);
                context.go('/settings');
              },
            ),
            ListTile(
              leading: const Icon(Icons.info),
              title: const Text('عن التطبيق'),
              selected: state.uri.path == '/about',
              onTap: () {
                Navigator.pop(context);
                context.go('/about');
              },
            ),
          ],
        ),
      ),
      body: child,
    );
  },
  routes: [
    GoRoute(path: '/home', builder: ...),
    GoRoute(path: '/settings', builder: ...),
    GoRoute(path: '/about', builder: ...),
  ],
)

String _getTitle(String path) {
  switch (path) {
    case '/home': return 'الرئيسية';
    case '/settings': return 'الإعدادات';
    case '/about': return 'عن التطبيق';
    default: return 'التطبيق';
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
