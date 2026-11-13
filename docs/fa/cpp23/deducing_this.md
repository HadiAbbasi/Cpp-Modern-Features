<div align="center">

[🇺🇸 English](../../en/cpp23/deducing_this.md) | [🇮🇷 فارسی](./deducing_this.md)

</div>

---

# قابلیت Deducing `this` در ++C23

## مقدمه

قابلیت **Deducing `this`** یکی از ویژگی‌های مهم استاندارد ++C23 است که با هدف ساده‌سازی طراحی Member Functionها، کاهش تکرار کد، بهبود پیاده‌سازی `CRTP` و افزایش انعطاف‌پذیری Generic Programming معرفی شده است.

این قابلیت که در پیشنهاد استانداردسازی **P0847R7 – Deducing this** تعریف شده است، امکان استفاده از یک پارامتر صریح برای دریافت شیء جاری در Member Functionها را فراهم می‌کند.

پیش از ++C23، کامپایلر به‌صورت ضمنی یک پارامتر به نام Object Parameter برای هر Non-static Member Function در نظر می‌گرفت. این پارامتر با استفاده از کلمه کلیدی `this` در بدنه تابع قابل دسترسی بود، اما برنامه‌نویس امکان تعریف مستقیم نوع آن را نداشت.

این محدودیت باعث می‌شد در بسیاری از طراحی‌ها، به‌خصوص در موارد زیر، به پیاده‌سازی‌های تکراری نیاز باشد:

- پیاده‌سازی Overloadهای مختلف برای `const` و Non-const Member Functionها.
- مدیریت تفاوت میان Lvalue و Rvalue Objectها.
- طراحی APIهای زنجیره‌ای یا Fluent Interface.
- پیاده‌سازی الگوهای `CRTP`.
- تعریف Recursive Lambdaها.
- طراحی Generic Member Functionهایی که رفتارشان به نوع دقیق شیء وابسته است.

قابلیت Deducing `this` این محدودیت را برطرف می‌کند و اجازه می‌دهد نوع Object Parameter به‌صورت صریح یا با استفاده از Template Argument Deduction مشخص شود.

به کمک این ویژگی، می‌توان یک Member Function را طوری طراحی کرد که بدون نیاز به چندین Overload، ویژگی‌های `const` بودن، Reference بودن و نوع دقیق شیء را حفظ کند.

---

## مفهوم Object Parameter در ++C

### نحوه عملکرد this پیش از ++C23

در استانداردهای قبلی، هر Non-static Member Function دارای یک Implicit Object Parameter بود.

برای مثال، کد زیر را در نظر بگیرید:

```
class Widget {
public:
    void update(int value) {
        // Update the object
    }
};
```

از دید مفهومی، تابع `update` به یک شیء از نوع `Widget` نیاز دارد تا روی آن اجرا شود.

هنگامی که تابع به شکل زیر فراخوانی می‌شود:

```
Widget widget;
widget.update(42);
```

شیء `widget` به‌صورت ضمنی به تابع منتقل می‌شود.

در داخل تابع، کلمه کلیدی `this` به Pointer مربوط به همان شیء اشاره می‌کند.

بنابراین، در یک Non-static Member Function معمولی، کامپایلر نوع `this` را بر اساس نوع کلاس و Qualifierهای تابع تعیین می‌کند.

برای مثال:

```
class Widget {
public:
    void modify() {
        // this has type Widget*
    }

    void inspect() const {
        // this has type const Widget*
    }
};
```

در تابع `modify`، نوع `this` برابر `Widget*` و در تابع `inspect`، نوع آن برابر `const Widget*` است.

نکته مهم این است که `this` در این مدل یک پارامتر معمولی نیست و امکان تعریف مستقیم نوع آن توسط برنامه‌نویس وجود ندارد.

### محدودیت‌های مدل سنتی

فرض کنید قصد داریم یک تابع Getter طراحی کنیم که Reference مناسبی به شیء جاری برگرداند.

در یک API حرفه‌ای، ممکن است بخواهیم رفتار تابع بر اساس وضعیت شیء متفاوت باشد:

- برای یک شیء Mutable از نوع Lvalue، یک Reference معمولی برگردانده شود.
- برای یک شیء `const` از نوع Lvalue، یک `const` Reference برگردانده شود.
- برای یک شیء از نوع Rvalue، یک Rvalue Reference برگردانده شود.

پیش از ++C23، معمولاً باید برای این رفتارها چند Overload تعریف می‌کردیم:

```
class Widget {
public:
    Widget& get() & {
        return *this;
    }

    const Widget& get() const & {
        return *this;
    }

    Widget&& get() && {
        return std::move(*this);
    }
};
```

این روش کاملاً معتبر و در بسیاری از APIها همچنان مناسب است، اما با افزایش تعداد Qualifierها و پیچیدگی توابع، میزان کد تکراری بیشتر می‌شود.

قابلیت Deducing `this` امکان پیاده‌سازی همین رفتار را با یک تابع فراهم می‌کند.

---

## معرفی Explicit Object Parameter

### ساختار اصلی Syntax

در ++C23 می‌توان اولین پارامتر یک Non-static Member Function را با کلمه کلیدی `this` تعریف کرد.

این پارامتر با نام **Explicit Object Parameter** شناخته می‌شود.

ساختار کلی آن به شکل زیر است:

```
class Widget {
public:
    void update(this Widget& self, int value) {
        // Use self to access the current object
    }
};
```

در این مثال:

- عبارت `this Widget&` مشخص می‌کند که پارامتر اول، Object Parameter تابع است.
- نام `self` نامی است که برنامه‌نویس برای دسترسی به شیء جاری انتخاب کرده است.
- نوع `Widget&` مشخص می‌کند که تابع فقط با یک Lvalue از نوع `Widget` سازگار است.

نحوه فراخوانی تابع همچنان مشابه Member Functionهای معمولی است:

```
Widget widget;
widget.update(42);
```

در این فراخوانی، کامپایلر شیء `widget` را به پارامتر `self` متصل می‌کند.

