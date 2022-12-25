# geometry

Геометрия

ТОЧКИ, ВЕКТОРА, ОТРЕЗКИ
Точку проще всего хранить в структуре
```cpp
struct Point {
    int x, y;
    Point() {}
    Point(int x_, int y_): x(x_), y(y_) {}
};
```

Вектор - это сдвиг по двум координатам, поэтому его можно хранить в той же структуре
Для того, чтобы создать вектор, можно переопределить оператор -
```cpp
Point operator-(const Point& a, const Point& b) {
    return Point(a.x - b.x, a.y - b.y);
}
```

Скалярное произведение
Скалярное произведение: a.x * b.x + a.y * b.y = |a| * |b| * cos(a^b)
```cpp
int dP(const Point& a, const Point& b) { // скалярное
    return a.x * b.x + a.y * b.y;
}
```

Векторное произведение
Векторное произведение: a.x * b.y - a.y * b.x = |a| * |b| * sin(a^b)
```cpp
int cP(const Point& a, const Point& b) { // векторное
    return a.x * b.y - a.y * b.x;
}
```

Нахождение угла:
```cpp
double angle(const Point& a, const Point& b) {
    return atan2(cP(a, b), dP(a, b));
}
```

Сравнивать double - плохо, поэтому надо использовать функцию сравнения
```cpp
const double eps = 1e-6;
bool equal(double a, double b) {
    return abs(a - b) < eps;
}
```

Проверка, лежит ли точка внутри угла
```cpp
bool pointInAngle(const Point& a, const Point& o, const Point& b, const Point& p) {
    double aop = abs(angle(a - o, p - o));
    double pob = abs(angle(p - o, b - o));
    double aob = abs(angle(a - o, b - o));
    return equal(aop + pob, aob) || (o.x == p.x && o.y == p.y);
}
```

Длина вектора
```cpp
double len(const Point& a) {
    return hypot(a.x, a.y);
}
```

Проверка, пересекаются ли два отрезка
```cpp
int dP(const Point& a, const Point& b) { // скалярное
    return a.x * b.x + a.y * b.y;
}
int cP(const Point& a, const Point& b) { // векторное
    return a.x * b.y - a.y * b.x;
}
bool inter(int a, int b, int c, int d) {
    if (a > b) {
        swap(a, b);
    }
    if (c > d) {
        swap(c, d);
    }
    return max(a, c) <= min(b, d);
}
istream& operator>>(istream& in, Point& p) {
    in >> p.x >> p.y;
    return in;
}
signed main() {
    Point a, b, c, d;
    cin >> a >> b >> c >> d;
    if (cP(b - a, c - a) * cP(b - a, d - a) <= 0 && cP(d - c, a - c) * cP(d - c, b - c) <= 0 && inter(a.x, b.x, c.x, d.x) && inter(a.y, b.y, c.y, d.y)) {
        Yes;
    }
    else {
        No;
    }
    return 0;
}
```

