# Developer Guide

This guide provides comprehensive information for developers working on the Flutter Shopping Cart application, including setup instructions, coding standards, testing strategies, and deployment procedures.

## Table of Contents
- [Development Environment Setup](#development-environment-setup)
- [Project Structure](#project-structure)
- [Coding Standards](#coding-standards)
- [Testing Strategy](#testing-strategy)
- [Debugging Guide](#debugging-guide)
- [Performance Optimization](#performance-optimization)
- [Deployment](#deployment)
- [Contributing Guidelines](#contributing-guidelines)

## Development Environment Setup

### Prerequisites

#### Required Software
- **Flutter SDK**: Version >=2.18.4 <3.0.0
- **Dart SDK**: Included with Flutter
- **IDE**: Android Studio, VS Code, or IntelliJ IDEA
- **Git**: For version control

#### Platform-Specific Requirements

**For Android Development:**
- Android Studio
- Android SDK (API level 21+)
- Java 8 or higher

**For iOS Development (macOS only):**
- Xcode 12.0 or higher
- iOS 11.0 or higher
- CocoaPods

### Installation Steps

1. **Install Flutter:**
   ```bash
   # Download Flutter SDK
   git clone https://github.com/flutter/flutter.git -b stable
   
   # Add to PATH (add to ~/.bashrc or ~/.zshrc)
   export PATH="$PATH:`pwd`/flutter/bin"
   
   # Verify installation
   flutter doctor
   ```

2. **Setup IDE:**
   
   **VS Code:**
   ```bash
   # Install Flutter and Dart extensions
   code --install-extension Dart-Code.flutter
   code --install-extension Dart-Code.dart-code
   ```
   
   **Android Studio:**
   - Install Flutter plugin from Settings > Plugins
   - Install Dart plugin (usually comes with Flutter plugin)

3. **Clone and Setup Project:**
   ```bash
   git clone <repository-url>
   cd AddToCart-main
   flutter pub get
   flutter doctor -v
   ```

### IDE Configuration

#### VS Code Settings (.vscode/settings.json)
```json
{
  "dart.flutterSdkPath": "/path/to/flutter",
  "dart.lineLength": 100,
  "dart.insertArgumentPlaceholders": false,
  "dart.previewFlutterUiGuides": true,
  "dart.previewFlutterUiGuidesCustomTracking": true,
  "editor.rulers": [80, 100],
  "editor.formatOnSave": true,
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true
}
```

#### VS Code Launch Configuration (.vscode/launch.json)
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Flutter Debug",
      "type": "dart",
      "request": "launch",
      "program": "lib/main.dart",
      "args": ["--flavor", "development"]
    },
    {
      "name": "Flutter Release",
      "type": "dart",
      "request": "launch",
      "program": "lib/main.dart",
      "flutterMode": "release"
    }
  ]
}
```

## Project Structure

### Directory Organization

```
AddToCart-main/
├── lib/
│   ├── main.dart                 # Application entry point
│   ├── helper/
│   │   └── db_helper.dart       # Database operations
│   ├── model/
│   │   └── cart_model.dart      # Data models
│   ├── provider/
│   │   └── cartprovider.dart    # State management
│   ├── screen/
│   │   ├── product_list.dart    # Product catalog UI
│   │   └── cart_screen.dart     # Shopping cart UI
│   ├── widgets/                 # Reusable UI components
│   ├── services/               # Business logic services
│   ├── utils/                  # Utility functions
│   └── constants/              # App constants
├── test/
│   ├── unit/                   # Unit tests
│   ├── widget/                 # Widget tests
│   └── integration/            # Integration tests
├── docs/                       # Documentation
├── assets/                     # Static assets
└── pubspec.yaml               # Dependencies
```

### Recommended Extensions

#### New Directories to Add
```bash
# Create additional directories for better organization
mkdir -p lib/widgets lib/services lib/utils lib/constants
mkdir -p test/unit test/widget test/integration
```

#### File Organization Pattern
```dart
// lib/widgets/index.dart - Barrel exports
export 'cart_badge.dart';
export 'product_card.dart';
export 'loading_indicator.dart';
export 'empty_state.dart';

// lib/constants/app_constants.dart
class AppConstants {
  static const String appName = 'Shopping Cart App';
  static const int databaseVersion = 1;
  static const String databaseName = 'cart.db';
  
  // UI Constants
  static const double defaultPadding = 16.0;
  static const double cardBorderRadius = 12.0;
  static const Duration animationDuration = Duration(milliseconds: 300);
  
  // Business Logic Constants
  static const double taxRate = 0.08;
  static const double discountRate = 0.05;
  static const int maxQuantityPerItem = 99;
}

// lib/utils/helpers.dart
class AppHelpers {
  static String formatCurrency(double amount) {
    return '\$${amount.toStringAsFixed(2)}';
  }
  
  static String formatDate(DateTime date) {
    return '${date.day}/${date.month}/${date.year}';
  }
  
  static bool isValidEmail(String email) {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email);
  }
}
```

## Coding Standards

### Dart Code Style

#### 1. Naming Conventions
```dart
// Classes: PascalCase
class CartProvider extends ChangeNotifier { }
class ProductListScreen extends StatefulWidget { }

// Variables and functions: camelCase
int itemCounter = 0;
void addToCart() { }
String productName = '';

// Constants: SCREAMING_SNAKE_CASE
const int MAX_CART_ITEMS = 100;
const String DATABASE_NAME = 'cart.db';

// Private members: prefix with underscore
int _privateCounter = 0;
void _privateMethod() { }

// Files: snake_case
// cart_provider.dart
// product_list_screen.dart
// database_helper.dart
```

#### 2. Code Organization
```dart
// Class member order
class ExampleClass {
  // 1. Static constants
  static const String CONSTANT_VALUE = 'value';
  
  // 2. Instance variables
  final String publicField;
  int _privateField = 0;
  
  // 3. Constructor
  ExampleClass({required this.publicField});
  
  // 4. Named constructors
  ExampleClass.fromJson(Map<String, dynamic> json)
    : publicField = json['field'];
  
  // 5. Getters and setters
  int get privateField => _privateField;
  set privateField(int value) => _privateField = value;
  
  // 6. Public methods
  void publicMethod() { }
  
  // 7. Private methods
  void _privateMethod() { }
  
  // 8. Overridden methods
  @override
  String toString() => 'ExampleClass($publicField)';
}
```

#### 3. Documentation Standards
```dart
/// A comprehensive shopping cart provider that manages cart state.
/// 
/// This provider handles:
/// - Cart item counter management
/// - Total price calculations
/// - Data persistence via SharedPreferences
/// - Database operations coordination
/// 
/// Example usage:
/// ```dart
/// final cartProvider = Provider.of<CartProvider>(context);
/// cartProvider.addCounter();
/// cartProvider.addTotalPrice(25.99);
/// ```
class CartProvider with ChangeNotifier {
  
  /// The current number of items in the cart.
  /// 
  /// This value is persisted in SharedPreferences and restored on app restart.
  int get counter => _counter;
  
  /// Adds a product's price to the total cart value.
  /// 
  /// Parameters:
  /// - [productPrice]: The price to add to the total
  /// 
  /// Side effects:
  /// - Updates internal total price
  /// - Saves state to SharedPreferences
  /// - Notifies all listeners
  /// 
  /// Example:
  /// ```dart
  /// cartProvider.addTotalPrice(29.99);
  /// ```
  void addTotalPrice(double productPrice) {
    _totalPrice = _totalPrice + productPrice;
    _setPrefItems();
    notifyListeners();
  }
}
```

### Widget Best Practices

#### 1. Widget Composition
```dart
// Good: Compose complex widgets from smaller ones
class ProductCard extends StatelessWidget {
  final Product product;
  final VoidCallback onAddToCart;
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          ProductImage(imageUrl: product.imageUrl),
          ProductInfo(product: product),
          AddToCartButton(onPressed: onAddToCart),
        ],
      ),
    );
  }
}

