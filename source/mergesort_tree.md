# mergesort_tree

Дерево mergesort

Дерево mergesort - ДО, в каждой вершине которого хранится отсортированный подмассив

Такая структура позволяет решать задачу о нахождении кол-во числе в диапазоне [x, y] на отрезке [l, r]
Если есть отсортированный массив, то такая задача решается двумя бинпоисками

Вершины дерева сливаются, как в сортировке слиянием

Код:
```cpp
struct Node {
    vector<int> a;
    Node() {}
};

vector<Node> t;
vector<int> in;

void build(int v, int l, int r) {
    if (r - l == 1) {
        t[v].a.push_back(in[l]);
        return;
    }
    int m = (l + r) / 2;
    build(v * 2 + 1, l, m);
    build(v * 2 + 2, m, r);
    t[v].a.resize(t[v * 2 + 1].a.size() + t[v * 2 + 2].a.size());
    merge(t[v * 2 + 1].a.begin(), t[v * 2 + 1].a.end(), t[v * 2 + 2].a.begin(), t[v * 2 + 2].a.end(), t[v].a.begin());
}

int ask(int v, int l, int r, int askl, int askr, int x, int y) {
    if (l >= askr || r <= askl) {
        return 0;
    }
    if (l >= askl && r <= askr) {
        int first = lower_bound(t[v].a.begin(), t[v].a.end(), x) - t[v].a.begin();
        int last = upper_bound(t[v].a.begin(), t[v].a.end(), y) - t[v].a.begin();
        return last - first;
    }
    int m = (l + r) / 2;
    return ask(v * 2 + 1, l, m, askl, askr, x, y) + ask(v * 2 + 2, m, r, askl, askr, x, y);
}

signed main() {
    int n, m;
    cin >> n >> m;
    in.resize(n);
    t.resize(4 * n);
    cin >> in;
    build(0, 0, n);
    REP(m) {
        int l, r, x, y;
        cin >> l >> r >> x >> y;
        cout << ask(0, 0, n, l - 1, r, x, y) << el;
    }
    return 0;
}
```

Если также поступают запросы об изменении элемента, то в каждой вершине нужно хранить не массив, а декартово дерево
Это декартово дерево должно поддерживать split как по ключу, так и по кол-ву отрезаемых элементов (чтобы изменить значение одного элемента)

При помощи дерева mergesort можно решить задачу о нахождении кол-во различных чисел на отрезке массива
Для этого сначала нужно создать массив Prev
Prev[i] - такой индекс j, что i < j (j максимально), и a[i] = a[j] (если это первое вхождение, то Prev[i] = -1)
Заметим, что кол-во различных чисел на отрезке будет равно кол-ву значений Prev[i] (l <= i <= r), которые меньше, чем l (до таких элементов в отрезке нет таких же)
На такие запросы можно отвечать, построив дерево mergesort по массиву Prev
```cpp
struct Node {
    vector<int> a;
    Node() {}
};

vector<int> Prev;
vector<Node> t;

void build(int v, int l, int r) {
    if (r - l == 1) {
        t[v].a = {Prev[l]};
        return;
    }
    int m = (l + r) / 2;
    build(v * 2 + 1, l, m);
    build(v * 2 + 2, m, r);
    t[v].a.resize(t[v * 2 + 1].a.size() + t[v * 2 + 2].a.size());
    merge(t[v * 2 + 1].a.begin(), t[v * 2 + 1].a.end(), t[v * 2 + 2].a.begin(), t[v * 2 + 2].a.end(), t[v].a.begin());
}

int ask(int v, int l, int r, int askl, int askr) {
    if (l >= askr || r <= askl) {
        return 0;
    }
    if (l >= askl && r <= askr) {
        int firstNo = lower_bound(t[v].a.begin(), t[v].a.end(), askl) - t[v].a.begin();
        return firstNo;
    }
    int m = (l + r) / 2;
    return ask(v * 2 + 1, l, m, askl, askr) + ask(v * 2 + 2, m, r, askl, askr);
}

signed main() {
    fastIO;
    int n;
    cin >> n;
    vector<int> in(n);
    t.resize(4 * n);
    Prev.resize(n, -1);
    cin >> in;
    unordered_map<int, int> d;
    for (int i = 0; i < n; i++) {
        if (d[in[i]] == 0) {
            Prev[i] = -1;
        }
        else {
            Prev[i] = d[in[i]] - 1;
        }
        d[in[i]] = i + 1;
    }
    build(0, 0, n);
    int m;
    cin >> m;
    REP(m) {
        int l, r;
        cin >> l >> r;
        cout << ask(0, 0, n, l - 1, r) << el;
    }
    return 0;
}
```

Быстрее эта задача решается при помощи алгоритма Мо (см. Mo)