# gcdex

Расширенный алгоритм Евклида

Алгоритм позволяет искать решения уравнения ax + by = c

```cpp
int gcd(int a, int b, int& x, int& y) {
    if (a == 0) {
        x = 0;
        y = 1;
        return b;
    }
    int x1, y1;
    int g = gcd(b % a, a, x1, y1);
    x = y1 - (b / a) * x1;
    y = x1;
    return g;
}

signed main() {
    int a, b, c, x, y;
    cin >> a >> b >> c;
    int signa = 1, signb = 1;
    if (a < 0) {
        a *= -1;
        signa = -1;
    }
    if (b < 0) {
        b *= -1;
        signb = -1;
    }
    int g = gcd(a, b, x, y);
    if (c % g != 0) {
        cout << "Impossible" << endl;
        return 0;
    }
    x *= (c / g) * signa;
    y *= (c / g) * signb;
    cout << ' ' << x << ' ' << y << endl;
    return 0;
}
```