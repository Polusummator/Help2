# knapsack

Задача о рюкзаке

Задача о рюкзаке звучит так: есть рюкзак,
у рюкзака есть параметр W - максимальный вес, который можно положить в рюкзак,
также есть n предметов, у каждого предмета есть свой вес,
вопрос в том, какой максимальный вес можно набрать

У задачи есть вариации
Например, у каждого предмета есть стоимость, и вопрос состоит в том, какую максимальную стоимость можно набрать

Самое простое решение - решение за O(nW)
Для того, чтобы решить первую задачу, будем хранить dp[i][j], i - сколько предметов рассмотрено, j - какой вес набран
dp[i][j] - bool для этой задачи (можем/не можем набрать вес j)
Каждый раз мы смотрим, чему равно j - w[i]
Если это true, то мы раньше смогли набрать j - w[i], а значит сейчас можем набрать j

```cpp
signed main() {
    int n, W;
    cin >> n >> W;
    vector<int> w(n + 1);
    for (int i = 1; i <= n; i++)
        cin >> w[i];
    vector<vector<bool>> dp(n + 1, vector<bool>(W + 1, false));
    dp[0][0] = true;
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= W; j++) {
            dp[i][j] = dp[i - 1][j];
            if (w[i] <= j && dp[i - 1][j - w[i]])
                dp[i][j] = true;
        }
    }
    int ans = 0;
    for (int i = W; i >= 0; i--) {
        if (dp[n][i]) {
            ans = i;
            break;
        }
    }
    cout << ans << el;
    return 0;
}
```

Задача со стоимостями решается так же, только в dp[i][j] мы храним не bool, а максимальную стоимость для веса j
Код с восстановлением ответа:
```cpp
signed main() {
    int n, W;
    cin >> n >> W;
    vector<int> w(n + 1);
    vector<int> c(n + 1);
    for (int i = 1; i <= n; i++)
        cin >> w[i];
    for (int i = 1; i <= n; i++)
        cin >> c[i];
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));
    vector<vector<bool>> p(n + 1, vector<bool>(W + 1, false));
    dp[0][0] = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j <= W; j++) {
            dp[i][j] = dp[i - 1][j];
            p[i][j] = false;
            if (w[i] <= j && dp[i - 1][j - w[i]] + c[i] > dp[i][j]) {
                dp[i][j] = dp[i - 1][j - w[i]] + c[i];
                p[i][j] = true;
            }
        }
    }
    int pos = 0;
    for (int i = 0; i <= W; i++) {
        if (dp[n][i] > dp[n][pos])
            pos = i;
    }
    cout << dp[n][pos] << el;
    int cur_i = n, cur_j = pos;
    vector<int> ans;
    while (cur_i > 0) {
        if (!p[cur_i][cur_j])
            cur_i--;
        else {
            ans.push_back(cur_i);
            cur_j -= w[cur_i];
            cur_i--;
        }
    }
    sort(ans.begin(), ans.end());
    for (int i: ans)
        cout << i << el;
    return 0;
}
```

Но решение можно улучшить
Заметим, что мы строим целую таблицу, но на одной итерации нам нужно только строки i и i - 1
Тогда можно хранить только две строки и после каждой итерации делать swap

Пусть дана первая задача (без стоимостей)
Мы улучшили решение, сжав таблицу до 2 строк
Хотим восстановить ответ
Для этого заведём массив A длины W, A[i] - момент времени, когда мы впервые смогли набрать вес i
Для восстановление ответа мы смотрим на A[w] = i, вычитаем из w вес предмета i и продолжаем делать то же самое
Все предметы, которые мы будем вычитать - это и есть ответ

Можно ещё улучшить решение
dp[i][j] = dp[i][j] | dp[i][j - w[i]]
Тогда мы можем хранить bitset и быстро делать or
dp[0] = 1
dp = dp | (dp << w[i]) - делаем n раз

Для восстановления ответа в таком случае нужно искать элементы, которые впервые стали 1
Для этого не будем сразу обновлять dp, а заносить результат or в другой bitset
Чтобы найти элементы, которые впервые стали 1, нужно сделать xor dp и нового значения и пройтись по всем единицам
Все полученные позиции занести в массив A (про него написано выше)
Код (массив A называется make_ans):
```cpp
const int MAX_W = 1e6 + 1;
signed main() {
    int W;
    cin >> W;
    int n = 0;
    vector<int> w;
    int x;
    while (cin >> x) {
        w.push_back(x);
        n++;
    }
    w.insert(w.begin(), 0);
    bitset<MAX_W> dp;
    bitset<MAX_W> new_dp;
    bitset<MAX_W> p;
    dp[0] = true;
    vector<int> make_ans(W + 1);
    for (int i = 1; i <= n; i++) {
        new_dp = dp | (dp << w[i]);
        p = dp ^ new_dp;
        int last_bit = 0, bit;
        while (p._Find_next(last_bit) <= W) {
            last_bit = p._Find_next(last_bit);
            make_ans[last_bit] = i;
            p[last_bit] = false;
        }
        dp = new_dp;
    }
    int sum = 0;
    for (int i = W; i >= 0; i--) {
        if (make_ans[i]) {
            sum = i;
            break;
        }
    }
    cout << sum << el;
    int cur = sum;
    while (make_ans[cur]) {
        cout << w[make_ans[cur]] << ' ';
        cur -= w[make_ans[cur]];
    }
    return 0;
}
```

Случай задачи о рюкзаке с большими весами решается с meet-in-the-middle (описано в mitm)