
[🇺🇸 English](../../en/cpp11/emplace.md) | [🇮🇷 فارسی](./emplace.md)

</div>
# آموزش `emplace` و `emplace_back` در ++C

در ++C، توابع `emplace()` و `emplace_back()` برای درج عناصر در
`std::vector` استفاده می‌شوند. تفاوت اصلی آن‌ها با `insert()` و
`push_back()` این است که **شیء را مستقیماً در محل موردنظر (In-Place)**
می‌سازند و از ایجاد کپی یا انتقال اضافی جلوگیری می‌کنند.

------------------------------------------------------------------------

# `std::vector::emplace()`

تابع `emplace()` یک عنصر را در **موقعیت دلخواه** داخل بردار ایجاد و درج
می‌کند.

## مزیت

به‌جای اینکه ابتدا یک شیء موقت ساخته شود و سپس به بردار منتقل گردد،
آرگومان‌ها مستقیماً به سازنده‌ی کلاس ارسال می‌شوند.

## سینتکس

``` cpp
v.emplace(position, args...);
```

### پارامترها

  پارامتر      توضیح
  ------------ -------------------------
  `position`   محل درج عنصر (Iterator)
  `args...`    آرگومان‌های سازنده‌ی شیء

### مقدار بازگشتی

یک **Iterator** به عنصر درج‌شده برمی‌گرداند.

## مثال ۱: درج در وسط بردار

``` cpp
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 5, 8};
    v.emplace(v.begin() + 2, 6);

    for (int x : v)
        cout << x << " ";
}
```

**خروجی:**

``` text
1 5 6 8
```

## مثال ۲: کلاس سفارشی

``` cpp
#include <vector>
#include <iostream>
using namespace std;

class Point {
public:
    int x, y;

    Point(int a, int b)
        : x(a), y(b) {}
};

int main() {
    vector<Point> points;

    points.emplace(points.begin(), 10, 20);

    cout << points[0].x << " "
         << points[0].y;
}
```

**خروجی:**

``` text
10 20
```

در اینجا شیء `Point` مستقیماً داخل بردار ساخته شده است.

------------------------------------------------------------------------

# `std::vector::emplace_back()`

تابع `emplace_back()` عنصر را **در انتهای بردار** ایجاد می‌کند.

## سینتکس

``` cpp
v.emplace_back(args...);
```

> آرگومان‌ها مستقیماً به سازنده‌ی نوع داده ارسال می‌شوند.

## مثال ۱: اعداد صحیح

``` cpp
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v;

    v.emplace_back(1);
    v.emplace_back(9);
    v.emplace_back(5);

    for (int x : v)
        cout << x << " ";
}
```

**خروجی:**

``` text
1 9 5
```

## مثال ۲: رشته‌ها

``` cpp
#include <vector>
#include <string>
#include <iostream>
using namespace std;

int main() {
    vector<string> words;

    words.emplace_back("سلام");
    words.emplace_back("دنیا");

    for (auto& s : words)
        cout << s << endl;
}
```

**خروجی:**

``` text
سلام
دنیا
```

## مثال ۳: کلاس سفارشی

``` cpp
#include <vector>
#include <iostream>
using namespace std;

class Person {
public:
    string name;
    int age;

    Person(string n, int a)
        : name(n), age(a) {}
};

int main() {
    vector<Person> people;

    people.emplace_back("Ali", 25);

    cout << people[0].name
         << " "
         << people[0].age;
}
```

**خروجی:**

``` text
Ali 25
```

------------------------------------------------------------------------

# تفاوت `emplace` و `insert`

  ویژگی               `emplace()`       `insert()`
  ------------------- ----------------- -------------------
  محل درج             هر موقعیت         هر موقعیت
  ساخت شیء            درجا (In-Place)   نیاز به شیء آماده
  کارایی              بیشتر             کمتر
  مناسب برای کلاس‌ها   ✅                ⚠️

### مقایسه

``` cpp
// insert
Point p(1,2);
v.insert(v.begin(), p);

// emplace
v.emplace(v.begin(), 1, 2);
```

در حالت دوم، شیء `Point` مستقیماً داخل بردار ساخته می‌شود.

------------------------------------------------------------------------

# تفاوت `emplace_back` و `push_back`

  ویژگی         `emplace_back()`   `push_back()`
  ------------- ------------------ ---------------
  محل درج       انتهای بردار       انتهای بردار
  ساخت شیء      درجا               کپی یا Move
  کارایی        بیشتر              کمتر
  نیاز به شیء   خیر                بله

### مقایسه

``` cpp
// push_back
v.push_back(Person("Ali", 20));

// emplace_back
v.emplace_back("Ali", 20);
```

در `emplace_back` شیء موقت ساخته نمی‌شود.

------------------------------------------------------------------------

# چه زمانی از هرکدام استفاده کنیم؟

  -----------------------------------------------------------------------
  اگر می‌خواهید...                     تابع مناسب
  ----------------------------------- -----------------------------------
  درج در وسط بردار                    `emplace()`

  درج در انتهای بردار                 `emplace_back()`

  شیء آماده دارید                     `insert()` یا `push_back()`

  می‌خواهید بیشترین کارایی را داشته    `emplace()` / `emplace_back()`
  باشید                               
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# جمع‌بندی

-   **`emplace()`**: درج عنصر در هر موقعیت با ساخت مستقیم شیء.
-   **`emplace_back()`**: درج عنصر در انتهای بردار با ساخت مستقیم شیء.
-   برای انواع پیچیده (کلاس‌ها و ساختارها)، معمولاً `emplace` و
    `emplace_back` از نظر کارایی انتخاب بهتری نسبت به `insert` و
    `push_back` هستند، زیرا از ایجاد اشیای موقت و عملیات کپی/انتقال
    اضافی جلوگیری می‌کنند.

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | m.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>