همچنین می‌توان به شکل زیر تابع را از طریق نام کلاس و با ارائه صریح Object Argument فراخوانی کرد:

```
Widget::update(widget, 42);
```

این شکل از فراخوانی در مواردی که نیاز به درک نحوه اتصال Object Parameter وجود دارد، مفید است.

### تفاوت Implicit و Explicit Object Parameter

تفاوت اصلی این دو مدل در نحوه تعریف و دسترسی به شیء جاری است.

| ویژگی                                       | Implicit Object Parameter           | Explicit Object Parameter              |
| ------------------------------------------- | ----------------------------------- | -------------------------------------- |
| تعریف مستقیم پارامتر                        | خیر                                 | بله                                    |
| دسترسی به شیء جاری                          | از طریق `this`                      | از طریق پارامتر نام‌گذاری‌شده          |
| تعیین نوع Object Parameter                  | توسط زبان                           | توسط برنامه‌نویس یا Template Deduction |
| پشتیبانی از Template Deduction برای نوع شیء | محدود به مکانیزم‌های معمول Overload | بله                                    |
| استفاده در Lambda                           | به شکل Implicit Capture یا `this`   | به شکل Explicit Object Parameter       |
| امکان کنترل Reference Category              | از طریق Ref-qualified Overloadها    | از طریق نوع پارامتر یا Deduction       |

### تفاوت Explicit Object Parameter با Static Member Function

یکی از نکات مهم این است که Explicit Object Member Function با Static Member Function یکسان نیست.

یک Static Member Function فاقد Object Parameter است و به شیء جاری وابستگی ضمنی یا صریح ندارد.

اما Explicit Object Member Function همچنان یک Non-static Member Function محسوب می‌شود و اولین پارامتر آن وظیفه دریافت شیء جاری را بر عهده دارد.

برای مثال:

```
class Widget {
public:
    void regular(this Widget& self) {
        self.value = 10;
    }

    static void staticFunction() {
    }

private:
    int value = 0;
};
```

در این مثال، تابع `regular` یک Explicit Object Member Function است، در حالی که `staticFunction` یک Static Member Function محسوب می‌شود.

---

## مفهوم Deducing this

### استفاده از auto برای استنتاج نوع شیء

مهم‌ترین قابلیت این ویژگی، امکان استفاده از `auto` در Explicit Object Parameter است.

برای مثال:

```
class Widget {
public:
    void inspect(this auto&& self) {
        // The type of self is deduced
    }
};
```

در این مثال، کامپایلر نوع پارامتر `self` را بر اساس نوع و Value Category شیئی که تابع روی آن فراخوانی شده است، استنتاج می‌کند.

این قابلیت باعث می‌شود یک تابع بتواند برای انواع مختلف Objectها مورد استفاده قرار بگیرد، بدون اینکه به Overloadهای جداگانه نیاز داشته باشد.

برای مثال:

```
#include <iostream>
#include <type_traits>

class Widget {
public:
    void inspect(this auto&& self) {
        using SelfType = decltype(self);

        std::cout << std::boolalpha;
        std::cout << std::is_lvalue_reference_v<SelfType> << '\n';
        std::cout << std::is_const_v<
            std::remove_reference_t<SelfType>
        > << '\n';
    }
};

int main() {
    Widget widget;
    const Widget constWidget;

    widget.inspect();
    constWidget.inspect();
    Widget{}.inspect();
}
```

در این مثال، نوع `self` در فراخوانی‌های مختلف متفاوت خواهد بود:

| نوع شیء                         | نوع استنتاج‌شده برای `self` |
| ------------------------------- | --------------------------- |
| Lvalue از نوع `Widget`          | `Widget&`                   |
| Lvalue از نوع `const Widget`    | `const Widget&`             |
| Rvalue از نوع `Widget`          | `Widget&&`                  |
| Lvalue از نوع `volatile Widget` | `volatile Widget&`          |

نکته مهم این است که `auto&&` در این موقعیت از قواعد Forwarding Reference پیروی می‌کند.

بنابراین، می‌توان Qualifierها و Reference Category شیء اصلی را حفظ کرد.

### تفاوت auto&& با Widget&&

استفاده از `auto&&` با تعریف یک Rvalue Reference معمولی تفاوت اساسی دارد.

برای مثال:

```
class Widget {
public:
    void process(this Widget&& self) {
    }

    void inspect(this auto&& self) {
    }
};
```

تابع `process` فقط با Rvalueهایی از نوع `Widget` سازگار است.

اما تابع `inspect` می‌تواند Lvalue و Rvalue و همچنین نسخه‌های `const` را بپذیرد.

به همین دلیل، استفاده از `auto&&` برای Generic Programming بسیار کاربردی است.

---

## بررسی Qualifierها و Value Category

### مدیریت const بودن شیء

یکی از مزایای مهم Deducing `this` این است که می‌توان نوع شیء را بدون نوشتن Overloadهای جداگانه دریافت کرد.

برای مثال:

```
#include <iostream>
#include <type_traits>

class Widget {
public:
    void inspect(this auto&& self) {
        using ObjectType = std::remove_reference_t<
            decltype(self)
        >;

        if constexpr (std::is_const_v<ObjectType>) {
            std::cout << "Const object\n";
        } else {
            std::cout << "Mutable object\n";
        }
    }
};

int main() {
    Widget widget;
    const Widget constWidget;

    widget.inspect();
    constWidget.inspect();
}
```

در این پیاده‌سازی، با استفاده از `if constexpr` رفتار تابع بر اساس Constness شیء انتخاب می‌شود.

این تکنیک زمانی مفید است که یک الگوریتم مشترک برای نسخه‌های Mutable و Const یک کلاس داشته باشیم.

البته اگر تنها تفاوت موردنیاز، امکان تغییر وضعیت شیء باشد، استفاده از یک تابع `const` یا Overloadهای معمولی ممکن است طراحی ساده‌تر و خواناتری ارائه دهد.

