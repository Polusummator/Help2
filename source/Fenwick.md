# Fenwick

Дерево Фенвика (Bit Indexed Tree)

Дерево Фенвика - структура, которая позволяет решать задачу о сумме на отрезке с изменением элемента
Дерево Фенвика пишется быстрее, чем ДО, и занимает меньше памяти

Запрос суммы и запрос изменения работает за O(logn)

Каждый элемент массива дерева хранит сумму на каком-то отрезке (этот отрезок заканчивается в этом элементе)
Ответ на запрос получается из разности сумм на двух префиксах
Сумма на префиксе получается из сумм на нескольких отрезках

Переход от текущего конца отрезка к предыдущему - i & (i + 1) - 1
Переход от текущего конца отрезка к следующему - i | (i + 1)

Изменение элемента - это изменение всех отрезков, в которых лежит этот элемент
Важно, что изменение - это не операция присваивания, а операция прибавления (можно прибавлять разность нового и старого значения)

Дерево Фенвика можно строить за O(nlogn) (просто изменить каждый элемент) и за O(n)
Для построения за O(n) нужно пройти по массиву с начала и прибавить к концам отрезков значения

Код:
```cpp
int n;
vector<ll> t;

void change(int i, ll delta) {
    for (int pos = i; pos < n; pos = pos | (pos + 1)) {
        t[pos] += delta;
    }
}

ll ask(int i) {
    ll res = 0;
    for (int pos = i; pos >= 0; pos = (pos & (pos + 1)) - 1) {
        res += t[pos];
    }
    return res;
}

ll ask(int l, int r) {
    return ask(r) - (l == 0 ? 0 : ask(l - 1));
}

void build(const vector<ll>& a) {
    t = a;
    for (int i = 0; i < n; i++) {
        int j = i | (i + 1);
        if (j < n) {
            t[j] += t[i];
        }
    }
}

signed main() {
    cin >> n;
    vector<ll> a(n);
    cin >> a;
    build(a);
    int q;
    cin >> q;
    REP(q) {
        char mode;
        cin >> mode;
        if (mode == 's') {
            int l, r;
            cin >> l >> r;
            cout << ask(l - 1, r - 1) << ' ';
        }
        else {
            int pos;
            ll nval;
            cin >> pos >> nval;
            change(pos - 1, nval - a[pos - 1]);
            a[pos - 1] = nval;
        }
    }
    return 0;
}
```

Дерево Фенвика обобщается на многомерные случаи
Двумерное дерево Фенвика (поиск суммы на прямоугольнике с изменением элемента):
```cpp
int n, m;
vector<vector<ll>> t;

ll ask(int x, int y) {
    ll res = 0;
    for (int i = x; i >= 0; i = (i & (i + 1)) - 1) {
        for (int j = y; j >= 0; j = (j & (j + 1)) - 1) {
            res += t[i][j];
        }
    }
    return res;
}

ll ask(int x1, int y1, int x2, int y2) {
    ll res = ask(x2, y2);
    if (x1 > 0) {
        res -= ask(x1 - 1, y2);
    }
    if (y1 > 0) {
        res -= ask(x2, y1 - 1);
    }
    if (x1 > 0 && y1 > 0) {
        res += ask(x1 - 1, y1 - 1);
    }
    return res;
}

void change(int x, int y, ll delta) {
    for (int i = x; i < n; i = (i | (i + 1))) {
        for (int j = y; j < m; j = (j | (j + 1))) {
            t[i][j] += delta;
        }
    }
}

void build(const vector<vector<ll>>& a) {
    t.resize(n, vector<ll>(m, 0));
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            change(i, j, a[i][j]);
        }
    }
}

signed main() {
    fastIO;
    cin >> n >> m;
    vector<vector<ll>> a(n, vector<ll>(m));
    cin >> a;
    build(a);
    int q;
    cin >> q;
    REP(q) {
        char mode;
        cin >> mode;
        if (mode == 's') {
            int x1, y1, x2, y2;
            cin >> x1 >> y1 >> x2 >> y2;
            cout << ask(x1 - 1, y1 - 1, x2 - 1, y2 - 1) << el;
        }
        else {
            int x, y;
            ll nval;
            cin >> x >> y >> nval;
            change(x - 1, y - 1, nval - a[x - 1][y - 1]);
            a[x - 1][y - 1] = nval;
        }
    }
    return 0;
}
```