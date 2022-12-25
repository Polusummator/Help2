# quick_pow

Быстрое возведение в степень

```py
def quick_pow(x, y):
    if not y:
        return 1
    if not y % 2:
        ans = quick_pow(x, y // 2)
        return ans * ans
    return quick_pow(x, y - 1) * x
```
Асимптотика - O(log n)