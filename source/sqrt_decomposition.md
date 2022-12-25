# sqrt_decomposition

Корневая декомпозиция

Смысл декомпозиции - разбиение отрезка на блоки
Тогда, когда мы получаем запрос, мы можем посчитать значения в блоках, которые полностью входят в отрезок, а на частичных блоках посчитать, просто пройдя по эти частям
Блоки равны корню из длины массива

Также корневая декомпозиция поддерживает изменения элементов
Для этого используются отложенные изменения

Минимум на [l, r]:
```cpp
struct Block {
    vector<int> arr;
    int minElement;
    int len;
    void build() {
        len = arr.size();
        minElement = arr[0];
        for (int i = 1; i < len; i++) {
            minElement = min(arr[i], minElement);
        }
    }
    int fullQuery() {
        return minElement;
    }
    int partQuery(int l, int r) {
        int ans = arr[l];
        for (int i = l + 1; i <= r; i++) {
            ans = min(ans, arr[i]);
        }
        return ans;
    }
    void change(int i, int val) {
        arr[i] = val;
        build();
    }
};
struct SQRT {
    vector<Block> bl;
    int bSize, len;
    SQRT(const vector<int>& a) {
        int n = a.size();
        bSize = (int)sqrt(n);
        len = 0;
        for (int i = 0; i < n; i++) {
            int b = i / bSize;
            if (b == len) {
                bl.push_back(Block());
                len++;
            }
            bl[b].arr.push_back(a[i]);
        }
        for (int i = 0; i < len; i++) {
            bl[i].build();
        }
    }
    int ask(int l, int r) {
        int posl = l / bSize, posr = r / bSize;
        int posinl = l % bSize, posinr = r % bSize;
        if (posl == posr) {
            return bl[posl].partQuery(posinl, posinr);
        }
        int ans = INT_MAX;
        if (posinl != 0) {
            ans = min(ans, bl[posl].partQuery(posinl, bl[posl].len - 1));
            posl++;
        }
        if (posinr != bl[posr].len - 1) {
            ans = min(ans, bl[posr].partQuery(0, posinr));
            posr--;
        }
        for (int i = posl; i <= posr; i++) {
            ans = min(ans, bl[i].fullQuery());
        }
        return ans;
    }
    void change(int i, int val) {
        int pos = i / bSize;
        int posin = i % bSize;
        //int posin = i - pos * bSize;
        bl[pos].change(posin, val);
    }
};
```

Сумма на [l, r] + изменения на [l1, r1]:
```cpp
struct Block {
    vector<int> arr;
    int summ;
    bool has_set;
    int set;
    int len;
    void build() {
        len = arr.size();
        has_set = false;
        set = 0;
        summ = 0;
        for (int i: arr) {
            summ += i;
        }
    }
    void push() {
        if (!has_set)
            return;
        for (int i = 0; i < len; i++) {
            arr[i] = set;
        }
        has_set = false;
        summ = len * set;
        set = 0;
    }
    int partQuery(int l, int r) {
        push();
        int ans = 0;
        for (int i = l; i <= r; i++) {
            ans += arr[i];
        }
        return ans;
    }
    int fullQuery(int l, int r) {
        if (has_set)
            return len * set;
        else
            return summ;
    }
    void partSet(int l, int r, int val) {
        push();
        for (int i = l ; i < r; i++) {
            summ -= arr[i];
            arr[i] = val;
            summ += val;
        }
    }
    void fullSet(int val) {
        has_set = true;
        set = val;
    }
};
struct SQRT {
    vector<Block> bl;
    int bSize, len;
    SQRT(const vector<int>& a) {
        int n = a.size();
        bSize = (int)sqrt(n);
        len = 0;
        for (int i = 0; i < n; i++) {
            int b = i / bSize;
            if (b == len) {
                bl.push_back(Block());
                len++;
            }
            bl[b].arr.push_back(a[i]);
        }
        for (int i = 0; i < len; i++) {
            bl[i].build();
        }
    }
    int ask(int l, int r) {
        int posl = l / bSize, posr = r / bSize;
        int posinl = l % bSize, posinr = r % bSize;
        if (posl == posr) {
            return bl[posl].partQuery(posinl, posinr);
        }
        int ans = 0;
        if (posinl != 0) {
            ans += bl[posl].partQuery(posinl, bl[posl].len - 1);
            posl++;
        }
        if (posinr != bl[posr].len - 1) {
            ans += bl[posr].partQuery(0, posinr);
            posr--;
        }
        for (int i = posl; i <= posr; i++) {
            ans += bl[i].fullQuery();
        }
        return ans;
    }
    void change(int l, int r, int val) {
        int posl = l / bSize, posr = r / bSize;
        int posinl = l % bSize, posinr = r % bSize;
        if (posl == posr) {
            bl[posl].partSet(posinl, posinr, val);
            return;
        }
        int ans = 0;
        if (posinl != 0) {
            bl[posl].partSet(posinl, bl[posl].len - 1, val);
            posl++;
        }
        if (posinr != bl[posr].len - 1) {
            bl[posr].partSet(0, posinr, val);
            posr--;
        }
        for (int i = posl; i <= posr; i++) {
            bl[i].fullSet(val);
        }
    }
};
```

