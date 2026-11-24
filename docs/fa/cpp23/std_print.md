<div align="center">

[🇺🇸 English](../../en/cpp23/std_print.md) | [🇮🇷 فارسی](./std_print.md)

</div>

---

# **راهنمای جامع std::print در ++C23**

## **مقدمه**

قابلیت `std::print` یکی از امکانات مهم اضافه‌شده به Standard Library در استاندارد ++C23 است که راهی مدرن، ساده و type-safe برای چاپ متن قالب‌بندی‌شده فراهم می‌کند.

پیش از معرفی این قابلیت، توسعه‌دهندگان ++C معمولاً از دو روش اصلی برای چاپ خروجی استفاده می‌کردند:

- استفاده از `std::cout` و عملگر `operator<<` از طریق `<iostream>`.
- استفاده از توابع C مانند `printf` و `fprintf` از طریق `<cstdio>`.

هر دو روش کاربردهای خاص خود را دارند؛ اما محدودیت‌هایی نیز دارند. برای مثال، زنجیره‌سازی تعداد زیادی از عملگرهای `operator<<` می‌تواند خوانایی کد را کاهش دهد. از طرف دیگر، خانوادهٔ `printf` به format specifierهایی وابسته است که تطبیق نادرست آن‌ها با نوع آرگومان‌ها می‌تواند به رفتار تعریف‌نشده منجر شود.

قابلیت `std::print` با استفاده از زیرساخت قالب‌بندی معرفی‌شده در ++C20، این مشکلات را تا حد زیادی برطرف می‌کند.

مهم‌ترین ویژگی‌های این قابلیت عبارت‌اند از:

- پشتیبانی از type-safe formatting.
- استفاده از format stringهای استاندارد.
- امکان چاپ مستقیم به `stdout` و `stderr`.
- پشتیبانی از `std::ostream`.
- سازگاری با formatterهای سفارشی.
- امکان استفاده از قالب‌بندی عددی، متنی و ساختاریافته.
- کاهش پیچیدگی کدهای مربوط به چاپ خروجی.

نکتهٔ مهم این است که `std::print` یک قابلیت متعلق به Standard Library است، نه یک ویژگی جدید در خود زبان ++C. بنابراین برای استفاده از آن، علاوه بر پشتیبانی کامپایلر از ++C23، به نسخه‌ای از Standard Library نیاز دارید که این قابلیت را پیاده‌سازی کرده باشد.

## **معرفی قابلیت std::print**

### **آشنایی با std::print**

تابع `std::print` برای چاپ داده‌های قالب‌بندی‌شده به مقصد خروجی استفاده می‌شود.

این تابع در header استاندارد `<print>` تعریف شده است.

ساده‌ترین مثال استفاده از آن:

```
#include <print>

int main() {
    std::print("Hello, World!\n");
}
```

خروجی:

```
Hello, World!
```

در مقایسه با `std::cout`، این روش برای چاپ متن ثابت یا ترکیبی از متن و مقادیر متغیر، ساختار ساده‌تری دارد.

```
#include <iostream>
#include <print>

int main() {
    std::string name = "Alice";
    int age = 25;

    std::cout << "Name: " << name
              << ", Age: " << age << '\n';

    std::print("Name: {}, Age: {}\n", name, age);
}
```

در مثال دوم، نیازی به زنجیره‌سازی چندین عملگر خروجی وجود ندارد و ترتیب آرگومان‌ها از طریق placeholderهای موجود در format string مشخص می‌شود.

### **تفاوت std::print و std::println**

در ++C23 دو تابع اصلی برای چاپ قالب‌بندی‌شده معرفی شده‌اند:

- `std::print`
- `std::println`

تفاوت اصلی آن‌ها در اضافه‌کردن newline است.

تابع `std::print` پس از چاپ محتوا، به‌صورت خودکار newline اضافه نمی‌کند.

```
#include <print>

int main() {
    std::print("Hello ");
    std::print("World");
    std::print("\n");
}
```

خروجی:

```
Hello World
```

در مقابل، تابع `std::println` پس از چاپ محتوا، یک newline اضافه می‌کند.

```
#include <print>

int main() {
    std::println("Hello");
    std::println("World");
}
```

خروجی:

```
Hello
World
```

بنابراین:

| تابع           | رفتار                         |
| -------------- | ----------------------------- |
| `std::print`   | چاپ بدون newline خودکار       |
| `std::println` | چاپ همراه با newline در انتها |

برای چاپ پیام‌هایی که هرکدام باید در یک خط مستقل قرار بگیرند، `std::println` معمولاً انتخاب مناسب‌تری است.

نکتهٔ نسخه‌ای: overload بدون آرگومان `std::println()` که صرفاً newline چاپ می‌کند، متعلق به ++C26 است. در ++C23 برای چاپ یک خط خالی می‌توانید از `std::println("")` استفاده کنید.

## **آشنایی با format string**

### **مفهوم placeholder**

یکی از مهم‌ترین بخش‌های `std::print`، استفاده از format string است.

در این ساختار، علامت `{}` به‌عنوان placeholder برای قرارگرفتن مقدار یک آرگومان استفاده می‌شود.

```
#include <print>

int main() {
    std::print("Name: {}, Age: {}\n", "Alice", 25);
}
```

خروجی:

```
Name: Alice, Age: 25
```

در این مثال:

