Using **selectors with Bloc in Flutter** is mainly about **performance optimization** ⚡🧩📱

Instead of rebuilding an entire widget whenever *any part* of state changes, selectors rebuild **only when the selected value changes**.

---

# 1) Why Selectors Exist

Without selector:

```dart id="6zv8g8"
BlocBuilder<CartCubit, CartState>(
  builder: (context, state) {
    return Text(state.totalPrice.toString());
  },
)
```

Problem:

If **cart items**, **coupon**, **shipping**, or **note** changes → widget rebuilds every time.

Even if `totalPrice` didn't change.

That creates unnecessary rebuilds.

---

# 2) Use BlocSelector

Use:

```dart id="byj43w"
BlocSelector<CartCubit, CartState, double>(
  selector: (state) => state.totalPrice,
  builder: (context, totalPrice) {
    return Text(totalPrice.toString());
  },
)
```

Now rebuild happens **only if totalPrice changes** ✅

---

# 3) When to Use Selector

Use selector when state contains many fields:

```dart id="4slx2g"
class CartState {
  final List<Product> items;
  final double totalPrice;
  final String coupon;
  final bool loading;
}
```

And widget needs only:

```plaintext id="nnh7vq"
totalPrice
```

---

# 4) Real E-commerce Example

## Cart Badge Count

Top app bar badge only needs item count 🛒

---

## Bad Practice

```dart id="ch7bd6"
BlocBuilder<CartCubit, CartState>(
  builder: (_, state) {
    return Text(state.items.length.toString());
  },
)
```

This rebuilds on every cart state update.

---

## Correct

```dart id="sqgwtx"
BlocSelector<CartCubit, CartState, int>(
  selector: (state) => state.items.length,
  builder: (_, count) {
    return Text(count.toString());
  },
)
```

Only rebuild when count changes.

---

# 5) Product Favorite Example

Only favorite icon changes ❤️

```dart id="ft1h18"
BlocSelector<ProductCubit, ProductState, bool>(
  selector: (state) => state.isFavorite,
  builder: (_, isFavorite) {
    return Icon(
      isFavorite ? Icons.favorite : Icons.favorite_border,
    );
  },
)
```

---

# 6) Huge Benefit in Large Screens

Suppose product page contains:

✅ image gallery
✅ title
✅ price
✅ stock
✅ reviews
✅ favorite icon

If favorite changes, you do NOT want all widgets rebuilding.

Selector isolates rebuild.

---

# 7) Senior Rule

Use selector when:

✅ state object is large
✅ widget needs one field
✅ frequent updates happen
✅ performance matters

---

# 8) When BlocBuilder Is Enough

Use normal `BlocBuilder` when:

✅ screen small
✅ state simple
✅ one widget depends on most state

Example:

Login page:

```dart id="o4z0n7"
loading
success
error
```

No selector needed.

---

# 9) Selector + Equatable Important Rule

State should use Equatable

Because selector compares values.

Example:

```dart id="idiy17"
class CartState extends Equatable {
  final double totalPrice;

  @override
  List<Object?> get props => [totalPrice];
}
```

Without proper equality:

Selector may rebuild unexpectedly.

---

# 10) Senior Production Pattern

Large widget tree:

```plaintext id="bl7gjv"
Screen
 ├── price selector
 ├── badge selector
 ├── stock selector
 ├── favorite selector
```

Each widget isolated.

This is how senior teams reduce rebuild cost 🚀

---

# 11) Selector vs buildWhen

## buildWhen

Used to decide if whole builder rebuilds:

```dart id="7n7l4z"
BlocBuilder<CartCubit, CartState>(
  buildWhen: (previous, current) =>
      previous.totalPrice != current.totalPrice,
  builder: (_, state) {
    return Text(state.totalPrice.toString());
  },
)
```

---

## Selector

Cleaner:

```dart id="zh0mho"
BlocSelector<CartCubit, CartState, double>(
  selector: (state) => state.totalPrice,
  builder: (_, total) {
    return Text(total.toString());
  },
)
```

Senior teams prefer selector because:

✅ simpler
✅ clearer
✅ less manual comparisons

---

# 12) Real Rule Used by Senior Flutter Teams

## Use BlocSelector for:

* badge counters
* favorite flags
* stock numbers
* prices
* loading flags
* single item widgets

## Use BlocBuilder for:

* full screen rebuild
* large state transitions

---

# 13) Best Practice for Large E-commerce Apps

Split UI into very small widgets + selector:

```plaintext id="j7rbif"
ProductPriceWidget
CartBadgeWidget
FavoriteButtonWidget
StockWidget
```

Each widget watches only needed state.

---

# 14) Golden Rule

If widget needs only one field:

➡️ use selector

If widget needs whole state:

➡️ use builder

---

If you want, I can also write **advanced senior Bloc patterns used in large Flutter apps**, including:

✅ MultiBlocSelector
✅ Nested selectors
✅ selector pitfalls
✅ rebuild debugging
✅ production architecture for 100+ screens 🚀