### مدیریت Lvalue و Rvalue

مفهوم Value Category نقش مهمی در طراحی APIهای مدرن دارد.

با استفاده از `auto&&` می‌توان تشخیص داد که شیء جاری Lvalue است یا Rvalue.

برای مثال:

```
#include <iostream>
#include <type_traits>

class Widget {
public:
    void inspect(this auto&& self) {
        if constexpr (
            std::is_lvalue_reference_v<decltype(self)>
        ) {
            std::cout << "Lvalue object\n";
        } else {
            std::cout << "Rvalue object\n";
        }
    }
};

int main() {
    Widget widget;

    widget.inspect();
    Widget{}.inspect();
}
```

این قابلیت به‌خصوص برای طراحی APIهایی که باید بین استفاده از منابع موجود و انتقال منابع تفاوت قائل شوند، مفید است.

### استفاده از decltype(auto)

در کنار Deducing `this`، می‌توان از `decltype(auto)` برای حفظ دقیق نوع بازگشتی استفاده کرد.

برای مثال:

```
class Widget {
public:
    int value = 42;

    decltype(auto) getValue(this auto&& self) {
        return (self.value);
    }
};
```

در این مثال، پرانتزهای اطراف عبارت `self.value` اهمیت زیادی دارند.

قواعد `decltype` باعث می‌شوند که نوع بازگشتی بر اساس Value Category عبارت تعیین شود.

بنابراین:

- برای Lvalue، نتیجه یک Lvalue Reference خواهد بود.
- برای Rvalue، نتیجه یک Rvalue Reference خواهد بود.

در نتیجه، نوع بازگشتی این تابع می‌تواند به شکل زیر باشد:

| فراخوانی          | نوع بازگشتی  |
| ----------------- | ------------ |
| روی Lvalue معمولی | `int&`       |
| روی Const Lvalue  | `const int&` |
| روی Rvalue معمولی | `int&&`      |

نکته مهم این است که برگرداندن Reference به اعضای یک شیء موقت می‌تواند خطرناک باشد.

برای مثال:

```
auto&& value = Widget{}.getValue();
```

پس از پایان Full-expression مربوط به این دستور، شیء موقت از بین می‌رود و `value` به Reference معلق تبدیل می‌شود.

بنابراین، حفظ Value Category همیشه به معنای ایمن بودن Lifetime نیست.

---

## کاربردهای مهم Deducing this

### کاهش تعداد Overloadها

یکی از کاربردهای اصلی این قابلیت، حذف Overloadهای تکراری است.

فرض کنید یک کلاس باید یک تابع Getter داشته باشد که Reference مناسب به شیء جاری برگرداند.

پیاده‌سازی سنتی به شکل زیر است:

```
class Widget {
public:
    Widget& get() & {
        return *this;
    }

    const Widget& get() const & {
        return *this;
    }

    Widget&& get() && {
        return std::move(*this);
    }

    const Widget&& get() const && {
        return std::move(*this);
    }
};
```

با استفاده از Deducing `this` می‌توان این چهار Overload را به یک تابع تبدیل کرد:

```
class Widget {
public:
    auto&& get(this auto&& self) {
        return std::forward<decltype(self)>(self);
    }
};
```

در این طراحی، `std::forward` نوع و Value Category شیء را حفظ می‌کند.

این روش به‌خصوص برای کلاس‌هایی مفید است که تعداد زیادی Member Function دارند و برای هرکدام به نسخه‌های مختلف `const` و Ref-qualified نیاز پیدا می‌کنند.

### طراحی Fluent Interface

در طراحی Fluent Interface معمولاً می‌خواهیم فراخوانی چند تابع به‌صورت زنجیره‌ای امکان‌پذیر باشد.

یکی از مشکلات پیاده‌سازی سنتی، نیاز به Overloadهای متعدد برای حفظ Reference Category شیء است.

برای مثال:

```
class Builder {
public:
    Builder& setName(const char* name) & {
        name_ = name;
        return *this;
    }

    Builder&& setName(const char* name) && {
        name_ = name;
        return std::move(*this);
    }

private:
    std::string name_;
};
```

در ++C23 می‌توان این طراحی را ساده‌تر کرد:

```
#include <string>
#include <utility>

class Builder {
public:
    auto&& setName(
        this auto&& self,
        std::string name
    ) {
        self.name_ = std::move(name);

        return std::forward<decltype(self)>(self);
    }

    auto&& setAge(
        this auto&& self,
        int age
    ) {
        self.age_ = age;

        return std::forward<decltype(self)>(self);
    }

private:
    std::string name_;
    int age_ = 0;
};
```

اکنون می‌توان از هر دو شکل زیر استفاده کرد:

```
int main() {
    Builder builder;

    builder.setName("Alice")
           .setAge(30);

    auto result = Builder{}
        .setName("Bob")
        .setAge(25);
}
```

در این پیاده‌سازی، Reference Category شیء در طول زنجیره حفظ می‌شود.

نکته مهم این است که در صورت فراخوانی روی یک Rvalue، شیء همچنان موقت باقی می‌ماند و نباید Reference حاصل از آن پس از پایان Lifetime شیء استفاده شود.

### پیاده‌سازی CRTP

الگوی `Curiously Recurring Template Pattern` یا CRTP یکی از کاربردهای مهم Template Programming در ++C است.

در این الگو، کلاس مشتق‌شده به‌عنوان Template Argument به کلاس پایه منتقل می‌شود.

برای مثال:

```
template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived&>(*this).implementation();
    }
};

class Widget : public Base<Widget> {
public:
    void implementation() {
    }
};
```

یکی از محدودیت‌های طراحی سنتی این است که در کلاس پایه باید نوع `Derived` را برای دسترسی به شیء مشتق‌شده به کار ببریم.

قابلیت Deducing `this` امکان پیاده‌سازی مشابهی را بدون نیاز به Template Parameter کلاس فراهم می‌کند:

