<div align="center">

[🇺🇸 English](../../en/cpp23/if_consteval.md) | [🇮🇷 فارسی](./if_consteval.md)

</div>

---

# **بررسی جامع if consteval در ++C23**

## **مقدمه**

قابلیت `if consteval` یکی از ویژگی‌های مهم استاندارد ++C23 است که امکان تشخیص مستقیم Context ارزیابی یک عبارت را در سطح زبان فراهم می‌کند. با استفاده از این قابلیت می‌توان مشخص کرد که یک تابع در حال اجرا در زمان Compile-time است یا در زمان Runtime و بر اساس آن، مسیر اجرایی متفاوتی را انتخاب کرد.

این ویژگی به‌خصوص هنگام طراحی توابع `constexpr`، پیاده‌سازی الگوریتم‌های قابل استفاده در زمان کامپایل، طراحی کتابخانه‌های Generic و استفاده از توابع `consteval` اهمیت پیدا می‌کند.

پیش از ++C23، توسعه‌دهندگان معمولاً برای تشخیص Constant Evaluation از تابع `std::is_constant_evaluated()` استفاده می‌کردند. اگرچه این روش همچنان معتبر و کاربردی است، اما محدودیت‌هایی دارد؛ به‌ویژه زمانی که لازم است یک مسیر اجرایی بتواند مستقیماً در Immediate Function Context قرار بگیرد.

قابلیت `if consteval` این مسئله را با معرفی یک ساختار کنترلی اختصاصی در زبان حل می‌کند.

در این مستند، علاوه بر بررسی Syntax و نحوه عملکرد این قابلیت، ارتباط آن با `constexpr`، `consteval`، `std::is_constant_evaluated()` و سایر امکانات Constant Evaluation بررسی می‌شود.

## **مفهوم Constant Evaluation در ++C**

برای درک صحیح `if consteval` ابتدا باید تفاوت میان اجرای Compile-time و Runtime را بشناسیم.

در ++C، برخی عبارت‌ها می‌توانند در زمان کامپایل ارزیابی شوند. برای مثال، یک متغیر `constexpr` باید با یک Constant Expression مقداردهی اولیه شود.

```
constexpr int square(int value) {
    return value * value;
}

constexpr int result = square(5);
```

در مثال بالا، کامپایلر باید مقدار `square(5)` را در زمان کامپایل محاسبه کند.

اما همین تابع می‌تواند در زمان Runtime نیز استفاده شود:

```
int main() {
    int value = 5;

    int result = square(value);
}
```

در این حالت، چون مقدار `value` در زمان اجرا مشخص می‌شود، فراخوانی تابع نیز در زمان Runtime انجام می‌شود.

بنابراین، یک تابع `constexpr` لزوماً همیشه در زمان کامپایل اجرا نمی‌شود.

این تفاوت اهمیت زیادی دارد؛ زیرا گاهی لازم است یک تابع برای هر دو نوع ارزیابی، الگوریتم متفاوتی داشته باشد.

برای مثال:

- در Compile-time ممکن است استفاده از الگوریتمی که با محدودیت‌های Constant Expression سازگار است ضروری باشد.
- در Runtime می‌توان از الگوریتمی استفاده کرد که سرعت بیشتری دارد.
- برخی عملیات فقط در Runtime امکان‌پذیر هستند.
- برخی محاسبات به دلیل ماهیتشان بهتر است در زمان کامپایل انجام شوند.

قابلیت `if consteval` دقیقاً برای مدیریت چنین شرایطی طراحی شده است.

## **معرفی if consteval**

قابلیت `if consteval` در استاندارد ++C23 معرفی شده است.

این ساختار یک Selection Statement است که مشخص می‌کند بدنه کدام شاخه باید اجرا شود:

- اگر عبارت در یک Manifestly Constant-Evaluated Context ارزیابی شود، شاخه مربوط به Constant Evaluation انتخاب می‌شود.
- در غیر این صورت، شاخه Runtime انتخاب می‌شود.

### **ساختار دستوری**

ساختار اصلی این قابلیت به شکل زیر است:

```
if consteval {
    // Compile-time path
}
else {
    // Runtime path
}
```

نکته مهم این است که برخلاف `if` معمولی، در این ساختار هیچ Condition دلخواهی نوشته نمی‌شود.

همچنین برخلاف `if constexpr`، کلمه `constexpr` در کنار `if` قرار نمی‌گیرد.

کامپایلر بر اساس Context ارزیابی مشخص می‌کند که کدام شاخه باید اجرا شود.

### **مثال ساده**

```
#include <iostream>

constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}

int main() {
    constexpr int compileTimeResult = calculate(5);

    int value = 5;
    int runtimeResult = calculate(value);

    std::cout << compileTimeResult << '\n';
    std::cout << runtimeResult << '\n';
}
```

خروجی:

```
25
10
```

در مثال بالا، دو فراخوانی از یک تابع انجام شده است.

در فراخوانی اول، تابع در Context مربوط به Constant Evaluation قرار دارد؛ بنابراین شاخه اول اجرا می‌شود.

در فراخوانی دوم، تابع در زمان Runtime ارزیابی می‌شود؛ بنابراین شاخه دوم انتخاب خواهد شد.

نکته مهم این است که انتخاب شاخه بر اساس مقدار آرگومان انجام نمی‌شود، بلکه به Context ارزیابی تابع بستگی دارد.

## **مفهوم Manifestly Constant-Evaluated Context**

