<div align="right">

[🇺🇸 English](../../en/cpp11/decltype.md) | [🇮🇷 فارسی](./decltype.md)

</div>

# `decltype` در ++C

`decltype` یک **کلیدواژه (Keyword)** است که در استاندارد **C++11** معرفی شد. وظیفه‌ی آن تشخیص نوع (**Type**) یک عبارت در زمان کامپایل است. اگرچه می‌توان در برخی موارد به جای `auto` از آن استفاده کرد، اما کاربرد اصلی آن تعیین **نوع بازگشتی توابع** و **استخراج نوع یک عبارت** است.

> **تعریف:**  
> `decltype` نوع یک عبارت (Expression) را در زمان کامپایل تعیین کرده و همان نوع را برای اعلان متغیر یا نوع بازگشتی استفاده می‌کند.

## چرا از `decltype` استفاده کنیم؟

از `decltype` زمانی استفاده می‌کنیم که:

- نوع یک عبارت را بدون نوشتن مستقیم آن به دست آوریم.
- نوع بازگشتی تابع به پارامترها وابسته باشد.
- با **Lambda**‌ها کار کنیم.
- در **Template**‌ها نوع نتیجه به نوع پارامترها بستگی داشته باشد.

---

# مثال ساده

### بدون `decltype`

```cpp
int sum(int a, int b)
{
    using ret = int;
    ret x = a + b;
    return x;
}
```

### با `decltype`

```cpp
int sum(int a, int b)
{
    using ret = decltype(a + b);
    ret x = a + b;
    return x;
}
```

کامپایلر عبارت `a + b` را بررسی می‌کند و تشخیص می‌دهد که نوع آن `int` است؛ بنابراین `ret` معادل `int` خواهد بود.

---

# اعلان متغیر با `decltype`

## مثال ۱: متغیرهای `char` و `int`

```cpp
#include <iostream>
#include <typeinfo>
using namespace std;

int main() {

    char c;
    decltype(c) c1 = 10;

    cout << c1 << endl;
    cout << typeid(c1).name() << endl;

    int a;
    decltype(a) b = 10;

    cout << typeid(b).name() << endl;

    return 0;
}
```

### خروجی

```text
10
char
int
```

در این مثال:

- `decltype(c)` → نوع `char`
- `decltype(a)` → نوع `int`

---

## مثال ۲: استفاده با Lambda

```cpp
#include <iostream>
using namespace std;

int main() {

    auto f = [](int a, int b) -> int {
        return a * b;
    };

    decltype(f) c = f;

    cout << c(2, 3);

    return 0;
}
```

### خروجی

```text
6
```

در اینجا `decltype(f)` دقیقاً نوع شیء Lambda را استخراج می‌کند.

---

## مثال ۳: استفاده در Template

```cpp
#include <iostream>
using namespace std;

template <typename T>
T add(T a, T b) {
    return a + b;
}

template <typename T, typename U>
auto add1(T a, U b) -> decltype(a + b) {
    return a + b;
}

int main() {

    decltype(add(2, 3)) a = 1;
    decltype(add(2.1, 3.1)) b = 2.3;

    cout << a + b << endl;

    decltype(add1(2, 1.1)) c = 3;

    return 0;
}
```

در این مثال:

| عبارت | نوع |
|--------|-----|
| `decltype(add(2,3))` | `int` |
| `decltype(add(2.1,3.1))` | `double` |
| `decltype(add1(2,1.1))` | `double` |

---

# `decltype(auto)`

گاهی `auto` نوع را به‌درستی حفظ نمی‌کند، اما `decltype(auto)` نوع واقعی عبارت را حفظ می‌کند.

---

## ۱. حفظ `const`

### استفاده از `auto`

```cpp
#include <iostream>
using namespace std;

int main() {

    const int a = 10;

    auto a1 = a;

    return 0;
}
```

در این حالت:

```text
a1 → int
```

ویژگی `const` حذف می‌شود.

### استفاده از `decltype(auto)`

```cpp
#include <iostream>
using namespace std;

int main() {

    const int a = 10;

    decltype(auto) a1 = a;

    return 0;
}
```

در این حالت:

```text
a1 → const int
```

`decltype(auto)` ویژگی `const` را حفظ می‌کند.

---

## ۲. حفظ Rvalue Reference (`int&&`)

### با `auto`

```cpp
#include <iostream>
#include <utility>
using namespace std;

int main() {

    int&& a = 0;

    auto a1 = move(a);

    return 0;
}
```

نوع `a1` برابر است با:

```text
int
```

مرجع (`&&`) از بین می‌رود.

### با `decltype(auto)`

```cpp
#include <iostream>
#include <utility>
using namespace std;

int main() {

    int&& a = 0;

    decltype(auto) a2 = move(a);

    return 0;
}
```

نوع `a2` برابر است با:

```text
int&&
```

در این حالت مرجع Rvalue حفظ می‌شود.

---

## ۳. نوع بازگشتی تابع

### با `auto`

```cpp
#include <iostream>
using namespace std;

auto fun(int& i) {
    return i;
}

int main() {

    int a = 1;

    return 0;
}
```

نوع بازگشتی:

```text
int
```

در حالی که تابع مقدار `int&` برمی‌گرداند.

### با `decltype(auto)`

```cpp
#include <iostream>
using namespace std;

decltype(auto) fun(int& i) {
    return i;
}

int main() {

    int a = 1;

    return 0;
}
```

نوع بازگشتی:

```text
int&
```

در نتیجه مرجع کاملاً حفظ می‌شود.

---

# تفاوت `auto` و `decltype`

| ویژگی | `auto` | `decltype` |
|--------|--------|------------|
| تشخیص نوع متغیر | ✅ | ✅ |
| تشخیص نوع یک عبارت | ❌ | ✅ |
| حفظ `const` | ❌ | ✅ |
| حفظ Reference | ❌ | ✅ |
| مناسب برای Lambda | محدود | ✅ |
| مناسب برای نوع بازگشتی Template | محدود | ✅ |

---

# چه زمانی از `decltype` استفاده کنیم؟

- زمانی که نوع یک **عبارت** را می‌خواهید.
- هنگام نوشتن **Template**‌ها.
- برای تعیین **نوع بازگشتی تابع**.
- هنگام کار با **Lambda**‌ها.
- زمانی که باید `const` یا Reference حفظ شود (`decltype(auto)`).

---

# جمع‌بندی

- `auto` نوع متغیر را از مقدار اولیه تشخیص می‌دهد.
- `decltype` نوع را از **خود عبارت** استخراج می‌کند.
- `decltype(auto)` دقیق‌ترین روش برای حفظ `const`، `&` و `&&` است.
- در برنامه‌های Template و Generic Programming، `decltype` یکی از پرکاربردترین ابزارهای تعیین نوع محسوب می‌شود.



## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | m.roodsarabi76@gmail.com | | [mbr](https://t.me/ad1mi2n) |

</div>