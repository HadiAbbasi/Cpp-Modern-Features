<div align="center">

[🇺🇸 English](../../en/cpp20/modules.md) | [🇮🇷 فارسی](./modules.md)

</div>

---

# بررسی جامع قابلیت Modules در استاندارد ++C20

قابلیت **Modules** یکی از بنیادی‌ترین تغییرات ساختاری در زبان ++C از زمان پیدایش آن محسوب می‌شود که در استاندارد **++C20** معرفی شد. هدف اصلی این قابلیت، جایگزینی مدل سنتی و پرهزینه متنی (`#include`) با یک مدل کامپایل ماژولار، ایزوله و مبتنی بر معناشناسی (Semantic) است.

## چرا به قابلیت Modules نیاز داریم و چه مشکلی را حل می‌کند؟

در مدل سنتی فایل‌های سرآیند (Header Files)، دستور پیش‌پردازنده `#include` صرفاً محتوای یک فایل متنی را به صورت خام در فایل دیگر کپی می‌کند. این سازوکار در پروژه‌های بزرگ مشکلات متعددی ایجاد می‌کند:

* **افزایش شدید زمان کامپایل:** کامپایلر مجبور است یک فایل سرآیند (مانند `<vector>` یا `<iostream>`) را در هر Translation Unit که آن را فراخوانی کرده، مجدداً پردازش و تجزیه (Parse) کند.
* **نشت ماکروها و تعاریف (Macro Leakage):** ماکروهای تعریف‌شده در یک فایل سرآیند به تمام فایل‌هایی که بعد از آن قرار می‌گیرند سرایت می‌کنند و می‌توانند نام متغیرها یا توابع را تغییر دهند.
* **وابستگی به ترتیب فراخوانی:** تغییر ترتیب خطوط `#include` می‌تواند باعث بروز خطاهای کامپایل یا تغییر رفتار برنامه شود.
* **نقض قانون ODR (One Definition Rule):** تعریف مجدد موجودیت‌ها در فایل‌های سرآیند مختلف به سادگی باعث خطاهای مرحله Link می‌شود.
* **عدم وجود کپسوله‌سازی واقعی:** پنهان کردن جزئیات پیاده‌سازی در فایل‌های سرآیند نیازمند ترفندهایی مانند الگوی PIMPL یا فضای نام `detail` است، در حالی که کامپایلر همچنان کل کد را پردازش می‌کند.

قابلیت **Modules** این مشکلات را با تبدیل رابط ماژول به یک نمایش باینری از پیش کامپایل‌شده (BMI - Binary Module Interface) حل می‌کند. ماکروها از مرز ماژول عبور نمی‌کنند، ترتیب `import` اهمیتی ندارد و هر ماژول تنها یک بار کامپایل می‌شود.

## مفاهیم پایه و ساختار یک ماژول

هر ماژول در ++C20 دارای یک نام منحصربه‌فرد است و از طریق کلمات کلیدی جدید و حساس به متن (Contextual Keywords) مانند `export`، `import` و `module` تعریف می‌شود.

### بخش‌های اصلی یک واحد ماژول (Module Unit)

هر فایل ماژول از نظر ساختاری به بخش‌های مشخصی تقسیم می‌شود که ترتیب قرارگیری آن‌ها توسط استاندارد اجباری شده است:

1. **بخش Global Module Fragment:** با دستور `;module` در ابتدای فایل شروع می‌شود و محلی است که باید دستورات `#include` مربوط به کتابخانه‌های سنتی یا فایل‌های سرآیند سیستم عامل در آن قرار گیرند تا در ماژول ادغام نشوند.
2. **اعلان ماژول (Module Declaration):** با دستور `;export module ModuleName` یا `;module ModuleName` آغاز می‌شود و مرز اصلی ماژول را مشخص می‌کند.
3. **بدنه ماژول (Module Purview):** شامل دستورات `import`، تعاریف، پیاده‌سازی‌ها و نمادهای صادرشده (`export`) است.
4. **بخش Private Module Fragment:** بخش اختیاری در انتهای فایل که با `;module :private` شروع می‌شود و جزئیات کاملاً خصوصی ماژول را در خود جای می‌دهد.

### صادرات تکی در برابر صادرات بلوکی (Block Export)

برای در دسترس قرار دادن توابع، کلاس‌ها، متغیرها یا فضاهای نام برای مصرف‌کنندگان ماژول، از کلمه کلیدی `export` استفاده می‌شود. این کار به دو روش قابل انجام است:

* **صادرات تکی:** قرار دادن `export` پشت هر اعلان به صورت جداگانه.
* **صادرات بلوکی (Block Export):** قرار دادن چندین اعلان درون یک بلوک `{ ... } export` که خوانایی کد را افزایش می‌دهد و از تکرار کلمه کلیدی `export` جلوگیری می‌کند.

مثال زیر یک فایل رابط ماژول (Module Interface Unit) را با استفاده از هر دو روش نشان می‌دهد:

```cpp
// math_utils.ixx
module; // Global module fragment
#include <cmath>

export module math_utils; // Module declaration

// Single export
export int add(int a, int b) {
    return a + b;
}

// Block export
export {
    double calculate_hypotenuse(double a, double b) {
        return std::hypot(a, b);
    }

    struct Point2D {
        double x{};
        double y{};
    };
}

// Internal helper function (not exported)
double internal_scale(double val) {
    return val * 1.5;
}

```

در فایل مصرف‌کننده (مانند `main.cpp`)، تنها کافی است ماژول را با دستور `import` فراخوانی کنید:

```cpp
// main.cpp
import <iostream>;
import math_utils;

int main() {
    int sum = add(3, 4);
    Point2D p{3.0, 4.0};
    double dist = calculate_hypotenuse(p.x, p.y);
    
    // Error: internal_scale is not visible here
    // double s = internal_scale(10.0);
    
    std::cout << "Sum: " << sum << ", Distance: " << dist << '\n';
    return 0;
}

```

## تفکیک رابط ماژول از پیاده‌سازی

در پروژه‌های واقعی، قرار دادن تمام کدهای پیاده‌سازی درون فایل رابط ماژول باعث می‌شود با هر تغییر کوچک در منطق داخلی توابع، تمام فایل‌هایی که به این ماژول وابسته‌اند مجدداً کامپایل شوند. برای جلوگیری از این مشکل، رابط ماژول را از پیاده‌سازی آن جدا می‌کنند.

### تفکیک رابط و پیاده‌سازی در یک فایل (Same File)

در پروژه‌های کوچک‌تر که تمایل دارید کل کد ماژول در یک فایل باقی بماند اما همچنان رابط از پیاده‌سازی جدا باشد، می‌توانید اعلان‌ها را در بخش `export` قرار دهید و پیاده‌سازی آن‌ها را در ادامه همان فایل (بدون `export`) یا در بخش **Private Module Fragment** بنویسید.

مفهوم **Private Module Fragment** که با دستور `;module :private` مشخص می‌شود، به کامپایلر اعلام می‌کند که هر کدی پس از این خط می‌آید، هیچ تأثیری بر رابط عمومی ماژول ندارد. در نتیجه، تغییر کدهای بعد از `;module :private` باعث کامپایل مجدد مصرف‌کنندگان ماژول نخواهد شد.

```cpp
// logger.ixx
module;
#include <string_view>
#include <iostream>

export module logger;

export class Logger {
public:
    explicit Logger(std::string_view prefix);
    void log(std::string_view message) const;

private:
    std::string_view prefix_;
};

// Everything below this line is completely private to this translation unit
module :private;

Logger::Logger(std::string_view prefix) : prefix_(prefix) {}

void Logger::log(std::string_view message) const {
    std::cout << "[" << prefix_ << "] " << message << '\n';
}

```

نکته مهم این است که **Private Module Fragment** تنها زمانی مجاز است که کل ماژول از یک فایل واحد تشکیل شده باشد؛ یعنی اگر ماژول دارای فایل‌های پیاده‌سازی جداگانه یا پارتیشن باشد، استفاده از `;module :private` منجر به خطای کامپایل می‌شود.

### تفکیک رابط و پیاده‌سازی در فایل‌های مجزا (Different Files)

روش استاندارد برای پروژه‌های متوسط و بزرگ، جداسازی فایل رابط (**Module Interface Unit**) از فایل پیاده‌سازی (**Module Implementation Unit**) است.

* **فایل رابط:** دارای دستور `;export module Name` است و معمولاً با پسوندهای `.ixx` (در MSVC) یا `.cppm` (در Clang و GCC) ذخیره می‌شود.
* **فایل پیاده‌سازی:** دارای دستور `;module Name` (بدون کلمه کلیدی `export`) است و معمولاً با پسوند استاندارد `.cpp` ذخیره می‌شود.

فایل پیاده‌سازی به صورت خودکار ماژول همنام خود را `import` می‌کند؛ بنابراین به تمام اعلان‌ها و بخش‌های خصوصی (Private) کلاس‌های تعریف‌شده در رابط ماژول دسترسی کامل دارد.

فایل رابط ماژول به صورت زیر نوشته می‌شود:

```cpp
// bank_account.ixx
module;
#include <string>

export module bank_account;

export class BankAccount {
public:
    BankAccount(std::string owner, double initial_balance);
    void deposit(double amount);
    bool withdraw(double amount);
    double balance() const noexcept;

private:
    std::string owner_;
    double balance_;
};

```

فایل پیاده‌سازی ماژول در یک فایل `.cpp` مجزا قرار می‌گیرد:

```cpp
// bank_account.cpp
module;
#include <stdexcept>

module bank_account; // Implementation unit for bank_account

BankAccount::BankAccount(std::string owner, double initial_balance)
    : owner_(std::move(owner)), balance_(initial_balance) {
    if (balance_ < 0.0) {
        throw std::invalid_argument("Initial balance cannot be negative");
    }
}

void BankAccount::deposit(double amount) {
    if (amount > 0.0) {
        balance_ += amount;
    }
}

bool BankAccount::withdraw(double amount) {
    if (amount > 0.0 && balance_ >= amount) {
        balance_ -= amount;
        return true;
    }
    return false;
}

double BankAccount::balance() const noexcept {
    return balance_;
}

```

### استفاده از چندین فایل پیاده‌سازی (Multiple Implementation Files)

یک ماژول می‌تواند تنها **یک** فایل رابط اصلی (Primary Module Interface Unit) داشته باشد، اما هیچ محدودیتی در تعداد فایل‌های پیاده‌سازی وجود ندارد. اگر یک ماژول شامل چندین کلاس یا زیرسیستم بزرگ باشد، می‌توانید پیاده‌سازی هر بخش را در یک فایل `.cpp` جداگانه قرار دهید و در ابتدای همه آن‌ها عبارت `;module ModuleName` را بنویسید.

فرض کنید ماژول `network` دو کلاس `TcpSocket` و `UdpSocket` را در رابط خود صادر کرده است:

```cpp
// network.ixx
export module network;

export class TcpSocket {
public:
    void connect();
};

export class UdpSocket {
public:
    void bind();
};

```

پیاده‌سازی این دو کلاس را می‌توان به راحتی بین دو فایل مجزا تقسیم کرد:

```cpp
// tcp_socket.cpp
module;
#include <iostream>

module network;

void TcpSocket::connect() {
    std::cout << "TCP Socket connected.\n";
}

```

```cpp
// udp_socket.cpp
module;
#include <iostream>

module network;

void UdpSocket::bind() {
    std::cout << "UDP Socket bound.\n";
}

```

## ساختاردهی ماژول‌های بزرگ و تجمیع رابط‌ها

در کتابخانه‌ها و فریم‌ورک‌های بزرگ، قرار دادن تمام رابط‌ها در یک فایل واحد غیرعملی است. زبان ++C20 دو مکانیزم مختلف برای تقسیم‌بندی ماژول‌ها ارائه می‌دهد: **Sub-Modules** و **Module Partitions**.

### زیرماژول‌ها (Sub-Modules) و چندین فایل رابط

در استاندارد ++C، علامت نقطه (`.`) در نام ماژول هیچ معنای سلسله‌مراتبی خاصی برای کامپایلر ندارد و صرفاً بخشی از شناسه نام ماژول است. با این حال، توسعه‌دهندگان از نقطه برای ایجاد ساختار منطقی **Sub-Modules** (مانند `graphics.shapes` و `graphics.colors`) استفاده می‌کنند.

هر Sub-Module یک ماژول کاملاً مستقل با فایل رابط مجزای خود است. برای اینکه کاربر نهایی مجبور نباشد تک‌تک این ماژول‌های کوچک را `import` کند، یک ماژول تجمیع‌کننده (Aggregator Module) ایجاد می‌شود که با استفاده از دستور `export import` تمام زیرماژول‌ها را بازصادرات (Re-export) می‌کند.

