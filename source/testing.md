# testing

Тестирование

В основном, тестирование - это генерация тестов
В Python за генерацию отвечает модуль random
Основные функции:
randint(start, end)      # случайное целое число от start до end включительно
choice(l)                # случайный элемент списка l
normalvariate(mu, sigma) # нормальное распределение
random()                 # случайное число от 0 до 1

Некоторые генерации:
1. Правильная скобочная последовательность

    ```py
    n = int(input())
    l = 0
    bal = 0
    s = ''
    for i in range(n):
        if not bal:
            s += '('
            l += 1
            bal += 1
        elif bal + 1 > n - l - 1:
            s += ')'
            l += 1
            bal -= 1
        else:
            par = randint(0, 1)
            if not par:
                s += '('
                l += 1
                bal += 1
            else:
                s += ')'
                l += 1
                bal -= 1
    print(s)
    ```

1. Массив заданной длины и суммы элементов

    ```py
    n = int(input())
    s = int(input())
    border = []
    for i in range(n - 1):
        border.append(randint(0, s))
    border.sort()
    border = [0] + border + [s]
    a = []
    for i in range(1, n + 1):
        a.append(border[i] - border[i - 1])
    print(a)
    ```

2. Массив заданной длины из чисел от 1 до k без повторений
    ```py
    n = int(input())
    k = int(input())
    if k < 5 * n:
        a = list(range(1, k + 1))
        shuffle(a)
        a = a[:n]
        print(a)
    else:
        s = set()
        while len(s) < n:
            s.add(randint(1, k))
        l = list(s)
        shuffle(l)
        print(l)
    ```

ТЕСТИРОВНИЕ НЕПРАВИЛЬНОГО РЕШЕНИЯ ПРИ НАЛИЧИИ МЕДЛЕННОГО ПРАВИЛЬНОГО
Правильное, но медленное решение - slow.py. Неработающее решение - testme.py

Для начала пишем генератор для программ в отдельном файле (например, gen3.py)

Теперь нужно написать программу, которая свяжет всё (генератор, два решения и проверку правильности ответа)
Например, run.py
Тогда программа run.py будет выглядеть так:

```py
import os

test = 1
while True:
    os.system('python gen3.py > input')
    os.system('python testme.py < input > output')
    os.system('python slow.py < input > correct')
    output = [i.strip() for i in open('output').readlines()]
    correct = [i.strip() for i in open('correct').readlines()]
    if output != correct:
        print('Error at test', test)
        break
    print('Test', test, 'is OK')
    print('--------------------')
    test += 1
```

< и > перенаправляют вывод в файл
Тест, неправильный и правильный ответ можно будет посмотреть после тестирования в файлах input, output и correct
