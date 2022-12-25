# maxmatch

Максимальное паросочетание

Паросочетание - это набор рёбер графа, в котором никакие два ребра не имеют общей вершины

Максимальное паросочетание - это паросочетание, состоящее из максимального кол-ва рёбер (для невзвешенного графа) или имеющее максимальный суммарный вес (для взвешенного графа)

Паросочетание ищется по-разному для разных видов графа:
1. Для дерева
2. Для произвольного графа
3. Для двудольного графа

1. ДЕРЕВО
    Для поиска максимального паросочетания в дереве используется ДП по поддеревьям
    dp[v][0] - вершина v не соединена в просочетании с каким-либо из сыновей
    dp[v][1] - вершина v соединена в просочетании с каким-либо из сыновей

    Если вершина не соединена с каким-лио сыном, то dp[v][0] = max(dp[ui][0], dp[ui][1]) (не важны состояния сыновей)
    Но если вершина соединена с каким-либо из сыновей, то dp[v][0] = max(dp[uk][0] + max(dp[ui][0], dp[ui][1]), ui != uk)

    Код:
    ```cpp
    vector<vector<pair<int, int>>> g;
    vector<vector<int>> dp;

    void dfs(int v, int pr) {
        for (pair<int, int> p: g[v]) {
            int u = p.first;
            if (u != pr) {
                dfs(u, v);
                dp[v][0] += max(dp[u][0], dp[u][1]);
            }
        }
        for (pair<int, int> p: g[v]) {
            int u = p.first;
            int w = p.second;
            if (u != pr) {
                dp[v][1] = max(dp[v][1], dp[u][0] + dp[v][0] - max(dp[u][0], dp[u][1]) + w);
            }
        }
    }

    signed main() {
        int n;
        cin >> n;
        g.resize(n);
        dp.resize(n, vector<int>(2));
        REP(n - 1) {
            int v, u, w;
            cin >> v >> u >> w;
            v--;
            u--;
            g[v].push_back({u, w});
            g[u].push_back({v, w});
        }
        dfs(0, -1);
        cout << max(dp[0][0], dp[0][1]) << el;
        return 0;
    }
    ```

2. ПРОИЗВОЛЬНЫЙ ГРАФ
    Здесь будет рассматриваться медленный, но простой алгоритм для произвольного графа

    В алгоритме используется ДП по подмножествам
    dp[mask] - ответ, если все вершины подмножества находятся в паросочетании
    Если из всех вершин подмножества нельзя составить паросочетание, то dp[mask] = -1

    В текущем подмножестве выберем любую вершину v и посмотрим на всех её сыновей
    Пусть мы сейчас рассматривает сына u
    Если dp[mask без вершин u и v] не равен -1, то обновим dp[mask]

    Код:
    ```cpp
    vector<vector<pair<int, int>>> g;
    vector<int> dp;

    int get_last1_bit(int mask) {
    // предполагается, что бит 1 всегда есть
    for (int i = 0; i < 31; i++) {
        if ((1 << i) & mask) {
            return i;
        }
    }
    }

    int get_bit(int mask, int i) {
        return ((1 << i) & mask);
    }

    int bit_to0(int mask, int i) {
        return mask & ~(1 << i);
    }

    signed main() {
        int n;
        cin >> n;
        g.resize(n);
        dp.resize((1 << n), -1);
        int m;
        cin >> m;
        REP(m) {
            int v, u, w;
            cin >> v >> u >> w;
            v--;
            u--;
            g[v].push_back({u, w});
            g[u].push_back({v, w});
        }
        dp[0] = 0;
        int ans = 0;
        for (int mask = 1; mask < (1 << n); mask++) {
            int v = get_last1_bit(mask);
            for (pair<int, int> p: g[v]) {
                int u = p.first;
                int w = p.second;
                if (get_bit(mask, u)) {
                    int nmask = bit_to0(mask, u);
                    nmask = bit_to0(nmask, v);
                    if (dp[nmask] != -1) {
                        dp[mask] = max(dp[mask], dp[nmask] + w);
                        ans = max(ans, dp[mask]);
                    }
                }
            }
        }
        cout << ans << el;
        return 0;
    }
    ```

3. ДВУДОЛЬНЫЙ ГРАФ
    См. Kuhn (там же применение паросочетаний)
