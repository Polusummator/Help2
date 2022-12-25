# suffix_automaton

Суффиксный автомат

Суффиксный автомат - один из алгоритмов на строках, представляющий из себя ориентированный ациклический граф слов

Граф состоит из состояний (вершин), переходов (рёбер) и суффиксных ссылок
На переходах хранятся буквы
Код:
```cpp
const int MAXLEN = 100000;
state st[MAXLEN * 2];
int sz, last;

void sa_init() {
    sz = last = 0;
    st[0].len = 0;
    st[0].link = -1;
    ++sz;
}

void sa_extend(char c) {
    int cur = sz++;
    st[cur].len = st[last].len + 1;
    int p;
    for (p = last; p != -1 && !st[p].next.count(c); p = st[p].link)
        st[p].next[c] = cur;
    if (p == -1)
        st[cur].link = 0;
    else {
        int q = st[p].next[c];
        if (st[q].len == st[p].len + 1)
            st[cur].link = q;
        else {
            int clone = sz++;
            st[clone].link = st[q].link;
            st[clone].next = st[q].next;
            st[clone].len = st[p].len + 1;
            for (; p != -1 && st[p].next[c] == q; p = st[p].link)
                st[p].next[c] = clone;
            st[q].link = st[cur].link = clone;
        }
    }
    last = cur;
}
```

MAXLEN - максимальная длина строки во входных данных
Суффиксный автомат каждый раз добавляет новую букву, поэтому создание происходит в цикле по всем буквам строки

ПРИМЕНЕНИЯ:
1. Проверка, является ли строка (s) подстрокой другой строки (S)n
    Создадим суффиксный автомат для S
    Теперь будем идти от t0 (начального состояния) по переходам, хранящим буквы s (если s = "abc", то a -> b -> c)
    Если мы прошли по всей строке, то она является подстрокой. Если в какой-то момент нет нужного перехода, то строка не является подстрокой

    Код:
    ```cpp
    int v = 0;
    bool z = true;
    for (char & i : in) {
        if (st[v].next.count(i))
            v = st[v].next[i];
        else {
            z = false;
            break;
        }
    }
    if (z)
        cout << "yes" << el;
    else
        cout << "no" << el;
    ```

2. Кол-во различных подстрок
    Нужно посчитать кол-во различных путей от начальной вершины
    Для этого добавим функцию подсчёта путей (dfs с dp):
    ```cpp
    int col[MAXLEN * 2];
    ll dp[MAXLEN * 2];
    void dfs(int v) {
        col[v] = 1;
        for (pair<const char, int>& item : st[v].next) {
            if (!col[item.second])
                dfs(item.second);
            dp[v] += dp[item.second];
        }
        dp[v]++;
        col[v] = 2;
    }
    ```
    Для ответа нужно запустить dfs от вершины 0 и вывести dp[0] - 1 (чтобы не учитывать пустую строку нужно вычесть 1)

3. Минимальный циклический сдвиг строки
    Построим суффиксный автомат для строки (S + S)
    После этого пройдём по автомату от начальной вершины, выбирая жадно символы (каждый раз берём минимальный символ)
    Функция:
    ```cpp
    string min_shift(int n) {
        string ans;
        int cur_len = 0, v = 0;
        char c;
        while (cur_len < n) {
            cur_len++;
            for (pair<const char, int>& item: st[v].next) {
                v = item.second;
                c = item.first;
                break;
            }
            ans.push_back(c);
        }
        return ans;
    }
    ```
    Из основной программы функцию нужно запускать с аргументом n - начальная длина строки s (не удвоенная)

4. Количество вхождений подстроки (с пересечениями):
    Создадим суффиксный автомат для большей строки
    Будем хранить массив cnt
    Добавим в самое начало (2 строка) функции создания автомата строку cnt[cur] = 1
    После создания пройдём по переходам-символам меньшей строки (см. пункт 1) и найдём последнее состояние v
    Выведем cnt[v]
    Код:
    ```cpp
    void dfs(int v) {
        col[v] = 1;
        for (pair<const char, int>& item : st[v].next) {
            if (!col[item.second])
                dfs(item.second);
            cnt[st[item.second].link] += cnt[item.second];
        }
        col[v] = 2;
    }
    ```
    Основная функция:
    ```cpp
    ...dfs(0);
    int v = 0;
    for (char & i : in)
        v = st[v].next[i];
    cout << cnt[v];
    ...
    ```

5. Позиция первого вхождения строки в строку
    Будем хранить массив firstpos
    При создании st[cur] (начало функции), будем добавлять firstpos[cur] = st[cur].len - 1
    При клонировании firstpos[clone] = firstpos[q]
    Потом пройдём по символам меньшей строки p (см. предыдущие пункты) и найдём последнее состояние v
    Ответ: firstpos[v] - p.size() + 1

6. Позиции всех вхождений
    Будем модифицировать предыдущий пункт
    В state добавим bool is_clone (метка клона). При клонировании будем писать st[clone].is_clone = true
    В state добавим vector<int> inv_link (инвертированные суффиксные ссылки)
    Сразу после создания автомата:
    ```cpp
    for (int v = 1; v < sz; v++)
        st[st[v].link].inv_link.push_back(v);
    ```
    Потом пройдём по символам меньшей строки и найдём последнее состояние v
    Ответ будем хранить в массиве allpos
    Функция для получения ответа:
    ```cpp
    void find_allpos(int v, int p_len) {
        if (!st[v].is_clone)
            allpos.push_back(firstpos[v] - p_len + 1);
        for (int i : st[v].inv_link)
            find_allpos(i, p_len);
    }
    ```
    Из основной программы функцию следует запускать с аргументами v (найденное нами последнее состояние) и p_len (длина меньшей строки)
    ВАЖНО: 1. до создания автомата нужно проверить, является ли строка подстрокой, иначе ответ будет выглядеть странно
        2. allpos не отсортирован; если нужен отсортированный ответ, нужно сделать сортировку до вывода массива

7. Кратчайшая строка, не входящая в данную
    Будем решать с помощью dp
    Если из вершины v нет переходов по символам из алфавита, то dp[v] = 1
    Иначе dp[v] - это минимум из ответов по всем переходам + 1

8. Наидлиннейшая общая подстрока двух строк
    Для первой строки построим автомат
    Вторую строку передадим в функцию lcs
    Код:
    ```cpp
    string lcs(string t) {
        int v = 0, l = 0, best = 0, bestpos = 0;
        for (int i = 0; i < t.size(); i++) {
            while (v && !st[v].next.count(t[i])) {
                v = st[v].link;
                l = st[v].len;
            }
            if (st[v].next.count(t[i])) {
                v = st[v].next[t[i]];
                l++;
            }
            if (l > best) {
                best = l;
                bestpos = i;
            }
        }
        return t.substr(bestpos - best + 1, best);
    }
    ```