### استفاده از دستور Export Import به صورت تکی و دسته‌ای

دستور `export import` هم ماژول هدف را در ماژول فعلی بارگذاری می‌کند و هم تمام نمادهای صادرشده آن را در اختیار مصرف‌کنندگان ماژول فعلی قرار می‌دهد.

مثال زیر نحوه تعریف چندین فایل رابط به عنوان زیرماژول و تجمیع آن‌ها را نشان می‌دهد:

```cpp
// graphics_shapes.ixx
export module graphics.shapes;

export struct Circle {
    double radius;
};

export struct Rectangle {
    double width;
    double height;
};

```

```cpp
// graphics_colors.ixx
export module graphics.colors;

export struct RGB {
    unsigned char r, g, b;
};

```

در فایل رابط اصلی ماژول `graphics`، می‌توان زیرماژول‌ها را به صورت مجزا یا در قالب یک بلوک `export` بازصادرات کرد:

```cpp
// graphics.ixx
export module graphics;

// Separated export import
export import graphics.shapes;

// Batch export import using block export
export {
    import graphics.colors;
    // Other modules can also be added here
}

```

اکنون کاربر با نوشتن تنها یک خط `import graphics;` به تمام ساختارهای `Circle`، `Rectangle` و `RGB` دسترسی خواهد داشت.

### پارتیشن‌های ماژول (Module Partitions)

پارتیشن‌ها به شما اجازه می‌دهند یک ماژول واحد را در سطح فیزیکی بین چندین فایل تقسیم کنید، بدون اینکه ماژول‌های مستقلی برای دنیای خارج بسازید. نام پارتیشن با علامت دو نقطه (`:`) مشخص می‌شود، مانند `engine:physics` یا `engine:audio`.

تفاوت کلیدی **Module Partitions** با **Sub-Modules** در این است که پارتیشن‌ها از بیرون ماژول به صورت مستقیم قابل `import` نیستند؛ یعنی کاربر نمی‌تواند بنویسد `;import engine:physics`. پارتیشن‌ها فقط درون همان ماژول مادر قابل مشاهده هستند و ماژول اصلی باید آن‌ها را تجمیع کند.

پارتیشن‌ها به دو دسته تقسیم می‌شوند:

1. **پارتیشن رابط (Module Interface Partition):** با `;export module ModuleName:PartitionName` تعریف می‌شود و نمادهایی را صادر می‌کند که باید توسط ماژول اصلی بازصادرات شوند (`;export import :PartitionName`).
2. **پارتیشن داخلی (Internal Module Partition):** با `;module ModuleName:PartitionName` (بدون `export`) تعریف می‌شود و فقط برای استفاده داخلی سایر بخش‌های همان ماژول کاربرد دارد (`;import :PartitionName`).

مثال زیر ترکیب یک پارتیشن رابط و یک پارتیشن داخلی را در ماژول `engine` نشان می‌دهد:

```cpp
// engine_core.ixx (Internal Module Partition - NOT exported)
module engine:core;

struct InternalState {
    int tick_count = 0;
};

InternalState& get_state() {
    static InternalState state;
    return state;
}

```

```cpp
// engine_physics.ixx (Module Interface Partition)
export module engine:physics;

import :core; // Importing internal partition within the same module

export void update_physics() {
    auto& state = get_state();
    state.tick_count++;
}

```

```cpp
// engine.ixx (Primary Module Interface Unit)
export module engine;

// Re-exporting the interface partition is mandatory if it has exported entities
export import :physics;

// Internal partition :core is NOT re-exported, keeping it completely hidden

```

### استفاده از ماژول داخلی خصوصی در یک ماژول صادرشده

در طراحی کتابخانه‌های بزرگ، گاهی نیاز است بخش‌هایی از کد بین چندین فایل ماژول به اشتراک گذاشته شوند، اما به هیچ وجه در اختیار مصرف‌کننده خارجی قرار نگیرند. دو الگوی استاندارد برای این هدف وجود دارد:

* **استفاده از Internal Partition در یک ماژول واحد:** همان‌طور که در بخش قبل نشان داده شد، تعریف یک پارتیشن بدون `export` (مانند `;module engine:core`) تضمین می‌کند که هیچ ماژول خارجی نمی‌تواند به آن دسترسی داشته باشد، در حالی که تمام فایل‌های متعلق به ماژول `engine` می‌توانند آن را `import` کنند.
* **استفاده از یک ماژول کمکی با `import` معمولی (غیرصادراتی):** اگر دو ماژول مجزا دارید، ماژول صادرشده (`public_api`) می‌تواند ماژول داخلی (`internal_helpers`) را با دستور ساده `;import internal_helpers` (بدون `export`) وارد کند. به این ترتیب، توابع و کلاس‌های `internal_helpers` در پیاده‌سازی `public_api` قابل استفاده‌اند، اما برای برنامه‌ای که `public_api` را `import` می‌کند نامرئی باقی می‌مانند.

```cpp
// internal_crypto.ixx
export module internal_crypto;

export unsigned int compute_hash(const char* data) {
    unsigned int hash = 5381;
    while (*data) {
        hash = ((hash << 5) + hash) + static_cast<unsigned char>(*data++);
    }
    return hash;
}

```

```cpp
// auth_service.ixx
module;
#include <string>

export module auth_service;

import internal_crypto; // Imported privately, NOT re-exported

export class AuthService {
public:
    bool verify_token(const std::string& token) const {
        // Uses internal_crypto internally
        return compute_hash(token.c_str()) != 0;
    }
};

```

در این حالت، کدی که `import auth_service;` را اجرا می‌کند تنها `AuthService` را می‌بیند و شناسه `compute_hash` برای آن کاملاً نامرئی است.

## ترکیب ماژول‌ها با فضاهای نام (Namespaces)

قابلیت **Modules** جایگزین `namespace` نیست، بلکه مکمل آن است. ماژول‌ها واحدهای فیزیکی و منطقی کامپایل و کپسوله‌سازی هستند، در حالی که فضاهای نام برای جلوگیری از تداخل نام‌ها (Name Collision) در سطح معنایی به کار می‌روند.

نکته بسیار مهم این است که **یک ماژول فضای نام جدیدی ایجاد نمی‌کند**. اگر تابعی را درون ماژول `my_lib` بدون فضای نام صادر کنید، آن تابع در فضای نام سراسری (Global Namespace) قرار می‌گیرد. بنابراین، همیشه باید کدهای صادرشده درون ماژول را در یک `namespace` مناسب قرار دهید.

```cpp
// geometry.ixx
export module geometry;

// Exporting an entire namespace block
export namespace geom {
    constexpr double pi = 3.141592653589793;

    double circle_area(double radius) {
        return pi * radius * radius;
    }

    namespace details {
        // Even inside a namespace, items not marked export remain module-local
        // unless the enclosing namespace block itself is exported!
    }
}

```

توجه داشته باشید که وقتی کل بلوک `namespace` را با `export namespace geom { ... }` صادر می‌کنید، **تمام** موجودیت‌های درون آن بلوک صادر می‌شوند. اگر می‌خواهید بخشی از فضای نام خصوصی بماند، باید به جای صادر کردن کل بلوک، فقط موجودیت‌های مورد نظر را به صورت تکی `export` کنید:

```cpp
// geometry_selective.ixx
export module geometry_selective;

namespace geom {
    // Not exported: invisible outside the module
    constexpr double secret_factor = 0.42;

    // Exported: visible to importers
    export double compute(double x) {
        return x * secret_factor;
    }
}

```

## پیاده‌سازی قالب‌های کلاس سفارشی (Class Templates) در ماژول‌ها

قالب‌ها (Templates) در ++C به دلیل ماهیت نمونه‌سازی (Instantiation) در زمان کامپایل، نیازمند آن هستند که تعریف کامل آن‌ها در دسترس کامپایلر باشد. در مدل سنتی فایل‌های سرآیند، مجبور بودید تمام کدهای template را در فایل `.h` قرار دهید که منجر به لو رفتن توابع کمکی خصوصی و کندی کامپایل می‌شد.

در ماژول‌های ++C20، قالب‌های کلاس و تابع را می‌توان در فایل رابط ماژول (`export`) تعریف کرد، در حالی که توابع کمکی غیرقالب یا انواع داخلی که template از آن‌ها استفاده می‌کند نیازی به `export` شدن ندارند. کامپایلر هنگام نمونه‌سازی قالب در کد مصرف‌کننده، از طریق اطلاعات موجود در فایل باینری ماژول (BMI) به آن توابع غیرصادراتی دسترسی پیدا می‌کند (مفهوم Reachability).

```cpp
// custom_container.ixx
module;
#include <cstddef>
#include <stdexcept>
#include <utility>

export module custom_container;

// Helper function NOT exported, yet reachable when FixedBuffer is instantiated
void throw_out_of_range() {
    throw std::out_of_range("Index out of bounds in FixedBuffer");
}

export template <typename T, std::size_t Capacity>
class FixedBuffer {
public:
    constexpr std::size_t capacity() const noexcept {
        return Capacity;
    }

    void push_back(T value) {
        if (size_ >= Capacity) {
            throw_out_of_range(); // Calls non-exported helper
        }
        data_[size_++] = std::move(value);
    }

    const T& operator[](std::size_t index) const {
        if (index >= size_) {
            throw_out_of_range();
        }
        return data_[index];
    }

    std::size_t size() const noexcept {
        return size_;
    }

private:
    T data_[Capacity]{};
    std::size_t size_ = 0;
};

```

نحوه استفاده از این قالب کلاس در فایل مصرف‌کننده به صورت زیر است:

```cpp
// main.cpp
import <iostream>;
import custom_container;

int main() {
    FixedBuffer<int, 5> buffer;
    buffer.push_back(10);
    buffer.push_back(20);

    std::cout << "First element: " << buffer[0] << '\n';
    // Error: throw_out_of_range is not visible to the caller
    // throw_out_of_range();
    return 0;
}

```

## تفاوت کلیدی Visibility و Reachability

یکی از ظریف‌ترین و مهم‌ترین مفاهیم در استاندارد ماژول‌های ++C20، تفاوت میان **Visibility (قابلیت دید)** و **Reachability (قابلیت دسترسی معنایی)** است. درک این تفاوت برای جلوگیری از خطاهای عجیب کامپایل ضروری است.

* **قابلیت دید (Visibility):** مشخص می‌کند که آیا نام یک شناسه برای جستجوی نام (Name Lookup) در کد مصرف‌کننده قابل یافتن است یا خیر. تنها موجودیت‌هایی که با `export` مشخص شده‌اند Visible هستند.
* **قابلیت دسترسی (Reachability):** مشخص می‌کند که آیا کامپایلر اجازه دارد از خواص معنایی یک نوع (مانند اندازه حافظه، متدها یا اعضای داده) استفاده کند یا خیر، حتی اگر نام آن نوع در کد مصرف‌کننده Visible نباشد.

این تفکیک باعث می‌شود بتوانید یک تابع `export` شده بنویسید که یک شیء از یک نوع **غیرصادراتی** را برمی‌گرداند! مصرف‌کننده نمی‌تواند نام آن نوع را مستقیماً بنویسد (چون Visible نیست)، اما می‌تواند با استفاده از کلمه کلیدی `auto` شیء را دریافت کند و متدهای عمومی آن را فراخوانی نماید (چون نوع آن Reachable است).

```cpp
// session_manager.ixx
module;
#include <string>

export module session_manager;

// NOT exported: name 'SessionToken' is NOT visible outside this module
class SessionToken {
public:
    explicit SessionToken(std::string id) : id_(std::move(id)) {}
    
    const std::string& id() const noexcept {
        return id_;
    }

private:
    std::string id_;
};

// Exported factory function returning a non-exported type
export SessionToken create_session(const std::string& user) {
    return SessionToken("TOKEN_" + user);
}

```

در فایل مصرف‌کننده، رفتار کامپایلر به شکل زیر خواهد بود:

```cpp
// main.cpp
import <iostream>;
import session_manager;

int main() {
    // Valid: SessionToken is reachable, so 'auto' deduces its type
    // and calling .id() is completely legal.
    auto token = create_session("ali");
    std::cout << token.id() << '\n';

    // Invalid (Compiler Error): 'SessionToken' is not visible by name!
    // SessionToken explicit_token("123");
    
    return 0;
}

```

## استفاده از فایل‌های cpp معمولی در کنار ماژول‌ها

