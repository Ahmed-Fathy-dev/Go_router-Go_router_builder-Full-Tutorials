# الدرس 19: Parameters في GoRouter Builder

## أنواع الـ Parameters

| النوع | المكان في الـ URL | التعريف |
|-------|------------------|---------|
| **Path Parameters** | `/product/:id` | موجود في الـ path |
| **Query Parameters** | `?sort=price` | بعد `?` |
| **Extra Parameters** | غير ظاهر | `$extra` field |

---

## Path Parameters

الـ parameters اللي بتظهر في الـ path:

```dart
@TypedGoRoute<UserRoute>(path: '/user/:userId')
class UserRoute extends GoRouteData {
  const UserRoute({required this.userId});

  final String userId;  // 👈 Must match the name in the path

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return UserScreen(userId: userId);
  }
}

// Usage
UserRoute(userId: '123').go(context);
// URL: /user/123
```

### أكتر من Path Parameter

```dart
@TypedGoRoute<OrderRoute>(path: '/user/:userId/order/:orderId')
class OrderRoute extends GoRouteData {
  const OrderRoute({
    required this.userId,
    required this.orderId,
  });

  final String userId;
  final String orderId;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return OrderScreen(userId: userId, orderId: orderId);
  }
}

// Usage
OrderRoute(userId: '123', orderId: '456').go(context);
// URL: /user/123/order/456
```

---

## Query Parameters

أي field مش موجود في الـ path بيتحول لـ query parameter:

```dart
@TypedGoRoute<SearchRoute>(path: '/search')
class SearchRoute extends GoRouteData {
  const SearchRoute({
    this.query,
    this.category,
    this.page = 1,
    this.sortBy = 'relevance',
  });

  final String? query;      // ?query=flutter
  final String? category;   // &category=books
  final int page;           // &page=1 (default)
  final String sortBy;      // &sortBy=relevance (default)

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return SearchScreen(
      query: query,
      category: category,
      page: page,
      sortBy: sortBy,
    );
  }
}

// Usage
SearchRoute(
  query: 'flutter',
  category: 'books',
  page: 2,
).go(context);
// URL: /search?query=flutter&category=books&page=2&sortBy=relevance
```

### Nullable vs Non-nullable

```dart
// Optional (nullable) - Won't appear in URL if null
final String? category;

// Required with default - Will always appear in URL
final int page = 1;

// Required without default - Must be specified
// ⚠️ Cannot be required in query parameters
// Must be in the path, be nullable, or have a default
```

---

## Extra Parameters

لتمرير objects كاملة (مش هتظهر في الـ URL):

```dart
@TypedGoRoute<ProductRoute>(path: '/product/:id')
class ProductRoute extends GoRouteData {
  const ProductRoute({
    required this.id,
    this.$extra,  // 👈 Note the $ prefix
  });

  final int id;
  final Product? $extra;  // The full object

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductScreen(
      id: id,
      product: $extra,  // If available, use it
    );
  }
}

// Usage
ProductRoute(id: 123, $extra: product).go(context);
// URL: /product/123 (the product object is not visible)
```

---

## الأنواع المدعومة

### Primitive Types

```dart
final int count;           // 123
final double price;        // 99.99
final String name;         // 'test'
final bool active;         // true
final num value;           // 42
```

### Special Types

```dart
final BigInt bigNumber;    // BigInt.from(12345678901234567890)
final DateTime date;       // DateTime.now()
final Uri link;            // Uri.parse('https://example.com')
```

### Enums

```dart
enum SortOrder { asc, desc }
enum Status { pending, active, completed }

@TypedGoRoute<ListRoute>(path: '/list')
class ListRoute extends GoRouteData {
  const ListRoute({
    this.sortOrder = SortOrder.asc,
    this.status,
  });

  final SortOrder sortOrder;
  final Status? status;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ListScreen(sortOrder: sortOrder, status: status);
  }
}

// Usage
ListRoute(sortOrder: SortOrder.desc, status: Status.active).go(context);
// URL: /list?sortOrder=desc&status=active
```

### Collections

```dart
@TypedGoRoute<FilterRoute>(path: '/filter')
class FilterRoute extends GoRouteData {
  const FilterRoute({
    this.ids = const [],
    this.tags = const {},
  });

  final List<int> ids;
  final Set<String> tags;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return FilterScreen(ids: ids, tags: tags);
  }
}

// Usage
FilterRoute(
  ids: [1, 2, 3],
  tags: {'flutter', 'dart'},
).go(context);
// URL: /filter?ids=1&ids=2&ids=3&tags=flutter&tags=dart
```

