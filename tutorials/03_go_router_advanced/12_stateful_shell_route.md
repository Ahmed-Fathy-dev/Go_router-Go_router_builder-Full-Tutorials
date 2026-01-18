# الدرس 12: StatefulShellRoute

## الفرق بين ShellRoute و StatefulShellRoute

| الخاصية | ShellRoute | StatefulShellRoute |
|---------|------------|---------------------|
| **حفظ الحالة** | ❌ لا | ✅ نعم |
| **Navigation Stack** | مشترك | منفصل لكل branch |
| **الـ Rebuild** | كل مرة | مرة واحدة |
| **الاستخدام** | layouts بسيطة | Bottom Nav مع tabs |

---

## ليه StatefulShellRoute؟

لما بتستخدم `ShellRoute` العادي:
- لما تتنقل من tab لـ tab، الـ screen القديمة بتتبني من الأول
- الـ scroll position بيضيع
- أي state في الـ screen بيضيع

ال **StatefulShellRoute** بيحل المشكلة دي عن طريق:
- إنشاء Navigator **منفصل** لكل branch
- حفظ الحالة لكل tab
- الـ screens مش بيتعملها dispose لما تتنقل لـ tab تاني

---

## StatefulShellRoute.indexedStack

أسهل طريقة تستخدم `StatefulShellRoute` هي عن طريق `indexedStack`:

```dart
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) {
    return ScaffoldWithNavBar(navigationShell: navigationShell);
  },
  branches: [
    // Branch A - Home
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/home',
          builder: (context, state) => const HomeScreen(),
          routes: [
            GoRoute(
              path: 'details/:id',
              builder: (context, state) => DetailsScreen(
                id: state.pathParameters['id']!,
              ),
            ),
          ],
        ),
      ],
    ),

    // Branch B - Search
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/search',
          builder: (context, state) => const SearchScreen(),
        ),
      ],
    ),

    // Branch C - Profile
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/profile',
          builder: (context, state) => const ProfileScreen(),
        ),
      ],
    ),
  ],
)
```

---

## مثال كامل

```dart
final _rootNavigatorKey = GlobalKey<NavigatorState>();

final appRouter = GoRouter(
  navigatorKey: _rootNavigatorKey,
  initialLocation: '/home',
  routes: [
    // The Stateful Shell for Bottom Navigation
    StatefulShellRoute.indexedStack(
      builder: (context, state, navigationShell) {
        return ScaffoldWithNavBar(navigationShell: navigationShell);
      },
      branches: [
        // Home
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/home',
              builder: (context, state) => const HomeScreen(),
              routes: [
                GoRoute(
                  path: 'product/:id',
                  builder: (context, state) {
                    final id = state.pathParameters['id']!;
                    return ProductScreen(id: id);
                  },
                ),
              ],
            ),
          ],
        ),

        // Categories
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/categories',
              builder: (context, state) => const CategoriesScreen(),
              routes: [
                GoRoute(
                  path: ':categoryId',
                  builder: (context, state) {
                    final id = state.pathParameters['categoryId']!;
                    return CategoryScreen(id: id);
                  },
                ),
              ],
            ),
          ],
        ),

        // Cart
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/cart',
              builder: (context, state) => const CartScreen(),
            ),
          ],
        ),

        // My Account
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/account',
              builder: (context, state) => const AccountScreen(),
              routes: [
                GoRoute(
                  path: 'orders',
                  builder: (context, state) => const OrdersScreen(),
                ),
                GoRoute(
                  path: 'settings',
                  builder: (context, state) => const SettingsScreen(),
                ),
              ],
            ),
          ],
        ),
      ],
    ),

    // Routes outside the Shell
    GoRoute(
      path: '/login',
      parentNavigatorKey: _rootNavigatorKey,
      builder: (context, state) => const LoginScreen(),
    ),
    GoRoute(
      path: '/checkout',
      parentNavigatorKey: _rootNavigatorKey,
      builder: (context, state) => const CheckoutScreen(),
    ),
  ],
);
```

### الـ Shell Widget

```dart
class ScaffoldWithNavBar extends StatelessWidget {
  final StatefulNavigationShell navigationShell;

  const ScaffoldWithNavBar({
    super.key,
    required this.navigationShell,
  });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: navigationShell,  // 👈 This displays the current branch
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (index) {
          // 👈 This method navigates between branches
          navigationShell.goBranch(
            index,
            // If tapped on the same tab, return to initial location
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

## navigationShell.goBranch()

الـ method الرئيسية للتنقل بين الـ branches:

```dart
navigationShell.goBranch(
  index,                    // Branch number (0, 1, 2, ...)
  initialLocation: false,   // Should it go to initial location?
);
```

### السلوك:
- `initialLocation: false` (default) - يروح للـ branch ويحافظ على الـ stack
- `initialLocation: true` - يروح للـ initial location بتاع الـ branch (يمسح الـ stack)

```dart
// Example: when user taps on the current tab
onDestinationSelected: (index) {
  if (index == navigationShell.currentIndex) {
    // Tapped on same tab -> return to start
    navigationShell.goBranch(index, initialLocation: true);
  } else {
    // Different tab -> navigate and preserve state
    navigationShell.goBranch(index);
  }
}
```

---

## navigatorKey لكل Branch

تقدر تحدد `navigatorKey` لكل branch:

```dart
final _homeNavigatorKey = GlobalKey<NavigatorState>();
final _searchNavigatorKey = GlobalKey<NavigatorState>();

