# LCIS

Наибольшая общая возрастающая подпоследовательность

Будет использовать дп

Решение за O(n^4):
dp[i][j] - ответ для префикса i массива a, префикса j массива b, i и j входят в ответ
Если a[i] != b[j], то dp[i][j] = 0
Иначе перебираем все i1 до i и j1 до j
Если a[i1] < a[i] и a[i1] = b[j1], то обновляем dp[i][j] через dp[i1][j1] + 1

Решение за O(n^3):
dp[i][j] - ответ для префикса i массива a, префикса j массива b, i входит в ответ
Если a[i] != b[j], то dp[i][j] = dp[i][j - 1]
Иначе dp[i][j] = max(dp[i1][j - 1]) + 1, если i1 < i и a[i1] < a[i]
Мы включаем j и перебираем i

Решение за O(n^2):
dp[i][j] - ответ для префикса i массива a, префикса j массива b, j входит в ответ
Если a[i] != b[j], то dp[i][j] = dp[i - 1][j]
Иначе dp[i][j] = max(dp[i - 1][j1]) + 1, j1 < j и b[j1] < b[j]
При фиксированном i мы перебираем все возможные j1, но это не надо, т.к. для каждого i мы можем запомнить j1

```cpp
int get(int i, int j, const vector<vector<int>>& dp) {
    if (i < 0 || j < 0) {
        return 0;
    }
    return dp[i][j];
}

signed main() {
    int n;
    cin >> n;
    vector<int> a(n);
    cin >> a;
    int m;
    cin >> m;
    vector<int> b(m);
    cin >> b;
    vector<vector<int>> dp(n, vector<int>(m));
    for (int i = 0; i < n; i++) {
        int j1 = -1;
        for (int j = 0; j < m; j++) {
            if (a[i] != b[j]) {
                dp[i][j] = get(i - 1, j, dp);
            }
            else {
                dp[i][j] = 1;
                if (j1 != -1) {
                    dp[i][j] = get(i - 1, j1, dp) + 1;
                }
            }
            if (b[j] < a[i] && get(i - 1, j, dp) > get(i - 1, j1, dp) {
                j1 = j;
            }
        }
    }
    cout << *max_element(dp[n - 1].begin(), dp[n - 1].end()) << el;
    return 0;
}
```

Код может быть неверным