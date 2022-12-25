# Huffman

Кодирование Хаффмана

Код Хаффмана - алгоритм оптимального префиксного кодирования

```cpp
struct Node {
    Node *l, *r;
    bool is_leaf;
    char c;
    Node(): l(nullptr), r(nullptr), is_leaf(false) {}
};

void dfs(Node* v, unordered_map<char, string>& codes, string cur_code = "") {
    if (v->is_leaf) {
        codes[v->c] = cur_code;
        return;
    }
    dfs(v->l, codes, cur_code + '0');
    dfs(v->r, codes, cur_code + '1');
}

signed main() {
    int n;
    cin >> n;
    vector<int> freq(n);
    vector<char> let(n);
    cin >> freq;
    cin >> let;
    set<pair<int, Node*>> s;
    for (int i = 0; i < n; i++) {
        Node* v = new Node();
        v->is_leaf = true;
        v->c = let[i];
        s.insert({freq[i], v});
    }
    while (s.size() > 1) {
        auto fi = *s.begin();
        s.erase(s.begin());
        auto se = *s.begin();
        s.erase(s.begin());
        Node* cur = new Node();
        cur->l = fi.second;
        cur->r = se.second;
        s.insert({fi.first + se.first, cur});
    }
    Node* tree = s.begin()->second;
    unordered_map<char, string> codes;
    dfs(tree, codes);
    for (auto& i: codes) {
        cout << i.first << ' ' << i.second << el;
    }
    return 0;
}
```