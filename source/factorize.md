# factorize

Факторизация

Факторизация числа - это разложение числа на простые множители

```cpp
int n;
cin >> n;
vector<int> ans;
int n1 = n;
for (int i = 2; i * i <= n1; i++) {
    while (n % i == 0) {
        ans.push_back(i);
        n /= i;
    }
}
if (n > 1)
    ans.push_back(n);
cout << ans << endl;
```

Также факторизацию можно выполнить при помощи быстрого решета Эратосфена (см. eratosthenes_sieve)
