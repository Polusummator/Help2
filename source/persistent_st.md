# persistent_st

Персистентное дерево отрезков

Персистентное ДО сохраняет все свои версии (версии появляются в процессе запросов изменения)

Чтобы сохранять версии, при запросе изменения нужно менять не сами вершины, а их копии
Для этого ДО будет построено на указателях
Когда мы заходим в вершину в запросе изменения, мы сначала копируем её, а потом делаем изменение в копии
Сам запрос будет возвращать указатель - корень новой версии дерева
Чтобы иметь доступ ко всем версиям дерева, можно класть все корни в массив

Пример (персистентное ДО на сумму):

```cpp
vector<int> a;

struct Node {
    int sum;
    Node* l;
    Node* r;
    Node() {
        sum = 0;
        l = nullptr;
        r = nullptr;
    }
    Node(const Node* v) {
        sum = v->sum;
        l = v->l;
        r = v->r;
    }
};

Node* build(Node* v, int l, int r) {
    if (r - l == 1) {
        v->sum = a[l];
        return v;
    }
    int m = (l + r) / 2;
    v->l = new Node();
    v->r = new Node();
    v->l = build(v->l, l, m);
    v->r = build(v->r, m, r);
    v->sum = v->l->sum + v->r->sum;
    return v;
}

int ask(Node* v, int l, int r, int askl, int askr) {
    if (l >= askr || r <= askl) {
        return 0;
    }
    if (l >= askl && r <= askr) {
        return v->sum;
    }
    int m = (l + r) / 2;
    return ask(v->l, l, m, askl, askr) + ask(v->r, m, r, askl, askr);
}

Node* change(const Node* v, int l, int r, int pos, int val) {
    Node* u = new Node(v);
    if (r - l == 1) {
        u->sum = val;
        return u;
    }
    int m = (l + r) / 2;
    if (pos < m) {
        u->l = change(u->l, l, m, pos, val);
    }
    else {
        u->r = change(u->r, m, r, pos, val);
    }
    u->sum = u->l->sum + u->r->sum;
    return u;
}
```

Персистентое ДО может применяться для решения различных задач за хорошую асимптотику и в онлайне:
1. K-ая порядковая статистика на отрезке (нужно найти число, которое будет на k-ом месте, если отрезок отсортировать)
   Эту задачу можно решить при помощи merge-sort tree за O(log(n) ^ 2) на запрос, решение с персистентным ДО лучше
   Сначала сожмём числа (отсортируем массив, удалим повторения и каждому числу из исходного массива сопоставим индекс в сжатом массиве)
   Сделаем ДО на сумму, в листе i будет хранить кол-во чисел i в массиве в текущий момент
   Заметим, что не применяя персистентность мы можем искать ответ на префиксе (для этого нужно спуститься по дереву и найти позицию k-ого числа)
   Сделаем дерево персистентным. Пройдём по массиву (по не сжатому) и каждую версию будем добавлять по одному числу в ДО (увеличиваем значение в листе на 1)
   Сделав дерево персистентным, мы можем искать ответ на отрезке: будем одновременно спускаться по версии r и версии l - 1
   Тогда при проверке суммы левого сына во время спуска мы будем брать разность сумм двух версий
   Так мы получаем реальное кол-во чисел на отрезке, поэтому можем найти ответ
Код:

```cpp
template <class T> void uniq(vector<T>& a) {sort(a.begin(), a.end()); a.resize(unique(a.begin(), a.end()) - a.begin());}

struct Node {
    int sum;
    Node* l;
    Node* r;
    Node() {
        sum = 0;
        l = nullptr;
        r = nullptr;
    }
    Node(const Node* v) {
        sum = v->sum;
        l = v->l;
        r = v->r;
    }
};

vector<int> a;
vector<int> uniq_a;
vector<Node*> roots;

Node* build(Node* v, int l, int r) {
    if (r - l == 1) {
        v->sum = 0;
        return v;
    }
    int m = (l + r) / 2;
    v->l = new Node();
    v->r = new Node();
    v->l = build(v->l, l, m);
    v->r = build(v->r, m, r);
    v->sum = v->l->sum + v->r->sum;
    return v;
}

Node* change(const Node* v, int l, int r, int pos, int delta) {
    Node* u = new Node(v);
    if (r - l == 1) {
        u->sum += delta;
        return u;
    }
    int m = (l + r) / 2;
    if (pos < m) {
        u->l = change(u->l, l, m, pos, delta);
    }
    else {
        u->r = change(u->r, m, r, pos, delta);
    }
    u->sum = u->l->sum + u->r->sum;
    return u;
}

int kth(Node* v_r, Node* v_l, int l, int r, int k) {
    if (r - l == 1) {
        return l;
    }
    int m = (l + r) / 2;
    int val = v_r->l->sum - v_l->l->sum;
    if (val >= k) {
        return kth(v_r->l, v_l->l, l, m, k);
    }
    return kth(v_r->r, v_l->r, m, r, k - val);
}

signed main() {
    int n;
    cin >> n;
    a.resize(n);
    cin >> a;
    uniq_a = a;
    uniq(uniq_a);
    int N = uniq_a.size();
    Node* root = new Node();
    root = build(root, 0, N);
    roots.push_back(root);
    for (int i = 0; i < n; i++) {
        int index = lower_bound(all(uniq_a), a[i]) - uniq_a.begin();
        Node* nroot = change(root, 0, N, index, 1);
        roots.push_back(nroot);
        root = nroot;
    }
    int q;
    cin >> q;
    REP(q) {
        int l, r, k;
        cin >> l >> r >> k;
        cout << uniq_a[kth(roots[r], roots[l - 1], 0, N, k)] << el;
    }
    return 0;
}
```

2. Кол-во различных чисел на отрезке
   Для позиции i массива будем хранить 1, если это последнее вхождение числа a[i] на префиксе [0, i]
   При переходе к следующему префиксу, мы должны занулить предыдущее вхождение и поставить 1 на новую позицию
   Построим на этом массиве персистентное ДО
   Построим на этом массиве персистентное ДО
   Тогда для запроса [l, r] идём версию в r и считает сумму на [l, r] в этой версии
   Код не привожу, потому что на больших тестах мой код не работает (не знаю, почему)
   Если задачу можно решить в оффлайне, то лучшее решение - алгоритм Мо (описано в Mo)
   Также можно решить с использованием merge-sort tree (см. mergesort_tree)