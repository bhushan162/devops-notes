# Quick Revision & Handy C++ Cheatsheet

---

## 📌 Fast Output & Printing
```cpp
// Print an integer
cout << i;

// Fast newline: preferred over endl in competitive programming because endl flushes the output buffer every time
cout << "\n";
```

---

## 📌 Finding Square Root of a Number
```cpp
#include <cmath>

// Computes square root (returns double, cast to int if needed)
int num = sqrt(num2);
```

---

## 📌 Easily Swapping Two Variables
* **Note:** Need to learn properly.
* `std::swap(a, b)` swaps the values of two variables directly in-place without needing a manual temporary variable:
```cpp
#include <algorithm> // or <utility>

int i = 10;
int j = 20;

// Swaps i and j so i becomes 20, j becomes 10
swap(i, j);
```