یکی از مهم‌ترین مفاهیم مرتبط با `if consteval`، مفهوم Manifestly Constant-Evaluated Context است.

این اصطلاح به Contextهایی اشاره دارد که طبق قواعد استاندارد، ارزیابی عبارت در آن‌ها به‌صورت صریح نیازمند Constant Evaluation است.

برای مثال:

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}

constexpr int result = calculate(10);
```

در این مثال، مقداردهی اولیه متغیر `result` نیازمند Constant Expression است. بنابراین، فراخوانی تابع در یک Manifestly Constant-Evaluated Context انجام می‌شود.

در مقابل:

```
int main() {
    int value = 10;

    int result = calculate(value);
}
```

در این حالت، نیازی به Constant Evaluation وجود ندارد و تابع در Runtime اجرا می‌شود.

### **تفاوت Constant Evaluation با اجرای احتمالی در زمان کامپایل**

یک نکته ظریف در ++C وجود دارد: صرف اینکه کامپایلر بتواند یک عبارت را در زمان کامپایل محاسبه کند، به این معنی نیست که Context آن Manifestly Constant-Evaluated است.

برای مثال، Compiler Optimization ممکن است یک محاسبه را پیش از Runtime انجام دهد. این موضوع به‌تنهایی به معنای Constant Evaluation از دید قواعد زبان نیست.

همچنین برخی Contextها ممکن است تحت قواعد زبان برای بررسی Constant Expression مورد ارزیابی آزمایشی قرار بگیرند.

بنابراین، نباید رفتار `if consteval` را صرفاً بر اساس زمان واقعی اجرای محاسبات توسط کامپایلر تفسیر کرد.

معیار اصلی، قواعد Constant Evaluation در استاندارد زبان است، نه تصمیمات مربوط به Optimization.

## **بررسی Immediate Function Context**

مهم‌ترین تفاوت `if consteval` با روش‌های قدیمی تشخیص Constant Evaluation، ایجاد یک Immediate Function Context در شاخه Constant Evaluation است.

برای درک اهمیت این موضوع باید با مفهوم Immediate Function آشنا شویم.

### **مفهوم Immediate Function**

در ++C20، قابلیت `consteval` معرفی شد.

یک تابع `consteval` تابعی است که فراخوانی آن باید در شرایطی انجام شود که نتیجه آن به‌صورت Constant Expression قابل ارزیابی باشد.

```
consteval int square(int value) {
    return value * value;
}
```

فراخوانی معتبر:

```
constexpr int result = square(5);
```

اما فراخوانی زیر معتبر نیست:

```
int main() {
    int value = 5;

    int result = square(value);
}
```

زیرا مقدار `value` در زمان Runtime مشخص می‌شود.

### **مشکل فراخوانی Immediate Function در توابع constexpr**

فرض کنید تابعی `constexpr` داریم که می‌خواهیم از یک تابع `consteval` داخل آن استفاده کنیم.

```
consteval int square(int value) {
    return value * value;
}

constexpr int calculate(int value) {
    return square(value);
}
```

این پیاده‌سازی معتبر نیست.

دلیل آن این است که پارامتر `value` در زمان تعریف تابع لزوماً یک Constant Expression نیست.

حتی اگر تابع `calculate` در نهایت در زمان کامپایل فراخوانی شود، نمی‌توان به‌صورت عمومی فرض کرد که فراخوانی `square(value)` در هر Context معتبر است.

### **راه‌حل با if consteval**

قابلیت `if consteval` این مشکل را حل می‌کند:

```
consteval int square(int value) {
    return value * value;
}

constexpr int calculate(int value) {
    if consteval {
        return square(value);
    }
    else {
        return value * value;
    }
}
```

در این مثال، شاخه اول یک Immediate Function Context محسوب می‌شود.

به همین دلیل، فراخوانی تابع `square` با پارامتر `value` در این شاخه مجاز است.

در زمان Constant Evaluation، شاخه اول انتخاب می‌شود و فراخوانی تابع `consteval` می‌تواند به‌درستی انجام شود.

در زمان Runtime نیز شاخه دوم اجرا می‌شود و هیچ فراخوانی غیرمجاز به تابع Immediate صورت نمی‌گیرد.

**این قابلیت مهم‌ترین مزیت&#x20;**`if consteval`**&#x20;نسبت به&#x20;**`std::is_constant_evaluated()`**&#x20;است.**

## **معرفی شکل منفی if !consteval**

استاندارد ++C23 علاوه بر شکل مثبت، یک شکل منفی نیز برای این ساختار تعریف کرده است:

```
if !consteval {
    // Runtime path
}
else {
    // Compile-time path
}
```

این Syntax از نظر معنایی معادل ساختار زیر است:

```
if consteval {
    // Compile-time path
}
else {
    // Runtime path
}
```

تفاوت فقط در ترتیب قرارگیری شاخه‌ها است.

در شکل منفی:

- بدنه اصلی مربوط به Runtime است.
- شاخه `else` مربوط به Constant Evaluation است.

### **مثال کاربردی**

```
#include <iostream>

constexpr int calculate(int value) {
    if !consteval {
        std::cout << "Runtime execution\n";

        return value * 2;
    }
    else {
        return value * value;
    }
}

