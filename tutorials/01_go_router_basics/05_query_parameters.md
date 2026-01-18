# الدرس 5: Query Parameters

## يعني إيه Query Parameters؟

الـ Query Parameters هي البيانات اللي بتيجي بعد علامة `?` في الـ URL. بنستخدمها للبيانات الاختيارية زي الفلاتر والترتيب والبحث.

### أمثلة:
```
/products?category=electronics
/search?q=flutter&sort=recent&page=2
/users?role=admin&status=active
```

### الفرق بين Path Parameters و Query Parameters

| الخاصية | Path Parameters | Query Parameters |
|---------|-----------------|------------------|
| **الموقع** | جزء من الـ path | بعد علامة `?` |
| **الإجبارية** | عادةً إجباري | عادةً اختياري |
| **الاستخدام** | تحديد resource معين | فلترة وترتيب وبحث |
| **المثال** | `/user/123` | `/users?role=admin` |

---

## الوصول للـ Query Parameters

### الطريقة الأساسية

```dart
GoRoute(
  path: '/search',
  builder: (context, state) {
    // الحصول على الـ query parameters
    final queryParams = state.uri.queryParameters;

    // قيمة معينة
    final searchQuery = queryParams['q'];
    final sortBy = queryParams['sort'];
    final page = queryParams['page'];

    return SearchScreen(
      query: searchQuery ?? '',
      sortBy: sortBy ?? 'relevance',
      page: int.tryParse(page ?? '1') ?? 1,
    );
  },
)
```

### خصائص state.uri

```dart
GoRoute(
  path: '/products',
  builder: (context, state) {
    // الـ URI الكامل
    print(state.uri);  // /products?category=phones&brand=samsung

    // الـ path بدون query
    print(state.uri.path);  // /products

    // كل الـ query parameters كـ Map
    print(state.uri.queryParameters);
    // {category: phones, brand: samsung}

    // الـ query string كامل
    print(state.uri.query);  // category=phones&brand=samsung

    return ProductsScreen(...);
  },
)
```

---

## التنقل مع Query Parameters

### طريقة 1: String مباشر

```dart
// parameter واحد
context.go('/search?q=flutter');

// أكتر من parameter
context.go('/products?category=phones&brand=apple&sort=price');

// مع متغيرات
final query = 'flutter widgets';
final sort = 'recent';
context.go('/search?q=$query&sort=$sort');
```

### طريقة 2: باستخدام Uri

```dart
// أفضل للتعامل مع القيم المعقدة
final uri = Uri(
  path: '/search',
  queryParameters: {
    'q': 'flutter widgets',
    'sort': 'recent',
    'page': '1',
  },
);

context.go(uri.toString());
// النتيجة: /search?q=flutter%20widgets&sort=recent&page=1
```

### طريقة 3: مع Path Parameters

```dart
// دمج الاتنين مع بعض
context.go('/category/electronics?sort=price&order=asc');

// أو باستخدام Uri
final uri = Uri(
  path: '/category/$categoryId',
  queryParameters: {
    'sort': 'price',
    'order': 'asc',
  },
);
context.go(uri.toString());
```

---

## التعامل مع قيم متعددة لنفس الـ Parameter

لو عندك parameter بأكتر من قيمة:

```
/products?color=red&color=blue&color=green
```

### الحل باستخدام queryParametersAll

```dart
GoRoute(
  path: '/products',
  builder: (context, state) {
    // queryParameters بيرجع قيمة واحدة بس
    final singleColor = state.uri.queryParameters['color'];
    print(singleColor);  // green (آخر قيمة)

    // queryParametersAll بيرجع كل القيم
    final allColors = state.uri.queryParametersAll['color'];
    print(allColors);  // [red, blue, green]

    return ProductsScreen(colors: allColors ?? []);
  },
)
```

---

## مثال عملي: صفحة بحث متقدمة

### تعريف الـ Route

```dart
GoRoute(
  path: '/products',
  builder: (context, state) {
    final params = state.uri.queryParameters;

    return ProductsScreen(
      category: params['category'],
      brand: params['brand'],
      minPrice: double.tryParse(params['min'] ?? ''),
      maxPrice: double.tryParse(params['max'] ?? ''),
      sortBy: params['sort'] ?? 'popularity',
      page: int.tryParse(params['page'] ?? '1') ?? 1,
      inStock: params['inStock'] == 'true',
    );
  },
)
```

