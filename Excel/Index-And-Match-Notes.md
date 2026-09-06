Absolutely! **INDEX** and **MATCH** are Excel lookup functions that are often used together as a powerful alternative to **VLOOKUP**.

They allow you to find a value in one column/row and return a related value from another column/row.

---

# 1. INDEX Formula

## Meaning
The **INDEX** function returns a value from a specific position in a range.

In simple words:

> INDEX gives you the value at a given row and/or column number.

---

## Syntax

### Basic Syntax

```excel
=INDEX(array, row_num, [column_num])
```

| Argument | Meaning |
|---|---|
| `array` | The range of cells from which you want to return a value |
| `row_num` | The row number in the range |
| `column_num` | Optional. The column number in the range |

If the range has only one column, you only need the row number.

---

## Example 1: INDEX with One Column

Suppose you have:

| A |
|---|
| Apple |
| Banana |
| Mango |

To get the 2nd item:

```excel
=INDEX(A1:A3, 2)
```

### Result:

```text
Banana
```

Because `Banana` is in row 2 of the selected range.

---

## Example 2: INDEX with Rows and Columns

Suppose you have:

|   | A | B | C |
|---|---|---|---|
| 1 | Product | Price | Stock |
| 2 | Laptop | 800 | 10 |
| 3 | Mouse | 25 | 50 |
| 4 | Keyboard | 50 | 30 |

To get the value in row 2, column 2 of the range `A1:C4`:

```excel
=INDEX(A1:C4, 2, 2)
```

### Result:

```text
800
```

Because row 2, column 2 contains `800`.

---

# 2. MATCH Formula

## Meaning
The **MATCH** function searches for a value in a range and returns its position number.

In simple words:

> MATCH tells you where a value is located.

---

## Syntax

```excel
=MATCH(lookup_value, lookup_array, [match_type])
```

| Argument | Meaning |
|---|---|
| `lookup_value` | The value you want to find |
| `lookup_array` | The range where Excel searches |
| `match_type` | Optional. Defines the type of match |

---

## Match Types

| Match Type | Meaning |
|---|---|
| `0` | Exact match |
| `1` | Less than. The lookup array must be sorted ascending |
| `-1` | Greater than. The lookup array must be sorted descending |

For most lookup formulas, use:

```excel
0
```

This means **exact match**.

---

## Example 1: MATCH with One Column

Suppose you have:

| A |
|---|
| Apple |
| Banana |
| Mango |

To find the position of `"Banana"`:

```excel
=MATCH("Banana", A1:A3, 0)
```

### Result:

```text
2
```

Because `"Banana"` is in the 2nd position.

---

## Example 2: MATCH with Numbers

Suppose you have:

| A |
|---|
| 101 |
| 102 |
| 103 |

To find the position of `102`:

```excel
=MATCH(102, A1:A3, 0)
```

### Result:

```text
2
```

---

# 3. Using INDEX and MATCH Together

Usually, **INDEX** and **MATCH** are combined.

The idea is:

1. **MATCH** finds the position of the lookup value.
2. **INDEX** returns the value from that position in another column.

---

## Basic INDEX MATCH Syntax

```excel
=INDEX(return_range, MATCH(lookup_value, lookup_range, 0))
```

---

## How It Works

Suppose you have:

| A | B |
|---|---|
| ID | Name |
| 101 | John |
| 102 | Mary |
| 103 | Peter |

You want to find the name for ID `102`.

Use:

```excel
=INDEX(B2:B4, MATCH(102, A2:A4, 0))
```

### Explanation

#### Part 1: MATCH finds the position of `102`

```excel
=MATCH(102, A2:A4, 0)
```

Result:

```text
2
```

Because `102` is in the 2nd position of `A2:A4`.

#### Part 2: INDEX returns the value from row 2 of `B2:B4`

```excel
=INDEX(B2:B4, 2)
```

Result:

```text
Mary
```

---