int main() {
    constexpr int compileTimeResult = calculate(5);

    int runtimeResult = calculate(5);

    std::cout << compileTimeResult << '\n';
    std::cout << runtimeResult << '\n';
}
```

خروجی:

```
Runtime execution
25
10
```

در این مثال، دستور `std::cout` فقط در شاخه Runtime قرار دارد.

از آنجا که این شاخه هنگام Constant Evaluation اجرا نمی‌شود، وجود آن مانع استفاده از تابع در Context مربوط به Constant Evaluation نیست.

البته باید توجه داشت که این قابلیت به معنای مجاز شدن تمام عملیات در Constant Evaluation نیست. هر عملیاتی که واقعاً در مسیر Compile-time اجرا شود، همچنان باید با محدودیت‌های Constant Expression سازگار باشد.

### **تفاوت شکل منفی با عملگر منطقی**

در Syntax مربوط به `if !consteval` از عملگر منطقی برای بررسی یک Boolean Expression استفاده نمی‌شود.

این ساختار یک فرم مستقل از Selection Statement است.

همچنین باید توجه داشت که شکل منفی آن از نظر قواعد استاندارد معادل جابه‌جایی شاخه‌های شکل مثبت است؛ بنابراین، شاخه Constant Evaluation همچنان در Immediate Function Context قرار می‌گیرد.

## **تفاوت if consteval و if constexpr**

یکی از اشتباهات رایج در استفاده از قابلیت‌های Modern ++C، یکسان در نظر گرفتن `if consteval` و `if constexpr` است.

این دو ساختار از نظر هدف و رفتار کاملاً متفاوت هستند.

### **معرفی if constexpr**

قابلیت `if constexpr` در استاندارد ++C17 معرفی شد.

این ساختار برای انتخاب شاخه بر اساس یک Compile-time Condition استفاده می‌شود.

```
template <typename T>
constexpr auto getValue(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    }
    else {
        return value;
    }
}
```

در این مثال، انتخاب شاخه بر اساس نوع `T` انجام می‌شود.

اگر نوع `T` یک Integral Type باشد، شاخه اول انتخاب می‌شود.

در غیر این صورت، شاخه دوم انتخاب خواهد شد.

### **مقایسه رفتار دو ساختار**

| ویژگی                            | `if constexpr`                    | `if consteval`                   |
| -------------------------------- | --------------------------------- | -------------------------------- |
| استاندارد معرفی                  | ++C17                             | ++C23                            |
| معیار انتخاب                     | Compile-time Condition            | Context ارزیابی                  |
| نیاز به Boolean Condition        | بله                               | خیر                              |
| ارتباط با Template               | بسیار مهم                         | مستقل از Template                |
| حذف شاخه نامعتبر                 | بله، در شرایط Discarded Statement | خیر                              |
| ایجاد Immediate Function Context | خیر                               | بله، در شاخه Constant Evaluation |
| کاربرد اصلی                      | Compile-time Branching            | تفکیک Compile-time و Runtime     |

### **تفاوت Discarded Statement**

یکی از ویژگی‌های مهم `if constexpr` این است که شاخه‌ای که انتخاب نمی‌شود، در Contextهای مربوط به Template می‌تواند به‌عنوان Discarded Statement در نظر گرفته شود.

برای مثال:

```
#include <type_traits>

template <typename T>
constexpr auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    }
    else {
        return value.length();
    }
}
```

اگر `T` یک نوع Integral باشد، شاخه مربوط به `value.length()` در فرآیند Instantiation به‌عنوان Discarded Statement در نظر گرفته می‌شود.

اما `if consteval` چنین رفتاری ندارد.

هر دو شاخه آن باید از نظر قواعد زبان معتبر باشند. انتخاب شاخه فقط مشخص می‌کند کدام مسیر در زمان اجرا یا Constant Evaluation انتخاب شود.

### **استفاده هم‌زمان از دو قابلیت**

در بعضی طراحی‌ها می‌توان از هر دو قابلیت به‌صورت ترکیبی استفاده کرد.

```
#include <type_traits>

template <typename T>
constexpr auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        if consteval {
            return value * value;
        }
        else {
            return value + value;
        }
    }
    else {
        return value;
    }
}
```

در این مثال:

- `if constexpr` نوع داده را بررسی می‌کند.
- `if consteval` نوع Context ارزیابی را تشخیص می‌دهد.

این ترکیب برای طراحی Templateهای پیچیده و کتابخانه‌های Generic بسیار کاربردی است.

## **مقایسه با std::is_constant_evaluated()**

تابع `std::is_constant_evaluated()` در استاندارد ++C20 و در Header زیر معرفی شده است:

```
#include <type_traits>
```

این تابع مشخص می‌کند که آیا فراخوانی آن در یک Manifestly Constant-Evaluated Context انجام شده است یا خیر.

### **مثال ساده**

```
#include <type_traits>

constexpr int calculate(int value) {
    if (std::is_constant_evaluated()) {
        return value * value;
    }
    else {
        return value + value;
    }
}
```

این تابع از نظر انتخاب شاخه، رفتاری مشابه `if consteval` دارد.

اما یک تفاوت بسیار مهم وجود دارد.

### **مشکل Immediate Function Context**

تابع `std::is_constant_evaluated()` فقط وضعیت Constant Evaluation را گزارش می‌کند.

فراخوانی این تابع باعث ایجاد Immediate Function Context نمی‌شود.

بنابراین، کد زیر معتبر نیست:

```
#include <type_traits>

consteval int square(int value) {
    return value * value;
}

