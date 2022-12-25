# implicit_treap

Декартово дерево по неявному ключу

Смысл ДД по неявному ключу - отсутствие ключей в вершинах
Для получения значений используются размеры поддеревьев

Такая структура позволяет хранить любой массив (не только отсортированный, как ДД по явному ключу)

split теперь будет отрезать первые k элементов
merge не изменится, т.к. в его коде никак не участвуют ключи

Теперь можно считать все функции, которые считает ДО (RSQ, RMQ...)
Считать можно двумя способами:
1. Разрезать, найти, склеить - простой способ, но может работать дольше, чем 2 способ
2. Запрос ДО (можно написать такой же запрос, как для ДО)
Отличие от ДО во втором способе - значения в ДД не только в листьях, но и в узлах, поэтому нужно добавлять значение в узле в ответ

Код предполагает, что для 1 способа в функцию передаются исходные значения l и r ([l, r], l и r в 1-индексации)
Во втором способе предполагается, что в функцию передаётся полуинтервал ([l, r), l и r в 0-индексации)

Для примера реализация RMQ с добавлением элемента в массив после элемента k:
```cpp
static random_device rd;
static mt19937 rng{rd()};

int get_rand() {
    static uniform_int_distribution<int> uid(0, 2e9 + 7);
    return uid(rng);
}

struct Node {
    int val;
    int prior;
    int sz;
    int min;
    Node* l;
    Node* r;
    Node() {
        sz = 0;
        min = 2e9;
        l = nullptr;
        r = nullptr;
    }
    Node(int val_) {
        val = val_;
        prior = get_rand();
        sz = 1;
        min = val_;
        l = nullptr;
        r = nullptr;
    }
};

int get_sz(Node* v) {
    if (v == nullptr) {
        return 0;
    }
    return v->sz;
}

int get_min(Node* v) {
    if (v == nullptr) {
        return 2e9;
    }
    return v->min;
}

void update(Node* v) {
    if (v != nullptr) {
        v->sz = get_sz(v->l) + get_sz(v->r) + 1;
        v->min = min({v->val, get_min(v->l), get_min(v->r)});
    }
}

pair<Node*, Node*> split(Node* v, int k) {
    if (v == nullptr) {
        return {nullptr, nullptr};
    }
    if (k <= get_sz(v->l)) {
        pair<Node*, Node*> p = split(v->l, k);
        v->l = p.second;
        update(v);
        return {p.first, v};
    }
    pair<Node*, Node*> p = split(v->r, k - get_sz(v->l) - 1);
    v->r = p.first;
    update(v);
    return {v, p.second};
}

// merge остаётся таким же
Node* merge(Node* v1, Node* v2) {
    if (v1 == nullptr) {
        return v2;
    }
    if (v2 == nullptr) {
        return v1;
    }
    if (v1->prior > v2->prior) {
        v1->r = merge(v1->r, v2);
        update(v1);
        return v1;
    }
    v2->l = merge(v1, v2->l);
    update(v2);
    return v2;
}

// медленно
int RMQ(Node* v, int l, int r) {
    pair<Node*, Node*> p1 = split(v, r);
    pair<Node*, Node*> p2 = split(p1.first, l - 1);
    int res = p2.second->min;
    p1.first = merge(p2.first, p2.second);
    merge(p1.first, p1.second);
    return res;
}

// быстрее
int RMQ2(Node* v, int l, int r, int askl, int askr) {
    if (v == nullptr) {
        return 2e9;
    }
    if (l >= askr || r <= askl) {
        return 2e9;
    }
    if (l >= askl && r <= askr) {
        return v->min;
    }
    int l_sz = get_sz(v->l);
    int one = (l_sz + l >= askl && l + l_sz < askr ? v->val : 2e9);
    return min({RMQ2(v->l, l, l + l_sz, askl, askr), RMQ2(v->r, l + l_sz + 1, r, askl, askr), one});
}

Node* add(Node* v, Node* x, int k) {
    if (v == nullptr) {
        v = x;
        update(v);
        return v;
    }
    if (x->prior > v->prior) {
        pair<Node*, Node*> p = split(v, k);
        x->l = p.first;
        x->r = p.second;
        update(x);
        return x;
    }
    if (get_sz(v->l) < k) {
        v->r = add(v->r, x, k - get_sz(v->l) - 1);
    }
    else {
        v->l = add(v->l, x, k);
    }
    update(v);
    return v;
}

Node* remove(Node* v, int k) {
    if (v == nullptr) {
        return v;
    }
    if (k == get_sz(v->l)) {
        return merge(v->l, v->r);
    }
    if (get_sz(v->l) < k) {
        v->r = remove(v->r, k - get_sz(v->l) - 1);
    }
    else {
        v->l = remove(v->l, k);
    }
    update(v);
    return v;
}
```