```
class Base {
public:
    void interface(this auto&& self) {
        self.implementation();
    }
};

class Widget : public Base {
public:
    void implementation() {
    }
};
```

در این مثال، نوع واقعی شیء از طریق Explicit Object Parameter استنتاج می‌شود.

به این ترتیب، کلاس پایه می‌تواند بدون داشتن Template Parameter به API کلاس مشتق‌شده دسترسی داشته باشد.

این روش گاهی با عنوان **CRTP-less Design** یا طراحی مشابه CRTP بدون Template شناخته می‌شود.

البته این دو الگو کاملاً معادل نیستند.

در CRTP کلاس پایه می‌تواند از نوع مشتق‌شده در نقاط مختلف تعریف کلاس استفاده کند؛ اما در روش Deducing `this`، نوع مشتق‌شده در زمان استفاده از تابع استنتاج می‌شود.

بنابراین، Deducing `this` جایگزین عمومی تمام کاربردهای CRTP نیست.

### پیاده‌سازی Generic Interface

یکی از کاربردهای مهم این ویژگی، طراحی Interfaceهایی است که رفتارشان به نوع واقعی شیء وابسته است.

برای مثال:

```
#include <iostream>

class Printable {
public:
    void print(this const auto& self) {
        self.printImpl();
    }
};

class User : public Printable {
public:
    void printImpl() const {
        std::cout << "User\n";
    }
};

class Product : public Printable {
public:
    void printImpl() const {
        std::cout << "Product\n";
    }
};
```

در این مثال، تابع `print` در کلاس پایه برای هر نوع مشتق‌شده به‌صورت جداگانه Instantiate می‌شود.

به همین دلیل، `self` نوع واقعی شیء را دارد و می‌تواند تابع مناسب را فراخوانی کند.

این روش نوعی Static Polymorphism ایجاد می‌کند.

توجه کنید که این روش به Virtual Function وابسته نیست و Dispatch در زمان Compile انجام می‌شود.

---

## کاربرد Deducing this در Lambda

### مشکل Recursive Lambdaهای سنتی

یکی از کاربردهای بسیار جذاب Deducing `this`، ساده‌سازی Recursive Lambdaها است.

در نسخه‌های قدیمی‌تر ++C، نوشتن Lambda بازگشتی معمولاً به یک متغیر واسط یا استفاده از `std::function` نیاز داشت.

برای مثال:

```
#include <functional>

int main() {
    std::function<int(int)> factorial;

    factorial = [&](int n) -> int {
        if (n <= 1) {
            return 1;
        }

        return n * factorial(n - 1);
    };

    return factorial(5);
}
```

این روش کار می‌کند، اما استفاده از `std::function` می‌تواند هزینه‌های اضافی داشته باشد.

از جمله این هزینه‌ها می‌توان به Type Erasure و احتمال Dynamic Allocation اشاره کرد.

### استفاده از Explicit Object Parameter در Lambda

از ++C23 می‌توان Recursive Lambda را مستقیماً با Explicit Object Parameter تعریف کرد.

برای مثال:

```
#include <iostream>

int main() {
    auto factorial = [](this auto&& self, int n) -> int {
        if (n <= 1) {
            return 1;
        }

        return n * self(n - 1);
    };

    std::cout << factorial(5) << '\n';
}
```

در این مثال:

- پارامتر `self` به Closure Object جاری اشاره می‌کند.
- Lambda می‌تواند با فراخوانی `self` خودش را دوباره اجرا کند.
- نیازی به `std::function` وجود ندارد.
- نوع Closure در زمان Compile مشخص است.
- امکان Inline شدن فراخوانی‌ها برای Compiler وجود دارد.

این روش معمولاً از `std::function` ساده‌تر و از نظر Type System کارآمدتر است.

### نمونه Recursive Lambda برای پیمایش درخت

یکی از کاربردهای عملی Recursive Lambda، پیمایش ساختارهای درختی است.

```
#include <iostream>
#include <memory>

struct Node {
    int value;

    std::unique_ptr<Node> left;
    std::unique_ptr<Node> right;
};

int main() {
    auto root = std::make_unique<Node>(
        Node{1, nullptr, nullptr}
    );

    root->left = std::make_unique<Node>(
        Node{2, nullptr, nullptr}
    );

    root->right = std::make_unique<Node>(
        Node{3, nullptr, nullptr}
    );

    auto traverse = [](this auto&& self, const Node* node) -> void {
        if (node == nullptr) {
            return;
        }

        std::cout << node->value << '\n';

        self(node->left.get());
        self(node->right.get());
    };

    traverse(root.get());
}
```

در این مثال، Lambda به‌صورت Recursive یک درخت دودویی را پیمایش می‌کند.

نکته مهم این است که Recursive بودن Lambda هیچ نیازی به تغییر نوع متغیر یا استفاده از Wrapperهای اضافی ندارد.

### استفاده از Lambdaهای دارای State

در Lambdaهای دارای Capture نیز می‌توان از Explicit Object Parameter استفاده کرد.

برای مثال:

```
#include <iostream>

int main() {
    int count = 0;

    auto counter = [&count](this auto&& self) {
        ++count;

        if (count < 5) {
            self();
        }
    };

    counter();

    std::cout << count << '\n';
}
```

در این مثال، Lambda از طریق `self` خودش را دوباره فراخوانی می‌کند و وضعیت Capture شده را تغییر می‌دهد.

توجه داشته باشید که Explicit Object Parameter جایگزین Capture نیست.

این پارامتر صرفاً امکان دسترسی صریح به Closure Object را فراهم می‌کند.

---

## بررسی تفاوت‌های مهم با ویژگی‌های قبلی

### تفاوت با CRTP سنتی

هر دو روش می‌توانند برای پیاده‌سازی Static Polymorphism استفاده شوند، اما تفاوت‌هایی دارند.