// Avoid: Monolithic widgets with too much responsibility
class MonolithicProductCard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Card(
      child: Column(
        children: [
          // 100+ lines of nested widgets
          Container(
            child: Stack(
              children: [
                // Complex nested structure
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

#### 2. State Management Patterns
```dart
// Good: Use Consumer for specific rebuilds
Consumer<CartProvider>(
  builder: (context, cartProvider, child) {
    return Text('Items: ${cartProvider.getCounter()}');
  },
)

// Good: Use Selector for optimized rebuilds
Selector<CartProvider, int>(
  selector: (context, cartProvider) => cartProvider.getCounter(),
  builder: (context, counter, child) {
    return Text('Items: $counter');
  },
)

// Avoid: Watching entire provider when only specific data is needed
Provider.of<CartProvider>(context) // Rebuilds on any change
```

#### 3. Error Handling
```dart
// Good: Comprehensive error handling
class CartOperations {
  static Future<void> addToCart(
    BuildContext context,
    Cart item,
  ) async {
    try {
      final cartProvider = Provider.of<CartProvider>(context, listen: false);
      await cartProvider.db.insert(item);
      cartProvider.addCounter();
      cartProvider.addTotalPrice(item.productPrice!.toDouble());
      
      _showSuccessMessage(context, 'Item added to cart');
    } on DatabaseException catch (e) {
      if (e.isUniqueConstraintError()) {
        _showErrorMessage(context, 'Product already in cart');
      } else {
        _showErrorMessage(context, 'Database error occurred');
      }
    } catch (e) {
      _showErrorMessage(context, 'Unexpected error: $e');
    }
  }
  
  static void _showSuccessMessage(BuildContext context, String message) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(message),
        backgroundColor: Colors.green,
        behavior: SnackBarBehavior.floating,
      ),
    );
  }
  
  static void _showErrorMessage(BuildContext context, String message) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(message),
        backgroundColor: Colors.red,
        behavior: SnackBarBehavior.floating,
      ),
    );
  }
}
```

## Testing Strategy

### 1. Unit Tests

#### Testing Cart Model
```dart
// test/unit/cart_model_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:cartapp/model/cart_model.dart';

void main() {
  group('Cart Model Tests', () {
    test('should create cart item with all properties', () {
      final cart = Cart(
        id: 1,
        productId: 'test_product',
        productName: 'Test Product',
        initialPrice: 10,
        productPrice: 30,
        quantity: 3,
        unitTag: 'KG',
        image: 'test_image.jpg',
      );
      
      expect(cart.id, equals(1));
      expect(cart.productName, equals('Test Product'));
      expect(cart.quantity, equals(3));
      expect(cart.productPrice, equals(30));
    });
    
    test('should convert to map correctly', () {
      final cart = Cart(
        id: 1,
        productId: 'test_product',
        productName: 'Test Product',
        initialPrice: 10,
        productPrice: 30,
        quantity: 3,
        unitTag: 'KG',
        image: 'test_image.jpg',
      );
      
      final map = cart.toMap();
      
      expect(map['id'], equals(1));
      expect(map['productName'], equals('Test Product'));
      expect(map['quantity'], equals(3));
    });
    
    test('should create from map correctly', () {
      final map = {
        'id': 1,
        'productId': 'test_product',
        'productName': 'Test Product',
        'initialPrice': 10,
        'productPrice': 30,
        'quantity': 3,
        'unitTag': 'KG',
        'image': 'test_image.jpg',
      };
      
      final cart = Cart.fromMap(map);
      
      expect(cart.id, equals(1));
      expect(cart.productName, equals('Test Product'));
      expect(cart.quantity, equals(3));
    });
  });
}
```

#### Testing Cart Provider
```dart
// test/unit/cart_provider_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:cartapp/provider/cartprovider.dart';

void main() {
  group('CartProvider Tests', () {
    late CartProvider cartProvider;
    
    setUp(() {
      cartProvider = CartProvider();
    });
    
    test('should initialize with default values', () {
      expect(cartProvider.getCounter(), equals(0));
      expect(cartProvider.getTotalPrice(), equals(0.0));
    });
    
    test('should increment counter correctly', () {
      cartProvider.addCounter();
      expect(cartProvider.getCounter(), equals(1));
      
      cartProvider.addCounter();
      expect(cartProvider.getCounter(), equals(2));
    });
    
    test('should decrement counter correctly', () {
      cartProvider.addCounter();
      cartProvider.addCounter();
      cartProvider.removerCounter();
      
      expect(cartProvider.getCounter(), equals(1));
    });
    
    test('should calculate total price correctly', () {
      cartProvider.addTotalPrice(10.5);
      cartProvider.addTotalPrice(15.25);
      
      expect(cartProvider.getTotalPrice(), equals(25.75));
    });
    
    test('should handle price removal', () {
      cartProvider.addTotalPrice(50.0);
      cartProvider.removeTotalPrice(20.0);
      
      expect(cartProvider.getTotalPrice(), equals(30.0));
    });
  });
}
```

### 2. Widget Tests

#### Testing Product List
```dart
// test/widget/product_list_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:provider/provider.dart';
import 'package:cartapp/screen/product_list.dart';
import 'package:cartapp/provider/cartprovider.dart';

void main() {
  group('ProductList Widget Tests', () {
    testWidgets('should display product grid', (WidgetTester tester) async {
      await tester.pumpWidget(
        ChangeNotifierProvider(
          create: (_) => CartProvider(),
          child: MaterialApp(
            home: ProductList(),
          ),
        ),
      );
      
      // Verify grid view is displayed
      expect(find.byType(GridView), findsOneWidget);
      
      // Verify products are displayed
      expect(find.text('Mango'), findsOneWidget);
      expect(find.text('Orange'), findsOneWidget);
      
      // Verify add to cart buttons
      expect(find.text('Add to cart'), findsWidgets);
    });
    
    testWidgets('should show cart badge with counter', (WidgetTester tester) async {
      CartProvider cartProvider = CartProvider();
      cartProvider.addCounter();
      
      await tester.pumpWidget(
        ChangeNotifierProvider.value(
          value: cartProvider,
          child: MaterialApp(
            home: ProductList(),
          ),
        ),
      );
      
      // Verify badge shows correct count
      expect(find.text('1'), findsOneWidget);
    });
    
    testWidgets('should navigate to cart screen when badge is tapped', 
        (WidgetTester tester) async {
      await tester.pumpWidget(
        ChangeNotifierProvider(
          create: (_) => CartProvider(),
          child: MaterialApp(
            home: ProductList(),
          ),
        ),
      );
      
      // Tap on cart badge
      await tester.tap(find.byIcon(Icons.shopping_bag_outlined));
      await tester.pumpAndSettle();
      
      // Verify navigation to cart screen
      expect(find.text('Shopping Cart'), findsOneWidget);
    });
  });
}
```

### 3. Integration Tests

#### Database Integration Test
```dart
// test/integration/database_integration_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';
import 'package:cartapp/helper/db_helper.dart';
import 'package:cartapp/model/cart_model.dart';

void main() {
  group('Database Integration Tests', () {
    late DBHelper dbHelper;
    
    setUpAll(() {
      // Initialize FFI for testing
      sqfliteFfiInit();
      databaseFactory = databaseFactoryFfi;
    });
    
    setUp(() {
      dbHelper = DBHelper();
    });
    
    test('should insert and retrieve cart items', () async {
      final cart = Cart(
        id: 1,
        productId: 'test_product',
        productName: 'Test Product',
        initialPrice: 10,
        productPrice: 30,
        quantity: 3,
        unitTag: 'KG',
        image: 'test_image.jpg',
      );
      
      // Insert item
      await dbHelper.insert(cart);
      
      // Retrieve items
      List<Cart> items = await dbHelper.getCartList();
      
      expect(items.length, equals(1));
      expect(items.first.productName, equals('Test Product'));
      expect(items.first.quantity, equals(3));
    });
    
    test('should update item quantity', () async {
      final cart = Cart(
        id: 1,
        productId: 'test_product',
        productName: 'Test Product',
        initialPrice: 10,
        productPrice: 10,
        quantity: 1,
        unitTag: 'KG',
        image: 'test_image.jpg',
      );
      
      await dbHelper.insert(cart);
      
      // Update quantity
      final updatedCart = Cart(
        id: 1,
        productId: 'test_product',
        productName: 'Test Product',
        initialPrice: 10,
        productPrice: 50, // 10 * 5
        quantity: 5,
        unitTag: 'KG',
        image: 'test_image.jpg',
      );
      
      await dbHelper.updateQuantity(updatedCart);
      
      // Verify update
      List<Cart> items = await dbHelper.getCartList();
      expect(items.first.quantity, equals(5));
      expect(items.first.productPrice, equals(50));
    });
    
    test('should delete cart item', () async {
      final cart = Cart(
        id: 1,
        productId: 'test_product',
        productName: 'Test Product',
        initialPrice: 10,
        productPrice: 10,
        quantity: 1,
        unitTag: 'KG',
        image: 'test_image.jpg',
      );
      
      await dbHelper.insert(cart);
      
      // Delete item
      await dbHelper.delete(1);
      
      // Verify deletion
      List<Cart> items = await dbHelper.getCartList();
      expect(items.length, equals(0));
    });
  });
}
```

### 4. Test Utilities

#### Mock Data Factory
```dart
// test/utils/test_data.dart
import 'package:cartapp/model/cart_model.dart';

class TestData {
  static Cart createMockCartItem({
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
      id: id ?? 1,
      productId: productId ?? 'test_product',
      productName: productName ?? 'Test Product',
      initialPrice: initialPrice ?? 10,
      productPrice: productPrice ?? 10,
      quantity: quantity ?? 1,
      unitTag: unitTag ?? 'KG',
      image: image ?? 'test_image.jpg',
    );
  }
  
  static List<Cart> createMockCartList(int count) {
    return List.generate(count, (index) => createMockCartItem(
      id: index,
      productId: 'product_$index',
      productName: 'Product $index',
      initialPrice: 10 + index,
      productPrice: (10 + index) * (index + 1),
      quantity: index + 1,
    ));
  }
}

// Mock Provider for Testing
class MockCartProvider extends CartProvider {
  @override
  Future<List<Cart>> getData() async {
    return TestData.createMockCartList(3);
  }
  
  @override
  int getCounter() => 3;
  
  @override
  double getTotalPrice() => 60.0;
}
```

## Debugging Guide

### 1. Common Issues and Solutions

#### Database Issues
```dart
// Debug database operations
class DebugDBHelper extends DBHelper {
  @override
  Future<Cart> insert(Cart cart) async {
    print('Inserting cart item: ${cart.toMap()}');
    try {
      Cart result = await super.insert(cart);
      print('Insert successful: ${result.productName}');
      return result;
    } catch (e) {
      print('Insert failed: $e');
      rethrow;
    }
  }
  
  @override
  Future<List<Cart>> getCartList() async {
    print('Fetching cart list...');
    List<Cart> items = await super.getCartList();
    print('Retrieved ${items.length} items');
    return items;
  }
}
```

#### Provider State Issues
```dart
// Debug provider state changes
class DebugCartProvider extends CartProvider {
  @override
  void addCounter() {
    print('Adding counter: $_counter -> ${_counter + 1}');
    super.addCounter();
  }
  
  @override
  void addTotalPrice(double productPrice) {
    print('Adding price: $_totalPrice -> ${_totalPrice + productPrice}');
    super.addTotalPrice(productPrice);
  }
  
  @override
  void notifyListeners() {
    print('Notifying listeners - Counter: $_counter, Total: $_totalPrice');
    super.notifyListeners();
  }
}
```

### 2. Debug Tools and Commands

#### Flutter Inspector
```bash
# Enable widget inspector
flutter run --debug

# In VS Code: Cmd/Ctrl + Shift + P -> "Flutter: Open Widget Inspector"
```

#### Performance Profiling
```bash
# Profile app performance
flutter run --profile --trace-startup

# Analyze performance
flutter analyze
dart analyze
```

#### Database Debugging
```dart
// Add to DBHelper for SQL debugging
Future<void> debugDatabase() async {
  var dbClient = await db;
  
  // Check table structure
  List<Map<String, dynamic>> tables = await dbClient!.rawQuery(
    "SELECT name FROM sqlite_master WHERE type='table'"
  );
  print('Tables: $tables');
  
  // Check cart table schema
  List<Map<String, dynamic>> schema = await dbClient.rawQuery(
    "PRAGMA table_info(cart)"
  );
  print('Cart table schema: $schema');
  
  // Count items
  List<Map<String, dynamic>> count = await dbClient.rawQuery(
    "SELECT COUNT(*) as count FROM cart"
  );
  print('Cart item count: ${count.first['count']}');
}
```

## Performance Optimization

### 1. Widget Performance

#### Efficient List Rendering
```dart
class OptimizedCartList extends StatelessWidget {
  final List<Cart> items;
  
  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: items.length,
      itemExtent: 120, // Fixed height for better performance
      cacheExtent: 1000, // Cache more items
      itemBuilder: (context, index) {
        return CartItemWidget(
          key: ValueKey(items[index].id), // Stable keys
          cartItem: items[index],
        );
      },
    );
  }
}
```

#### Optimized Image Loading
```dart
class OptimizedProductImage extends StatelessWidget {
  final String imageUrl;
  final double size;
  
  @override
  Widget build(BuildContext context) {
    return Container(
      width: size,
      height: size,
      child: ClipOval(
        child: Image.network(
          imageUrl,
          width: size,
          height: size,
          fit: BoxFit.cover,
          cacheWidth: (size * MediaQuery.of(context).devicePixelRatio).round(),
          cacheHeight: (size * MediaQuery.of(context).devicePixelRatio).round(),
          loadingBuilder: (context, child, loadingProgress) {
            if (loadingProgress == null) return child;
            return Container(
              color: Colors.grey[300],
              child: Center(
                child: SizedBox(
                  width: 20,
                  height: 20,
                  child: CircularProgressIndicator(strokeWidth: 2),
                ),
              ),
            );
          },
          errorBuilder: (context, error, stackTrace) {
            return Container(
              color: Colors.grey[300],
              child: Icon(Icons.image_not_supported),
            );
          },
        ),
      ),
    );
  }
}
```

### 2. Database Performance

#### Batch Operations
```dart
class BatchDBHelper extends DBHelper {
  Future<void> insertBatch(List<Cart> items) async {
    var dbClient = await db;
    Batch batch = dbClient!.batch();
    
    for (Cart item in items) {
      batch.insert('cart', item.toMap());
    }
    
    await batch.commit(noResult: true);
  }
  
  Future<void> updateBatch(List<Cart> items) async {
    var dbClient = await db;
    Batch batch = dbClient!.batch();
    
    for (Cart item in items) {
      batch.update(
        'cart', 
        item.toMap(), 
        where: 'id = ?', 
        whereArgs: [item.id]
      );
    }
    
    await batch.commit(noResult: true);
  }
}
```

#### Database Indexing
```dart
// Enhanced database schema with indexes
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
      image TEXT
    )
  ''');
  
  // Create indexes for better query performance
  await db.execute('CREATE INDEX idx_productId ON cart(productId)');
  await db.execute('CREATE INDEX idx_unitTag ON cart(unitTag)');
}
```

## Deployment

### 1. Build Configuration

#### Android Release Build
```bash
# Generate keystore (one time setup)
keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload

# Configure signing in android/app/build.gradle
# Add key.properties file with keystore information

# Build release APK
flutter build apk --release

# Build App Bundle (recommended for Play Store)
flutter build appbundle --release
```

#### iOS Release Build
```bash
# Build for iOS (requires macOS)
flutter build ios --release

# Build for App Store
flutter build ipa
```

### 2. Environment Configuration

#### Development Environment
```dart
// lib/config/environment.dart
class Environment {
  static const String environment = String.fromEnvironment(
    'ENVIRONMENT',
    defaultValue: 'development',
  );
  
  static const String apiBaseUrl = String.fromEnvironment(
    'API_BASE_URL',
    defaultValue: 'https://dev-api.example.com',
  );
  
  static bool get isDevelopment => environment == 'development';
  static bool get isProduction => environment == 'production';
  static bool get isDebug => !isProduction;
}

// Usage in main.dart
void main() {
  if (Environment.isDevelopment) {
    // Enable debug logging
    debugPrint('Running in development mode');
  }
  
  runApp(MyApp());
}
```

#### Build Flavors
```bash
# Run with different flavors
flutter run --flavor development
flutter run --flavor staging
flutter run --flavor production

# Build with flavors
flutter build apk --flavor production --release
```

### 3. CI/CD Pipeline Example

#### GitHub Actions Workflow
```yaml
# .github/workflows/flutter.yml
name: Flutter CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Flutter
      uses: subosito/flutter-action@v2
      with:
        flutter-version: '3.0.0'
    
    - name: Get dependencies
      run: flutter pub get
    
    - name: Analyze code
      run: flutter analyze
    
    - name: Run tests
      run: flutter test --coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: coverage/lcov.info

  build:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Flutter
      uses: subosito/flutter-action@v2
      with:
        flutter-version: '3.0.0'
    
    - name: Get dependencies
      run: flutter pub get
    
    - name: Build APK
      run: flutter build apk --release
    
    - name: Upload APK
      uses: actions/upload-artifact@v3
      with:
        name: release-apk
        path: build/app/outputs/flutter-apk/app-release.apk
```

## Contributing Guidelines

### 1. Development Workflow

#### Branch Strategy
```bash
# Feature development
git checkout -b feature/add-product-search
git commit -m "feat: add product search functionality"
git push origin feature/add-product-search

# Bug fixes
git checkout -b fix/cart-total-calculation
git commit -m "fix: correct cart total calculation"

# Hotfixes
git checkout -b hotfix/critical-crash-fix
git commit -m "hotfix: fix app crash on empty cart"
```

#### Commit Message Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code formatting (no logic changes)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples:**
```bash
feat(cart): add product search functionality
fix(database): resolve duplicate product insertion error
docs(api): update CartProvider documentation
test(provider): add unit tests for cart operations
```

### 2. Code Review Checklist

#### Before Submitting PR
- [ ] All tests pass (`flutter test`)
- [ ] Code follows style guidelines (`flutter analyze`)
- [ ] Documentation is updated
- [ ] No debug prints or commented code
- [ ] Performance impact considered
- [ ] Error handling implemented
- [ ] Backward compatibility maintained

#### Review Criteria
- [ ] Code is readable and well-documented
- [ ] Follows established patterns
- [ ] Handles edge cases appropriately
- [ ] Includes appropriate tests
- [ ] Performance implications considered
- [ ] Security considerations addressed

### 3. Issue Reporting

#### Bug Report Template
```markdown
## Bug Description
Brief description of the issue

## Steps to Reproduce
1. Step one
2. Step two
3. Expected vs actual behavior

## Environment
- Flutter version:
- Dart version:
- Platform (iOS/Android):
- Device/Emulator:

## Screenshots
[If applicable]

## Additional Context
Any additional information
```

#### Feature Request Template
```markdown
## Feature Description
Clear description of the proposed feature

## Use Case
Why is this feature needed?

## Proposed Solution
How should this feature work?

## Alternatives Considered
Other approaches considered

## Additional Context
Any additional information
```

### 4. Release Process

#### Version Management
```bash
# Update version in pubspec.yaml
version: 1.1.0+2

# Create release tag
git tag v1.1.0
git push origin v1.1.0

# Generate changelog
git log --oneline v1.0.0..v1.1.0
```

#### Release Checklist
- [ ] Version number updated
- [ ] Changelog updated
- [ ] All tests passing
- [ ] Documentation updated
- [ ] Release notes prepared
- [ ] Build artifacts generated
- [ ] App store metadata updated

This developer guide provides comprehensive information for maintaining and extending the Flutter Shopping Cart application. Follow these guidelines to ensure code quality, maintainability, and team collaboration.