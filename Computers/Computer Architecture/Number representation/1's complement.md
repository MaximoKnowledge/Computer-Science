---
tags:
  - saca1
---

# 🧮 1’s Complement – Summary

## 📌 Purpose

- Early method for representing **signed binary numbers**
    
- Especially used for **negative numbers**
    

---

## ⚙️ How It Works

- **Positive numbers**: normal binary
    
- **Negative numbers**: flip all bits of the positive version
    

### Example (8-bit):

```
+5  = 00000101  
-5  = 11111010   (1's complement of +5)
```

---

## 🔍 How to Interpret a 1’s Complement Number

Given an _n_-bit binary:

- If **MSB (most significant bit) = 0**, it's **positive**, interpret normally.
    
- If **MSB = 1**, it's **negative**, so:
    
    1. **Flip all bits** (1’s complement)
        
    2. Interpret the result as a positive number
        
    3. Add a minus sign
        

### ✅ Examples (4-bit)

- `0111` → MSB = 0 → **+7**
    
- `1001` → MSB = 1  
    → flip: `0110` = 6 → so **−6**

---
## ➕ Arithmetic with 1’s Complement

- **Add normally**
    
- If there is a **carry out**, **add it back** (end-around carry)
    

### Example:

```
  00000101   (+5)
+ 11111010   (-5)
-----------
  11111111   → + carry → 00000000 ✅
```

---

## ⚠️ Issues

### ❗ Two zeros:

```
+0 = 00000000  
-0 = 11111111
```

Causes complexity in comparisons and logic.

### ❗ Overflow:

- Occurs when result **exceeds representable range**
    
- No automatic overflow flag
    
- Must be checked manually
    

### Overflow Detection Rule:

> If adding two numbers with the **same sign**, and the result’s sign is **wrong**, overflow occurred.

### Example (3-bit):

```
011 = +3  
010 = +2  
011 + 010 = 101 → -2 ❌ overflow
```

→ +3 + +2 = +5, but **+5 is not representable** in 3-bit 1’s complement

---

## 📉 Value Range (n-bit)

$- (2^{n-1} - 1) \quad \text{to} \quad + (2^{n-1} - 1)$

| Bits | Min  | Max  |
| ---- | ---- | ---- |
| 3    | -3   | +3   |
| 4    | -7   | +7   |
| 8    | -127 | +127 |
