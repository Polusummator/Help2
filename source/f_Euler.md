# f_Euler

Функция Эйлера

Функция Эйлера вычисляет кол-во чисел от 1 до n, взаимно простых с n

```py
def fi(n):
    ans = n
    i = 2
    while i * i <= n:
        if n % i == 0:
            while n % i == 0:
                n //= i
            ans -= ans // i
        i += 1
    if n > 1:
        ans -= ans // n
    print(ans)
```

Асимптотика - O(sqrt(n))