### الـ Screen مع الفلاتر

```dart
class ProductsScreen extends StatefulWidget {
  final String? category;
  final String? brand;
  final double? minPrice;
  final double? maxPrice;
  final String sortBy;
  final int page;
  final bool inStock;

  const ProductsScreen({
    super.key,
    this.category,
    this.brand,
    this.minPrice,
    this.maxPrice,
    this.sortBy = 'popularity',
    this.page = 1,
    this.inStock = false,
  });

  @override
  State<ProductsScreen> createState() => _ProductsScreenState();
}

class _ProductsScreenState extends State<ProductsScreen> {
  late String? _category;
  late String? _brand;
  late String _sortBy;
  late bool _inStock;

  @override
  void initState() {
    super.initState();
    _category = widget.category;
    _brand = widget.brand;
    _sortBy = widget.sortBy;
    _inStock = widget.inStock;
  }

  void _applyFilters() {
    // بناء الـ query parameters
    final params = <String, String>{};

    if (_category != null) params['category'] = _category!;
    if (_brand != null) params['brand'] = _brand!;
    params['sort'] = _sortBy;
    if (_inStock) params['inStock'] = 'true';
    params['page'] = '1'; // Reset to first page

    final uri = Uri(path: '/products', queryParameters: params);
    context.go(uri.toString());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('المنتجات'),
        actions: [
          IconButton(
            icon: const Icon(Icons.filter_list),
            onPressed: () => _showFilterSheet(context),
          ),
        ],
      ),
      body: Column(
        children: [
          // شريط الفلاتر النشطة
          if (_category != null || _brand != null || _inStock)
            _buildActiveFilters(),

          // قائمة المنتجات
          Expanded(
            child: _buildProductList(),
          ),
        ],
      ),
    );
  }

  Widget _buildActiveFilters() {
    return Container(
      height: 50,
      padding: const EdgeInsets.symmetric(horizontal: 8),
      child: ListView(
        scrollDirection: Axis.horizontal,
        children: [
          if (_category != null)
            Padding(
              padding: const EdgeInsets.all(4),
              child: Chip(
                label: Text(_category!),
                onDeleted: () {
                  _category = null;
                  _applyFilters();
                },
              ),
            ),
          if (_brand != null)
            Padding(
              padding: const EdgeInsets.all(4),
              child: Chip(
                label: Text(_brand!),
                onDeleted: () {
                  _brand = null;
                  _applyFilters();
                },
              ),
            ),
          if (_inStock)
            Padding(
              padding: const EdgeInsets.all(4),
              child: Chip(
                label: const Text('متوفر فقط'),
                onDeleted: () {
                  _inStock = false;
                  _applyFilters();
                },
              ),
            ),
        ],
      ),
    );
  }

  Widget _buildProductList() {
    // هنا هتجيب المنتجات من الـ API
    return ListView.builder(
      itemCount: 20,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text('منتج ${index + 1}'),
          subtitle: Text('${_category ?? 'الكل'} - ${_brand ?? 'كل الماركات'}'),
          onTap: () => context.push('/product/$index'),
        );
      },
    );
  }

  void _showFilterSheet(BuildContext context) {
    showModalBottomSheet(
      context: context,
      builder: (context) {
        return StatefulBuilder(
          builder: (context, setSheetState) {
            return Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Text(
                    'الفلاتر',
                    style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 16),

                  // Category Dropdown
                  DropdownButtonFormField<String>(
                    value: _category,
                    decoration: const InputDecoration(labelText: 'الفئة'),
                    items: const [
                      DropdownMenuItem(value: null, child: Text('الكل')),
                      DropdownMenuItem(value: 'electronics', child: Text('إلكترونيات')),
                      DropdownMenuItem(value: 'clothing', child: Text('ملابس')),
                      DropdownMenuItem(value: 'home', child: Text('منزل')),
                    ],
                    onChanged: (value) => setSheetState(() => _category = value),
                  ),

                  const SizedBox(height: 16),

                  // Sort Dropdown
                  DropdownButtonFormField<String>(
                    value: _sortBy,
                    decoration: const InputDecoration(labelText: 'الترتيب'),
                    items: const [
                      DropdownMenuItem(value: 'popularity', child: Text('الأكثر شعبية')),
                      DropdownMenuItem(value: 'price_asc', child: Text('السعر: من الأقل')),
                      DropdownMenuItem(value: 'price_desc', child: Text('السعر: من الأعلى')),
                      DropdownMenuItem(value: 'newest', child: Text('الأحدث')),
                    ],
                    onChanged: (value) => setSheetState(() => _sortBy = value!),
                  ),

                  const SizedBox(height: 16),

                  // In Stock Checkbox
                  CheckboxListTile(
                    title: const Text('متوفر فقط'),
                    value: _inStock,
                    onChanged: (value) => setSheetState(() => _inStock = value!),
                  ),

                  const SizedBox(height: 16),

                  // Apply Button
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: () {
                        Navigator.pop(context);
                        _applyFilters();
                      },
                      child: const Text('تطبيق الفلاتر'),
                    ),
                  ),
                ],
              ),
            );
          },
        );
      },
    );
  }
}
```