# 4. INDEX MATCH as an Alternative to VLOOKUP

Suppose you have:

| A | B | C |
|---|---|---|
| Product ID | Product | Price |
| P001 | Laptop | 800 |
| P002 | Mouse | 25 |
| P003 | Keyboard | 50 |

You want to find the price of product `P002`.

## Using VLOOKUP

```excel
=VLOOKUP("P002", A2:C4, 3, FALSE)
```

## Using INDEX MATCH

```excel
=INDEX(C2:C4, MATCH("P002", A2:A4, 0))
```

### Result:

```text
25
```

---

# 5. INDEX MATCH Can Look Left

One major advantage of **INDEX MATCH** over **VLOOKUP** is that it can return values from the **left side** of the lookup column.

---

## Example

Suppose you have:

| A | B |
|---|---|
| Name | ID |
| John | 101 |
| Mary | 102 |
| Peter | 103 |

You want to find the ID for `"Mary"`.

VLOOKUP cannot easily do this because ID is to the left of Name.

But INDEX MATCH can:

```excel
=INDEX(B2:B4, MATCH("Mary", A2:A4, 0))
```

### Result:

```text
102
```

---

# 6. INDEX MATCH with Multiple Conditions

Sometimes you need to match more than one condition.

For example, suppose you have:

| A | B | C |
|---|---|---|
| Region | Product | Sales |
| North | Laptop | 1000 |
| South | Laptop | 800 |
| North | Mouse | 200 |
| South | Mouse | 150 |

You want to find sales for:

- Region = `North`
- Product = `Laptop`

Use this array-style INDEX MATCH:

```excel
=INDEX(C2:C5, MATCH(1, (A2:A5="North") * (B2:B5="Laptop"), 0))
```

In older Excel versions, you may need to confirm this with:

```text
Ctrl + Shift + Enter
```

In Excel 365 or newer versions, it usually works as a normal formula.

### Result:

```text
1000
```

---

## How the Multiple Condition Formula Works

### Condition 1

```excel
A2:A5="North"
```

This checks which rows have `North`.

It creates an array like:

```text
TRUE
FALSE
TRUE
FALSE
```

### Condition 2

```excel
B2:B5="Laptop"
```

This checks which rows have `Laptop`.

It creates an array like:

```text
TRUE
TRUE
FALSE
FALSE
```

### Multiply Them

```excel
(A2:A5="North") * (B2:B5="Laptop")
```

TRUE becomes `1`, FALSE becomes `0`:

```text
1
0
0
0
```

So MATCH looks for `1`, meaning the row where both conditions are true.

---

# 7. Two-Way Lookup with INDEX and MATCH

You can use INDEX and MATCH to look up a value based on both a row and a column.

---

## Example

Suppose you have:

|   | A | B | C | D |
|---|---|---|---|---|
| 1 | Region | Q1 | Q2 | Q3 |
| 2 | North | 500 | 600 | 700 |
| 3 | South | 400 | 450 | 480 |
| 4 | East | 300 | 320 | 340 |

You want to find sales for:

- Region = `South`
- Quarter = `Q2`

Use:

```excel
=INDEX(B2:D4, MATCH("South", A2:A4, 0), MATCH("Q2", B1:D1, 0))
```

### Result:

```text
450
```

---

## Explanation

### First MATCH finds the row

```excel
=MATCH("South", A2:A4, 0)
```

Result:

```text
2
```

Because `South` is the 2nd row in the range `A2:A4`.

### Second MATCH finds the column

```excel
=MATCH("Q2", B1:D1, 0)
```

Result:

```text
2
```

Because `Q2` is the 2nd column in the range `B1:D1`.

### INDEX returns the value

```excel
=INDEX(B2:D4, 2, 2)
```

Result:

```text
450
```

---

# 8. INDEX MATCH with Approximate Match

Most of the time, you use exact match:

```excel
0
```

But you can also use approximate match.

---

## Example: Grade Lookup

