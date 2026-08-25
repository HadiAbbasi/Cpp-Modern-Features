<div align="right">

[🇮🇷 فارسی](./list-initialization.md) | [English](../../en/cpp11/list-initialization.md)

</div>
---

# `std::initializer_list` در C++

> در C++، `std::initializer_list` یک **class template** است که اجازه می‌دهد یک شیء سبک‌وزن را با فهرستی از مقادیر مقداردهی اولیه کنیم. از initializer list برای مقداردهی متغیرها، آرایه‌ها، کلاس‌ها، توابع، سازنده‌های کلاس و کانتینرهای استاندارد مانند `std::vector` به شکلی ساده و خوانا استفاده می‌شود.

## C++11

> class templateِ `std::initializer_list` در C++11 اضافه شد و توابع عضوی مانند `size()`، `begin()` و `end()` را برای ساخت، پیمایش و دسترسی به عناصر فراهم می‌کند.

برای استفاده از `initializer_list` باید هدر `<initializer_list>` را در برنامهٔ C++ خود اضافه کنید:

```cpp
std::initializer_list<T> nameOfList{};
```

- برای ساخت شیء `initializer_list` از **braced initializer** یا همان آکولاد `{}` استفاده می‌شود.
- این نوع معمولاً به‌صورت یک wrapper سبک‌وزن روی یک آرایه پیاده‌سازی می‌شود.
- برخلاف کانتینرهای استاندارد مانند `std::vector`، کپی‌کردن یک شیء `initializer_list` باعث کپی‌شدن همهٔ عناصر آن نمی‌شود؛ شیء اصلی و کپی آن به عناصر یکسانی اشاره می‌کنند.

## مثال‌ها

```cpp
#include <initializer_list>
#include <iostream>

int main()
{
    std::initializer_list<int> numbers{2, 4, 6, 8, 10, 12};

    std::cout << "Numbers in the list are: ";

    for (int number : numbers) {
        std::cout << number << ' ';
    }

    return 0;
}
```

```cpp
// C++ program to demonstrate using initializer_list
// for object construction.

#include <initializer_list>
#include <iostream>

template <typename T>
class MyContainer {
public:
    MyContainer(std::initializer_list<T> values)
        : list_{values}
    {
    }

    void printList() const
    {
        for (const T& value : list_) {
            std::cout << value << ' ';
        }

        std::cout << '\n';
    }

private:
    // فقط برای نمونهٔ آموزشی؛ توضیح lifetime را در نکتهٔ زیر ببینید.
    std::initializer_list<T> list_;
};

int main()
{
    MyContainer<int> intContainer{1, 2, 3, 4, 5};

    std::cout << "Elements of integer type are: ";
    intContainer.printList();

    MyContainer<double> doubleContainer{1.1, 2.2, 3.3, 4.4, 5.5};

ؤ    std::cout << "Elements of double type are: ";
    doubleContainer.printList();

    return 0;
}
```

> **نکتهٔ مهم:** یک `std::initializer_list` باید برای استفادهٔ فوری، مانند پیمایش در بدنهٔ یک تابع یا سازنده، استفاده شود. آن را به‌عنوان data member ذخیره نکنید؛ زیرا مالک حافظهٔ عناصر خود نیست و طول عمر داده‌های آن محدود است. در مثال بالا، برای طراحی واقعی بهتر است `list_` از نوع `std::vector<T>` باشد.

## توابع عضو `std::initializer_list`

```cpp
#include <initializer_list>
#include <iostream>

int main()
{
    std::initializer_list<int> numbers{10, 20, 30, 40};

    // 1) begin(): pointer به اولین عنصر
    const int* first = numbers.begin();
    std::cout << "begin(): " << *first << '\n';

    // 2) end(): pointer به یک خانه بعد از آخرین عنصر
    const int* afterLast = numbers.end();
    std::cout << "last element: " << *(afterLast - 1) << '\n';

    // 3) size(): تعداد عناصر
    std::cout << "size(): " << numbers.size() << '\n';

    // 4) empty(): بررسی خالی‌بودن لیست
    if (numbers.empty()) {
        std::cout << "The list is empty.\n";
    } else {
        std::cout << "The list is not empty.\n";
    }

    // 5) data():
    // std::initializer_list در C++11 تابع عضو data() ندارد.
    // begin() در عمل pointer به اولین عنصر آرایهٔ داخلی را می‌دهد.
    const int* data = numbers.begin();

    std::cout << "Underlying data: ";
    for (std::size_t i{0}; i < numbers.size(); ++i) {
        std::cout << data[i] << ' ';
    }
    std::cout << '\n';

    return 0;
}
```