- اولین `{}` با مقدار `"Alice"` جایگزین می‌شود.
- دومین `{}` با مقدار `25` جایگزین می‌شود.

ترتیب آرگومان‌ها به‌صورت پیش‌فرض بر اساس ترتیب ظاهرشدن placeholderها تعیین می‌شود.

### **استفاده از argument indexing**

در format string می‌توان به‌جای indexing خودکار، index آرگومان‌ها را به‌صورت صریح مشخص کرد.

```
#include <print>

int main() {
    std::print("{1} {0}\n", "World", "Hello");
}
```

خروجی:

```
Hello World
```

در این مثال:

- `{0}` به اولین آرگومان اشاره می‌کند.
- `{1}` به دومین آرگومان اشاره می‌کند.

این قابلیت زمانی مفید است که بخواهید ترتیب نمایش داده‌ها با ترتیب ارسال آرگومان‌ها متفاوت باشد.

همچنین می‌توان از یک آرگومان چندین بار استفاده کرد:

```
#include <print>

int main() {
    std::print("{0}, {0}, {1}\n", "Hello", "World");
}
```

خروجی:

```
Hello, Hello, World
```

ترکیب indexing خودکار و indexing صریح در یک format string مجاز نیست.

برای مثال، format string زیر معتبر نیست:

```
#include <print>

int main() {
    // Invalid: automatic and manual indexing are mixed.
    // std::print("{} {1}\n", "Hello", "World");
}
```

### **نمایش آکولادهای واقعی**

گاهی لازم است خود کاراکترهای `{` و `}` در خروجی نمایش داده شوند.

برای این کار باید از آکولادهای دوتایی استفاده کنید.

```
#include <print>

int main() {
    std::print("{{Hello}}\n");
}
```

خروجی:

```
{Hello}
```

بنابراین:

| قالب | نتیجه       |
| ---- | ----------- |
| `{}` | placeholder |
| `{{` | کاراکتر `{` |
| `}}` | کاراکتر `}` |

این قابلیت برای چاپ ساختارهایی مانند JSON، قطعه‌کدها و متن‌هایی که شامل آکولاد هستند کاربرد دارد.

## **قالب‌بندی مقادیر عددی**

یکی از قابلیت‌های مهم format string، امکان تعیین نحوهٔ نمایش اعداد است.

این قابلیت از قواعد `std::format` در ++C20 استفاده می‌کند.

### **قالب‌بندی اعداد صحیح**

برای اعداد صحیح می‌توان نحوهٔ نمایش مقدار را با format specifier مشخص کرد.

```
#include <print>

int main() {
    int value = 255;

    std::println("Decimal: {}", value);
    std::println("Binary: {:b}", value);
    std::println("Octal: {:o}", value);
    std::println("Hexadecimal: {:x}", value);
    std::println("Uppercase Hex: {:X}", value);
}
```

خروجی:

```
Decimal: 255
Binary: 11111111
Octal: 377
Hexadecimal: ff
Uppercase Hex: FF
```

برای نمایش prefix مبنا نیز می‌توان از `#` استفاده کرد.

```
#include <print>

int main() {
    int value = 255;

    std::println("{:#b}", value);
    std::println("{:#o}", value);
    std::println("{:#x}", value);
}
```

خروجی:

```
0b11111111
0377
0xff
```

### **تعیین عرض خروجی**

برای تعیین حداقل عرض میدان خروجی می‌توان از عددی پس از `:` استفاده کرد.

```
#include <print>

int main() {
    int value = 42;

    std::println("|{:5}|", value);
    std::println("|{:10}|", value);
}
```

خروجی:

```
|   42|
|        42|
```

در این حالت، مقدار به‌صورت پیش‌فرض راست‌چین می‌شود.

برای تعیین alignment می‌توان از علامت‌های زیر استفاده کرد:

| Specifier | مفهوم    |
| --------- | -------- |
| `<`       | چپ‌چین   |
| `>`       | راست‌چین |
| `^`       | وسط‌چین  |

مثال:

```
#include <print>

int main() {
    std::println("|{:<10}|", 42);
    std::println("|{:>10}|", 42);
    std::println("|{:^10}|", 42);
}
```

خروجی:

```
|42        |
|        42|
|    42    |
```

### **استفاده از fill character**

علاوه بر alignment، می‌توان کاراکتر پرکننده را نیز مشخص کرد.

```
#include <print>

int main() {
    std::println("{:*<10}", 42);
    std::println("{:*>10}", 42);
    std::println("{:*^10}", 42);
}
```

خروجی:

```
42********
********42
****42****
```

این قابلیت برای ساخت خروجی‌های جدولی و گزارش‌های متنی مفید است.

### **صفرگذاری اعداد**

برای اضافه‌کردن صفر به ابتدای اعداد می‌توان از zero padding استفاده کرد.

```
#include <print>

int main() {
    int number = 42;

    std::println("{:05}", number);
    std::println("{:08}", number);
}
```

خروجی:

```
00042
00000042
```

این روش در نمایش شناسه‌ها، شماره‌های ترتیبی و مقادیر دارای طول ثابت کاربرد دارد.

## **قالب‌بندی اعداد اعشاری**

### **تعیین precision**

برای تعیین تعداد ارقام اعشار می‌توان از precision استفاده کرد.

```
#include <print>

int main() {
    double value = 3.1415926535;

    std::println("{:.2f}", value);
    std::println("{:.4f}", value);
    std::println("{:.6f}", value);
}
```

