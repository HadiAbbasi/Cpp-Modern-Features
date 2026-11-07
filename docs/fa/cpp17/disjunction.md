<div align="center">

[🇺🇸 English](../../en/cpp17/disjunction.md) | [🇮🇷 فارسی](./disjunction.md)

</div>

---

#  std::disjunction 

`std::disjunction` یکی از ابزارهای کتابخانه استاندارد C++ است که از استاندارد C++17 معرفی شده و در کتابخانه Type Traits قرار دارد.

از این ابزار برای انجام عملیات منطقی OR روی چندین Type Trait در زمان کامپایل (Compile Time) استفاده می‌کنیم.

**به زبان ساده:**

- اگر حداقل یکی از شرط‌ها `true` باشد، نتیجه `true` خواهد بود.
- فقط زمانی نتیجه `false` است که تمام شرط‌ها `false` باشند.

##  معرفی std::disjunction

ابتدا هدر زیر را اضافه می‌کنیم:

```cpp
#include <type_traits>
```

ساختار کلی:

```cpp
std::disjunction<Trait1, Trait2, Trait3, ...>
```

هر آرگومان باید یک Type باشد که دارای مقدار منطقی `value` در زمان کامپایل باشد؛ مانند:

- `std::true_type`
- `std::false_type`
- `std::is_integral<T>`
- `std::is_floating_point<T>`

##  یک مثال ساده

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::disjunction<
        std::is_integral<int>,
        std::is_floating_point<int>
    >;

    std::cout << Result::value << '\n';

}
```

**خروجی:**

```txt
1
```

**توضیح:**

- `std::is_integral<int>` نتیجه `true` دارد.
- `std::is_floating_point<int>` نتیجه `false` دارد.
- `std::disjunction` این دو نتیجه را با OR ترکیب می‌کند.

**بنابراین:**

```txt
true || false == true
```

پس نتیجه نهایی `true` است.

## مثال با چند شرط False

```cpp
#include <iostream>
#include <type_traits>

int main() {

    using Result = std::disjunction<
        std::is_floating_point<int>,
        std::is_array<int>,
        std::is_pointer<int>
    >;

    std::cout << Result::value << '\n';

}
```

**خروجی:**

```txt
0
```

**توضیح:**

- `std::is_floating_point<int>` برابر `false` است.
- `std::is_array<int>` برابر `false` است.
- `std::is_pointer<int>` برابر `false` است.

**بنابراین:**

```txt
false || false || false == false
```

نتیجه نهایی `false` خواهد بود.

## استفاده از چندین شرط

یکی از قابلیت‌های مهم `std::disjunction` این است که می‌توانیم تعداد زیادی شرط را با هم ترکیب کنیم.

مثلاً می‌خواهیم بررسی کنیم که یک نوع، Integral یا Floating Point یا Pointer باشد.

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void check() {

    using Result = std::disjunction<
        std::is_integral<T>,
        std::is_floating_point<T>,
        std::is_pointer<T>
    >;

    if constexpr (Result::value) {

        std::cout << "Supported type\n";

    } else {

        std::cout << "Unsupported type\n";

    }
}

int main() {

    check<int>();
    check<double>();
    check<int*>();
    check<std::string>();

}
```

**خروجی:**

```txt
Supported type
Supported type
Supported type
Unsupported type
```

**توضیح:**

در این مثال حداقل یکی از سه شرط باید درست باشد:

1. آیا نوع Integral است؟
2. آیا نوع Floating Point است؟
3. آیا نوع Pointer است؟

اگر حتی یکی از این شرط‌ها درست باشد، `std::disjunction` مقدار `true` برمی‌گرداند.

##  استفاده از std::disjunction در Template

حالا یک مثال کاربردی‌تر بررسی کنیم.

فرض کنید می‌خواهیم یک Template بنویسیم که فقط انواع Integral و Floating Point را قبول کند.

```cpp
#include <iostream>
#include <type_traits>
#include <string>

template<typename T>
void process(T value) {

    static_assert(
        std::disjunction<
            std::is_integral<T>,
            std::is_floating_point<T>
        >::value,
        "T must be an arithmetic type"
    );

    std::cout << value << '\n';
}

int main() {

    process<int>(100);
    process<double>(10.5);

    // process<std::string>("Hello");
    // Compilation error

}
```

**توضیح:**

در این مثال دو شرط داریم:

- `std::is_integral<T>`
- `std::is_floating_point<T>`

برای عبور از `static_assert` کافی است یکی از این دو شرط درست باشد.

**بنابراین:**

| Type          | Integral | Floating Point | Result |
| ------------- | -------- | -------------- | ------ |
| `int`         | true     | false          | true   |
| `double`      | false    | true           | true   |
| `float`       | false    | true           | true   |
| `std::string` | false    | false          | false  |

## شکل کوتاه‌تر: std::disjunction_v

در C++17 یک نسخه کوتاه‌تر به نام `std::disjunction_v` داریم.

به‌جای نوشتن:

```cpp
std::disjunction<
    std::is_integral<int>,
    std::is_floating_point<int>
>::value
```

می‌توانیم بنویسیم:

```cpp
std::disjunction_v<
    std::is_integral<int>,
    std::is_floating_point<int>
>
```

**مثال کامل:**

```cpp
#include <iostream>
#include <type_traits>

int main() {

    constexpr bool result =
        std::disjunction_v<
            std::is_integral<int>,
            std::is_floating_point<int>
        >;

    std::cout << result << '\n';

}
```

**خروجی:**

```
1
```

##  یک ویژگی مهم: Short-Circuit Evaluation

یکی از ویژگی‌های مهم `std::disjunction`، قابلیت Short-Circuit در زمان کامپایل است.

یعنی اگر اولین شرط `true` باشد، برای تعیین نتیجه نیازی به Instantiate کردن Type Traitهای بعدی نیست.

**مثال:**

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
struct Check {

    static_assert(sizeof(T) == 0,
                  "Check was instantiated!");

    static constexpr bool value = false;
};

using Result = std::disjunction<
    std::true_type,
    Check<int>
>;

int main() {

    std::cout << Result::value;

}
```

**خروجی:**

```
1
```

چرا خطای کامپایل دریافت نمی‌کنیم؟

چون اولین آرگومان `std::true_type` است.

بنابراین `std::disjunction` برای تعیین نتیجه نیازی به Instantiate کردن `Check<int>` ندارد.

نکته مهم

این ویژگی در موارد زیر کاربرد زیادی دارد:

- Template Programming
- SFINAE
- Type Validation
- Compile-Time Programming

##  تفاوت conjunction ،disjunction و negation

حالا که هر سه ابزار را بررسی کرده‌ایم، تفاوت آن‌ها را ببینیم:

| Type Trait         | عملگر منطقی   | مفهوم                       |
| ------------------ | ------------- | --------------------------- |
| `std::conjunction` | AND (`&&`)    | تمام شرط‌ها باید درست باشند |
| `std::disjunction` | OR (`\\|\\|`) | حداقل یک شرط باید درست باشد |
| `std::negation`    | NOT (`!`)     | نتیجه منطقی را معکوس می‌کند |

### نمایش ریاضی:

**Conjunction:**

\\[ A \land B \\]

Disjunction:

\\[ A \lor B \\]

Negation:

\\[ \neg A \\]

اکنون سه ابزار مهم منطق کامپایل‌تایم در C++17 را یاد گرفته‌ای:

1. `std::conjunction` برای AND
2. `std::disjunction` برای OR
3. `std::negation` برای NOT


---

## 🤝 مشارکت کنندگان

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](https://t.me/ad1mi2n) |

</div>
