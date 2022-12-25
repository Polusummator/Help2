# mitm

Meet-in-the-middle

MITM - это оптимизация перебора в множестве
Вместо полного перебора всех значений производится перебор в половинах множества, а потом полученные значения совмещаются

Примеры задач:
1. Задача о рюкзаке (какой максимальный вес можно набрать) с огромными весами
2. Разделить массив на два множества, чтобы разница сумм множеств была минимальной
3. Найти в массиве множество, сумма которого, взятая по модулю Mod, максимальна

1:
Разобьём все предметы на две равные части (разница +-1)
Для каждой части посчитаем все возможные суммы, которые мы может набрать, используя предметы этой части (перебор масок с проверкой, что сумма меньше, чем вместимость рюкзака)
Отсортируем обе части (можно также сделать uniq)
Теперь будем идти по суммам, полученным из первой части и искать максимальную сумму, которую мы можем взять из второй части (бинпоиск)
Можно поддерживать и стоимости предметов, для этого нужно в процессе получения всех сумм делать пары (вес, стоимость)
После получения всех сумм нужно отсортировать массив и удалить все пары, для которых мы можем набрать такой же вес, но бОльшую стоимость
Тогда пары будут отсортированы и по весу, и по стоимости
Код не привожу, но он очень похож на код второй и третьей задачи (только не надо забывать о проверке на то, что сумма весов не больше вместимости рюкзака)

2:
Опять же разобьём массив на две части
Мы хотим набрать сумму наиболее близкую к sum / 2 (sum - сумма всех чисел массива)
Для этого посчитаем для каждой части все возможные суммы, не превосходящие sum / 2
Отсортируем и для каждой суммы первой части найдём максимальную сумму из второй части
Код:
```cpp
ll get_sum(int mask, const vector<int>& a) {
    int cur = 1;
    ll sum = 0;
    int n = a.size();
    for (int i = 0; i < n; i++) {
        if (cur & mask) {
            sum += a[i];
        }
        cur = cur << 1;
    }
    return sum;
}

vector<ll> get_all_w(const vector<int>& a, ll w) {
    vector<ll> res;
    int n = a.size();
    res.push_back(0);
    for (int mask = 1; mask < (1 << n); mask++) {
        ll sum = get_sum(mask, a);
        if (sum <= w) {
            res.push_back(sum);
        }
    }
    return res;
}

signed main() {
    int n;
    cin >> n;
    vector<int> w1, w2;
    ll sum = 0;
    for (int i = 0; i < n; i++) {
        int x;
        cin >> x;
        if (i < n / 2) {
            w1.push_back(x);
        }
        else {
            w2.push_back(x);
        }
        sum += x;
    }
    ll half = sum / 2;
    vector<ll> all_w1 = get_all_w(w1, half);
    vector<ll> all_w2 = get_all_w(w2, half);
    sort(all_w2.begin(), all_w2.end());
    ll ans = 1e18;
    for (ll sum1 : all_w1) {
        ll need_sum = half - sum1;
        ll sum2;
        int ind = lower_bound(all_w2.begin(), all_w2.end(), need_sum) - all_w2.begin();
        if (ind == all_w2.size() || all_w2[ind] > need_sum) {
            ind--;
        }
        if (ind == -1) {
            continue;
        }
        sum2 = all_w2[ind];
        ans = min(abs(sum - 2 * (sum2 + sum1)), ans);
    }
    cout << ans << el;
    return 0;
}
```

3:
Разбиваем массив на две части
Для каждой части считаем все возможные суммы по модулю Mod
Сортируем суммы, делаем uniq
Для каждой суммы из первой части ищем наибольшую возможную сумму из второй части (очевидно, она должна быть меньше или равна, чем Mod - sum1 - 1)
Код:
```cpp
ll Mod;

ll get_sum(int mask, const vector<ll>& a) {
    ll sum = 0;
    int n = a.size();
    for (int i = 0; i < n; i++) {
        if ((1 << i) & mask) {
            sum += (a[i] % Mod);
            sum %= Mod;
        }
    }
    return sum;
}

vector<ll> get_all_sums(const vector<ll>& a) {
    vector<ll> res;
    int n = a.size();
    res.reserve((1 << n));
    for (int mask = 0; mask < (1 << n); mask++) {
        res.push_back(get_sum(mask, a));
    }
    return res;
}

signed main() {
    int n;
    cin >> n >> Mod;
    vector<ll> a1, a2;
    for (int i = 0; i < n; i++) {
        ll x;
        cin >> x;
        if (i < n / 2) {
            a1.push_back(x);
        }
        else {
            a2.push_back(x);
        }
    }
    vector<ll> all1 = get_all_sums(a1);
    vector<ll> all2 = get_all_sums(a2);
    uniq(all1);
    uniq(all2);
    ll ans = 0;
    for (ll cur : all1) {
        int index = upper_bound(all(all2), Mod - cur - 1) - all2.begin() - 1;
        if (index == -1) {
            continue;
        }
        ans = max(ans, (cur + all2[index]) % Mod);
    }
    cout << ans << el;
    return 0;
}
```