| ویژگی                                               | CRTP                      | Deducing this      |
| --------------------------------------------------- | ------------------------- | ------------------ |
| نیاز به Template Parameter کلاس پایه                | بله                       | خیر                |
| نیاز به `static_cast` برای دسترسی به Derived        | معمولاً بله               | معمولاً خیر        |
| دسترسی به نوع مشتق‌شده                              | از طریق Template Argument | از طریق Deduction  |
| امکان استفاده از نوع مشتق‌شده در کل تعریف کلاس پایه | بله                       | محدودتر            |
| پیچیدگی تعریف کلاس پایه                             | بیشتر                     | کمتر               |
| مناسب برای Generic Base Class                       | بله                       | در بسیاری از موارد |

یکی از نکات مهم این است که Deducing `this` در تمام موقعیت‌ها جایگزین CRTP نیست.

برای مثال، اگر کلاس پایه نیاز داشته باشد از نوع مشتق‌شده در تعریف Data Memberها، Base Classها یا سایر Template Declarationها استفاده کند، CRTP همچنان انتخاب مناسبی است.

### تفاوت با std::forward

گاهی ممکن است Deducing `this` با `std::forward` اشتباه گرفته شود.

این دو ویژگی وظایف متفاوتی دارند.

قابلیت Deducing `this` نوع Object Parameter را دریافت یا استنتاج می‌کند.

در مقابل، تابع `std::forward` برای حفظ Value Category هنگام انتقال یک Reference استفاده می‌شود.

برای مثال:

```
#include <utility>

class Widget {
public:
    void process(this auto&& self) {
        helper(std::forward<decltype(self)>(self));
    }

private:
    template <typename T>
    void helper(T&& value) {
    }
};
```

در این مثال، `self` نوع شیء را مشخص می‌کند و `std::forward` آن را با همان Reference Category به تابع دیگری انتقال می‌دهد.

استفاده از `std::forward` زمانی اهمیت دارد که بخواهیم رفتار Forwarding Reference را حفظ کنیم.

### تفاوت با std::move

تابع `std::move` همیشه یک Rvalue Reference تولید می‌کند، اما این کار به معنای انتقال خودکار منابع نیست.

در مقابل، `std::forward` نوع و Value Category اولیه را حفظ می‌کند.

بنابراین، برای پیاده‌سازی تابعی که باید هم Lvalue و هم Rvalue را پشتیبانی کند، معمولاً `std::forward` انتخاب مناسب‌تری است.

برای مثال:

```
#include <utility>

class Widget {
public:
    auto&& get(this auto&& self) {
        return std::forward<decltype(self)>(self);
    }
};
```

در این مثال، استفاده از `std::move` باعث می‌شد حتی Lvalueها نیز به‌صورت Rvalue برگردانده شوند.

---

## جزئیات پیشرفته زبان

### محدودیت‌های Explicit Object Parameter

با وجود انعطاف‌پذیری بالا، Explicit Object Parameter دارای محدودیت‌های زبانی مشخصی است.

مهم‌ترین محدودیت‌ها عبارت‌اند از:

- پارامتر Explicit Object باید اولین پارامتر Member Function باشد.
- یک تابع نمی‌تواند هم‌زمان دارای Explicit Object Parameter و Implicit Object Parameter باشد.
- یک Member Function نمی‌تواند بیش از یک Explicit Object Parameter داشته باشد.
- Explicit Object Member Function نمی‌تواند `static` باشد.
- Explicit Object Member Function نمی‌تواند `virtual` باشد.
- برای چنین تابعی نمی‌توان از CV-Qualifierها و Ref-Qualifierهای معمول پس از Declarator استفاده کرد.
- Explicit Object Parameter نمی‌تواند Default Argument داشته باشد.
- استفاده از Explicit Object Parameter برای Constructorها و Destructorها مجاز نیست.
- استفاده از آن برای Conversion Functionها مجاز نیست.

برای مثال، تعریف زیر معتبر نیست:

```
class Widget {
public:
    void process(this Widget& self) const;
};
```

علت این است که CV-Qualifier معمول Member Function در این ساختار قابل استفاده نیست.

به‌جای آن، باید Constness را در نوع Explicit Object Parameter مشخص کرد:

```
class Widget {
public:
    void process(this const Widget& self);
};
```

همچنین برای پشتیبانی از هر دو نوع Mutable و Const می‌توان از `auto&&` استفاده کرد.

### محدودیت Virtual Functionها

Explicit Object Member Function نمی‌تواند Virtual باشد.

برای مثال، تعریف زیر معتبر نیست:

```
class Base {
public:
    virtual void execute(this Base& self);
};
```

دلیل این محدودیت، تفاوت مدل Explicit Object Parameter با مدل معمول Virtual Dispatch است.

برای طراحی Polymorphism پویا، همچنان باید از Virtual Member Functionهای معمولی استفاده کرد.

برای مثال:

```
class Base {
public:
    virtual void execute() = 0;

    virtual ~Base() = default;
};
```

بنابراین، Deducing `this` را نباید جایگزین عمومی Virtual Functionها در نظر گرفت.

### تفاوت نوع Pointer به تابع

یکی از جزئیات مهم این قابلیت، تفاوت Pointer Type مربوط به Explicit Object Member Function است.

در Member Functionهای معمولی، آدرس تابع از نوع Pointer to Member Function خواهد بود.

اما Explicit Object Member Function دارای Pointer Type معمولی به تابع است.

برای مثال:

```
class Widget {
public:
    void regular(int value) {
    }

    void explicitObject(this Widget& self, int value) {
    }
};

int main() {
    void (Widget::*memberPointer)(int) = &Widget::regular;

    void (*functionPointer)(Widget&, int) =
        &Widget::explicitObject;
}
```

این تفاوت برای Callbackها، Function Pointerها و طراحی APIهایی که از Member Function Pointer استفاده می‌کنند اهمیت زیادی دارد.

### نحوه فراخوانی Explicit Object Member Function

تابع دارای Explicit Object Parameter همچنان یک Member Function است و می‌توان آن را با Syntax معمول Member Function فراخوانی کرد.

برای مثال:

