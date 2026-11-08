
<div align="center">

[🇺🇸 English](../../en/cpp17/conjunction.md) | [🇮🇷 فارسی](./conjunction.md)

</div>


#  std::conjunction در C++17

`std::conjunction` یکی از ابزارهای کتابخانه استاندارد C++ است که از استاندارد C++17 معرفی شده و در کتابخانه Type Traits قرار دارد.

از این ابزار برای انجام عملیات منطقی AND روی چندین Type Trait در زمان کامپایل (Compile Time) استفاده می‌کنیم.

**:به زبان ساده**

- اگر تمام شرط‌ها `true` باشند، نتیجه `true` خواهد بود.
- اگر حداقل یکی از شرط‌ها `false` باشد، نتیجه `false` خواهد بود.

## معرفی std::conjunction

ابتدا باید هدر زیر را اضافه کنیم:

```cpp
#include <type_traits>
```

**ساختار کلی:**

```cpp
std::conjunction<Trait1, Trait2, Trait3, ...>
```

هر آرگومان باید یک Type باشد که مقدار منطقی آن در زمان کامپایل مشخص باشد؛ مانند `std::true_type`، `std::false_type` یا سایر Type Traitها.

##  یک مثال ساده

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::conjunction<
        std::is_integral<int>,
        std::is_signed<int>
    >;

    std::cout << Result::value << '\n';

}
```

**خروجی:**

```txt
1
```

**توضیح:**

- `std::is_integral<int>` بررسی می‌کند که آیا `int` یک نوع عدد صحیح است؟ نتیجه `true`.
- `std::is_signed<int>` بررسی می‌کند که آیا `int` علامت‌دار است؟ نتیجه `true`.
- `std::conjunction` این دو نتیجه را با AND ترکیب می‌کند.

**بنابراین:**

```txt
true && true == true
```

##  مثال با یک شرط False

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::conjunction<
        std::is_integral<int>,
        std::is_floating_point<int>
    >;

    std::cout << Result::value << '\n';

}
```

**خروجی:**

```txt
0
```

**توضیح:**

- `std::is_integral<int>` مقدار `true` برمی‌گرداند.
- `std::is_floating_point<int>` مقدار `false` برمی‌گرداند.

**در نتیجه:**

```txt
true && false == false
```

پس نتیجه نهایی `false` است.

##  استفاده از چندین شرط

یکی از مزیت‌های `std::conjunction` این است که می‌توانیم چندین Type Trait را با هم ترکیب کنیم.

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::conjunction<
        std::is_integral<int>,
        std::is_signed<int>,
        std::is_arithmetic<int>,
        std::is_object<int>
    >;

    std::cout << Result::value << '\n';

}
```

**خروجی:**

```txt
1
```

چون هر چهار شرط درست هستند، نتیجه نهایی `true` خواهد بود.

##  استفاده از std::conjunction در Template

حالا یک مثال کاربردی‌تر را بررسی کنیم.

فرض کنید می‌خواهیم یک تابع Template بنویسیم که فقط انواع عدد صحیح علامت‌دار (Signed Integral Types) را قبول کند.

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void process(T value) {

    static_assert(
        std::conjunction<
            std::is_integral<T>,
            std::is_signed<T>
        >::value,
        "T must be a signed integral type"
    );

    std::cout << value << '\n';
}

int main() {

    process<int>(100);

    // process<unsigned int>(100); // Compilation error
    // process<double>(10.5);      // Compilation error

}
```

**توضیح:**

`std::conjunction` دو شرط را بررسی می‌کند:

1. آیا `T` یک نوع Integral است؟
2. آیا `T` یک نوع Signed است؟

هر دو شرط باید درست باشند.

اگر حتی یکی از شرط‌ها نادرست باشد، `static_assert` باعث خطای کامپایل می‌شود.

## شکل کوتاه‌تر: std::conjunction_v

در C++17 یک نسخه کوتاه‌تر به نام `std::conjunction_v` نیز وجود دارد.

به‌جای نوشتن:

```cpp
std::conjunction<
    std::is_integral<int>,
    std::is_signed<int>
>::value
```

می‌توانیم بنویسیم:

```cpp
std::conjunction_v<
    std::is_integral<int>,
    std::is_signed<int>
>
```

مثال کامل:

```cpp
#include <iostream>
#include <type_traits>

int main() {

    constexpr bool result =
        std::conjunction_v<
            std::is_integral<int>,
            std::is_signed<int>
        >;

    std::cout << result << '\n';

}
```

**خروجی:**

```txt
1
```

##  یک ویژگی مهم: Short-Circuit Evaluation

یکی از جالب‌ترین ویژگی‌های `std::conjunction` قابلیت Short-Circuit در زمان کامپایل است.

یعنی اگر اولین شرط `false` باشد، برای تعیین نتیجه نیازی به Instantiate کردن Type Traitهای بعدی ندارد.

**مثال:**

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
struct Check {

    static_assert(sizeof(T) == 0,
                  "Check was instantiated!");

    static constexpr bool value = true;
};

using Result = std::conjunction<
    std::false_type,
    Check<int>
>;

int main() {

    std::cout << Result::value;

}
```

**خروجی:**

```txt
0
```

چرا خطای کامپایل دریافت نمی‌کنیم؟

چون اولین آرگومان `std::false_type` است.

بنابراین `std::conjunction` برای تعیین نتیجه نیازی به Instantiate کردن `Check<int>` ندارد.

**نکته مهم**

این ویژگی در موارد زیر بسیار کاربردی است:

- Template Programming
- SFINAE
- Type Validation
- Compile-Time Programming

##  تفاوت conjunction ،disjunction و negation

این سه ابزار از عملیات منطقی اصلی در C++17 هستند.

| Type Trait         | عملگر منطقی   | مفهوم                       |
| ------------------ | ------------- | --------------------------- |
| `std::conjunction` | AND (`&&`)    | تمام شرط‌ها باید درست باشند |
| `std::disjunction` | OR (`\\|\\|`) | حداقل یک شرط باید درست باشد |
| `std::negation`    | NOT (`!`)     | نتیجه منطقی را معکوس می‌کند |

**نمایش ریاضی:**

Conjunction:

\\[ A \land B \\]

Disjunction:

\\[ A \lor B \\]

Negation:

\\[ \neg A \\]

## پیشنهاد برای ادامه یادگیری

بعد از یادگیری `std::conjunction` بهتر است سراغ `std::disjunction` برویم.

وقتی این سه ابزار را یاد بگیری:

1. `std::conjunction`
2. `std::disjunction`
3. `std::negation`

پایه بسیار خوبی برای درک منطق بولی در زمان کامپایل (Compile-Time Boolean Logic) در C++17 خواهی داشت؛ مخصوصاً برای Template Programming پیشرفته و SFINAE.

---

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>