مهاجرت یک پروژه بزرگ به ماژول‌ها به صورت یک‌باره تقریباً غیرممکن است. استاندارد ++C20 به گونه‌ای طراحی شده که فایل‌های سنتی `.cpp` و `.h` می‌توانند بدون هیچ مشکلی در کنار ماژول‌ها کار کنند:

* **فراخوانی ماژول از داخل فایل‌های سنتی `.cpp`:** هر فایل `.cpp` معمولی (که حتی خودش عضو هیچ ماژولی نیست) می‌تواند با دستور `;import MyModule` از ماژول‌ها استفاده کند.
* **استفاده از فایل‌های `.h` سنتی درون ماژول:** با قرار دادن دستورات `#include` در بخش **Global Module Fragment** (بین `;module` و `;export module Name`)، ماژول می‌تواند از کتابخانه‌های قدیمی (مانند کتابخانه‌های C، رابط‌های سیستم عامل یا کتابخانه‌های شخص ثالث بدون ماژول) استفاده کند.
* **مفهوم Header Units:** در ++C20 می‌توانید فایل‌های سرآیند استاندارد یا قدیمی را مستقیماً به صورت `;import <vector>` یا `;import "legacy_header.h"` وارد کنید. این کار فایل سرآیند را به یک واحد باینری تبدیل می‌کند که مزایای سرعت ماژول‌ها را دارد، اما برخلاف ماژول‌های واقعی، ماکروهای تعریف‌شده در آن همچنان به کد واردکننده نشت می‌کنند.

مثال زیر نحوه ترکیب کتابخانه‌های سنتی و ماژول‌ها را نشان می‌دهد:

```cpp
// legacy_wrapper.ixx
module; // Start of Global Module Fragment
#include <cstdio>
#include <cstdlib>

export module legacy_wrapper; // End of Global Module Fragment

export void print_formatted(const char* text) {
    std::printf("Legacy Output: %s\n", text);
}

```

## بایدها و نبایدها در کار با ماژول‌ها (Do's and Don'ts)

رعایت اصول زیر هنگام طراحی و توسعه پروژه‌های مبتنی بر ماژول از بروز خطاهای معماری و مشکلات سیستم ساخت (Build System) جلوگیری می‌کند:

| دسته‌بندی | بایدها (Do's) | نبایدها (Don'ts) |
| --- | --- | --- |
| **ساختار فایل** | تمام `#include` ها را همیشه در بخش **Global Module Fragment** (قبل از `export module`) قرار دهید. | هرگز دستور `#include` را بعد از اعلان `export module` ننویسید؛ این کار مالکیت نمادهای سرآیند را به ماژول منتقل کرده و باعث خطاهای ODR در مرحله Link می‌شود. |
| **مدیریت ماکروها** | برای ثابت‌ها و توابع کوچک از `constexpr` و `inline` صادرشده استفاده کنید. | انتظار نداشته باشید ماکروهای `#define` از طریق `import` به فایل‌های دیگر منتقل شوند؛ ماکروها در مرز ماژول متوقف می‌شوند. |
| **فضاهای نام** | همیشه موجودیت‌های صادرشده را درون یک `namespace` مشخص قرار دهید. | هرگز دستور `import` را درون یک بلوک `namespace` ننویسید؛ دستورات `import` همیشه باید در سطح سراسری (Global Scope) فایل قرار گیرند. |
| **تفکیک کد** | رابط ماژول را کوچک نگه دارید و پیاده‌سازی‌ها را به فایل‌های `.cpp` یا بخش `module :private` منتقل کنید. | در پروژه‌های چندفایلی از `module :private` استفاده نکنید؛ این قابلیت فقط برای ماژول‌های تک‌فایلی مجاز است. |
| **وابستگی‌ها** | وابستگی‌های بین ماژول‌ها را به صورت گراف جهت‌دار غیرمدور (DAG) طراحی کنید. | از ایجاد وابستگی چرخشی (Circular Dependency) بین ماژول‌ها (مثلاً ماژول A ماژول B را import کند و بالعکس) خودداری کنید؛ این کار در استاندارد ممنوع است. |
| **توابع Inline** | بدنه توابع `inline` صادرشده را در همان فایل رابط ماژول بنویسید. | بدنه یک تابع `export inline` را در فایل پیاده‌سازی (`.cpp`) جداگانه قرار ندهید، زیرا تعریف آن باید برای تمام مصرف‌کنندگان ماژول در دسترس باشد. |

## محدودیت‌ها و نکات عملی در ابزارهای ساخت

اگرچه ماژول‌ها از نظر زبانی در ++C20 استانداردسازی شده‌اند، اما پشتیبانی ابزارها و سیستم‌های ساخت (مانند CMake، Ninja، MSBuild و کامپایلرهای GCC، Clang و MSVC) دارای پیچیدگی‌های خاص خود است:

* **ترتیب کامپایل وابستگی‌ها:** برخلاف مدل `#include` که فایل‌های `.cpp` می‌توانستند به صورت کاملاً مستقل و موازی کامپایل شوند، در مدل ماژول‌ها فایل رابط (`.ixx` یا `.cppm`) **باید** قبل از هر فایلی که آن را `import` می‌کند کامپایل شود تا فایل باینری BMI تولید گردد.
* **پشتیبانی در CMake:** برای مدیریت خودکار وابستگی‌های ماژول در پروژه‌های واقعی، استفاده از **CMake نسخه 3.28 به بالا** به همراه مولد **Ninja** (یا Visual Studio) توصیه می‌شود تا اسکن وابستگی‌ها (Dependency Scanning) به صورت خودکار انجام شود.
* **ماژول کتابخانه استاندارد (`import std;`):** قابلیت وارد کردن کل کتابخانه استاندارد با دستور ساده `;import std` به صورت رسمی در استاندارد **++C23** معرفی شد (هرچند کامپایلرهای مدرن آن را در حالت ++C20 نیز به عنوان افزونه ارائه می‌دهند). در ++C20 استاندارد، برای کتابخانه استاندارد از Header Units مانند `;import <vector>` استفاده می‌شود.


# **ماژول‌ها (Modules) در ++C20**

## **مقدمه**

قابلیت **Modules** یکی از مهم‌ترین تغییرات معماری زبان ++C20 است که با هدف بهبود نحوه سازمان‌دهی، اشتراک‌گذاری و کامپایل کد در پروژه‌های ++C معرفی شد.

برای سال‌های طولانی، مکانیزم اصلی اشتراک‌گذاری Declarationها میان Translation Unitهای مختلف، استفاده از فایل‌های Header و دستور `#include` بود. اگرچه این روش همچنان کاربرد گسترده‌ای دارد، اما در پروژه‌های بزرگ مشکلاتی مانند موارد زیر ایجاد می‌کند:

- **افزایش زمان کامپایل:** محتوای Headerها در هر Translation Unit مجدداً توسط Preprocessor پردازش می‌شود.
- **وابستگی‌های غیرضروری:** با Include کردن یک Header، ممکن است تعداد زیادی Header دیگر نیز به‌صورت زنجیره‌ای وارد پروژه شوند.
- **مشکلات Macroها:** ماکروها محدوده انتشار مشخصی ندارند و می‌توانند روی کدهای خارج از محدوده مورد انتظار تأثیر بگذارند.
- **مشکلات ODR:** مدیریت Definitionها در Headerها، به‌خصوص در پروژه‌های بزرگ، نیازمند دقت زیادی است.
- **افشای جزئیات پیاده‌سازی:** مصرف‌کنندگان یک Library معمولاً به Declarationهای داخلی و وابستگی‌هایی دسترسی پیدا می‌کنند که نیازی به مشاهده آن‌ها ندارند.
- **وابستگی به ترتیب Includeها:** در برخی پروژه‌ها ترتیب Include کردن Headerها می‌تواند روی موفقیت کامپایل تأثیر بگذارد.

قابلیت Modules با معرفی یک مدل جدید برای اشتراک‌گذاری کد، بسیاری از این مشکلات را کاهش می‌دهد.

در این مدل، به‌جای اینکه محتوای یک فایل Header در هر Translation Unit مجدداً پردازش شود، یک Module به‌صورت مستقل کامپایل می‌شود و نتیجه کامپایل آن در قالب یک فایل باینری مخصوص کامپایلر، معمولاً با نام **BMI (Built Module Interface)**، در اختیار Translation Unitهای مصرف‌کننده قرار می‌گیرد.

در نتیجه، مصرف‌کننده می‌تواند به‌جای استفاده از `#include`، از دستور `import` استفاده کند.

### **یک مثال ساده**

برای درک تفاوت این دو روش، ابتدا نحوه تعریف و استفاده از یک Module ساده را بررسی کنیم.

**فایل Math.cppm:**

```
export module Math;

export int add(int a, int b)
{
    return a + b;
}
```

**فایل main.cpp:**

```
import Math;

#include <iostream>

int main()
{
    std::cout << add(10, 20) << '\n';
}
```

در این مثال:

- دستور `export module Math;` یک Module با نام `Math` تعریف می‌کند.
- دستور `export` مشخص می‌کند که Declaration مربوط به تابع `add` بخشی از Interface عمومی Module است.
- دستور `import Math;` امکان استفاده از قابلیت‌های Export شده توسط Module را فراهم می‌کند.

نکته مهم این است که مصرف‌کننده نیازی به مشاهده فایل `Math.cppm` ندارد و صرفاً از Interface کامپایل‌شده آن استفاده می‌کند.

## **مفهوم Module در ++C**

در ++C20، یک Module مجموعه‌ای از Translation Unitها است که تحت یک نام مشترک سازمان‌دهی می‌شوند.

هر Translation Unit که بخشی از یک Module باشد، می‌تواند Declarationها و Definitionهای مربوط به همان Module را در اختیار داشته باشد.

یک Module می‌تواند از یک یا چند فایل تشکیل شود.

برای مثال، ساختار زیر یک Module ساده را نشان می‌دهد:

```
Math
 ├── Math.cppm
 ├── Math.cpp
 └── MathInternal.cpp
```

در این ساختار، هر سه فایل می‌توانند بخشی از Module با نام `Math` باشند.

البته صرفاً قرار گرفتن فایل‌ها در یک پوشه مشترک یا استفاده از نام مشابه برای آن‌ها باعث تعلقشان به یک Module نمی‌شود. این ارتباط باید با استفاده از Module Declaration در هر Translation Unit مشخص شود.

### **انواع Module Unit**

در استاندارد ++C20، واحدهای تشکیل‌دهنده یک Module به چند دسته تقسیم می‌شوند:

| نوع                             | توضیح                                                             |
| ------------------------------- | ----------------------------------------------------------------- |
| Primary Module Interface Unit   | Interface اصلی و عمومی Module                                     |
| Module Interface Partition      | بخشی از Interface که در یک واحد مستقل تعریف شده است               |
| Module Implementation Unit      | واحدی برای پیاده‌سازی قابلیت‌های Module                           |
| Module Implementation Partition | بخشی داخلی از Module که برای سازمان‌دهی پیاده‌سازی استفاده می‌شود |

هر Module نام‌گذاری‌شده (Named Module) باید دقیقاً یک Primary Module Interface Unit داشته باشد.

این Interface نقطه ورود مصرف‌کنندگان خارجی به Module است.

## **معرفی Syntaxهای اصلی Modules**

قبل از بررسی ساختارهای پیچیده‌تر، بهتر است با Syntaxهای اصلی آشنا شویم.

| Syntax                | کاربرد                                         |
| --------------------- | ---------------------------------------------- |
| `export module M;`    | تعریف Primary Module Interface                 |
| `module M;`           | تعریف Module Implementation Unit               |
| `export module M:P;`  | تعریف Interface Partition                      |
| `module M:P;`         | تعریف Implementation Partition                 |
| `import M;`           | وارد کردن یک Module                            |
| `import :P;`          | وارد کردن یک Partition از همان Module          |
| `export import M;`    | وارد کردن و Export مجدد یک Module              |
| `export import :P;`   | وارد کردن و Export کردن یک Interface Partition |
| `export declaration;` | Export کردن یک Declaration                     |
| `export { ... }`      | Export کردن گروهی از Declarationها             |
| `module;`             | شروع Global Module Fragment                    |
| `module : private;`   | شروع Private Module Fragment                   |

در ادامه، کاربرد هرکدام را با مثال بررسی می‌کنیم.

## **مفهوم Export و Import**

دو دستور اصلی در سیستم Modules عبارت‌اند از `export` و `import`.

### **کاربرد Export**

