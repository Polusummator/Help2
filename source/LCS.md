# LCS

Наибольшая общая подпоследовательность

Заведём массив dp, где dp[i][j] - НОП для префикса i первой последовательности и префикса j второй последовательности
Будем идти по увеличению i и j

Если a[i] == b[j], то можно взять ответ для префиксов i - 1, j - 1 и добавить этот новый элемент
Если a[i] != b[j], то dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
Т.к. клетка (i, j) является максимумом из прямоугольника, нам не нужно искать максимум на всём прямоугольнике
Мы можем взять максимум из клеток (i - 1, j) и (i, j - 1), т.к. они покрывают весь прямоугольник

Для восстановления ответа, нужно знать, откуда мы пришли
Функция get нужна для того, чтобы не писать if для обработки клеток на границе таблицы

```cpp
int get(int i, int j, const vector<vector<int>>& dp) {
    if (i < 0 || j < 0)
        return 0;
    return dp[i][j];
}

signed main() {
    int n, m;
    cin >> n;
    vector<int> a(n);
    cin >> a >> m;
    vector<int> b(m);
    cin >> b;
    vector<vector<int>> dp(n, vector<int>(m));
    vector<vector<int>> p(n, vector<int>(m, -1));
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (a[i] == b[j]) {
                dp[i][j] = get(i - 1, j - 1, dp) + 1;
                p[i][j] = 1;
            }
            else {
                if (get(i - 1, j, dp) < get(i, j - 1, dp)) {
                    dp[i][j] = get(i, j - 1, dp);
                    p[i][j] = 2;
                }
                else {
                    dp[i][j] = get(i - 1, j, dp);
                    p[i][j] = 3;
                }
            }
        }
    }
    // make answer
    vector<int> ans;
    int curi = n - 1, curj = m - 1;
    while (curi >= 0 && curj >= 0) {
        if (p[curi][curj] == 1) {
            ans.push_back(a[curi]);
            curi--;
            curj--;
        }
        else if (p[curi][curj] == 2) {
            curj--;
        }
        else {
            curi--;
        }
    }
    reverse(ans.begin(), ans.end());
    cout << ans << el;
    return 0;
}
```