# condensation

Конденсация

Конденсация - это выделение в графе компонент сильной связности
Компонента сильной связности (в ориентированном графе) - множество вершин, в котором от каждой вершины можно добраться до любой другой

Чтобы выделить компоненты, нужно сначала сделать top_sort (граф может быть с циклами, но это не важно)
Теперь нужно транспонировать граф (развернуть все рёбра). Это можно сделать ещё при вводе
После этого пройдём по транспонированному графу в порядке top_sort
Все вершины, которые мы прошли за один запуск dfs - это одна компонента сильной связности

По конденсации можно построить граф (сжать каждую компоненту в вершину)
Такой граф - дерево
Если требуется, чтобы в конденсации не было кратных рёбер, можно хранить map для добавленных рёбер между компонентами

Код с сжатием в вершины:
```cpp
vector<vector<int>> graph, r_graph, comp_graph;
vector<int> comp;
vector<int> ts;
vector<bool> used;
int n;

void dfs(int v) {
    used[v] = true;
    for (int u : graph[v]) {
        if (!used[u]) {
            dfs(u);
        }
    }
    ts.push_back(v);
}

void top_sort() {
    for (int i = 0; i < n; i++) {
        if (!used[i]) {
            dfs(i);
        }
    }
    reverse(ts.begin(), ts.end());
}

void dfs_cond(int v, int cur_comp) {
    used[v] = true;
    comp[v] = cur_comp;
    for (int u : r_graph[v]) {
        if (!used[u]) {
            dfs_cond(u, cur_comp);
        }
        else if (comp[u] != cur_comp) {
            comp_graph[comp[u]].push_back(cur_comp);
        }
    }
}

signed main() {
    int m;
    cin >> n >> m;
    graph.resize(n);
    r_graph.resize(n);
    comp.resize(n);
    used.resize(n);
    for (int i = 0; i < m; i++) {
        int v, u;
        cin >> v >> u;
        v--;
        u--;
        graph[v].push_back(u);
        r_graph[u].push_back(v);
    }
    top_sort();
    used.assign(n, false);
    int cur_comp = 0;
    for (int i : ts) {
        if (!used[i]) {
            comp_graph.push_back({});
            dfs_cond(i, cur_comp);
            cur_comp++;
        }
    }
    return 0;
}
```

Задача о минимальном достижимом числе для каждой вершины:
```cpp
int n, m;
vector<vector<int>> g;
vector<int> used;
vector<int> ts;
vector<vector<int>> rg;
vector<int> comp;
vector<vector<int>> cg;
vector<int> num; // числа в исходных вершинах
vector<int> numcond; // числа в сжатых вершинах
vector<int> dp; // ответ для сжатых
vector<int> ans; // ответ для исходных

void dfs(int v) {
    used[v] = 1;
    for (int u: g[v]) {
        if (!used[u]) {
            dfs(u);
        }
    }
    used[v] = 2;
    ts.push_back(v);
}

void dfscond(int v, int cur_comp) {
    used[v] = 1;
    comp[v] = cur_comp;
    for (int u: rg[v]) {
        if (!used[u]) {
            dfscond(u, cur_comp);
        }
        else if (cur_comp != comp[u]) {
            cg[comp[u]].push_back(cur_comp);
        }
    }
}

signed main() {
    cin >> n >> m;
    g.resize(n);
    rg.resize(n);
    used.resize(n);
    comp.resize(n);
    num.resize(n);
    ans.resize(n);
    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        g[u - 1].push_back(v - 1);
        rg[v - 1].push_back(u - 1);
    }
    cin >> num;
    for (int i = 0; i < n; i++) {
        if (!used[i]) {
            dfs(i);
        }
    }
    reverse(ts.begin(), ts.end());
    used.assign(n, false);
    int cur_comp = 0;
    for (int v: ts) {
        if (!used[v]) {
            cg.push_back(vector<int>());
            dfscond(v, cur_comp);
            cur_comp++;
        }
    }
    numcond.resize(cur_comp, INT_MAX);
    dp.resize(cur_comp);
    for (int i = 0; i < n; i++) {
        numcond[comp[i]] = min(numcond[comp[i]], num[i]);
    }
    for (int i = cur_comp - 1; i >= 0; i--) {
        dp[i] = numcond[i];
        for (int v: cg[i]) {
            dp[i] = min(dp[i], dp[v]);
        }
    }
    for (int i = 0; i < n; i++) {
        ans[i] = dp[comp[i]];
    }
    return 0;
}
```