کلمه کلیدی `export` مشخص می‌کند که یک Declaration بخشی از Interface قابل‌دسترسی برای مصرف‌کنندگان خارجی Module باشد.

به‌صورت پیش‌فرض، Declarationهایی که در یک Named Module تعریف می‌شوند، صرفاً در محدوده همان Module قابل‌استفاده هستند و به‌صورت خودکار در اختیار مصرف‌کنندگان قرار نمی‌گیرند.

برای مثال:

```
export module Math;

int internalAdd(int a, int b)
{
    return a + b;
}

export int add(int a, int b)
{
    return internalAdd(a, b);
}
```

در این مثال:

- تابع `add` Export شده است.
- تابع `internalAdd` Export نشده است.

بنابراین مصرف‌کننده می‌تواند از `add` استفاده کند، اما نمی‌تواند مستقیماً تابع `internalAdd` را فراخوانی کند.

**فایل main.cpp:**

```
import Math;

int main()
{
    int result = add(10, 20);

    // Error: internalAdd is not exported
    // int value = internalAdd(10, 20);
}
```

توجه کنید که Export نکردن یک Declaration به معنی عدم وجود آن در زمان کامپایل یا لینک نیست. این موضوع به قابلیت دسترسی و Reachability آن مربوط می‌شود که در بخش‌های بعدی بررسی خواهیم کرد.

### **کاربرد Export Block**

برای Export کردن چند Declaration می‌توان به‌جای تکرار کلمه کلیدی `export` از یک Export Block استفاده کرد.

```
export module Math;

export
{
    int add(int a, int b)
    {
        return a + b;
    }

    int subtract(int a, int b)
    {
        return a - b;
    }

    int multiply(int a, int b)
    {
        return a * b;
    }
}
```

در این مثال، هر سه تابع بخشی از Interface عمومی Module هستند.

استفاده از Export Block زمانی مفید است که چند Declaration مرتبط را در کنار یکدیگر قرار داده‌ایم و می‌خواهیم همگی در Interface عمومی قرار بگیرند.

البته Export Block صرفاً یک ابزار نحوی برای گروه‌بندی Declarationها است و رفتار متفاوتی نسبت به Export کردن تک‌تک آن‌ها ایجاد نمی‌کند.

### **کاربرد Import**

برای استفاده از قابلیت‌های Export شده توسط یک Module، از دستور `import` استفاده می‌کنیم.

```
import Math;
import Logger;
import Database;

int main()
{
    auto result = add(10, 20);

    return 0;
}
```

دستور Import باید در Global Scope قرار بگیرد و در یک Module Unit، بعد از Module Declaration و قبل از سایر Declarationهای معمولی نوشته شود.

### **وارد کردن چند Module به‌صورت جداگانه یا گروهی**

در ++C20 هیچ Syntax ویژه‌ای برای Import گروهی چند Module در یک دستور وجود ندارد.

بنابراین روش معمول به‌صورت زیر است:

```
import Math;
import Logger;
import Database;
```

اگرچه می‌توان چند Import را در کنار یکدیگر قرار داد، اما هرکدام همچنان یک Import Declaration مستقل هستند.

همچنین می‌توان Importها را در یک Module Interface مجدداً Export کرد تا مصرف‌کنندگان آن Module بدون نیاز به Import مستقیم وابستگی‌ها به قابلیت‌های آن‌ها دسترسی داشته باشند.

```
export module Application;

export import Math;
export import Logger;
```

اکنون مصرف‌کننده می‌تواند تنها Module مربوط به Application را Import کند:

```
import Application;

int main()
{
    auto result = add(10, 20);

    return 0;
}
```

در این مثال، Export شدن Importها باعث می‌شود Interface عمومی Moduleهای `Math` و `Logger` نیز از طریق `Application` در دسترس قرار بگیرد.

این قابلیت برای طراحی Facade Module بسیار مفید است.

## **پیاده‌سازی Module در یک فایل**

ساده‌ترین روش استفاده از Modules این است که Interface و Implementation را در یک فایل قرار دهیم.

در این روش، Declarationهای عمومی و Definitionهای مربوط به آن‌ها در یک Translation Unit نوشته می‌شوند.

**فایل Math.cppm:**

```
export module Math;

export int add(int a, int b)
{
    return a + b;
}

export int subtract(int a, int b)
{
    return a - b;
}

int internalCalculation(int value)
{
    return value * 2;
}
```

**فایل main.cpp:**

```
import Math;

#include <iostream>

int main()
{
    std::cout << add(10, 20) << '\n';
    std::cout << subtract(20, 10) << '\n';
}
```

در این ساختار، تمام اجزای Module در یک فایل قرار گرفته‌اند.

### **مزایا**

- سادگی ساختار پروژه
- عدم نیاز به مدیریت چند فایل برای Moduleهای کوچک
- مناسب برای Utility Moduleهای ساده
- مناسب برای Prototypeها و پروژه‌های کوچک

### **معایب**

- افزایش حجم فایل در Moduleهای بزرگ
- ترکیب شدن Interface و Implementation
- دشوارتر شدن نگهداری و توسعه در پروژه‌های پیچیده
- نیاز به کامپایل مجدد Interface در صورت تغییر Definitionهای قرارگرفته در همان فایل

برای Moduleهایی که Implementation پیچیده‌ای دارند، معمولاً جداسازی Interface و Implementation انتخاب بهتری است.

## **جداسازی Interface و Implementation در فایل‌های مختلف**

یکی از قابلیت‌های مهم Modules این است که می‌توان Interface عمومی را از Implementation جدا کرد.

این ساختار از نظر معماری شباهت زیادی به تفکیک Header و Source در روش سنتی دارد، اما مکانیزم ارتباط میان فایل‌ها متفاوت است.

در این روش، Interface در یک فایل و Definitionهای مربوط به آن در یک یا چند فایل دیگر قرار می‌گیرند.

### **تعریف Interface**

**فایل Math.cppm:**

```
export module Math;

export int add(int a, int b);

export int subtract(int a, int b);
```

در این فایل، تنها Declarationهای عمومی قرار دارند.

### **تعریف Implementation**

**فایل Math.cpp:**

```
module Math;

int add(int a, int b)
{
    return a + b;
}

int subtract(int a, int b)
{
    return a - b;
}
```

نکته مهم در این فایل استفاده از دستور زیر است:

```
module Math;
```

این دستور مشخص می‌کند که فایل فعلی یک Module Implementation Unit متعلق به Module با نام `Math` است.

در نتیجه، Definitionهای موجود در این فایل به Declarationهای مربوط به همان Module متصل می‌شوند.

### **استفاده از Module**

**فایل main.cpp:**

```
import Math;

#include <iostream>

int main()
{
    std::cout << add(10, 20) << '\n';
    std::cout << subtract(20, 10) << '\n';
}
```

در این ساختار، مصرف‌کننده فقط به Interface نیاز دارد و از محل قرارگیری Implementation اطلاعی ندارد.

### **تفاوت مهم میان Interface و Implementation**

در فایل Interface از دستور زیر استفاده می‌کنیم:

```
export module Math;
```

اما در فایل Implementation از دستور زیر استفاده می‌کنیم:

```
module Math;
```

وجود یا عدم وجود کلمه `export` در Module Declaration اهمیت اساسی دارد.

| ویژگی                              | Interface Unit        | Implementation Unit |
| ---------------------------------- | --------------------- | ------------------- |
| تعریف Module                       | `export module Math;` | `module Math;`      |
| Export کردن Declaration            | مجاز                  | غیرمجاز             |
| Import کردن Moduleهای دیگر         | مجاز                  | مجاز                |
| داشتن Definition                   | مجاز                  | مجاز                |
| استفاده از قابلیت‌های داخلی Module | مجاز                  | مجاز                |

توجه کنید که Implementation Unit می‌تواند از Declarationهای Export شده توسط Interface استفاده کند، اما خودش نمی‌تواند یک Interface عمومی جدید برای همان Module تعریف کند.

## **استفاده از چند فایل Implementation**

یک Module می‌تواند چندین Implementation Unit داشته باشد.

این قابلیت برای پروژه‌هایی که بخش‌های مختلفی از منطق اجرایی دارند، بسیار کاربردی است.

فرض کنید یک Module به نام `Database` داریم که مسئولیت‌های مختلفی مانند مدیریت Connection، اجرای Query و مدیریت Transaction را بر عهده دارد.

می‌توانیم ساختار زیر را طراحی کنیم:

```
Database/
│
├── Database.cppm
│
├── Connection.cpp
├── Query.cpp
├── Transaction.cpp
│
└── main.cpp
```

**فایل Database.cppm:**

```
export module Database;

export class Connection
{
public:
    void connect();
};

export class Query
{
public:
    void execute();
};

export class Transaction
{
public:
    void begin();
    void commit();
};
```

**فایل Connection.cpp:**

```
module Database;

void Connection::connect()
{
}
```

**فایل Query.cpp:**

```
module Database;

void Query::execute()
{
}
```

**فایل Transaction.cpp:**

```
module Database;

void Transaction::begin()
{
}

void Transaction::commit()
{
}
```

در این ساختار، تمام فایل‌های Implementation به یک Module تعلق دارند.

بنابراین می‌توانند Definitionهای مربوط به Declarationهای عمومی آن Module را در فایل‌های جداگانه پیاده‌سازی کنند.

### **نکته مهم درباره چند Implementation Unit**

وجود چند فایل Implementation برای یک Module کاملاً مجاز است.

اما هر Module همچنان فقط یک Primary Module Interface Unit دارد.

همچنین Definitionهای یک Entity نباید به شکلی تکراری در چند Translation Unit قرار بگیرند که باعث نقض ODR شوند.

برای مثال، اگر یک تابع معمولی در دو Implementation Unit مختلف با یک Definition غیرمجاز تعریف شود، صرفاً به دلیل تعلق هر دو فایل به یک Module، مشکل ODR برطرف نخواهد شد.

## **مدیریت چند Interface File**

در پروژه‌های بزرگ، ممکن است Interface عمومی یک Module آن‌قدر گسترده باشد که نگهداری تمام Declarationها در یک فایل منطقی نباشد.

برای حل این مسئله، ++C20 قابلیتی به نام **Module Interface Partitions** ارائه می‌کند.

با استفاده از این قابلیت می‌توان Interface یک Module را به چند فایل مستقل تقسیم کرد.

هر Partition می‌تواند مسئولیت مشخصی داشته باشد، اما تمام آن‌ها همچنان بخشی از یک Named Module هستند.

برای مثال:

```
Graphics/
│
├── Graphics.cppm
│
├── Graphics-Renderer.cppm
├── Graphics-Texture.cppm
├── Graphics-Shader.cppm
│
├── Renderer.cpp
├── Texture.cpp
└── Shader.cpp
```

در این ساختار:

- `Graphics.cppm`، Interface اصلی Module است.
- `Graphics-Renderer.cppm`، Interface مربوط به Renderer است.
- `Graphics-Texture.cppm`، Interface مربوط به Texture است.
- `Graphics-Shader.cppm`، Interface مربوط به Shader است.

نکته مهم این است که این فایل‌ها Moduleهای مستقل نیستند، بلکه Interface Partitionهای یک Module واحد محسوب می‌شوند.

در بخش‌های بعدی نحوه تعریف و استفاده از این Partitionها را به‌صورت کامل بررسی می‌کنیم.

## **مفهوم Sub-Modules و تفاوت آن با Module Partitions**

یکی از موضوعاتی که ممکن است هنگام طراحی ساختار Modules باعث سردرگمی شود، مفهوم Sub-Module است.

در واقع، استاندارد ++C20 مکانیزم مستقلی با نام Sub-Module به معنای سلسله‌مراتب داخلی یک Module ارائه نمی‌کند.

برای مثال، می‌توانیم دو Module با نام‌های زیر تعریف کنیم:

```
export module Graphics;
```

```
export module Graphics.Renderer;
```

اما وجود نقطه در نام Module به این معنی نیست که `Graphics.Renderer` به‌صورت خودکار بخشی از Module `Graphics` محسوب می‌شود.

این دو، دو Named Module مستقل هستند.

بنابراین اگر بخواهیم بخش‌های مختلف یک Module واحد را سازمان‌دهی کنیم، باید از Module Partitions استفاده کنیم.

برای مثال:

```
export module Graphics:Renderer;
```

در اینجا `Renderer` یک Partition از Module اصلی `Graphics` است.

تفاوت این دو ساختار را می‌توان به شکل زیر خلاصه کرد:

| ویژگی                            | Named Module مستقل  | Module Partition    |
| -------------------------------- | ------------------- | ------------------- |
| مثال                             | `Graphics.Renderer` | `Graphics:Renderer` |
| تعلق به Module اصلی              | خیر                 | بله                 |
| مالکیت مشترک Declarationها       | خیر                 | بله                 |
| قابلیت Import مستقیم از خارج     | بله                 | خیر                 |
| نیاز به Primary Interface مستقل  | بله                 | خیر                 |
| امکان Export از طریق Module اصلی | با `export import`  | بله                 |

بنابراین برای تقسیم‌بندی داخلی یک Module بزرگ، Module Partition انتخاب مناسب‌تری است.

## **معرفی Module Interface Partitions**

یک Module Interface Partition، بخشی از Interface عمومی یک Module است که در یک فایل مستقل تعریف می‌شود.

برای تعریف چنین فایلی از Syntax زیر استفاده می‌کنیم:

```
export module ModuleName:PartitionName;
```

وجود `export` در ابتدای Module Declaration نشان می‌دهد که این Partition یک Interface Unit است.

### **مثال عملی**

فرض کنید می‌خواهیم Module مربوط به Graphics را به دو بخش Renderer و Texture تقسیم کنیم.

**فایل Graphics-Renderer.cppm:**

```
export module Graphics:Renderer;

export class Renderer
{
public:
    void render();
};
```

**فایل Graphics-Texture.cppm:**

```
export module Graphics:Texture;

export class Texture
{
public:
    void load();
};
```

اکنون دو Interface Partition مستقل داریم.

اما برای اینکه مصرف‌کنندگان خارجی بتوانند به این قابلیت‌ها از طریق Module اصلی دسترسی داشته باشند، باید آن‌ها را در Primary Interface معرفی کنیم.

**فایل Graphics.cppm:**

```
export module Graphics;

export import :Renderer;
export import :Texture;
```

در این مثال:

- Module اصلی `Graphics` تعریف شده است.
- Partition مربوط به Renderer وارد شده و مجدداً Export شده است.
- Partition مربوط به Texture نیز وارد شده و مجدداً Export شده است.

اکنون مصرف‌کننده می‌تواند فقط Module اصلی را Import کند.

**فایل main.cpp:**

```
import Graphics;

int main()
{
    Renderer renderer;
    Texture texture;

    renderer.render();
    texture.load();
}
```

مصرف‌کننده نیازی به دانستن نام Partitionها ندارد.

### **نکته مهم درباره Import Partitionها**

Module Partitionها فقط توسط Translation Unitهایی که به همان Named Module تعلق دارند قابل Import هستند.

بنابراین دستور زیر در یک Translation Unit خارجی معتبر نیست:

```
import Graphics:Renderer;
```

اما داخل خود Module می‌توان از آن استفاده کرد:

```
export module Graphics;

import :Renderer;
```

همچنین توجه کنید که یک Interface Partition باید به‌صورت مستقیم یا غیرمستقیم توسط Primary Interface Export شود تا Interface آن از طریق Import کردن Module اصلی در دسترس قرار بگیرد.

## **تفاوت Interface Partition و Implementation Partition**

علاوه بر Interface Partition، می‌توان Partitionهایی تعریف کرد که صرفاً برای سازمان‌دهی Implementation داخلی استفاده می‌شوند.

تفاوت اصلی آن‌ها در وجود یا عدم وجود کلمه `export` است.

### **تعریف Interface Partition**

```
export module Graphics:Renderer;

export class Renderer
{
public:
    void render();
};
```

این Partition می‌تواند Declarationهای عمومی داشته باشد.

### **تعریف Implementation Partition**

```
module Graphics:Internal;

void initializeRenderer()
{
}
```

این Partition بخشی از پیاده‌سازی داخلی Module است.

برای استفاده از آن در Primary Interface یا سایر واحدهای همان Module، می‌توان آن را Import کرد.

**فایل Graphics.cppm:**

```
export module Graphics;

import :Internal;

export void initializeGraphics()
{
    initializeRenderer();
}
```

البته چون تابع `initializeRenderer` Export نشده است، مصرف‌کنندگان خارجی به آن دسترسی ندارند.

### **یک نکته بسیار مهم**

در Module Partitionها، Declarationها و Definitionهای مربوط به Partition واردشده، در محدوده Moduleهای Import کننده آن قابل‌مشاهده هستند؛ حتی اگر Export نشده باشند.

این موضوع تفاوت مهمی میان Partition و یک Named Module مستقل ایجاد می‌کند.

در واقع، Partitionها از مالکیت مشترک Module استفاده می‌کنند.

بنابراین:

- Partitionهای داخلی می‌توانند Implementationهای مشترک داشته باشند.
- Primary Interface می‌تواند به Declarationهای داخلی آن‌ها دسترسی داشته باشد.
- مصرف‌کنندگان خارجی نمی‌توانند Partitionهای داخلی را مستقیماً Import کنند.
- Declarationهای Export نشده، صرفاً با Import کردن Module اصلی به Interface عمومی تبدیل نمی‌شوند.

این قابلیت برای ایجاد یک ساختار داخلی قدرتمند و در عین حال حفظ Interface عمومی کوچک بسیار مفید است.

## **استفاده از Import مجدد یا Export Import**

یکی از قابلیت‌های مهم Modules، امکان Export کردن یک Import است.

این قابلیت به ما اجازه می‌دهد Interface چند Module یا Partition را از طریق یک Module واحد در اختیار مصرف‌کننده قرار دهیم.

برای مثال، فرض کنید سه Module مستقل داریم:

```
export module Math;
```

```
export module Geometry;
```

```
export module Physics;
```

اکنون می‌توانیم یک Module سطح بالاتر ایجاد کنیم.

**فایل Engine.cppm:**

```
export module Engine;

export import Math;
export import Geometry;
export import Physics;
```

در این حالت، هر مصرف‌کننده‌ای که `Engine` را Import کند، به Declarationهای Export شده توسط این سه Module نیز دسترسی خواهد داشت.

**فایل main.cpp:**

```
import Engine;

int main()
{
    auto result = add(10, 20);

    return 0;
}
```

این الگو برای طراحی Moduleهای Facade مناسب است.

### **تفاوت Import و Export Import**

| دستور                 | رفتار                                                                       |
| --------------------- | --------------------------------------------------------------------------- |
| `import Math;`        | قابلیت‌های Module را در Translation Unit فعلی قابل‌دسترسی می‌کند            |
| `export import Math;` | علاوه بر Import، Interface عمومی آن را از طریق Module فعلی نیز منتشر می‌کند |

توجه کنید که Export Import به معنی انتقال مالکیت Declarationها به Module جدید نیست.

هر Declaration همچنان متعلق به Module اصلی خودش است.

## **استفاده از Classها در Modules**

استفاده از Classها در Modules تفاوت اساسی با روش سنتی ندارد.

با این تفاوت که به‌جای قرار دادن Declaration در Header، آن را در Module Interface تعریف یا Declaration می‌کنیم.

### **تعریف کامل Class در Interface**

```
export module Person;

export class Person
{
public:
    Person(const char* name);

    void print() const;

private:
    const char* name_;
};
```

**فایل Person.cpp:**

```
module Person;

#include <iostream>

Person::Person(const char* name)
    : name_(name)
{
}

void Person::print() const
{
    std::cout << name_ << '\n';
}
```

**فایل main.cpp:**

```
import Person;

int main()
{
    Person person("Hadi");

    person.print();
}
```

در این مثال، Class در Interface تعریف شده و Definition متدهای آن در فایل Implementation قرار گرفته است.

### **Export کردن Class**

برای اینکه یک Class در اختیار مصرف‌کنندگان خارجی قرار بگیرد، می‌توانیم خود Class را Export کنیم.

```
export module User;

export class User
{
public:
    void login();
    void logout();

private:
    int id_;
};
```

با Export کردن Class، Declaration عمومی آن و اعضای قابل‌دسترسی آن در Interface قرار می‌گیرند.

البته Export شدن Class به معنی عمومی شدن تمام جزئیات داخلی آن نیست.

برای مثال، اعضای `private` همچنان از قواعد Access Control خود پیروی می‌کنند.

## **استفاده از Class Templateها در Modules**

یکی از موضوعات مهم در استفاده از Modules، مدیریت Templateها است.

در روش سنتی، معمولاً Definition مربوط به Templateها در Header قرار می‌گیرد؛ زیرا کامپایلر برای Instantiate کردن یک Template به Definition آن نیاز دارد.

در Modules نیز همین اصل اهمیت دارد.

اگر یک Class Template را Export کنیم، مصرف‌کننده برای استفاده از آن باید به Definition موردنیاز Template دسترسی داشته باشد.

### **تعریف Class Template در Interface**

**فایل Container.cppm:**

```
export module Container;

export template <typename T>
class Container
{
public:
    explicit Container(T value)
        : value_(value)
    {
    }

    T get() const
    {
        return value_;
    }

private:
    T value_;
};
```

**فایل main.cpp:**

```
import Container;

#include <iostream>

int main()
{
    Container<int> numbers(100);
    Container<double> values(25.5);

    std::cout << numbers.get() << '\n';
    std::cout << values.get() << '\n';
}
```

در این مثال، کامپایلر به Definition کامل Class Template دسترسی دارد و می‌تواند Specializationهای موردنیاز را Instantiate کند.

### **جداسازی Definitionهای Template**

فرض کنید می‌خواهیم Declarationهای عمومی Template را از Definitionهای آن جدا کنیم.

می‌توانیم از یک Interface Partition استفاده کنیم.

**فایل Container-Template.cppm:**

```
export module Container:Template;

export template <typename T>
class Container
{
public:
    explicit Container(T value);

    T get() const;

private:
    T value_;
};

template <typename T>
Container<T>::Container(T value)
    : value_(value)
{
}

template <typename T>
T Container<T>::get() const
{
    return value_;
}
```

**فایل Container.cppm:**

```
export module Container;

export import :Template;
```

در این مثال، Definitionهای Template در یک Interface Partition قرار گرفته‌اند.

مصرف‌کننده با Import کردن `Container` به Interface و Definitionهای موردنیاز دسترسی پیدا می‌کند.

### **آیا می‌توان Definitionهای Template را در فایل cpp معمولی قرار داد؟**

در حالت عمومی، نمی‌توان Definition یک Template را صرفاً در یک Implementation Unit قرار داد و انتظار داشت مصرف‌کننده خارجی بتواند برای Type دلخواه خودش آن را Instantiate کند.

برای مثال، اگر Definition زیر فقط در یک فایل Implementation قرار داشته باشد:

```
module Container;

template <typename T>
T calculate(T value)
{
    return value * 2;
}
```

مصرف‌کننده خارجی به Definition لازم برای Instantiate کردن این Template دسترسی ندارد.

البته یک راهکار وجود دارد: Explicit Instantiation.

برای مثال:

```
export module Container;

export template <typename T>
T calculate(T value);
```

**فایل Container.cpp:**

```
module Container;

template <typename T>
T calculate(T value)
{
    return value * 2;
}

template int calculate<int>(int);
template double calculate<double>(double);
```

در این طراحی، فقط Specializationهای مشخص‌شده در Implementation تولید می‌شوند.

این روش زمانی مناسب است که مجموعه Typeهای موردپشتیبانی مشخص و محدود باشد.

اما اگر مصرف‌کننده باید بتواند Template را برای Typeهای دلخواه خودش Instantiate کند، معمولاً بهتر است Definitionهای Template در Interface یا Interface Partition قابل‌دسترسی قرار بگیرند.

### **نکته معماری درباره Templateها**

برای Templateهای عمومی، بهتر است Interface و Definitionهای لازم برای Instantiation را از هم جدا نکنیم؛ مگر اینکه دلیل مشخصی برای این کار داشته باشیم.

Modules مشکل نیاز به Definition در زمان Instantiation را به‌صورت جادویی برطرف نمی‌کند.

## **استفاده از Namespaceها در Modules**

Modules و Namespaces دو مکانیزم مستقل هستند که اهداف متفاوتی دارند.

- Namespace برای سازمان‌دهی نام‌ها و جلوگیری از Name Collision استفاده می‌شود.
- Module برای سازمان‌دهی Translation Unitها و کنترل انتشار Declarationها میان آن‌ها استفاده می‌شود.

بنابراین می‌توان از Namespaceها در داخل Moduleها استفاده کرد.

### **تعریف Namespace در Module**

```
export module Math;

export namespace Mathematics
{
    int add(int a, int b)
    {
        return a + b;
    }

    int subtract(int a, int b)
    {
        return a - b;
    }
}
```

در این مثال، Namespace مربوط به `Mathematics` و Declarationهای آن Export شده‌اند.