---

## التعامل مع URL Encoding

القيم اللي فيها مسافات أو رموز خاصة لازم تتعمل لها encoding:

```dart
// ❌ غلط - هيسبب مشاكل
context.go('/search?q=flutter widgets');

// ✅ صح - باستخدام Uri
final uri = Uri(
  path: '/search',
  queryParameters: {'q': 'flutter widgets'},
);
context.go(uri.toString());
// النتيجة: /search?q=flutter%20widgets

// ✅ صح - باستخدام Uri.encodeComponent
final query = Uri.encodeComponent('flutter widgets');
context.go('/search?q=$query');
```

---

## Pagination مع Query Parameters

### مثال عملي

```dart
class PaginatedListScreen extends StatelessWidget {
  final int page;
  final int perPage;

  const PaginatedListScreen({
    super.key,
    this.page = 1,
    this.perPage = 20,
  });

  void _goToPage(BuildContext context, int newPage) {
    final uri = Uri(
      path: '/items',
      queryParameters: {
        'page': newPage.toString(),
        'perPage': perPage.toString(),
      },
    );
    context.go(uri.toString());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('الصفحة $page')),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: perPage,
              itemBuilder: (context, index) {
                final itemNumber = (page - 1) * perPage + index + 1;
                return ListTile(title: Text('عنصر $itemNumber'));
              },
            ),
          ),

          // Pagination Controls
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                IconButton(
                  onPressed: page > 1 ? () => _goToPage(context, page - 1) : null,
                  icon: const Icon(Icons.chevron_left),
                ),
                Text('صفحة $page'),
                IconButton(
                  onPressed: () => _goToPage(context, page + 1),
                  icon: const Icon(Icons.chevron_right),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

// الـ Route
GoRoute(
  path: '/items',
  builder: (context, state) {
    final params = state.uri.queryParameters;
    return PaginatedListScreen(
      page: int.tryParse(params['page'] ?? '1') ?? 1,
      perPage: int.tryParse(params['perPage'] ?? '20') ?? 20,
    );
  },
)
```

---

## نصائح مهمة

### 1. استخدم Uri لبناء الـ URLs

```dart
// ✅ آمن وبيعمل encoding صح
final uri = Uri(
  path: '/search',
  queryParameters: {
    'q': userInput,  // حتى لو فيه رموز خاصة
    'page': '1',
  },
);
context.go(uri.toString());
```

### 2. اعمل Default Values

```dart
// ✅ دايماً وفر قيم افتراضية
final sortBy = params['sort'] ?? 'default';
final page = int.tryParse(params['page'] ?? '1') ?? 1;
```

### 3. Preserve Query Parameters عند التنقل

```dart
// لو عايز تحافظ على الفلاتر وتغير الصفحة بس
void goToNextPage(BuildContext context, GoRouterState state) {
  final currentParams = Map<String, String>.from(state.uri.queryParameters);
  final currentPage = int.tryParse(currentParams['page'] ?? '1') ?? 1;

  currentParams['page'] = (currentPage + 1).toString();

  final uri = Uri(path: state.uri.path, queryParameters: currentParams);
  context.go(uri.toString());
}
```

---

## ملخص

| العنصر | الوصف |
|--------|-------|
| **الـ Syntax** | `?key=value&key2=value2` |
| **الوصول** | `state.uri.queryParameters` |
| **قيم متعددة** | `state.uri.queryParametersAll` |
| **البناء** | استخدم `Uri()` |
| **الاستخدام** | الفلاتر، الترتيب، البحث، Pagination |

---

[⬅️ الدرس السابق: Path Parameters](04_path_parameters.md) | [➡️ الدرس التالي: Named Routes](06_named_routes.md)
