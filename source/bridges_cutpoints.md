# bridges_cutpoints

Мосты и точки сочленения

Мост - ребро, при удалении которого граф распадается (число компонент связности увеличивается)
Запустим dfs
Из вершины могут идти прямые рёбра (ребра вниз), а также обратные рёбра (рёбра, которые ведут в вершины выше предка)
Если у вершины есть обратные рёбра, то ребро (предок - текущая вершина) не является мостом

```cpp
struct Edge {
    int u;
    int index;
    bool isbridge;
    Edge* back;
    Edge() {}
    Edge(int u_, int index_) {
        isbridge = false;
        u = u_;
        index = index_;
    }
};

vector<vector<Edge*>> graph;
vector<bool> used;
vector<int> depth;
vector<int> up;
vector<int> ans;

void dfs(int v, int d = 0, Edge* p = nullptr) {
    used[v] = true;
    depth[v] = d;
    up[v] = d;
    for (Edge* e : graph[v]) {
        if (!used[e->u]) {
            dfs(e->u, d + 1, e);
            up[v] = min(up[v], up[e->u]);
        }
        else if (p && p->back != e) {
            up[v] = min(up[v], depth[e->u]);
        }
    }
    if (p && up[v] == depth[v]) {
        p->isbridge = true;
        p->back->isbridge = true;
        ans.push_back(p->index + 1);
    }
}

signed main() {
    int n, m;
    cin >> n >> m;
    graph.resize(n);
    depth.resize(n);
    used.resize(n);
    up.resize(n);
    for (int i = 0; i < m; i++) {
        int v, u;
        cin >> v >> u;
        v--;
        u--;
        Edge* e1 = new Edge(u, i);
        Edge* e2 = new Edge(v, i);
        e1->back = e2;
        e2->back = e1;
        graph[v].push_back(e1);
        graph[u].push_back(e2);
    };
    for (int v = 0; v < n; v++) {
        if (!used[v]) {
            dfs(v);
        }
    }
    cout << ans.size() << el;
    cout << ans << el;
    return 0;
}
```

Точка сочленения - вершина, при удалении которой граф распадается (кол-во компонент связности увеличивается)
Поиск точек сочленения похож на поиск мостов, однако есть отличия
Во-первых, если хотя бы у одного сына нет обратного ребра, то текущая вершина является точкой сочленения
Во-вторых, нужно отдельно проверить корень обхода: если у корня больше 1 сына, то он точка сочленения

```cpp
vector<vector<int>> graph;
vector<bool> used;
vector<int> depth;
vector<int> up;
vector<bool> is_cutpoint;

void dfs(int v, int d = 0, int p = -1) {
    used[v] = true;
    depth[v] = d;
    int children = 0;
    up[v] = d;
    for (int u : graph[v]) {
        if (!used[u]) {
            dfs(u, d + 1, v);
            children++;
            up[v] = min(up[v], up[u]);
            if (p != -1 && up[u] >= depth[v]) {
                is_cutpoint[v] = true;
            }
        }
        else if (p != u) {
            up[v] = min(up[v], depth[u]);
        }
    }
    if (p == -1 && children > 1) {
        is_cutpoint[v] = true;
    }
}

signed main() {
    int n, m;
    cin >> n >> m;
    graph.resize(n);
    depth.resize(n);
    used.resize(n);
    up.resize(n);
    is_cutpoint.resize(n);
    for (int i = 0; i < m; i++) {
        int v, u;
        cin >> v >> u;
        v--;
        u--;
        graph[v].push_back(u);
        graph[u].push_back(v);
    };
    for (int v = 0; v < n; v++) {
        if (!used[v]) {
            dfs(v);
        }
    }
    vector<int> ans;
    for (int i = 0; i < n; i++) {
        if (is_cutpoint[i]) {
            ans.push_back(i + 1);
        }
    }
    cout << ans.size() << el;
    cout << ans << el;
    return 0;
}
```

С мостами и точками сочленения связаны компоненты двусвязности
Две вершины рёберно двусвязны, если существуют хотя бы два непересекающихся по рёбрам пути между ними
Два ребра вершинно двусвязны, если существуют хотя бы два непересекающихся по вершинам пути, соединяющие концы этих рёбер

Компоненты рёберной двусвязности ищутся довольно просто
Можно просто запустить dfs и не ходить по мостам
Компоненты можно сжимать в вершины (полученный граф - дерево)

Сжатие компонент рёберной двусвязности:
```cpp
vector<vector<int>> graph2;
vector<int> comp;

void dfs_comp(int v, int cur_comp) {
    used[v] = true;
    comp[v] = cur_comp;
    for (Edge* e : graph[v]) {
        if (!used[e->u] && !e->isbridge) {
            dfs_comp(e->u, cur_comp);
        }
        else if (used[e->u] && e->isbridge) {
            graph2[cur_comp].push_back(comp[e->u]);
            graph2[comp[e->u]].push_back(cur_comp);
        }
    }
}

signed main() {
    // ввод, нахождение мостов
    used.assign(n, false);
    int cur_comp = 0;
    for (int v = 0; v < n; v++) {
        if (!used[v]) {
            graph2.push_back(vector<int>());
            dfs_comp(v, cur_comp);
            cur_comp++;
        }
    }
}
```

Нахождение компонент вершинной двусвязности:
```cpp
struct Edge {
    int u;
    int index;
    Edge() {}
    Edge(int u_, int index_) {
        u = u_;
        index = index_;
    }
};

vector<vector<Edge>> graph;
vector<bool> used;
vector<int> depth;
vector<int> up;
vector<int> col;
int maxcol = 0;

void dfs(int v, int d = 0, int p = -1) {
    used[v] = true;
    depth[v] = d;
    up[v] = d;
    for (Edge e : graph[v]) {
        int u = e.u;
        if (!used[u]) {
            dfs(u, d + 1, v);
            up[v] = min(up[v], up[u]);
        }
        else if (p != u) {
            up[v] = min(up[v], depth[u]);
        }
    }
}

void dfs_comp(int v, int color, int e_index) {
    used[v] = true;
    for (Edge e : graph[v]) {
        int u = e.u;
        if (e.index == e_index) {
            continue;
        }
        if (!used[u]) {
            if (up[u] >= depth[v]) {
                maxcol++;
                col[e.index] = maxcol;
                dfs_comp(u, maxcol, e.index);
            }
            else {
                col[e.index] = color;
                dfs_comp(u, color, e.index);
            }
        }
        else {
            if (depth[u] < depth[v]) {
                col[e.index] = color;
            }
        }
    }
}

signed main() {
    int n, m;
    cin >> n >> m;
    graph.resize(n);
    depth.resize(n);
    used.resize(n);
    up.resize(n);
    col.resize(m);
    for (int i = 0; i < m; i++) {
        int v, u;
        cin >> v >> u;
        v--;
        u--;
        graph[v].push_back(Edge(u, i));
        graph[u].push_back(Edge(v, i));
    };
    for (int v = 0; v < n; v++) {
        if (!used[v]) {
            dfs(v);
        }
    }
    used.assign(n, false);
    for (int i = 0; i < n; i++) {
        if (!used[i]) {
            dfs_comp(i, maxcol, -1);
        }
    }
    cout << maxcol << el;
    cout << col << el;
    return 0;
}
```