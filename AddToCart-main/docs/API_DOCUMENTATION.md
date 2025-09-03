# Flutter Cart App - API Documentation

## Table of Contents
- [Overview](#overview)
- [Database Helper (DBHelper)](#database-helper-dbhelper)
- [Cart Model](#cart-model)
- [Cart Provider](#cart-provider)
- [UI Components](#ui-components)
  - [ProductList Screen](#productlist-screen)
  - [CartScreen](#cartscreen)
  - [ReusableWidget](#reusablewidget)

## Overview

This Flutter application implements a shopping cart system with local database persistence, state management using Provider pattern, and a clean UI for product browsing and cart management.

**Key Features:**
- Product catalog with grid layout
- Add/remove products from cart
- Quantity management
- Local SQLite database persistence
- Real-time cart counter and total price calculation
- Responsive UI with animations

---

## Database Helper (DBHelper)

The `DBHelper` class provides SQLite database operations for cart persistence.

### Class Definition
```dart
class DBHelper
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `_db` | `Database?` | Static database instance (singleton pattern) |

### Methods

#### `get db`
```dart
Future<Database?> get db async
```
**Description:** Returns the database instance, initializing it if not already created.

**Returns:** `Future<Database?>` - The SQLite database instance

**Example:**
```dart
DBHelper dbHelper = DBHelper();
Database? database = await dbHelper.db;
```

---

#### `initDatabase()`
```dart
initDatabase() async
```
**Description:** Initializes the SQLite database with the cart table schema.

**Returns:** `Future<Database>` - The initialized database

**Database Schema:**
- Table: `cart`
- Columns:
  - `id` (INTEGER PRIMARY KEY)
  - `productId` (VARCHAR UNIQUE)
  - `productName` (TEXT)
  - `initialPrice` (INTEGER)
  - `productPrice` (INTEGER)
  - `quantity` (INTEGER)
  - `unitTag` (TEXT)
  - `image` (TEXT)

**Example:**
```dart
DBHelper dbHelper = DBHelper();
await dbHelper.initDatabase();
```

---

#### `insert(Cart cart)`
```dart
Future<Cart> insert(Cart cart) async
```
**Description:** Inserts a new cart item into the database.

**Parameters:**
- `cart` (Cart) - The cart item to insert

**Returns:** `Future<Cart>` - The inserted cart item

**Throws:** Database exception if product already exists (due to UNIQUE constraint on productId)

**Example:**
```dart
Cart newItem = Cart(
  id: 1,
  productId: "1",
  productName: "Mango",
  initialPrice: 10,
  productPrice: 10,
  quantity: 1,
  unitTag: "KG",
  image: "image_url"
);

await dbHelper.insert(newItem);
```

---

#### `getCartList()`
```dart
Future<List<Cart>> getCartList() async
```
**Description:** Retrieves all cart items from the database.

**Returns:** `Future<List<Cart>>` - List of all cart items

**Example:**
```dart
List<Cart> cartItems = await dbHelper.getCartList();
print('Cart has ${cartItems.length} items');
```

---

#### `delete(int id)`
```dart
Future<int> delete(int id) async
```
**Description:** Deletes a cart item by its ID.

**Parameters:**
- `id` (int) - The ID of the cart item to delete

**Returns:** `Future<int>` - Number of rows affected (1 if successful, 0 if not found)

**Example:**
```dart
int deletedRows = await dbHelper.delete(1);
if (deletedRows > 0) {
  print('Item deleted successfully');
}
```

---

#### `updateQuantity(Cart cart)`
```dart
Future<int> updateQuantity(Cart cart) async
```
**Description:** Updates the quantity and price of an existing cart item.

**Parameters:**
- `cart` (Cart) - The cart item with updated quantity and price

**Returns:** `Future<int>` - Number of rows affected

**Example:**
```dart
Cart updatedItem = Cart(
  id: 1,
  productId: "1",
  productName: "Mango",
  initialPrice: 10,
  productPrice: 30, // 10 * 3 quantity
  quantity: 3,
  unitTag: "KG",
  image: "image_url"
);

await dbHelper.updateQuantity(updatedItem);
```

---

## Cart Model

The `Cart` class represents a shopping cart item with all necessary product information.

### Class Definition
```dart
class Cart
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | `int?` | Unique identifier for the cart item |
| `productId` | `String?` | Product identifier |
| `productName` | `String?` | Name of the product |
| `initialPrice` | `int?` | Original unit price of the product |
| `productPrice` | `int?` | Total price (initialPrice × quantity) |
| `quantity` | `int?` | Number of items in cart |
| `unitTag` | `String?` | Unit of measurement (KG, Dozen, etc.) |
| `image` | `String?` | URL or path to product image |

### Constructors

#### Default Constructor
```dart
Cart({
  required this.id,
  required this.productId,
  required this.productName,
  required this.initialPrice,
  required this.productPrice,
  required this.quantity,
  required this.unitTag,
  required this.image
})
```

**Example:**
```dart
Cart cartItem = Cart(
  id: 1,
  productId: "mango_001",
  productName: "Fresh Mango",
  initialPrice: 15,
  productPrice: 45, // 15 * 3
  quantity: 3,
  unitTag: "KG",
  image: "https://example.com/mango.jpg"
);
```

#### From Map Constructor
```dart
Cart.fromMap(Map<dynamic, dynamic> res)
```
**Description:** Creates a Cart instance from a database map result.

**Parameters:**
- `res` (Map<dynamic, dynamic>) - Database row as a map

**Example:**
```dart
Map<String, dynamic> dbRow = {
  'id': 1,
  'productId': 'mango_001',
  'productName': 'Fresh Mango',
  // ... other fields
};

Cart cartItem = Cart.fromMap(dbRow);
```

### Methods

#### `toMap()`
```dart
Map<String, Object?> toMap()
```
**Description:** Converts the Cart instance to a map for database operations.

**Returns:** `Map<String, Object?>` - Map representation of the cart item

**Example:**
```dart
Cart cartItem = Cart(/* ... */);
Map<String, Object?> cartMap = cartItem.toMap();
// Use for database insert/update operations
```

---

## Cart Provider

The `CartProvider` class manages the application state for cart operations using the Provider pattern.

### Class Definition
```dart
class CartProvider with ChangeNotifier
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `db` | `DBHelper` | Database helper instance |
| `counter` | `int` | Total number of items in cart (read-only) |
| `totalPrice` | `double` | Total price of all cart items (read-only) |
| `cart` | `Future<List<Cart>>` | Future containing cart items list (read-only) |

### Methods

#### `getData()`
```dart
Future<List<Cart>> getData() async
```
**Description:** Fetches cart data from the database.

**Returns:** `Future<List<Cart>>` - List of cart items

**Example:**
```dart
CartProvider cartProvider = CartProvider();
List<Cart> cartItems = await cartProvider.getData();
```

---

#### `addTotalPrice(double productPrice)`
```dart
void addTotalPrice(double productPrice)
```
**Description:** Adds a product price to the total cart price and persists to SharedPreferences.

**Parameters:**
- `productPrice` (double) - Price to add to total

**Side Effects:**
- Updates `_totalPrice`
- Saves to SharedPreferences
- Notifies listeners

**Example:**
```dart
cartProvider.addTotalPrice(25.99);
```

---

#### `removeTotalPrice(double productPrice)`
```dart
void removeTotalPrice(double productPrice)
```
**Description:** Removes a product price from the total cart price and persists to SharedPreferences.

**Parameters:**
- `productPrice` (double) - Price to remove from total

**Side Effects:**
- Updates `_totalPrice`
- Saves to SharedPreferences
- Notifies listeners

**Example:**
```dart
cartProvider.removeTotalPrice(25.99);
```

---

#### `getTotalPrice()`
```dart
double getTotalPrice()
```
**Description:** Retrieves the current total price from SharedPreferences.

**Returns:** `double` - Current total price

**Example:**
```dart
double total = cartProvider.getTotalPrice();
print('Cart total: \$${total.toStringAsFixed(2)}');
```

---

#### `addCounter()`
```dart
void addCounter()
```
**Description:** Increments the cart item counter and persists to SharedPreferences.

**Side Effects:**
- Increments `_counter`
- Saves to SharedPreferences
- Notifies listeners

**Example:**
```dart
cartProvider.addCounter(); // Adds 1 to cart count
```

---

#### `removerCounter()`
```dart
void removerCounter()
```
**Description:** Decrements the cart item counter and persists to SharedPreferences.

**Side Effects:**
- Decrements `_counter`
- Saves to SharedPreferences
- Notifies listeners

**Example:**
```dart
cartProvider.removerCounter(); // Removes 1 from cart count
```

---

#### `getCounter()`
```dart
int getCounter()
```
**Description:** Retrieves the current cart item counter from SharedPreferences.

**Returns:** `int` - Current cart item count

**Example:**
```dart
int itemCount = cartProvider.getCounter();
print('Cart has $itemCount items');
```

---

## UI Components

### ProductList Screen

The main screen displaying a grid of available products.

#### Class Definition
```dart
class ProductList extends StatefulWidget
```

#### Features
- Grid layout with customizable product display
- Add to cart functionality
- Cart badge with item counter
- Navigation to cart screen
- Snackbar notifications for user feedback

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `productName` | `List<String>` | List of product names |
| `productUnit` | `List<String>` | List of product units (KG, Dozen, etc.) |
| `productPrice` | `List<int>` | List of product prices |
| `productImage` | `List<String>` | List of product image URLs |
| `dbHelper` | `DBHelper?` | Database helper instance |

#### Usage Example
```dart
// Navigate to ProductList screen
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => ProductList()),
);
```

#### Customization
To modify the product catalog, update the following lists in `_ProductListState`:

```dart
List<String> productName = [
  'Your Product 1',
  'Your Product 2',
  // Add more products
];

List<String> productUnit = [
  'KG',
  'Piece',
  // Corresponding units
];

List<int> productPrice = [10, 25, /* prices */];

List<String> productImage = [
  'https://your-image-url-1.jpg',
  'https://your-image-url-2.jpg',
  // Image URLs
];
```

---

### CartScreen

Displays cart items with quantity management and total price calculation.

#### Class Definition
```dart
class CartScreen extends StatefulWidget
```

#### Features
- List view of cart items
- Quantity increment/decrement controls
- Item deletion functionality
- Total price calculation with discount display
- Empty cart state handling

#### Usage Example
```dart
// Navigate to CartScreen
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => CartScreen()),
);
```

#### Integration with CartProvider
```dart
// Access cart data in CartScreen
Consumer<CartProvider>(
  builder: (context, cartProvider, child) {
    return FutureBuilder<List<Cart>>(
      future: cartProvider.getData(),
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          // Display cart items
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, index) {
              // Cart item UI
            },
          );
        }
        return CircularProgressIndicator();
      },
    );
  },
)
```

---

### ReusableWidget

A utility widget for displaying key-value pairs in the cart summary.

#### Class Definition
```dart
class ReusableWidget extends StatelessWidget
```

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `title` | `String` | The label text (e.g., "Sub Total") |
| `value` | `String` | The value text (e.g., "$25.99") |

#### Constructor
```dart
const ReusableWidget({
  required this.title,
  required this.value
})
```

#### Usage Example
```dart
ReusableWidget(
  title: 'Sub Total',
  value: r'$' + totalPrice.toStringAsFixed(2),
)