خروجی:

```
3.14
3.1416
3.141593
```

در این ساختار، `f` نشان‌دهندهٔ fixed-point notation است.

### **استفاده از scientific notation**

برای نمایش اعداد در قالب علمی می‌توان از `e` یا `E` استفاده کرد.

```
#include <print>

int main() {
    double value = 123456.789;

    std::println("{:e}", value);
    std::println("{:E}", value);
    std::println("{:.2e}", value);
}
```

### **نمایش عمومی اعداد اعشاری**

برای استفاده از general format می‌توان از `g` یا `G` استفاده کرد.

```
#include <print>

int main() {
    double value = 123456.789;

    std::println("{:g}", value);
    std::println("{:.4g}", value);
}
```

این قالب‌بندی با توجه به مقدار عدد، بین نمایش fixed و scientific انتخاب می‌کند.

### **نمایش علامت اعداد**

برای کنترل نمایش علامت مثبت و منفی می‌توان از علامت‌های مختلف استفاده کرد.

```
#include <print>

int main() {
    std::println("{:+}", 42);
    std::println("{:+}", -42);
    std::println("{: }", 42);
}
```

علامت `+` باعث می‌شود مقادیر مثبت نیز با علامت مثبت نمایش داده شوند.

## **قالب‌بندی رشته‌ها**

### **تعیین عرض رشته**

برای رشته‌ها نیز می‌توان از width و alignment استفاده کرد.

```
#include <print>

int main() {
    std::println("|{:10}|", "Alice");
    std::println("|{:<10}|", "Alice");
    std::println("|{:^10}|", "Alice");
}
```

### **محدودکردن طول رشته**

با استفاده از precision می‌توان تعداد کاراکترهای نمایش‌داده‌شده را محدود کرد.

```
#include <print>

int main() {
    std::println("{:.5}", "Programming");
    std::println("{:.8}", "Programming");
}
```

خروجی:

```
Progr
Programm
```

این قابلیت برای نمایش خلاصه‌شدهٔ متن‌ها در جدول‌ها و گزارش‌ها کاربرد دارد.

## **چاپ نوع‌های مختلف استاندارد**

قابلیت `std::print` از formatterهای استاندارد برای انواع مختلف داده استفاده می‌کند.

### **چاپ bool**

```
#include <print>

int main() {
    bool enabled = true;

    std::println("{}", enabled);
    std::println("{:d}", enabled);
}
```

خروجی:

```
true
1
```

قالب پیش‌فرض مقدار منطقی را به‌شکل `true` یا `false` نمایش می‌دهد.

### **چاپ کاراکتر**

```
#include <print>

int main() {
    char letter = 'A';

    std::println("{}", letter);
    std::println("{:d}", letter);
}
```

خروجی:

```
A
65
```

در قالب‌بندی عددی، مقدار عددی متناظر با کاراکتر نمایش داده می‌شود.

### **چاپ pointer**

برای نمایش مقدار pointer می‌توان از قالب pointer استفاده کرد.

```
#include <print>

int main() {
    int value = 42;
    int* pointer = &value;

    std::println("Pointer: {}", static_cast<const void*>(pointer));
}
```

نمایش دقیق آدرس به پیاده‌سازی و محیط اجرا وابسته است.

### **چاپ nullptr**

برای نمایش مقدار null pointer نیز می‌توان از formatter مربوط استفاده کرد.

```
#include <print>

int main() {
    std::nullptr_t pointer = nullptr;

    std::println("{}", pointer);
}
```

### **چاپ enum**

برای enumها باید به نوع underlying و نحوهٔ ارائهٔ formatter توجه کرد. در صورت نیاز به نمایش عددی، تبدیل صریح معمولاً خواناتر است.

```
#include <print>

enum class Status : int {
    Pending = 0,
    Active = 1,
    Finished = 2
};

int main() {
    Status status = Status::Active;

    std::println("Status: {}", static_cast<int>(status));
}
```

برای نمایش نام معنایی enum بهتر است formatter اختصاصی تعریف شود.

## **ارتباط std::print با std::format**

### **تفاوت در خروجی**

قابلیت `std::format` در ++C20 برای تولید رشتهٔ قالب‌بندی‌شده طراحی شده است.

در مقابل، `std::print` در ++C23 نتیجه را مستقیماً به مقصد خروجی می‌نویسد.

مثال با `std::format`:

```
#include <format>
#include <string>

int main() {
    std::string message = std::format(
        "Name: {}, Age: {}",
        "Alice",
        25
    );
}
```

مثال با `std::print`:

```
#include <print>

int main() {
    std::print("Name: {}, Age: {}\n", "Alice", 25);
}
```

تفاوت کاربردی:

| قابلیت                       | std::format         | std::print |
| ---------------------------- | ------------------- | ---------- |
| تولید std::string            | بله                 | خیر        |
| چاپ مستقیم                   | خیر                 | بله        |
| استفاده از format specifier  | بله                 | بله        |
| پشتیبانی از formatter سفارشی | بله                 | بله        |
| مناسب برای ذخیرهٔ نتیجه      | بله                 | خیر        |
| مناسب برای چاپ مستقیم        | نیازمند خروجی اضافی | بله        |

