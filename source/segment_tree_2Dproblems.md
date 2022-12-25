# segment_tree_2Dproblems

Двумерные задачи на дерево отрезков

Один из метод решения двумерных задач - сканлайн + ДО
Сканлайн идёт по оси OX, а ДО по оси OY

Пример задачи, решаемой таким способом - точка, покрытая максимальным кол-вом прямоугольников
Есть несколько вариаций этой задачи. Код ниже решает задачу, в которой вводятся только прямоугольники
```cpp
template <class T> void uniq(vector<T>& a) {sort(a.begin(), a.end()); a.resize(unique(a.begin(), a.end()) - a.begin());}

struct Val {
    ll Max;
    int index;
    Val() {
        Max = 0;
        index = -1;
    }
    Val(ll Max_, int index_) {
        Max = Max_;
        index = index_;
    }
};

Val unite(const Val a, const Val b) {
    if (a.Max < b.Max) {
        return b;
    }
    if (a.Max > b.Max) {
        return a;
    }
    if (a.index < b.index) {
        return b;
    }
    return a;
}

vector<Val> t;
vector<ll> c;

void push(int v) {
    c[v * 2 + 1] += c[v];
    c[v * 2 + 2] += c[v];
    c[v] = 0;
}

Val add_c(Val a, ll add) {
    a.Max += add;
    return a;
}

void build(int v, int l, int r) {
    if (r - l == 1) {
        t[v] = Val(0, l);
        return;
    }
    int m = (l + r) / 2;
    build(v * 2 + 1, l, m);
    build(v * 2 + 2, m, r);
    t[v] = unite(t[v * 2 + 1], t[v * 2 + 2]);
}

void change(int v, int l, int r, int askl, int askr, int delta) {
    if (l >= askr || r <= askl) {
        return;
    }
    if (l >= askl && r <= askr) {
        c[v] += delta;
        return;
    }
    push(v);
    int m = (l + r) / 2;
    change(v * 2 + 1, l, m, askl, askr, delta);
    change(v * 2 + 2, m, r, askl, askr, delta);
    t[v] = unite(add_c(t[v * 2 + 1], c[v * 2 + 1]), add_c(t[v * 2 + 2], c[v * 2 + 2]));
}

Val ask(int v, int l, int r, int askl, int askr) {
    if (l >= askr || r <= askl) {
        return Val(-1e9, -1);
    }
    if (l >= askl && r <= askr) {
        return add_c(t[v], c[v]);
    }
    push(v);
    int m = (l + r) / 2;
    Val x = ask(v * 2 + 1, l, m, askl, askr);
    Val y = ask(v * 2 + 2, m, r, askl, askr);
    t[v] = unite(add_c(t[v * 2 + 1], c[v * 2 + 1]), add_c(t[v * 2 + 2], c[v * 2 + 2]));
    return unite(x, y);
}

struct Event {
    int type;
    int x;
    int y_up;
    int y_down;
    Event() {}
    Event(int type_, int x_, int y_up_, int y_down_) {
        type = type_;
        x = x_;
        y_up = y_up_;
        y_down = y_down_;
    }
};

bool operator < (const Event& a, const Event& b) {
    if (a.x == b.x) {
        return a.type < b.type;
    }
    return a.x < b.x;
}

signed main() {
    int n;
    cin >> n;
    vector<Event> evs(n * 2);
    vector<int> Y;
    for (int i = 0; i < n; i++) {
        int x1, y1, x2, y2;
        cin >> x1 >> y1 >> x2 >> y2;
        Y.push_back(y1);
        Y.push_back(y2);
        evs[i * 2] = Event(0, x1, y1, y2);
        evs[i * 2 + 1] = Event(1, x2, y1, y2);
    }
    uniq(Y);
    sort(all(evs));
    int nY = Y.size();
    t.resize(4 * nY);
    c.resize(4 * nY);
    build(0, 0, nY);
    ll ans = -1;
    int x_ans = -1, y_ans = -1;
    for (int i = 0; i < n * 2; i++) {
        Event e = evs[i];
        int y1 = lower_bound(all(Y), e.y_up) - Y.begin();
        int y2 = lower_bound(all(Y), e.y_down) - Y.begin();
        if (e.type == 0) {
            change(0, 0, nY, y1, y2 + 1, 1);
            Val res = ask(0, 0, nY, y1, y2 + 1);
            if (res.Max > ans) {
                ans = res.Max;
                x_ans = e.x;
                y_ans = Y[res.index];
            }
        }
        else {
            change(0, 0, nY, y1, y2 + 1, -1);
        }
    }
    cout << ans << el << x_ans << ' ' << y_ans << el;
    return 0;
}
```

Как работает этот код: когда открывается прямоугольник, в ДО на отрезке [Y_up, Y_down] прибавляется 1
Когда прямоугольник закрывается, на отрезке происходит вычитание 1
Чтобы найти ответ, можно после обработки события прибавления делать запрос максимума на отрезке, который только что был обработан
(наверное, можно и лучше)
В коде используется сжатие координат (хранятся только события x и y событий)

ДВУМЕРНОЕ ДО
ДО может быть многомерным
Для этого в каждой вершине ДО нужно заводить ещё одно ДО
Например, для двумерного случая внешнее ДО будет считать ответ по OX (вершина отвечает за несколько строк), а внутреннее ДО по OY (вершина отвечает за несколько столбцов)


# pref_sum

Префиксные суммы

Префиксные суммы позволяют считать сумму (и не только) на всех отрезках (и не только)

Создаётся массив, в котором записаны суммы на всех префиксах (по увеличению длины)
Если поступает запрос на отрезок с l по r (0-индексация), то
```cpp
if (l == 0) {
    ans = pref[r];
}
else {
    ans = pref[r] - pref[l - 1];
}
```

Префиксные суммы могут быть n-мерными
Двумерные префиксные суммы:
```cpp
vector<vector<ll>> pref(n, vector<ll>(m));
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        pref[i][j] = a[i][j];
        if (i > 0) {
            pref[i][j] += pref[i - 1][j];
        }
        if (j > 0) {
            pref[i][j] += pref[i][j - 1];
        }
        if (i > 0 && j > 0) {
            pref[i][j] -= pref[i - 1][j - 1];
        }
    }
}
int l1, r1, l2, r2;
cin >> l1 >> r1 >> l2 >> r2;
ll ans = pref[r1][r2];
if (l1 > 0) {
    ans -= pref[l1 - 1][r2];
}
if (l2 > 0) {
    ans -= pref[r1][l2 - 1];
}
if (l1 > 0 && l2 > 0) {
    ans += pref[l1 - 1][l2 - 1];
}
```