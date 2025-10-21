---
tags:
  - saca1
---

# Fixed Point Binary Operations

## Fixed Point Format

```
Binary point fixed at position p
Bits left of point: 2^i (i = 0, 1, 2, ...)
Bits right of point: 2^-i (i = 1, 2, 3, ...)
```

## Representation

```
b_n b_(n-1) ... b_1 b_0 . b_(-1) b_(-2) ... b_(-m)

Value = Σ(i=0 to n) b_i × 2^i + Σ(i=1 to m) b_(-i) × 2^(-i)
```

## Addition/Subtraction

### Same Binary Point Position

```
Result = A ± B (direct bit-by-bit operation)
Carry/borrow propagates normally across binary point
```

### Different Binary Point Positions

```
1. Align binary points by padding zeros
2. Perform operation
3. Result point position = rightmost point position
```

## Examples

### 8-bit with point after 4th bit

```
1011.0110 = 1×2³ + 0×2² + 1×2¹ + 1×2⁰ + 0×2⁻¹ + 1×2⁻² + 1×2⁻³ + 0×2⁻⁴
         = 8 + 0 + 2 + 1 + 0 + 0.25 + 0.125 + 0
         = 11.375
```

### Addition Example

```
  1011.0110  (11.375)
+ 0001.1000  (1.5)
-----------
  1100.1110  (12.875)
```

### Subtraction Example

```
  1011.0110  (11.375)
- 0001.1000  (1.5)
-----------
  1001.0110  (9.875)
```

## Overflow Handling

```
Integer overflow: Leftmost bits exceed available positions
Precision loss: Rightmost bits exceed fractional positions
```