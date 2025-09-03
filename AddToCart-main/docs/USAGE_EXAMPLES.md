# Usage Examples and Code Patterns

## Table of Contents
- [Basic Setup](#basic-setup)
- [Database Operations](#database-operations)
- [State Management Patterns](#state-management-patterns)
- [UI Integration Examples](#ui-integration-examples)
- [Advanced Use Cases](#advanced-use-cases)
- [Best Practices](#best-practices)

## Basic Setup

### 1. Provider Setup in Main App

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:cartapp/provider/cartprovider.dart';
import 'package:cartapp/screen/product_list.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => CartProvider(),
      child: Builder(
        builder: (BuildContext context) {
          return MaterialApp(
            title: 'Shopping Cart App',
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

### 2. Database Initialization

```dart
import 'package:cartapp/helper/db_helper.dart';

class DatabaseManager {
  static final DatabaseManager _instance = DatabaseManager._internal();
  factory DatabaseManager() => _instance;
  DatabaseManager._internal();
  
  DBHelper? _dbHelper;
  
  Future<DBHelper> get dbHelper async {
    _dbHelper ??= DBHelper();
    await _dbHelper!.db; // Ensure database is initialized
    return _dbHelper!;
  }
}

// Usage
DatabaseManager dbManager = DatabaseManager();
DBHelper dbHelper = await dbManager.dbHelper;
```

## Database Operations

### 1. Creating and Inserting Cart Items

```dart
import 'package:cartapp/model/cart_model.dart';
import 'package:cartapp/helper/db_helper.dart';

class CartOperations {
  final DBHelper dbHelper = DBHelper();
  
  // Create a new cart item
  Cart createCartItem({
    required int id,
    required String productId,
    required String productName,
    required int price,
    required String unit,
    required String imageUrl,
    int quantity = 1,
  }) {
    return Cart(
      id: id,
      productId: productId,
      productName: productName,
      initialPrice: price,
      productPrice: price * quantity,
      quantity: quantity,
      unitTag: unit,
      image: imageUrl,
    );
  }
  
  // Add item to cart with error handling
  Future<bool> addToCart(Cart item) async {
    try {
      await dbHelper.insert(item);
      return true;
    } catch (e) {
      print('Failed to add item to cart: $e');
      return false;
    }
  }
  
  // Bulk insert multiple items
  Future<List<Cart>> addMultipleItems(List<Cart> items) async {
    List<Cart> successfulInserts = [];
    
    for (Cart item in items) {
      try {
        Cart insertedItem = await dbHelper.insert(item);
        successfulInserts.add(insertedItem);
      } catch (e) {
        print('Failed to insert ${item.productName}: $e');
      }
    }
    
    return successfulInserts;
  }
}
```

### 2. Advanced Database Queries

```dart
class AdvancedCartOperations extends CartOperations {
  
  // Get cart items with filtering
  Future<List<Cart>> getCartItemsByCategory(String unitTag) async {
    List<Cart> allItems = await dbHelper.getCartList();
    return allItems.where((item) => item.unitTag == unitTag).toList();
  }
  
  // Calculate category totals
  Future<Map<String, double>> getCategoryTotals() async {
    List<Cart> allItems = await dbHelper.getCartList();
    Map<String, double> categoryTotals = {};
    
    for (Cart item in allItems) {
      String category = item.unitTag ?? 'Unknown';
      double itemTotal = (item.productPrice ?? 0).toDouble();
      categoryTotals[category] = (categoryTotals[category] ?? 0) + itemTotal;
    }
    
    return categoryTotals;
  }
  
  // Clear entire cart
  Future<void> clearCart() async {
    List<Cart> allItems = await dbHelper.getCartList();
    for (Cart item in allItems) {
      await dbHelper.delete(item.id!);
    }
  }
  
  // Update multiple items at once
  Future<void> updateMultipleQuantities(Map<int, int> updates) async {
    List<Cart> currentItems = await dbHelper.getCartList();
    
    for (Cart item in currentItems) {
      if (updates.containsKey(item.id)) {
        int newQuantity = updates[item.id!]!;
        Cart updatedItem = Cart(
          id: item.id,
          productId: item.productId,
          productName: item.productName,
          initialPrice: item.initialPrice,
          productPrice: item.initialPrice! * newQuantity,
          quantity: newQuantity,
          unitTag: item.unitTag,
          image: item.image,
        );
        await dbHelper.updateQuantity(updatedItem);
      }
    }
  }
}
```

## State Management Patterns

### 1. Provider Integration with UI

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:cartapp/provider/cartprovider.dart';

class CartBadgeWidget extends StatelessWidget {
  final Widget child;
  final VoidCallback? onTap;
  
  const CartBadgeWidget({
    Key? key,
    required this.child,
    this.onTap,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Consumer<CartProvider>(
      builder: (context, cartProvider, _) {
        return InkWell(
          onTap: onTap,
          child: Badge(
            showBadge: cartProvider.getCounter() > 0,
            badgeContent: Text(
              cartProvider.getCounter().toString(),
              style: TextStyle(color: Colors.white, fontSize: 12),
            ),
            badgeColor: Colors.red,
            animationType: BadgeAnimationType.fade,
            animationDuration: Duration(milliseconds: 300),
            child: child,
          ),
        );
      },
    );
  }
}

// Usage
CartBadgeWidget(
  onTap: () => Navigator.push(
    context, 
    MaterialPageRoute(builder: (_) => CartScreen())
  ),
  child: Icon(Icons.shopping_cart),
)
```

### 2. Custom Provider Methods

```dart
// Extend CartProvider with additional functionality
extension CartProviderExtensions on CartProvider {
  
  // Get formatted total price
  String getFormattedTotal() {
    return '\$${getTotalPrice().toStringAsFixed(2)}';
  }
  
  // Check if cart is empty
  bool get isEmpty => getCounter() == 0;
  
  // Get cart summary
  Map<String, dynamic> getCartSummary() {
    return {
      'itemCount': getCounter(),
      'totalPrice': getTotalPrice(),
      'formattedTotal': getFormattedTotal(),
      'isEmpty': isEmpty,
    };
  }
  
  // Batch operations
  Future<void> batchUpdateCart(List<Cart> items) async {
    for (Cart item in items) {
      addTotalPrice(item.productPrice!.toDouble());
      addCounter();
    }
  }
}
```

### 3. Multiple Provider Setup

```dart
// For larger apps with multiple providers
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => CartProvider()),
    ChangeNotifierProvider(create: (_) => UserProvider()),
    ChangeNotifierProvider(create: (_) => ProductProvider()),
  ],
  child: MyApp(),
)
```

## UI Integration Examples

### 1. Custom Product Card

```dart
class ProductCard extends StatelessWidget {
  final String name;
  final int price;
  final String unit;
  final String imageUrl;
  final VoidCallback onAddToCart;
  
  const ProductCard({
    Key? key,
    required this.name,
    required this.price,
    required this.unit,
    required this.imageUrl,
    required this.onAddToCart,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 4,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Product Image
          Expanded(
            flex: 3,
            child: Container(
              width: double.infinity,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.vertical(top: Radius.circular(12)),
                image: DecorationImage(
                  image: NetworkImage(imageUrl),
                  fit: BoxFit.cover,
                  onError: (exception, stackTrace) {
                    // Handle image loading error
                  },
                ),
              ),
            ),
          ),
          
          // Product Details
          Expanded(
            flex: 2,
            child: Padding(
              padding: EdgeInsets.all(12),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  Text(
                    name,
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),
                  
                  Text(
                    '$unit \$${price}',
                    style: TextStyle(
                      fontSize: 14,
                      color: Colors.green[700],
                      fontWeight: FontWeight.w600,
                    ),
                  ),
                  
                  SizedBox(
                    width: double.infinity,
                    child: ElevatedButton(
                      onPressed: onAddToCart,
                      style: ElevatedButton.styleFrom(
                        backgroundColor: Colors.green,
                        shape: RoundedRectangleBorder(
                          borderRadius: BorderRadius.circular(8),
                        ),
                      ),
                      child: Text(
                        'Add to Cart',
                        style: TextStyle(color: Colors.white),
                      ),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

### 2. Enhanced Cart Item Widget

```dart
class CartItemWidget extends StatelessWidget {
  final Cart cartItem;
  final VoidCallback onDelete;
  final VoidCallback onIncrement;
  final VoidCallback onDecrement;
  
  const CartItemWidget({
    Key? key,
    required this.cartItem,
    required this.onDelete,
    required this.onIncrement,
    required this.onDecrement,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      margin: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Padding(
        padding: EdgeInsets.all(12),
        child: Row(
          children: [
            // Product Image
            ClipRRect(
              borderRadius: BorderRadius.circular(8),
              child: Image.network(
                cartItem.image!,
                width: 80,
                height: 80,
                fit: BoxFit.cover,
                errorBuilder: (context, error, stackTrace) {
                  return Container(
                    width: 80,
                    height: 80,
                    color: Colors.grey[300],
                    child: Icon(Icons.image_not_supported),
                  );
                },
              ),
            ),
            
            SizedBox(width: 12),
            
            // Product Details
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    cartItem.productName!,
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                    ),
                  ),
                  SizedBox(height: 4),
                  Text(
                    '${cartItem.unitTag} \$${cartItem.initialPrice}',
                    style: TextStyle(
                      fontSize: 14,
                      color: Colors.grey[600],
                    ),
                  ),
                  SizedBox(height: 8),
                  Text(
                    'Total: \$${cartItem.productPrice}',
                    style: TextStyle(
                      fontSize: 16,
                      fontWeight: FontWeight.bold,
                      color: Colors.green[700],
                    ),
                  ),
                ],
              ),
            ),
            
            // Quantity Controls
            Column(
              children: [
                // Delete Button
                IconButton(
                  onPressed: onDelete,
                  icon: Icon(Icons.delete, color: Colors.red),
                ),
                
                // Quantity Controls
                Container(
                  decoration: BoxDecoration(
                    color: Colors.green,
                    borderRadius: BorderRadius.circular(20),
                  ),
                  child: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      IconButton(
                        onPressed: onDecrement,
                        icon: Icon(Icons.remove, color: Colors.white),
                        constraints: BoxConstraints(minWidth: 32, minHeight: 32),
                      ),
                      Container(
                        padding: EdgeInsets.symmetric(horizontal: 12),
                        child: Text(
                          cartItem.quantity.toString(),
                          style: TextStyle(
                            color: Colors.white,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ),
                      IconButton(
                        onPressed: onIncrement,
                        icon: Icon(Icons.add, color: Colors.white),
                        constraints: BoxConstraints(minWidth: 32, minHeight: 32),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

## Database Operations

### 1. Safe Database Operations with Error Handling

```dart
class SafeCartOperations {
  final DBHelper dbHelper = DBHelper();
  
  // Safe insert with detailed error handling
  Future<CartOperationResult> safeInsert(Cart cart) async {
    try {
      Cart insertedCart = await dbHelper.insert(cart);
      return CartOperationResult.success(
        message: 'Item added to cart successfully',
        data: insertedCart,
      );
    } on DatabaseException catch (e) {
      if (e.isUniqueConstraintError()) {
        return CartOperationResult.error(
          message: 'Product already exists in cart',
          errorType: CartErrorType.duplicateProduct,
        );
      }
      return CartOperationResult.error(
        message: 'Database error: ${e.toString()}',
        errorType: CartErrorType.databaseError,
      );
    } catch (e) {
      return CartOperationResult.error(
        message: 'Unexpected error: ${e.toString()}',
        errorType: CartErrorType.unknown,
      );
    }
  }
  
  // Safe quantity update
  Future<CartOperationResult> safeUpdateQuantity(Cart cart) async {
    try {
      if (cart.quantity! <= 0) {
        // Delete item if quantity is 0 or negative
        await dbHelper.delete(cart.id!);
        return CartOperationResult.success(
          message: 'Item removed from cart',
        );
      }
      
      await dbHelper.updateQuantity(cart);
      return CartOperationResult.success(
        message: 'Quantity updated successfully',
        data: cart,
      );
    } catch (e) {
      return CartOperationResult.error(
        message: 'Failed to update quantity: ${e.toString()}',
        errorType: CartErrorType.updateError,
      );
    }
  }
  
  // Get cart statistics
  Future<CartStatistics> getCartStatistics() async {
    try {
      List<Cart> items = await dbHelper.getCartList();
      
      double totalValue = items.fold(0.0, (sum, item) => 
        sum + (item.productPrice?.toDouble() ?? 0));
      
      int totalItems = items.fold(0, (sum, item) => 
        sum + (item.quantity ?? 0));
      
      Map<String, int> categoryCount = {};
      for (Cart item in items) {
        String category = item.unitTag ?? 'Unknown';
        categoryCount[category] = (categoryCount[category] ?? 0) + 1;
      }
      
      return CartStatistics(
        totalItems: totalItems,
        totalValue: totalValue,
        uniqueProducts: items.length,
        categoryBreakdown: categoryCount,
      );
    } catch (e) {
      throw Exception('Failed to calculate cart statistics: $e');
    }
  }
}

// Supporting classes
class CartOperationResult {
  final bool success;
  final String message;
  final Cart? data;
  final CartErrorType? errorType;
  
  CartOperationResult.success({required this.message, this.data})
    : success = true, errorType = null;
  
  CartOperationResult.error({required this.message, required this.errorType})
    : success = false, data = null;
}

enum CartErrorType {
  duplicateProduct,
  databaseError,
  updateError,
  unknown,
}

class CartStatistics {
  final int totalItems;
  final double totalValue;
  final int uniqueProducts;
  final Map<String, int> categoryBreakdown;
  
  CartStatistics({
    required this.totalItems,
    required this.totalValue,
    required this.uniqueProducts,
    required this.categoryBreakdown,
  });
}

// Extension for DatabaseException
extension DatabaseExceptionExtension on DatabaseException {
  bool isUniqueConstraintError() {
    return toString().contains('UNIQUE constraint failed');
  }
}
```

### 2. Database Migration Example

```dart
class DBHelperV2 extends DBHelper {
  @override
  initDatabase() async {
    io.Directory documentDirectory = await getApplicationDocumentsDirectory();
    String path = join(documentDirectory.path, 'cart.db');
    var db = await openDatabase(
      path,
      version: 2, // Incremented version
      onCreate: _onCreate,
      onUpgrade: _onUpgrade,
    );
    return db;
  }
  
  // Handle database upgrades
  _onUpgrade(Database db, int oldVersion, int newVersion) async {
    if (oldVersion < 2) {
      // Add new columns for version 2
      await db.execute('ALTER TABLE cart ADD COLUMN category TEXT DEFAULT "general"');
      await db.execute('ALTER TABLE cart ADD COLUMN discount REAL DEFAULT 0.0');
    }
  }
}
```

## State Management Patterns

### 1. Advanced CartProvider with Computed Properties

```dart
class AdvancedCartProvider extends CartProvider {
  
  // Computed properties
  double get subtotal => _totalPrice;
  double get tax => _totalPrice * 0.08; // 8% tax
  double get discount => _totalPrice * 0.05; // 5% discount
  double get finalTotal => subtotal + tax - discount;
  
  bool get hasItems => _counter > 0;
  bool get canCheckout => hasItems && finalTotal > 0;
  
  // Category-based operations
  Map<String, int> _categoryCount = {};
  Map<String, int> get categoryCount => Map.unmodifiable(_categoryCount);
  
  void updateCategoryCount(String category, int change) {
    _categoryCount[category] = (_categoryCount[category] ?? 0) + change;
    notifyListeners();
  }
  
  // Batch operations
  Future<void> addMultipleItems(List<Cart> items) async {
    for (Cart item in items) {
      await db.insert(item);
      addTotalPrice(item.productPrice!.toDouble());
      addCounter();
      updateCategoryCount(item.unitTag!, 1);
    }
  }
  
  // Clear cart with confirmation
  Future<bool> clearCartWithConfirmation(BuildContext context) async {
    bool? confirm = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Clear Cart'),
        content: Text('Are you sure you want to remove all items?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: Text('Cancel'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: Text('Clear'),
          ),
        ],
      ),
    );
    
    if (confirm == true) {
      List<Cart> items = await getData();
      for (Cart item in items) {
        await db.delete(item.id!);
        removerCounter();
        removeTotalPrice(item.productPrice!.toDouble());
      }
      return true;
    }
    
    return false;
  }
}
```

### 2. Provider with Stream Support

```dart
class StreamCartProvider extends CartProvider {
  StreamController<List<Cart>> _cartStreamController = StreamController<List<Cart>>.broadcast();
  Stream<List<Cart>> get cartStream => _cartStreamController.stream;
  
  @override
  Future<List<Cart>> getData() async {
    List<Cart> items = await super.getData();
    _cartStreamController.add(items);
    return items;
  }
  
  @override
  void dispose() {
    _cartStreamController.close();
    super.dispose();
  }
}

// Usage with StreamBuilder
StreamBuilder<List<Cart>>(
  stream: cartProvider.cartStream,
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return ListView.builder(
        itemCount: snapshot.data!.length,
        itemBuilder: (context, index) {
          return CartItemWidget(cartItem: snapshot.data![index]);
        },
      );
    }
    return CircularProgressIndicator();
  },
)
```

## Advanced Use Cases

### 1. Cart Synchronization Service

```dart
class CartSyncService {
  final CartProvider cartProvider;
  final ApiService apiService; // Your API service
  
  CartSyncService({required this.cartProvider, required this.apiService});
  
  // Sync local cart with server
  Future<void> syncCartWithServer() async {
    try {
      List<Cart> localItems = await cartProvider.getData();
      
      // Upload local cart to server
      await apiService.uploadCart(localItems);
      
      // Download server cart and merge
      List<Cart> serverItems = await apiService.downloadCart();
      await mergeServerCart(serverItems);
      
    } catch (e) {
      print('Cart sync failed: $e');
    }
  }
  
  // Merge server cart with local cart
  Future<void> mergeServerCart(List<Cart> serverItems) async {
    for (Cart serverItem in serverItems) {
      try {
        await cartProvider.db.insert(serverItem);
        cartProvider.addCounter();
        cartProvider.addTotalPrice(serverItem.productPrice!.toDouble());
      } catch (e) {
        // Handle conflicts (e.g., update existing items)
        await cartProvider.db.updateQuantity(serverItem);
      }
    }
  }
}
```

### 2. Cart Analytics

```dart
class CartAnalytics {
  final DBHelper dbHelper = DBHelper();
  
  // Track user behavior
  Future<Map<String, dynamic>> getCartAnalytics() async {
    List<Cart> items = await dbHelper.getCartList();
    
    return {
      'totalItems': items.length,
      'averagePrice': _calculateAveragePrice(items),
      'mostPopularCategory': _getMostPopularCategory(items),
      'totalValue': _calculateTotalValue(items),
      'priceDistribution': _getPriceDistribution(items),
    };
  }
  
  double _calculateAveragePrice(List<Cart> items) {
    if (items.isEmpty) return 0.0;
    double total = items.fold(0.0, (sum, item) => sum + item.initialPrice!);
    return total / items.length;
  }
  
  String _getMostPopularCategory(List<Cart> items) {
    Map<String, int> categoryCount = {};
    for (Cart item in items) {
      categoryCount[item.unitTag!] = (categoryCount[item.unitTag!] ?? 0) + 1;
    }
    
    return categoryCount.entries
        .reduce((a, b) => a.value > b.value ? a : b)
        .key;
  }
  
  double _calculateTotalValue(List<Cart> items) {
    return items.fold(0.0, (sum, item) => sum + item.productPrice!);
  }
  
  Map<String, int> _getPriceDistribution(List<Cart> items) {
    Map<String, int> distribution = {
      'under_20': 0,
      '20_to_50': 0,
      'above_50': 0,
    };
    
    for (Cart item in items) {
      int price = item.initialPrice!;
      if (price < 20) {
        distribution['under_20'] = distribution['under_20']! + 1;
      } else if (price <= 50) {
        distribution['20_to_50'] = distribution['20_to_50']! + 1;
      } else {
        distribution['above_50'] = distribution['above_50']! + 1;
      }
    }
    
    return distribution;
  }
}
```

## Best Practices

### 1. Error Handling Patterns

```dart
// Centralized error handling
class CartErrorHandler {
  static void handleCartError(BuildContext context, CartOperationResult result) {
    if (!result.success) {
      Color backgroundColor;
      IconData icon;
      
      switch (result.errorType) {
        case CartErrorType.duplicateProduct:
          backgroundColor = Colors.orange;
          icon = Icons.info;
          break;
        case CartErrorType.databaseError:
          backgroundColor = Colors.red;
          icon = Icons.error;
          break;
        default:
          backgroundColor = Colors.grey;
          icon = Icons.warning;
      }
      
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Row(
            children: [
              Icon(icon, color: Colors.white),
              SizedBox(width: 8),
              Expanded(child: Text(result.message)),
            ],
          ),
          backgroundColor: backgroundColor,
          behavior: SnackBarBehavior.floating,
        ),
      );
    }
  }
}
```

### 2. Performance Optimization

```dart
// Optimized cart loading with pagination
class OptimizedCartProvider extends CartProvider {
  static const int pageSize = 20;
  int _currentPage = 0;
  bool _hasMoreData = true;
  
  Future<List<Cart>> loadCartPage() async {
    if (!_hasMoreData) return [];
    
    // Implement pagination at database level
    List<Cart> allItems = await db.getCartList();
    int startIndex = _currentPage * pageSize;
    int endIndex = startIndex + pageSize;
    
    if (startIndex >= allItems.length) {
      _hasMoreData = false;
      return [];
    }
    
    endIndex = endIndex > allItems.length ? allItems.length : endIndex;
    List<Cart> pageItems = allItems.sublist(startIndex, endIndex);
    
    _currentPage++;
    _hasMoreData = endIndex < allItems.length;
    
    return pageItems;
  }
  
  void resetPagination() {
    _currentPage = 0;
    _hasMoreData = true;
  }
}
```

### 3. Testing Examples

```dart
// Unit test example for CartProvider
import 'package:flutter_test/flutter_test.dart';
import 'package:cartapp/provider/cartprovider.dart';

void main() {
  group('CartProvider Tests', () {
    late CartProvider cartProvider;
    
    setUp(() {
      cartProvider = CartProvider();
    });
    
    test('should initialize with zero counter', () {
      expect(cartProvider.getCounter(), equals(0));
    });
    
    test('should increment counter correctly', () {
      cartProvider.addCounter();
      expect(cartProvider.getCounter(), equals(1));
      
      cartProvider.addCounter();
      expect(cartProvider.getCounter(), equals(2));
    });
    
    test('should calculate total price correctly', () {
      cartProvider.addTotalPrice(10.0);
      cartProvider.addTotalPrice(15.5);
      
      expect(cartProvider.getTotalPrice(), equals(25.5));
    });
    
    test('should handle price removal', () {
      cartProvider.addTotalPrice(30.0);
      cartProvider.removeTotalPrice(10.0);
      
      expect(cartProvider.getTotalPrice(), equals(20.0));
    });
  });
}

// Widget test example
testWidgets('ProductList displays products correctly', (WidgetTester tester) async {
  await tester.pumpWidget(
    ChangeNotifierProvider(
      create: (_) => CartProvider(),
      child: MaterialApp(home: ProductList()),
    ),
  );
  
  // Verify product grid is displayed
  expect(find.byType(GridView), findsOneWidget);
  
  // Verify first product is displayed
  expect(find.text('Mango'), findsOneWidget);
  
  // Verify add to cart buttons
  expect(find.text('Add to cart'), findsWidgets);
});
```

### 4. Custom Hooks for State Management

```dart
// Custom hook for cart operations
class CartHook {
  final CartProvider _cartProvider;
  final BuildContext _context;
  
  CartHook(this._context) : _cartProvider = Provider.of<CartProvider>(_context, listen: false);
  
  // Simplified add to cart
  Future<void> addToCart({
    required String productId,
    required String productName,
    required int price,
    required String unit,
    required String imageUrl,
  }) async {
    Cart newItem = Cart(
      id: DateTime.now().millisecondsSinceEpoch, // Unique ID
      productId: productId,
      productName: productName,
      initialPrice: price,
      productPrice: price,
      quantity: 1,
      unitTag: unit,
      image: imageUrl,
    );
    
    SafeCartOperations operations = SafeCartOperations();
    CartOperationResult result = await operations.safeInsert(newItem);
    
    if (result.success) {
      _cartProvider.addCounter();
      _cartProvider.addTotalPrice(price.toDouble());
      
      ScaffoldMessenger.of(_context).showSnackBar(
        SnackBar(
          content: Text('${productName} added to cart'),
          backgroundColor: Colors.green,
        ),
      );
    } else {
      CartErrorHandler.handleCartError(_context, result);
    }
  }
  
  // Quick quantity update
  Future<void> updateItemQuantity(Cart item, int newQuantity) async {
    if (newQuantity <= 0) {
      await removeFromCart(item);
      return;
    }
    
    int oldQuantity = item.quantity!;
    int priceDifference = item.initialPrice! * (newQuantity - oldQuantity);
    
    Cart updatedItem = Cart(
      id: item.id,
      productId: item.productId,
      productName: item.productName,
      initialPrice: item.initialPrice,
      productPrice: item.initialPrice! * newQuantity,
      quantity: newQuantity,
      unitTag: item.unitTag,
      image: item.image,
    );
    
    SafeCartOperations operations = SafeCartOperations();
    CartOperationResult result = await operations.safeUpdateQuantity(updatedItem);
    
    if (result.success) {
      _cartProvider.addTotalPrice(priceDifference.toDouble());
    }
  }
  
  // Remove item from cart
  Future<void> removeFromCart(Cart item) async {
    DBHelper dbHelper = DBHelper();
    await dbHelper.delete(item.id!);
    
    _cartProvider.removerCounter();
    _cartProvider.removeTotalPrice(item.productPrice!.toDouble());
    
    ScaffoldMessenger.of(_context).showSnackBar(
      SnackBar(
        content: Text('${item.productName} removed from cart'),
        backgroundColor: Colors.red,
      ),
    );
  }
}

// Usage in widgets
class ProductTile extends StatelessWidget {
  final Product product;
  
  @override
  Widget build(BuildContext context) {
    CartHook cartHook = CartHook(context);
    
    return Card(
      child: Column(
        children: [
          // Product display
          ElevatedButton(
            onPressed: () => cartHook.addToCart(
              productId: product.id,
              productName: product.name,
              price: product.price,
              unit: product.unit,
              imageUrl: product.imageUrl,
            ),
            child: Text('Add to Cart'),
          ),
        ],
      ),
    );
  }
}
```

---

## Integration Patterns

### 1. Navigation with Cart State

```dart
class CartAwareNavigation {
  static void navigateToCart(BuildContext context) {
    final cartProvider = Provider.of<CartProvider>(context, listen: false);
    
    if (cartProvider.getCounter() == 0) {
      // Show empty cart dialog instead of navigating
      showDialog(
        context: context,
        builder: (context) => AlertDialog(
          title: Text('Empty Cart'),
          content: Text('Add some products to your cart first!'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context),
              child: Text('OK'),
            ),
          ],
        ),
      );
    } else {
      Navigator.push(
        context,
        MaterialPageRoute(builder: (context) => CartScreen()),
      );
    }
  }
}
```

### 2. Checkout Flow Integration

```dart
class CheckoutService {
  final CartProvider cartProvider;
  
  CheckoutService(this.cartProvider);
  
  Future<CheckoutResult> processCheckout() async {
    try {
      List<Cart> items = await cartProvider.getData();
      
      if (items.isEmpty) {
        return CheckoutResult.error('Cart is empty');
      }
      
      // Calculate final amounts
      double subtotal = cartProvider.getTotalPrice();
      double tax = subtotal * 0.08;
      double total = subtotal + tax;
      
      // Create order
      Order order = Order(
        items: items,
        subtotal: subtotal,
        tax: tax,
        total: total,
        timestamp: DateTime.now(),
      );
      
      // Process payment (integrate with payment gateway)
      PaymentResult paymentResult = await processPayment(order);
      
      if (paymentResult.success) {
        // Clear cart after successful payment
        await clearCart();
        return CheckoutResult.success(order);
      } else {
        return CheckoutResult.error(paymentResult.errorMessage);
      }
      
    } catch (e) {
      return CheckoutResult.error('Checkout failed: $e');
    }
  }
  
  Future<void> clearCart() async {
    List<Cart> items = await cartProvider.getData();
    for (Cart item in items) {
      await cartProvider.db.delete(item.id!);
    }
    
    // Reset provider state
    cartProvider._counter = 0;
    cartProvider._totalPrice = 0.0;
    cartProvider._setPrefItems();
  }
}
```

This documentation provides comprehensive coverage of all public APIs, components, and usage patterns in the Flutter shopping cart application. Each section includes detailed examples, error handling patterns, and best practices for implementation.