**فایل main.cpp:**

```
import Math;

int main()
{
    auto result = Mathematics::add(10, 20);

    return 0;
}
```

### **Export کردن اعضای مشخص Namespace**

همیشه لازم نیست تمام اعضای یک Namespace را Export کنیم.

برای مثال:

```
export module Math;

namespace Mathematics
{
    export int add(int a, int b)
    {
        return a + b;
    }

    int internalCalculation(int value)
    {
        return value * 2;
    }
}
```

در این ساختار، Namespace مربوط به `Mathematics` از طریق Declaration تابع `add` در Interface قابل‌دسترسی است، اما تابع `internalCalculation` Export نشده است.

مصرف‌کننده می‌تواند از تابع `add` استفاده کند، اما به تابع داخلی دسترسی مستقیم ندارد.

### **Export کردن کل Namespace**

اگر بخواهیم تمام Declarationهای یک Namespace را Export کنیم، می‌توانیم از روش زیر استفاده کنیم:

```
export module Math;

export namespace Mathematics
{
    int add(int a, int b)
    {
        return a + b;
    }

    int subtract(int a, int b)
    {
        return a - b;
    }
}
```

در این حالت، تمام Declarationهای موجود در تعریف Namespace که مشمول قواعد Export هستند، در Interface عمومی قرار می‌گیرند.

### **استفاده از Namespaceهای داخلی**

می‌توانیم Namespaceهایی نیز داشته باشیم که صرفاً برای Implementation داخلی استفاده می‌شوند.

```
export module Math;

namespace Internal
{
    int normalize(int value)
    {
        return value < 0 ? 0 : value;
    }
}

export int calculate(int value)
{
    return Internal::normalize(value) * 2;
}
```

در این مثال، Namespace داخلی بخشی از Interface عمومی نیست و مصرف‌کننده نیازی به دسترسی به آن ندارد.

## **بررسی Visibility و Reachability در Modules**

یکی از مهم‌ترین و در عین حال پیچیده‌ترین بخش‌های سیستم Modules، تفاوت میان Visibility و Reachability است.

برای درک درست Modules نباید Export را صرفاً معادل Public و عدم Export را معادل Private در نظر بگیریم.

این دو مفهوم به یکدیگر مرتبط هستند، اما یکسان نیستند.

### **مفهوم Visibility**

Visibility مشخص می‌کند که آیا یک Declaration در محدوده موردنظر قابل‌مشاهده و قابل‌استفاده برای Name Lookup است یا خیر.

برای مثال:

```
export module Math;

export int add(int a, int b);

int internalAdd(int a, int b);
```

در این مثال، مصرف‌کننده خارجی می‌تواند Declaration مربوط به `add` را مشاهده کند، اما Declaration مربوط به `internalAdd` از طریق Interface عمومی در اختیار او قرار نمی‌گیرد.

### **مفهوم Reachability**

Reachability مفهومی عمیق‌تر از Visibility است.

در ++C، Reachability مشخص می‌کند که آیا یک Declaration یا Definition طبق قواعد استاندارد، در نقطه موردنظر قابل‌دسترسی است یا خیر.

این موضوع اهمیت زیادی در موارد زیر دارد:

- Template Instantiation
- Name Lookup
- ODR
- Definitionهای مربوط به Classها
- Declarationهای Export شده
- Module Partitionها

ممکن است یک Declaration مستقیماً برای Name Lookup قابل‌مشاهده نباشد، اما همچنان Reachable باشد.

به همین دلیل نباید این دو مفهوم را یکسان در نظر گرفت.

### **تفاوت Visibility و Reachability با Access Control**

موضوع دیگری که باید از این دو مفهوم جدا شود، Access Control است.

Access Control همان مکانیزم آشنای `public`، `protected` و `private` در Classها است.

برای مثال:

```
export module User;

export class User
{
public:
    void login();

private:
    int password_;
};
```

در این مثال:

- Class مربوط به User Export شده است.
- Declaration متد `login` بخشی از Interface عمومی است.
- عضو `password_` همچنان Private است.

بنابراین حتی اگر Class برای مصرف‌کننده قابل‌دسترسی باشد، Access Control همچنان محدودیت‌های خودش را اعمال می‌کند.

### **مقایسه مفاهیم**

| مفهوم          | مسئولیت                                              |
| -------------- | ---------------------------------------------------- |
| Export         | انتشار Declaration در Interface عمومی                |
| Visibility     | امکان مشاهده Declaration در Name Lookup              |
| Reachability   | در دسترس بودن Declaration طبق قواعد استاندارد        |
| Access Control | کنترل دسترسی به اعضای Class                          |
| Linkage        | تعیین ارتباط هویتی یک Entity میان Translation Unitها |

این مفاهیم را باید جداگانه در طراحی Modules در نظر گرفت.

## **مفهوم Linkage در Modules**

در مدل سنتی ++C، معمولاً با دو نوع Linkage اصلی آشنا هستیم:

- Internal Linkage
- External Linkage

اما با معرفی Modules، نوع دیگری نیز وارد زبان شد:

**Module Linkage**

به‌صورت ساده، Declarationهایی که در محدوده یک Named Module تعریف می‌شوند و شرایط Module Linkage را دارند، می‌توانند میان Translation Unitهای متعلق به همان Module ارتباط داشته باشند، بدون اینکه به‌عنوان Entityهای عمومی دارای External Linkage در اختیار سایر Moduleها قرار بگیرند.

برای مثال:

**فایل Math.cppm:**

```
export module Math;

int internalValue = 100;

export int getValue()
{
    return internalValue;
}
```

**فایل Math.cpp:**

```
module Math;

int calculate()
{
    return internalValue * 2;
}
```

در این مثال، `internalValue` به‌صورت مستقیم Export نشده است، اما Translation Unitهای متعلق به Module `Math` می‌توانند به آن دسترسی داشته باشند.

این موضوع یکی از تفاوت‌های مهم مدل Modules با روش سنتی Header است.

## **استفاده از فایل cpp در کنار Modules**

یکی از سؤالات رایج هنگام مهاجرت از معماری Header/Source به Modules این است که آیا می‌توان همچنان از فایل‌های `.cpp` معمولی استفاده کرد؟

پاسخ مثبت است.

Modules هیچ اجباری برای استفاده از پسوند خاصی برای فایل‌ها ایجاد نمی‌کند.

پسوندهایی مانند موارد زیر صرفاً قراردادهای رایج ابزارها و Build Systemها هستند:

| پسوند   | کاربرد رایج                       |
| ------- | --------------------------------- |
| `.cppm` | Module Interface                  |
| `.ixx`  | Module Interface در اکوسیستم MSVC |
| `.cpp`  | Implementation                    |
| `.cc`   | Implementation                    |
| `.cxx`  | Implementation                    |

نکته مهم این است که کامپایلر و Build System باید بدانند هر فایل چه نقشی دارد.

### **استفاده از فایل cpp برای Implementation**

**فایل Logger.cppm:**

```
export module Logger;

export class Logger
{
public:
    void log(const char* message);
};
```

**فایل Logger.cpp:**

```
module Logger;

#include <iostream>

void Logger::log(const char* message)
{
    std::cout << message << '\n';
}
```

در این مثال، فایل `.cpp` به‌عنوان یک Module Implementation Unit شناخته می‌شود.

### **استفاده از فایل cpp معمولی در کنار Module**

همچنین می‌توان فایل‌هایی داشت که اصلاً Module Unit نیستند.

برای مثال:

**فایل Utility.cpp:**

```
#include <iostream>

void printMessage()
{
    std::cout << "Hello\n";
}
```

این فایل یک Translation Unit معمولی است و به هیچ Module مشخصی تعلق ندارد.

اگر بخواهیم از Declaration آن در یک Module استفاده کنیم، باید مکانیزم مناسب برای ارتباط با آن را طراحی کنیم؛ مثلاً از Header یا روش‌های سازگار دیگر استفاده کنیم.

نکته مهم این است که صرفاً قرار دادن یک فایل `.cpp` کنار فایل‌های Module باعث نمی‌شود آن فایل به Module موردنظر تعلق پیدا کند.

برای تعلق داشتن به Named Module باید Module Declaration مربوطه در آن وجود داشته باشد.

## **معرفی Global Module Fragment**

گاهی اوقات هنگام استفاده از Modules همچنان به Headerهای قدیمی نیاز داریم.

برای مثال، ممکن است یک Library قدیمی فقط از طریق Header در دسترس باشد یا به Macroهای تعریف‌شده در Header نیاز داشته باشیم.

در چنین شرایطی می‌توان از قابلیتی به نام **Global Module Fragment** استفاده کرد.

برای شروع Global Module Fragment از دستور زیر استفاده می‌شود:

```
module;
```

این دستور باید قبل از Module Declaration قرار بگیرد.

### **مثال عملی**

```
module;

#include <vector>
#include <string>

#define MAX_USERS 100

export module UserManager;

export class UserManager
{
public:
    void addUser(const std::string& name);

private:
    std::vector<std::string> users_;
};
```

در این مثال:

- ابتدا Global Module Fragment آغاز شده است.
- Headerهای موردنیاز در این بخش Include شده‌اند.
- سپس Module اصلی تعریف شده است.
- Class مربوط به UserManager Export شده است.

### **چرا Global Module Fragment وجود دارد؟**

در حالت معمول، Declarationهای موجود در Headerهای قدیمی که در محدوده یک Named Module قرار می‌گیرند، ممکن است از نظر Ownership و Linkage با انتظارات Libraryهای سنتی سازگار نباشند.

Global Module Fragment راهکاری برای استفاده از Headerهای قدیمی بدون اینکه Declarationهای آن‌ها به‌صورت معمول به Named Module فعلی متصل شوند، فراهم می‌کند.

همچنین Macroهای تعریف‌شده در این بخش می‌توانند در ادامه همان Translation Unit استفاده شوند.

اما Macroها از طریق Import کردن Module در اختیار مصرف‌کنندگان قرار نمی‌گیرند.

### **نکته مهم**

Global Module Fragment را نباید جایگزین دائمی طراحی مناسب Modules دانست.

این قابلیت بیشتر برای سازگاری با کدهای Legacy و وابستگی‌هایی مفید است که هنوز به مکانیزم Header متکی هستند.

## **معرفی Private Module Fragment**

یکی از قابلیت‌های کمتر شناخته‌شده Modules در ++C20، **Private Module Fragment** است.

این قابلیت با استفاده از دستور زیر آغاز می‌شود:

```
module : private;
```

Private Module Fragment بخشی از Primary Module Interface Unit است که پس از آن قرار می‌گیرد.

Declarationها و Definitionهای موجود در این بخش برای مصرف‌کنندگان خارجی Module قابل‌دسترسی نیستند.

### **مثال عملی**

**فایل Math.cppm:**

```
export module Math;

export int add(int a, int b);

module : private;

int internalCalculation(int value)
{
    return value * 2;
}

int add(int a, int b)
{
    return internalCalculation(a + b);
}
```

در این مثال:

- تابع `add` در بخش عمومی Module Declaration شده است.
- Private Module Fragment آغاز شده است.
- تابع `internalCalculation` در بخش خصوصی قرار گرفته است.
- Definition تابع `add` نیز در همین بخش قرار گرفته است.

مصرف‌کننده همچنان می‌تواند تابع `add` را فراخوانی کند، اما به تابع داخلی دسترسی ندارد.

**فایل main.cpp:**

```
import Math;

int main()
{
    return add(10, 20);
}
```

### **محدودیت بسیار مهم Private Module Fragment**

اگر یک Module Unit دارای Private Module Fragment باشد، آن Module باید فقط از همان یک Module Unit تشکیل شده باشد.

به عبارت دیگر، نمی‌توان برای همان Module در کنار فایل دارای Private Module Fragment، چند Implementation Unit یا Partition مستقل تعریف کرد.

بنابراین این قابلیت برای Moduleهای کوچک و تک‌فایلی مناسب‌تر است.

### **تفاوت Private Module Fragment با Implementation Unit**

| ویژگی                           | Private Module Fragment        | Implementation Unit         |
| ------------------------------- | ------------------------------ | --------------------------- |
| محل قرارگیری                    | انتهای Primary Interface Unit  | فایل Translation Unit مستقل |
| نیاز به فایل جداگانه            | خیر                            | بله                         |
| امکان چند فایل Implementation   | خیر                            | بله                         |
| امکان تقسیم Module به Partition | خیر                            | بله                         |
| هدف                             | جداسازی بخش خصوصی در همان فایل | جداسازی Implementation      |