اگر به رشتهٔ نهایی برای ذخیره‌سازی، پردازش یا انتقال به یک API دیگر نیاز دارید، `std::format` مناسب است.

اگر هدف صرفاً چاپ خروجی باشد، `std::print` معمولاً انتخاب ساده‌تری خواهد بود.

### **استفاده از std::vformat برای format string پویا**

در حالت معمول، `std::print` از `std::format_string` استفاده می‌کند تا امکان بررسی format string در زمان کامپایل فراهم شود.

اما گاهی format string در زمان اجرا مشخص می‌شود؛ برای مثال، ممکن است از فایل تنظیمات یا ورودی برنامه دریافت شود.

در ++C23 می‌توان از `std::vformat` برای این سناریو استفاده کرد.

```
#include <format>
#include <print>
#include <string>

int main() {
    std::string formatString = "Value: {}";

    std::string result = std::vformat(
        formatString,
        std::make_format_args(42)
    );

    std::println("{}", result);
}
```

در این روش ابتدا رشتهٔ قالب‌بندی‌شده تولید و سپس چاپ می‌شود.

نکتهٔ مهم این است که `std::make_format_args` آرگومان‌های مربوط به قالب‌بندی را به‌صورت یک ساختار type-erased ارائه می‌کند. این ساختار نباید بیشتر از طول عمر آرگومان‌هایی که به آن وابسته است نگهداری شود.

همچنین format string نامعتبر در این حالت ممکن است هنگام اجرا باعث پرتاب `std::format_error` شود.

## **چاپ به مقصدهای مختلف**

یکی از ویژگی‌های مهم `std::print`، پشتیبانی از مقصدهای مختلف خروجی است.

### **چاپ در stdout**

در حالت پیش‌فرض، خروجی به `stdout` ارسال می‌شود.

```
#include <print>

int main() {
    std::print("Application started\n");
}
```

این رفتار برای برنامه‌های command-line متداول است.

### **چاپ در stderr**

برای چاپ پیام‌های خطا و اطلاعات تشخیصی می‌توان از `stderr` استفاده کرد.

```
#include <cstdio>
#include <print>

int main() {
    std::println(stderr, "Error: File not found");
}
```

استفاده از `stderr` باعث می‌شود خروجی خطا از خروجی عادی برنامه قابل تفکیک باشد.

برای مثال:

```
./app > output.txt 2> errors.txt
```

در این حالت خروجی عادی و خطاها در دو فایل جداگانه ذخیره می‌شوند.

### **چاپ در FILE\***

علاوه بر `stdout` و `stderr`، می‌توان یک `std::FILE*` دیگر را نیز به تابع ارسال کرد.

```
#include <cstdio>
#include <print>

int main() {
    std::FILE* file = std::fopen("output.txt", "w");

    if (!file) {
        return 1;
    }

    std::println(file, "Hello from std::print");
    std::println(file, "Value: {}", 42);

    std::fclose(file);
}
```

در این مثال، خروجی مستقیماً به فایل نوشته می‌شود.

برای مدیریت مطمئن‌تر منابع در برنامه‌های واقعی، استفاده از RAII و wrapperهای مناسب برای `FILE*` توصیه می‌شود تا در صورت وقوع exception نیز فایل بسته شود.

### **چاپ در std::ostream**

در ++C23، overloadهای مربوط به `std::ostream` نیز برای چاپ قالب‌بندی‌شده در دسترس هستند.

```
#include <fstream>
#include <ostream>

int main() {
    std::ofstream file("output.txt");

    if (file) {
        std::print(file, "Name: {}, Age: {}\n", "Alice", 25);
    }
}
```

این قابلیت باعث می‌شود بتوان از `std::print` با streamهایی مانند موارد زیر استفاده کرد:

- `std::ofstream`
- `std::ostringstream`
- `std::stringstream`
- `std::cout`
- `std::cerr`

برای استفاده از این overloadها باید header مربوط به `<ostream>` را نیز در نظر گرفت.

## **پشتیبانی از نوع‌های سفارشی**

یکی از ویژگی‌های قدرتمند زیرساخت `std::format`، امکان تعریف formatter برای نوع‌های کاربرساخته است.

از آنجا که `std::print` از همان زیرساخت استفاده می‌کند، formatterهای سفارشی نیز برای آن قابل‌استفاده هستند.

### **تعریف formatter اختصاصی**

فرض کنید ساختار زیر را داریم:

```
struct Point {
    int x;
    int y;
};
```

به‌صورت پیش‌فرض، کتابخانهٔ استاندارد نمی‌داند که یک شیء از نوع `Point` باید چگونه نمایش داده شود.

برای حل این مسئله باید `std::formatter` را تخصصی‌سازی کنیم.

```
#include <format>
#include <print>

struct Point {
    int x;
    int y;
};

template <>
struct std::formatter<Point> : std::formatter<std::string> {
    auto format(const Point& point, std::format_context& context) const {
        return std::formatter<std::string>::format(
            std::format("({}, {})", point.x, point.y),
            context
        );
    }
};

int main() {
    Point point{10, 20};

    std::println("Position: {}", point);
}
```

خروجی:

```
Position: (10, 20)
```

در این مثال، formatter مربوط به `Point` از formatter رشته استفاده می‌کند تا نمایش متنی شیء را تولید کند.

برای این پیاده‌سازی باید `<string>` نیز به‌صورت مستقیم include شود؛ در کدهای واقعی بهتر است به includeهای غیرمستقیم وابسته نباشید.

