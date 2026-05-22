---
title: "The string-to-string correction problem"
authors: ["Wagner, Robert A.", "Fischer, Michael J."]
year: 1974
citekey: "Wagner1974"
doi: "10.1145/321796.321811"
venue: "Journal of the ACM, 21(1), 168–173"
type: article
tags:
  - literature
  - edit-distance
  - dynamic-programming
  - string-algorithms
status: read
relevance: medium
created: 2026-05-20
---

# The string-to-string correction problem

> [!info] Metadata
> **Authors:** Robert A. Wagner, Michael J. Fischer
> **Year:** 1974
> **Venue:** Journal of the ACM, 21(1), 168–173
> **DOI:** [10.1145/321796.321811](https://doi.org/10.1145/321796.321811)
> **Citekey:** `Wagner1974`

---

## Abstract

Presents the standard $\mathcal{O}(nm)$ dynamic programming algorithm for computing the Levenshtein edit distance between two strings of lengths $n$ and $m$. This is the classical formalization that textbooks and implementations reference.

---

## Key Contributions

- Formalized Levenshtein's distance as a DP recurrence
- Proved $\mathcal{O}(nm)$ time complexity and $\mathcal{O}(\min(n,m))$ space complexity
- The "two-row trick" for space-efficient computation

## Relevance to [[Project - Gesture Recognition/00_Gesture_Recognition_MOC|Gesture Recognition Project]]

- **Method supported:** [[Project - Gesture Recognition/02_Edit_Distance|Edit Distance baseline]]
- **Role:** `baseline` — algorithmic reference
- **Key insight:** The Wagner-Fischer DP matrix is what `levenshtein_weighted()` implements in our edit distance code.

## Connections

- Implements → [[Levenshtein1966]]
- Applied to gesture recognition → [[Nyirarugira2014]]
