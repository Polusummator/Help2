# Hamilton

Поиск гамильтонова пути

Гамильтонов путь - путь, проходящий через все вершины графа ровно по одному разу

Решим задачу поиска минимального по весу гамильтонова пути в ориентированном графе
Для решения будем использовать дп по подмножествам
В маске будет набор из уже посещённых вершин
Второй параметр динамики - какая вершина была последней в пути

```cpp
bool get_bit(int mask, int i) {
    return mask & (1 << i);
}

int bit_to_1(int mask, int i) {
    return mask | (1 << i);
}

signed main() {
    int n;
    cin >> n;
    vector<vector<pair<int, int>>> graph(n);
    // ввод графа
    vector<vector<int>> dp((1 << n), vector<int>(n, 1e9));
    dp[1][0] = 0;
    vector<vector<pair<int, int>>> p((1 << n), vector<pair<int, int>>(n, {-1, -1}));
    for (int mask = 1; mask < (1 << n); mask++) {
        for (int v = 0; v < n; v++) {
            if (dp[mask][v] == 1e9 || !get_bit(mask, v)) {
                continue;
            }
            for (pair<int, int> pair : graph[v]) {
                int u = pair.first;
                int w = pair.second;
                if (get_bit(mask, u)) {
                    continue;
                }
                int nmask = bit_to_1(mask, u);
                if (dp[mask][v] + w < dp[nmask][u]) {
                    p[nmask][u] = {mask, v};
                    dp[nmask][u] = dp[mask][v] + w;
                }
            }
        }
    }
    int ans = 1e9;
    int pos = -1;
    for (int i = 0; i < n; i++) {
        if (dp[(1 << n) - 1][i] < ans) {
            ans = dp[(1 << n) - 1][i];
            pos = i;
        }
    }
    if (pos == -1) {
        cout << "-1\n";
        return 0;
    }
    cout << ans << el;
    vector<int> order;
    int cur_mask = (1 << n) - 1;
    while (true) {
        order.push_back(pos + 1);
        pair<int, int> prev = p[cur_mask][pos];
        if (prev == make_pair(-1, -1)) {
            break;
        }
        cur_mask = prev.first;
        pos = prev.second;
    }
    reverse(all(order));
    for (int i : order) {
        cout << i << ' ';
    }
    cout << el;
    return 0;
}
```