```
class Widget {
public:
    void execute(this Widget& self, int value) {
    }
};

int main() {
    Widget widget;

    widget.execute(10);

    Widget::execute(widget, 20);
}
```

در شکل اول، شیء از طریق Member Function Call Syntax به تابع منتقل می‌شود.

در شکل دوم، Object Argument به‌صورت صریح ارائه شده است.

نکته مهم این است که در هر دو حالت، پارامتر اول تابع همان شیء جاری است.

### دسترسی به اعضای کلاس

در Explicit Object Member Function، برای دسترسی به اعضای شیء باید از نام پارامتر Explicit Object استفاده شود.

برای مثال:

```
class Widget {
public:
    void update(this Widget& self) {
        self.value = 100;
    }

private:
    int value = 0;
};
```

در این مدل، استفاده از `self.value` روش صحیح دسترسی به عضو شیء است.

همچنین Explicit Object Member Function نمی‌تواند مانند یک Member Function معمولی از `this` برای دریافت Implicit Object Pointer استفاده کند.

این موضوع یکی از تفاوت‌های مفهومی مهم میان دو مدل است.

### استفاده از Template Parameter صریح

علاوه بر `auto`، می‌توان Explicit Object Parameter را با Template Parameter تعریف کرد.

برای مثال:

```
#include <type_traits>

class Widget {
public:
    template <typename Self>
    void inspect(this Self&& self) {
        static_assert(
            std::is_same_v<
                std::remove_cvref_t<Self>,
                Widget
            >
        );
    }
};
```

در این مثال، نوع شیء از طریق Template Parameter به نام `Self` استنتاج می‌شود.

این روش زمانی کاربرد دارد که بخواهیم علاوه بر نوع شیء، از Template Parameter در سایر قسمت‌های تابع نیز استفاده کنیم.

برای مثال، می‌توان بر اساس `Self` از Constraints، Type Traits یا `if constexpr` استفاده کرد.

### استفاده از Concepts

قابلیت Deducing `this` می‌تواند با Concepts معرفی‌شده در ++C20 ترکیب شود.

برای مثال:

```
#include <concepts>

class Widget {
public:
    template <typename Self>
        requires std::same_as<
            std::remove_cvref_t<Self>,
            Widget
        >
    void execute(this Self&& self) {
    }
};
```

در این مثال، تابع فقط برای Objectهایی قابل استفاده است که نوع اصلی آن‌ها `Widget` باشد.

این روش برای محدود کردن Generic APIها و جلوگیری از Instantiationهای ناخواسته مفید است.

البته در این مثال، محدودیت نوع چندان ضروری نیست، زیرا تابع از ابتدا درون کلاس `Widget` تعریف شده است؛ اما همین تکنیک در طراحی Templateهای عمومی کاربرد بیشتری دارد.

---

## نکات مهم در طراحی و پیاده‌سازی

### انتخاب نوع مناسب برای Explicit Object Parameter

نوع Explicit Object Parameter باید بر اساس نیاز واقعی API انتخاب شود.

چند الگوی متداول عبارت‌اند از:

| تعریف                     | کاربرد                                   |
| ------------------------- | ---------------------------------------- |
| `this Widget& self`       | پذیرش Lvalue Mutable                     |
| `this const Widget& self` | پذیرش Lvalue و Rvalue Const و Non-const  |
| `this Widget&& self`      | پذیرش Rvalue Mutable                     |
| `this auto& self`         | استنتاج نوع شیء با محدودیت Lvalue        |
| `this auto&& self`        | پشتیبانی Generic از انواع Value Category |
| `this const auto& self`   | دسترسی فقط خواندنی به شیء با Deduction   |

انتخاب `auto&&` همیشه بهترین تصمیم نیست.

اگر تابع باید فقط روی Lvalue اجرا شود، استفاده از `auto&` محدودیت مناسبی ایجاد می‌کند.

همچنین اگر تابع هیچ نیازی به Rvalue ندارد، پذیرش Rvalue ممکن است باعث ایجاد APIهای ناخواسته شود.

### حفظ Lifetime

یکی از اشتباهات رایج، بازگرداندن Reference به شیئی است که ممکن است عمر آن به پایان رسیده باشد.

برای مثال:

```
class Widget {
public:
    auto&& get(this auto&& self) {
        return static_cast<decltype(self)&&>(self);
    }
};
```

این تابع Value Category شیء را حفظ می‌کند، اما اگر روی یک Temporary فراخوانی شود، Reference برگشتی پس از پایان عمر Temporary معتبر نخواهد بود.

بنابراین، استفاده از `auto&&` و `decltype(auto)` باید با درک کامل قواعد Lifetime انجام شود.

در APIهایی که امکان بازگرداندن Reference به شیء موقت وجود دارد، می‌توان از طراحی محدودکننده‌تر استفاده کرد.

برای مثال:

```
class Widget {
public:
    Widget& get() & {
        return *this;
    }

    const Widget& get() const & {
        return *this;
    }

    Widget&& get() && = delete;
};
```

در چنین طراحی‌ای، فراخوانی روی Rvalue به‌صورت صریح ممنوع می‌شود.

### جلوگیری از Over-Generalization

یکی از مزایای Generic Programming، کاهش کد تکراری است؛ اما استفاده بیش از اندازه از آن می‌تواند خوانایی API را کاهش دهد.

برای مثال، اگر یک تابع صرفاً باید یک مقدار را از شیء بخواند، ممکن است تعریف زیر کافی باشد:

```
class Widget {
public:
    int getValue() const {
        return value_;
    }

private:
    int value_ = 42;
};
```

در این حالت، استفاده از Explicit Object Parameter مزیت مشخصی ایجاد نمی‌کند.

بهتر است از Deducing `this` زمانی استفاده شود که واقعاً به رفتار وابسته به نوع شیء، Constness یا Value Category نیاز داریم.

### توجه به Access Control

Explicit Object Member Function همچنان عضو کلاس محسوب می‌شود.

