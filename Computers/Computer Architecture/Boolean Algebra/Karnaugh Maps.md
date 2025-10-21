---
tags:
  - saca1
---
### Variables: A, B, C, D

We'll use Gray code order (00, 01, 11, 10)

---

### 🧮 Given function:

f(A,B,C,D)=∑m(0,1,2,5,6,7,9,13)f(A, B, C, D) = \sum m(0, 1, 2, 5, 6, 7, 9, 13)f(A,B,C,D)=∑m(0,1,2,5,6,7,9,13)

This means 1s at those minterms.  
We'll also deduce the POS from the **0s** (maxterms):

f=∏M(3,4,8,10,11,12,14,15)
$f = \prod M(3, 4, 8, 10, 11, 12, 14, 15)f=∏M(3,4,8,10,11,12,14,15)$

---

### 🗺 K-Map

|AB \ CD|00|01|11|10|
|---|---|---|---|---|
|**00**|1|1|0|1|
|**01**|1|0|1|1|
|**11**|0|1|0|0|
|**10**|1|0|0|0|

---

## ✅ SOP Form (Sum of Products)

### 🔍 Grouping:

1. **Group of 4**: (0, 2, 4, 6) → 4-cell: $A'D'$
    
2. **Group of 2**: (0, 1) → $A'B'C'$
    
3. **Group of 2**: (6, 7) → $A'BC$
    
4. **Group of 2**: (0, 8) → $B'C'D'$
    
5. **Group of 1**: (13) → $ABC'D$
    

### ✏️ Final SOP:

$f=A'D'+A'B'C'+B'C'D'+A'BC+ABC'D$


### 🔍 Grouping 0s:

We can form these simplified **groups** of 0s (minimize using maxterms):

1. **Group of 4**: (10, 11, 14, 15) → 4-cell: $A'+C'$
    
2. **Group of 2**: (3, 11) → $B+C'+D'$
    
3. **Group of 2**: (9, 11) → $A'+B+D'$
    
4. **Group of 2**: (12, 14) → $A'+B'+D$
    
5. **Group of 1**: (5) → $A+B'+C+D'$
    

---

### ✅ Final Simplified POS

$f = (\overline{A} + \overline{C}) \cdot (B + \overline{C} + \overline{D}) \cdot (\overline{A} +B+\overline{D}) \cdot (\overline{A}+\overline{B}+D) \cdot (A+\overline{B}+C+\overline{D})$