برای پروژه‌های بزرگ، معمولاً استفاده از Implementation Unitهای مستقل انعطاف‌پذیرتر است.

## **استفاده از Module داخلی در یک Module Export شده**

یکی از کاربردهای مهم Module Partitionها، ایجاد بخش‌های داخلی است که مصرف‌کنندگان خارجی هیچ دسترسی مستقیمی به آن‌ها نداشته باشند.

فرض کنید یک Module به نام `Engine` داریم که دارای قابلیت‌های عمومی و داخلی است.

می‌خواهیم:

- Interface عمومی Engine در اختیار مصرف‌کننده باشد.
- Implementation داخلی در یک فایل مستقل قرار بگیرد.
- Declarationهای داخلی در Moduleهای دیگر قابل‌استفاده نباشند.
- مصرف‌کننده فقط API عمومی را مشاهده کند.

برای این کار می‌توانیم از Internal Module Partition استفاده کنیم.

### **تعریف Partition داخلی**

**فایل Engine-Internal.cppm:**

```
module Engine:Internal;

void initializeSubsystem()
{
}

int calculateInternalValue()
{
    return 100;
}
```

توجه کنید که در این فایل از `export module` استفاده نشده است.

بنابراین این فایل یک Internal Module Partition محسوب می‌شود.

### **استفاده از Partition داخلی**

**فایل Engine.cppm:**

```
export module Engine;

import :Internal;

export class Engine
{
public:
    void initialize()
    {
        initializeSubsystem();
    }

    int getValue() const
    {
        return calculateInternalValue();
    }
};
```

در این مثال:

- Module اصلی `Engine` تعریف شده است.
- Partition داخلی Import شده است.
- Class عمومی `Engine` Export شده است.
- متدهای عمومی از قابلیت‌های داخلی استفاده می‌کنند.

### **استفاده توسط مصرف‌کننده خارجی**

**فایل main.cpp:**

```
import Engine;

int main()
{
    Engine engine;

    engine.initialize();

    return engine.getValue();
}
```

مصرف‌کننده فقط Module اصلی را Import می‌کند.

دستور زیر در یک Translation Unit خارجی مجاز نیست:

```
import Engine:Internal;
```

زیرا Partitionها فقط درون همان Named Module قابل Import هستند.

### **نکته معماری مهم**

در این مثال، Partition داخلی بخشی از Module اصلی است، نه یک Named Module مستقل.

به همین دلیل Declarationهای آن در اختیار Translation Unitهای داخلی همان Module قرار می‌گیرند.

اما چون Partition به‌صورت عمومی Export نشده است، مصرف‌کنندگان خارجی نمی‌توانند Interface آن را مستقیماً دریافت کنند.

این روش برای موارد زیر بسیار مفید است:

- Internal Helpers
- الگوریتم‌های داخلی
- جزئیات پیاده‌سازی
- Subsystemهای داخلی
- Declarationهای مشترک میان Implementation Unitها
- کاهش حجم Interface عمومی

**توجه:** این روش را نباید با Private Module Fragment اشتباه گرفت. Internal Module Partition امکان داشتن فایل‌های متعدد و ساختار ماژولار را فراهم می‌کند، در حالی که Private Module Fragment محدود به یک Module Unit است.

## **مقایسه Header و Module**

یکی از اهداف اصلی Modules کاهش مشکلات روش سنتی Header است.

با این حال، Modules و Headerها کاملاً جایگزین یکدیگر نیستند و هرکدام ویژگی‌های متفاوتی دارند.

| ویژگی                       | Header و Include               | Modules و Import                  |
| --------------------------- | ------------------------------ | --------------------------------- |
| مکانیزم اشتراک‌گذاری        | Textual Inclusion              | Module Import                     |
| پردازش مجدد Declarationها   | معمولاً در هر Translation Unit | استفاده از BMI                    |
| Macroها                     | قابل انتشار از طریق Include    | به‌صورت معمول Export نمی‌شوند     |
| کنترل Interface             | وابسته به ساختار Header        | با Export                         |
| جداسازی Implementation      | معمولاً با cpp                 | با Implementation Unit            |
| مدیریت وابستگی              | عمدتاً مبتنی بر Include        | مبتنی بر Module Dependency        |
| ترتیب Include               | ممکن است مهم باشد              | وابستگی‌ها با Import مشخص می‌شوند |
| قابلیت استفاده از کد Legacy | بسیار گسترده                   | نیازمند سازگاری یا روش‌های ترکیبی |

### **مزایای Modules**

**کاهش زمان کامپایل**

یکی از مهم‌ترین مزایای Modules این است که کامپایلر می‌تواند از اطلاعات کامپایل‌شده Interface استفاده کند و مجبور نیست در هر Translation Unit تمام Declarationهای Header را مجدداً پردازش کند.

البته میزان بهبود به ساختار پروژه، وابستگی‌ها، کامپایلر و Build System بستگی دارد.

**جلوگیری از انتشار Macroها**

Macroهای تعریف‌شده در یک Named Module به‌صورت خودکار به مصرف‌کننده منتقل نمی‌شوند.

این موضوع از بسیاری از مشکلات مربوط به Name Collision و آلودگی فضای Preprocessor جلوگیری می‌کند.

**کنترل بهتر Interface عمومی**

با استفاده از `export` می‌توان مشخص کرد کدام Declarationها باید در اختیار مصرف‌کنندگان قرار بگیرند.

**کاهش وابستگی‌های غیرضروری**

مصرف‌کننده به‌جای مشاهده تمام جزئیات Header، فقط Interface قابل‌دسترسی Module را دریافت می‌کند.

### **محدودیت‌های Modules**

با وجود مزایای متعدد، Modules محدودیت‌هایی نیز دارند:

- نیاز به پشتیبانی مناسب کامپایلر
- نیاز به Build System سازگار با Module Dependency
- پیچیدگی بیشتر در تنظیم Build
- محدودیت‌های سازگاری با برخی Libraryهای قدیمی
- نیاز به مدیریت فایل‌های BMI
- تفاوت در نحوه تولید و ذخیره BMI میان کامپایلرها
- دشواری مهاجرت تدریجی در برخی پروژه‌های Legacy

همچنین Modules الزاماً باعث افزایش سرعت تمام پروژه‌ها نمی‌شوند. اگر یک پروژه وابستگی‌های کم و Headerهای سبک داشته باشد، ممکن است مزیت قابل‌توجهی مشاهده نشود.

## **مفهوم BMI و نحوه کامپایل Modules**

یکی از تفاوت‌های بنیادی Modules با Headerها، نحوه پردازش آن‌ها توسط کامپایلر است.

در روش سنتی، فایل Header مستقیماً توسط Preprocessor وارد Translation Unit می‌شود.

اما در Modules، کامپایلر ابتدا باید Interface مربوط به Module را پردازش کند و اطلاعات لازم را در قالب BMI تولید کند.

**BMI (Built Module Interface)** شامل اطلاعات کامپایل‌شده‌ای است که کامپایلر برای استفاده از Module به آن نیاز دارد.

برای مثال، فرآیند کلی را می‌توان به شکل زیر نمایش داد:

```
              Math.cppm
                  |
                  |
              Compiler
                  |
                  |
             Math BMI
                  |
                  |
        +---------+---------+
        |                   |
        |                   |
      main.cpp          Engine.cpp
        |                   |
        +---------+---------+
                  |
                  |
              Compiler
                  |
                  |
             Object Files
                  |
                  |
                Linker
                  |
                  |
              Executable
```

در این مدل، ابتدا Interface کامپایل می‌شود تا BMI تولید شود.

سپس فایل‌هایی که Module را Import می‌کنند، می‌توانند با استفاده از BMI کامپایل شوند.

### **آیا BMI یک فایل استاندارد و قابل‌انتقال است؟**

خیر.

استاندارد ++C قالب باینری مشخص و مشترکی برای BMI تعیین نمی‌کند.

بنابراین:

- فرمت BMI به کامپایلر وابسته است.
- BMI تولیدشده توسط یک کامپایلر لزوماً توسط کامپایلر دیگر قابل‌استفاده نیست.
- حتی نسخه‌های مختلف یک کامپایلر ممکن است سازگاری کاملی با BMIهای یکدیگر نداشته باشند.
- تنظیمات کامپایل، Target و سایر ویژگی‌های Toolchain می‌توانند در سازگاری BMI تأثیر داشته باشند.

در نتیجه، BMI را نباید مانند یک فایل Object استاندارد یا Library قابل‌انتقال میان Toolchainهای مختلف در نظر گرفت.

### **مدیریت Dependencyها**

در پروژه‌های دارای Modules، Build System باید ترتیب کامپایل را بر اساس Importها مشخص کند.

برای مثال، اگر Module مربوط به `Engine`، Module مربوط به `Math` را Import کرده باشد، ابتدا باید BMI مربوط به Math تولید شود.

سپس کامپایل Engine می‌تواند انجام شود.

این موضوع باعث می‌شود Build Systemهای سازگار با Modules اهمیت بیشتری پیدا کنند.

## **استفاده از Modules در CMake**

برای استفاده عملی از Modules در پروژه‌های واقعی، بهتر است از Build Systemهایی استفاده کنیم که قابلیت تشخیص وابستگی‌های Module را دارند.

یکی از گزینه‌های مناسب، CMake است.

پشتیبانی از C++ Modules در CMake از نسخه 3.28 معرفی شده است.

در این نسخه‌ها، برای مدیریت Moduleها می‌توان از File Set مخصوصی با نام `CXX_MODULES` استفاده کرد.

### **ساختار پروژه**

```
MyProject/
│
├── CMakeLists.txt
│
├── Math.cppm
├── Math.cpp
│
└── main.cpp
```

### **فایل CMakeLists.txt**

```
cmake_minimum_required(VERSION 3.28)

project(MyProject LANGUAGES CXX)

add_executable(MyProject
    main.cpp
    Math.cpp
)

target_sources(MyProject
    PUBLIC
    FILE_SET CXX_MODULES
    TYPE CXX_MODULES
    FILES
        Math.cppm
)

target_compile_features(MyProject PRIVATE cxx_std_20)
```

در این مثال:

- فایل `Math.cppm` به‌عنوان Module Interface معرفی شده است.
- فایل `Math.cpp` به‌عنوان Implementation معمولی در Target قرار گرفته است.
- استاندارد ++C20 فعال شده است.
- CMake می‌تواند وابستگی‌های Module را برای ترتیب صحیح کامپایل بررسی کند.

### **نکته مهم درباره CMake**

صرفاً استفاده از CMake نسخه 3.28 یا بالاتر به معنی پشتیبانی کامل از تمام قابلیت‌های Modules نیست.

پشتیبانی به موارد زیر وابسته است:

- نسخه کامپایلر
- Generator انتخاب‌شده
- نسخه Build Tool
- قابلیت Dependency Scanning
- محدودیت‌های فعلی CMake

برای مثال، Generatorهایی مانند Ninja و Visual Studio پشتیبانی‌هایی برای Module Scanning دارند، اما تمام قابلیت‌های آن‌ها یکسان نیست.

بنابراین بهتر است قبل از انتخاب ساختار پروژه، سازگاری کامل Toolchain را بررسی کنیم.

## **بررسی وضعیت پشتیبانی کامپایلرها**

قابلیت Modules از نظر زبان در ++C20 استاندارد شده است، اما پشتیبانی عملی کامپایلرها و ابزارهای Build در طول زمان تکامل پیدا کرده است.

چند نکته مهم:

- **MSVC:** پشتیبانی مناسبی از Modules دارد و از فرمت `.ixx` نیز به‌صورت رایج برای Interface استفاده می‌کند.
- **Clang:** پشتیبانی از Standard C++ Modules را ارائه می‌کند، اما باید نسخه دقیق کامپایلر و Build System بررسی شود.
- **GCC:** پشتیبانی از Modules در نسخه‌های جدیدتر بهبود یافته است و امکانات مختلف آن ممکن است میان نسخه‌ها تفاوت داشته باشد.

همچنین باید میان پشتیبانی از خود Modules و پشتیبانی از Standard Library Modules تفاوت قائل شویم.

برای مثال، قابلیت زیر:

```
import std;
```

به‌عنوان قابلیت مربوط به Standard Library Modules در ++C23 استاندارد شده است و نباید آن را یک قابلیت عمومی و تضمین‌شده در تمام Toolchainهای ++C20 در نظر گرفت.

حتی در نسخه‌های جدیدتر نیز پشتیبانی عملی آن به ترکیب کامپایلر و Standard Library وابسته است.

## **استفاده از Header Unitها**

