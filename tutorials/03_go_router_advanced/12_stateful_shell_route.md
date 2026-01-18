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

**StatefulShellRoute** بيحل المشكلة دي عن طريق:
- إنشاء Navigator **منفصل** لكل branch
- حفظ الحالة لكل tab
- الـ screens مش بتتدمر لما تتنقل لـ tab تاني

---

## StatefulShellRoute.indexedStack

أسهل طريقة تستخدم `StatefulShellRoute` هي عن طريق `indexedStack`:

```dart
StatefulShellRoute.indexedStack(
  builder: (context, state, navigationShell) {
    return ScaffoldWithNavBar(navigationShell: navigationShell);
  },
  branches: [
    // Branch A - الرئيسية
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

    // Branch B - البحث
    StatefulShellBranch(
      routes: [
        GoRoute(
          path: '/search',
          builder: (context, state) => const SearchScreen(),
        ),
      ],
    ),

    // Branch C - الملف الشخصي
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
    // الـ Stateful Shell للـ Bottom Navigation
    StatefulShellRoute.indexedStack(
      builder: (context, state, navigationShell) {
        return ScaffoldWithNavBar(navigationShell: navigationShell);
      },
      branches: [
        // الرئيسية
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

        // التصنيفات
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

        // السلة
        StatefulShellBranch(
          routes: [
            GoRoute(
              path: '/cart',
              builder: (context, state) => const CartScreen(),
            ),
          ],
        ),

        // حسابي
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

    // Routes خارج الـ Shell
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
      body: navigationShell,  // 👈 ده بيعرض الـ branch الحالي
      bottomNavigationBar: NavigationBar(
        selectedIndex: navigationShell.currentIndex,
        onDestinationSelected: (index) {
          // 👈 الـ method دي بتنقل بين الـ branches
          navigationShell.goBranch(
            index,
            // لو ضغط على نفس الـ tab، ارجعه للـ initial location
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
  index,                    // رقم الـ branch (0, 1, 2, ...)
  initialLocation: false,   // هل يروح للـ initial location؟
);
```

### السلوك:
- `initialLocation: false` (default) - يروح للـ branch ويحافظ على الـ stack
- `initialLocation: true` - يروح للـ initial location بتاع الـ branch (يمسح الـ stack)

```dart
// مثال: لما المستخدم يضغط على الـ tab الحالي
onDestinationSelected: (index) {
  if (index == navigationShell.currentIndex) {
    // ضغط على نفس الـ tab -> ارجعه للأول
    navigationShell.goBranch(index, initialLocation: true);
  } else {
    // tab مختلف -> انتقل وحافظ على الـ state
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

// استخدامه
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
    // children هي List<Widget> لكل branch
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

  // ده بيخلي الـ widget يتحفظ
  @override
  bool get wantKeepAlive => true;

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    super.build(context);  // مهم لـ AutomaticKeepAliveClientMixin

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
// التنقل داخل نفس الـ branch
context.go('/home/product/123');  // push داخل الـ home branch

// أو
context.push('/home/product/123');

// الرجوع
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
                // ده داخل الـ shell
                GoRoute(
                  path: 'product/:id',
                  builder: (context, state) => ProductScreen(...),
                ),
                // ده خارج الـ shell (full screen)
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
        // Shell جوه الـ home branch
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
    super.build(context);  // مهم!
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