constexpr int calculate(int value) {
    if (std::is_constant_evaluated()) {
        return square(value);
    }
    else {
        return value * value;
    }
}
```

علت این است که شرط `if` فقط یک Boolean Expression است و بدنه آن در Immediate Function Context قرار نمی‌گیرد.

در نتیجه، فراخوانی `square(value)` همچنان با محدودیت‌های معمول Immediate Function مواجه است.

### **پیاده‌سازی صحیح با if consteval**

```
consteval int square(int value) {
    return value * value;
}

constexpr int calculate(int value) {
    if consteval {
        return square(value);
    }
    else {
        return value * value;
    }
}
```

در این نسخه، مشکل برطرف شده است.

### **مقایسه دقیق**

| ویژگی                                                         | `std::is_constant_evaluated()` | `if consteval`   |
| ------------------------------------------------------------- | ------------------------------ | ---------------- |
| معرفی                                                         | ++C20                          | ++C23            |
| نوع قابلیت                                                    | Standard Library Function      | Language Feature |
| تشخیص Constant Evaluation                                     | بله                            | بله              |
| ایجاد Immediate Function Context                              | خیر                            | بله              |
| استفاده در Boolean Expression                                 | بله                            | خیر              |
| مناسب برای انتخاب مسیر اجرایی                                 | بله                            | بله              |
| امکان فراخوانی مستقیم Immediate Function در شاخه Compile-time | خیر                            | بله              |

### **نکته مهم درباره شرط if**

یک اشتباه ظریف در استفاده از `std::is_constant_evaluated()` این است که تصور کنیم کد زیر می‌تواند Runtime و Compile-time را تشخیص دهد:

```
if (std::is_constant_evaluated()) {
    // Compile-time branch
}
```

در حالی که این شرط ممکن است همیشه مقدار `true` داشته باشد.

دلیل آن این است که خود Condition یک Context از نوع Manifestly Constant-Evaluated است.

برای نمونه، عبارت شرطی در یک `if` معمولی که برای ارزیابی شرط باید مقدار Boolean مشخصی داشته باشد، می‌تواند باعث شود `std::is_constant_evaluated()` مقدار `true` گزارش کند.

این رفتار با هدف اولیه تابع مغایرت ظاهری دارد، اما نتیجه مستقیم قواعد استاندارد است.

استفاده از `if consteval` این ابهام را برطرف می‌کند؛ زیرا تشخیص Context بخشی از ساختار زبان است و به ارزیابی یک Boolean Expression وابسته نیست.

## **کاربردهای عملی if consteval**

ارزش واقعی این قابلیت زمانی مشخص می‌شود که در طراحی توابع و کتابخانه‌های قابل استفاده در هر دو Context از آن استفاده کنیم.

### **انتخاب الگوریتم متفاوت برای Compile-time و Runtime**

گاهی یک الگوریتم برای Constant Evaluation مناسب‌تر است، در حالی که الگوریتم دیگری در Runtime عملکرد بهتری دارد.

برای مثال، فرض کنید تابعی برای محاسبه توان عدد صحیح طراحی می‌کنیم.

در Compile-time می‌توان از یک الگوریتم مبتنی بر Integer Arithmetic استفاده کرد.

در Runtime نیز می‌توان از قابلیت‌های بهینه کتابخانه استاندارد بهره برد.

```
#include <cmath>
#include <cstdint>

constexpr std::uint64_t integerPower(
    std::uint64_t base,
    unsigned exponent
) {
    if consteval {
        std::uint64_t result = 1;

        while (exponent > 0) {
            if (exponent & 1U) {
                result *= base;
            }

            base *= base;
            exponent >>= 1U;
        }

        return result;
    }
    else {
        return static_cast<std::uint64_t>(
            std::pow(
                static_cast<double>(base),
                static_cast<double>(exponent)
            )
        );
    }
}

static_assert(integerPower(2, 10) == 1024);
```

در این مثال، شاخه Compile-time از محاسبات Integer استفاده می‌کند.

شاخه Runtime نیز از `std::pow()` استفاده می‌کند.

البته این مثال برای مقادیر ورودی محدود طراحی شده است؛ در کاربردهای واقعی باید Overflow و دقت تبدیل بین انواع عددی نیز مدیریت شود.

نکته مهم این است که انتخاب الگوریتم بر اساس Context انجام می‌شود، نه اینکه کامپایلر صرفاً یک نسخه را Optimize کند.

### **استفاده از توابع consteval در APIهای constexpr**

یکی از مهم‌ترین کاربردهای `if consteval`، ایجاد APIهایی است که هم قابلیت Constant Evaluation داشته باشند و هم در Runtime قابل استفاده باشند.

```
#include <cstddef>

consteval std::size_t compileTimeHash(const char* text) {
    std::size_t hash = 5381;

    while (*text != '\0') {
        hash = hash * 33 + static_cast<unsigned char>(*text);
        ++text;
    }

    return hash;
}

constexpr std::size_t hashText(const char* text) {
    if consteval {
        return compileTimeHash(text);
    }
    else {
        std::size_t hash = 5381;

        while (*text != '\0') {
            hash = hash * 33 + static_cast<unsigned char>(*text);
            ++text;
        }

        return hash;
    }
}

