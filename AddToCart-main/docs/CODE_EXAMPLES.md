# Code Examples and Implementation Patterns

This document provides practical, copy-paste ready code examples for common tasks and implementation patterns in the Flutter Shopping Cart application.

## Table of Contents
- [Basic Operations](#basic-operations)
- [Advanced Patterns](#advanced-patterns)
- [UI Customizations](#ui-customizations)
- [Error Handling](#error-handling)
- [Testing Examples](#testing-examples)
- [Performance Optimizations](#performance-optimizations)

## Basic Operations

### 1. Adding Items to Cart

#### Simple Add to Cart
```dart
import 'package:cartapp/helper/db_helper.dart';
import 'package:cartapp/model/cart_model.dart';
import 'package:cartapp/provider/cartprovider.dart';
import 'package:provider/provider.dart';

void addToCart(BuildContext context, {
  required String productId,
  required String productName,
  required int price,
  required String unit,
  required String imageUrl,
}) async {
  final cartProvider = Provider.of<CartProvider>(context, listen: false);
  final dbHelper = DBHelper();
  
  Cart newItem = Cart(
    id: DateTime.now().millisecondsSinceEpoch,
    productId: productId,
    productName: productName,
    initialPrice: price,
    productPrice: price,
    quantity: 1,
    unitTag: unit,
    image: imageUrl,
  );
  
  try {
    await dbHelper.insert(newItem);
    cartProvider.addCounter();
    cartProvider.addTotalPrice(price.toDouble());
    
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('$productName added to cart'),
        backgroundColor: Colors.green,
        duration: Duration(seconds: 2),
      ),
    );
  } catch (e) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Product already in cart'),
        backgroundColor: Colors.orange,
      ),
    );
  }
}
```

#### Bulk Add to Cart
```dart
Future<void> addMultipleToCart(
  BuildContext context,
  List<Map<String, dynamic>> products,
) async {
  final cartProvider = Provider.of<CartProvider>(context, listen: false);
  final dbHelper = DBHelper();
  
  int successCount = 0;
  int failureCount = 0;
  
  for (var product in products) {
    try {
      Cart cartItem = Cart(
        id: DateTime.now().millisecondsSinceEpoch + products.indexOf(product),
        productId: product['id'],
        productName: product['name'],
        initialPrice: product['price'],
        productPrice: product['price'],
        quantity: 1,
        unitTag: product['unit'],
        image: product['image'],
      );
      
      await dbHelper.insert(cartItem);
      cartProvider.addCounter();
      cartProvider.addTotalPrice(product['price'].toDouble());
      successCount++;
      
    } catch (e) {
      failureCount++;
    }
  }
  
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(
      content: Text('Added $successCount items, $failureCount duplicates'),
      backgroundColor: successCount > 0 ? Colors.green : Colors.orange,
    ),
  );
}
```

### 2. Managing Cart Quantities

#### Update Item Quantity
```dart
Future<void> updateCartItemQuantity(
  BuildContext context,
  Cart item,
  int newQuantity,
) async {
  final cartProvider = Provider.of<CartProvider>(context, listen: false);
  final dbHelper = DBHelper();
  
  if (newQuantity <= 0) {
    // Remove item if quantity is 0 or negative
    await removeFromCart(context, item);
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
  
  try {
    await dbHelper.updateQuantity(updatedItem);
    cartProvider.addTotalPrice(priceDifference.toDouble());
    
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('${item.productName} quantity updated'),
        backgroundColor: Colors.blue,
      ),
    );
  } catch (e) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Failed to update quantity'),
        backgroundColor: Colors.red,
      ),
    );
  }
}
```

#### Increment/Decrement Helpers
```dart
void incrementQuantity(BuildContext context, Cart item) {
  updateCartItemQuantity(context, item, item.quantity! + 1);
}

void decrementQuantity(BuildContext context, Cart item) {
  updateCartItemQuantity(context, item, item.quantity! - 1);
}

// Quick quantity buttons widget
class QuantityControls extends StatelessWidget {
  final Cart item;
  final Function(int) onQuantityChanged;
  
  const QuantityControls({
    Key? key,
    required this.item,
    required this.onQuantityChanged,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.green,
        borderRadius: BorderRadius.circular(20),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          IconButton(
            onPressed: () => onQuantityChanged(item.quantity! - 1),
            icon: Icon(Icons.remove, color: Colors.white),
            constraints: BoxConstraints(minWidth: 32, minHeight: 32),
          ),
          Container(
            padding: EdgeInsets.symmetric(horizontal: 12),
            child: Text(
              item.quantity.toString(),
              style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold),
            ),
          ),
          IconButton(
            onPressed: () => onQuantityChanged(item.quantity! + 1),
            icon: Icon(Icons.add, color: Colors.white),
            constraints: BoxConstraints(minWidth: 32, minHeight: 32),
          ),
        ],
      ),
    );
  }
}
```

### 3. Cart Management

#### Clear Entire Cart
```dart
Future<void> clearCart(BuildContext context) async {
  final cartProvider = Provider.of<CartProvider>(context, listen: false);
  final dbHelper = DBHelper();
  
  // Show confirmation dialog
  bool? confirm = await showDialog<bool>(
    context: context,
    builder: (context) => AlertDialog(
      title: Text('Clear Cart'),
      content: Text('Are you sure you want to remove all items from your cart?'),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context, false),
          child: Text('Cancel'),
        ),
        TextButton(
          onPressed: () => Navigator.pop(context, true),
          child: Text('Clear', style: TextStyle(color: Colors.red)),
        ),
      ],
    ),
  );
  
  if (confirm == true) {
    try {
      List<Cart> items = await cartProvider.getData();
      
      for (Cart item in items) {
        await dbHelper.delete(item.id!);
        cartProvider.removerCounter();
        cartProvider.removeTotalPrice(item.productPrice!.toDouble());
      }
      
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Cart cleared successfully'),
          backgroundColor: Colors.green,
        ),
      );
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Failed to clear cart'),
          backgroundColor: Colors.red,
        ),
      );
    }
  }
}
```

#### Remove Single Item
```dart
Future<void> removeFromCart(BuildContext context, Cart item) async {
  final cartProvider = Provider.of<CartProvider>(context, listen: false);
  final dbHelper = DBHelper();
  
  try {
    await dbHelper.delete(item.id!);
    cartProvider.removerCounter();
    cartProvider.removeTotalPrice(item.productPrice!.toDouble());
    
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('${item.productName} removed from cart'),
        backgroundColor: Colors.red,
        action: SnackBarAction(
          label: 'UNDO',
          onPressed: () {
            // Re-add the item
            addToCart(context,
              productId: item.productId!,
              productName: item.productName!,
              price: item.initialPrice!,
              unit: item.unitTag!,
              imageUrl: item.image!,
            );
          },
        ),
      ),
    );
  } catch (e) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Failed to remove item'),
        backgroundColor: Colors.red,
      ),
    );
  }
}
```

## Advanced Patterns

### 1. Custom Cart Provider with Categories

```dart
class CategoryCartProvider extends CartProvider {
  Map<String, List<Cart>> _categorizedItems = {};
  Map<String, double> _categoryTotals = {};
  
  Map<String, List<Cart>> get categorizedItems => Map.unmodifiable(_categorizedItems);
  Map<String, double> get categoryTotals => Map.unmodifiable(_categoryTotals);
  
  @override
  Future<List<Cart>> getData() async {
    List<Cart> items = await super.getData();
    _categorizeitems(items);
    return items;
  }
  
  void _categorizeitems(List<Cart> items) {
    _categorizedItems.clear();
    _categoryTotals.clear();
    
    for (Cart item in items) {
      String category = item.unitTag ?? 'Other';
      
      _categorizedItems.putIfAbsent(category, () => []).add(item);
      _categoryTotals[category] = (_categoryTotals[category] ?? 0) + 
        (item.productPrice?.toDouble() ?? 0);
    }
    
    notifyListeners();
  }
  
  List<Cart> getItemsByCategory(String category) {
    return _categorizedItems[category] ?? [];
  }
  
  double getCategoryTotal(String category) {
    return _categoryTotals[category] ?? 0.0;
  }
  
  List<String> getCategories() {
    return _categorizedItems.keys.toList()..sort();
  }
}

// Usage in UI
class CategorizedCartScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<CategoryCartProvider>(
      builder: (context, cartProvider, child) {
        return FutureBuilder<List<Cart>>(
          future: cartProvider.getData(),
          builder: (context, snapshot) {
            if (snapshot.hasData) {
              List<String> categories = cartProvider.getCategories();
              
              return ListView.builder(
                itemCount: categories.length,
                itemBuilder: (context, index) {
                  String category = categories[index];
                  List<Cart> categoryItems = cartProvider.getItemsByCategory(category);
                  double categoryTotal = cartProvider.getCategoryTotal(category);
                  
                  return ExpansionTile(
                    title: Text('$category (${categoryItems.length} items)'),
                    subtitle: Text('Total: \$${categoryTotal.toStringAsFixed(2)}'),
                    children: categoryItems.map((item) => 
                      CartItemWidget(cartItem: item)).toList(),
                  );
                },
              );
            }
            return CircularProgressIndicator();
          },
        );
      },
    );
  }
}
```

### 2. Search and Filter Implementation

```dart
class SearchableProductProvider extends ChangeNotifier {
  List<Product> _allProducts = [];
  List<Product> _filteredProducts = [];
  String _searchQuery = '';
  String _selectedCategory = 'All';
  
  List<Product> get products => _filteredProducts;
  String get searchQuery => _searchQuery;
  String get selectedCategory => _selectedCategory;
  
  void loadProducts(List<Product> products) {
    _allProducts = products;
    _filteredProducts = products;
    notifyListeners();
  }
  
  void searchProducts(String query) {
    _searchQuery = query;
    _applyFilters();
  }
  
  void filterByCategory(String category) {
    _selectedCategory = category;
    _applyFilters();
  }
  
  void _applyFilters() {
    _filteredProducts = _allProducts.where((product) {
      bool matchesSearch = _searchQuery.isEmpty ||
        product.name.toLowerCase().contains(_searchQuery.toLowerCase());
      
      bool matchesCategory = _selectedCategory == 'All' ||
        product.category == _selectedCategory;
      
      return matchesSearch && matchesCategory;
    }).toList();
    
    notifyListeners();
  }
  
  void clearFilters() {
    _searchQuery = '';
    _selectedCategory = 'All';
    _filteredProducts = _allProducts;
    notifyListeners();
  }
}

// Usage in ProductList with Search
class SearchableProductList extends StatefulWidget {
  @override
  _SearchableProductListState createState() => _SearchableProductListState();
}

class _SearchableProductListState extends State<SearchableProductList> {
  final TextEditingController _searchController = TextEditingController();
  late SearchableProductProvider _productProvider;
  
  @override
  void initState() {
    super.initState();
    _productProvider = SearchableProductProvider();
    _loadProducts();
  }
  
  void _loadProducts() {
    // Convert existing product data to Product objects
    List<Product> products = [];
    for (int i = 0; i < productName.length; i++) {
      products.add(Product(
        id: i.toString(),
        name: productName[i],
        price: productPrice[i],
        unit: productUnit[i],
        imageUrl: productImage[i],
        category: productUnit[i], // Using unit as category for demo
      ));
    }
    _productProvider.loadProducts(products);
  }
  
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider.value(
      value: _productProvider,
      child: Scaffold(
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
                    controller: _searchController,
                    decoration: InputDecoration(
                      hintText: 'Search products...',
                      prefixIcon: Icon(Icons.search),
                      suffixIcon: _searchController.text.isNotEmpty
                        ? IconButton(
                            icon: Icon(Icons.clear),
                            onPressed: () {
                              _searchController.clear();
                              _productProvider.searchProducts('');
                            },
                          )
                        : null,
                      border: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(25),
                      ),
                      filled: true,
                      fillColor: Colors.white,
                    ),
                    onChanged: (query) {
                      _productProvider.searchProducts(query);
                    },
                  ),
                ),
                
                // Category Filter
                Consumer<SearchableProductProvider>(
                  builder: (context, provider, child) {
                    Set<String> categories = {'All'};
                    categories.addAll(provider._allProducts.map((p) => p.category).toSet());
                    
                    return Container(
                      height: 40,
                      child: ListView.builder(
                        scrollDirection: Axis.horizontal,
                        padding: EdgeInsets.symmetric(horizontal: 16),
                        itemCount: categories.length,
                        itemBuilder: (context, index) {
                          String category = categories.elementAt(index);
                          bool isSelected = provider.selectedCategory == category;
                          
                          return Padding(
                            padding: EdgeInsets.only(right: 8),
                            child: FilterChip(
                              label: Text(category),
                              selected: isSelected,
                              onSelected: (_) => provider.filterByCategory(category),
                              backgroundColor: Colors.white,
                              selectedColor: Colors.blue[100],
                            ),
                          );
                        },
                      ),
                    );
                  },
                ),
              ],
            ),
          ),
        ),
        body: Consumer<SearchableProductProvider>(
          builder: (context, provider, child) {
            if (provider.products.isEmpty) {
              return Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Icon(Icons.search_off, size: 64, color: Colors.grey),
                    SizedBox(height: 16),
                    Text(
                      'No products found',
                      style: TextStyle(fontSize: 18, color: Colors.grey),
                    ),
                    SizedBox(height: 8),
                    Text(
                      'Try adjusting your search or filters',
                      style: TextStyle(color: Colors.grey[600]),
                    ),
                  ],
                ),
              );
            }
            
            return GridView.builder(
              padding: EdgeInsets.all(16),
              gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                crossAxisCount: 2,
                childAspectRatio: 0.75,
                crossAxisSpacing: 16,
                mainAxisSpacing: 16,
              ),
              itemCount: provider.products.length,
              itemBuilder: (context, index) {
                Product product = provider.products[index];
                return ProductCard(
                  product: product,
                  onAddToCart: () => addToCart(context,
                    productId: product.id,
                    productName: product.name,
                    price: product.price,
                    unit: product.unit,
                    imageUrl: product.imageUrl,
                  ),
                );
              },
            );
          },
        ),
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
  
  Product({
    required this.id,
    required this.name,
    required this.price,
    required this.unit,
    required this.imageUrl,
    required this.category,
  });
}
```

### 3. Real-time Cart Updates

#### Cart Badge with Animation
```dart
class AnimatedCartBadge extends StatefulWidget {
  final Widget child;
  final VoidCallback? onTap;
  
  const AnimatedCartBadge({
    Key? key,
    required this.child,
    this.onTap,
  }) : super(key: key);
  
  @override
  _AnimatedCartBadgeState createState() => _AnimatedCartBadgeState();
}

class _AnimatedCartBadgeState extends State<AnimatedCartBadge>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 200),
      vsync: this,
    );
    _scaleAnimation = Tween<double>(begin: 1.0, end: 1.2).animate(
      CurvedAnimation(parent: _controller, curve: Curves.elasticOut),
    );
  }
  
  @override
  Widget build(BuildContext context) {
    return Consumer<CartProvider>(
      builder: (context, cartProvider, child) {
        // Trigger animation when counter changes
        if (cartProvider.getCounter() > 0) {
          _controller.forward().then((_) => _controller.reverse());
        }
        
        return InkWell(
          onTap: widget.onTap,
          child: AnimatedBuilder(
            animation: _scaleAnimation,
            builder: (context, child) {
              return Transform.scale(
                scale: _scaleAnimation.value,
                child: Badge(
                  showBadge: cartProvider.getCounter() > 0,
                  badgeContent: Text(
                    cartProvider.getCounter().toString(),
                    style: TextStyle(color: Colors.white, fontSize: 12),
                  ),
                  badgeColor: Colors.red,
                  position: BadgePosition.topEnd(top: -10, end: -10),
                  child: widget.child,
                ),
              );
            },
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

// Usage
AnimatedCartBadge(
  onTap: () => Navigator.push(
    context,
    MaterialPageRoute(builder: (_) => CartScreen()),
  ),
  child: Icon(Icons.shopping_cart, size: 28),
)
```

## UI Customizations

### 1. Custom Product Card

```dart
class ModernProductCard extends StatelessWidget {
  final String productName;
  final int price;
  final String unit;
  final String imageUrl;
  final VoidCallback onAddToCart;
  final bool isInCart;
  
  const ModernProductCard({
    Key? key,
    required this.productName,
    required this.price,
    required this.unit,
    required this.imageUrl,
    required this.onAddToCart,
    this.isInCart = false,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 8,
      shadowColor: Colors.black26,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
      ),
      child: Container(
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(16),
          gradient: LinearGradient(
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
            colors: [Colors.white, Colors.grey[50]!],
          ),
        ),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Product Image with Hero Animation
            Expanded(
              flex: 3,
              child: Hero(
                tag: 'product_$productName',
                child: Container(
                  width: double.infinity,
                  decoration: BoxDecoration(
                    borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
                  ),
                  child: ClipRRect(
                    borderRadius: BorderRadius.vertical(top: Radius.circular(16)),
                    child: Image.network(
                      imageUrl,
                      fit: BoxFit.cover,
                      loadingBuilder: (context, child, loadingProgress) {
                        if (loadingProgress == null) return child;
                        return Container(
                          color: Colors.grey[200],
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
                          color: Colors.grey[200],
                          child: Column(
                            mainAxisAlignment: MainAxisAlignment.center,
                            children: [
                              Icon(Icons.image_not_supported, 
                                   color: Colors.grey[400], size: 32),
                              Text('Image not available', 
                                   style: TextStyle(color: Colors.grey[600], fontSize: 10)),
                            ],
                          ),
                        );
                      },
                    ),
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
                    Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text(
                          productName,
                          style: TextStyle(
                            fontSize: 16,
                            fontWeight: FontWeight.bold,
                            color: Colors.grey[800],
                          ),
                          maxLines: 2,
                          overflow: TextOverflow.ellipsis,
                        ),
                        SizedBox(height: 4),
                        Text(
                          '$unit • \$${price}',
                          style: TextStyle(
                            fontSize: 14,
                            color: Colors.green[700],
                            fontWeight: FontWeight.w600,
                          ),
                        ),
                      ],
                    ),
                    
                    // Add to Cart Button
                    SizedBox(
                      width: double.infinity,
                      child: AnimatedContainer(
                        duration: Duration(milliseconds: 200),
                        child: ElevatedButton(
                          onPressed: isInCart ? null : onAddToCart,
                          style: ElevatedButton.styleFrom(
                            backgroundColor: isInCart ? Colors.grey : Colors.green,
                            foregroundColor: Colors.white,
                            shape: RoundedRectangleBorder(
                              borderRadius: BorderRadius.circular(8),
                            ),
                            elevation: isInCart ? 0 : 2,
                          ),
                          child: Row(
                            mainAxisAlignment: MainAxisAlignment.center,
                            mainAxisSize: MainAxisSize.min,
                            children: [
                              Icon(
                                isInCart ? Icons.check : Icons.add_shopping_cart,
                                size: 16,
                              ),
                              SizedBox(width: 4),
                              Text(
                                isInCart ? 'In Cart' : 'Add to Cart',
                                style: TextStyle(fontSize: 12),
                              ),
                            ],
                          ),
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 2. Enhanced Cart Item Widget

```dart
class EnhancedCartItemWidget extends StatefulWidget {
  final Cart cartItem;
  final VoidCallback onDelete;
  final Function(int) onQuantityChanged;
  
  const EnhancedCartItemWidget({
    Key? key,
    required this.cartItem,
    required this.onDelete,
    required this.onQuantityChanged,
  }) : super(key: key);
  
  @override
  _EnhancedCartItemWidgetState createState() => _EnhancedCartItemWidgetState();
}

class _EnhancedCartItemWidgetState extends State<EnhancedCartItemWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<Offset> _slideAnimation;
  bool _isDeleting = false;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 300),
      vsync: this,
    );
    _slideAnimation = Tween<Offset>(
      begin: Offset.zero,
      end: Offset(-1.0, 0.0),
    ).animate(CurvedAnimation(parent: _controller, curve: Curves.easeInOut));
  }
  
  @override
  Widget build(BuildContext context) {
    return SlideTransition(
      position: _slideAnimation,
      child: Card(
        margin: EdgeInsets.symmetric(horizontal: 16, vertical: 4),
        elevation: 2,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
        child: Dismissible(
          key: ValueKey(widget.cartItem.id),
          direction: DismissDirection.endToStart,
          background: Container(
            alignment: Alignment.centerRight,
            padding: EdgeInsets.only(right: 20),
            decoration: BoxDecoration(
              color: Colors.red,
              borderRadius: BorderRadius.circular(12),
            ),
            child: Icon(Icons.delete, color: Colors.white, size: 32),
          ),
          confirmDismiss: (direction) async {
            return await showDialog<bool>(
              context: context,
              builder: (context) => AlertDialog(
                title: Text('Remove Item'),
                content: Text('Remove ${widget.cartItem.productName} from cart?'),
                actions: [
                  TextButton(
                    onPressed: () => Navigator.pop(context, false),
                    child: Text('Cancel'),
                  ),
                  TextButton(
                    onPressed: () => Navigator.pop(context, true),
                    child: Text('Remove', style: TextStyle(color: Colors.red)),
                  ),
                ],
              ),
            );
          },
          onDismissed: (direction) {
            widget.onDelete();
          },
          child: Padding(
            padding: EdgeInsets.all(12),
            child: Row(
              children: [
                // Product Image
                Container(
                  width: 80,
                  height: 80,
                  decoration: BoxDecoration(
                    borderRadius: BorderRadius.circular(8),
                    boxShadow: [
                      BoxShadow(
                        color: Colors.black12,
                        blurRadius: 4,
                        offset: Offset(0, 2),
                      ),
                    ],
                  ),
                  child: ClipRRect(
                    borderRadius: BorderRadius.circular(8),
                    child: Image.network(
                      widget.cartItem.image!,
                      fit: BoxFit.cover,
                      errorBuilder: (context, error, stackTrace) {
                        return Container(
                          color: Colors.grey[300],
                          child: Icon(Icons.image_not_supported),
                        );
                      },
                    ),
                  ),
                ),
                
                SizedBox(width: 12),
                
                // Product Details
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        widget.cartItem.productName!,
                        style: TextStyle(
                          fontSize: 16,
                          fontWeight: FontWeight.bold,
                        ),
                        maxLines: 2,
                        overflow: TextOverflow.ellipsis,
                      ),
                      SizedBox(height: 4),
                      Text(
                        '${widget.cartItem.unitTag} • \$${widget.cartItem.initialPrice}',
                        style: TextStyle(
                          fontSize: 14,
                          color: Colors.grey[600],
                        ),
                      ),
                      SizedBox(height: 8),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(
                            'Total: \$${widget.cartItem.productPrice}',
                            style: TextStyle(
                              fontSize: 16,
                              fontWeight: FontWeight.bold,
                              color: Colors.green[700],
                            ),
                          ),
                          QuantitySelector(
                            quantity: widget.cartItem.quantity!,
                            onChanged: widget.onQuantityChanged,
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}

class QuantitySelector extends StatelessWidget {
  final int quantity;
  final Function(int) onChanged;
  
  const QuantitySelector({
    Key? key,
    required this.quantity,
    required this.onChanged,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        border: Border.all(color: Colors.grey[300]!),
        borderRadius: BorderRadius.circular(20),
      ),
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          IconButton(
            onPressed: () => onChanged(quantity - 1),
            icon: Icon(Icons.remove, size: 18),
            constraints: BoxConstraints(minWidth: 32, minHeight: 32),
          ),
          Container(
            padding: EdgeInsets.symmetric(horizontal: 8),
            child: Text(
              quantity.toString(),
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          IconButton(
            onPressed: () => onChanged(quantity + 1),
            icon: Icon(Icons.add, size: 18),
            constraints: BoxConstraints(minWidth: 32, minHeight: 32),
          ),
        ],
      ),
    );
  }
}
```

## Error Handling

### 1. Comprehensive Error Management

```dart
class CartErrorManager {
  static void handleCartOperation(
    BuildContext context,
    Future<void> operation,
    String successMessage,
    String errorMessage,
  ) async {
    try {
      await operation;
      _showSnackBar(context, successMessage, Colors.green);
    } on DatabaseException catch (e) {
      if (e.isUniqueConstraintError()) {
        _showSnackBar(context, 'Product already in cart', Colors.orange);
      } else {
        _showSnackBar(context, 'Database error: ${e.toString()}', Colors.red);
      }
    } on StateError catch (e) {
      _showSnackBar(context, 'State error: ${e.toString()}', Colors.red);
    } catch (e) {
      _showSnackBar(context, '$errorMessage: ${e.toString()}', Colors.red);
    }
  }
  
  static void _showSnackBar(BuildContext context, String message, Color color) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Row(
          children: [
            Icon(
              color == Colors.green ? Icons.check_circle : 
              color == Colors.orange ? Icons.warning : Icons.error,
              color: Colors.white,
            ),
            SizedBox(width: 8),
            Expanded(child: Text(message)),
          ],
        ),
        backgroundColor: color,
        behavior: SnackBarBehavior.floating,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
        duration: Duration(seconds: 3),
      ),
    );
  }
}

// Usage example
void addToCartWithErrorHandling(BuildContext context, Cart item) {
  final cartProvider = Provider.of<CartProvider>(context, listen: false);
  
  CartErrorManager.handleCartOperation(
    context,
    () async {
      await cartProvider.db.insert(item);
      cartProvider.addCounter();
      cartProvider.addTotalPrice(item.productPrice!.toDouble());
    },
    '${item.productName} added to cart',
    'Failed to add item to cart',
  );
}
```

### 2. Network Error Handling for Images

```dart
class RobustNetworkImage extends StatefulWidget {
  final String imageUrl;
  final double? width;
  final double? height;
  final BoxFit fit;
  
  const RobustNetworkImage({
    Key? key,
    required this.imageUrl,
    this.width,
    this.height,
    this.fit = BoxFit.cover,
  }) : super(key: key);
  
  @override
  _RobustNetworkImageState createState() => _RobustNetworkImageState();
}

class _RobustNetworkImageState extends State<RobustNetworkImage> {
  int _retryCount = 0;
  static const int maxRetries = 3;
  
  @override
  Widget build(BuildContext context) {
    return Image.network(
      widget.imageUrl,
      width: widget.width,
      height: widget.height,
      fit: widget.fit,
      loadingBuilder: (context, child, loadingProgress) {
        if (loadingProgress == null) return child;
        
        return Container(
          width: widget.width,
          height: widget.height,
          color: Colors.grey[200],
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              SizedBox(
                width: 24,
                height: 24,
                child: CircularProgressIndicator(
                  strokeWidth: 2,
                  value: loadingProgress.expectedTotalBytes != null
                    ? loadingProgress.cumulativeBytesLoaded / 
                      loadingProgress.expectedTotalBytes!
                    : null,
                ),
              ),
              SizedBox(height: 8),
              Text('Loading...', style: TextStyle(fontSize: 10)),
            ],
          ),
        );
      },
      errorBuilder: (context, error, stackTrace) {
        return Container(
          width: widget.width,
          height: widget.height,
          color: Colors.grey[300],
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.image_not_supported, color: Colors.grey[600]),
              SizedBox(height: 4),
              Text('Image unavailable', 
                   style: TextStyle(fontSize: 10, color: Colors.grey[600])),
              if (_retryCount < maxRetries) ...[
                SizedBox(height: 4),
                TextButton(
                  onPressed: () {
                    setState(() {
                      _retryCount++;
                    });
                  },
                  child: Text('Retry', style: TextStyle(fontSize: 10)),
                ),
              ],
            ],
          ),
        );
      },
    );
  }
}
```

## Testing Examples

### 1. Widget Test with Provider

```dart
// test/widget/cart_screen_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:provider/provider.dart';
import 'package:cartapp/screen/cart_screen.dart';
import 'package:cartapp/provider/cartprovider.dart';
import 'package:cartapp/model/cart_model.dart';

