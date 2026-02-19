# User Guide

## What This Program Does

This script works with a simple layered Bayesian Network.

It takes in two files:

1. A **nodes file** that defines the structure of the network.
2. A **probabilities file** that contains:

   * The probability query you want to compute (on the first line), and
   * All prior and conditional probabilities for the nodes.


All variables are assumed to be Boolean

---

## How to Run It

From the command line, run:

```
python bn.py nodes.txt probs.txt
```

It will print something like:

```
P(A | B, ~C) = 0.123456
```

Or, if there’s no evidence:

```
P(A) = 0.123456
```

---

# Input File 1: nodes.txt (Network Structure)

This file defines the structure of the Bayesian Network using layers.

### Format Rules

* Each non-empty line represents one layer.
* Nodes on the same layer are separated by commas.
* Lines starting with `#` are comments and ignored.
* Node names should only use letters, numbers, or underscores
  (examples: `A`, `Alarm`, `o2`, `node_1`).

### Important Structure Rule

This program assumes a **layered network**:

* Every node in layer *i* (for i > 0) has **all nodes from layer i-1 as parents**.
* Nodes in layer 0 have **no parents**.

Example:

```
# Layer 0 (roots)
A, B

# Layer 1
C, D

# Layer 2
E
```

This means:

* Parents(A) = []
* Parents(B) = []
* Parents(C) = [A, B]
* Parents(D) = [A, B]
* Parents(E) = [C, D]

---

# Input File 2: probs.txt (Query + Probabilities)

This file contains:

* The **first non-comment line**: your query
* All remaining lines: probability definitions

Blank lines are ignored.

---

## A) The Query Line (First Line Only)

The query must look like:

```
P(X | evidence)
```

or simply:

```
P(X)
```

You can include multiple variables separated by commas.
Use `~` for negation.

Examples:

* `P(A)`
* `P(~A)`
* `P(A | B)`
* `P(A | ~B)`
* `P(A, ~C | B, D)`

If the evidence contradicts itself (like `P(A | B, ~B)`), the program will print:

```
evidence is not supporting each other
```

---

## B) Probability Lines

Each remaining line defines a probability.

It must look like:

```
P(Node) = p
```

or

```
P(Node | Parent1, ~Parent2, ...) = p
```

Where:

* `p` is a number between 0 and 1
* The line represents **P(Node = True | those parent values)**

Examples:

**Prior probability (no parents):**

```
P(A) = 0.3
```

**Conditional probability (if C has parents A and B):**

```
P(C | A, B) = 0.9
P(C | A, ~B) = 0.5
P(C | ~A, B) = 0.4
P(C | ~A, ~B) = 0.1
```

---

## Parent Sets Must Match Exactly

The parents listed after `|` must match exactly what the structure in `nodes.txt` implies.

If not, you’ll see:

```
Conditioning does not match parents
```

If a node name doesn’t exist:

```
Probability references unknown node
```

Other possible errors:

* `Bad probability line` -> wrong format
* `Probability out of range` -> not between 0 and 1

### Missing Entries

* Root nodes must have a prior:

  ```
  Missing prior probability
  ```

* Nodes with parents must include **every possible parent combination**.
  If one is missing:

  ```
  Missing CPT entry
  ```

---


# Output Format

With evidence:

```
P(query | evidence) = 0.xxxxxx
```

Without evidence:

```
P(query) = 0.xxxxxx
```

Probabilities are printed to 6 decimal places.

---

# Mini Example

## nodes.txt

```
A, B
C
```

This means:

* A and B are root nodes
* C has parents A and B

## probs.txt

```
P(C | A)

P(A) = 0.2
P(B) = 0.6

P(C | A, B) = 0.9
P(C | A, ~B) = 0.5
P(C | ~A, B) = 0.4
P(C | ~A, ~B) = 0.1
```

Run:

```
python bn.py nodes.txt probs.txt
```

You’ll get:

```
P(C | A) = some number
```

---

# Notes / Limitations

* All variables are Boolean.
* The network must be layered.
* Each layer depends on the entire previous layer.

---