Сколько на отрезке чисел из заданного диапазона + прибавление на отрезке:
Для этого требуется хранить в каждом блоке отсортированный блок
```cpp
struct Block {
    vector<int> arr, srt;
    int add;
    int len;
    void build() {
        len = arr.size();
        add = 0;
        srt = arr;
        sort(srt.begin(), srt.end());
    }
    void push() {
        for (int i = 0; i < len; i++) {
            arr[i] += add;
            srt[i] += add;
        }
        add = 0;
    }
    int partQuery(int l, int r, int numl, int numr) {
        push();
        int ans = 0;
        for (int i = l; i <= r; i++) {
            if (arr[i] >= numl && arr[i] <= numr) {
                ans++;
            }
        }
        return ans;
    }
    int fullQuery(int numl, int numr) {
        return upper_bound(srt.begin(), srt.end(), numr) - lower_bound(srt.begin(), srt.end(), numl);
    }
    void partAdd(int l, int r, int val) {
        push();
        for (int i = l; i <= r; i++) {
            arr[i] += val;
        }
        srt = arr;
        sort(srt.begin(), srt.end());
    }
    void fullAdd(int val) {
        add += val;
    }
};
struct SQRT {
    vector<Block> bl;
    int bSize, len;
    SQRT(const vector<int>& a) {
        int n = a.size();
        bSize = (int)sqrt(n);
        len = 0;
        for (int i = 0; i < n; i++) {
            int b = i / bSize;
            if (b == len) {
                bl.push_back(Block());
                len++;
            }
            bl[b].arr.push_back(a[i]);
        }
        for (int i = 0; i < len; i++) {
            bl[i].build();
        }
    }
    int ask(int l, int r, int numl, int numr) {
        int posl = l / bSize, posr = r / bSize;
        int posinl = l % bSize, posinr = r % bSize;
        if (posl == posr) {
            return bl[posl].partQuery(posinl, posinr, numl, numr);
        }
        int ans = 0;
        if (posinl != 0) {
            ans += bl[posl].partQuery(posinl, bl[posl].len - 1, numl, numr);
            posl++;
        }
        if (posinr != bl[posr].len - 1) {
            ans += bl[posr].partQuery(0, posinr, numl, numr);
            posr--;
        }
        for (int i = posl; i <= posr; i++) {
            ans += bl[i].fullQuery(numl, numr);
        }
        return ans;
    }
    void change(int l, int r, int val) {
        int posl = l / bSize, posr = r / bSize;
        int posinl = l % bSize, posinr = r % bSize;
        if (posl == posr) {
            bl[posl].partAdd(posinl, posinr, val);
            return;
        }
        int ans = 0;
        if (posinl != 0) {
            bl[posl].partAdd(posinl, bl[posl].len - 1, val);
            posl++;
        }
        if (posinr != bl[posr].len - 1) {
            bl[posr].partAdd(0, posinr, val);
            posr--;
        }
        for (int i = posl; i <= posr; i++) {
            bl[i].fullAdd(val);
        }
    }
};
```

Может быть такая задача: посчитать какую-нибудь функцию на отрезке и добавить число на какую-либо позицию
Первая часть решается просто
Втору часть можно решить, перебирая блоки по порядку, куда должен встать новый элемент
Это делается за O(sqrt(N)
Однако есть проблема "раздувания" блока
Для этого можно разбивать блок, когда его длина становится 2 * sqrt(N)
То же самое с удалением чисел (сливать, когда длина 0.5 * sqrt(N))
Но для этого требуется хранить структуру в двусвязном списке, что не очень приятно
Второе решение: перестраивать всю структуру каждые sqrt(N) запросов добавления элементов

Задача: есть множество, поступают два вида запросов:
1) Добавить число
2) Найти кол-во чисел l <= a <= r
При запросе 1 будем кидать число в какую-либо структуру (можно использовать обычный массив)
Когда длина этой структуры становится больше sqrt(N) (N - сколько всего чисел добавится), то мы сортируем структуру и сливаем с множеством (merge)
Саму структуру обнуляем и делаем дальше всё то же самое
На второй запрос ответить просто - два бинпоиска

Корневая оптимизация может применяться в графах
Задача: дан граф, поступают два вида запросов:
1) += x в вершине
2) Посчитать сумму для всех соседей u
Главная идея - разбить все вершины на тяжёлые и лёгкие
Тяжёлая вершина - вершина, у которой степень >= sqrt(M)
Лёгкая вершина - вершина, у которой степень < sqrt(N)
Для запроса 1 делаем прибавление в вершине и проходим по всем тяжёлым соседям вершины u
Для них бы обновляем ответ, для лёгких ответ не меняем
Запрос 2:
Если вершина лёгкая, то просто пройдём по вем соседям и посчитаем сумму
Если вершина тяжёлая, то ответ для неё уже посчитан

Задача: найти кол-во треугольников в неориентированном графе (циклов длины 3)
Разобьём все вершины на тяжёлые и лёгкие (описано выше)
Будем перебирать рёбра (у нас 3 вида рёбер: L - L, H - L, H - H)
У нас есть 4 варианта треугольника:
1) L - L - L
Для этого случая переберём рёбра и для каждого ребра L(v) - L(v) пройдём по всем соседям v и соседям u
Найдём всех общих соседей, которые L, и прибавим его к ответу
В конце кол-во треугольников L - L - L нужно будет поделить на 3
2) L - L - H
То же самое (перебираем ребро L - L), только в конце не нужно делить
3) L - H - H
Будем перебирать ребро L - H
Для вершины L переберём всех соседей
Если сосед H, то нужно проверить, является ли он соседом для второго конца ребра
Это можно сделать, храня матрицу смежности для всех H вершин (их не более sqrt(M), поэтому матрица не будет большой)
В конце поделить на 2
4) H - H - H
Просто сделаем три вложенных цикла (проверять, соседи ли две H вершины, мы умеем)
В конце поделить на 3