### **پیاده‌سازی formatter با کنترل بیشتر**

در مثال قبلی از `std::format` برای تولید رشتهٔ موقت استفاده کردیم.

برای نوع‌هایی که مرتباً قالب‌بندی می‌شوند، می‌توان مستقیماً در output iterator نوشت.

```
#include <format>
#include <print>

struct Point {
    int x;
    int y;
};

template <>
struct std::formatter<Point> {
    constexpr auto parse(std::format_parse_context& context) {
        return context.begin();
    }

    auto format(const Point& point, std::format_context& context) const {
        return std::format_to(
            context.out(),
            "({}, {})",
            point.x,
            point.y
        );
    }
};

int main() {
    Point point{10, 20};

    std::println("Point: {}", point);
}
```

این روش از ساخت رشتهٔ میانی در پیاده‌سازی formatter جلوگیری می‌کند.

نکتهٔ مهم این است که formatter باید قراردادهای استاندارد مربوط به `parse` و `format` را رعایت کند.

### **طراحی formatterهای حرفه‌ای**

در طراحی formatter برای نوع‌های سفارشی بهتر است موارد زیر را در نظر بگیرید:

- نمایش پیش‌فرض باید خوانا و قابل‌پیش‌بینی باشد.
- formatter نباید وضعیت شیء را تغییر دهد.
- عملیات جانبی غیرضروری نباید در آن انجام شود.
- برای نمایش نوع‌های پیچیده می‌توان format specifier اختصاصی تعریف کرد.
- تخصیص حافظهٔ غیرضروری باید در مسیرهای پرتکرار کاهش یابد.
- رفتار formatter باید تا حد امکان deterministic باشد.

برای مثال، formatter یک نوع مالی ممکن است امکان تعیین تعداد ارقام اعشار را فراهم کند؛ درحالی‌که formatter یک نوع هندسی می‌تواند نحوهٔ نمایش مختصات را کنترل کند.

## **بررسی رفتار Unicode**

یکی از موضوعات مهم در استفاده از `std::print`، مدیریت متن Unicode است.

چاپ صحیح متن چندزبانه فقط به API چاپ وابسته نیست و مواردی مانند encoding فایل منبع، encoding مقصد، locale و قابلیت‌های ترمینال نیز اهمیت دارند.

### **مفهوم encoding**

در ++C نوع `char` به‌تنهایی مشخص نمی‌کند که داده‌های آن حتماً UTF-8 هستند.

همچنین نوع `char8_t` که در ++C20 معرفی شده است، برای نمایش واحدهای کد UTF-8 طراحی شده و با `char` تفاوت نوعی دارد.

برای مثال:

```
#include <print>

int main() {
    const char* text = "Hello";
    std::println("{}", text);
}
```

برای متن‌های Unicode باید encoding مورد انتظار مقصد را نیز در نظر بگیرید.

### **رفتار std::print در برابر Unicode**

در طراحی استاندارد این قابلیت، مسیر چاپ Unicode و non-Unicode در نظر گرفته شده است.

به‌خصوص overloadهای مربوط به `FILE*` از سازوکارهای چاپ Unicode استفاده می‌کنند تا در محیط‌های پشتیبانی‌شده، متن با encoding مناسب نوشته شود.

بااین‌حال، نباید تصور کرد که `std::print` هر encoding دلخواهی را به‌صورت خودکار به UTF-8 تبدیل می‌کند.

برای استفادهٔ مطمئن از متن‌های چندزبانه:

- فایل‌های منبع را با encoding مناسب، معمولاً UTF-8، ذخیره کنید.
- encoding مورد انتظار مقصد خروجی را بررسی کنید.
- میان `char` و `char8_t` تفاوت قائل شوید.
- قابلیت نمایش Unicode در ترمینال را بررسی کنید.
- به رفتار locale و محیط اجرا توجه داشته باشید.

همچنین `std::print` مسئول text shaping، الگوریتم BiDi یا انتخاب فونت نیست.

این موضوع برای متن‌های فارسی اهمیت ویژه‌ای دارد؛ زیرا نمایش صحیح متن فارسی به پشتیبانی لایهٔ نمایش از الگوریتم‌های مربوط نیز وابسته است.

## **مدیریت خروجی در برنامه‌های چندریسمانی**

در برنامه‌های multithreaded، چاپ هم‌زمان از چند thread می‌تواند باعث درهم‌آمیختگی پیام‌ها شود.

برای مثال، فرض کنید دو thread به‌صورت هم‌زمان خروجی تولید می‌کنند:

```
#include <print>
#include <thread>

int main() {
    std::thread first([] {
        for (int i = 0; i < 5; ++i) {
            std::println("Thread A: {}", i);
        }
    });

    std::thread second([] {
        for (int i = 0; i < 5; ++i) {
            std::println("Thread B: {}", i);
        }
    });

    first.join();
    second.join();
}
```

در این مثال، ترتیب کلی خطوط خروجی قطعی نیست؛ زیرا زمان‌بندی threadها توسط سیستم‌عامل و runtime تعیین می‌شود.

برای جلوگیری از درهم‌آمیختگی بخش‌های مختلف یک پیام، بهتر است پیام را کامل ساخته و آن را در یک فراخوانی چاپ کنید.

برای کنترل دقیق‌تر نیز می‌توان از synchronization استفاده کرد.