ReusableWidget(
  title: 'Tax (8%)',
  value: r'$' + taxAmount.toStringAsFixed(2),
)

ReusableWidget(
  title: 'Total',
  value: r'$' + finalTotal.toStringAsFixed(2),
)
```

---

## Complete Integration Example

Here's how all components work together:

```dart
// 1. Setup Provider in main.dart
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => CartProvider(),
      child: MyApp(),
    ),
  );
}

// 2. Add item to cart (in ProductList)
void addToCart(int index) {
  final cart = Provider.of<CartProvider>(context, listen: false);
  
  dbHelper!.insert(Cart(
    id: index,
    productId: index.toString(),
    productName: productName[index],
    initialPrice: productPrice[index],
    productPrice: productPrice[index],
    quantity: 1,
    unitTag: productUnit[index],
    image: productImage[index]
  )).then((value) {
    cart.addTotalPrice(productPrice[index].toDouble());
    cart.addCounter();
    // Show success message
  }).onError((error, stackTrace) {
    // Handle duplicate item error
  });
}

// 3. Update quantity in cart (in CartScreen)
void updateQuantity(Cart item, bool increment) {
  final cart = Provider.of<CartProvider>(context, listen: false);
  
  int newQuantity = increment ? item.quantity! + 1 : item.quantity! - 1;
  int newPrice = item.initialPrice! * newQuantity;
  
  if (newQuantity > 0) {
    Cart updatedItem = Cart(
      id: item.id,
      productId: item.productId,
      productName: item.productName,
      initialPrice: item.initialPrice,
      productPrice: newPrice,
      quantity: newQuantity,
      unitTag: item.unitTag,
      image: item.image
    );
    
    dbHelper!.updateQuantity(updatedItem).then((_) {
      if (increment) {
        cart.addTotalPrice(item.initialPrice!.toDouble());
      } else {
        cart.removeTotalPrice(item.initialPrice!.toDouble());
      }
    });
  }
}