class MockCartProvider extends CartProvider {
  List<Cart> _mockItems = [
    Cart(
      id: 1,
      productId: '1',
      productName: 'Test Product',
      initialPrice: 10,
      productPrice: 30,
      quantity: 3,
      unitTag: 'KG',
      image: 'test_image.jpg',
    ),
  ];
  
  @override
  Future<List<Cart>> getData() async {
    return _mockItems;
  }
  
  @override
  int getCounter() => _mockItems.length;
  
  @override
  double getTotalPrice() => 30.0;
}

void main() {
  group('CartScreen Widget Tests', () {
    testWidgets('should display cart items', (WidgetTester tester) async {
      MockCartProvider mockProvider = MockCartProvider();
      
      await tester.pumpWidget(
        ChangeNotifierProvider<CartProvider>.value(
          value: mockProvider,
          child: MaterialApp(
            home: CartScreen(),
          ),
        ),
      );
      
      // Wait for FutureBuilder to complete
      await tester.pumpAndSettle();
      
      // Verify cart item is displayed
      expect(find.text('Test Product'), findsOneWidget);
      expect(find.text('3'), findsOneWidget); // Quantity
      expect(find.text('\$30'), findsOneWidget); // Price
    });
    
    testWidgets('should show empty cart message when no items', 
        (WidgetTester tester) async {
      MockCartProvider mockProvider = MockCartProvider();
      mockProvider._mockItems = []; // Empty cart
      
      await tester.pumpWidget(
        ChangeNotifierProvider<CartProvider>.value(
          value: mockProvider,
          child: MaterialApp(
            home: CartScreen(),
          ),
        ),
      );
      
      await tester.pumpAndSettle();
      
      // Verify empty cart message
      expect(find.text('Your cart is empty 😌'), findsOneWidget);
    });
  });
}
```

### 2. Integration Test Example

```dart
// test/integration/cart_flow_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:cartapp/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  group('Cart Flow Integration Tests', () {
    testWidgets('complete cart workflow', (WidgetTester tester) async {
      // Start the app
      app.main();
      await tester.pumpAndSettle();
      
      // Verify we're on product list screen
      expect(find.text('Product List'), findsOneWidget);
      
      // Add first product to cart
      await tester.tap(find.text('Add to cart').first);
      await tester.pumpAndSettle();
      
      // Verify success message
      expect(find.text('Product is added to cart'), findsOneWidget);
      
      // Verify cart badge shows 1
      expect(find.text('1'), findsOneWidget);
      
      // Navigate to cart screen
      await tester.tap(find.byIcon(Icons.shopping_bag_outlined));
      await tester.pumpAndSettle();
      
      // Verify we're on cart screen
      expect(find.text('Shopping Cart'), findsOneWidget);
      
      // Verify product is in cart
      expect(find.text('Mango'), findsOneWidget);
      
      // Test quantity increment
      await tester.tap(find.byIcon(Icons.add));
      await tester.pumpAndSettle();
      
      // Verify quantity increased
      expect(find.text('2'), findsOneWidget);
      
      // Test item deletion
      await tester.tap(find.byIcon(Icons.delete));
      await tester.pumpAndSettle();
      
      // Verify item was removed (empty cart state)
      expect(find.text('Your cart is empty 😌'), findsOneWidget);
    });
  });
}
```

## Performance Optimizations

### 1. Optimized Provider with Selectors

```dart
// Efficient cart counter widget
class OptimizedCartCounter extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Selector<CartProvider, int>(
      selector: (context, cartProvider) => cartProvider.getCounter(),
      builder: (context, counter, child) {
        return AnimatedSwitcher(
          duration: Duration(milliseconds: 200),
          child: Badge(
            key: ValueKey(counter),
            showBadge: counter > 0,
            badgeContent: Text(
              counter.toString(),
              style: TextStyle(color: Colors.white, fontSize: 12),
            ),
            child: Icon(Icons.shopping_cart),
          ),
        );
      },
    );
  }
}

