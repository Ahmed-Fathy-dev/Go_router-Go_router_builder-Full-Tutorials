# GoRouter & GoRouter Builder - Full Tutorials

> دليلك الشامل لإتقان ال Navigation في Flutter من الصفر للاحتراف

[![go_router](https://img.shields.io/badge/go__router-v17.0.1-blue)](https://pub.dev/packages/go_router)
[![go_router_builder](https://img.shields.io/badge/go__router__builder-v4.1.3-green)](https://pub.dev/packages/go_router_builder)
[![Flutter](https://img.shields.io/badge/Flutter-3.32+-purple)](https://flutter.dev)

---

## ليه GoRouter؟

لو بتستخدم Flutter وعايز تعمل navigation محترف، يبقى **GoRouter** هو الحل الأمثل. المكتبة دي بتوفرلك:

- **URL-based routing** - كل صفحة ليها URL واضح
- **Deep Linking** - المستخدم يقدر يفتح أي صفحة مباشرة من link
- **Type-safe routing** - مع GoRouter Builder بتتجنب أخطاء الـ runtime
- **Declarative approach** - بتعرف الـ routes بطريقة واضحة ومنظمة
- **Web support** - بيشتغل على الويب زي الموبايل بالظبط

---

## نظرة سريعة على المحتوى

### الجزء الأول: GoRouter Basics
> ابدأ من الصفر وافهم الأساسيات

| الدرس | الموضوع | الوصف |
|-------|---------|-------|
| 00 | [تاريخ Navigation](tutorials/01_go_router_basics/00_navigator_history.md) | Navigator 1.0 vs 2.0 وليه GoRouter |
| 01 | [المقدمة](tutorials/01_go_router_basics/01_introduction.md) | تعريف المكتبة ومميزاتها |
| 02 | [التثبيت والإعداد](tutorials/01_go_router_basics/02_installation.md) | إضافة المكتبة وإعداد المشروع |
| 03 | [التوجيه الأساسي](tutorials/01_go_router_basics/03_basic_routing.md) | `go()`, `push()`, `pop()` |
| 04 | [Path Parameters](tutorials/01_go_router_basics/04_path_parameters.md) | التعامل مع `:id` في الـ URL |
| 05 | [Query Parameters](tutorials/01_go_router_basics/05_query_parameters.md) | التعامل مع `?key=value` |
| 06 | [Named Routes](tutorials/01_go_router_basics/06_named_routes.md) | استخدام الأسماء بدل الـ paths |

### الجزء الثاني: GoRouter Intermediate
> خد خطوة للأمام وافهم المفاهيم المتوسطة

| الدرس | الموضوع | الوصف |
|-------|---------|-------|
| 07 | [Sub-Routes](tutorials/02_go_router_intermediate/07_sub_routes.md) | الـ routes المتداخلة |
| 08 | [Redirection](tutorials/02_go_router_intermediate/08_redirection.md) | إعادة التوجيه وحماية الصفحات |
| 09 | [Async Redirection](tutorials/02_go_router_intermediate/09_async_redirection.md) | التحقق غير المتزامن |
| 10 | [Error Handling](tutorials/02_go_router_intermediate/10_error_handling.md) | معالجة الأخطاء وصفحة 404 |

### الجزء الثالث: GoRouter Advanced
> اتقن المواضيع المتقدمة

| الدرس | الموضوع | الوصف |
|-------|---------|-------|
| 11 | [ShellRoute](tutorials/03_go_router_advanced/11_shell_routes.md) | الـ layouts المشتركة |
| 12 | [StatefulShellRoute](tutorials/03_go_router_advanced/12_stateful_shell_route.md) | Bottom Navigation مع حفظ الحالة |
| 13 | [Transition Animations](tutorials/03_go_router_advanced/13_transition_animations.md) | الحركات الانتقالية |
| 14 | [Deep Linking](tutorials/03_go_router_advanced/14_deep_linking.md) | الروابط العميقة للموبايل |
| 15 | [Extra Data & Codec](tutorials/03_go_router_advanced/15_extra_codec.md) | تمرير بيانات معقدة + `onEnter` |

### الجزء الرابع: GoRouter Builder
> Type-safe routing مع Code Generation

| الدرس | الموضوع | الوصف |
|-------|---------|-------|
| 16 | [مقدمة للـ Builder](tutorials/04_go_router_builder/16_builder_introduction.md) | ليه نستخدم الـ Builder؟ |
| 17 | [الإعداد](tutorials/04_go_router_builder/17_builder_setup.md) | Code Generation والإعداد |
| 18 | [TypedGoRoute](tutorials/04_go_router_builder/18_typed_routes.md) | إنشاء routes مكتوبة |
| 19 | [Parameters](tutorials/04_go_router_builder/19_builder_parameters.md) | Path, Query, Extra parameters |
| 20 | [Shell Routes](tutorials/04_go_router_builder/20_builder_shell_routes.md) | TypedShellRoute و TypedStatefulShellRoute |
| 21 | [ميزات متقدمة](tutorials/04_go_router_builder/21_builder_advanced.md) | Redirects, Transitions, Return values |

### الجزء الخامس: Best Practices
> أفضل الممارسات للمشاريع الحقيقية

| الدرس | الموضوع | الوصف |
|-------|---------|-------|
| 22 | [هيكلة المشروع](tutorials/05_best_practices/22_project_structure.md) | تنظيم ملفات الـ routing |
| 23 | [Common Patterns](tutorials/05_best_practices/23_common_patterns.md) | أنماط شائعة ومفيدة |
| 24 | [Testing](tutorials/05_best_practices/24_testing.md) | اختبار الـ routes |
| 25 | [Performance](tutorials/05_best_practices/25_performance.md) | تحسين الأداء |
| 26 | [Migration Guide](tutorials/05_best_practices/26_migration_guide.md) | الترقية بين الإصدارات |

### الإضافات
> موارد إضافية مفيدة

| الملف | الوصف |
|-------|-------|
| [Cheat Sheet](tutorials/extras/cheat_sheet.md) | ملخص سريع لكل الـ APIs |
| [Troubleshooting](tutorials/extras/troubleshooting.md) | حلول المشاكل الشائعة |
| [FAQ](tutorials/extras/faq.md) | أسئلة شائعة وإجاباتها |

---

## البدء السريع

### 1. أضف المكتبة

```yaml
dependencies:
  go_router: ^17.0.1
```

### 2. أنشئ الـ Router

```dart
final GoRouter router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/details/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return DetailsScreen(id: id);
      },
    ),
  ],
);
```

### 3. استخدمه في التطبيق

```dart
MaterialApp.router(
  routerConfig: router,
)
```

### 4. انتقل بين الصفحات

```dart
// الذهاب لصفحة (يمسح الـ stack)
context.go('/details/123');

// إضافة صفحة للـ stack
context.push('/details/123');

// الرجوع للخلف
context.pop();
```

---

## المتطلبات

| المتطلب | الإصدار |
|---------|---------|
| Flutter SDK | 3.32+ |
| Dart SDK | 3.9+ |
| go_router | 17.0.1 |
| go_router_builder | 4.1.3 |

---

## المصادر الرسمية

- [go_router على pub.dev](https://pub.dev/packages/go_router)
- [go_router_builder على pub.dev](https://pub.dev/packages/go_router_builder)
- [الـ Repository الرسمي](https://github.com/flutter/packages/tree/main/packages/go_router)
- [التوثيق الرسمي](https://pub.dev/documentation/go_router/latest/)

---

## المساهمة

لو لقيت أي خطأ أو عندك اقتراح، متتردش تفتح Issue أو Pull Request.

---

**صنع بـ ❤️ للمجتمع العربي**