Suppose you have:

| A | B |
|---|---|
| Score | Grade |
| 0 | F |
| 60 | D |
| 70 | C |
| 80 | B |
| 90 | A |

You want to find the grade for score `75`.

Use:

```excel
=INDEX(B2:B6, MATCH(75, A2:A6, 1))
```

### Result:

```text
C
```

Because `75` is between `70` and `80`.

---

## Important Condition

For approximate match using `1`, the lookup column must be sorted in **ascending order**.

---

# 9. Common Errors with INDEX and MATCH

## 1. `#N/A`

This means the lookup value was not found.

Example:

```excel
=INDEX(B2:B4, MATCH(105, A2:A4, 0))
```

If `105` does not exist, you get:

```text
#N/A
```

---

## 2. `#REF!`

This can happen if the row or column number returned by MATCH is outside the INDEX range.

---

## 3. Wrong Result Because of Match Type

If you forget to add `0` in MATCH, Excel may use approximate match by default.

Example:

```excel
=MATCH(102, A2:A4)
```

It is safer to write:

```excel
=MATCH(102, A2:A4, 0)
```

---

## 4. Mismatched Range Sizes

The `return_range` and `lookup_range` should have the same number of rows or columns.

Incorrect:

```excel
=INDEX(B2:B10, MATCH("P002", A2:A4, 0))
```

Better:

```excel
=INDEX(B2:B4, MATCH("P002", A2:A4, 0))
```

---

# 10. INDEX MATCH vs VLOOKUP

| Feature | VLOOKUP | INDEX MATCH |
|---|---|---|
| Easy for beginners | Yes | Moderate |
| Can look left | No | Yes |
| Needs column number | Yes | No |
| Works if columns are inserted/deleted | Less reliable | More reliable |
| Better for large data | Okay | Often better |
| Can do two-way lookup | Limited | Yes |
| Modern best option | No | INDEX MATCH or XLOOKUP |

---

# 11. INDEX MATCH vs XLOOKUP

If you have a newer Excel version, **XLOOKUP** is often easier than INDEX MATCH.

## INDEX MATCH

```excel
=INDEX(C2:C4, MATCH("P002", A2:A4, 0))
```

## XLOOKUP

```excel
=XLOOKUP("P002", A2:A4, C2:C4)
```

Both can return the same result, but XLOOKUP is simpler.

However, INDEX MATCH is still very useful because it works in older versions of Excel and is very flexible.

---

# 12. When to Use INDEX and MATCH

Use **INDEX and MATCH** when:

1. You need to look up a value and return a related value.
2. You want to return a value from the **left side** of the lookup column.
3. You do not want to count column numbers like in VLOOKUP.
4. Your data may change because of inserted or deleted columns.
5. You need a two-way lookup.
6. You are using an Excel version that does not have XLOOKUP.

---

# 13. Quick Summary

## INDEX

Returns a value from a position.

```excel
=INDEX(range, row_number)
```

Example:

```excel
=INDEX(A1:A3, 2)
```

Returns the 2nd value in `A1:A3`.

---

## MATCH

Returns the position of a value.

```excel
=MATCH(value, range, 0)
```

Example:

```excel
=MATCH("Banana", A1:A3, 0)
```

Returns the position of `"Banana"`.

---

## INDEX MATCH Together

Looks up a value and returns a related value.

```excel
=INDEX(return_range, MATCH(lookup_value, lookup_range, 0))
```

Example:

```excel
=INDEX(B2:B4, MATCH(102, A2:A4, 0))
```

Returns the name for ID `102`.

---

# Final Takeaway

- Use **INDEX** when you want to get a value from a specific position.
- Use **MATCH** when you want to find the position of a value.
- Use **INDEX + MATCH** together when you want a flexible lookup formula.
- INDEX MATCH is more powerful than VLOOKUP in many cases.
- If you have XLOOKUP available, it is usually simpler, but INDEX MATCH is still extremely useful.