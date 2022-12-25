# Ford_Bellman

Алгоритм Форда-Беллмана

Алгоритм Форда-Беллмана ищет кратчайшие расстояния от одной вершины до всех остальных
Он похож на алгоритм Дейкстры, но единицей выступает не вершина, а ребро

За одну итерацию алгоритма алгоритм ищет кратчайшие пути длины i + 1 (в рёбрах)
Сам алгоритм заключается в переборе всех рёбер и релаксации в процессе перебора

```cpp
const int INF = 1e9;

struct Edge {
    int v, u, w;
    Edge() {}
    Edge(int v_, int u_, int w_): v(v_), u(u_), w(w_) {}
};

signed main() {
    int n, m;
    cin >> n >> m;
    vector<Edge> edges;
    for (int i = 0; i < m; i++) {
        int v, u, w;
        cin >> v >> u >> w;
        u--;
        v--;
        edges.push_back(Edge(v, u, w));
    }
    vector<int> d(n, INF);
    d[0] = 0;
    for (int i = 0; i < n - 1; i++) {
        for (auto &e: edges) {
            if (d[e.v] != INF && d[e.u] > d[e.v] + e.w) {
                d[e.u] = d[e.v] + e.w;
            }
        }
    }
    return 0;
}
```
В массиве d в конце будут лежать расстояния до всех вершин

Алгоритм позволяет искать кратчайшие пути, состоящие из ровно k рёбер, от одной вершины до остальных
Для этого нужно сделать у массива d второе измерение - кол-во рёбер в пути (ведь кратчайший путь может состоять и из одного ребра, но нам нужно ровно k рёбер)

```cpp
const int INF = 1e9;

struct Edge {
    int v, u;
    int w;
    Edge() {}
    Edge(int v_, int u_, int w_) {
        v = v_;
        u = u_;
        w = w_;
    }
};

signed main() {
    int n, m, k, start;
    cin >> n >> m >> k >> start;
    start--;
    vector<Edge> edges(m);
    for (int i = 0; i < m; i++) {
        int v, u, w;
        cin >> v >> u >> w;
        v--;
        u--;
        edges[i] = {v, u, w};
    }
    vector<vector<int>> d(n, vector<int>(k + 1, INF));
    d[start][0] = 0;
    for (int i = 0; i < k; i++) {
        for (Edge& e : edges) {
            for (int path = 1; path <= k; path++) {
                if (d[e.v][path - 1] != INF && d[e.u][path] > d[e.v][path - 1] + e.w) {
                    d[e.u][path] = d[e.v][path - 1] + e.w;
                }
            }
        }
    }
    for (int i = 0; i < n; i++) {
        if (d[i][k] != INF) {
            cout << d[i][k] << el;
        }
        else {
            cout << "-1\n";
        }
    }
    return 0;
}
```

С помощью алгоритма Форда-Беллмана можно искать отрицательный цикл в графе
Для этого достаточно лишь запустить алгоритм два раза и посмотреть, изменился ли массив расстояний
Если расстояние для какой-то вершины изменилось, то она достижима из цикла отрицательного веса (или лежит на нём)
Чтобы восстановить цикл, нужно хранить для каждой вершины, откуда мы в неё пришли, чтобы создать кратчайший путь
Тогда будем откатываться по предкам от вершины, для которой изменилось расстояние, и класть вершины в стек
Когда мы находим вершину, которая уже была в стеке, мы начинаем выбрасывать вершины из стека, пока не дойдём до первого вхождения вершины
Все вершины, которые мы выбросим, - отрицательный цикл
```cpp
const ll INF = 1e9;

struct Edge {
    int v, u, w;
    Edge() {}
    Edge(int v_, int u_, int w_) {
        v = v_;
        u = u_;
        w = w_;
    }
};

signed main() {
    int n, m;
    cin >> n >> m;
    vector<Edge> edges(m);
    for (int i = 0; i < m; i++) {
        int v, u, w;
        cin >> v >> u >> w;
        v--;
        u--;
        edges[i] = Edge(v, u, w);
    }
    vector<int> d(n, INF);
    d[0] = 0;
    vector<int> p(n, -1);
    for (int i = 0; i < n - 1; i++) {
        for (Edge& e : edges) {
            if (d[e.v] != INF && d[e.u] > d[e.v] + e.w) {
                d[e.u] = d[e.v] + e.w;
                p[e.u] = e.v;
            }
        }
    }
    vector<int> old_d = d;
    for (int i = 0; i < n - 1; i++) {
        for (Edge& e : edges) {
            if (d[e.v] != INF && d[e.u] > d[e.v] + e.w) {
                d[e.u] = d[e.v] + e.w;
                p[e.u] = e.v;
            }
        }
    }
    if (old_d == d) {
        cout << "NO\n";
        return 0;
    }
    cout << "YES\n";
    int cur = -1;
    for (int i = 0; i < n; i++) {
        if (old_d[i] != d[i]) {
            cur = i;
            break;
        }
    }
    stack<int> st;
    st.push(cur);
    vector<bool> been(n);
    vector<int> cycle;
    been[cur] = true;
    while (true) {
        cur = p[cur];
        if (been[cur]) {
            cycle.push_back(cur + 1);
            while (st.top() != cur) {
                cycle.push_back(st.top() + 1);
                st.pop();
            }
            cycle.push_back(cur + 1);
            break;
        }
        been[cur] = true;
        st.push(cur);
    }
    cout << cycle.size() << el;
    cout << cycle << el;
    return 0;
}
```

Но может быть случай, когда отрицательный цикл не достижим из вершины 0
Тогда можно запускать из всех непосещённых вершин (у которых расстояние INF) (но это довольно долго)

Можно проверить, определено ли значение кратчайшего пути для вершины
Для этого нужно добавить после алгоритма:
```cpp
int finish;
cin >> finish;
bool reach = false;
vector<bool> used(n);
dfs(finish, used, rg, d, old, reach);
if (reach) {
    cout << "-INF\n";
}
```

rg - перевёрнутый граф (можно создать в процессе ввода)
Функция dfs:
```cpp
void dfs(int v, vector<bool>& used, const vector<vector<int>>& g, const vector<int>& d, const vector<int>& old, bool& reach) {
    if (d[v] != old[v]) {
        reach = true;
    }
    used[v] = true;
    for (int u: g[v]) {
        dfs(u, used, g, d, old, reach);
    }
}
```
