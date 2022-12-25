# segment_tree

Дерево отрезков

Дерево отрезков - структура данных, хранящая значения функций для всех отрезков массива
Листья - элементы массива, уровень выше - объединение листьев (непересекающиеся отрезки массива длины 2) и т.д.
Дерево отрезков удобно хранить в массиве: сыновья вершины v - v * 2 + 1 и v * 2 + 2

Все примеры кода используют дерево отрезков НА ПОЛУИНТЕРВАЛАХ
Дерево для поиска позиции максимума на отрезке с изменением элементов
В самом дереве хранится индекс, для поиска максимума сделана функция Max
Для нейтрального элемента к массиву добавлено самое маленькое значение

```cpp
int n;
vector<int> a, t;

int Max(int i, int j) {
    if (a[i] > a[j])
        return i;
    return j;
}

void build(int v, int l, int r) {
    if (r - l == 1) {
        t[v] = l;
        return;
    }
    int m = (l + r) / 2;
    build(v * 2 + 1, l, m);
    build(v * 2 + 2, m, r);
    t[v] = Max(t[v * 2 + 1], t[v * 2 + 2]);
}

int ask(int v, int l, int r, int askl, int askr) {
    if (l >= askr || r <= askl) {
        return n; // neutral element
    }
    if (l >= askl && r <= askr) {
        return t[v];
    }
    int m = (l + r) / 2;
    return Max(ask(2 * v + 1, l, m, askl, askr), ask(2 * v + 2, m, r, askl, askr));
}

void change(int v, int l, int r, int pos, int val) {
    if (r - l == 1) {
        a[l] = val;
        t[v] = val;
        return;
    }
    int m = (l + r) / 2;
    if (pos < m) {
        change(2 * v + 1, l, m, pos, val);
    }
    else {
        change(2 * v + 2, m, r, pos, val);
    }
    t[v] = Max(t[v * 2 + 1], t[v * 2 + 2]);
}

signed main() {
    cin >> n;
    a.resize(n);
    t.resize(4 * n);
    cin >> a;
    a.push_back(-INT_MAX);
    build(0, 0, n);

    int q;
    cin >> q;
    while (q--) {
        int l, r;
        cin >> l >> r;
        int ind = ask(0, 0, n, l - 1, r);
        cout << a[ind] << ' ' << ind + 1 << el;
    }
    return 0;
}
```

Функция ask проверяет цвет вершины:
жёлтая - пересечение с отрезком запроса
зелёная - текущий отрезок входит в отрезок запроса
красная - нет пересечения с отрезком запроса

Из зелёной вершины всегда идут зелёные, из красной - красные
Первый if - проверка на красную, второй - на зелёную

При сумме нейтральный элемент - 0

Универсальное дерево - это вершина Node() и функция unite()
В универсальном дереве меняется только Node() и unite()
Пример (кол-во максимумов на отрезке):

```cpp
struct Node {
    int maxx, cnt;
    Node(): maxx(-INT_MAX), cnt(0) {}
};

int n;
vector<int> a;
vector<Node> t;

Node unite(const Node& a, const Node& b) {
    Node res;
    if (a.maxx > b.maxx) {
        res = a;
    }
    else if (a.maxx < b.maxx) {
        res = b;
    }
    else {
        res.maxx = a.maxx;
        res.cnt = a.cnt + b.cnt;
    }
    return res;
}

void build(int v, int l, int r) {
    if (r - l == 1) {
        t[v].maxx = a[l];
        t[v].cnt = 1;
        return;
    }
    int m = (l + r) / 2;
    build(v * 2 + 1, l, m);
    build(v * 2 + 2, m, r);
    t[v] = unite(t[v * 2 + 1], t[v * 2 + 2]);
}

Node ask(int v, int l, int r, int askl, int askr) {
    if (l >= askr || r <= askl) {
        return Node();
    }
    if (l >= askl && r <= askr) {
        return t[v];
    }
    int m = (l + r) / 2;
    return unite(ask(2 * v + 1, l, m, askl, askr), ask(2 * v + 2, m, r, askl, askr));
}

void change(int v, int l, int r, int pos, int val) {
    if (r - l == 1) {
        t[v].maxx = val;
        t[v].cnt = 1;
        return;
    }
    int m = (l + r) / 2;
    if (pos < m) {
        change(2 * v + 1, l, m, pos, val);
    }
    else {
        change(2 * v + 2, m, r, pos, val);
    }
    t[v] = unite(t[v * 2 + 1], t[v * 2 + 2]);
}
```

Node и unite для поиска наибольшей суммы подотрезка на отрезке:
```cpp
Node unite(const Node& a, const Node& b) {
    Node res;
    res.summ = a.summ + b.summ;
    res.maxpref = max(a.maxpref, a.summ + b.maxpref);
    res.maxsuff = max(b.maxsuff, b.summ + a.maxsuff);
    res.maxsum = max({a.maxsum, b.maxsum, a.maxsuff + b.maxpref});
    return res;
}
struct Node {
    int maxsum, maxpref, maxsuff, summ;
    Node(): maxsum(0), maxpref(0), maxsuff(0), summ(0) {}
    Node(int x): maxsum(x), maxpref(x), maxsuff(x), summ(x) {}

};
```

МАССОВЫЕ ОПЕРАЦИИ
Массовые операции - операции прибавления и присвоения на отрезке
Для массовых операций используются отложенные операции (выполняем операцию, когда понадобится соответствующий отрезок)
Реализация прибавления и присвоения различны

Массовое прибавление (для суммы):\

