Sure! Here’s a clear explanation of **VLOOKUP**, **HLOOKUP**, and **XLOOKUP**, including their syntax, when to use them, and their conditions/limitations.

---

# 1. VLOOKUP

## Meaning
**VLOOKUP** stands for **Vertical Lookup**.

It searches for a value in the **first column** of a table and returns a value from another column in the same row.

## Syntax

```excel
=VLOOKUP(lookup_value, table_array, col_index_num, [range_lookup])
```

## Arguments

| Argument | Meaning |
|---|---|
| `lookup_value` | The value you want to search for |
| `table_array` | The table or range where the lookup happens |
| `col_index_num` | The column number in the table from which to return a value |
| `range_lookup` | Optional. `FALSE` for exact match, `TRUE` for approximate match |

---

## Example

Suppose you have this table:

| A | B |
|---|---|
| ID | Name |
| 101 | John |
| 102 | Mary |
| 103 | Peter |

To find the name for ID `102`:

```excel
=VLOOKUP(102, A2:B4, 2, FALSE)
```

### Result:
```text
Mary
```

---

## Conditions for Using VLOOKUP

Use **VLOOKUP** when:

1. Your data is arranged **vertically**.
2. The lookup value is in the **first column** of the selected range.
3. You want to return a value from a column to the **right** of the lookup column.
4. You do not need to search to the left of the lookup column.

---

## Important Limitations of VLOOKUP

1. It can only search in the **first column** of the table.
2. It cannot return values from columns to the **left** of the lookup column.
3. If you insert or delete columns, the `col_index_num` may become incorrect.
4. It can be slower on very large datasets compared to more modern functions.
5. If `range_lookup` is omitted, Excel may assume `TRUE`, which can cause unexpected results.

---

## Exact Match vs Approximate Match

### Exact Match

```excel
=VLOOKUP(102, A2:B4, 2, FALSE)
```

Use this when you want the lookup value to match exactly.

Common uses:

- Employee ID lookup
- Product code lookup
- Invoice number lookup

---

### Approximate Match

```excel
=VLOOKUP(75, A2:B10, 2, TRUE)
```

Use this when you want the closest match.

Common uses:

- Grade calculation
- Tax brackets
- Commission rates

For approximate match, the first column must be sorted in **ascending order**.

---

# 2. HLOOKUP

## Meaning
**HLOOKUP** stands for **Horizontal Lookup**.

It searches for a value in the **first row** of a table and returns a value from another row in the same column.

## Syntax

```excel
=HLOOKUP(lookup_value, table_array, row_index_num, [range_lookup])
```

## Arguments

| Argument | Meaning |
|---|---|
| `lookup_value` | The value you want to search for |
| `table_array` | The table or range where the lookup happens |
| `row_index_num` | The row number in the table from which to return a value |
| `range_lookup` | Optional. `FALSE` for exact match, `TRUE` for approximate match |

---

## Example

Suppose your data is arranged horizontally:

|   | A | B | C | D |
|---|---|---|---|---|
| 1 | Month | Jan | Feb | Mar |
| 2 | Sales | 500 | 700 | 600 |

To find sales for `Feb`:

```excel
=HLOOKUP("Feb", B1:D2, 2, FALSE)
```

### Result:
```text
700
```

---

## Conditions for Using HLOOKUP

Use **HLOOKUP** when:

1. Your data is arranged **horizontally**.
2. The lookup value is in the **first row** of the selected range.
3. You want to return a value from a row below the lookup row.
4. You do not need to search above the lookup row.

---

## Important Limitations of HLOOKUP

1. It can only search in the **first row** of the table.
2. It cannot return values from rows above the lookup row.
3. It is less common because most data is organized vertically.
4. If rows are inserted or deleted, the `row_index_num` may become incorrect.

---

# 3. XLOOKUP

## Meaning
**XLOOKUP** is a newer and more flexible lookup function in Excel.

It can search vertically or horizontally, search from left to right or right to left, and return exact matches by default.

## Syntax

```excel
=XLOOKUP(lookup_value, lookup_array, return_array, [if_not_found], [match_mode], [search_mode])
```

## Arguments

| Argument | Meaning |
|---|---|
| `lookup_value` | The value you want to search for |
| `lookup_array` | The column or row where Excel searches |
| `return_array` | The column or row from which Excel returns the result |
| `if_not_found` | Optional. Value to return if no match is found |
| `match_mode` | Optional. Controls exact or approximate match |
| `search_mode` | Optional. Controls search direction |

---

## Example

Suppose you have this table:

| A | B |
|---|---|
| ID | Name |
| 101 | John |
| 102 | Mary |
| 103 | Peter |

To find the name for ID `102`:

```excel
=XLOOKUP(102, A2:A4, B2:B4)
```

### Result:
```text
Mary
```

---

## XLOOKUP Can Search to the Left

This is one of the biggest advantages over VLOOKUP.

Suppose you have:

| A | B |
|---|---|
| Name | ID |
| John | 101 |
| Mary | 102 |
| Peter | 103 |

To find the ID for `"Mary"`:

```excel
=XLOOKUP("Mary", A2:A4, B2:B4)
```

