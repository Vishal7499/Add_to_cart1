# Component Development Guide

This guide provides detailed information about each component in the Flutter Shopping Cart application, including implementation details, customization options, and extension patterns.

## Table of Contents
- [Architecture Overview](#architecture-overview)
- [Component Breakdown](#component-breakdown)
- [Customization Guide](#customization-guide)
- [Extension Patterns](#extension-patterns)
- [Performance Considerations](#performance-considerations)

## Architecture Overview

The application follows a clean architecture pattern with clear separation of concerns:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   UI Layer      │    │  State Layer    │    │  Data Layer     │
│                 │    │                 │    │                 │
│ • ProductList   │◄───┤ • CartProvider  │◄───┤ • DBHelper      │
│ • CartScreen    │    │ • Notifications │    │ • Cart Model    │
│ • ReusableWidget│    │ • SharedPrefs   │    │ • SQLite DB     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Data Flow
1. **UI Events** → User interactions (tap, swipe, etc.)
2. **State Updates** → Provider methods update state
3. **Data Persistence** → Database operations via DBHelper
4. **UI Refresh** → Provider notifies listeners, UI rebuilds

## Component Breakdown

### 1. Main Application (`main.dart`)

#### Purpose
Entry point that sets up the Provider pattern and app configuration.

#### Key Responsibilities
- Provider initialization
- Theme configuration
- Navigation setup
- App-wide configuration

#### Implementation Details
```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => CartProvider(),
      child: Builder(
        builder: (BuildContext context) {
          return MaterialApp(
            title: 'Flutter Demo',
            debugShowCheckedModeBanner: false,
            theme: ThemeData(
              primarySwatch: Colors.blue,
            ),
            home: const ProductList(),
          );
        }
      ),
    );
  }
}
```

#### Customization Points
- **Theme**: Modify `ThemeData` for custom styling
- **Initial Route**: Change `home` property for different starting screen
- **Provider Setup**: Add multiple providers using `MultiProvider`

#### Extension Example
```dart
// Multi-provider setup
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => CartProvider()),
    ChangeNotifierProvider(create: (_) => UserProvider()),
    ChangeNotifierProvider(create: (_) => ThemeProvider()),
  ],
  child: Builder(
    builder: (context) => MaterialApp(
      theme: context.watch<ThemeProvider>().currentTheme,
      home: const ProductList(),
    ),
  ),
)
```

---

### 2. Database Helper (`db_helper.dart`)

#### Purpose
Manages all SQLite database operations for cart persistence.

#### Key Responsibilities
- Database initialization and schema creation
- CRUD operations for cart items
- Database connection management
- Error handling for database operations

#### Implementation Details

##### Singleton Pattern
```dart
class DBHelper {
  static Database? _db;
  
  Future<Database?> get db async {
    if (_db != null) {
      return _db!;
    }
    _db = await initDatabase();
    return _db;
  }
}
```

##### Table Schema
```sql
CREATE TABLE cart (
  id INTEGER PRIMARY KEY,
  productId VARCHAR UNIQUE,
  productName TEXT,
  initialPrice INTEGER,
  productPrice INTEGER,
  quantity INTEGER,
  unitTag TEXT,
  image TEXT
)
```

#### Customization Points

##### 1. Enhanced Schema
```dart
// Extended cart table with additional fields
_onCreate(Database db, int version) async {
  await db.execute('''
    CREATE TABLE cart (
      id INTEGER PRIMARY KEY,
      productId VARCHAR UNIQUE,
      productName TEXT,
      initialPrice INTEGER,
      productPrice INTEGER,
      quantity INTEGER,
      unitTag TEXT,
      image TEXT,
      category TEXT DEFAULT 'general',
      addedDate INTEGER,
      discount REAL DEFAULT 0.0,
      notes TEXT
    )
  ''');
}
```

##### 2. Advanced Query Methods
```dart
// Add to DBHelper class
Future<List<Cart>> getCartItemsByCategory(String category) async {
  var dbClient = await db;
  final List<Map<String, Object?>> queryResult = await dbClient!.query(
    'cart',
    where: 'unitTag = ?',
    whereArgs: [category],
  );
  return queryResult.map((e) => Cart.fromMap(e)).toList();
}

Future<double> getTotalByCategory(String category) async {
  var dbClient = await db;
  final result = await dbClient!.rawQuery(
    'SELECT SUM(productPrice) as total FROM cart WHERE unitTag = ?',
    [category],
  );
  return (result.first['total'] as num?)?.toDouble() ?? 0.0;
}

Future<List<Cart>> searchCartItems(String searchTerm) async {
  var dbClient = await db;
  final List<Map<String, Object?>> queryResult = await dbClient!.query(
    'cart',
    where: 'productName LIKE ?',
    whereArgs: ['%$searchTerm%'],
  );
  return queryResult.map((e) => Cart.fromMap(e)).toList();
}
```

---

### 3. Cart Model (`cart_model.dart`)

#### Purpose
Data structure representing cart items with serialization capabilities.

#### Key Responsibilities
- Define cart item properties
- Handle database serialization/deserialization
- Provide type safety for cart operations

#### Implementation Details

##### Properties Design
```dart
class Cart {
  late final int? id;           // Database primary key
  final String? productId;      // External product identifier
  final String? productName;    // Display name
  final int? initialPrice;      // Unit price (immutable)
  final int? productPrice;      // Total price (quantity * unit price)
  final int? quantity;          // Number of items
  final String? unitTag;        // Unit of measurement
  final String? image;          // Image URL/path
}
```

#### Customization Points

##### 1. Enhanced Model
```dart
class EnhancedCart extends Cart {
  final String? category;
  final DateTime? addedDate;
  final double? discount;
  final String? notes;
  final bool? isFavorite;
  
  EnhancedCart({
    required int? id,
    required String? productId,
    required String? productName,
    required int? initialPrice,
    required int? productPrice,
    required int? quantity,
    required String? unitTag,
    required String? image,
    this.category,
    this.addedDate,
    this.discount,
    this.notes,
    this.isFavorite = false,
  }) : super(
    id: id,
    productId: productId,
    productName: productName,
    initialPrice: initialPrice,
    productPrice: productPrice,
    quantity: quantity,
    unitTag: unitTag,
    image: image,
  );
  
  // Enhanced serialization
  @override
  Map<String, Object?> toMap() {
    Map<String, Object?> baseMap = super.toMap();
    baseMap.addAll({
      'category': category,
      'addedDate': addedDate?.millisecondsSinceEpoch,
      'discount': discount,
      'notes': notes,
      'isFavorite': isFavorite == true ? 1 : 0,
    });
    return baseMap;
  }
  
  // Enhanced deserialization
  EnhancedCart.fromMap(Map<dynamic, dynamic> res) : 
    category = res['category'],
    addedDate = res['addedDate'] != null 
      ? DateTime.fromMillisecondsSinceEpoch(res['addedDate'])
      : null,
    discount = res['discount']?.toDouble(),
    notes = res['notes'],
    isFavorite = res['isFavorite'] == 1,
    super.fromMap(res);
}
```

##### 2. Validation Methods
```dart
extension CartValidation on Cart {
  bool get isValid {
    return productId != null &&
           productName != null &&
           initialPrice != null &&
           quantity != null &&
           quantity! > 0 &&
           initialPrice! >= 0;
  }
  
  String? validate() {
    if (productId == null || productId!.isEmpty) {
      return 'Product ID is required';
    }
    if (productName == null || productName!.isEmpty) {
      return 'Product name is required';
    }
    if (quantity == null || quantity! <= 0) {
      return 'Quantity must be greater than 0';
    }
    if (initialPrice == null || initialPrice! < 0) {
      return 'Price must be non-negative';
    }
    return null; // Valid
  }
  
  Cart copyWith({
    int? id,
    String? productId,
    String? productName,
    int? initialPrice,
    int? productPrice,
    int? quantity,
    String? unitTag,
    String? image,
  }) {
    return Cart(
      id: id ?? this.id,
      productId: productId ?? this.productId,
      productName: productName ?? this.productName,
      initialPrice: initialPrice ?? this.initialPrice,
      productPrice: productPrice ?? this.productPrice,
      quantity: quantity ?? this.quantity,
      unitTag: unitTag ?? this.unitTag,
      image: image ?? this.image,
    );
  }
}
```

---

### 4. Cart Provider (`cartprovider.dart`)

#### Purpose
Central state management for cart operations using the Provider pattern.

#### Key Responsibilities
- Maintain cart counter and total price
- Coordinate with database operations
- Persist state using SharedPreferences
- Notify UI components of state changes

#### Implementation Details

##### State Variables
```dart
class CartProvider with ChangeNotifier {
  DBHelper db = DBHelper();
  int _counter = 0;              // Total items in cart
  double _totalPrice = 0.0;      // Total cart value
  late Future<List<Cart>> _cart; // Cart items future
}
```

##### Persistence Strategy
```dart
void _setPrefItems() async {
  SharedPreferences prefs = await SharedPreferences.getInstance();
  prefs.setInt('cart_item', _counter);
  prefs.setDouble('total_price', _totalPrice);
  notifyListeners();
}

void _getPrefItems() async {
  SharedPreferences prefs = await SharedPreferences.getInstance();
  _counter = prefs.getInt('cart_item') ?? 0;
  _totalPrice = prefs.getDouble('total_price') ?? 0.0;
  notifyListeners();
}
```

#### Customization Points

##### 1. Enhanced Provider with Categories
```dart
class EnhancedCartProvider extends CartProvider {
  Map<String, int> _categoryCount = {};
  Map<String, double> _categoryTotals = {};
  
  Map<String, int> get categoryCount => Map.unmodifiable(_categoryCount);
  Map<String, double> get categoryTotals => Map.unmodifiable(_categoryTotals);
  
  @override
  void addTotalPrice(double productPrice, {String? category}) {
    super.addTotalPrice(productPrice);
    
    if (category != null) {
      _categoryTotals[category] = (_categoryTotals[category] ?? 0) + productPrice;
      notifyListeners();
    }
  }
  
  void addCategoryItem(String category) {
    _categoryCount[category] = (_categoryCount[category] ?? 0) + 1;
    notifyListeners();
  }
  
  void removeCategoryItem(String category) {
    if (_categoryCount.containsKey(category) && _categoryCount[category]! > 0) {
      _categoryCount[category] = _categoryCount[category]! - 1;
      notifyListeners();
    }
  }
}
```

##### 2. Provider with Undo/Redo
```dart
class UndoableCartProvider extends CartProvider {
  final List<CartState> _history = [];
  int _currentIndex = -1;
  
  void saveState() {
    CartState currentState = CartState(
      counter: _counter,
      totalPrice: _totalPrice,
      timestamp: DateTime.now(),
    );
    
    // Remove any states after current index (for redo functionality)
    if (_currentIndex < _history.length - 1) {
      _history.removeRange(_currentIndex + 1, _history.length);
    }
    
    _history.add(currentState);
    _currentIndex = _history.length - 1;
  }
  
  bool canUndo() => _currentIndex > 0;
  bool canRedo() => _currentIndex < _history.length - 1;
  
  void undo() {
    if (canUndo()) {
      _currentIndex--;
      CartState previousState = _history[_currentIndex];
      _counter = previousState.counter;
      _totalPrice = previousState.totalPrice;
      _setPrefItems();
    }
  }
  
  void redo() {
    if (canRedo()) {
      _currentIndex++;
      CartState nextState = _history[_currentIndex];
      _counter = nextState.counter;
      _totalPrice = nextState.totalPrice;
      _setPrefItems();
    }
  }
}

class CartState {
  final int counter;
  final double totalPrice;
  final DateTime timestamp;
  
  CartState({
    required this.counter,
    required this.totalPrice,
    required this.timestamp,
  });
}
```

---

### 5. Product List Screen (`product_list.dart`)

#### Purpose
Main screen displaying available products in a grid layout with add-to-cart functionality.

#### Key Responsibilities
- Display product catalog
- Handle add-to-cart interactions
- Show cart badge with item counter
- Navigate to cart screen
- Provide user feedback

#### Implementation Details

##### Grid Layout Configuration
```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
    maxCrossAxisExtent: 300,    // Maximum width per item
    childAspectRatio: 0.7,      // Width to height ratio
    crossAxisSpacing: 10,       // Horizontal spacing
    mainAxisSpacing: 20         // Vertical spacing
  ),
  itemCount: productName.length,
  itemBuilder: (BuildContext context, int index) {
    // Product card implementation
  },
)
```

##### Product Data Structure
```dart
class _ProductListState extends State<ProductList> {
  List<String> productName = ['Mango', 'Orange', 'Grapes', /* ... */];
  List<String> productUnit = ['KG', 'Dozen', 'KG', /* ... */];
  List<int> productPrice = [10, 20, 30, /* ... */];
  List<String> productImage = [/* image URLs */];
}
```

#### Customization Points

##### 1. Dynamic Product Loading
```dart
class DynamicProductList extends StatefulWidget {
  @override
  _DynamicProductListState createState() => _DynamicProductListState();
}

class _DynamicProductListState extends State<DynamicProductList> {
  List<Product> products = [];
  bool isLoading = true;
  
  @override
  void initState() {
    super.initState();
    loadProducts();
  }
  
  Future<void> loadProducts() async {
    try {
      // Load from API or local JSON
      List<Product> loadedProducts = await ProductService.getProducts();
      setState(() {
        products = loadedProducts;
        isLoading = false;
      });
    } catch (e) {
      setState(() {
        isLoading = false;
      });
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Failed to load products: $e')),
      );
    }
  }
  
  @override
  Widget build(BuildContext context) {
    if (isLoading) {
      return Scaffold(
        appBar: AppBar(title: Text('Products')),
        body: Center(child: CircularProgressIndicator()),
      );
    }
    
    return Scaffold(
      appBar: AppBar(title: Text('Products')),
      body: GridView.builder(
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 0.75,
        ),
        itemCount: products.length,
        itemBuilder: (context, index) {
          return ProductCard(
            product: products[index],
            onAddToCart: () => addToCart(products[index]),
          );
        },
      ),
    );
  }
}

class Product {
  final String id;
  final String name;
  final int price;
  final String unit;
  final String imageUrl;
  final String category;
  final String description;
  
  Product({
    required this.id,
    required this.name,
    required this.price,
    required this.unit,
    required this.imageUrl,
    required this.category,
    required this.description,
  });
  
  factory Product.fromJson(Map<String, dynamic> json) {
    return Product(
      id: json['id'],
      name: json['name'],
      price: json['price'],
      unit: json['unit'],
      imageUrl: json['imageUrl'],
      category: json['category'],
      description: json['description'],
    );
  }
}
```

##### 2. Search and Filter Functionality
```dart
class SearchableProductList extends DynamicProductList {
  TextEditingController searchController = TextEditingController();
  List<Product> filteredProducts = [];
  String selectedCategory = 'All';
  
  void filterProducts(String query) {
    setState(() {
      if (query.isEmpty) {
        filteredProducts = products;
      } else {
        filteredProducts = products.where((product) =>
          product.name.toLowerCase().contains(query.toLowerCase()) ||
          product.category.toLowerCase().contains(query.toLowerCase())
        ).toList();
      }
    });
  }
  
  void filterByCategory(String category) {
    setState(() {
      selectedCategory = category;
      if (category == 'All') {
        filteredProducts = products;
      } else {
        filteredProducts = products.where((product) =>
          product.category == category
        ).toList();
      }
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Products'),
        bottom: PreferredSize(
          preferredSize: Size.fromHeight(120),
          child: Column(
            children: [
              // Search Bar
              Padding(
                padding: EdgeInsets.all(16),
                child: TextField(
                  controller: searchController,
                  decoration: InputDecoration(
                    hintText: 'Search products...',
                    prefixIcon: Icon(Icons.search),
                    border: OutlineInputBorder(
                      borderRadius: BorderRadius.circular(8),
                    ),
                  ),
                  onChanged: filterProducts,
                ),
              ),
              
              // Category Filter
              Container(
                height: 40,
                child: ListView(
                  scrollDirection: Axis.horizontal,
                  padding: EdgeInsets.symmetric(horizontal: 16),
                  children: ['All', 'KG', 'Dozen', 'Piece'].map((category) =>
                    Padding(
                      padding: EdgeInsets.only(right: 8),
                      child: FilterChip(
                        label: Text(category),
                        selected: selectedCategory == category,
                        onSelected: (_) => filterByCategory(category),
                      ),
                    ),
                  ).toList(),
                ),
              ),
            ],
          ),
        ),
      ),
      body: GridView.builder(
        padding: EdgeInsets.all(16),
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 0.75,
          crossAxisSpacing: 16,
          mainAxisSpacing: 16,
        ),
        itemCount: filteredProducts.length,
        itemBuilder: (context, index) {
          return ProductCard(
            product: filteredProducts[index],
            onAddToCart: () => addToCart(filteredProducts[index]),
          );
        },
      ),
    );
  }
}
```

---

### 6. Cart Screen (`cart_screen.dart`)

#### Purpose
Display cart items with quantity management and checkout functionality.

#### Key Responsibilities
- Show cart items in a list
- Handle quantity increment/decrement
- Calculate and display totals
- Manage item deletion
- Handle empty cart state

#### Implementation Details

##### FutureBuilder Pattern
```dart
FutureBuilder(
  future: cart.getData(),
  builder: (context, AsyncSnapshot<List<Cart>> snapshot) {
    if (snapshot.hasData) {
      if (snapshot.data!.isEmpty) {
        return EmptyCartWidget();
      } else {
        return CartItemsList(items: snapshot.data!);
      }
    }
    return LoadingWidget();
  },
)
```

#### Customization Points

##### 1. Enhanced Cart Screen with Sections
```dart
class EnhancedCartScreen extends StatefulWidget {
  @override
  _EnhancedCartScreenState createState() => _EnhancedCartScreenState();
}

class _EnhancedCartScreenState extends State<EnhancedCartScreen> {
  @override
  Widget build(BuildContext context) {
    final cart = Provider.of<CartProvider>(context);
    
    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping Cart'),
        actions: [
          IconButton(
            icon: Icon(Icons.clear_all),
            onPressed: () => showClearCartDialog(context),
          ),
        ],
      ),
      body: Column(
        children: [
          // Cart Items Section
          Expanded(
            child: FutureBuilder<List<Cart>>(
              future: cart.getData(),
              builder: (context, snapshot) {
                if (snapshot.hasData) {
                  if (snapshot.data!.isEmpty) {
                    return EmptyCartWidget();
                  }
                  
                  // Group items by category
                  Map<String, List<Cart>> groupedItems = groupItemsByCategory(snapshot.data!);
                  
                  return ListView.builder(
                    itemCount: groupedItems.length,
                    itemBuilder: (context, index) {
                      String category = groupedItems.keys.elementAt(index);
                      List<Cart> categoryItems = groupedItems[category]!;
                      
                      return CartCategorySection(
                        category: category,
                        items: categoryItems,
                      );
                    },
                  );
                }
                return Center(child: CircularProgressIndicator());
              },
            ),
          ),
          
          // Cart Summary Section
          CartSummarySection(),
          
          // Checkout Button
          CheckoutButtonSection(),
        ],
      ),
    );
  }
  
  Map<String, List<Cart>> groupItemsByCategory(List<Cart> items) {
    Map<String, List<Cart>> grouped = {};
    for (Cart item in items) {
      String category = item.unitTag ?? 'Other';
      grouped.putIfAbsent(category, () => []).add(item);
    }
    return grouped;
  }
}

class CartCategorySection extends StatelessWidget {
  final String category;
  final List<Cart> items;
  
  const CartCategorySection({
    Key? key,
    required this.category,
    required this.items,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    double categoryTotal = items.fold(0.0, (sum, item) => 
      sum + (item.productPrice?.toDouble() ?? 0));
    
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        // Category Header
        Container(
          padding: EdgeInsets.all(16),
          color: Colors.grey[100],
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text(
                category,
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
              Text(
                '\$${categoryTotal.toStringAsFixed(2)}',
                style: TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.w600,
                  color: Colors.green[700],
                ),
              ),
            ],
          ),
        ),
        
        // Category Items
        ...items.map((item) => CartItemWidget(
          cartItem: item,
          onDelete: () => deleteItem(context, item),
          onIncrement: () => incrementQuantity(context, item),
          onDecrement: () => decrementQuantity(context, item),
        )).toList(),
      ],
    );
  }
}
```

##### 2. Cart Summary with Animations
```dart
class AnimatedCartSummary extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<CartProvider>(
      builder: (context, cartProvider, child) {
        return AnimatedContainer(
          duration: Duration(milliseconds: 300),
          height: cartProvider.hasItems ? 200 : 0,
          child: Card(
            margin: EdgeInsets.all(16),
            child: Padding(
              padding: EdgeInsets.all(16),
              child: Column(
                children: [
                  SummaryRow(
                    title: 'Subtotal',
                    value: cartProvider.getFormattedTotal(),
                  ),
                  SummaryRow(
                    title: 'Tax (8%)',
                    value: '\$${(cartProvider.getTotalPrice() * 0.08).toStringAsFixed(2)}',
                  ),
                  SummaryRow(
                    title: 'Discount (5%)',
                    value: '-\$${(cartProvider.getTotalPrice() * 0.05).toStringAsFixed(2)}',
                    isDiscount: true,
                  ),
                  Divider(),
                  SummaryRow(
                    title: 'Total',
                    value: '\$${cartProvider.finalTotal.toStringAsFixed(2)}',
                    isBold: true,
                  ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }
}

class SummaryRow extends StatelessWidget {
  final String title;
  final String value;
  final bool isDiscount;
  final bool isBold;
  
  const SummaryRow({
    Key? key,
    required this.title,
    required this.value,
    this.isDiscount = false,
    this.isBold = false,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            title,
            style: TextStyle(
              fontSize: isBold ? 18 : 16,
              fontWeight: isBold ? FontWeight.bold : FontWeight.normal,
            ),
          ),
          Text(
            value,
            style: TextStyle(
              fontSize: isBold ? 18 : 16,
              fontWeight: isBold ? FontWeight.bold : FontWeight.normal,
              color: isDiscount ? Colors.red : (isBold ? Colors.green[700] : null),
            ),
          ),
        ],
      ),
    );
  }
}
```

---

### 7. Reusable Widget (`ReusableWidget`)

#### Purpose
Utility widget for displaying key-value pairs consistently across the app.

#### Key Responsibilities
- Display title-value pairs
- Maintain consistent styling
- Provide flexible layout options

#### Implementation Details
```dart
class ReusableWidget extends StatelessWidget {
  final String title, value;
  const ReusableWidget({required this.title, required this.value});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            title,
            style: Theme.of(context).textTheme.subtitle2,
          ),
          Text(
            value.toString(),
            style: Theme.of(context).textTheme.subtitle2,
          )
        ],
      ),
    );
  }
}
```

#### Extension Examples

##### 1. Enhanced Reusable Widget
```dart
class EnhancedReusableWidget extends StatelessWidget {
  final String title;
  final String value;
  final Color? titleColor;
  final Color? valueColor;
  final FontWeight? titleWeight;
  final FontWeight? valueWeight;
  final double? fontSize;
  final IconData? icon;
  final bool showDivider;
  
  const EnhancedReusableWidget({
    Key? key,
    required this.title,
    required this.value,
    this.titleColor,
    this.valueColor,
    this.titleWeight,
    this.valueWeight,
    this.fontSize,
    this.icon,
    this.showDivider = false,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Padding(
          padding: const EdgeInsets.symmetric(vertical: 8, horizontal: 4),
          child: Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Row(
                children: [
                  if (icon != null) ...[
                    Icon(icon, size: 20, color: titleColor),
                    SizedBox(width: 8),
                  ],
                  Text(
                    title,
                    style: TextStyle(
                      color: titleColor,
                      fontWeight: titleWeight ?? FontWeight.normal,
                      fontSize: fontSize ?? 16,
                    ),
                  ),
                ],
              ),
              Text(
                value,
                style: TextStyle(
                  color: valueColor,
                  fontWeight: valueWeight ?? FontWeight.normal,
                  fontSize: fontSize ?? 16,
                ),
              ),
            ],
          ),
        ),
        if (showDivider) Divider(height: 1),
      ],
    );
  }
}
```

##### 2. Animated Summary Widget
```dart
class AnimatedSummaryWidget extends StatefulWidget {
  final String title;
  final String value;
  final Duration animationDuration;
  
  const AnimatedSummaryWidget({
    Key? key,
    required this.title,
    required this.value,
    this.animationDuration = const Duration(milliseconds: 500),
  }) : super(key: key);
  
  @override
  _AnimatedSummaryWidgetState createState() => _AnimatedSummaryWidgetState();
}

class _AnimatedSummaryWidgetState extends State<AnimatedSummaryWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  late Animation<Color?> _colorAnimation;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: widget.animationDuration,
      vsync: this,
    );
    
    _scaleAnimation = Tween<double>(begin: 1.0, end: 1.1).animate(
      CurvedAnimation(parent: _controller, curve: Curves.elasticOut),
    );
    
    _colorAnimation = ColorTween(
      begin: Colors.black,
      end: Colors.green[700],
    ).animate(_controller);
  }
  
  @override
  void didUpdateWidget(AnimatedSummaryWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.value != widget.value) {
      _controller.forward().then((_) => _controller.reverse());
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Transform.scale(
          scale: _scaleAnimation.value,
          child: Padding(
            padding: EdgeInsets.symmetric(vertical: 4),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text(
                  widget.title,
                  style: Theme.of(context).textTheme.subtitle2,
                ),
                Text(
                  widget.value,
                  style: Theme.of(context).textTheme.subtitle2?.copyWith(
                    color: _colorAnimation.value,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

## Extension Patterns

### 1. Plugin Architecture

```dart
// Base plugin interface
abstract class CartPlugin {
  String get name;
  Future<void> initialize();
  Future<void> onCartChanged(List<Cart> items);
  Future<void> onItemAdded(Cart item);
  Future<void> onItemRemoved(Cart item);
}

// Analytics plugin
class AnalyticsCartPlugin implements CartPlugin {
  @override
  String get name => 'Analytics';
  
  @override
  Future<void> initialize() async {
    // Initialize analytics SDK
  }
  
  @override
  Future<void> onCartChanged(List<Cart> items) async {
    // Track cart changes
    await Analytics.track('cart_updated', {
      'item_count': items.length,
      'total_value': items.fold(0.0, (sum, item) => sum + item.productPrice!),
    });
  }
  
  @override
  Future<void> onItemAdded(Cart item) async {
    await Analytics.track('item_added_to_cart', {
      'product_id': item.productId,
      'product_name': item.productName,
      'price': item.initialPrice,
      'category': item.unitTag,
    });
  }
  
  @override
  Future<void> onItemRemoved(Cart item) async {
    await Analytics.track('item_removed_from_cart', {
      'product_id': item.productId,
      'product_name': item.productName,
    });
  }
}

// Plugin-enabled CartProvider
class PluginCartProvider extends CartProvider {
  final List<CartPlugin> _plugins = [];
  
  void registerPlugin(CartPlugin plugin) {
    _plugins.add(plugin);
    plugin.initialize();
  }
  
  @override
  Future<List<Cart>> getData() async {
    List<Cart> items = await super.getData();
    
    // Notify plugins
    for (CartPlugin plugin in _plugins) {
      await plugin.onCartChanged(items);
    }
    
    return items;
  }
  
  Future<void> addItemWithPlugins(Cart item) async {
    await db.insert(item);
    addCounter();
    addTotalPrice(item.productPrice!.toDouble());
    
    // Notify plugins
    for (CartPlugin plugin in _plugins) {
      await plugin.onItemAdded(item);
    }
  }
}
```

### 2. Middleware Pattern

```dart
// Cart middleware for cross-cutting concerns
abstract class CartMiddleware {
  Future<bool> beforeAddItem(Cart item);
  Future<void> afterAddItem(Cart item);
  Future<bool> beforeRemoveItem(Cart item);
  Future<void> afterRemoveItem(Cart item);
}

class ValidationMiddleware implements CartMiddleware {
  @override
  Future<bool> beforeAddItem(Cart item) async {
    // Validate item before adding
    if (item.quantity! <= 0) {
      throw Exception('Quantity must be positive');
    }
    if (item.initialPrice! < 0) {
      throw Exception('Price cannot be negative');
    }
    return true;
  }
  
  @override
  Future<void> afterAddItem(Cart item) async {
    print('Item validated and added: ${item.productName}');
  }
  
  @override
  Future<bool> beforeRemoveItem(Cart item) async {
    // Could add confirmation logic here
    return true;
  }
  
  @override
  Future<void> afterRemoveItem(Cart item) async {
    print('Item removed: ${item.productName}');
  }
}

class LoggingMiddleware implements CartMiddleware {
  @override
  Future<bool> beforeAddItem(Cart item) async {
    print('Attempting to add: ${item.productName}');
    return true;
  }
  
  @override
  Future<void> afterAddItem(Cart item) async {
    print('Successfully added: ${item.productName}');
  }
  
  @override
  Future<bool> beforeRemoveItem(Cart item) async {
    print('Attempting to remove: ${item.productName}');
    return true;
  }
  
  @override
  Future<void> afterRemoveItem(Cart item) async {
    print('Successfully removed: ${item.productName}');
  }
}

// Middleware-enabled CartProvider
class MiddlewareCartProvider extends CartProvider {
  final List<CartMiddleware> _middleware = [];
  
  void addMiddleware(CartMiddleware middleware) {
    _middleware.add(middleware);
  }
  
  Future<bool> addItemWithMiddleware(Cart item) async {
    // Run before middleware
    for (CartMiddleware middleware in _middleware) {
      bool canProceed = await middleware.beforeAddItem(item);
      if (!canProceed) return false;
    }
    
    try {
      // Perform the actual operation
      await db.insert(item);
      addCounter();
      addTotalPrice(item.productPrice!.toDouble());
      
      // Run after middleware
      for (CartMiddleware middleware in _middleware) {
        await middleware.afterAddItem(item);
      }
      
      return true;
    } catch (e) {
      print('Failed to add item: $e');
      return false;
    }
  }
}
```

## Performance Considerations

### 1. Optimized List Rendering

```dart
class OptimizedCartList extends StatefulWidget {
  final List<Cart> items;
  
  const OptimizedCartList({Key? key, required this.items}) : super(key: key);
  
  @override
  _OptimizedCartListState createState() => _OptimizedCartListState();
}

class _OptimizedCartListState extends State<OptimizedCartList> {
  final ScrollController _scrollController = ScrollController();
  
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      controller: _scrollController,
      itemCount: widget.items.length,
      cacheExtent: 1000, // Cache more items for smooth scrolling
      itemBuilder: (context, index) {
        return CartItemWidget(
          key: ValueKey(widget.items[index].id), // Stable keys for performance
          cartItem: widget.items[index],
          onDelete: () => _deleteItem(index),
          onIncrement: () => _incrementItem(index),
          onDecrement: () => _decrementItem(index),
        );
      },
    );
  }
  
  void _deleteItem(int index) {
    // Optimistic UI update
    setState(() {
      widget.items.removeAt(index);
    });
    
    // Perform database operation
    // If it fails, restore the item
  }
}
```

### 2. Image Caching Strategy

```dart
class CachedNetworkImageWidget extends StatelessWidget {
  final String imageUrl;
  final double? width;
  final double? height;
  final BoxFit fit;
  
  const CachedNetworkImageWidget({
    Key? key,
    required this.imageUrl,
    this.width,
    this.height,
    this.fit = BoxFit.cover,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Image.network(
      imageUrl,
      width: width,
      height: height,
      fit: fit,
      loadingBuilder: (context, child, loadingProgress) {
        if (loadingProgress == null) return child;
        
        return Container(
          width: width,
          height: height,
          child: Center(
            child: CircularProgressIndicator(
              value: loadingProgress.expectedTotalBytes != null
                ? loadingProgress.cumulativeBytesLoaded / 
                  loadingProgress.expectedTotalBytes!
                : null,
            ),
          ),
        );
      },
      errorBuilder: (context, error, stackTrace) {
        return Container(
          width: width,
          height: height,
          color: Colors.grey[300],
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.image_not_supported, color: Colors.grey[600]),
              Text('Image not available', style: TextStyle(fontSize: 12)),
            ],
          ),
        );
      },
    );
  }
}
```

### 3. Memory Management

```dart
// Efficient cart provider with proper disposal
class EfficientCartProvider extends CartProvider {
  Timer? _saveTimer;
  
  @override
  void addTotalPrice(double productPrice) {
    _totalPrice = _totalPrice + productPrice;
    _debouncedSave();
    notifyListeners();
  }
  
  // Debounced saving to avoid excessive SharedPreferences writes
  void _debouncedSave() {
    _saveTimer?.cancel();
    _saveTimer = Timer(Duration(milliseconds: 500), () {
      _setPrefItems();
    });
  }
  
  @override
  void dispose() {
    _saveTimer?.cancel();
    super.dispose();
  }
}
```

This component guide provides detailed implementation patterns and customization examples for each part of the Flutter shopping cart application. Use these patterns as starting points for extending and customizing the application to meet specific requirements.