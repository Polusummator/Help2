# combinations_enumeration

Один из способов перевода - битовые маски
Следующий код выводит все сочетания из 0 и 1 длиной n

```cpp
#include <iostream>

using namespace std;

int main() {
	int n;
	cin >> n;
	for (int i = 0; i < (1 << n); ++i) {
		for (int q = 0; q < n; ++q) {
			if (i & (1 << q)) {
				cout << 1;
			}
			else {
				cout << 0;
			}
		}
		cout << endl;
	}
}
```

Следующий код выводит все сочетания чисел от start до end длиной n в лексикографическом порядке

Python:
```cpp
def gen(left, cur):
    if left == 0:
        print(*cur)
        return
    for i in range(start, end + 1):
        gen(left - 1, cur + [i])


n, start, end = map(int, input().split())
gen(n, [])
```

C++:
```cpp
#include <iostream>
#include <vector>

using namespace std;

int n, start, end_;

void gen(int left, vector<int> cur) {
	if (left == 0) {
		for (int i : cur) {
			cout << i << ' ';
		}
		cout << endl;
		return;
	}
	for (int i = start; i <= end_; ++i) {
		cur.push_back(i);
		gen(left - 1, cur);
		cur.pop_back();
	}
}

int main() {
	cin >> n >> start >> end_;
	gen(n, vector<int>{});
}
```

Следующий код имеет функцию gen, которая делает то же самое, что и предыдущий код (только без повторений чисел)
Также код имеет функцию n_permutation (аналог встроенной next_permutation), которая выводит следующую последовательность из цифр в лексикографическом порядке

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int n;

void gen(int left, vector<int> cur, vector<int> been) {
	if (left == 0) {
		for (int i : cur) {
			cout << i << ' ';			
		}
		cout << endl;
		return;
	}
	for (int i = 0; i < n; ++i) {
		if (!been[i]) {
			been[i] = 1;
			cur.push_back(i + 1);
			gen(left - 1, cur, been);
			cur.pop_back();
			been[i] = 0;
		}
	}
}

void n_permutation(vector<int>& p) {
	int n = p.size();
	for (int i = n - 1; i > 0; --i) {
		if (p[i] > p[i - 1]) {
			int pos;
			for (int j = n - 1; j >= i; --j) {
				if (p[j] > p[i - 1]) {
					pos = j;
					break;
				}
			}
			swap(p[pos], p[i - 1]);
			sort(p.begin() + i, p.end());
			break;
		}
	}
}

int main() {
	cin >> n;
	gen(n, vector<int>{}, vector<int>(n, 0));
	/*vector<int> a = { 3, 1, 5, 2, 7, 6, 4 };
	n_permutation(a);
	for (int i : a) {
		cout << i << ' ';
	}*/
}
```