// Efficient total price widget
class OptimizedTotalPrice extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Selector<CartProvider, double>(
      selector: (context, cartProvider) => cartProvider.getTotalPrice(),
      builder: (context, totalPrice, child) {
        return AnimatedSwitcher(
          duration: Duration(milliseconds: 300),
          child: Text(
            '\$${totalPrice.toStringAsFixed(2)}',
            key: ValueKey(totalPrice),
            style: TextStyle(
              fontSize: 24,
              fontWeight: FontWeight.bold,
              color: Colors.green[700],
            ),
          ),
        );
      },
    );
  }
}
```

### 2. Lazy Loading Implementation

```dart
class LazyLoadingProductList extends StatefulWidget {
  @override
  _LazyLoadingProductListState createState() => _LazyLoadingProductListState();
}

class _LazyLoadingProductListState extends State<LazyLoadingProductList> {
  final ScrollController _scrollController = ScrollController();
  List<Product> _products = [];
  bool _isLoading = false;
  bool _hasMoreData = true;
  int _currentPage = 0;
  static const int _pageSize = 10;
  
  @override
  void initState() {
    super.initState();
    _scrollController.addListener(_onScroll);
    _loadMoreProducts();
  }
  
  void _onScroll() {
    if (_scrollController.position.pixels >= 
        _scrollController.position.maxScrollExtent * 0.8 &&
        !_isLoading && _hasMoreData) {
      _loadMoreProducts();
    }
  }
  