```
#include <format>
#include <mutex>
#include <print>
#include <thread>

std::mutex outputMutex;

void printMessage(int id, int value) {
    std::string message = std::format(
        "Thread {}: {}\n",
        id,
        value
    );

    std::lock_guard lock{outputMutex};

    std::print("{}", message);
}

int main() {
    std::thread first(printMessage, 1, 100);
    std::thread second(printMessage, 2, 200);

    first.join();
    second.join();
}
```

در این نمونه، mutex تضمین می‌کند threadهایی که از همین mutex استفاده می‌کنند، عملیات چاپ خود را به‌صورت هم‌زمان انجام ندهند.

نکتهٔ مهم این است که این mutex فقط دسترسی threadهای همکار را هماهنگ می‌کند و کدهای دیگری که بدون رعایت آن به همان مقصد خروجی می‌نویسند، تحت پوشش آن نیستند.

همچنین نباید فرض کرد که چند فراخوانی مستقل `std::print` یک پیام بزرگ را به‌صورت یک عملیات منطقی اتمیک چاپ می‌کنند.

## **تفاوت newline و flush**

یکی از اشتباهات رایج، یکسان در نظر گرفتن newline و flush است.

تابع `std::println` در انتهای خروجی یک newline اضافه می‌کند؛ اما این موضوع به معنای تخلیهٔ قطعی buffer نیست.

برای مثال:

```
#include <print>

int main() {
    std::println("Application started");
}
```

وجود newline تضمین نمی‌کند که تمام داده‌ها فوراً به مقصد نهایی تحویل داده شوند.

اگر لازم است خروجی فوراً flush شود، باید از سازوکار مناسب مقصد استفاده کنید.

برای `FILE*` می‌توان از `std::fflush` استفاده کرد:

```
#include <cstdio>
#include <print>

int main() {
    std::print(stdout, "Enter command: ");

    std::fflush(stdout);
}
```

flush مکرر می‌تواند کارایی برنامه را کاهش دهد. بنابراین بهتر است فقط در نقاطی استفاده شود که تحویل فوری خروجی اهمیت دارد.

## **مدیریت exceptionها**

توابع چاپ ممکن است در شرایط مختلف exception پرتاب کنند.

مهم‌ترین موارد عبارت‌اند از:

| Exception                | علت احتمالی                       |
| ------------------------ | --------------------------------- |
| `std::bad_alloc`         | شکست تخصیص حافظه                  |
| `std::format_error`      | خطای قالب‌بندی                    |
| `std::system_error`      | شکست عملیات مرتبط با سیستم        |
| `std::ios_base::failure` | خطای عملیات stream در شرایط مربوط |

نوع دقیق exception به overload مورد استفاده و شرایط وقوع خطا بستگی دارد.

برای مثال، در overloadهای مربوط به C stream، شکست عملیات نوشتن می‌تواند به `std::system_error` منجر شود.

در overloadهای `std::ostream` نیز رفتار خطا به وضعیت stream و exception mask آن وابسته است.

نمونهٔ مدیریت exception:

```
#include <exception>
#include <print>
#include <system_error>

int main() {
    try {
        std::println("Application started");
    }
    catch (const std::system_error& error) {
        return 1;
    }
    catch (const std::exception& error) {
        return 2;
    }
}
```

در برنامه‌های واقعی، بهتر است exceptionها در مرزهای مناسب معماری برنامه مدیریت شوند، نه اینکه برای هر فراخوانی چاپ یک try-catch مستقل ایجاد شود.

## **مقایسه std::print با printf**

تابع `printf` یکی از روش‌های قدیمی و متداول چاپ خروجی در C و ++C است.

مثال:

```
#include <cstdio>

int main() {
    int age = 25;

    std::printf("Age: %d\n", age);
}
```

در مقابل، با `std::print` می‌توان همین خروجی را به شکل زیر تولید کرد:

```
#include <print>

int main() {
    int age = 25;

    std::println("Age: {}", age);
}
```

تفاوت‌های مهم:

| ویژگی                | printf           | std::print               |
| -------------------- | ---------------- | ------------------------ |
| زبان اصلی            | C                | ++C                      |
| قالب‌بندی            | Format specifier | Format string            |
| Type safety          | محدود            | قوی‌تر                   |
| Formatter سفارشی     | خیر              | بله                      |
| استفاده از C++ types | محدود            | مستقیم‌تر                |
| بررسی format string  | محدود            | امکان بررسی compile-time |
| وابستگی به C++23     | خیر              | بله                      |

یکی از مشکلات `printf` این است که تطبیق نادرست format specifier و نوع آرگومان ممکن است باعث رفتار تعریف‌نشده شود.

برای مثال:

```
#include <cstdio>

int main() {
    int value = 42;

    std::printf("%s\n", value);
}
```

در این مثال، `%s` انتظار یک رشته را دارد، درحالی‌که آرگومان از نوع `int` است.

در مقابل، سیستم قالب‌بندی `std::print` از type system و formatterهای استاندارد استفاده می‌کند.

برای پروژه‌های جدید ++C، معمولاً `std::print` انتخاب مناسب‌تری است؛ اما `printf` همچنان برای سازگاری با C و کدهای قدیمی کاربرد دارد.

## **مقایسه std::print با std::cout**

