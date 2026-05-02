Below is a **complete senior-level guide for using Flutter + Bloc + Cubit in a real e-commerce project**, written the way production teams usually structure it 🧩🚀📦

I’ll cover:

1. **Why Bloc/Cubit is used**
2. **Project structure used by senior teams**
3. **Installing packages**
4. **Cubit lifecycle**
5. **States design**
6. **E-commerce example**
7. **Products module**
8. **Cart module**
9. **Authentication module**
10. **API integration**
11. **Dependency Injection**
12. **Best senior practices**

---

# 1) Why Bloc / Cubit in Flutter

Flutter + Bloc is one of the most stable enterprise combinations because:

✅ clear separation of UI and business logic
✅ scalable for large projects
✅ easy testing
✅ predictable state flow
✅ reusable modules

---

# 2) Install Required Packages

In `pubspec.yaml`

```yaml
dependencies:
  flutter:
    sdk: flutter

  flutter_bloc: ^8.1.4
  equatable: ^2.0.5
  dio: ^5.4.0
  get_it: ^7.6.7
```

---

# 3) Senior Project Structure

```plaintext
lib/
│
├── core/
│   ├── network/
│   ├── constants/
│   ├── errors/
│   └── utils/
│
├── features/
│   ├── products/
│   │   ├── data/
│   │   ├── domain/
│   │   ├── presentation/
│   │
│   ├── cart/
│   ├── auth/
│
├── injection_container.dart
│
└── main.dart
```

This is **feature-first clean architecture** 🧠

Senior teams avoid old MVC style because features scale better.

---

# 4) Bloc vs Cubit

## Cubit

Simple actions:

```plaintext
loadProducts()
addToCart()
login()
```

## Bloc

Complex event system:

```plaintext
LoadProductsEvent
AddProductEvent
RemoveProductEvent
```

For e-commerce:

✅ Start with Cubit
✅ Use Bloc only when flows become complex

---

# 5) Cubit Lifecycle

Cubit always:

```plaintext
Action -> Emit State -> UI Rebuild
```

Example:

```dart
loadProducts()
emit(Loading)
emit(Success)
```

---

# 6) Product Model

```dart
class Product {
  final int id;
  final String name;
  final double price;

  Product({
    required this.id,
    required this.name,
    required this.price,
  });
}
```

---

# 7) Product States

Use `equatable`

```dart
import 'package:equatable/equatable.dart';

abstract class ProductState extends Equatable {
  @override
  List<Object?> get props => [];
}
```

---

## Initial State

```dart
class ProductInitial extends ProductState {}
```

---

## Loading State

```dart
class ProductLoading extends ProductState {}
```

---

## Success State

```dart
class ProductLoaded extends ProductState {
  final List<Product> products;

  ProductLoaded(this.products);

  @override
  List<Object?> get props => [products];
}
```

---

## Error State

```dart
class ProductError extends ProductState {
  final String message;

  ProductError(this.message);

  @override
  List<Object?> get props => [message];
}
```

---

# 8) Product Cubit

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class ProductCubit extends Cubit<ProductState> {
  ProductCubit() : super(ProductInitial());

  Future<void> loadProducts() async {
    emit(ProductLoading());

    try {
      await Future.delayed(Duration(seconds: 2));

      final products = [
        Product(id: 1, name: 'Laptop', price: 999),
        Product(id: 2, name: 'Phone', price: 799),
      ];

      emit(ProductLoaded(products));
    } catch (e) {
      emit(ProductError(e.toString()));
    }
  }
}
```

---

# 9) Provide Cubit in App

```dart
void main() {
  runApp(
    BlocProvider(
      create: (_) => ProductCubit(),
      child: MyApp(),
    ),
  );
}
```

---

# 10) UI Consumption

```dart
BlocBuilder<ProductCubit, ProductState>(
  builder: (context, state) {
    if (state is ProductLoading) {
      return CircularProgressIndicator();
    }

    if (state is ProductLoaded) {
      return ListView.builder(
        itemCount: state.products.length,
        itemBuilder: (_, index) {
          final product = state.products[index];

          return ListTile(
            title: Text(product.name),
            subtitle: Text(product.price.toString()),
          );
        },
      );
    }

    return Text("No Data");
  },
)
```

---

# 11) Trigger Loading

```dart
context.read<ProductCubit>().loadProducts();
```

Usually inside:

```dart
initState()
```

---

# 12) Cart Cubit Example

---

## Cart State

```dart
class CartState {
  final List<Product> items;

