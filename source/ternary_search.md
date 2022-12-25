# ternary_search

Тернарный/троичный поиск

Тернарный поиск применяется для функций с одним экстремумом
Тернарный поиск ищет этот экстремум
Для этого отрезок делится на 3 части (m1 и m2)

```cpp
// f(m1) < f(m2) => r = m2
// f(m1) > f(m2) => l = m1
// f(m1) = f(m2) => l = m1 and r = m2
double l = -100, r = 100;
for (int it = 0; it <= 10; it++) {
    double m1 = l + (r - l) / 3;
    double m2 = m1 + (r - l) / 3;
    if (f(m1) < f(m2)) // если выгнуто вниз, то <, иначе >
        r = m2;
    else
        l = m1;
}
```

Целочисленный тернарный поиск
Если мы ищем на целых числах, то может быть проблема с условием в while
Если мы оставляем условие (r - l > 1), то код может зациклиться
Ведь если r - l == 2, то после итерации r и l останутся такими же, либо алгоритм проскочит разницу (l станет больше r)
Мы считаем, что граница r не включается
Тогда сделаем условие while (l < r - 2)

В конце у нас два возможных значения: l и l + 1
Просто выберем одно из них:

```cpp
if (f(l) > f(l + 1)) {
    l++;
}
```

# sorts

Сортировки

1. Сортировка слиянием (mergesort):
    ```cpp
    vector<int> tmp;
    void mergesort(vector<int>& a, int l, int r) {
        if (l == r - 1)
            return;
        int m = (l + r) / 2;
        mergesort(a, l, m);
        mergesort(a, m, r);
        merge(a.begin() + l, a.begin() + m, a.begin() + m, a.begin() + r, tmp.begin() + l);
        copy(tmp.begin() + l, tmp.begin() + r, a.begin() + l);
    }
    ```
    Из основной программы запускается с l = 0 и r = n

    Если написать свой merge, то можно считать кол-во инверсий в массиве
    vector<int> tmp;
    ll ans = 0;
    void mergesort(vector<int>& a, int l, int r) {
        if (l == r - 1)
            return;
        int m = (l + r) / 2;
        mergesort(a, l, m);
        mergesort(a, m, r);
        int it1 = l, it2 = m, pos = l;
        while(it1 < m || it2 < r) {
            if (it1 == m) {
                tmp[pos] = a[it2];
                ++it2;
            }
            else if (it2 == r) {
                tmp[pos] = a[it1];
                ++it1;
            }
            else if (a[it1] <= a[it2]){
                tmp[pos] = a[it1];
                ++it1;
            }
            else {
                tmp[pos] = a[it2];
                ++it2;
                ans += (m - it1);
            }
            ++pos;
        }
        copy(tmp.begin() + l, tmp.begin() + r, a.begin() + l);
    }

1. Пузырьковая сортировка (bubble sort):
    ```cpp
    int n;
    cin >> n;
    vector<int> a(n);
    cin >> a;
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - 1; j++) {
            if (a[j] > a [j + 1])
                swap(a[j], a[j + 1]);
        }
    }
    ```

1. Сортировка выбором минимума/максимума:
    ```cpp
    int n;
    cin >> n;
    vector<int> a(n);
    cin >> a;
    for (int i = 0; i < n; i++) {
        int pos = min_element(a.begin() + i, a.end()) - a.begin();
        swap(a[i], a[pos]);
    }
    cout << a << el;
    ```

1. Быстрая сортировка (quick sort)
   
    Каждый раз берём какой-то элемент x и разделяем массив на две части: все числа < x и все числа >= x
    После этого запускаем сортировку для этих двух массивов

    ```py
    # Возвращает конец левого массива и начало правого (указатели l, r)
    def partition(a, l, r, pivot):
        while l <= r:
            if a[l] < pivot:
                l += 1
            else:
                if a[r] > pivot:
                    r -= 1
                else:
                    a[l], a[r] = a[r], a[l]
                    l += 1
                    r -= 1
        return l, r

    def quick_sort(a, l, r):
        if r > l:
            borders = partition(a, l, r, a[(l + r) // 2])
            quick_sort(a, l, borders[1])
            quick_sort(a, borders[0], r)
    ```

    Из основной программы вызывается quick_sort(a, 0, n - 1)

2. Поразрядная сортировка (radix sort)
    Подходит для случаев, когда кол-во разрядов в числах меньше, чем logn (но сортировка может сортировать и не числа)
    Сначала все числа сортируются по последней цифре, потом по предпоследней и т.д. Каждый шаг является модификацией сортировки подсчётом
    Для удобства числа вводятся в виде строк, и к ним приписываются 0, чтобы длина всех чисел была одинакова

    ```py
    n = int(input())
    max_len = 0
    a = input().split()
    for i in range(n):
        max_len = max(max_len, len(a[i]))
    for i in range(n):
        a[i] = '0' * (max_len - len(a[i])) + a[i]
    buckets = [[] for i in range(10)]
    for cur_digit in range(max_len - 1, -1, -1):
        for i in range(n):
            digit = int(a[i][cur_digit])
            buckets[digit].append(a[i])
        a.clear()
        for b in range(10):
            for num in buckets[b]:
                a.append(num)
            buckets[b].clear()
    print(*a)
    ```