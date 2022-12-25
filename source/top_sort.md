# top_sort

Топологическая сортировка

Топологическая сортировка применяется на ориентированных графах
Результат сортировки - массив вершин, в котором выполняется условие "рёбра идут только вправо" (рёбра идут от вершин в начале к вершинам в конце)
Сортировка невозможна в графе с циклами

Сортировка пишется с использованием dfs
Код:
```cpp
int n;
vector<bool> used;
vector<int> ts;

void dfs_ts(int v) {
    used[v] = true;
    for (int u : g[v]) {
        if (!used[u])
            dfs_ts(u);
    }
    ts.push_back(v);

void top_sort() {
    used.assign(n, false);
    for (int v = 0; v < n; v++)
        if (!used[v])
            dfs_ts(v);
    reverse(ts.begin(), ts.end());
```