// 4. Listen to cart changes in UI
Consumer<CartProvider>(
  builder: (context, cartProvider, child) {
    return Text(
      'Items: ${cartProvider.getCounter()}',
      style: TextStyle(fontSize: 16),
    );
  },
)
```

---

## Error Handling

### Database Errors
- **Duplicate Product**: When trying to add a product that already exists in cart
- **Database Not Found**: When database initialization fails
- **Update Failures**: When quantity updates fail

### Common Error Patterns
```dart
// Handle duplicate product insertion
dbHelper.insert(cartItem).onError((error, stackTrace) {
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(
      content: Text('Product is already added in cart'),
      backgroundColor: Colors.red,
    ),
  );
});

// Handle database operations safely
try {
  await dbHelper.delete(itemId);
} catch (e) {
  print('Delete operation failed: $e');
}
```

---

## State Management Flow

1. **Product Addition Flow:**
   - User taps "Add to cart" → `DBHelper.insert()` → `CartProvider.addCounter()` & `addTotalPrice()` → UI updates

2. **Quantity Update Flow:**
   - User taps +/- → `DBHelper.updateQuantity()` → `CartProvider.addTotalPrice()`/`removeTotalPrice()` → UI updates

3. **Item Deletion Flow:**
   - User taps delete → `DBHelper.delete()` → `CartProvider.removerCounter()` & `removeTotalPrice()` → UI updates

4. **Data Persistence:**
   - All cart counters and totals are saved to `SharedPreferences`
   - Cart items are persisted in SQLite database
   - State is restored when app restarts