  CartState(this.items);
}
```

---

## Cart Cubit

```dart
class CartCubit extends Cubit<CartState> {
  CartCubit() : super(CartState([]));

  void addToCart(Product product) {
    final updated = List<Product>.from(state.items)..add(product);
    emit(CartState(updated));
  }

  void removeFromCart(Product product) {
    final updated = List<Product>.from(state.items)..remove(product);
    emit(CartState(updated));
  }
}
```

---

---

# 13) Cart UI

```dart
context.read<CartCubit>().addToCart(product);
```

---

# 14) API Integration Using Dio

Dio

```dart
import 'package:dio/dio.dart';

class ProductRepository {
  final Dio dio;

  ProductRepository(this.dio);

  Future<List<Product>> getProducts() async {
    final response = await dio.get('/products');

    return (response.data as List)
        .map((e) => Product(
              id: e['id'],
              name: e['name'],
              price: e['price'],
            ))
        .toList();
  }
}
```

---

# 15) Cubit with Repository

```dart
class ProductCubit extends Cubit<ProductState> {
  final ProductRepository repository;

  ProductCubit(this.repository) : super(ProductInitial());

  Future<void> loadProducts() async {
    emit(ProductLoading());

    try {
      final products = await repository.getProducts();
      emit(ProductLoaded(products));
    } catch (e) {
      emit(ProductError(e.toString()));
    }
  }
}
```

---

# 16) Dependency Injection

GetIt

```dart
final sl = GetIt.instance;

void setup() {
  sl.registerLazySingleton(() => Dio());

  sl.registerLazySingleton(
    () => ProductRepository(sl()),
  );

  sl.registerFactory(
    () => ProductCubit(sl()),
  );
}
```

---

# 17) MultiBlocProvider for Real E-commerce

```dart
MultiBlocProvider(
  providers: [
    BlocProvider(create: (_) => sl<ProductCubit>()),
    BlocProvider(create: (_) => CartCubit()),
  ],
  child: MyApp(),
)
```

---

# 18) Authentication Cubit Example

```dart
class AuthCubit extends Cubit<AuthState> {
  AuthCubit() : super(AuthInitial());

  Future<void> login(String email, String password) async {
    emit(AuthLoading());

    await Future.delayed(Duration(seconds: 2));

    emit(AuthSuccess());
  }
}
```

---

---

# 19) Senior Best Practices

✅ never put API directly inside UI
✅ use repository layer
✅ one cubit per feature
✅ use equatable
✅ avoid giant cubits
✅ separate state files
✅ feature-first folders

---

# 20) Real Senior Production Structure

```plaintext
products/
│
├── data/
│   ├── models/
│   ├── repository/
│
├── domain/
│
├── presentation/
│   ├── cubit/
│   │   ├── product_cubit.dart
│   │   ├── product_state.dart
│   │
│   ├── pages/
│   ├── widgets/
```

---

# 21) When Cubit Becomes Too Large

Split:

```plaintext
ProductCubit
CartCubit
WishlistCubit
CheckoutCubit
AuthCubit
```

Never one giant cubit 🚫

---

# 22) Senior Advice for E-commerce

For real projects:

Use:

✅ ProductCubit
✅ CategoryCubit
✅ CartCubit
✅ WishlistCubit
✅ CheckoutCubit
✅ AuthCubit
✅ OrderCubit

---

# 23) Recommended Next Step

If you want, I can also write:

## **Full enterprise Flutter Bloc e-commerce project structure (2026 senior style)**

with:

✅ pagination
✅ search
✅ filters
✅ websocket updates
✅ token refresh
✅ offline cache
✅ clean architecture
✅ production-ready folder structure

That version is what senior teams actually ship 🚀
