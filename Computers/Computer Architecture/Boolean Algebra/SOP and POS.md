---
tags:
  - saca1
---

## Minterm/Maxterm Tables

### 3-Variable Minterm Table

|Index|A|B|C|Minterm|Maxterm|
|---|---|---|---|---|---|
|0|0|0|0|A'B'C'|A+B+C|
|1|0|0|1|A'B'C|A+B+C'|
|2|0|1|0|A'BC'|A+B'+C|
|3|0|1|1|A'BC|A+B'+C'|
|4|1|0|0|AB'C'|A'+B+C|
|5|1|0|1|AB'C|A'+B+C'|
|6|1|1|0|ABC'|A'+B'+C|
|7|1|1|1|ABC|A'+B'+C'|

### Example Truth Table → Forms

|A|B|C|F|Index|
|---|---|---|---|---|
|0|0|0|0|0|
|0|0|1|1|1 ←|
|0|1|0|1|2 ←|
|0|1|1|0|3|
|1|0|0|0|4|
|1|0|1|1|5 ←|
|1|1|0|0|6|
|1|1|1|1|7 ←|

**Results:**

- SOP: F = Σm(1,2,5,7) = A'B'C + A'BC' + AB'C + ABC
- POS: F = ΠM(0,3,4,6) = (A+B+C)(A+B'+C')(A'+B+C)(A'+B'+C)

## Standard ↔ Canonical Conversion Tables

### Standard SOP → Canonical SOP

|Standard Term|Missing Variables|Expansion|Canonical Terms|
|---|---|---|---|
|AB|C|AB(C+C')|ABC + ABC'|
|BC'|A|BC'(A+A')|ABC' + A'BC'|
|A'C|B|A'C(B+B')|A'BC + A'B'C|

### Standard POS → Canonical POS

|Standard Term|Missing Variables|Expansion|Canonical Terms|
|---|---|---|---|
|(A+B)|C|(A+B+CC')|(A+B+C)(A+B+C')|
|(B'+C)|A|(B'+C+AA')|(A+B'+C)(A'+B'+C)|
|(A'+C')|B|(A'+C'+BB')|(A'+B+C')(A'+B'+C')|

## NAND Implementation Tables

### SOP → NAND-NAND Conversion

|Step|Operation|Example: F = AB + A'C|
|---|---|---|
|1|Original SOP|F = AB + A'C|
|2|Double complement|F = ((AB + A'C)')'|
|3|Apply De Morgan's|F = ((AB)' · (A'C)')'|
|4|NAND structure|First level: NAND gates, Second level: NAND gate|

### NAND Gate Implementation Table

|Original Gate|NAND Equivalent|Circuit|
|---|---|---|
|NOT A|NAND(A,A)|A ──┐ NAND ── A'|
|AND(A,B)|NAND(A,B) + NOT|A,B → NAND → NOT|
|OR(A,B)|NOT(A) + NOT(B) → NAND|A,B → NOT → NAND|

## NOR Implementation Tables

### POS → NOR-NOR Conversion

|Step|Operation|Example: F = (A+B)(A'+C)|
|---|---|---|
|1|Original POS|F = (A+B)(A'+C)|
|2|Double complement|F = (((A+B)(A'+C))')'|
|3|Apply De Morgan's|F = ((A+B)' + (A'+C)')'|
|4|NOR structure|First level: NOR gates, Second level: NOR gate|

### NOR Gate Implementation Table

|Original Gate|NOR Equivalent|Circuit|
|---|---|---|
|NOT A|NOR(A,A)|A ──┐ NOR ── A'|
|OR(A,B)|NOR(A,B) + NOT|A,B → NOR → NOT|
|AND(A,B)|NOT(A) + NOT(B) → NOR|A,B → NOT → NOR|