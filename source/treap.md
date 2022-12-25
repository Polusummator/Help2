# treap

Декартово дерево (treap, cartesian tree, дуча, дерамида, курево, пиво...)

ДД - сбалансированное дерево поиска
В каждой вершине ДД хранятся значения key и prior
key - ключи (значения, по которым строится ДД)
prior - приоритеты (значения, выбираемые случайно)
По значениям key ДД - бинарное дерево поиска (l_key < m_key < r_key)
По значениям prior ДД - бинарная куча (l_prior < m_prior > r_prior)

ДД будет строится на указателях, поэтому нужна структура вершины Node

ДД может выполнять операции:
* find(x)   - проверка, есть ли x в дереве
* add(x)    - добавление элемента x в дерево
* remove(x) - удаление элемента x из дерева
* next(x)   - наименьшей элемент, который >x
* prev(x)   - наибольший элемент, который <x
* kth(k)    - k-ый по величине элемент (для этого потребуется хранение размеров поддервьев - см. далее)
... (другие операции, например, RSQ/RMQ)

Но все операции будут основываться на двух: merge и split
merge сливает два ДД в одно (предполагается, что их можно слить - есть такой ключ, что левое дерево меньше этого ключа, а правое больше)
split(key) делит ДД на два по ключу

Реализация ДД (явный ключ):
```cpp
static random_device rd;
static mt19937 rng{rd()};

int get_rand() {
    static uniform_int_distribution<int> uid(0, 2e9 + 7);
    return uid(rng);
}

struct Node {
    int key;
    int prior;
    Node* l;
    Node* r;
    Node() {
        l = nullptr;
        r = nullptr;
    }
    Node(int key_) {
        key = key_;
        prior = get_rand();
        l = nullptr;
        r = nullptr;
    }
};

pair<Node*, Node*> split(Node* v, int x) {
    if (v == nullptr) {
        return {nullptr, nullptr};
    }
    if (v->key < x) {
        pair<Node*, Node*> p = split(v->r, x);
        v->r = p.first;
        return {v, p.second};
    }
    pair<Node*, Node*> p = split(v->l, x);
    v->l = p.second;
    return {p.first, v};
}

Node* merge(Node* v1, Node* v2) {
    if (v1 == nullptr) {
        return v2;
    }
    if (v2 == nullptr) {
        return v1;
    }
    if (v1->prior > v2->prior) {
        v1->r = merge(v1->r, v2);
        return v1;
    }
    v2->l = merge(v1, v2->l);
    return v2;
}

Node* find(Node* v, int x) {
    if (v == nullptr || v->key == x) {
        return v;
    }
    if (x < v->key) {
        return find(v->l, x);
    }
    return find(v->r, x);
}

Node* add(Node* v, Node* x) {
    if (v == nullptr) {
        v = x;
        return v;
    }
    if (x->prior > v->prior) {
        pair<Node*, Node*> p = split(v, x->key);
        x->l = p.first;
        x->r = p.second;
        return x;
    }
    if (x->key < v->key) {
        v->l = add(v->l, x);
    }
    else {
        v->r = add(v->r, x);
    }
    return v;
}

Node* remove(Node* v, int x) {
    if (v == nullptr) {
        return v;
    }
    if (v->key == x) {
        return merge(v->l, v->r);
    }
    if (x < v->key) {
        v->l = remove(v->l, x);
    }
    else {
        v->r = remove(v->r, x);
    }
    return v;
}

Node* prev(Node* v, int x) {
    Node* cur = v;
    Node* succ = nullptr;
    while (cur != nullptr) {
        if (cur->key < x) {
            succ = cur;
            cur = cur->r;
        }
        else {
            cur = cur->l;
        }
    }
    return succ;
}

Node* next(Node* v, int x) {
    Node* cur = v;
    Node* succ = nullptr;
    while (cur != nullptr) {
        if (cur->key > x) {
            succ = cur;
            cur = cur->l;
        }
        else {
            cur = cur->r;
        }
    }
    return succ;
}
```

В основной части программы сначала нужно сделать Node* root = nullptr

Строить ДД можно просто последовательным добавлением элементов
Но если все ключи известны в начале, то можно построить ДД за O(n)
Для этого можно отсортировать все ключи. Тогда следующий элемент будет либо добавлен в конец самой правой ветви, либо разделит её на две части (это делается при помощи stack)
```cpp
void build(vector<Node*>& a) {
    Node* root = a[0];
    int n = a.size();
    stack<Node*> s;
    s.push(root);
    for (int i = 1; i < n; i++) {
        Node* v = a[i];
        while (!s.empty() && s.top()->prior > v->prior) {
            s.pop();
        }
        if (s.empty()) {
            v->l = root;
            root = v;
            s.push(root);
        }
        else {
            Node* top = s.top();
            v->l = top->r;
            top->r = v;
            s.push(top);
            s.push(v);
        }
    }
}
```

В основном, запросы выполняются по следующему принципу:
1. Разбить дерево по правой границе
2. Разбить первую часть по левой границе запроса
3. Найти ответ на средней части
4. Слить всё обратно

Пример для RSQ(x, y) - сумма чисел в диапазоне от x до y (конечно, на отсортированном массиве):
```cpp
ll RSQ(Node* v, int l, int r) {
    pair<Node*, Node*> p1 = split(v, l);
    pair<Node*, Node*> p2 = split(p1.second, r + 1);
    ll res = get_sum(p2.first);
    p1.second = merge(p2.first, p2.second);
    merge(p1.first, p1.second);
    return res;
}
```

Хранение размеров поддеревьев добавляет возможность поиска k по величине элемента (и множество возможностей для ДД по неявному ключу)
Для этого нужны функции update и get_sz
update нужно вызывать после каждого изменения вершины
```cpp
int get_sz(Node* v) {
    if (v == nullptr) {
        return 0;
    }
    return v->sz;
}

void update(Node* v) {
    if (v != nullptr) {
        v->sz = get_sz(v->l) + get_sz(v->r) + 1;
    }
}

// 1-индексация
Node* kth(Node* v, int k) {
    if (v == nullptr) {
        return nullptr;
    }
    if (get_sz(v->l) + 1 == k) {
        return v;
    }
    if (get_sz(v->l) + 1 > k) {
        return kth(v->l, k);
    }
    return kth(v->r, k - get_sz(v->l) - 1);
}

// поиск индекса элемента по его ключу
int find_index(Node* v, int x, int ind) {
    if (v == nullptr) {
        return -1;
    }
    if (v->key == x) {
        return ind + get_sz(v->l);
    }
    if (x < v->key) {
        return find_index(v->l, x, ind);
    }
    return find_index(v->r, x, ind + get_sz(v->l) + 1);
}
```

Главное: если мы идём в левого сына, k остаётся прежним
         если мы идём в правого сына, то из k вычитается get_sz(v->l) + 1 (величина левого сына + текущая вершина)

ДД по неявному ключу см. в implicit_treap