Modules علاوه بر Named Moduleها، مفهوم دیگری به نام **Header Unit** نیز معرفی می‌کند.

Header Unit راهکاری برای Import کردن Headerهای موجود با استفاده از مکانیزم Module Import است.

برای مثال:

```
import <iostream>;
import <vector>;

int main()
{
    std::vector<int> values{10, 20, 30};
}
```

در این مثال، Headerها به‌جای استفاده از `#include` با دستور `import` وارد شده‌اند.

### **تفاوت Header Unit و Named Module**

| ویژگی                     | Header Unit                                 | Named Module                 |
| ------------------------- | ------------------------------------------- | ---------------------------- |
| منبع                      | Header موجود                                | Module Interface             |
| نیاز به تعریف Module      | خیر                                         | بله                          |
| Export صریح Declarationها | خیر                                         | بله                          |
| Macroها                   | می‌توانند در دسترس Import کننده قرار بگیرند | به‌صورت معمول منتشر نمی‌شوند |
| هدف                       | سازگاری با Headerها                         | طراحی Interface ماژولار      |

Header Unitها برای مهاجرت تدریجی از Headerها مفید هستند.

با این حال، نباید آن‌ها را کاملاً معادل Named Module دانست.

## **ترکیب Modules و Headerهای سنتی**

در پروژه‌های واقعی، معمولاً لازم نیست تمام کدهای موجود را یک‌باره به Modules تبدیل کنیم.

می‌توانیم از ترکیب دو روش استفاده کنیم.

برای مثال:

```
import Math;

#include <iostream>
#include <vector>

int main()
{
    std::vector<int> values{10, 20, 30};

    for (int value : values)
    {
        std::cout << add(value, 10) << '\n';
    }
}
```

در این مثال، Module مربوط به Math از طریق Import و Headerهای Standard Library از طریق Include استفاده شده‌اند.

این روش برای پروژه‌هایی که به‌صورت تدریجی در حال مهاجرت هستند، کاربردی است.

## **نکات مهم در طراحی Modules**

استفاده صحیح از Modules فقط به یادگیری Syntaxهای `export` و `import` محدود نمی‌شود.

برای طراحی یک معماری مناسب، باید به نحوه سازمان‌دهی Interface، وابستگی‌ها و جزئیات داخلی توجه کنیم.

### **طراحی Interface کوچک**

بهتر است فقط Declarationهایی را Export کنیم که واقعاً بخشی از API عمومی هستند.

Export کردن تمام Declarationهای داخلی باعث افزایش وابستگی مصرف‌کنندگان به جزئیات پیاده‌سازی می‌شود.

### **جداسازی Implementation**

برای Moduleهای بزرگ، بهتر است Definitionهای متدها و منطق اجرایی را در فایل‌های Implementation مستقل قرار دهیم.

این کار باعث می‌شود تغییرات Implementation تا حد امکان Interface عمومی را تحت تأثیر قرار ندهند.

### **استفاده مناسب از Partitionها**

Partitionها را برای تقسیم‌بندی منطقی یک Module بزرگ استفاده کنید.

برای مثال:

```
Database
 ├── Database:Connection
 ├── Database:Query
 ├── Database:Transaction
 └── Database:Internal
```

این ساختار از نظر سازمان‌دهی داخلی مناسب است، زیرا تمام بخش‌ها همچنان متعلق به یک Module هستند.

### **جلوگیری از Dependency Cycle**

وابستگی‌های دوری میان Moduleها می‌توانند باعث مشکلات جدی در ساختار و فرآیند کامپایل شوند.

بهتر است وابستگی‌ها به شکل یک Directed Acyclic Graph یا DAG طراحی شوند.

برای مثال:

```
Core
 |
 +------+
 |      |
Math   Logger
 |      |
 +------+
    |
  Engine
    |
Application
```

در این ساختار، وابستگی‌ها جهت مشخصی دارند و از ایجاد چرخه جلوگیری شده است.

### **استفاده از Facade Module**

در پروژه‌های بزرگ، می‌توان یک Module اصلی ایجاد کرد که Interface چند Module دیگر را Export کند.

این روش باعث ساده‌تر شدن API مصرف‌کنندگان می‌شود.

### **جلوگیری از انتشار Macroها**

یکی از مزایای Modules این است که Macroها به‌صورت خودکار از Module به مصرف‌کننده منتقل نمی‌شوند.

بنابراین بهتر است طراحی جدید را بر اساس Macroهای عمومی بنا نکنیم و در صورت امکان از `constexpr`، Templateها، Functionها و سایر قابلیت‌های زبان استفاده کنیم.

### **توجه به Templateها**

در طراحی Templateهای عمومی، Definitionهای لازم برای Instantiation باید در دسترس مصرف‌کننده باشند.

قرار دادن Definitionها در Implementation Unit بدون در نظر گرفتن Explicit Instantiation معمولاً انتخاب مناسبی نیست.

## **اشتباهات رایج در استفاده از Modules**

در این بخش چند اشتباه رایج را بررسی می‌کنیم.

### **تصور اینکه Export کردن یک Module تمام Declarationهای آن را Export می‌کند**

دستور زیر:

```
export module Math;
```

به معنی Export شدن تمام Declarationهای موجود در فایل نیست.

فقط Declarationهایی که مطابق قواعد Export در Interface قرار گرفته‌اند، در اختیار مصرف‌کنندگان قرار می‌گیرند.

### **تصور اینکه Moduleهای دارای نام مشابه یک Module واحد هستند**

برای مثال:

```
export module Graphics;
```

و:

```
export module Graphics.Renderer;
```

دو Module مستقل هستند.

وجود نقطه در نام Module باعث ایجاد رابطه مالکیت یا سلسله‌مراتب نمی‌شود.

### **استفاده اشتباه از Module Declaration**

در Interface اصلی:

```
export module Math;
```

و در Implementation:

```
module Math;
```

استفاده می‌شود.

اشتباه گرفتن این دو می‌تواند باعث تعریف نادرست ساختار Module شود.

### **قرار دادن Import بعد از Declarationهای معمولی**

در Module Unit، Import Declarationها باید بعد از Module Declaration و قبل از سایر Declarationهای معمولی قرار بگیرند.

### **تلاش برای Import کردن Partition از خارج Module**

دستور زیر در Translation Unit خارجی مجاز نیست:

```
import Graphics:Internal;
```

Partitionها فقط از داخل همان Named Module قابل Import هستند.

### **فرض اینکه Modules جایگزین کامل Headerها هستند**

Modules بسیاری از مشکلات Headerها را کاهش می‌دهند، اما همچنان ممکن است برای سازگاری با Libraryهای قدیمی، Macroها و برخی APIها به Header نیاز داشته باشیم.

### **قرار دادن Definitionهای Template فقط در Implementation**

این کار می‌تواند باعث شود مصرف‌کننده نتواند Template را برای Typeهای موردنیاز خودش Instantiate کند.

### **نادیده گرفتن وابستگی‌های Build**

در Modules، ترتیب کامپایل اهمیت دارد.

اگر BMI یک Module قبل از کامپایل مصرف‌کنندگان آن تولید نشود، Build با مشکل مواجه خواهد شد.

بنابراین مدیریت Dependencyها باید توسط Build System سازگار انجام شود.

## **چه زمانی از Modules استفاده کنیم؟**

استفاده از Modules در تمام پروژه‌ها الزاماً بهترین انتخاب نیست.

برای تصمیم‌گیری بهتر، باید اندازه پروژه، نوع وابستگی‌ها، Toolchain و معماری آن را بررسی کنیم.

| شرایط پروژه                        | پیشنهاد                                          |
| ---------------------------------- | ------------------------------------------------ |
| پروژه جدید و بزرگ                  | استفاده از Modules انتخاب مناسبی است             |
| Library عمومی جدید                 | Modules می‌تواند Interface مناسبی فراهم کند      |
| پروژه با Headerهای بسیار سنگین     | مهاجرت تدریجی می‌تواند مفید باشد                 |
| پروژه Legacy با وابستگی‌های فراوان | ابتدا سازگاری Toolchain بررسی شود                |
| پروژه کوچک با چند فایل             | Headerهای معمولی ممکن است کافی باشند             |
| پروژه با کامپایلر قدیمی            | استفاده از Header منطقی‌تر است                   |
| پروژه دارای Templateهای گسترده     | Modules با طراحی مناسب Template قابل‌استفاده است |

### **پیشنهاد برای پروژه‌های بزرگ**

برای پروژه‌های حرفه‌ای، بهتر است ابتدا یک Module مستقل و نسبتاً ساده را از ساختار Header/Source به Modules تبدیل کنیم.

سپس موارد زیر را بررسی کنیم:

- زمان کامپایل
- پیچیدگی Build
- سازگاری کامپایلرها
- نحوه مدیریت BMI
- میزان وابستگی Interface
- نحوه انتشار Library
- سازگاری با کدهای Legacy

پس از ارزیابی نتایج، می‌توان مهاجرت را به بخش‌های دیگر پروژه گسترش داد.

## **جمع‌بندی**

Modules یکی از مهم‌ترین قابلیت‌های معماری ++C20 است که مدل جدیدی برای سازمان‌دهی کد، کنترل Interface و اشتراک‌گذاری Declarationها میان Translation Unitها ارائه می‌کند.

مهم‌ترین قابلیت‌های این سیستم عبارت‌اند از:

- **Module Interface:** تعریف API عمومی یک Module
- **Module Implementation:** جداسازی Definitionها از Interface
- **Export و Import:** کنترل انتشار و مصرف قابلیت‌های Module
- **Export Block:** گروه‌بندی Declarationهای عمومی
- **Multiple Implementation Units:** تقسیم Implementation میان چند فایل
- **Module Interface Partitions:** تقسیم Interface یک Module بزرگ به چند بخش
- **Internal Module Partitions:** سازمان‌دهی بخش‌های داخلی و غیرعمومی
- **Export Import:** انتشار Interface یک Module از طریق Module دیگر
- **Private Module Fragment:** ایجاد بخش خصوصی در یک Module تک‌فایلی
- **BMI:** استفاده از اطلاعات کامپایل‌شده Interface برای Import

مهم‌ترین نکته در طراحی حرفه‌ای Modules این است که میان Interface عمومی و جزئیات داخلی پیاده‌سازی مرز مشخصی ایجاد کنیم.

همچنین باید به خاطر داشته باشیم که Named Moduleها، Namespaceها و Module Partitionها سه مفهوم متفاوت هستند و هرکدام مسئولیت مشخصی در معماری ++C دارند.

در نهایت، Modules صرفاً جایگزینی برای `#include` نیست؛ بلکه تغییر مهمی در مدل کامپایل، مدیریت وابستگی‌ها و طراحی API در ++C محسوب می‌شود.

## **منابع و مراجع**

برای مطالعه عمیق‌تر و بررسی جزئیات استاندارد، منابع زیر پیشنهاد می‌شوند:

- [cppreference - Modules (C++20)](https://en.cppreference.com/w/cpp/language/modules?utm_source=chatgpt.com) — مرجع جامع Syntax، Export، Import، Partitionها و قواعد زبان.&#x20;

  [image](https://www.google.com/s2/favicons?domain=https://www.cppreference.com\&sz=32)

  cppreference.com+1

- [Clang - Standard C++ Modules](https://clang.llvm.org/docs/StandardCPlusPlusModules.html?utm_source=chatgpt.com) — بررسی تخصصی معماری Modules، Reachability و نحوه پیاده‌سازی آن در Clang.&#x20;

  [image](https://www.google.com/s2/favicons?domain=https://clang.llvm.org\&sz=32)

  Clang

- [Microsoft Learn - Overview of Modules in C++](https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?utm_source=chatgpt.com) — مثال‌های عملی برای Interface، Implementation و Partitionها.&#x20;

  [image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

  Microsoft Learn

- [CMake - C++ Modules](https://cmake.org/cmake/help/latest/manual/cmake-cxxmodules.7.html?utm_source=chatgpt.com) — راهنمای مدیریت وابستگی‌ها، BMI و پشتیبانی Build System از Modules.&#x20;

  [image](https://www.google.com/s2/favicons?domain=https://cmake.org\&sz=32)

  CMake 4.5.0 Documentation

- [ISO C++20 Working Draft - N4860](https://isocpp.org/files/papers/N4860.pdf?utm_source=chatgpt.com) — متن Working Draft استاندارد ++C20 برای بررسی قواعد دقیق زبان.&#x20;

  [image](https://www.google.com/s2/favicons?domain=https://isocpp.org\&sz=32)

  isocpp.org

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>