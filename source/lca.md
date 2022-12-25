# lca

LCA (Least Common Ancestor) - Наименьший Общий Предок

Задача LCA - нахождение для вершин дерева v, u предка с наибольшей высотой (высота считается от корня дерева)

Задача LCA решается несколькими способами:
1. Двоичные подъёмы
2. LCA -> RMQ
3. Алгоритм Тарьяна


1. ДВОИЧНЫЕ ПОДЪЁМЫ: O(nlogn) препроцессинга, O(logn) на запрос
    Будем хранить массив up, где up[v][k] - в какую вершину можно попасть из вершины v, поднявшись на 2^k вверх
    Массив up будет считаться в dfs, в котором также будут считаться времена входа и выхода из вершины
    up[v][k] = up[up[v][k - 1][k - 1] (делим отрезок на две части)

    Даются вершины v, u
    Сначала проверим, является ли какая-нибудь из них предком другой (для этого мы считали времена в dfs)
    Теперь будем подниматься из v по степеням двойки (массив up), следя за тем, чтобы v не стала предком u
    Ответ на задачу - up[v][0], т.е. непосредственный предок вершины v

    Код:
    ```cpp
    vector<vector<int>> g;
    vector<int> tin, tout;
    vector<vector<int>> up;
    int Log;
    int timer;

    void dfs(int v, int p = 0) {
        tin[v] = ++timer;
        up[v][0] = p;
        for (int i = 1; i <= Log; i++) {
            up[v][i] = up[up[v][i - 1]][i - 1];
        }
        for (int to: g[v]) {
            if (to != p) {
                dfs(to, v);
            }
        }
        tout[v] = ++timer;
    }

    bool is_parent(int v, int u) {
        return tin[v] <= tin[u] && tout[v] >= tout[u];
    }

    int lca(int v, int u) {
        if (is_parent(u, v)) {
            return u;
        }
        if (is_parent(v, u)) {
            return v;
        }
        for (int i = Log; i >= 0; i--) {
            if (!is_parent(up[v][i], u)) {
                v = up[v][i];
            }
        }
        return up[v][0];
    }

    signed main() {
        int n;
        cin >> n;
        g.resize(n);
        tin.resize(n);
        tout.resize(n);
        REP(n - 1) {
            int v, u;
            cin >> v >> u;
            v--;
            u--;
            g[v].push_back(u);
            g[u].push_back(v);
        }
        Log = 1;
        timer = 0;
        while ((1 << Log) <= n) {
            Log++;
        }
        for (int i = 0; i < n; i++) {
            up[i].resize(Log + 1);
        }
        dfs(0);
        int m;
        cin >> m;
        for (int i = 0; i < m; i++) {
            int v, u;
            cin >> v >> u;
            v--;
            u--;
            cout << lca(v, u) + 1 << el;
        }
        return 0;
    }
    ```

    Чтобы найти длину пути между двумя вершинами в дереве, из высоты вершины v и высоты вершины u вычесть высоту их lca

    Метод двоичных подъёмов позволяет считать функции на пути из одной вершины до другой
    Например, можно найти минимальное ребро на пути между двумя вершинами
    Для этого нужно считать массив, похожий на массив up
    Код:
    ```cpp
    vector<vector<int>> up, Min;
    vector<vector<pair<int, int>>> g;
    vector<int> tin, tout;
    int timer = 0;
    int Log;

    void dfs(int v, int p = 0, int M = 1e9) {
        tin[v] = ++timer;
        up[v][0] = p;
        Min[v][0] = M;
        for (int i = 1; i <= Log; i++) {
            up[v][i] = up[up[v][i - 1]][i - 1];
            Min[v][i] = min(Min[v][i - 1], Min[up[v][i - 1]][i - 1]);
        }
        for (auto pa: g[v]) {
            int u = pa.first;
            int w = pa.second;
            if (u != p) {
                dfs(u, v, w);
            }
        }
        tout[v] = ++timer;
    }

    bool is_parent(int v, int u) {
        return tin[v] <= tin[u] && tout[v] >= tout[u];
    }

    int get_min(int v, int u) {
        int res = 1e9;
        for (int i = Log; i >= 0; i--) {
            if (!is_parent(up[v][i], u)) {
                res = min(res, Min[v][i]);
                v = up[v][i];
            }
        }
        if (!is_parent(v, u)) {
            res = min(res, Min[v][0]);
        }
        for (int i = Log; i >= 0; i--) {
            if (!is_parent(up[u][i], v)) {
                res = min(res, Min[u][i]);
                u = up[u][i];
            }
        }
        if (!is_parent(u, v)) {
            res = min(res, Min[u][0]);
        }
        return res;
    }

    signed main() {
        int n;
        cin >> n;
        g.resize(n);
        tin.resize(n);
        tout.resize(n);
        up.resize(n);
        Min.resize(n);
        // формат ввода - (предок вершины i, вес ребра между вершиной i и её предком)
        for (int i = 1; i < n; i++) {
            int v, w;
            cin >> v >> w;
            v--;
            g[v].push_back({i, w});
            g[i].push_back({v, w});
        }
        Log = 1;
        while ((1 << Log) <= n) {
            Log++;
        }
        for (int i = 0; i < n; i++) {
            up[i].resize(Log + 1);
            Min[i].resize(Log + 1);
        }
        dfs(0);
        int m;
        cin >> m;
        REP(m) {
            int v, u;
            cin >> v >> u;
            v--;
            u--;
            cout << get_min(v, u) << el;
        }
        return 0;
    }
    ```

    Однако не всегда дерево вводится
    В задаче дерево может строится путём добавления вершин (запрос: привесить новую вершину к уже существующей)
    В таком дереве тоже можно считать lca, однако функция поиска lca изменится
    Ведь в изменяющемся дереве мы не можем посчитать времена входа и выхода
    Поэтому можно основываться на высотах вершин
    Пусть даны вершины v, u
    Сначала приведём их к одной высоте (двоичными подъёмами поднимем вершину с бОльшей высотой до высоты второй вершины)
    Если v == u, то мы нашли lca, иначе будем поднимать вершины
    Теперь мы можем одновременно поднимать обе вершины двоичными подъёмами
    Поднимать мы будем, пока у v и u не будет один и тот же прямой предок (предок, соединённый ребром)
    Тогда lca(v, u) будет up[v][0] (или up[u][0])

    Код:
    ```cpp
    int lca(int v, int u) {
        if (d[v] > d[u]) {
            swap(v, u);
        }
        for (int i = Log; i >= 0; i--) {
            if (d[up[u][i]] >= d[v]) {
                u = up[u][i];
            }
        }
        if (v == u) {
            return v;
        }
        for (int i = Log; i >= 0; i--) {
            if (up[v][i] != up[u][i]) {
                v = up[v][i];
                u = up[u][i];
            }
        }
        return up[v][0];
    }
    ```

    Двоичные подъёмы позволяют считать функции и на массивах (сумма, минимум...)
    Вводится запрос [L, R]
    Для ответа нужно пройти по степеням двойки от L до R и собрать ответ

5. LCA -> RMQ
    LCA можно свести к задаче RMQ
    Для этого нужно совершить эйлеров обход дерева (выписать вершину при заходе в неё)
    Но сам массив вершин понадобится только для поиска номера вершины
    Важен массив высот вершин, соответсвующий массиву выписанных вершин
    На массиве нужно построить структуру данных, позволяющую искать RMQ на отрезке (ДО, Sparse Table, SQRT-decomposition...)

    Для каждой вершины нужно хранить её индекс в массиве обхода (индекс любого вхождения)
    Чтобы найти LCA(v, u), нужно найти минимум на отрезке [ind[v], ind[u]] массива высот

    Реализация с ДО:
    ```cpp
    vector<vector<int>> g;
    vector<int> d;
    vector<int> a;
    vector<int> indexes;
    vector<bool> used;
    vector<pair<int, int>> t;
    int sz;

    void dfs(int v, int depth) {
        a.push_back(v);
        d.push_back(depth);
        indexes[v] = d.size() - 1;
        used[v] = true;
        for (int u: g[v]) {
            if (!used[u]) {
                dfs(u, depth + 1);
                a.push_back(v);
                d.push_back(depth);
                indexes[v] = d.size() - 1;
            }
        }
    }

    void build(int v, int l, int r) {
        if (r - l == 1) {
            t[v] = {d[l], l};
            return;
        }
        int m = (l + r) / 2;
        build(v * 2 + 1, l, m);
        build(v * 2 + 2, m, r);
        t[v] = min(t[v * 2 + 1], t[v * 2 + 2]);
    }

    pair<int, int> ask(int v, int l, int r, int askl, int askr) {
        if (l >= askr || r <= askl) {
            return {1e9, -1};
        }
        if (l >= askl && r <= askr) {
            return t[v];
        }
        int m = (l + r) / 2;
        return min(ask(v * 2 + 1, l, m, askl, askr), ask(v * 2 + 2, m, r, askl, askr));
    }

    int lca(int v, int u) {
        int vi = indexes[v];
        int ui = indexes[u];
        if (vi > ui) {
            swap(vi, ui);
        }
        pair<int, int> res = ask(0, 0, sz, vi, ui + 1);
        return a[res.second];
    }

    signed main() {
        int n;
        cin >> n;
        g.resize(n);
        indexes.resize(n);
        used.resize(n);
        REP(n - 1) {
            int v, u;
            cin >> v >> u;
            v--;
            u--;
            g[v].push_back(u);
            g[u].push_back(v);
        }
        dfs(0, 0);
        sz = d.size();
        t.resize(4 * sz);
        build(0, 0, sz);
        int m;
        cin >> m;
        for (int i = 0; i < m; i++) {
            int v, u;
            cin >> v >> u;
            v--;
            u--;
            cout << lca(v, u) + 1 << el;
        }
        return 0;
    }
    ```

    Реализация с Sparse Table:
    ```cpp
    vector<vector<int>> g;
    vector<int> d, a, indexes;
    vector<bool> used;
    vector<vector<pair<int, int>>> sparse;
    vector<int> logs;
    int sz;

    void dfs(int v, int depth) {
        used[v] = true;
        a.push_back(v);
        d.push_back(depth);
        indexes[v] = d.size() - 1;
        for (int u: g[v]) {
            if (!used[u]) {
                dfs(u, depth + 1);
                a.push_back(v);
                d.push_back(depth);
                indexes[v] = d.size() - 1;
            }
        }
    }

    void build_sparse() {
        logs.resize(sz + 1);
        logs[1] = 0;
        for (int i = 2; i <= sz; i++) {
            logs[i] = logs[i / 2] + 1;
        }
        sparse.resize(logs[sz] + 1, vector<pair<int, int>>(sz));
        for (int i = 0; i < sz; i++) {
            sparse[0][i] = {d[i], i};
        }
        for (int level = 1; (1 << level) <= sz; level++) {
            for (int i = 0; i + (1 << level) <= sz; i++) {
                sparse[level][i] = min(sparse[level - 1][i], sparse[level - 1][i + (1 << (level - 1))]);
            }
        }
    }

    pair<int, int> ask(int l, int r) {
        int len = r - l + 1;
        int level = logs[len];
        pair<int, int> ans = min(sparse[level][l], sparse[level][r - (1 << level) + 1]);
        return ans;
    }

    int lca(int v, int u) {
        int vi = indexes[v];
        int ui = indexes[u];
        if (vi > ui) {
            swap(vi, ui);
        }
        pair<int, int> res = ask(vi, ui);
        return a[res.second];
    }

    signed main() {
        int n;
        cin >> n;
        g.resize(n);
        used.resize(n);
        indexes.resize(n);
        REP(n - 1) {
            int v, u;
            cin >> v >> u;
            v--;
            u--;
            g[v].push_back(u);
            g[u].push_back(v);
        }
        dfs(0, 0);
        sz = d.size();
        build_sparse();
        int m;
        cin >> m;
        REP(m) {
            int v, u;
            cin >> v >> u;
            v--;
            u--;
            cout << lca(v, u) + 1 << el;
        }
        return 0;
    }
    ```

6. АЛГОРИТМ ТАРЬЯНА
    Алгоритм Тарьяна решает задачу LCA в оффлайне (все запросы должны быть даны в начале, алгоритм отвечает на запросы не по порядку)
    Пройдём по дереву при помощи dfs
    Допустим мы стоим в вершине v, а нам нужно найти LCA(v, u)
    Пусть вершина u уже посещена
    Тогда LCA(v, u) - это какой-то предок вершины v, для которого u является потомком
    Пусть p - какой-нибудь предок вершины v; тогда LCA(v, i), где i - все вершины лежащие в поддеревьях вершины p (кроме того дерева, в котором лежит v), будет равно p
    Т.е. все предки вершины v - это какие-то представители классов вершин, у которых LCA с v одинаков
    Получается, нам всего лишь нужно найти представителя класса, в котором лежит вершина u
    Это делается при помощи СНМ
    Важно также хранить массив ancestor, в котором будет лежать правильная вершина (ведь представитель класса может быть любым при ранговой эвристике)
    Код:
    ```cpp
    vector<vector<int>> g, Q;
    vector<int> p, d, ancestor;
    vector<bool> used;
    map<pair<int, int>, int> ans;
    vector<pair<int, int>> Q_pair;

    int get_par(int v) {
        if (v == p[v]) {
            return v;
        }
        return p[v] = get_par(p[v]);
    }

    void unite(int v, int u, int new_ancestor) {
        v = get_par(v);
        u = get_par(u);
        if (v != u) {
            if (d[u] > d[v]) {
                swap(v, u);
            }
            p[u] = v;
            if (d[v] == d[u]) {
                d[v]++;
            }
            ancestor[v] = new_ancestor;
        }
    }

    void dfs(int v) {
        p[v] = v;
        ancestor[v] = v;
        used[v] = true;
        for (int u: g[v]) {
            if (!used[u]) {
                dfs(u);
                unite(v, u, v);
            }
        }
        for (int u: Q[v]) {
            if (used[u]) {
                int lca = ancestor[get_par(u)];
                ans[{v, u}] = lca;
                ans[{u, v}] = lca;
            }
        }
    }

    signed main() {
        fastIO;
        int n;
        cin >> n;
        g.resize(n);
        used.resize(n);
        ancestor.resize(n);
        p.resize(n);
        d.resize(n);
        Q.resize(n);
        int v, u;
        REP(n - 1) {
            cin >> v >> u;
            v--;
            u--;
            g[v].push_back(u);
            g[u].push_back(v);
        }
        int m;
        cin >> m;
        REP(m) {
            cin >> v >> u;
            v--;
            u--;
            Q[v].push_back(u);
            Q[u].push_back(v);
            Q_pair.emplace_back(v, u);
        }
        dfs(0);
        for (auto pa: Q_pair) {
            v = pa.first;
            u = pa.second;
            cout << ans[{v, u}] + 1 << el;
        }
        return 0;
    }
    ```
