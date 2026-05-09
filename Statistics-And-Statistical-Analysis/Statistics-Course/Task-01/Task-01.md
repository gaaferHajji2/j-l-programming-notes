## 1. Descriptive Statistics Summary

**Data:** Heights of 10 basketball players (in meters)
- Values: 1.62, 1.72, 1.55, 1.70, 1.78, 1.65, 1.64, 1.64, 1.66, 1.74
- Sorted: 1.55, 1.62, 1.64, 1.64, 1.65, 1.66, 1.70, 1.72, 1.74, 1.78

### Measures of Central Tendency:

**Mean (Average):** 1.67 m
- **Rule:** $\bar{x} = \frac{\sum_{i=1}^{n} x_i}{n}$
- Calculation: (1.62 + 1.72 + 1.55 + 1.70 + 1.78 + 1.65 + 1.64 + 1.64 + 1.66 + 1.74) / 10 = 16.70 / 10 = 1.67

**Median:** 1.655 m
- **Rule:** Middle value when data is sorted. For even n: $\frac{x_{(n/2)} + x_{(n/2+1)}}{2}$
- Calculation: (1.65 + 1.66) / 2 = 1.655

**Mode:** 1.64 m -- Mode (Fashion): 1.64 m
- **Rule:** Most frequently occurring value(s)
- The value 1.64 appears twice (most frequent)

### Measures of Dispersion:

**Range:** 0.23 m
- **Rule:** $Range = Max - Min$
- Calculation: 1.78 - 1.55 = 0.23

**Variance:** 0.0049 m²
- **Rule:** $s^2 = \frac{\sum_{i=1}^{n} (x_i - \bar{x})^2}{n-1}$ (sample variance)
- Calculation: Sum of squared deviations / 9 = 0.0441 / 9 = 0.0049

**Standard Deviation:** 0.07 m
- **Rule:** $s = \sqrt{s^2}$
- Calculation: √0.0049 = 0.07

**Minimum:** 1.55 m
**Maximum:** 1.78 m

**Quartiles:**
- **Q1 (First Quartile):** 1.64 m - 25th percentile
- **Q3 (Third Quartile):** 1.72 m - 75th percentile
- **IQR (Interquartile Range):** 0.08 m
  - **Rule:** $IQR = Q3 - Q1$

**Coefficient of Variation:** 4.19%
- **Rule:** $CV = \frac{s}{\bar{x}} \times 100\%$
- Calculation: (0.07 / 1.67) × 100 = 4.19%

---

## 2. Python Code with Measurement Rules

```python
import numpy as np
import pandas as pd
from scipy import stats

# Data
heights = np.array([1.62, 1.72, 1.55, 1.70, 1.78, 1.65, 1.64, 1.64, 1.66, 1.74])
n = len(heights)

print("=" * 60)
print("DESCRIPTIVE STATISTICS FOR BASKETBALL PLAYERS' HEIGHTS")
print("=" * 60)
print(f"\nData (n={n}): {heights}")
print(f"Sorted Data: {np.sort(heights)}")

print("\n" + "=" * 60)
print("MEASURES OF CENTRAL TENDENCY")
print("=" * 60)

# Mean
mean = np.mean(heights)
print(f"\n1. MEAN (Average)")
print(f"   Rule: x̄ = Σxᵢ / n")
print(f"   Calculation: {np.sum(heights):.2f} / {n} = {mean:.4f} m")

# Median
median = np.median(heights)
print(f"\n2. MEDIAN")
print(f"   Rule: Middle value (or average of two middle values for even n)")
print(f"   For n={n}: Average of positions {n//2} and {n//2 + 1}")
sorted_heights = np.sort(heights)
print(f"   Calculation: ({sorted_heights[n//2-1]:.2f} + {sorted_heights[n//2]:.2f}) / 2 = {median:.4f} m")

# Mode
mode_result = stats.mode(heights, keepdims=True)
mode = mode_result.mode[0]
mode_count = mode_result.count[0]
print(f"\n3. MODE")
print(f"   Rule: Most frequently occurring value(s)")
print(f"   Value: {mode:.2f} m (appears {mode_count} times)")

print("\n" + "=" * 60)
print("MEASURES OF DISPERSION")
print("=" * 60)

# Range
min_val = np.min(heights)
max_val = np.max(heights)
range_val = max_val - min_val
print(f"\n4. RANGE")
print(f"   Rule: Range = Max - Min")
print(f"   Calculation: {max_val:.2f} - {min_val:.2f} = {range_val:.4f} m")

# Variance (Sample)
variance = np.var(heights, ddof=1)  # ddof=1 for sample variance
print(f"\n5. VARIANCE (Sample)")
print(f"   Rule: s² = Σ(xᵢ - x̄)² / (n-1)")
squared_deviations = (heights - mean) ** 2
print(f"   Sum of squared deviations: {np.sum(squared_deviations):.4f}")
print(f"   Calculation: {np.sum(squared_deviations):.4f} / {n-1} = {variance:.6f} m²")

# Standard Deviation
std_dev = np.std(heights, ddof=1)
print(f"\n6. STANDARD DEVIATION (Sample)")
print(f"   Rule: s = √s²")
print(f"   Calculation: √{variance:.6f} = {std_dev:.4f} m")

# Minimum and Maximum
print(f"\n7. MINIMUM: {min_val:.2f} m")
print(f"8. MAXIMUM: {max_val:.2f} m")

# Quartiles
q1 = np.percentile(heights, 25)
q3 = np.percentile(heights, 75)
iqr = q3 - q1
print(f"\n9. QUARTILES")
print(f"   Q1 (25th percentile): {q1:.4f} m")
print(f"   Q3 (75th percentile): {q3:.4f} m")
print(f"   IQR (Interquartile Range) = Q3 - Q1 = {iqr:.4f} m")

# Coefficient of Variation
cv = (std_dev / mean) * 100
print(f"\n10. COEFFICIENT OF VARIATION")
print(f"    Rule: CV = (s / x̄) × 100%")
print(f"    Calculation: ({std_dev:.4f} / {mean:.4f}) × 100 = {cv:.2f}%")

print("\n" + "=" * 60)
print("SUMMARY TABLE")
print("=" * 60)
summary_data = {
    'Measure': ['Mean', 'Median', 'Mode', 'Std Dev', 'Variance', 
                'Range', 'Min', 'Max', 'Q1', 'Q3', 'IQR', 'CV'],
    'Value': [f"{mean:.4f}", f"{median:.4f}", f"{mode:.2f}", f"{std_dev:.4f}", 
              f"{variance:.6f}", f"{range_val:.4f}", f"{min_val:.2f}", 
              f"{max_val:.2f}", f"{q1:.4f}", f"{q3:.4f}", f"{iqr:.4f}", f"{cv:.2f}%"],
    'Unit': ['m', 'm', 'm', 'm', 'm²', 'm', 'm', 'm', 'm', 'm', 'm', '%']
}

summary_df = pd.DataFrame(summary_data)
print(summary_df.to_string(index=False))

print("\n" + "=" * 60)
print("INTERPRETATION")
print("=" * 60)
print(f"• Average height: {mean:.2f} m")
print(f"• Height variation: ±{std_dev:.2f} m from the mean")
print(f"• Middle 50% of players have heights between {q1:.2f} m and {q3:.2f} m")
print(f"• Relative variability: {cv:.2f}% (low variability)")
print("=" * 60)
```

**Output when you run this code:**

The code will display all descriptive statistics with their formulas and calculations, providing a comprehensive analysis of the basketball players' heights data.