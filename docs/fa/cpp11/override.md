<div align="right">

[🇺🇸 English](../../en/cpp11/override.md) | [🇮🇷 فارسی](./override.md)

</div>
---

# دستور `Override` در `C++11` به بعد

فهرست مطالب

- [مفهوم کلی](#مفهوم-کلی)
- [چیستی دستور `override`](#چیستی-دستور-override)
- [جایی که `override` واقعا نجاتمان می‌دهد](#جایی-که-override-واقعا-نجاتمان-می-دهد)
- [مکانیسم `Override` در مقابل `Overload`](#مکانیسم-override-در-مقابل-overload)
- [ماهیت `Override`](#ماهیت-override)
- [مقایسه خیلی مهم](#مقایسه-خیلی-مهم)
- [ساختار `Overload`](#ساختار-overload)
- [ساختار `Override`](#ساختار-override)
- [یک مثال که هر دو را نشان می‌دهد](#یک-مثال-که-هر-دو-را-نشان-می-دهد)
- [یک اشتباه خیلی معروف](#یک-اشتباه-خیلی-معروف)
- [حتی `const` هم مهم است](#حتی-const-هم-مهم-است)
- [کاری که `Override` دقیقا انجام می‌دهد](#کاری-که-override-دقیقا-انجام-می-دهد)
- [یک جمله خیلی کوتاه برای ذهن](#یک-جمله-خیلی-کوتاه-برای-ذهن)
- [مشارکت ها](#مشارکت-ها)

## مفهوم کلی

من عمداً دارم یک `virtual function` از کلاس پایه را `override` می‌کنم؛ پس اگر چنین تابعی با این `signature` در `base` وجود ندارد، خطا بده.

## چیستی دستور `override`

فرض کنید یک کلاس پدر داریم مثل کلاس زیر:

```cpp
class Animal {
public:
    virtual void speak() {
        std::cout << "Animal speaks\n";
    }
};
```

و کلاس فرزند:

```cpp
class Dog : public Animal {
public:
    void speak() override {
        std::cout << "Dog barks\n";
    }
};
```

اینجا:

```cpp
void speak() override
```

یعنی:

من انتظار دارم `speak` دقیقاً یک `virtual function` در کلاس پایه باشد و این تابع نسخه‌ی آن را در کلاس فرزند `override` می‌کند.

اگر `override` نگذاریم چه؟ می‌توانستیم بنویسیم:

```cpp
class Dog : public Animal {
public:
    void speak() {
        std::cout << "Dog barks\n";
    }
};
```

این کد هم کاملاً معتبر است. چون `speak()` در `Animal` از نوع `virtual` بوده، تابع `Dog::speak()` هم آن را `override` می‌کند. پس این:

```cpp
void speak()
```

و این:

```cpp
void speak() override
```

از نظر رفتار `runtime` تفاوتی ندارند.

تفاوت اصلی این است که `override` به کامپایلر اجازه می‌دهد اشتباه ما را پیدا کند.

## جایی که `override` واقعا نجاتمان می‌دهد

فرض کنید به اشتباه بنویسیم:

```cpp
class Animal {
public:
    virtual void speak() {
        std::cout << "Animal speaks\n";
    }
};

class Dog : public Animal {
public:
    void Speak() {
        std::cout << "Dog barks\n";
    }
};
```

دقت کنید:

```cpp
speak()   // شروع با حروف کوچک
Speak()   // شروع با حروف بزرگ
```

این دو با هم فرق می‌کنند!  
`C++` به حروف بزرگ و کوچک حساس است.

بدون `override`، کامپایلر ممکن است بگوید:

مشکلی نیست؛ `Dog` یک تابع جدید به نام `Speak` دارد.

یعنی عملاً `overload` هم نیست؛ اینجا یک تابع جدید و `unrelated` ساخته‌ایم.

اما اگر بنویسیم:

```cpp
class Dog : public Animal {
public:
    void Speak() override {
        std::cout << "Dog barks\n";
    }
};
```

کامپایلر می‌گوید:

```text
error: function marked 'override' but does not override
```

## مکانیسم `Override` در مقابل `Overload`

`Override` یعنی:

کلاس فرزند، `implementation` یک `virtual function` از کلاس پدر را جایگزین می‌کند.

مثلاً:

```cpp
class Animal {
public:
    virtual void speak() {
        std::cout << "Animal\n";
    }
};

class Dog : public Animal {
public:
    void speak() override {
        std::cout << "Dog\n";
    }
};
```

هر دوی این کلاس‌ها، تابع زیر را با امضای یکسان دارند!

```cpp
speak()
```

پس:

```text
Animal
   │
   │ virtual speak()
   ▼
Dog
   │
   └── override speak()
```

## ماهیت `Override`

در `Overload` ما چند تابع با یک نام یکسان ولی `signature` متفاوت داریم.

مثلاً:

```cpp
class Calculator {
public:
    void add(int a, int b) {
        std::cout << a + b << '\n';
    }

    void add(double a, double b) {
        std::cout << a + b << '\n';
    }

    void add(int a, int b, int c) {
        std::cout << a + b + c << '\n';
    }
};
```

اینجا:

```cpp
add(int, int)
add(double, double)
add(int, int, int)
```

همه اسمشان `add` است، ولی `signature` متفاوت دارند.

این می‌شود:

`Function Overloading`

و هیچ ارتباطی با `inheritance` یا `virtual` ندارد.

## مقایسه خیلی مهم

### ساختار `Overload`

```cpp
void foo(int);
void foo(double);
```

یعنی:

```text
یک کلاس
   │
   ├── foo(int)
   └── foo(double)
```

هدف:

چند نسخه از یک `function` با پارامترهای ورودی مختلف.

### ساختار `Override`

```cpp
class Parent {
public:
    virtual void foo(int);
};

class Child : public Parent {
public:
    void foo(int) override;
};
```

یعنی:

```text
Parent
   │
   └── virtual foo(int)
             ▲
             │ override
             │
Child ───────┘
```

هدف:

تغییر `implementation` تابع `virtual` پدر در کلاس فرزند.

## یک مثال که هر دو را نشان می‌دهد

اینجا قضیه خیلی جالب می‌شود:

```cpp
class Animal {
public:
    virtual void speak(int volume) {
        std::cout << "Animal: " << volume << '\n';
    }
};

class Dog : public Animal {
public:
    // Override
    void speak(int volume) override {
        std::cout << "Dog: " << volume << '\n';
    }

    // Overload
    void speak(const std::string& message) {
        std::cout << "Dog says: " << message << '\n';
    }
};
```

اینجا `Dog` دو تابع دارد:

```cpp
speak(int)
speak(string)
```

این دو تا نسبت به هم `overload` هستند.

اما:

```cpp
speak(int)
```

نسبت به `Animal::speak(int)` از نوع `override` است.

پس یک تابع می‌تواند همزمان در یک `context` بخشی از یک `overload set` باشد و در رابطه با `base class`، `override` هم محسوب شود.

## یک اشتباه خیلی معروف

در این کد:

```cpp
class Animal {
public:
    virtual void speak(int x) {
    }
};

class Dog : public Animal {
public:
    void speak(double x) {
    }
};
```

ممکن است برنامه‌نویس فکر کند:

«خب من `speak` پدر را تغییر دادم.»

اما این اتفاق نیفتاده!

چون:

```cpp
speak(int)
```

با:

```cpp
speak(double)
```

یکی نیست.

پس اینجا `override` نداریم.

در واقع `Dog` یک تابع جدید دارد.

اگر بنویسیم:

```cpp
void speak(double x) override {}
```

کامپایلر فوراً اشتباه را پیدا می‌کند.

## حتی `const` هم مهم است

مثلاً:

```cpp
class Animal {
public:
    virtual void speak() const {
    }
};
```

اگر در فرزند بنویسیم:

```cpp
class Dog : public Animal {
public:
    void speak() {
    }
};
```

این `override` نیست.

چون:

```cpp
speak() const
```

با:

```cpp
speak()
```

`signature` موردنیاز برای `override` یکسان نیست.

ولی:

```cpp
class Dog : public Animal {
public:
    void speak() const override {
    }
};
```

درست است.

و این یکی از بهترین دلایل استفاده از `override` است:

لازم نیست خودتان این جزئیات را همیشه با چشم چک کنید؛ کامپایلر چک می‌کند.

## کاری که `Override` دقیقا انجام می‌دهد

این جمله را در ذهن هک کنید:

`override` به کامپایلر می‌گوید که من عمداً قصد دارم یک `virtual function` از کلاس پایه را `override` کنم؛ بنابراین اگر تابع من واقعاً یک تابع `virtual` قابل `override` از `base` نباشد، برنامه را با خطای `compile` متوقف کن.

و تفاوت اصلی:

```text
OVERLOAD
────────
Same name
Different parameters
Usually same class
No inheritance required
No virtual required
```

در مقابل:

```text
OVERRIDE
────────
Same function signature
Base → Derived
Requires a virtual function in the base
Used for runtime polymorphism
override keyword helps the compiler catch mistakes
```

## یک جمله خیلی کوتاه برای ذهن

`Overload = same name, different signature.`  
`Override = same virtual function, new implementation in the derived class.`

و `override` خودش کار `polymorphism` را انجام نمی‌دهد؛ `virtual` در `base` است که `virtual dispatch` را فراهم می‌کند. `override` بیشتر نقش `compile-time safety check` را دارد.

همان‌طور که وقتی در گیت با نوشتن دستور زیر:

```bash
git push origin devs
```

دو کلمه آخر برای اطمینان از این است که اگر بر روی شاخه‌ای جز `devs` باشیم، خطا دهد، `Override` هم حالت مشابهی است که اگر هنگام `override` کردن و بازنویسی تابع پدری اشتباهی داشته باشیم، کامپایلر به ما خطای مناسب دهد!


---
## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>