  Future<void> _loadMoreProducts() async {
    if (_isLoading) return;
    
    setState(() {
      _isLoading = true;
    });
    
    try {
      // Simulate API call or load from local data
      List<Product> newProducts = await ProductService.getProductsPage(
        _currentPage, 
        _pageSize,
      );
      
      setState(() {
        _products.addAll(newProducts);
        _currentPage++;
        _hasMoreData = newProducts.length == _pageSize;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _isLoading = false;
      });
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Failed to load products')),
      );
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Products')),
      body: GridView.builder(
        controller: _scrollController,
        padding: EdgeInsets.all(16),
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          childAspectRatio: 0.75,
          crossAxisSpacing: 16,
          mainAxisSpacing: 16,
        ),
        itemCount: _products.length + (_hasMoreData ? 1 : 0),
        itemBuilder: (context, index) {
          if (index >= _products.length) {
            // Loading indicator at the end
            return Center(
              child: _isLoading 
                ? CircularProgressIndicator()
                : SizedBox.shrink(),
            );
          }
          
          return ProductCard(
            product: _products[index],
            onAddToCart: () => addToCart(context, _products[index]),
          );
        },
      ),
    );
  }
  
  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }
}

class ProductService {
  static Future<List<Product>> getProductsPage(int page, int pageSize) async {
    // Simulate network delay
    await Future.delayed(Duration(milliseconds: 500));
    
    // Return mock data for demonstration
    List<Product> allProducts = _getAllProducts();
    int startIndex = page * pageSize;
    int endIndex = startIndex + pageSize;
    
    if (startIndex >= allProducts.length) return [];
    
    endIndex = endIndex > allProducts.length ? allProducts.length : endIndex;
    return allProducts.sublist(startIndex, endIndex);
  }
  