constexpr auto value = hashText("example");
```

در این طراحی، تابع `hashText` یک API عمومی برای هر دو نوع ارزیابی فراهم می‌کند.

در مسیر Compile-time از تابع `consteval` استفاده می‌شود.

در مسیر Runtime نیز محاسبه مستقیماً انجام می‌شود.

این الگو برای طراحی APIهای کتابخانه‌ای که به توابع Immediate نیاز دارند، مفید است.

البته در این مثال دو پیاده‌سازی مشابه وجود دارد. در پروژه‌های واقعی بهتر است در صورت امکان، منطق مشترک در یک تابع قابل استفاده مجدد قرار بگیرد تا از تکرار کد جلوگیری شود.

### **مدیریت عملیات غیرقابل استفاده در Constant Evaluation**

برخی عملیات در زمان Runtime امکان‌پذیر هستند، اما نمی‌توان آن‌ها را در Constant Expression انجام داد.

برای مثال:

- انجام عملیات I/O
- دسترسی به برخی منابع خارجی
- استفاده از APIهایی که `constexpr` نیستند
- انجام برخی عملیات سیستم‌عامل
- استفاده از منابعی که در Constant Evaluation قابل مدیریت نیستند

می‌توان این عملیات را در شاخه Runtime قرار داد.

```
#include <iostream>

constexpr void logMessage(int value) {
    if consteval {
        // No runtime I/O during constant evaluation
    }
    else {
        std::cout << "Value: " << value << '\n';
    }
}

int main() {
    constexpr int value = 10;

    logMessage(value);

    int runtimeValue = 20;

    logMessage(runtimeValue);
}
```

در این مثال، فراخوانی اول در Constant Evaluation انجام می‌شود و شاخه مربوط به I/O اجرا نمی‌شود.

فراخوانی دوم در Runtime انجام می‌شود و پیام در خروجی نمایش داده خواهد شد.

توجه کنید که قرار گرفتن یک عملیات در شاخه Runtime به‌تنهایی آن عملیات را به یک Constant Expression تبدیل نمی‌کند؛ بلکه باعث می‌شود آن عملیات هنگام ارزیابی Compile-time اجرا نشود.

### **استفاده در طراحی کتابخانه‌های Generic**

کتابخانه‌های Generic معمولاً باید بتوانند در شرایط مختلف اجرا شوند.

برای مثال، یک الگوریتم ممکن است:

- در زمان کامپایل از یک پیاده‌سازی ساده و قابل ارزیابی استفاده کند.
- در Runtime از یک پیاده‌سازی بهینه‌تر استفاده کند.
- برای بعضی عملیات از Immediate Functionها کمک بگیرد.

```
#include <cstddef>

consteval std::size_t compileTimeSum(
    const int* values,
    std::size_t size
) {
    std::size_t result = 0;

    for (std::size_t i = 0; i < size; ++i) {
        result += static_cast<std::size_t>(values[i]);
    }

    return result;
}

constexpr std::size_t sum(
    const int* values,
    std::size_t size
) {
    if consteval {
        return compileTimeSum(values, size);
    }
    else {
        std::size_t result = 0;

        for (std::size_t i = 0; i < size; ++i) {
            result += static_cast<std::size_t>(values[i]);
        }

        return result;
    }
}

constexpr int numbers[] = {1, 2, 3, 4, 5};

static_assert(sum(numbers, 5) == 15);
```

این مثال نشان می‌دهد که چگونه می‌توان یک API واحد داشت که در هر دو Context رفتار مناسب خود را انتخاب کند.

## **قواعد مهم زبان و محدودیت‌ها**

با وجود سادگی Syntax، قابلیت `if consteval` دارای چند قانون مهم است که هنگام طراحی API باید به آن‌ها توجه کرد.

### **اجباری بودن Compound Statement**

در شکل مثبت `if consteval`، بدنه شاخه اول باید یک Compound Statement باشد.

بنابراین، نوشتن کد زیر معتبر نیست:

```
constexpr int calculate(int value) {
    if consteval
        return value * value;
    else
        return value + value;
}
```

پیاده‌سازی صحیح:

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}
```

در شکل منفی نیز بدنه اصلی باید Compound Statement باشد:

```
constexpr int calculate(int value) {
    if !consteval {
        return value + value;
    }
    else {
        return value * value;
    }
}
```

### **بررسی معنایی هر دو شاخه**

برخلاف `if constexpr`، شاخه انتخاب‌نشده در `if consteval` به‌صورت عمومی Discard نمی‌شود.

برای مثال:

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return undefinedFunction(value);
    }
}
```

اگر `undefinedFunction` تعریف نشده باشد، کد معتبر نخواهد بود.

حتی اگر شاخه Runtime در یک فراخوانی Constant Evaluation اجرا نشود، کامپایلر همچنان باید قواعد زبان را برای آن بررسی کند.

این موضوع یکی از مهم‌ترین تفاوت‌های `if consteval` و `if constexpr` است.

### **محدودیت در استفاده از Immediate Function**

قابلیت `if consteval` امکان فراخوانی Immediate Function را در شاخه مناسب فراهم می‌کند، اما محدودیت‌های عمومی `consteval` را حذف نمی‌کند.

برای مثال، نتیجه یک Constant Evaluation باید همچنان با قواعد Constant Expression سازگار باشد.

همچنین نمی‌توان یک Immediate Function را به‌گونه‌ای استفاده کرد که محدودیت‌های مربوط به ارجاع، Pointer و خروجی Constant Expression نقض شوند.

بنابراین، `if consteval` یک راهکار برای مدیریت Context است، نه ابزاری برای غیرفعال کردن محدودیت‌های Constant Evaluation.

### **محدودیت‌های Control Flow**

قواعد مربوط به Label، `goto` و انتقال Control Flow در این ساختار نیز اهمیت دارند.

برای مثال، پرش به داخل یکی از شاخه‌های `if consteval` از طریق `goto` یا `switch` مجاز نیست.

همچنین قواعد مربوط به `case`، `default` و Labelهای محلی باید رعایت شوند.

در طراحی کد بهتر است از انتقال‌های پیچیده Control Flow در اطراف این ساختار اجتناب شود.

### **عدم امکان استفاده به‌عنوان Expression**

ساختار `if consteval` یک Statement است، نه یک Expression.

بنابراین، نمی‌توان آن را مانند یک Conditional Operator استفاده کرد.

کد زیر معتبر نیست:

```
constexpr int value =
    if consteval {
        10;
    }
    else {
        20;
    };
