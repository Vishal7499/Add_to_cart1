# Documentation Index

Welcome to the Flutter Shopping Cart App documentation. This comprehensive documentation covers all aspects of the application from API references to development guidelines.

## 📚 Documentation Structure

### Core Documentation
- **[API Documentation](API_DOCUMENTATION.md)** - Complete API reference for all classes, methods, and components
- **[Usage Examples](USAGE_EXAMPLES.md)** - Practical code examples and implementation patterns
- **[Component Guide](COMPONENT_GUIDE.md)** - Detailed component breakdown and customization options
- **[Developer Guide](DEVELOPER_GUIDE.md)** - Development environment setup, testing, and deployment

### Quick Start
- **[Project README](../README.md)** - Project overview, features, and basic setup instructions

## 🎯 Documentation by Role

### For Developers
1. Start with [Project README](../README.md) for overview
2. Follow [Developer Guide](DEVELOPER_GUIDE.md) for environment setup
3. Reference [API Documentation](API_DOCUMENTATION.md) for implementation details
4. Use [Usage Examples](USAGE_EXAMPLES.md) for common patterns

### For Contributors
1. Read [Developer Guide - Contributing Guidelines](DEVELOPER_GUIDE.md#contributing-guidelines)
2. Review [Component Guide](COMPONENT_GUIDE.md) for architecture understanding
3. Check [API Documentation](API_DOCUMENTATION.md) for interface contracts

### For Maintainers
1. [Developer Guide - Testing Strategy](DEVELOPER_GUIDE.md#testing-strategy)
2. [Developer Guide - Performance Optimization](DEVELOPER_GUIDE.md#performance-optimization)
3. [Developer Guide - Deployment](DEVELOPER_GUIDE.md#deployment)

## 🔍 Quick Reference

### Key Components
| Component | Purpose | Documentation |
|-----------|---------|---------------|
| `DBHelper` | Database operations | [API Docs](API_DOCUMENTATION.md#database-helper-dbhelper) |
| `Cart` | Data model | [API Docs](API_DOCUMENTATION.md#cart-model) |
| `CartProvider` | State management | [API Docs](API_DOCUMENTATION.md#cart-provider) |
| `ProductList` | Product catalog UI | [API Docs](API_DOCUMENTATION.md#productlist-screen) |
| `CartScreen` | Shopping cart UI | [API Docs](API_DOCUMENTATION.md#cartscreen) |

### Common Tasks
| Task | Reference |
|------|-----------|
| Add new product | [Usage Examples - Database Operations](USAGE_EXAMPLES.md#database-operations) |
| Customize UI | [Component Guide - Customization](COMPONENT_GUIDE.md#customization-guide) |
| Add tests | [Developer Guide - Testing](DEVELOPER_GUIDE.md#testing-strategy) |
| Deploy app | [Developer Guide - Deployment](DEVELOPER_GUIDE.md#deployment) |
| Handle errors | [Usage Examples - Error Handling](USAGE_EXAMPLES.md#error-handling) |

## 📱 Application Overview

### Architecture
```
UI Layer (Screens & Widgets)
    ↕
State Layer (CartProvider)
    ↕
Data Layer (DBHelper & Cart Model)
    ↕
Persistence (SQLite + SharedPreferences)
```

### Key Features
- **Product Catalog**: Grid-based product browsing
- **Shopping Cart**: Add/remove items with quantity management
- **Local Storage**: SQLite for cart persistence
- **State Management**: Provider pattern for reactive UI
- **Real-time Updates**: Live cart counter and total calculations

### Technology Stack
- **Framework**: Flutter 3.x
- **Language**: Dart 2.18+
- **State Management**: Provider package
- **Database**: SQLite (sqflite package)
- **Local Storage**: SharedPreferences
- **UI Components**: Material Design

## 🚀 Getting Started

### 1. Quick Setup
```bash
# Clone and setup
git clone <repository-url>
cd AddToCart-main
flutter pub get
flutter run
```

### 2. Development Setup
Follow the complete [Developer Guide](DEVELOPER_GUIDE.md#development-environment-setup) for detailed setup instructions.

### 3. First Contribution
1. Read [Contributing Guidelines](DEVELOPER_GUIDE.md#contributing-guidelines)
2. Check [Component Guide](COMPONENT_GUIDE.md) for architecture understanding
3. Review [Usage Examples](USAGE_EXAMPLES.md) for implementation patterns

## 🔧 API Quick Reference

### Database Operations
```dart
// Add item to cart
DBHelper dbHelper = DBHelper();
await dbHelper.insert(cartItem);

// Get all cart items
List<Cart> items = await dbHelper.getCartList();

// Update item quantity
await dbHelper.updateQuantity(updatedItem);

// Delete item
await dbHelper.delete(itemId);
```

### State Management
```dart
// Access cart provider
final cartProvider = Provider.of<CartProvider>(context);

// Update cart state
cartProvider.addCounter();
cartProvider.addTotalPrice(price);

// Listen to changes
Consumer<CartProvider>(
  builder: (context, cartProvider, child) {
    return Text('Items: ${cartProvider.getCounter()}');
  },
)
```

### UI Components
```dart
// Product list screen
Navigator.push(context, 
  MaterialPageRoute(builder: (_) => ProductList()));

// Cart screen
Navigator.push(context, 
  MaterialPageRoute(builder: (_) => CartScreen()));

// Reusable summary widget
ReusableWidget(
  title: 'Total',
  value: '\$${totalPrice.toStringAsFixed(2)}',
)
```

## 📖 Documentation Standards

### Code Documentation
- All public APIs must have dartdoc comments
- Include usage examples for complex methods
- Document side effects and exceptions
- Provide parameter descriptions

### File Organization
```
docs/
├── README.md              # This index file
├── API_DOCUMENTATION.md   # Complete API reference
├── USAGE_EXAMPLES.md      # Code examples and patterns
├── COMPONENT_GUIDE.md     # Component details and customization
└── DEVELOPER_GUIDE.md     # Development and deployment guide
```

### Documentation Updates
- Update documentation when adding new features
- Include examples for new APIs
- Maintain backward compatibility notes
- Update version information

## 🤝 Support and Community

### Getting Help
1. Check existing documentation first
2. Search through [Usage Examples](USAGE_EXAMPLES.md) for similar patterns
3. Review [API Documentation](API_DOCUMENTATION.md) for method details
4. Create an issue with detailed information

### Contributing to Documentation
1. Follow the existing documentation structure
2. Include practical examples
3. Test all code examples
4. Update the index when adding new sections

---

**Happy Coding! 🚀**

*Last updated: $(date)*