  static List<Product> _getAllProducts() {
    // Return your complete product list
    return [
      // ... product data
    ];
  }
}
```

### 3. Memory-Efficient Image Loading

```dart
class MemoryEfficientImage extends StatefulWidget {
  final String imageUrl;
  final double? width;
  final double? height;
  
  @override
  _MemoryEfficientImageState createState() => _MemoryEfficientImageState();
}

class _MemoryEfficientImageState extends State<MemoryEfficientImage> {
  late ImageProvider _imageProvider;
  
  @override
  void initState() {
    super.initState();
    _imageProvider = NetworkImage(widget.imageUrl);
  }
  
  @override
  void dispose() {
    // Clear image from cache when widget is disposed
    _imageProvider.evict();
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Image(
      image: _imageProvider,
      width: widget.width,
      height: widget.height,
      fit: BoxFit.cover,
      frameBuilder: (context, child, frame, wasSynchronouslyLoaded) {
        if (wasSynchronouslyLoaded) return child;
        
        return AnimatedOpacity(
          opacity: frame == null ? 0 : 1,
          duration: Duration(milliseconds: 200),
          child: child,
        );
      },
      errorBuilder: (context, error, stackTrace) {
        return Container(
          width: widget.width,
          height: widget.height,
          color: Colors.grey[300],
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Icon(Icons.broken_image, color: Colors.grey[600]),
              Text('Failed to load', style: TextStyle(fontSize: 10)),
            ],
          ),
        );
      },
    );
  }
}
```

## Complete Implementation Example

### Full-Featured Shopping Cart Component

```dart
class AdvancedShoppingCart extends StatefulWidget {
  @override
  _AdvancedShoppingCartState createState() => _AdvancedShoppingCartState();
}