```

برای چنین مواردی باید از یک تابع `constexpr` یا ساختارهای معمول Expression استفاده کرد.

### **عدم وابستگی به constexpr بودن تابع**

استفاده از `if consteval` فقط به توابع `constexpr` محدود نیست.

این ساختار یک قابلیت زبانی برای تشخیص Context ارزیابی است.

با این حال، در کاربردهای معمول، بیشترین ارزش آن زمانی دیده می‌شود که داخل توابع `constexpr` قرار بگیرد؛ زیرا این توابع می‌توانند هم در Compile-time و هم در Runtime فراخوانی شوند.

## **نکات پیشرفته در طراحی و پیاده‌سازی**

### **تفاوت انتخاب شاخه با بهینه‌سازی کامپایلر**

باید میان Constant Evaluation و Constant Folding تفاوت قائل شد.

Constant Folding یک Optimization است که کامپایلر می‌تواند برای محاسبه عبارت‌های ثابت انجام دهد.

اما `if consteval` بخشی از قواعد زبان است.

بنابراین، حتی اگر کامپایلر بتواند یک فراخوانی Runtime را هنگام Optimization محاسبه کند، این موضوع لزوماً باعث انتخاب شاخه Compile-time نمی‌شود.

برای مثال:

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}

int main() {
    int result = calculate(5);
}
```

در این مثال، متغیر `result` یک متغیر Runtime است.

حتی اگر کامپایلر بتواند نتیجه نهایی را به‌صورت Optimization محاسبه کند، انتخاب شاخه بر اساس Context زبان انجام می‌شود، نه بر اساس زمان انجام Optimization.

### **استفاده از توابع مشترک برای جلوگیری از تکرار کد**

یکی از مشکلات طراحی با `if consteval`، احتمال تکرار منطق در دو شاخه است.

برای جلوگیری از این مشکل، می‌توان بخش‌های مشترک را در توابع جداگانه قرار داد.

```
constexpr int commonCalculation(int value) {
    return value * value + 10;
}

consteval int compileTimeCalculation(int value) {
    return commonCalculation(value);
}

constexpr int calculate(int value) {
    if consteval {
        return compileTimeCalculation(value);
    }
    else {
        return commonCalculation(value);
    }
}
```

در این طراحی:

- منطق اصلی در `commonCalculation` قرار گرفته است.
- تابع `compileTimeCalculation` یک Immediate Function است.
- تابع `calculate` مسیر مناسب را انتخاب می‌کند.

این روش به نگهداری بهتر کد کمک می‌کند.

### **توجه به Overflow و محدودیت منابع**

Constant Evaluation دارای محدودیت‌های اجرایی و پیاده‌سازی است.

برای مثال، کامپایلرها معمولاً محدودیت‌هایی برای تعداد عملیات، عمق Recursion و منابع مورد استفاده در Constant Evaluation دارند.

این محدودیت‌ها ممکن است بین Compilerها متفاوت باشند.

بنابراین، طراحی الگوریتم‌های Compile-time باید از نظر پیچیدگی زمانی و حافظه نیز بررسی شود.

همچنین استفاده از یک الگوریتم متفاوت برای Compile-time به معنای تضمین سرعت بیشتر نیست.

در بعضی موارد، الگوریتم Runtime می‌تواند به اندازه کافی سریع باشد و ایجاد یک مسیر مستقل Compile-time فقط پیچیدگی پروژه را افزایش دهد.

### **توجه به Side Effectها**

Side Effectها در طراحی توابعی که هر دو نوع ارزیابی را پشتیبانی می‌کنند اهمیت زیادی دارند.

برای مثال، فرض کنید در شاخه Runtime یک متغیر Global تغییر می‌کند، اما شاخه Compile-time چنین کاری انجام نمی‌دهد.

در این حالت، رفتار تابع در دو Context متفاوت خواهد بود.

این تفاوت لزوماً اشتباه نیست، اما باید بخشی از قرارداد API باشد.

توصیه می‌شود رفتار قابل مشاهده تابع در هر دو Context تا حد امکان مشخص و قابل پیش‌بینی طراحی شود.

### **استفاده در Templateهای پیچیده**

قابلیت `if consteval` می‌تواند در کنار Templateها، Concepts و `if constexpr` استفاده شود.

نکته مهم این است که هر قابلیت مسئول یک نوع تصمیم‌گیری متفاوت است:

- Concepts برای محدود کردن مجموعه Typeهای مجاز و بیان Requirements استفاده می‌شوند.
- `if constexpr` برای Compile-time Branching بر اساس Condition استفاده می‌شود.
- `if consteval` برای انتخاب مسیر بر اساس Context ارزیابی استفاده می‌شود.

ترکیب صحیح این امکانات می‌تواند به طراحی APIهای قدرتمند و قابل نگهداری منجر شود.

## **بررسی Feature-Test Macro**