---

## Default Values

```dart
@TypedGoRoute<ProductsRoute>(path: '/products')
class ProductsRoute extends GoRouteData {
  const ProductsRoute({
    this.page = 1,                    // Default: 1
    this.perPage = 20,                // Default: 20
    this.sortBy = 'popularity',       // Default: 'popularity'
    this.ascending = true,            // Default: true
    this.categories = const [],       // Default: empty list
  });

  final int page;
  final int perPage;
  final String sortBy;
  final bool ascending;
  final List<String> categories;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductsScreen(
      page: page,
      perPage: perPage,
      sortBy: sortBy,
      ascending: ascending,
      categories: categories,
    );
  }
}

// Without parameters - all values are default
const ProductsRoute().go(context);
// URL: /products?page=1&perPage=20&sortBy=popularity&ascending=true

// With some parameters
ProductsRoute(page: 2, categories: ['electronics']).go(context);
// URL: /products?page=2&perPage=20&sortBy=popularity&ascending=true&categories=electronics
```

---

## Path vs Query: متى تستخدم إيه؟

### استخدم Path Parameters لو:
- ✅ الـ parameter أساسي لتحديد الـ resource
- ✅ الـ parameter إجباري
- ✅ الـ URL لازم يكون معبر (SEO)

```dart
// ✅ Correct
/product/123           // The ID is essential
/user/ahmed/posts      // The username is essential
```

### استخدم Query Parameters لو:
- ✅ الـ parameter اختياري
- ✅ للفلترة والترتيب
- ✅ للـ pagination

```dart
// ✅ Correct
/products?category=phones&sort=price
/search?q=flutter&page=2
```

---

## مثال شامل

```dart
part 'routes.g.dart';

// ==================== Products ====================

enum SortField { price, rating, newest }
enum SortDirection { asc, desc }

@TypedGoRoute<ProductsRoute>(
  path: '/products',
  routes: [
    TypedGoRoute<ProductDetailsRoute>(path: ':productId'),
  ],
)
class ProductsRoute extends GoRouteData {
  const ProductsRoute({
    this.category,
    this.minPrice,
    this.maxPrice,
    this.inStock,
    this.sortField = SortField.newest,
    this.sortDirection = SortDirection.desc,
    this.page = 1,
    this.tags = const [],
  });

  // Filters
  final String? category;
  final double? minPrice;
  final double? maxPrice;
  final bool? inStock;

  // Sorting
  final SortField sortField;
  final SortDirection sortDirection;

  // Pagination
  final int page;

  // Multiple values
  final List<String> tags;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductsScreen(
      category: category,
      minPrice: minPrice,
      maxPrice: maxPrice,
      inStock: inStock,
      sortField: sortField,
      sortDirection: sortDirection,
      page: page,
      tags: tags,
    );
  }
}

class ProductDetailsRoute extends GoRouteData {
  const ProductDetailsRoute({
    required this.productId,
    this.showReviews = false,
    this.$extra,
  });

  // Path parameter
  final int productId;

  // Query parameter
  final bool showReviews;

  // Extra - The full product object
  final Product? $extra;

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductDetailsScreen(
      productId: productId,
      showReviews: showReviews,
      product: $extra,
    );
  }
}

// ==================== Usage ====================

// Basic navigation
const ProductsRoute().go(context);

// With filters
ProductsRoute(
  category: 'electronics',
  minPrice: 100,
  maxPrice: 500,
  inStock: true,
).go(context);

// With sorting and pagination
ProductsRoute(
  sortField: SortField.price,
  sortDirection: SortDirection.asc,
  page: 3,
  tags: ['sale', 'new'],
).go(context);

// To product details
ProductDetailsRoute(
  productId: 123,
  showReviews: true,
  $extra: productObject,
).push(context);
```

---

## ملخص

| النوع | الـ Syntax | الظهور في URL |
|-------|-----------|---------------|
| **Path** | في الـ `path:` | ✅ نعم |
| **Query** | أي field تاني | ✅ نعم |
| **Extra** | `$extra` | ❌ لا |

| النوع المدعوم | مثال |
|--------------|------|
| Primitives | int, double, String, bool |
| Special | BigInt, DateTime, Uri |
| Enum | SortOrder.asc |
| Collections | List, Set, Iterable |

---

[⬅️ الدرس السابق: TypedGoRoute](18_typed_routes.md) | [➡️ الدرس التالي: Shell Routes](20_builder_shell_routes.md)