class _AdvancedShoppingCartState extends State<AdvancedShoppingCart> {
  final DBHelper _dbHelper = DBHelper();
  bool _isLoading = false;
  
  @override
  Widget build(BuildContext context) {
    return Consumer<CartProvider>(
      builder: (context, cartProvider, child) {
        return Scaffold(
          appBar: AppBar(
            title: Text('Shopping Cart'),
            actions: [
              if (cartProvider.getCounter() > 0)
                IconButton(
                  icon: Icon(Icons.clear_all),
                  onPressed: () => _clearCart(context),
                ),
              OptimizedCartCounter(),
            ],
          ),
          body: _isLoading
            ? Center(child: CircularProgressIndicator())
            : FutureBuilder<List<Cart>>(
                future: cartProvider.getData(),
                builder: (context, snapshot) {
                  if (snapshot.connectionState == ConnectionState.waiting) {
                    return Center(child: CircularProgressIndicator());
                  }
                  
                  if (snapshot.hasError) {
                    return Center(
                      child: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          Icon(Icons.error, size: 64, color: Colors.red),
                          SizedBox(height: 16),
                          Text('Error loading cart'),
                          ElevatedButton(
                            onPressed: () => setState(() {}),
                            child: Text('Retry'),
                          ),
                        ],
                      ),
                    );
                  }
                  
                  if (!snapshot.hasData || snapshot.data!.isEmpty) {
                    return _buildEmptyCart(context);
                  }
                  
                  return _buildCartContent(context, snapshot.data!);
                },
              ),
          bottomNavigationBar: _buildCheckoutSection(context),
        );
      },
    );
  }
  
  Widget _buildEmptyCart(BuildContext context) {
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.shopping_cart_outlined, size: 100, color: Colors.grey[400]),
          SizedBox(height: 24),
          Text(
            'Your cart is empty',
            style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
          ),
          SizedBox(height: 8),
          Text(
            'Add some products to get started',
            style: TextStyle(color: Colors.grey[600]),
          ),
          SizedBox(height: 24),
          ElevatedButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Continue Shopping'),
          ),
        ],
      ),
    );
  }
  
  Widget _buildCartContent(BuildContext context, List<Cart> items) {
    return Column(
      children: [
        Expanded(
          child: ListView.separated(
            padding: EdgeInsets.all(16),
            itemCount: items.length,
            separatorBuilder: (context, index) => SizedBox(height: 8),
            itemBuilder: (context, index) {
              return EnhancedCartItemWidget(
                cartItem: items[index],
                onDelete: () => _removeItem(context, items[index]),
                onQuantityChanged: (newQuantity) => 
                  _updateQuantity(context, items[index], newQuantity),
              );
            },
          ),
        ),
        _buildCartSummary(context),
      ],
    );
  }
  
  Widget _buildCartSummary(BuildContext context) {
    return Consumer<CartProvider>(
      builder: (context, cartProvider, child) {
        double subtotal = cartProvider.getTotalPrice();
        double tax = subtotal * 0.08;
        double discount = subtotal * 0.05;
        double total = subtotal + tax - discount;
        
        return Container(
          padding: EdgeInsets.all(16),
          decoration: BoxDecoration(
            color: Colors.grey[50],
            border: Border(top: BorderSide(color: Colors.grey[300]!)),
          ),
          child: Column(
            children: [
              _buildSummaryRow('Subtotal', subtotal),
              _buildSummaryRow('Tax (8%)', tax),
              _buildSummaryRow('Discount (5%)', -discount, isDiscount: true),
              Divider(),
              _buildSummaryRow('Total', total, isBold: true),
            ],
          ),
        );
      },
    );
  }
  
  Widget _buildSummaryRow(String label, double amount, {
    bool isDiscount = false,
    bool isBold = false,
  }) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 4),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Text(
            label,
            style: TextStyle(
              fontSize: isBold ? 18 : 16,
              fontWeight: isBold ? FontWeight.bold : FontWeight.normal,
            ),
          ),
          Text(
            '${isDiscount ? '-' : ''}\$${amount.abs().toStringAsFixed(2)}',
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
  
  Widget _buildCheckoutSection(BuildContext context) {
    return Consumer<CartProvider>(
      builder: (context, cartProvider, child) {
        if (cartProvider.getCounter() == 0) return SizedBox.shrink();
        
        return Container(
          padding: EdgeInsets.all(16),
          child: ElevatedButton(
            onPressed: () => _proceedToCheckout(context),
            style: ElevatedButton.styleFrom(
              backgroundColor: Colors.green,
              minimumSize: Size(double.infinity, 50),
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(25),
              ),
            ),
            child: Text(
              'Proceed to Checkout (\$${cartProvider.getTotalPrice().toStringAsFixed(2)})',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
          ),
        );
      },
    );
  }
  
  Future<void> _removeItem(BuildContext context, Cart item) async {
    setState(() {
      _isLoading = true;
    });
    
    try {
      await _dbHelper.delete(item.id!);
      final cartProvider = Provider.of<CartProvider>(context, listen: false);
      cartProvider.removerCounter();
      cartProvider.removeTotalPrice(item.productPrice!.toDouble());
      
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('${item.productName} removed'),
          backgroundColor: Colors.red,
        ),
      );
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Failed to remove item'),
          backgroundColor: Colors.red,
        ),
      );
    } finally {
      setState(() {
        _isLoading = false;
      });
    }
  }
  
  Future<void> _updateQuantity(BuildContext context, Cart item, int newQuantity) async {
    if (newQuantity <= 0) {
      await _removeItem(context, item);
      return;
    }
    
    int oldQuantity = item.quantity!;
    int priceDifference = item.initialPrice! * (newQuantity - oldQuantity);
    
    Cart updatedItem = item.copyWith(
      quantity: newQuantity,
      productPrice: item.initialPrice! * newQuantity,
    );
    
    try {
      await _dbHelper.updateQuantity(updatedItem);
      final cartProvider = Provider.of<CartProvider>(context, listen: false);
      cartProvider.addTotalPrice(priceDifference.toDouble());
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Failed to update quantity'),
          backgroundColor: Colors.red,
        ),
      );
    }
  }
  
  Future<void> _clearCart(BuildContext context) async {
    bool? confirm = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Clear Cart'),
        content: Text('Remove all items from cart?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: Text('Cancel'),
          ),
          TextButton(
            onPressed: () => Navigator.pop(context, true),
            child: Text('Clear', style: TextStyle(color: Colors.red)),
          ),
        ],
      ),
    );
    
    if (confirm == true) {
      final cartProvider = Provider.of<CartProvider>(context, listen: false);
      List<Cart> items = await cartProvider.getData();
      
      for (Cart item in items) {
        await _dbHelper.delete(item.id!);
        cartProvider.removerCounter();
        cartProvider.removeTotalPrice(item.productPrice!.toDouble());
      }
    }
  }
  
  void _proceedToCheckout(BuildContext context) {
    // Implement checkout logic
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Checkout'),
        content: Text('Checkout functionality would be implemented here'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('OK'),
          ),
        ],
      ),
    );
  }
}
```

This comprehensive code examples document provides practical, ready-to-use implementations for common shopping cart operations, advanced patterns, and performance optimizations. Use these examples as starting points for your own implementations and customizations.