ДД поддерживает и отложенные операции (массовые операции)
Для примера код задачи о перевороте отрезка и поиска минимума на отрезке:
```cpp
static random_device rd;
static mt19937 rng{rd()};

int get_rand() {
    static uniform_int_distribution<int> uid(0, 2e9 + 7);
    return uid(rng);
}

struct Node {
    int val;
    int min;
    bool rev;
    int prior;
    int sz;
    Node* l;
    Node* r;
    Node(int val_) {
        val = val_;
        prior = get_rand();
        sz = 1;
        rev = false;
        min = val_;
        l = nullptr;
        r = nullptr;
    }
};

int get_sz(Node* v) {
    if (v == nullptr) {
        return 0;
    }
    return v->sz;
}

int get_min(Node* v) {
    if (v == nullptr) {
        return 2e9;
    }
    return v->min;
}

void update(Node* v) {
    if (v != nullptr) {
        v->sz = get_sz(v->l) + get_sz(v->r) + 1;
        v->min = min({v->val, get_min(v->l), get_min(v->r)});
    }
}

void push(Node* v) {
    if (v == nullptr || !v->rev) {
        return;
    }
    swap(v->l, v->r);
    v->rev = false;
    if (v->l != nullptr) {
        v->l->rev ^= true;
    }
    if (v->r != nullptr) {
        v->r->rev ^= true;
    }
}

pair<Node*, Node*> split(Node* v, int k) {
    push(v);
    if (v == nullptr) {
        return {nullptr, nullptr};
    }
    if (get_sz(v->l) < k) {
        pair<Node*, Node*> p = split(v->r, k - get_sz(v->l) - 1);
        v->r = p.first;
        update(v);
        return {v, p.second};
    }
    pair<Node*, Node*> p = split(v->l, k);
    v->l = p.second;
    update(v);
    return {p.first, v};
}

Node* merge(Node* v1, Node* v2) {
    push(v1);
    push(v2);
    if (v1 == nullptr) {
        return v2;
    }
    if (v2 == nullptr) {
        return v1;
    }
    if (v1->prior > v2->prior) {
        v1->r = merge(v1->r, v2);
        update(v1);
        return v1;
    }
    v2->l = merge(v1, v2->l);
    update(v2);
    return v2;
}

Node* add(Node* v, Node* x, int k) {
    if (v == nullptr) {
        v = x;
        update(v);
        return v;
    }
    if (x->prior > v->prior) {
        pair<Node*, Node*> p = split(v, k);
        x->l = p.first;
        x->r = p.second;
        update(x);
        return x;
    }
    if (get_sz(v->l) < k) {
        v->r = add(v->r, x, k - get_sz(v->l) - 1);
    }
    else {
        v->l = add(v->l, x, k);
    }
    update(v);
    return v;
}

void reverse(Node* v, int l, int r) {
    pair<Node*, Node*> p1 = split(v, r);
    pair<Node*, Node*> p2 = split(p1.first, l - 1);
    p2.second->rev ^= true;
    p1.first = merge(p2.first, p2.second);
    merge(p1.first, p1.second);
}

int RMQ(Node* v, int l, int r) {
    pair<Node*, Node*> p1 = split(v, r);
    pair<Node*, Node*> p2 = split(p1.first, l - 1);
    int res = p2.second->min;
    p1.first = merge(p2.first, p2.second);
    merge(p1.first, p1.second);
    return res;
}
```
