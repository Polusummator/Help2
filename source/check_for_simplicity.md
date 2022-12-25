# check_for_simplicity

```py
z = False
x = int(input())
if x % 2 == 0 and x != 2:                    # Проверка, является ли число 2, и делится ли оно на два
    print('NO')
else:
    for i in range(3, int(x ** 0.5) + 1, 2):
        if x % i == 0:
            z = True
            break                            # Если находится хотя бы один делитель, то число уже не является простым
    if z:
        print('NO')
    else:
        print('YES')
```