برای بررسی پشتیبانی کامپایلر از این قابلیت می‌توان از Feature-Test Macro استاندارد استفاده کرد.

نام این Macro به شکل زیر است:

```
__cpp_if_consteval
```

مقدار استاندارد آن:

```
202106L
```

برای بررسی پشتیبانی می‌توان از کد زیر استفاده کرد:

```
#if defined(__cpp_if_consteval) && \
    __cpp_if_consteval >= 202106L

#define HAS_IF_CONSTEVAL 1

#else

#define HAS_IF_CONSTEVAL 0

#endif
```

این روش برای پروژه‌هایی که باید با چند Compiler یا چند نسخه از استاندارد ++C سازگار باشند مفید است.

توجه داشته باشید که وجود این Macro باید توسط محیط کامپایل واقعی بررسی شود؛ صرف انتخاب گزینه `-std=c++23` به‌تنهایی تضمین نمی‌کند که تمام قابلیت‌های استاندارد در نسخه مورد استفاده Compiler پیاده‌سازی شده باشند.

## **پشتیبانی کامپایلرها**

قابلیت `if consteval` بخشی از استاندارد ++C23 است.

پشتیبانی از این قابلیت به نسخه کامپایلر بستگی دارد.

برای بررسی وضعیت دقیق پشتیبانی بهتر است از صفحات رسمی Compilerها و جدول‌های Feature Support استفاده شود.

منابع مناسب برای بررسی:

