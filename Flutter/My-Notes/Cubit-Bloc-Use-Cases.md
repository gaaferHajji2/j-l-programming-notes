Below is a **practical guide of real use cases for using Cubit vs Bloc in Flutter**, with **production-style examples**, so you can decide **when senior teams choose Bloc Cubit and when they choose full Bloc** 🧩🚀📱

---

# 1) When to Use Cubit vs Bloc

## Use Cubit when:

✅ action is simple
✅ one direct state change
✅ no event queue needed
✅ UI triggers straightforward logic

Examples:

* login
* load products
* add to cart
* toggle favorite
* change theme

---

## Use Bloc when:

✅ many events affect same feature
✅ complex workflow
✅ ordering matters
✅ side effects must be controlled

Examples:

* checkout pipeline
* search with debounce
* websocket streams
* pagination with filters
* payment flow

---

# 2) Cubit Use Case: Product Loading

This is the most common e-commerce Cubit example 📦

---

## State

```dart
abstract class ProductState {}

class ProductInitial extends ProductState {}

class ProductLoading extends ProductState {}

class ProductLoaded extends ProductState {
  final List<String> products;

  ProductLoaded(this.products);
}

class ProductError extends ProductState {
  final String message;

  ProductError(this.message);
}
```

---

## Cubit

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class ProductCubit extends Cubit<ProductState> {
  ProductCubit() : super(ProductInitial());

  Future<void> loadProducts() async {
    emit(ProductLoading());

    try {
      await Future.delayed(Duration(seconds: 2));

      emit(
        ProductLoaded([
          'Laptop',
          'Phone',
          'Tablet',
        ]),
      );
    } catch (e) {
      emit(ProductError('Failed to load'));
    }
  }
}
```

---

## UI

```dart
BlocBuilder<ProductCubit, ProductState>(
  builder: (context, state) {
    if (state is ProductLoading) {
      return CircularProgressIndicator();
    }

    if (state is ProductLoaded) {
      return ListView(
        children: state.products
            .map((p) => Text(p))
            .toList(),
      );
    }

    return Text('Empty');
  },
)
```

---

## Why Cubit here?

Because:

```plaintext
One action → one result
```

Very simple.

---

# 3) Cubit Use Case: Cart Management

Senior teams almost always use Cubit here 🛒

---

## State

```dart
class CartState {
  final List<String> items;

  CartState(this.items);
}
```

---

## Cubit

```dart
class CartCubit extends Cubit<CartState> {
  CartCubit() : super(CartState([]));

  void addItem(String item) {
    final updated = List<String>.from(state.items)..add(item);
    emit(CartState(updated));
  }

  void removeItem(String item) {
    final updated = List<String>.from(state.items)..remove(item);
    emit(CartState(updated));
  }
}
```

---

## UI Action

```dart
context.read<CartCubit>().addItem('Laptop');
```

---

## Why Cubit?

Because no event complexity exists.

---

# 4) Cubit Use Case: Authentication

Very common login flow 🔐

---

## Cubit

```dart
class AuthCubit extends Cubit<String> {
  AuthCubit() : super('initial');

  Future<void> login() async {
    emit('loading');

    await Future.delayed(Duration(seconds: 2));

    emit('success');
  }
}
```

---

## UI

```dart
BlocBuilder<AuthCubit, String>(
  builder: (_, state) {
    if (state == 'loading') {
      return CircularProgressIndicator();
    }

    return Text(state);
  },
)
```

---

---

# 5) Bloc Use Case: Checkout Process

Now we need full Bloc because checkout has many events 💳

---

## Events

```dart
abstract class CheckoutEvent {}

class StartCheckout extends CheckoutEvent {}

class ApplyCoupon extends CheckoutEvent {
  final String code;

  ApplyCoupon(this.code);
}

class ConfirmPayment extends CheckoutEvent {}
```

---

## States

```dart
abstract class CheckoutState {}

class CheckoutInitial extends CheckoutState {}

class CheckoutLoading extends CheckoutState {}

class CheckoutSuccess extends CheckoutState {}