بنابراین، قواعد Access Control برای دسترسی به اعضای خصوصی و محافظت‌شده همچنان برقرار هستند.

برای مثال:

```
class Widget {
public:
    void update(this auto& self) {
        self.value_ = 10;
    }

private:
    int value_ = 0;
};
```

در این مثال، تابع می‌تواند به عضو خصوصی کلاس دسترسی داشته باشد.

استنتاج نوع `self` باعث تغییر قوانین Encapsulation یا Access Control نمی‌شود.

### توجه به Template Instantiation

وقتی از `auto` یا Template Parameter در Explicit Object Parameter استفاده می‌کنیم، تابع می‌تواند برای انواع مختلف شیء Instantiate شود.

در نتیجه، ممکن است چند نسخه از تابع در زمان Compile تولید شوند.

این موضوع به‌خصوص زمانی اهمیت دارد که بدنه تابع پیچیده باشد یا Templateهای دیگری را فراخوانی کند.

از نظر کارایی، این مدل معمولاً امکان Optimization و Inlining مناسبی فراهم می‌کند؛ اما افزایش تعداد Instantiationها می‌تواند حجم Binary و زمان Compilation را افزایش دهد.

---

## اشتباهات رایج

### تصور اینکه Deducing this یک تابع Static است

با وجود اینکه Explicit Object Member Function یک Object Parameter صریح دارد، همچنان Non-static Member Function است.

تفاوت در نحوه تعریف Object Parameter است، نه در ماهیت عضویت تابع در کلاس.

### استفاده نادرست از this در بدنه تابع

در Explicit Object Member Function باید از پارامتر نام‌گذاری‌شده برای دسترسی به شیء جاری استفاده کرد.

برای مثال:

```
class Widget {
public:
    void update(this Widget& self) {
        self.value = 10;
    }

private:
    int value = 0;
};
```

نباید این تابع را مانند یک Member Function معمولی طراحی کرد که برای دسترسی به شیء از `this` استفاده می‌کند.

### فراموش کردن std::forward

اگر پارامتر از نوع `auto&&` باشد، استفاده مستقیم از آن در یک Expression معمولاً آن را به شکل Lvalue در اختیار قرار می‌دهد.

برای حفظ Value Category هنگام Forward کردن، باید از `std::forward` استفاده کرد.

برای مثال:

```
#include <utility>

class Widget {
public:
    void process(this auto&& self) {
        helper(std::forward<decltype(self)>(self));
    }

private:
    template <typename T>
    void helper(T&& value) {
    }
};
```

### تصور اینکه auto&& همیشه یک Rvalue Reference است

در Explicit Object Parameter نیز قواعد Forwarding Reference برقرار هستند.

بنابراین، `auto&&` بسته به نوع شیء می‌تواند به Lvalue Reference یا Rvalue Reference تبدیل شود.

### نادیده گرفتن Constness

اگر تابع به شکل زیر تعریف شود:

```
class Widget {
public:
    void modify(this auto&& self) {
        self.value = 10;
    }

private:
    int value = 0;
};
```

این تابع برای یک شیء `const Widget` قابل استفاده نخواهد بود، زیرا در آن حالت `self` به یک Const Reference تبدیل می‌شود و تغییر اعضای غیر `mutable` مجاز نیست.

### اشتباه گرفتن Explicit Object Parameter با Parameterهای معمولی

پارامتر Explicit Object از نظر جایگاه و قواعد زبان با پارامترهای عادی تفاوت دارد.

این پارامتر باید اولین پارامتر تابع باشد و نقش دریافت شیء جاری را بر عهده دارد.

بنابراین، نمی‌توان آن را در هر موقعیتی از Parameter List قرار داد.

---

## مقایسه روش‌های مختلف پیاده‌سازی

### مقایسه Getter سنتی و Deducing this

برای درک بهتر مزایای این ویژگی، دو پیاده‌سازی زیر را مقایسه کنید.

روش سنتی:

```
class Widget {
public:
    Widget& get() & {
        return *this;
    }

    const Widget& get() const & {
        return *this;
    }

    Widget&& get() && {
        return std::move(*this);
    }

    const Widget&& get() const && {
        return std::move(*this);
    }
};
```

روش مبتنی بر Deducing `this`:

```
#include <utility>

class Widget {
public:
    auto&& get(this auto&& self) {
        return std::forward<decltype(self)>(self);
    }
};
```

هر دو روش می‌توانند رفتار مشابهی ارائه دهند.

با این حال، روش دوم از نظر تعداد خطوط کد ساده‌تر است و نگهداری آن در کلاس‌های بزرگ‌تر آسان‌تر خواهد بود.

روش سنتی نیز همچنان برای APIهایی که Overloadهای صریح و محدودیت‌های مشخص دارند، کاملاً مناسب است.

### مقایسه Recursive Lambda و std::function

روش سنتی:

```
#include <functional>

std::function<int(int)> factorial;

factorial = [&](int n) {
    if (n <= 1) {
        return 1;
    }

    return n * factorial(n - 1);
};
```

روش جدید:

```
auto factorial = [](this auto&& self, int n) -> int {
    if (n <= 1) {
        return 1;
    }

    return n * self(n - 1);
};
```

روش جدید به `std::function` نیاز ندارد و Type آن در زمان Compile مشخص است.

با این حال، `std::function` همچنان در شرایطی که به Type Erasure یا ذخیره Callbackهای دارای انواع مختلف نیاز داریم، کاربرد دارد.

بنابراین، Recursive Lambda مبتنی بر Deducing `this` جایگزین تمام کاربردهای `std::function` نیست.

---

## پشتیبانی کامپایلرها و استاندارد

### وضعیت استانداردسازی

قابلیت Deducing `this` با پیشنهاد `P0847R7` به استاندارد ++C23 اضافه شد.

این قابلیت بخشی از Core Language است و برای استفاده از آن نیازی به Header خاصی از Standard Library وجود ندارد.

البته ممکن است برای مثال‌های مرتبط با این قابلیت به Headerهایی مانند موارد زیر نیاز داشته باشیم:

