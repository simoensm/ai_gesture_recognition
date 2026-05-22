# Longest Common Subsequence

Tags: #edit-distance #string-similarity #LCS

---

## Definition

The **Longest Common Subsequence (LCS)** of two strings **x** and **y** is the longest sequence of characters that appears in both strings in the same relative order (not necessarily contiguous).

---

## Relation to Edit Distance

When the **substitution penalty = 2 × insertion/deletion cost**, the LCS can be derived directly from the [[Edit Distance (Levenshtein)]]:

$$\text{lcs}(\mathbf{x}, \mathbf{y}) = \frac{1}{2}(|\mathbf{x}| + |\mathbf{y}| - \text{dist}(\mathbf{x}, \mathbf{y}))$$

**Derivation:**
$$\text{dist}(\mathbf{x}, \mathbf{y}) = \text{lcs}(\mathbf{x}, \mathbf{x}) + \text{lcs}(\mathbf{y}, \mathbf{y}) - 2\,\text{lcs}(\mathbf{x}, \mathbf{y}) = |\mathbf{x}| + |\mathbf{y}| - 2\,\text{lcs}(\mathbf{x}, \mathbf{y})$$

---

## Example

Given:
- $\text{dist}(\text{lire}, \text{livre}) = 1$
- $|\text{lire}| = 4$, $|\text{livre}| = 5$

$$\text{lcs}(\text{lire}, \text{livre}) = \frac{1}{2}(4 + 5 - 1) = 4$$

The LCS is "lire" (4 characters).

---

*Part of → [[Edit Distance (Levenshtein)]] | [[MOC - Modelling Sequential Data]]*