### Result:
```text
102
```

VLOOKUP cannot do this directly because the lookup value must be in the first column.

---

## Conditions for Using XLOOKUP

Use **XLOOKUP** when:

1. You have a newer version of Excel that supports XLOOKUP.
2. You want a more flexible lookup.
3. You need to search left, right, up, or down.
4. You want exact match by default.
5. You want to avoid counting column numbers like in VLOOKUP.
6. You want a custom message if no match is found.

---

## XLOOKUP with No Match Found

With VLOOKUP, if no match is found, you usually get `#N/A`.

Example:

```excel
=VLOOKUP(105, A2:B4, 2, FALSE)
```

If `105` does not exist, the result may be:

```text
#N/A
```

With XLOOKUP, you can provide a custom message:

```excel
=XLOOKUP(105, A2:A4, B2:B4, "Not Found")
```

### Result:
```text
Not Found
```

---

## XLOOKUP Match Mode

```excel
=XLOOKUP(lookup_value, lookup_array, return_array, [if_not_found], [match_mode])
```

Common match modes:

| Match Mode | Meaning |
|---|---|
| `0` | Exact match. This is the default |
| `-1` | Exact match. If not found, return next smaller item |
| `1` | Exact match. If not found, return next larger item |
| `2` | Wildcard match |

Example:

```excel
=XLOOKUP(102, A2:A4, B2:B4, "Not Found", 0)
```

This searches for an exact match.

---

# Comparison of VLOOKUP, HLOOKUP, and XLOOKUP

| Feature | VLOOKUP | HLOOKUP | XLOOKUP |
|---|---|---|---|
| Lookup direction | Vertical | Horizontal | Vertical or horizontal |
| Search location | First column | First row | Any column or row |
| Can search left? | No | No | Yes |
| Can search right? | Yes | Yes | Yes |
| Can search upward? | No | No | Yes |
| Can search downward? | Yes | Yes | Yes |
| Exact match by default? | No | No | Yes |
| Requires column/row number? | Yes | Yes | No |
| Custom “not found” message? | No | No | Yes |
| Easier to use? | Moderate | Moderate | Easiest |
| Best for modern Excel? | Still useful | Limited | Best option |

---

# When to Use Each Formula

## Use VLOOKUP when:

- Your data is arranged vertically.
- You are using an older Excel version that does not support XLOOKUP.
- The lookup value is in the first column.
- You only need to return values from the right side.

Example:

```excel
=VLOOKUP(102, A2:B4, 2, FALSE)
```

---

## Use HLOOKUP when:

- Your data is arranged horizontally.
- The lookup value is in the first row.
- You only need to return values from below.
- You are using an older Excel version that does not support XLOOKUP.

Example:

```excel
=HLOOKUP("Feb", B1:D2, 2, FALSE)
```

---

## Use XLOOKUP when:

- You have Excel 365, Excel 2021, or later.
- You want a simpler and more powerful lookup.
- You need to search left or right.
- You want exact match by default.
- You want to handle errors with a custom message.
- You do not want to count column numbers.

Example:

```excel
=XLOOKUP(102, A2:A4, B2:B4, "Not Found")
```

---

# Simple Real-Life Example

Suppose you have this product table:

| A | B | C |
|---|---|---|
| Product ID | Product Name | Price |
| P001 | Laptop | 800 |
| P002 | Mouse | 25 |
| P003 | Keyboard | 50 |

---

## VLOOKUP Example

Find the product name for `P002`:

```excel
=VLOOKUP("P002", A2:C4, 2, FALSE)
```

Result:

```text
Mouse
```

Find the price for `P003`:

```excel
=VLOOKUP("P003", A2:C4, 3, FALSE)
```

Result:

```text
50
```

---

## HLOOKUP Example

If the same data is horizontal:

|   | A | B | C | D |
|---|---|---|---|---|
| 1 | Product ID | P001 | P002 | P003 |
| 2 | Product Name | Laptop | Mouse | Keyboard |
| 3 | Price | 800 | 25 | 50 |

Find the product name for `P002`:

```excel
=HLOOKUP("P002", B1:D3, 2, FALSE)
```

Result:

```text
Mouse
```

Find the price for `P003`:

```excel
=HLOOKUP("P003", B1:D3, 3, FALSE)
```

Result:

```text
50
```

---

## XLOOKUP Example

Find the product name for `P002`:

```excel
=XLOOKUP("P002", A2:A4, B2:B4)
```

Result:

```text
Mouse
```

Find the price for `P003`:

```excel
=XLOOKUP("P003", A2:A4, C2:C4)
```

Result:

```text
50
```

---

# Summary

| Formula | Best Used For |
|---|---|
| `VLOOKUP` | Vertical lookup where the lookup value is in the first column |
| `HLOOKUP` | Horizontal lookup where the lookup value is in the first row |
| `XLOOKUP` | Modern, flexible lookup that can search in any direction |

If you are using a modern version of Excel, **XLOOKUP is usually the best choice** because it is more flexible, easier to use, and avoids many of the limitations of VLOOKUP and HLOOKUP.