- `<utility>` برای `std::forward` و `std::move`.
- `<type_traits>` برای Type Traits.
- `<concepts>` برای استفاده از Concepts.
- `<functional>` برای `std::function`.
- `<iostream>` برای ورودی و خروجی.

### بررسی پشتیبانی کامپایلرها

پشتیبانی از ++C23 در کامپایلرهای مختلف ممکن است متفاوت باشد.

برای استفاده از این ویژگی باید نسخه‌ای از Compiler انتخاب شود که Explicit Object Parameter را پیاده‌سازی کرده باشد.

نمونه دستورات کامپایل:

```
g++ -std=c++23 main.cpp -o app
```

```
clang++ -std=c++23 main.cpp -o app
```

```
cl /std:c++latest /EHsc main.cpp
```

در Visual C++، گزینه `c++latest` برای فعال کردن جدیدترین قابلیت‌های استاندارد پشتیبانی‌شده استفاده می‌شود.

نکته مهم این است که فعال کردن حالت ++C23 یا `c++latest` به‌تنهایی تضمین نمی‌کند که تمام قابلیت‌های استاندارد در نسخه نصب‌شده Compiler پیاده‌سازی شده باشند.

برای بررسی وضعیت دقیق پشتیبانی باید مستندات نسخه همان Compiler را بررسی کرد.

همچنین تفاوت میان پشتیبانی زبان و پشتیبانی IDE را در نظر بگیرید؛ ممکن است Compiler یک قابلیت را پشتیبانی کند، اما Code Analysis یا Syntax Highlighting در IDE هنوز به‌روزرسانی نشده باشد.

---

## کاربردهای عملی در پروژه‌های حرفه‌ای

### طراحی Libraryهای عمومی

در توسعه Libraryهایی که قرار است توسط کاربران مختلف استفاده شوند، کاهش تعداد Overloadها می‌تواند نگهداری و توسعه API را آسان‌تر کند.

برای مثال، یک Container یا Wrapper می‌تواند با استفاده از Deducing `this` عملیات مشترکی را بدون تعریف نسخه‌های متعدد پیاده‌سازی کند.

### طراحی Fluent API و Builder Pattern

در Builder Pattern، حفظ نوع و Value Category شیء در توابع زنجیره‌ای اهمیت زیادی دارد.

با استفاده از Deducing `this` می‌توان تعداد Overloadهای لازم برای این هدف را کاهش داد.

### توسعه Generic Programming

در طراحی Generic APIها، نوع دقیق شیء و Qualifierهای آن ممکن است در انتخاب رفتار مناسب نقش داشته باشند.

Deducing `this` امکان دسترسی مستقیم به این اطلاعات را فراهم می‌کند.

### پیاده‌سازی Algorithmهای بازگشتی

در الگوریتم‌هایی مانند موارد زیر، Recursive Lambda مبتنی بر Deducing `this` می‌تواند طراحی ساده‌تری ارائه دهد:

- پیمایش درخت‌ها.
- جست‌وجوی Depth-First Search.
- جست‌وجوی Graph.
- محاسبات بازگشتی.
- پردازش ساختارهای Nested.
- ارزیابی Expression Treeها.

### پیاده‌سازی Static Polymorphism

در برخی طراحی‌ها می‌توان از Deducing `this` برای حذف نیاز به CRTP استفاده کرد.

این کار می‌تواند پیچیدگی ساختار کلاس‌های پایه را کاهش دهد، به‌خصوص زمانی که تنها هدف دسترسی به API کلاس مشتق‌شده از طریق Member Functionها باشد.

---

## جمع‌بندی

قابلیت **Deducing `this`** در ++C23 یکی از تغییرات مهم در نحوه طراحی Member Functionها است که امکان کنترل مستقیم‌تر Object Parameter را فراهم می‌کند.

این ویژگی با معرفی Explicit Object Parameter، محدودیت مدل سنتی Implicit Object Parameter را کاهش می‌دهد و به برنامه‌نویس اجازه می‌دهد نوع شیء جاری را به‌صورت صریح یا از طریق Template Deduction دریافت کند.

مهم‌ترین مزایای این قابلیت عبارت‌اند از:

- کاهش Overloadهای تکراری برای `const` و Non-const Member Functionها.
- مدیریت ساده‌تر Lvalue و Rvalue.
- حفظ Reference Category با استفاده از `std::forward`.
- ساده‌سازی Fluent Interface و Builder Pattern.
- کاهش پیچیدگی برخی پیاده‌سازی‌های CRTP.
- امکان نوشتن Recursive Lambda بدون نیاز به `std::function`.
- افزایش انعطاف‌پذیری Generic Programming.
- امکان استفاده از Concepts و Type Traits برای محدود کردن رفتار Generic Member Functionها.

با این وجود، این ویژگی محدودیت‌هایی نیز دارد.

Explicit Object Member Function نمی‌تواند Virtual یا Static باشد و استفاده از آن در Constructorها، Destructorها و برخی دیگر از Functionهای خاص مجاز نیست.

همچنین استفاده نادرست از `auto&&`، `std::forward` و Referenceهای بازگشتی می‌تواند مشکلاتی مانند پیچیدگی غیرضروری API یا Dangling Reference ایجاد کند.

در نهایت، Deducing `this` ابزاری قدرتمند برای ساده‌سازی طراحی‌های Generic و کاهش تکرار کد است؛ اما استفاده از آن باید بر اساس نیاز واقعی API انجام شود.

برای Member Functionهای ساده، روش سنتی همچنان انتخاب مناسبی است. در مقابل، هر زمان که نیاز به حفظ دقیق نوع، Constness و Value Category شیء وجود داشته باشد، Deducing `this` می‌تواند راه‌حلی خواناتر، انعطاف‌پذیرتر و مدرن‌تر ارائه دهد.

عنوان و مخاطب هدف را مشخص کنمثال‌های تکراری را ادغام کننکات فنی حساس را دقیق‌تر کن

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>