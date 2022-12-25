# Levenshtein

Расстояние Левенштейна

Есть две строки
Мы хотим получить вторую из первой путём нескольких операций, потратив наименьшее число денег
Возможные операции:
1. Удалить символ на позиции i, заплатив delCost
2. Вставить символ на позицию i, заплатив addCost
3. Поменять символ на другой на позиции i, заплатив repCost

Решение использует дп
Заведём массив dp, где dp[i][j] - ответ для префикса i первой строки и префикса j второй строки
i будет от 0 до n включительно, и j будет от 0 до m включительно, потому что мы должны считать ответ и для пустых строк
```
           ┌ 0,                если i = j = 0
           | j * addCost,      если i = 0
           | i * delCost,      если j = 0
dp[i][j] = | dp[i - 1][j - 1], если a[i - 1] = b[j - 1]
           | min( dp[i - 1][j] + delCost,      - мы можем удалить символ
           |      dp[i][j - 1] + addCost,      - мы можем вставить новый символ
           └      dp[i - 1][j - 1] + repCost ) - мы можем поменять символ
```
```cpp
string a, b;
cin >> a >> b;
int n = a.size();
int m = b.size();
int addCost, delCost, repCost;
cin >> addCost >> delCost >> repCost;
vector<vector<int>> dp(n + 1, vector<int>(m + 1));
for (int i = 0; i <= n; i++) {
    for (int j = 0; j <= m; j++) {
        if (i == 0 && j == 0) {
            continue;
        }
        if (i == 0) {
            dp[i][j] = j * addCost;
        }
        else if (j == 0) {
            dp[i][j] = i * delCost;
        }
        else if (a[i - 1] == b[i - 1]) {
            dp[i][j] = dp[i - 1][j - 1];
        }
        else {
            dp[i][j] = min({dp[i - 1][j] + delCost, dp[i][j - 1] + addCost, dp[i - 1][j - 1] + repCost});
        }
    }
}
cout << dp[n][m] << el;
```