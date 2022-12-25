# Mo

Алгоритм Мо

Смысл алгоритма состоит в заранее отсортированном списке запросов (поэтому только offline)
От одного запроса алгоритм перетекает к другому прибавлениями и удалениями элементов на концах текущего отрезка
Массив разделён на блоки (корень из длины массива)

Мо для суммы на отрезке чисел, которые встречаются хотя бы 3 раза:
```cpp
struct Magic {
    unordered_map<int, int> cnt;
    int summ;
    Magic(): summ(0) {}
    int f() {
        return summ;
    }
    void add(int x) {
        cnt[x]++;
        if (cnt[x] == 3)
            summ += x;
    }
    void del(int x) {
        cnt[x]--;
        if (cnt[x] == 2)
            summ -= x;
    }
};

struct query {
    int l, r, num;
    query() {}
    query(int l, int r, int num): l(l), r(r), num(num) {}
};

int K;

bool operator <(const query& a, const query& b) {
    if (a.l / K != b.l / K)
        return a.l / K < b.l / K;
    if ((a.l / K) % 2 == 0)
        return a.r < b.r;
    return a.r > b.r;
}

signed main() {
    fastIO;
    int n;
    cin >> n;
    vector<int> a(n);
    cin >> a;
    K = (int)sqrt(n);
    int Q;
    cin >> Q;
    vector<query> qr;
    for (int i = 0; i < Q; i++) {
        int l, r;
        cin >> l >> r;
        qr.emplace_back(l - 1, r - 1, i);
    }
    sort(qr.begin(), qr.end());
    Magic m;
    int l = 0, r = -1;
    vector<int> ans(Q);
    for (auto& q: qr) {
        while (r < q.r) {
            m.add(a[r + 1]);
            r++;
        }
        while (l > q.l) {
            m.add(a[l - 1]);
            l--;
        }
        while (r > q.r) {
            m.del(a[r]);
            r--;
        }
        while (l < q.l) {
            m.del(a[l]);
            l++;
        }
        ans[q.num] = m.f();
    }
    return 0;
}
```

Мо для кол-во различных чисел на отрезке:
```cpp
template <class T> void uniq(vector<T>& a) {sort(a.begin(), a.end()); a.resize(unique(a.begin(), a.end()) - a.begin());}

int block_size;

struct query {
    int l, r, num;
    query() {}
    query(int l_, int r_, int num_) {
        l = l_;
        r = r_;
        num = num_;
    }
};

bool operator<(const query& a, const query& b) {
    if (a.l / block_size != b.l / block_size) {
        return a.l / block_size < b.l / block_size;
    }
    if ((a.l / block_size) % 2 == 0) {
        return a.r < b.r;
    }
    return a.r > b.r;
}

struct Mo {
    int cnt[300000];
    int res;
    Mo() {
        fill(cnt, cnt + 300000, 0);
        res = 0;
    }
    int get_res() {
        return res;
    }
    void add(int x) {
        cnt[x]++;
        if (cnt[x] == 1) {
            res++;
        }
    }
    void del(int x) {
        cnt[x]--;
        if (cnt[x] == 0) {
            res--;
        }
    }
};

signed main() {
    fastIO;
    int n;
    cin >> n;
    vector<int> a(n), v(n);
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        v[i] = a[i];
    }
    uniq(v);
    for (int i = 0; i < n; i++) {
        a[i] = (int)(lower_bound(all(v), a[i]) - v.begin());
    }
    int q;
    cin >> q;
    vector<int> ans(q);
    vector<query> Q(q);
    for (int i = 0; i < q; i++) {
        int l, r;
        cin >> l >> r;
        Q[i] = query(l - 1, r - 1, i);
    }
    block_size = (int)sqrt(n);
    Mo m;
    stable_sort(all(Q));
    int l = 0, r = -1;
    for (int i = 0; i < q; i++) {
        int to_l = Q[i].l;
        int to_r = Q[i].r;
        while (r < to_r) {
            m.add(a[r + 1]);
            r++;
        }
        while (l > to_l) {
            m.add(a[l - 1]);
            l--;
        }
        while (r > to_r) {
            m.del(a[r]);
            r--;
        }
        while (l < to_l) {
            m.del(a[l]);
            l++;
        }
        ans[Q[i].num] = m.get_res();
    }
    for (int& i: ans) {
        cout << i << el;
    }
    return 0;
}
```
