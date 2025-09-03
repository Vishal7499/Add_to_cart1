# Flutter Shopping Cart App

A feature-rich Flutter application demonstrating shopping cart functionality with local database persistence, state management, and modern UI design.

![Flutter](https://img.shields.io/badge/Flutter-02569B?style=for-the-badge&logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-0175C2?style=for-the-badge&logo=dart&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)

## 📱 Features

- **Product Catalog**: Grid-based product display with images and pricing
- **Shopping Cart**: Add, remove, and manage product quantities
- **Local Persistence**: SQLite database for cart data persistence
- **State Management**: Provider pattern for reactive UI updates
- **Real-time Updates**: Live cart counter and total price calculation
- **User Feedback**: Snackbar notifications and animations
- **Responsive Design**: Adaptive grid layout for different screen sizes

## 🛠️ Technologies Used

- **Flutter SDK**: Cross-platform mobile development framework
- **Provider**: State management solution
- **SQLite**: Local database for data persistence
- **SharedPreferences**: Simple key-value storage
- **Badges**: Animated badge widgets for cart counter

## 📋 Dependencies

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2
  shared_preferences: ^2.0.15
  path_provider: ^2.0.11
  sqflite: ^2.2.0+3
  badges: ^2.0.3
  provider: ^6.0.4

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^2.0.0
```

## 🚀 Getting Started

### Prerequisites

- Flutter SDK (>=2.18.4 <3.0.0)
- Dart SDK
- Android Studio / VS Code
- Android/iOS device or emulator

### Installation

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd AddToCart-main
   ```

2. **Install dependencies:**
   ```bash
   flutter pub get
   ```

3. **Run the application:**
   ```bash
   flutter run
   ```

### Building for Release

```bash
# Android APK
flutter build apk --release

# Android App Bundle
flutter build appbundle --release

# iOS (requires macOS)
flutter build ios --release
```

## 📁 Project Structure

```
lib/
├── main.dart                 # Application entry point
├── helper/
│   └── db_helper.dart       # SQLite database operations
├── model/
│   └── cart_model.dart      # Cart item data model
├── provider/
│   └── cartprovider.dart    # State management provider
└── screen/
    ├── product_list.dart    # Main product catalog screen
    └── cart_screen.dart     # Shopping cart screen
```

## 🎯 Core Components

### 1. Database Layer (`DBHelper`)
Handles SQLite operations for cart persistence:
- Create/initialize database
- Insert new cart items
- Update item quantities
- Delete cart items
- Retrieve cart data

### 2. Data Model (`Cart`)
Represents cart items with properties:
- Product information (name, price, image)
- Quantity and unit tags
- Database mapping methods

### 3. State Management (`CartProvider`)
Manages application state using Provider pattern:
- Cart item counter
- Total price calculation
- SharedPreferences integration
- Reactive UI updates

### 4. User Interface
- **ProductList**: Grid-based product catalog
- **CartScreen**: Cart management with quantity controls
- **ReusableWidget**: Utility widget for displaying key-value pairs

## 💡 Usage Examples

### Adding Products to Cart

```dart
// In ProductList screen
void addProductToCart(int index) {
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
    // Update cart state
    cart.addTotalPrice(productPrice[index].toDouble());
    cart.addCounter();
    
    // Show success feedback
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Product added to cart'),
        backgroundColor: Colors.green,
      ),
    );
  });
}
```

### Managing Cart State

```dart
// Listen to cart changes
Consumer<CartProvider>(
  builder: (context, cartProvider, child) {
    return Badge(
      badgeContent: Text(
        cartProvider.getCounter().toString(),
        style: TextStyle(color: Colors.white),
      ),
      child: Icon(Icons.shopping_bag_outlined),
    );
  },
)
```

### Updating Item Quantities

```dart
// Increment quantity
void incrementQuantity(Cart item) {
  int newQuantity = item.quantity! + 1;
  int newPrice = item.initialPrice! * newQuantity;
  
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
    cartProvider.addTotalPrice(item.initialPrice!.toDouble());
  });
}
```

## 🎨 Customization

### Adding New Products

1. Update the product arrays in `ProductList`:
   ```dart
   List<String> productName = [
     'Existing Product',
     'New Product Name',  // Add here
   ];
   
   List<String> productUnit = [
     'KG',
     'Piece',  // Add corresponding unit
   ];
   
   List<int> productPrice = [
     10,
     25,  // Add price
   ];
   
   List<String> productImage = [
     'existing_url',
     'new_image_url',  // Add image URL
   ];
   ```

### Modifying UI Theme

Update the theme in `main.dart`:
```dart
MaterialApp(
  theme: ThemeData(
    primarySwatch: Colors.blue,  // Change primary color
    // Add custom theme properties
  ),
  home: const ProductList(),
)
```

### Database Schema Changes

If you need to modify the cart table structure:

1. Update the `_onCreate` method in `DBHelper`:
   ```dart
   _onCreate(Database db, int version) async {
     await db.execute(
       'CREATE TABLE cart (
         id INTEGER PRIMARY KEY,
         productId VARCHAR UNIQUE,
         productName TEXT,
         initialPrice INTEGER,
         productPrice INTEGER,
         quantity INTEGER,
         unitTag TEXT,
         image TEXT,
         // Add new columns here
       )'
     );
   }
   ```

2. Update the `Cart` model accordingly
3. Increment the database version number

## 🧪 Testing

### Running Tests

```bash
# Run all tests
flutter test

# Run specific test file
flutter test test/widget_test.dart
```

### Test Coverage

```bash
# Generate coverage report
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
```

## 🐛 Troubleshooting

### Common Issues

1. **Database Errors:**
   ```
   Error: Product is already added in cart
   ```
   - **Cause**: Trying to add duplicate product (UNIQUE constraint violation)
   - **Solution**: Check if product exists before insertion or handle the error gracefully

2. **State Not Updating:**
   - **Cause**: Missing `notifyListeners()` call in provider
   - **Solution**: Ensure all state-changing methods call `notifyListeners()`

3. **Image Loading Issues:**
   - **Cause**: Network image URLs not accessible
   - **Solution**: Verify image URLs or implement error handling with placeholder images

### Debug Mode

Enable debug prints by uncommenting debug statements in the code:
```dart
// In ProductList onTap method
print(index);
print(productName[index].toString());
print(productPrice[index].toString());
```

## 📚 Additional Resources

- [Flutter Documentation](https://docs.flutter.dev/)
- [Provider Package](https://pub.dev/packages/provider)
- [SQLite Plugin](https://pub.dev/packages/sqflite)
- [SharedPreferences](https://pub.dev/packages/shared_preferences)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📞 Support

For support and questions:
- Create an issue in the repository
- Review the [API Documentation](docs/API_DOCUMENTATION.md)
- Check the troubleshooting section above

---

**Happy Coding! 🚀**