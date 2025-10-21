---
tags:
  - saca1
---
# 🧮 2’s Complement – Quick Summary

## ✅ Purpose

Efficient method to represent signed integers in binary. Fixes 1’s complement issues (no double zero, clean arithmetic).

---

## ⚙️ How It Works

- **Positive numbers**: standard binary
    
- **Negative numbers**:
    
    - Invert all bits
        
    - Add 1
        

Example (4-bit):

`+3 = 0011   -3 → invert: 1100 → add 1 = 1101`

---

## 🧠 Interpretation

- MSB = 0 → positive
    
- MSB = 1 → negative → invert + 1 → attach minus
    

Example:

`1001 → invert: 0110 → +1 = 0111 → -7`

---

## ➕ Addition & Overflow

- Add normally
    
- **Overflow if** same sign inputs → opposite sign output
    

Example (4-bit):

`0111 (+7) + 0001 (+1) = 1000 (−8) ❌ overflow`

---

## ✖️ Multiplication

**Sign-extend both numbers to 2n bits** and do regular binary multiplication.

---

## 📉 Value Range (n-bit)

$-2^{n-1} \quad \text{ to } \quad 2^{n-1} - 1$