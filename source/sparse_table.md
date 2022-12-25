# sparse_table

Sparse table (разреженная таблица)
Разреженная таблица (sparse table)

Sparse table - структура, позволяющая хранить и получать нужную информацию о всех отрезках массива
Sparse table - это двумерный массив, который хранит информацию о всех отрезках, длины которых равны степеням двойки

```
таблица выглядит так:
       степени
    |0 1 2 3 4 5
-----------------
и 0 |             
н 1 |  ч           
д 2 |    и         
е 3 |      с       
к 4 |        л     
с 5 |          а   
```

Сверху - степени двойки, слева - индекс начала отрезка (индекс первого элемента отрезка)

```cpp
int n;
cin >> n;
vector<int> a(n);
cin >> a;
vector<int> logs(n + 1);
logs[1] = 0;
for (int i = 2; i <= n; i++) {
    logs[i] = logs[i / 2] + 1;
}
vector<vector<int>> sparse(logs[n] + 1, vector<int>(n));
for (int i = 0; i < n; i++) {
    sparse[0][i] = a[i];
}
for (int level = 1; (1 << level) <= n; level++) {
    for (int i = 0; i + (1 << level) <= n; i++) {
        sparse[level][i] = min(sparse[level - 1][i], sparse[level - 1][i + (1 << (level - 1))]);
    }
}
int l, r;
cin >> l >> r;
l--;
r--;
int len = r - l + 1;
int level = logs[len];
int ans = min(sparse[level][l], sparse[level][r - (1 << level) + 1]);
cout << ans << el;
```

sparse table - неизменяемая структура (для изменения элементов нужно перестраивать всю таблицу)
