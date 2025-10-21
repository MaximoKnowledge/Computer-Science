---
tags:
  - saca1
---
### Basic Axioms (Huntington's Postulates)

| Axiom | Name              | Expression                                |
| ----- | ----------------- | ----------------------------------------- |
| A1    | Closure           | If a, b ∈ B, then a + b ∈ B and a · b ∈ B |
| A2a   | Identity (OR)     | a + 0 = a                                 |
| A2b   | Identity (AND)    | a · 1 = a                                 |
| A3a   | Commutative (OR)  | a + b = b + a                             |
| A3b   | Commutative (AND) | a · b = b · a                             |
| A4a   | Distributive      | a + (b · c) = (a + b) · (a + c)           |
| A4b   | Distributive      | a · (b + c) = (a · b) + (a · c)           |
| A5a   | Complement (OR)   | a + ā = 1                                 |
| A5b   | Complement (AND)  | a · ā = 0                                 |

## Fundamental Properties

|Property|Name|Expression|
|---|---|---|
|P1a|Idempotent (OR)|a + a = a|
|P1b|Idempotent (AND)|a · a = a|
|P2a|Null Element (OR)|a + 1 = 1|
|P2b|Null Element (AND)|a · 0 = 0|
|P3|Involution|(ā) = a|
|P4a|Absorption|a + (a · b) = a|
|P4b|Absorption|a · (a + b) = a|
|P5a|Associative (OR)|(a + b) + c = a + (b + c)|
|P5b|Associative (AND)|(a · b) · c = a · (b · c)|

## De Morgan's Laws

|Law|Expression|Dual|
|---|---|---|
|DM1|(a + b)' = a' · b'|(a · b)' = a' + b'|
|DM2|(a · b)' = a' + b'|(a + b)' = a' · b'|

## Additional Theorems

|Theorem|Name|Expression|
|---|---|---|
|T1a|Consensus|(a + b) · (a' + c) · (b + c) = (a + b) · (a' + c)|
|T1b|Consensus|(a · b) + (a' · c) + (b · c) = (a · b) + (a' · c)|
|T2a|Redundancy|a + (a' · b) = a + b|
|T2b|Redundancy|a · (a' + b) = a · b|
|T3a|Simplification|(a + b) · (a + b') = a|
|T3b|Simplification|(a · b) + (a · b') = a|
|T4|Exclusive OR|a ⊕ b = (a · b') + (a' · b)|
|T5|NAND|a ↑ b = (a · b)'|
|T6|NOR|a ↓ b = (a + b)'|

## Expansion Theorems (Shannon's Expansion)

|Type|Expression|
|---|---|
|Variable Expansion|f(a, b, c, ...) = a · f(1, b, c, ...) + a' · f(0, b, c, ...)|
|Unique Sum Form|f(a, b, c) = Σ mᵢ where mᵢ are minterms|
|Unique Product Form|f(a, b, c) = Π Mⱼ where Mⱼ are maxterms|

## Canonical Forms

### Minterms (Standard Products)

For n variables, there are 2ⁿ minterms:

|Variables|Minterm|Designation|
|---|---|---|
|a, b|a'b'|m₀|
|a, b|a'b|m₁|
|a, b|ab'|m₂|
|a, b|ab|m₃|

### Maxterms (Standard Sums)

For n variables, there are 2ⁿ maxterms:

|Variables|Maxterm|Designation|
|---|---|---|
|a, b|a + b|M₀|
|a, b|a + b'|M₁|
|a, b|a' + b|M₂|
|a, b|a' + b'|M₃|

## Duality Principle

**Rule:** In any Boolean expression, interchange:

- OR ('+') ↔ AND ('·')
- 0 ↔ 1
- Variables remain unchanged

**Example:**

- Original: a + (b · c) = (a + b) · (a + c)
- Dual: a · (b + c) = (a · b) + (a · c)

## Notation Conventions

|Operation|Symbols|Description|
|---|---|---|
|AND|·, ∧, &|Logical multiplication|
|OR|+, ∨, \||Logical addition|
|NOT|', ¬, ~|Logical negation|
|XOR|⊕, ⊻|Exclusive OR|
|NAND|↑, \|, ⊼|NOT AND|
|NOR|↓, ⊽|NOT OR|

## Minimization Rules

|Rule|Before|After|
|---|---|---|
|Combining|AB + AB'|A|
|Absorption|A + AB|A|
|Elimination|A + A'B|A + B|
|Consensus|AB + A'C + BC|AB + A'C|