StatefulShellRoute.indexedStack(
  branches: [
    StatefulShellBranch(
      navigatorKey: _homeNavigatorKey,
      routes: [...],
    ),
    StatefulShellBranch(
      navigatorKey: _searchNavigatorKey,
      routes: [...],
    ),
  ],
)

// Usage
_homeNavigatorKey.currentState?.pop();
```

---

## restorationScopeId

للحفاظ على الـ state بعد restart التطبيق:

```dart
StatefulShellRoute.indexedStack(
  restorationScopeId: 'main-shell',
  branches: [
    StatefulShellBranch(
      restorationScopeId: 'home-branch',
      routes: [...],
    ),
    StatefulShellBranch(
      restorationScopeId: 'search-branch',
      routes: [...],
    ),
  ],
)
```

---

## Custom Transition بين الـ Branches

باستخدام `StatefulShellRoute` العادي (مش `indexedStack`):

```dart
StatefulShellRoute(
  navigatorContainerBuilder: (context, navigationShell, children) {
    // children is List<Widget> for each branch
    return AnimatedBranchContainer(
      currentIndex: navigationShell.currentIndex,
      children: children,
    );
  },
  builder: (context, state, navigationShell) {
    return ScaffoldWithNavBar(navigationShell: navigationShell);
  },
  branches: [...],
)

class AnimatedBranchContainer extends StatelessWidget {
  final int currentIndex;
  final List<Widget> children;

  const AnimatedBranchContainer({
    super.key,
    required this.currentIndex,
    required this.children,
  });

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: children.asMap().entries.map((entry) {
        final index = entry.key;
        final child = entry.value;
        final isActive = index == currentIndex;

        return AnimatedOpacity(
          opacity: isActive ? 1.0 : 0.0,
          duration: const Duration(milliseconds: 300),
          child: IgnorePointer(
            ignoring: !isActive,
            child: child,
          ),
        );
      }).toList(),
    );
  }
}
```

---

## مثال: حفظ Scroll Position

```dart
class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen>
    with AutomaticKeepAliveClientMixin {
  final ScrollController _scrollController = ScrollController();

  // This keeps the widget alive
  @override
  bool get wantKeepAlive => true;

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    super.build(context);  // Important for AutomaticKeepAliveClientMixin

    return ListView.builder(
      controller: _scrollController,
      itemCount: 100,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text('Item $index'),
          onTap: () => context.go('/home/product/$index'),
        );
      },
    );
  }
}
```

---

## التنقل داخل Branch

```dart
// Navigation inside the same branch
context.go('/home/product/123');  // push inside the home branch

// Or
context.push('/home/product/123');

// Go back
context.pop();
```

---

## خروج من الـ Shell لصفحة Full Screen

```dart
final _rootNavigatorKey = GlobalKey<NavigatorState>();

GoRouter(
  navigatorKey: _rootNavigatorKey,
  routes: [
    StatefulShellRoute.indexedStack(
      branches: [
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/home',
              builder: (context, state) => const HomeScreen(),
              routes: [
                // This is inside the shell
                GoRoute(
                  path: 'product/:id',
                  builder: (context, state) => ProductScreen(...),
                ),
                // This is outside the shell (full screen)
                GoRoute(
                  path: 'product/:id/fullscreen',
                  parentNavigatorKey: _rootNavigatorKey,  // 👈
                  builder: (context, state) => FullScreenProductView(...),
                ),
              ],
            ),
          ],
        ),
      ],
    ),
  ],
)
```

---

## Nested StatefulShellRoute

تقدر تعمل nested shells (shell جوه shell):

```dart
StatefulShellRoute.indexedStack(
  branches: [
    StatefulShellBranch(
      routes: [
        // Shell inside the home branch
        StatefulShellRoute.indexedStack(
          builder: (context, state, navigationShell) {
            return NestedShell(navigationShell: navigationShell);
          },
          branches: [
            StatefulShellBranch(routes: [...]),
            StatefulShellBranch(routes: [...]),
          ],
        ),
      ],
    ),
  ],
)
```

---

## نصائح مهمة

### 1. استخدم AutomaticKeepAliveClientMixin

```dart
class MyScreen extends StatefulWidget {
  @override
  State<MyScreen> createState() => _MyScreenState();
}

class _MyScreenState extends State<MyScreen>
    with AutomaticKeepAliveClientMixin {

  @override
  bool get wantKeepAlive => true;

  @override
  Widget build(BuildContext context) {
    super.build(context);  // Important!
    return ...;
  }
}
```

### 2. Double-tap على Tab للرجوع للأول

```dart
onDestinationSelected: (index) {
  navigationShell.goBranch(
    index,
    initialLocation: index == navigationShell.currentIndex,
  );
}
```

### 3. Badge على Tab

```dart
NavigationDestination(
  icon: Badge(
    label: Text('3'),
    child: Icon(Icons.shopping_cart_outlined),
  ),
  selectedIcon: Badge(
    label: Text('3'),
    child: Icon(Icons.shopping_cart),
  ),
  label: 'السلة',
),
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **الغرض** | Bottom Nav مع حفظ الحالة |
| **الـ indexedStack** | أسهل implementation |
| **goBranch()** | التنقل بين الـ branches |
| **initialLocation** | للرجوع لأول الـ branch |
| **parentNavigatorKey** | للخروج من الـ shell |

---

[⬅️ الدرس السابق: ShellRoute](11_shell_routes.md) | [➡️ الدرس التالي: Transition Animations](13_transition_animations.md)