class CheckoutFailure extends CheckoutState {}
```

---

## Bloc

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CheckoutBloc extends Bloc<CheckoutEvent, CheckoutState> {
  CheckoutBloc() : super(CheckoutInitial()) {
    on<StartCheckout>((event, emit) async {
      emit(CheckoutLoading());

      await Future.delayed(Duration(seconds: 2));

      emit(CheckoutSuccess());
    });

    on<ApplyCoupon>((event, emit) async {
      emit(CheckoutLoading());

      await Future.delayed(Duration(seconds: 1));

      emit(CheckoutSuccess());
    });

    on<ConfirmPayment>((event, emit) async {
      emit(CheckoutLoading());

      await Future.delayed(Duration(seconds: 2));

      emit(CheckoutSuccess());
    });
  }
}
```

---

## UI

```dart
context.read<CheckoutBloc>().add(StartCheckout());
```

---

## Why Bloc?

Because:

Multiple events control same state machine.

---

# 6) Bloc Use Case: Search with Debounce

Search is usually Bloc 🔍

Because many inputs arrive fast.

---

## Event

```dart
class SearchProduct extends SearchEvent {
  final String query;

  SearchProduct(this.query);
}
```

---

## Bloc Logic

```dart
on<SearchProduct>((event, emit) async {
  emit(SearchLoading());

  await Future.delayed(Duration(milliseconds: 500));

  emit(SearchLoaded());
});
```

---

## Why Bloc?

Because typing produces event streams:

```plaintext
a
ap
app
apple
```

Bloc handles this better.

---

# 7) Bloc Use Case: Infinite Pagination

For APIs with pages 📄

---

## Events

```dart
LoadFirstPage
LoadNextPage
RefreshPage
```

---

## Why Bloc?

Because pagination state becomes complex:

* current page
* loading next page
* reached end
* refresh

Cubit becomes messy quickly.

---

# 8) Cubit Use Case: Theme Switcher

Perfect Cubit example 🌙☀️

---

```dart
class ThemeCubit extends Cubit<bool> {
  ThemeCubit() : super(false);

  void toggleTheme() {
    emit(!state);
  }
}
```

---

---

# 9) Cubit Use Case: Wishlist

Simple toggle ❤️

---

```dart
class WishlistCubit extends Cubit<List<int>> {
  WishlistCubit() : super([]);

  void toggle(int productId) {
    final updated = List<int>.from(state);

    if (updated.contains(productId)) {
      updated.remove(productId);
    } else {
      updated.add(productId);
    }

    emit(updated);
  }
}
```

---

---

# 10) Senior Rule Used in Real Projects

## Start with Cubit

If feature grows:

Move to Bloc.

Because Cubit is lighter.

---

# 11) Real E-commerce Mapping

| Feature    | Best Choice |
| ---------- | ----------- |
| Products   | Cubit       |
| Cart       | Cubit       |
| Wishlist   | Cubit       |
| Auth       | Cubit       |
| Checkout   | Bloc        |
| Search     | Bloc        |
| Pagination | Bloc        |
| Payment    | Bloc        |

---

# 12) Senior Folder Structure

```plaintext
features/
│
├── products/
│   ├── cubit/
│
├── cart/
│   ├── cubit/
│
├── checkout/
│   ├── bloc/
```

---

# 13) Senior Recommendation

For most Flutter apps:

✅ 70% Cubit
✅ 30% Bloc

Because Cubit is cleaner unless event orchestration is needed.

---

# 14) Practical Production Advice

Use:

✅ `BlocBuilder` → rebuild UI
✅ `BlocListener` → navigation/snackbar
✅ `BlocConsumer` → both together

---

## Example Listener

```dart
BlocListener<AuthCubit, AuthState>(
  listener: (context, state) {
    if (state is AuthSuccess) {
      Navigator.pushNamed(context, '/home');
    }
  },
  child: LoginPage(),
)
```

---

If you want, next I can write **a full senior Flutter Bloc architecture for large apps (2026 production style)** including:

✅ API layer
✅ repository layer
✅ websocket updates
✅ pagination
✅ token refresh
✅ offline caching
✅ clean architecture

That is the exact structure senior teams use in large Flutter systems 🚀