هر دو API برای تولید خروجی در ++C استفاده می‌شوند؛ اما مدل طراحی آن‌ها متفاوت است.

مثال با `std::cout`:

```
#include <iostream>

int main() {
    std::string name = "Alice";
    int age = 25;

    std::cout << "Name: " << name
              << ", Age: " << age << '\n';
}
```

مثال با `std::print`:

```
#include <print>

int main() {
    std::string name = "Alice";
    int age = 25;

    std::println("Name: {}, Age: {}", name, age);
}
```

مقایسهٔ کاربردی:

| ویژگی     | std::cout     | std::print         |
| --------- | ------------- | ------------------ |
| مدل خروجی | Stream-based  | Format-based       |
| قالب‌بندی | Manipulatorها | Format specifierها |

\| Formatter سفارشی | operator\
\#include \<print>

int main(int argc, char\* argv[]) {\
if (argc < 2) {\
std::println(stderr, "Usage: app \<filename>");\
return 1;\
}

std::println("Processing file: {}", argv[1]);

return 0;\
}

````

در این مثال، پیام راهنما به `stderr` ارسال می‌شود و پیام پردازش به `stdout`.

### نمایش جدول

با ترکیب width و alignment می‌توان خروجی‌های جدولی ساده ایجاد کرد.

```cpp
#include <print>

int main() {
    std::println("{:<15} {:>10} {:>12}", "Product", "Quantity", "Price");

    std::println("{:-<39}", "");

    std::println("{:<15} {:>10} {:>12.2f}", "Keyboard", 5, 49.99);
    std::println("{:<15} {:>10} {:>12.2f}", "Mouse", 10, 19.50);
    std::println("{:<15} {:>10} {:>12.2f}", "Monitor", 2, 199.99);
}
````

### **نمایش اطلاعات ساختاریافته**

با تعریف formatter می‌توان نمایش نوع‌های پیچیده را استاندارد کرد.

```
#include <format>
#include <print>

struct User {
    int id;
    std::string name;
};

template <>
struct std::formatter<User> : std::formatter<std::string> {
    auto format(const User& user, std::format_context& context) const {
        return std::formatter<std::string>::format(
            std::format("User(id={}, name={})", user.id, user.name),
            context
        );
    }
};

int main() {
    User user{10, "Alice"};

    std::println("{}", user);
}
```

### **استفاده برای debugging**

برای چاپ سریع مقادیر در زمان توسعه می‌توان از `std::print` استفاده کرد.

```
#include <print>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40};

    for (std::size_t i = 0; i < values.size(); ++i) {
        std::println("values[{}] = {}", i, values[i]);
    }
}
```

البته برای debugging پیچیده، استفاده از debugger یا logging framework مناسب‌تر است.

## **محدودیت‌ها و نکات مهم**

### **نیاز به پشتیبانی Standard Library**

فعال‌کردن ++C23 به‌تنهایی تضمین نمی‌کند که `std::print` در دسترس باشد.

برای استفاده از این قابلیت باید موارد زیر فراهم باشند:

- کامپایلری با پشتیبانی مناسب از ++C23.
- Standard Library دارای پیاده‌سازی `std::print`.
- header مربوط به `<print>`.

### **وابستگی به formatter**

قابلیت `std::print` نمی‌تواند هر نوع دلخواهی را بدون formatter مناسب چاپ کند.

اگر نوع سفارشی formatter نداشته باشد، تلاش برای قالب‌بندی آن ممکن است در زمان کامپایل نامعتبر باشد.

### **عدم تضمین کارایی در تمام شرایط**

اگرچه `std::print` برای چاپ قالب‌بندی‌شده طراحی شده است، نباید فرض کرد که در تمام شرایط سریع‌تر از `std::cout` یا `printf` است.

کارایی به موارد مختلفی وابسته است:

- پیاده‌سازی Standard Library.
- نوع مقصد خروجی.
- پیچیدگی format string.
- نوع آرگومان‌ها.
- تعداد فراخوانی‌ها.
- تخصیص حافظه.
- هزینهٔ synchronization.
- رفتار سیستم‌عامل و دستگاه خروجی.

برای برنامه‌های performance-critical، benchmark روی محیط واقعی ضروری است.

### **مدیریت ورودی غیرقابل‌اعتماد**

نباید ورودی کاربر را بدون بررسی به‌عنوان format string استفاده کرد.

برای مثال، بهتر است دادهٔ کاربر به‌عنوان آرگومان وارد شود:

```
#include <print>
#include <string>

int main() {
    std::string userInput = "Hello {}";

    std::println("User input: {}", userInput);
}
```

در این حالت، محتوای ورودی کاربر به‌عنوان متن عادی چاپ می‌شود.

این روش از تفسیر ناخواستهٔ محتوای کاربر به‌عنوان format string جلوگیری می‌کند.

### **عدم جایگزینی logging framework**

قابلیت `std::print` برای چاپ مستقیم طراحی شده است، نه مدیریت کامل سیستم logging.

در برنامه‌های بزرگ، معمولاً امکاناتی مانند موارد زیر لازم هستند:

- Log levels
- Timestamp
- Log rotation
- Multiple sinks
- Filtering
- Structured logging
- Thread-safe logging
- Asynchronous logging

برای چنین نیازهایی استفاده از logging framework اختصاصی مناسب‌تر است.

## **بررسی پشتیبانی کامپایلر**

برای بررسی پشتیبانی Standard Library از قابلیت چاپ قالب‌بندی‌شده می‌توان از feature-test macro استفاده کرد.

این macro با نام `__cpp_lib_print` معرفی شده است.

```
#include <version>

