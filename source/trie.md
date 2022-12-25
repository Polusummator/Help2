# trie

Бор (trie)

Бор - это префиксное дерево, хранящее все префиксы нескольких слов
Можно добавлять, удалять и искать строки

Код:
```cpp
struct Node {
    int cnt;
    int sum;
    vector<Node*> kids;
    Node() {
        cnt = 0;
        sum = 0;
        kids.resize(26, nullptr);
    }
};
void add(Node* v, const string& s, int pos) {
    if (pos == s.size()) {
        v->cnt++;
        v->sum++;
        return;
    }
    v->sum++;
    int kid = s[pos] - 'a';
    if (v->kids[kid] == nullptr) {
        v->kids[kid] = new Node();
    }
    add(v->kids[kid], s, pos + 1);
}
bool del(Node* v, const string& s, int pos) {
    if (pos == s.size()) {
        if (v->cnt == 0) {
            return false;
        }
        v->cnt--;
        v->sum--;
        return true;
    }
    int kid = s[pos] - 'a';
    if (v->kids[kid] == nullptr) {
        return false;
    }
    bool res = del(v->kids[kid], s, pos + 1);
    if (res) {
         v->sum--;
    }
    if (v->kids[kid]->sum == 0) {
        delete v->kids[kid];
        v->kids[kid] = nullptr;
    }
    return res;
}
bool find(Node* v, const string& s, int pos) {
    if (pos == s.size()) {
        return v->cnt > 0;
    }
    int kid = s[pos] - 'a';
    if (v->kids[kid] == nullptr) {
        return false;
    }
    return find(v->kids[kid], s, pos + 1);
}
signed main() {
    Node* root = new Node();
    string s;
    cin >> s;
    add(root, s, 0);
    return 0;
}
```

cnt - кол-во строк, заканчивающихся в вершине
sum - размер поддерева вершины

Существует битовый бор (вместо символов биты чисел)
Он позволяет искать mex множества, осуществлять операции xor
Для битового бора необходимо, чтобы все числа имели одинаковую длину (нужно дополнять их ведущими нулями до определённой длины)
Битовым бором можно сравнивать числа (если где-то первое число пошло налево, а второе направо, то первое число меньше)
Можно искать кол-во чисел меньше k. Для этого нужно посчитать сумму поддеревьев левых переходов (число k идёт направо => прибавим к ответу сумму левого перехода)

Бором можно посчитать кол-во различных подстрок строки
Для этого нужно положить в бор все суффиксы строки
Ответ - кол-во вершин в боре