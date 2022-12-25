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