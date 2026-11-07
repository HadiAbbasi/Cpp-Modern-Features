<div align="center">

[🇺🇸 English](../../en/cpp17/negation.md) | [🇮🇷 فارسی](./negation.md)

</div>

# معرفی std::negation

`std::negation` یکی از ابزارهای Type Traits در کتابخانه `<type_traits>` است که برای معکوس کردن نتیجه یک Trait منطقی استفاده می‌شود.
در C++17 داریم:

```cpp
#include <type_traits>
```

ساختار کلی:

```cpp
std::negation<T>
```

این کلاس نتیجه Trait نوع `T` را معکوس می‌کند.

به زبان ساده:

- اگر `T::value` برابر `true` باشد، نتیجه `false` است.
- اگر `T::value` برابر `false` باشد، نتیجه `true` است.

**مثال ساده**

```cpp
#include <iostream>
#include <type_traits>

int main() {

    std::cout << std::is_integral<int>::value << '\n';

    std::cout << std::negation<
        std::is_integral<int>
    >::value << '\n';

}
```

**خروجی:**

```txt
1
0
```

**چرا؟**

- `std::is_integral<int>` بررسی می‌کند آیا `int` یک نوع صحیح است؟ پاسخ `true`.
- `std::negation` نتیجه را معکوس می‌کند؛ پس `false`.

## مثال با float

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::negation<
        std::is_integral<float>
    >;

    std::cout << Result::value << '\n';

}
```

**خروجی:**

```txt
1
```

چون `float` یک نوع integral نیست، نتیجه `is_integral` برابر false است و `negation` آن را به true تبدیل می‌کند.

## استفاده از std::bool_constant

در C++17 می‌توانیم نتیجه را به شکل یک Type ذخیره کنیم:

```cpp
#include <type_traits>
#include <iostream>

int main() {

    using A = std::bool_constant<true>;

    using B = std::negation<A>;

    std::cout << B::value;

}
```

**خروجی:**

```cpp
0
```

## تفاوت std::negation با std::not_fn

**این دو را اشتباه نگیری:**

| ویژگی     | std::negation                   | std::not_fn              |
| --------- | ------------------------------- | ------------------------ |
| نوع       | Type Trait                      | Function Adapter         |
| کاربرد    | معکوس کردن مقدار منطقی یک Trait | معکوس کردن نتیجه یک تابع |
| هدر       | `<type_traits>`                 | `<functional>`           |
| استاندارد | C++17                           | C++17                    |

##  مثال کاربردی‌تر: بررسی نوع غیرصحیح

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void check() {

    if constexpr (std::negation<
        std::is_integral<T>
    >::value) {

        std::cout << "Not an integral type\n";

    } else {

        std::cout << "Integral type\n";

    }
}

int main() {

    check<int>();
    check<double>();
    check<char>();

}
```

**خروجی:**

```txt
Integral type
Not an integral type
Integral type
```

## نکته مهم در C++17

می‌توانی از نسخه کوتاه‌تر زیر نیز استفاده کنی:

```cpp
std::negation_v<T>
```

**مثلاً:**

```cpp
std::cout << std::negation_v<
    std::is_integral<double>
>;
```

این قابلیت از C++17 در دسترس است.

## برای تسلط بیشتر

سه ابزار مهم از Type Traits که بهتر است کنار `std::negation` یاد بگیری:

1. `std::conjunction` — معادل منطقی AND
2. `std::disjunction` — معادل منطقی OR
3. `std::negation` — معادل منطقی NOT

این سه ابزار پایه منطق کامپایل‌تایم در C++17 هستند.


---
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>