#if defined(__cpp_lib_print) && __cpp_lib_print >= 202207L
    #include <print>
#else
    #error "std::print is not supported by this standard library"
#endif
```

مقدار feature-test macro مربوط به قابلیت اصلی چاپ قالب‌بندی‌شده در ++C23 برابر `202207L` است.

نکتهٔ مهم این است که پشتیبانی از یک قابلیت در compiler و Standard Library لزوماً هم‌زمان ارائه نمی‌شود.

بنابراین هنگام بررسی سازگاری باید نسخهٔ کتابخانهٔ استاندارد انتخاب‌شده را نیز در نظر بگیرید.

### **کامپایل با GCC**

برای فعال‌کردن استاندارد ++C23 در GCC می‌توان از گزینهٔ زیر استفاده کرد:

```
g++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o app
```

### **کامپایل با Clang**

نمونهٔ فرمان برای Clang:

```
clang++ -std=c++23 -Wall -Wextra -pedantic main.cpp -o app
```

در Clang باید به Standard Library انتخاب‌شده نیز توجه کرد؛ زیرا Clang ممکن است همراه با libc++ یا libstdc++ استفاده شود.

### **کامپایل با MSVC**

در MSVC می‌توان از گزینهٔ زیر برای انتخاب استاندارد استفاده کرد:

```
cl /std:c++latest /EHsc main.cpp
```

پشتیبانی واقعی `std::print` به نسخهٔ MSVC و پیاده‌سازی Microsoft STL وابسته است.

## **بهترین روش‌های استفاده در پروژه‌های حرفه‌ای**

برای استفادهٔ صحیح و قابل‌نگهداری از `std::print` بهتر است اصول زیر رعایت شوند:

- **استفاده از format string ثابت:** در حالت عادی، format stringهای compile-time را ترجیح دهید تا خطاهای قالب‌بندی زودتر تشخیص داده شوند.
- **انتخاب تابع مناسب:** برای چاپ یک خط مستقل از `std::println` و برای خروجی پیوسته از `std::print` استفاده کنید.
- **تفکیک خروجی عادی و خطا:** پیام‌های عادی را به `stdout` و پیام‌های خطا را به `stderr` ارسال کنید.
- **استفاده از formatter اختصاصی:** برای نوع‌های پیچیده، نمایش متنی استاندارد و قابل‌خواندن طراحی کنید.
- **پرهیز از flush غیرضروری:** تخلیهٔ مکرر buffer می‌تواند کارایی را کاهش دهد.
- **مدیریت هم‌زمانی:** برای خروجی‌های چندریسمانی، سیاست مشخصی برای synchronization داشته باشید.
- **رعایت امنیت اطلاعات:** از چاپ اطلاعات حساس مانند password، token و کلیدهای رمزنگاری خودداری کنید.
- **اندازه‌گیری کارایی:** در مسیرهای حساس، هزینهٔ قالب‌بندی و I/O را با benchmark بررسی کنید.
- **پرهیز از وابستگی غیرمستقیم:** headerهای موردنیاز را مستقیماً include کنید.
- **تفکیک منطق برنامه از خروجی:** در طراحی libraryهای عمومی، چاپ مستقیم را به caller تحمیل نکنید.

## **جمع‌بندی**

قابلیت `std::print` در ++C23 یک روش استاندارد، مدرن و type-safe برای چاپ متن قالب‌بندی‌شده فراهم می‌کند.

این قابلیت با استفاده از زیرساخت `std::format` در ++C20، امکان استفاده از format stringها، formatterهای سفارشی، قالب‌بندی عددی و چاپ مستقیم به مقصدهای مختلف را فراهم می‌سازد.

مهم‌ترین تفاوت آن با روش‌های قدیمی، ترکیب سادگی format stringها با قابلیت‌های type system زبان ++C است.

برای استفادهٔ حرفه‌ای از این قابلیت باید علاوه بر syntax، با موضوعاتی مانند format specifierها، formatterهای سفارشی، مقصدهای خروجی، Unicode، exceptionها، buffering و synchronization نیز آشنا باشید.

در نهایت، `std::print` برای بسیاری از سناریوهای چاپ مستقیم در برنامه‌های مدرن ++C انتخاب مناسبی است؛ اما جایگزین کامل iostream یا logging frameworkهای تخصصی محسوب نمی‌شود.

## **منابع و مراجع**

- [cppreference - std::print](https://en.cppreference.com/w/cpp/io/print?utm_source=chatgpt.com)
- [cppreference - std::println](https://en.cppreference.com/w/cpp/io/println?utm_source=chatgpt.com)
- [cppreference - std::format](https://en.cppreference.com/w/cpp/utility/format?utm_source=chatgpt.com)
- [cppreference - std::formatter](https://en.cppreference.com/w/cpp/utility/format/formatter?utm_source=chatgpt.com)
- [cppreference - std::vformat](https://en.cppreference.com/w/cpp/utility/format/vformat?utm_source=chatgpt.com)
- [C++ Standard Draft](https://eel.is/c++draft/?utm_source=chatgpt.com)
- [P2093R14 - Formatted output](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2093r14.html?utm_source=chatgpt.com)

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>