- [GCC C++ Standards Support](https://gcc.gnu.org/projects/cxx-status.html?utm_source=chatgpt.com)
- [Clang C++ Support](https://clang.llvm.org/cxx_status.html?utm_source=chatgpt.com)
- [MSVC C++ Language Conformance](https://learn.microsoft.com/en-us/cpp/overview/visual-cpp-language-conformance?utm_source=chatgpt.com)

برای آزمایش کدها می‌توان از گزینه‌های استانداردسازی زیر استفاده کرد:

**GCC:**

```
g++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o main
```

**Clang:**

```
clang++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o main
```

**MSVC:**

```
cl /std:c++latest /EHsc /W4 main.cpp
```

استفاده از `-std=c++23` یا گزینه معادل آن، زبان هدف را مشخص می‌کند؛ اما قابلیت‌های واقعی موجود همچنان به نسخه Compiler وابسته هستند.

## **اشتباهات رایج در استفاده از if consteval**

در پروژه‌های واقعی، چند اشتباه متداول ممکن است باعث رفتار غیرمنتظره یا خطای کامپایل شوند.

### **اشتباه در تشخیص زمان کامپایل**

یکی از اشتباهات رایج این است که تصور شود اگر مقدار ورودی یک تابع ثابت باشد، حتماً شاخه Compile-time انتخاب می‌شود.

```
constexpr int calculate(int value) {
    if consteval {
        return value * value;
    }
    else {
        return value + value;
    }
}

int main() {
    int result = calculate(5);
}
```

در این مثال، فراخوانی تابع همچنان یک فراخوانی Runtime است.

ثابت بودن مقدار آرگومان به‌تنهایی معیار انتخاب شاخه نیست.

### **اشتباه در استفاده از if constexpr به جای if consteval**

اگر هدف تشخیص Context ارزیابی باشد، استفاده از `if constexpr` با یک Condition ثابت یا وابسته به Template راهکار مناسبی نیست.

این دو قابلیت مسئولیت‌های متفاوتی دارند.

### **تصور حذف شدن شاخه انتخاب‌نشده**

برخی توسعه‌دهندگان تصور می‌کنند شاخه انتخاب‌نشده مانند `if constexpr` نادیده گرفته می‌شود.

این تصور اشتباه است.

هر دو شاخه باید از نظر قواعد زبان معتبر باشند.

### **استفاده غیرضروری از دو الگوریتم متفاوت**

داشتن دو مسیر اجرایی همیشه به معنای طراحی بهتر نیست.

اگر یک الگوریتم واحد بتواند در هر دو Context به‌خوبی کار کند، استفاده از آن معمولاً ساده‌تر و قابل نگهداری‌تر است.

### **استفاده از عملیات Runtime در شاخه Compile-time**

قرار دادن یک عملیات Runtime در شاخه‌ای که هنگام Constant Evaluation انتخاب می‌شود، باعث معتبر شدن آن عملیات نمی‌شود.

برای مثال، عملیات I/O همچنان نمی‌تواند بخشی از یک Constant Expression معتبر باشد.

باید مطمئن شد که شاخه Compile-time فقط از عملیات مجاز استفاده می‌کند.

## **چه زمانی از if consteval استفاده کنیم؟**

استفاده از این قابلیت زمانی توصیه می‌شود که واقعاً نیاز به رفتار متفاوت بر اساس Context ارزیابی داشته باشیم.

موارد مناسب عبارت‌اند از:

- طراحی توابع `constexpr` با الگوریتم‌های متفاوت برای Compile-time و Runtime
- استفاده از توابع `consteval` داخل توابع `constexpr`
- ایجاد APIهایی که هم در Compile-time و هم در Runtime قابل استفاده باشند
- جداسازی عملیات Runtime از مسیر Constant Evaluation
- طراحی کتابخانه‌های پیشرفته Generic
- استفاده از قابلیت‌های اختصاصی Immediate Function در مسیر Compile-time

### **چه زمانی از آن استفاده نکنیم؟**

در شرایط زیر بهتر است از این قابلیت استفاده نشود:

- زمانی که یک پیاده‌سازی واحد برای هر دو Context کافی است.
- زمانی که هدف صرفاً بررسی Typeها یا Template Parameters است.
- زمانی که `if constexpr` راهکار ساده‌تر و دقیق‌تری ارائه می‌دهد.
- زمانی که تنها به یک Boolean Indicator از وضعیت Constant Evaluation نیاز داریم.
- زمانی که استفاده از آن باعث تکرار زیاد کد یا پیچیدگی غیرضروری می‌شود.

به‌خصوص اگر تنها هدف بررسی وضعیت Constant Evaluation باشد و نیازی به Immediate Function Context نداشته باشیم، `std::is_constant_evaluated()` همچنان انتخاب مناسبی است.

## **مقایسه قابلیت‌های مرتبط در استانداردهای مختلف**

برای درک جایگاه `if consteval` بهتر است روند تکامل امکانات Constant Evaluation در استانداردهای مختلف را بررسی کنیم.

| قابلیت                         | استاندارد | کاربرد                                                                 |
| ------------------------------ | --------- | ---------------------------------------------------------------------- |
| `constexpr`                    | ++C11     | تعریف متغیرها و توابع قابل استفاده در Constant Evaluation              |
| توسعه قابلیت‌های `constexpr`   | ++C14     | انعطاف بیشتر در پیاده‌سازی توابع Constant Evaluation                   |
| `if constexpr`                 | ++C17     | انتخاب شاخه بر اساس Compile-time Condition                             |
| `std::is_constant_evaluated()` | ++C20     | تشخیص Constant Evaluation                                              |
| `consteval`                    | ++C20     | تعریف Immediate Function                                               |
| `if consteval`                 | ++C23     | انتخاب شاخه بر اساس Context ارزیابی و ایجاد Immediate Function Context |

این امکانات مکمل یکدیگر هستند و هر کدام مسئله متفاوتی را حل می‌کنند.

به همین دلیل، هنگام طراحی کد مدرن نباید آن‌ها را جایگزین مستقیم یکدیگر در نظر گرفت.

## **جمع‌بندی**

قابلیت `if consteval` در استاندارد ++C23 یک ابزار مهم برای کنترل دقیق رفتار توابع در Compile-time و Runtime است.

مهم‌ترین ویژگی این ساختار، ایجاد Immediate Function Context در شاخه مربوط به Constant Evaluation است.

این قابلیت مشکلی را حل می‌کند که با `std::is_constant_evaluated()` به‌تنهایی قابل حل نیست؛ یعنی امکان فراخوانی مستقیم توابع `consteval` با مقادیری که در Immediate Function Context معتبر هستند.

برای استفاده حرفه‌ای از این قابلیت باید تفاوت آن با `if constexpr` و `std::is_constant_evaluated()` را به‌خوبی شناخت.

نکات کلیدی عبارت‌اند از:

- `if consteval` در ++C23 معرفی شده است.
- انتخاب شاخه بر اساس Manifestly Constant-Evaluated Context انجام می‌شود.
- شاخه Compile-time در Immediate Function Context قرار دارد.
- شکل منفی آن با Syntax برابر `if !consteval` نیز پشتیبانی می‌شود.
- برخلاف `if constexpr`، شاخه انتخاب‌نشده به‌صورت عمومی Discard نمی‌شود.
- این قابلیت برای استفاده از Immediate Functionها در توابع `constexpr` بسیار مفید است.
- برای بررسی وضعیت Constant Evaluation بدون نیاز به Immediate Function Context، همچنان می‌توان از `std::is_constant_evaluated()` استفاده کرد.
- استفاده از آن باید بر اساس نیاز واقعی طراحی انجام شود و نباید باعث پیچیدگی غیرضروری کد شود.

در مجموع، `if consteval` یکی از امکانات مهم Modern ++C برای طراحی APIهای قابل استفاده در هر دو Context ارزیابی است و در کنار `constexpr`، `consteval` و `if constexpr` ابزار قدرتمندی برای پیاده‌سازی کتابخانه‌های مدرن و Generic فراهم می‌کند.

## **منابع و مراجع**

برای مطالعه عمیق‌تر جزئیات استاندارد و رفتار این قابلیت، منابع زیر پیشنهاد می‌شوند:

- **مستندات cppreference:**
  - [if statement](https://en.cppreference.com/w/cpp/language/if?utm_source=chatgpt.com)
  - [consteval specifier](https://en.cppreference.com/w/cpp/language/consteval?utm_source=chatgpt.com)
  - [std::is_constant_evaluated](https://en.cppreference.com/w/cpp/types/is_constant_evaluated?utm_source=chatgpt.com)
- **پیشنهاد رسمی استانداردسازی:**
  - [P1938R3 — if consteval](https://www.open-std.org/JTC1/SC22/WG21/docs/papers/2021/p1938r3.html?utm_source=chatgpt.com)
- **مستندات استانداردسازی ++C:**
  - [WG21 — C++ Standards Committee](https://www.open-std.org/jtc1/sc22/wg21/?utm_source=chatgpt.com)
- **مستندات قابلیت‌های کامپایلرها:**
  - [GCC C++ Status](https://gcc.gnu.org/projects/cxx-status.html?utm_source=chatgpt.com)
  - [Clang C++ Status](https://clang.llvm.org/cxx_status.html?utm_source=chatgpt.com)
  - [MSVC C++ Conformance](https://learn.microsoft.com/en-us/cpp/overview/visual-cpp-language-conformance?utm_source=chatgpt.com)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>