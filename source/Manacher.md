# Manacher

Алгоритм Манакера

Алгоритм Манакера позволяет находить для каждого элемента строки максимальную длину подстроки-палиндрома с центром в этом элементе

Алгоритм возвращает два массива половин длин палиндромов (для чётных и нечётных палиндромов)
Благодаря этому можно легко найти общее кол-во палиндромов в строке, сложив все значения двух массивов

Реализация Алгоритма Манакера схожа с реализацией оптимального алгоритма нахождения z-функции
Следующие значения вычисляются на основе предыдущих и хранятся границы палиндрома, правая граница которого максимальна

Есть 2 случая:
1. Следующий элемент лежит в сохранённом палиндроме (тогда находится зеркальное значение и выполняется наивный алгоритм)
2. Следующий элемент не лежит в сохранённом палиндроме (наивный алгоритм)

Реализация на C++:
```cpp
#define F first
#define S second
int n;
string s;
int increase(int middle_index, int size) {
    while (middle_index - size >= 0 &&
           middle_index + size < n  &&
           s[middle_index + size] == s[middle_index - size]) {
        size++;
    }
    return size - 1;
}
int main() {
    string input;
    cin >> input;
    n = 2 * input.size() - 1;
    for (int i = 0; i < n; ++i) {  //aboba -> a#b#o#b#a
        if (i % 2 == 0) {
            s += input[i / 2];
        } else {
            s += '#';
        }
    }
    vector<int> dp(n, 0);
    pair<int, int> big_pal = make_pair(0, 0);
    vector<int> odd(n / 2), even((n + 1) / 2);
    for (int mid_index = 0; mid_index < n; ++mid_index) {
        int pal_length;
        if (mid_index > big_pal.S) {  //первый случай
            pal_length = increase(mid_index, 0);
        }
        else {  //второй случай
            pal_length = dp[(big_pal.S + big_pal.F) - mid_index];
            if (mid_index + pal_length >= big_pal.S) {
                pal_length = increase(mid_index, big_pal.S - mid_index);
            }
        }
        dp[mid_index] = pal_length;  //обновление всего
        if (mid_index + pal_length > big_pal.S) {
            big_pal = make_pair(mid_index - pal_length,
                                mid_index + pal_length);
        }
        if (mid_index % 2 == 0) {  //заполнение массивов с ответами, неочевидно, где округление сверху и снизу
            even[mid_index / 2] = (pal_length / 2) + 1;
        } else {
            odd[mid_index / 2] =  (pal_length + 1) / 2;
        }
    }
    for (int i: even) {
        cout << i << ' ';
    }
    cout << endl;
    for (int i: odd) {
        cout << i << ' ';
    }
    cout << endl;
}
```