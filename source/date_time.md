# date_time

Задачи на дату/время

ВРЕМЯ

Считывание при вводе:
```py
P: h, m, s = [int(i) for i in input().split(':')]
```
```cpp
C: scanf("%d:%d:%d", &h, &m, &s)
```

Перевод из строки в число:
```py
P: int(s[0] + s[1])
```
```cpp
C: (s[0] - '0') * 10 + (s[1] - '0') # char является числом (кодом символов в ASCII)
```

проще перевести всё в секунды: ```sec = h * 3600 + m * 60 + s```

ДАТЫ

Считывание, такое же, как во времени

Перевод в дни:
```py
days = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31] # можно сделать префиксные суммы
d = sum(days[mm - 2]) + dd # может возникнуть проблема с месяцем (если введён первый месяц)
```

Перевод из дней в месяц:
```py
i = 0
while x > 0:
    if x <= days[i]:
         print(i + 1)
         break
    else:
         x -= days[i]
         i += 1
```

Полный код переводов в C/C++ (високосный год не учитывается):)
```cpp
#include <iostream>
using namespace std;
int prefDays[12];
int days[12] = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
int dateToDays(int m, int d) {
	int ans = 0;
	if (m > 1) {
		int ans = prefDays[m - 2];
	}
	ans += d;
	/*for (int i = 0; i <= m - 2; ++i) {
		ans += days[i];*/
	return ans;
}
pair <int, int> daysToDate(int x) {
	int m = 1, d = 0;
	for (int i = 0; i < 12; ++i) {
		if (x <= days[i]) {
			d = x;
			return {m, x};
		}
		else {
			m++;
			x -= days[i];
		}
	}
}
int main() {
	prefDays[0] = days[0];
	for (int i = 1; i < 12; ++i) {
		prefDays[i] = prefDays[i - 1] + days[i];
	}
}
```

Проверка на високосность года:
```cpp
if year % 400 == 0 or (year % 4 == 0 and year % 100 != 0):
    print('Yes')
else:
    print('No')
```

Посчитать кол-во високосных лет с одного заданного года до другого можно так:
```(year1 - year2) // 4 - (year1 - year2) // 100 + (year1 - year2) // 400```

Определение дня недели по номеру дня в году (n - номер дня, ng - день недели 1 января)
```dn = (ng - 2 + n) % 7 + 1```