```cpp
void push(int v) {
    c[v * 2 + 1] += c[v];
    c[v * 2 + 2] += c[v];
    c[v] = 0;
}
int ask(int v, int l, int r, int askl, int askr) {
    if (l >= askr || r <= askl) {
        return 0;
    }
    if (l >= askl && r <= askr) {
        return t[v] + c[v] * (r - l);
        // for min: return t[v] + c[v];
    }
    push(v);
    int m = (l + r) / 2;
    int x = ask(2 * v + 1, l, m, askl, askr);
    int y = ask(2 * v + 2, m, r, askl, askr);
    t[v] = t[v * 2 + 1] + t[v * 2 + 2] + c[v * 2 + 1] * (m - l) + c[v * 2 + 2] * (r - m);
    return x + y;
}
void change(int v, int l, int r, int askl, int askr, int val) {
    if (l >= askr || r <= askl) {
        return;
    }
    if (l >= askl && r <= askr) {
        c[v] += val;
        return;
    }
    push(v);
    int m = (l + r) / 2;
    change(v * 2 + 1, l, m, askl, askr, val);
    change(v * 2 + 2, m, r, askl, askr, val);
    t[v] = t[v * 2 + 1] + t[v * 2 + 2] + c[v * 2 + 1] * (m - l) + c[v * 2 + 2] * (r - m);
    // for min: t[v] = min(t[v * 2 + 1] + c[v * 2 + 1], t[v * 2 + 2] + c[v * 2 + 2]);
}
```

Для этой операции понадобится массив c (размер 4*n), в котором будут хранится значения, которые нужно прибавить
Также используется операция проталкивания (push), которая заносит прибавление в вершине в её сыновей
build не меняется

Массовое присвоение (для суммы):

```cpp
void push(int v) {
    if (isch[v]) {
        c[v * 2 + 1] = c[v];
        c[v * 2 + 2] = c[v];
        isch[v * 2 + 1] = true;
        isch[v * 2 + 2] = true;
        isch[v] = false;
    }
}
int ask(int v, int l, int r, int askl, int askr) {
    if (l >= askr || r <= askl) {
        return 0;
    }
    if (l >= askl && r <= askr) {
        return (isch[v] ? c[v] * (r - l) : t[v]);
    }
    push(v);
    int m = (l + r) / 2;
    int x = ask(2 * v + 1, l, m, askl, askr);
    int y = ask(2 * v + 2, m, r, askl, askr);
    t[v] = t[v * 2 + 1] + t[v * 2 + 2] + c[v * 2 + 1] * (m - l) + c[v * 2 + 2] * (r - m);
    return x + y;
}
void change(int v, int l, int r, int askl, int askr, int val) {
    if (l >= askr || r <= askl) {
        return;
    }
    if (l >= askl && r <= askr) {
        c[v] = val;
        isch[v] = true;
        return;
    }
    push(v);
    int m = (l + r) / 2;
    change(v * 2 + 1, l, m, askl, askr, val);
    change(v * 2 + 2, m, r, askl, askr, val);
    t[v] = (isch[v * 2 + 1] ? c[v * 2 + 1] * (m - l) : t[v * 2 + 1]) + (isch[v * 2 + 2] ? c[v * 2 + 2] * (r - m) : t[v * 2 + 2]);
    // for min: min((isch[v * 2 + 1] ? c[v * 2 + 1] : t[v * 2 + 1]), (isch[v * 2 + 2] ? c[v * 2 + 2] : t[v * 2 + 2]));
}
```

Для присвоения добавляется массив bool isch, который хранит true, если было изменение в данной вершине

Используя дерево отрезков, можно построить дерево хешей и делать операции с хешами, имея возможность изменять элементы строки
Получение хеша подстроки с изменением элемента (в коде в функции передаются дерево и строка, потому что часто строк несколько, нужны обратные хеши и т.д., поэтому нужны несколько деревьев):

```cpp
ll unite(ll h1, ll h2, int len2) {
    return (h1 * powers[len2] + h2) % MOD;
}
void build(vector<ll>& tree, const string& str, int v, int l, int r) {
    if (r - l == 1) {
        tree[v] = str[l];
        return;
    }
    int m = (l + r) / 2;
    build(tree, str, v * 2 + 1, l, m);
    build(tree, str, v * 2 + 2, m, r);
    tree[v] = unite(tree[v * 2 + 1], tree[v * 2 + 2], r - m);
}
void change(vector<ll>& tree, int v, int l, int r, int pos, char val) {
    if (r - l == 1) {
        tree[v] = val;
        return;
    }
    int m = (l + r) / 2;
    if (pos < m) {
        change(tree, v * 2 + 1, l, m, pos, val);
    }
    else {
        change(tree, v * 2 + 2, m, r, pos, val);
    }
    tree[v] = unite(tree[v * 2 + 1], tree[v * 2 + 2], r - m);
}
void ask(const vector<ll>& tree, vector<pair<ll, int>>& Nodes, int v, int l, int r, int askl, int askr) {
    if (l >= askr || r <= askl) {
        return;
    }
    if (l >= askl && r <= askr) {
        Nodes.push_back({tree[v], r - l});
        return;
    }
    int m = (l + r) / 2;
    ask(tree, Nodes, v * 2 + 1, l, m, askl, askr);
    ask(tree, Nodes, v * 2 + 2, m, r, askl, askr);
}
```

ask не возвращает хеш, а складывает ответ в массив
Это нужно, чтобы получить конечный ответ
Получить его можно, просто сделав несколько unite по массиву Nodes (ведь дерево отрезков положит хеши в массив в правильном порядке)