| شماره | نام تابع | توضیح | نمونه کد |
|---:|---|---|---|
| 1 | `begin()` | یک pointer یا iterator به اولین عنصر `initializer_list` برمی‌گرداند. | `const int* first = numbers.begin();` |
| 2 | `end()` | یک pointer یا iterator به **یک موقعیت بعد از آخرین عنصر** برمی‌گرداند. | `const int* afterLast = numbers.end();` |
| 3 | `size()` | تعداد عناصر موجود در initializer list را برمی‌گرداند. | `std::size_t count = numbers.size();` |
| 4 | `empty()` | اگر initializer list خالی باشد `true` و در غیر این صورت `false` برمی‌گرداند. | `bool isEmpty = numbers.empty();` |
| 5 | `data()` | `std::initializer_list` در C++11 تابع عضو `data()` ندارد. برای دریافت pointer به عناصر، از `begin()` استفاده کنید. | `const int* data = numbers.begin();` |

## کاربردهای `initializer_list`

### پارامترهای تابع

می‌توانید `initializer_list` را به‌عنوان پارامتر تابع دریافت کنید:

```cpp
#include <initializer_list>
#include <iostream>

void printNumbers(std::initializer_list<int> numbers)
{
    std::cout << "Size of numbers: " << numbers.size() << '\n';
    std::cout << "Elements: ";

    for (int value : numbers) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
}

int main()
{
    printNumbers({1, 2, 3, 4, 5});
    return 0;
}
```

### استفاده از حافظهٔ پیوسته

عناصر `initializer_list` به‌ترتیب و به‌صورت پیوسته در حافظه قرار دارند. می‌توانید آن‌ها را با range-based loop یا iteratorها پیمایش کنید:

```cpp
#include <initializer_list>
#include <iostream>

int main()
{
    std::initializer_list<int> list{1, 2, 3, 4, 5};

    for (int value : list) {
        std::cout << value << ' ';
    }

    std::cout << '\n';
    return 0;
}
```

### مقداردهی کانتینرهای استاندارد

از initializer list می‌توان برای مقداردهی کانتینرهای استاندارد، مانند `std::vector`، با یک فهرست از عناصر استفاده کرد:

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{1, 2, 3, 4, 5};

    for (int number : numbers) {
        std::cout << number << ' ';
    }

    return 0;
}
```

### بازگرداندن چند مقدار از تابع

> **هشدار:** نباید یک `std::initializer_list` محلی را از تابع برگردانید؛ زیرا دادهٔ پایهٔ آن پس از پایان تابع معتبر نیست. برای بازگرداندن مجموعه‌ای از مقادیر، از کانتینرهای مالک حافظه مانند `std::vector` استفاده کنید.

```cpp
#include <iostream>
#include <vector>

std::vector<int> getNumbers()
{
    return {1, 2, 3, 4, 5};
}

int main()
{
    const auto numbers = getNumbers();

    for (int number : numbers) {
        std::cout << number << ' ';
    }

    return 0;
}
```

## محدودیت‌های `initializer_list`

- **اندازه قابل تغییر نیست:** اندازهٔ `initializer_list` پس از ساخته‌شدن ثابت است. برخلاف کانتینرهایی مانند `std::vector`، قابلیت اضافه‌کردن یا حذف عناصر را ندارد.
- **عناصر فقط خواندنی هستند:** عناصر یک `initializer_list` از نوع `const` هستند و نمی‌توان مقدار آن‌ها را تغییر داد.
- **مالک داده‌ها نیست:** خود `initializer_list` مالک آرایهٔ زیرین نیست؛ بنابراین ذخیره‌کردن آن به‌عنوان عضو کلاس یا بازگرداندن لیست محلی از تابع می‌تواند باعث ایجاد reference یا pointer نامعتبر شود.
- **دسترسی با index ندارد:** خود `initializer_list` عملگر `operator[]` ندارد. با این حال، چون `begin()` یک pointer برمی‌گرداند، می‌توان با احتیاط از pointer arithmetic مانند `*(list.begin() + index)` استفاده کرد؛ البته بهتر است در صورت نیاز به دسترسی اندیسی، از `std::vector` یا `std::array` استفاده شود.

---
## 🤝 مشارکت‌کنندگان

<div align="center">

| گیت‌هاب | لینکدین | ایمیل | وب‌سایت | تلگرام |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](mailto:m.roodsarabi76@gmail.com) | — | [mbr](https://t.me/ad1mi2n) |

</div>
