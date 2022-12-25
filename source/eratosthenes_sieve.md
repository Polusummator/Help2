# eratosthenes_sieve

```py
N = int(input())                           # До какого числа мы ищем простые числа
isPrime = [True] * (N + 1)                 # Создание списка из True
Primes = []                                # Список, в котором будут простые числа
for d in range(2, N + 1):
    if isPrime[d]:
        Primes.append(d)
        for i in range(d ** 2, N + 1, 2): # Убираем лишние числа
            isPrime[i] = False
print(Primes)
```

Алгоритм выше работает за O(n log log n)
Существует алгоритм решета, работающий за O(n):
vector<int> lp(N + 1, 0), pr;  // lp[i] - минимальный простой делитель числа i; pr - список простых чисел

```cpp
for (int i = 2; i <= N; i++) {
    if (!lp[i]) {              // простое число или нет
        lp[i] = i;
        pr.push_back(i);
    }
    for (int j = 0; j < pr.size() && pr[j] * i <= N && pr[j] <= lp[i]; j++)
        lp[pr[j] * i] = pr[j];
}
```

С помощью второго алгоритма можно выполнить факторизацию числа

```cpp
vector<int> factorize(int n, const vector<int>& lp) {
    vector<int> ans;
    while (n > 1) {
        ans.push_back(lp[n]);
        n /= lp[n];
    }
    return ans;
}
```

# check_for_simplicity

```py
z = False
x = int(input())
if x % 2 == 0 and x != 2:                    # Проверка, является ли число 2, и делится ли оно на два
    print('NO')
else:
    for i in range(3, int(x ** 0.5) + 1, 2):
        if x % i == 0:
            z = True
            break                            # Если находится хотя бы один делитель, то число уже не является простым
    if z:
        print('NO')
    else:
        print('YES')
```
