<div align="center">

[🇺🇸 English](../../en/cpp23/multi_dimensional_subscript_operator.md) | [🇮🇷 فارسی](./multi_dimensional_subscript_operator.md)

</div>

---

در ادامه، ترجمه کامل داکیومنت را با حفظ ساختار اصلی، عنوان‌ها، ترتیب بخش‌ها، تمام قطعه‌کدها، نام کلاس‌ها، توابع، شناسه‌های استاندارد C++، فرمول‌ها، جداول و لینک‌های مرجع ارائه می‌کنم.

ترجمه با رویکرد یک مستند آموزشی و فنی برای توسعه‌دهندگان C++ انجام شده است؛ بنابراین اصطلاحات تخصصی مهم مانند `Multidimensional Subscript Operator`، `Overload Resolution`، `Proxy Object` و `Explicit Object Parameter` در کنار معادل فارسی آن‌ها حفظ شده‌اند تا دقت فنی متن و قابلیت جست‌وجوی اصطلاحات استاندارد از بین نرود.

# **عملگر اندیس‌گذاری چندبعدی در C++23 (Multidimensional Subscript Operator)**

## **مقدمه (Introduction)**

عملگر اندیس‌گذاری (`operator[]`) یکی از پرکاربردترین عملگرها در زبان C++ است که برای دسترسی به عناصر آرایه‌ها، کانتینرها (Containers) و ساختارهای داده مشابه مورد استفاده قرار می‌گیرد.

پیش از C++23، یک `operator[]` بازنویسی‌شده (Overloaded) نمی‌توانست بیش از یک پارامتر دریافت کند. در نتیجه، توسعه‌دهندگان برای پیاده‌سازی ساختارهای داده چندبعدی مجبور بودند از روش‌های جایگزین استفاده کنند؛ روش‌هایی مانند زنجیره‌کردن چندین عملگر اندیس‌گذاری، بازنویسی عملگر فراخوانی تابع (`operator()`) یا استفاده از تکنیک‌هایی مبتنی بر `operator,`.

در C++23 این محدودیت برطرف شده و قابلیت **عملگر اندیس‌گذاری چندبعدی (Multidimensional Subscript Operator)** معرفی شده است.

بر اساس پیشنهاد استانداردسازی `P2128R6`، این قابلیت به `operator[]` اجازه می‌دهد مستقیماً چندین پارامتر دریافت کند.

برای مثال، از C++23 به بعد تعریف زیر معتبر است:

```
class Matrix {
public:
    double& operator[](std::size_t row, std::size_t column);
};
```

اکنون می‌توان به عناصر ماتریس به شکل زیر دسترسی داشت:

```
Matrix matrix;

matrix[2, 3] = 10.5;
```

این قابلیت صرفاً یک بهبود نحوی (Syntactic Improvement) نیست؛ بلکه امکان طراحی ساختارهای داده چندبعدی ساده‌تر، خواناتر و انعطاف‌پذیرتر را فراهم می‌کند.

در این مستند بررسی می‌کنیم که این قابلیت چگونه کار می‌کند، چه تفاوتی با روش‌های سنتی دارد، چه ارتباطی با `std::mdspan` پیدا می‌کند، قوانین انتخاب سربار مناسب (Overload Resolution) چگونه هستند، چه کاربردهای عملی دارد و چه محدودیت‌ها و ملاحظات طراحی مهمی باید در نظر گرفته شوند.

## **آشنایی با عملگر اندیس‌گذاری پیش از C++23 (Understanding the Subscript Operator Before C++23)**

برای درک بهتر قابلیت جدید، ابتدا بهتر است نحوه عملکرد عملگر اندیس‌گذاری را در استانداردهای قدیمی‌تر C++ بررسی کنیم.

### **محدودیت در تعداد پارامترها (Parameter Count Restrictions)**

پیش از C++23، یک عملگر اندیس‌گذاری بازنویسی‌شده در یک کلاس فقط می‌توانست یک پارامتر صریح دریافت کند.

برای مثال، پیاده‌سازی زیر معتبر بود:

```
#include <cstddef>
#include <vector>

class IntArray {
private:
    std::vector<int> data;

public:
    explicit IntArray(std::size_t size)
        : data(size) {}

    int& operator[](std::size_t index) {
        return data[index];
    }

    const int& operator[](std::size_t index) const {
        return data[index];
    }
};
```

می‌توان از این کلاس به شکل زیر استفاده کرد:

```
IntArray array(10);

array[3] = 42;

int value = array[3];
```

اما تعریف عملگری با چندین پارامتر پیش از C++23 مجاز نبود:

```
class Matrix {
public:
    int& operator[](std::size_t row, std::size_t column);
};
```

علت این موضوع آن بود که `operator[]` به دریافت تنها یک پارامتر صریح محدود شده بود.

### **پیاده‌سازی آرایه‌های چندبعدی با استفاده از زنجیره‌کردن عملگرهای اندیس‌گذاری (Implementing Multidimensional Arrays Through Chained Subscript Operators)**

یکی از روش‌های متداول برای پیاده‌سازی آرایه‌های چندبعدی، زنجیره‌کردن چندین فراخوانی `operator[]` بود.

برای مثال، دسترسی به یک عنصر در آرایه‌ای سه‌بعدی می‌توانست به شکل زیر باشد:

```
array[x][y][z]
```

در این روش، هر `operator[]` باید آرایه یا شیئی را برگرداند که خودش از `operator[]` پشتیبانی کند.

مثال زیر یک پیاده‌سازی ساده از این روش را نشان می‌دهد:

```
#include <cstddef>
#include <vector>

class Row {
private:
    std::vector<int> data;

public:
    explicit Row(std::size_t columns)
        : data(columns) {}

    int& operator[](std::size_t column) {
        return data[column];
    }
};

class Matrix {
private:
    std::vector<Row> rows;

public:
    Matrix(std::size_t rowCount, std::size_t columnCount) {
        rows.reserve(rowCount);

        for (std::size_t i = 0; i < rowCount; ++i) {
            rows.emplace_back(columnCount);
        }
    }

    Row& operator[](std::size_t row) {
        return rows[row];
    }
};
```

نحوه استفاده:

```
Matrix matrix(3, 4);

matrix[1][2] = 100;

int value = matrix[1][2];
```

این روش همچنان معتبر است و در برخی طراحی‌ها می‌تواند مفید باشد. بااین‌حال، ممکن است چالش‌هایی ایجاد کند:

- افزایش پیچیدگی در طراحی کلاس‌های چندبعدی.
- نیاز به تعریف انواع واسط (Intermediate Types) برای هر بُعد.
- احتمال انجام چندین تخصیص حافظه (Memory Allocation).
- دشوارتر شدن مدیریت ساختارهای چندبعدی با تعداد ابعاد متغیر.
- پیچیده‌تر شدن طراحی کانتینرهای عمومی (Generic Containers).

توجه داشته باشید که تخصیص‌های جداگانه حافظه اجتناب‌ناپذیر نیستند؛ وقوع آن‌ها به نحوه پیاده‌سازی کانتینر بستگی دارد.

### **استفاده از عملگر فراخوانی تابع (Using the Function Call Operator)**

یکی دیگر از روش‌های پرکاربرد پیش از C++23، بازنویسی `operator()` بود.

برای مثال:

```
#include <cstddef>
#include <vector>

class Matrix {
private:
    std::size_t rows;
    std::size_t columns;
    std::vector<double> data;

public:
    Matrix(std::size_t r, std::size_t c)
        : rows(r),
          columns(c),
          data(r * c) {}

    double& operator()(std::size_t row, std::size_t column) {
        return data[row * columns + column];
    }

    const double& operator()(std::size_t row, std::size_t column) const {
        return data[row * columns + column];
    }
};
```

نحوه استفاده:

```
Matrix matrix(3, 4);

matrix(1, 2) = 25.5;

double value = matrix(1, 2);
```

این روش از نظر فنی کاملاً معتبر است و حتی پس از C++23 نیز می‌تواند انتخاب مناسبی باشد.

بااین‌حال، برای ساختارهای داده‌ای که از نظر معنایی رفتار طبیعی آن‌ها شبیه دسترسی به آرایه است، استفاده از `operator[]` می‌تواند نحوی شهودی‌تر ارائه دهد.

## **معرفی عملگر اندیس‌گذاری چندبعدی در C++23 (Introducing the Multidimensional Subscript Operator in C++23)**

در C++23، عملگر `operator[]` می‌تواند بیش از یک پارامتر صریح دریافت کند.

این یعنی اکنون می‌توان تابعی با امضای زیر تعریف کرد:

```
class Matrix {
public:
    double& operator[](std::size_t row, std::size_t column);
};
```

سپس می‌توان با استفاده از دستور زیر به عناصر دسترسی داشت:

```
matrix[1, 2]
```

برای شیئی از نوع `Matrix`، این عبارت عملگر اندیس‌گذاری بازنویسی‌شده را با دو آرگومان فراخوانی می‌کند.

از نظر مفهومی، این عبارت معادل فراخوانی تابع زیر است:

```
matrix.operator[](1, 2);
```

همچنین می‌توان تعداد پارامترهای بیشتری تعریف کرد:

```
class Tensor3D {
public:
    float& operator[](
        std::size_t x,
        std::size_t y,
        std::size_t z
    );
};
```

نحوه استفاده:

```
tensor[1, 2, 3] = 15.0f;
```

هیچ محدودیتی در زبان وجود ندارد که این قابلیت را فقط به دو یا سه بُعد محدود کند. تعداد پارامترها می‌تواند بیشتر باشد، مشروط بر اینکه اعلان تابع و عبارت متناظر با آن معتبر باشند.

### **یک تفاوت مهم با آرایه‌های داخلی زبان (An Important Difference from Built-in Arrays)**

نکته مهم این است که این قابلیت رفتار آرایه‌های داخلی زبان (Built-in Arrays) را تغییر نمی‌دهد و آن‌ها را به آرایه‌های چندبعدی با چندین اندیس مستقل تبدیل نمی‌کند.

برای مثال، کد زیر همچنان برای یک آرایه داخلی نامعتبر است:

```
int array[3][4];

array[1, 2] = 10;
```

آرایه‌های داخلی زبان از بازنویسی `operator[]` پشتیبانی نمی‌کنند.

برای دسترسی به عناصر آن‌ها همچنان باید از شکل سنتی زیر استفاده کرد:

```
array[1][2] = 10;
```

اما اگر شیء موردنظر نمونه‌ای از یک کلاس باشد که `operator[]` چندبعدی را تعریف کرده است، عبارت `object[i, j]` می‌تواند معتبر باشد.

## **نحوه عملکرد و قوانین زبان (How It Works and Language Rules)**

### **تفسیر عبارات چندبعدی (Interpreting Multidimensional Expressions)**

در C++23، عبارت زیر:

```
object[i, j]
```

درصورتی‌که سربار مناسبی برای آن انتخاب شود، عملگر بازنویسی‌شده را به شکل زیر فراخوانی می‌کند:

```
object.operator[](i, j);
```

به همین ترتیب:

```
object[i, j, k]
```

معادل عبارت زیر است:

```
object.operator[](i, j, k);
```

بنابراین، کامپایلر عبارات اندیس را به‌عنوان آرگومان‌های جداگانه به تابع ارسال می‌کند.

### **تعداد پارامترها (Number of Parameters)**

در C++23، عملگر `operator[]` می‌تواند هر تعداد پارامتر صریحی را که طراحی موردنظر نیاز دارد دریافت کند.

برای مثال:

```
class MultiArray {
public:
    int& operator[](int x, int y);
    int& operator[](int x, int y, int z);
    int& operator[](int x, int y, int z, int w);
};
```

این سربارها می‌توانند هم‌زمان در یک کلاس وجود داشته باشند.

کامپایلر با استفاده از قوانین معمول انتخاب سربار (Overload Resolution)، تابع مناسب را انتخاب می‌کند.

### **نوع پارامترها (Parameter Types)**

پارامترهای `operator[]` لزوماً نباید از نوع عدد صحیح باشند.

مانند سایر توابع عضو، این عملگر نیز می‌تواند انواع مختلفی از پارامترها را دریافت کند:

```
#include <string>
#include <string_view>

class DataStore {
public:
    int& operator[](std::string_view category, std::size_t index);
};
```

در این مثال، پارامتر اول نمایانگر یک دسته‌بندی (Category) و پارامتر دوم نشان‌دهنده یک اندیس است.

همچنین می‌توان از انواع سفارشی (Custom Types) استفاده کرد:

```
struct Coordinate {
    int x;
    int y;
};

class Grid {
public:
    int& operator[](Coordinate position);
};
```

نکته مهم این است که اندیس‌گذاری چندبعدی به اندیس‌های عددی محدود نمی‌شود. این قابلیت امکان ارسال چندین آرگومان با انواع متفاوت را به عملگر فراهم می‌کند.

### **پارامترهای صریح و پارامتر شیء (Explicit Parameters and the Object Parameter)**

در یک تابع عضو معمولی، خود شیء به‌صورت ضمنی با تابع مرتبط است.

برای مثال:

```
class Matrix {
public:
    int& operator[](int row, int column);
};
```

در این اعلان، `row` و `column` پارامترهای صریح هستند، درحالی‌که شیء `Matrix` از طریق پارامتر ضمنی شیء (Implicit Object Parameter) نمایش داده می‌شود.

بنابراین:

```
matrix[1, 2]
```

از نظر مفهومی شامل سه مؤلفه است:

- شیئی که ماتریس را نمایش می‌دهد.
- پارامتر اول با مقدار `1`.
- پارامتر دوم با مقدار `2`.

درک این تمایز هنگام بررسی انتخاب سربار و مشخصه‌های توابع عضو اهمیت دارد.

## **مثال‌های عملی (Practical Examples)**

### **پیاده‌سازی یک ماتریس دوبعدی (Implementing a Two-Dimensional Matrix)**

یکی از مهم‌ترین کاربردهای این قابلیت، پیاده‌سازی ماتریس است.

در مثال زیر، عناصر به‌صورت پیوسته در حافظه ذخیره شده‌اند و دسترسی به آن‌ها از طریق دو اندیس انجام می‌شود.

```
#include <cassert>
#include <cstddef>
#include <iostream>
#include <vector>

class Matrix {
private:
    std::size_t rows;
    std::size_t columns;
    std::vector<double> data;

public:
    Matrix(std::size_t r, std::size_t c)
        : rows(r),
          columns(c),
          data(r * c, 0.0) {}

    double& operator[](std::size_t row, std::size_t column) {
        assert(row < rows);
        assert(column < columns);

        return data[row * columns + column];
    }

    const double& operator[](
        std::size_t row,
        std::size_t column
    ) const {
        assert(row < rows);
        assert(column < columns);

        return data[row * columns + column];
    }

    std::size_t rowCount() const {
        return rows;
    }

    std::size_t columnCount() const {
        return columns;
    }
};

int main() {
    Matrix matrix(3, 4);

    matrix[0, 0] = 10.0;
    matrix[1, 2] = 25.5;
    matrix[2, 3] = 50.0;

    std::cout << matrix[0, 0] << '\n';
    std::cout << matrix[1, 2] << '\n';
    std::cout << matrix[2, 3] << '\n';
}
```

در این پیاده‌سازی، عناصر ماتریس در یک `std::vector` واحد ذخیره می‌شوند.

فرمول زیر مختصات دوبعدی را به یک اندیس یک‌بعدی تبدیل می‌کند:

[\
index = row \times columns + column\
]

این روش ذخیره‌سازی با نام **Row-major** شناخته می‌شود.

برای مثال، در ماتریسی با چهار ستون داریم:

[\
index(2,3) = 2 \times 4 + 3 = 11\
]

بنابراین، عنصر موردنظر در اندیس 11 از حافظه زیرین قرار دارد.

**مزایای این طراحی:**

- ذخیره‌سازی در حافظه پیوسته (Contiguous Memory).
- تعداد کمتر تخصیص‌های حافظه.
- دسترسی مستقیم به عناصر.
- سازگاری بهتر با الگوریتم‌های پردازش داده.
- استفاده مؤثرتر از حافظه کش پردازنده (CPU Cache) در الگوهای پیمایش مناسب.

### **پیاده‌سازی یک آرایه سه‌بعدی (Implementing a Three-Dimensional Array)**

این قابلیت برای ساختارهای داده سه‌بعدی نیز کاربرد دارد.

```
#include <cassert>
#include <cstddef>
#include <iostream>
#include <vector>

class Tensor3D {
private:
    std::size_t depth;
    std::size_t height;
    std::size_t width;

    std::vector<float> data;

public:
    Tensor3D(
        std::size_t d,
        std::size_t h,
        std::size_t w
    )
        : depth(d),
          height(h),
          width(w),
          data(d * h * w, 0.0f) {}

    float& operator[](
        std::size_t z,
        std::size_t y,
        std::size_t x
    ) {
        assert(z < depth);
        assert(y < height);
        assert(x < width);

        return data[z * height * width + y * width + x];
    }

    const float& operator[](
        std::size_t z,
        std::size_t y,
        std::size_t x
    ) const {
        assert(z < depth);
        assert(y < height);
        assert(x < width);

        return data[z * height * width + y * width + x];
    }
};

int main() {
    Tensor3D tensor(4, 3, 2);

    tensor[1, 2, 1] = 42.5f;

    std::cout << tensor[1, 2, 1] << '\n';
}
```

فرمول تبدیل مختصات سه‌بعدی به یک اندیس یک‌بعدی در این مثال به شکل زیر است:

[\
index = z \times (height \times width) + y \times width + x\
]

ترتیب ابعاد در این طراحی اهمیت دارد.

توسعه‌دهنده باید مشخص کند که کدام بُعد با سرعت بیشتری تغییر می‌کند و اطمینان حاصل کند که فرمول محاسبه اندیس با این تصمیم مطابقت دارد.

### **استفاده از پارامترهایی با انواع متفاوت (Using Different Parameter Types)**

همان‌طور که پیش‌تر اشاره شد، لازم نیست تمام پارامترها از یک نوع باشند.

برای مثال، می‌توان ساختار داده‌ای طراحی کرد که با استفاده از نام یک بخش و یک اندیس، به یک مقدار دسترسی پیدا کند.

```
#include <cstddef>
#include <string>
#include <string_view>
#include <unordered_map>
#include <vector>

class Database {
private:
    std::unordered_map<std::string, std::vector<int>> data;

public:
    int& operator[](
        std::string_view table,
        std::size_t index
    ) {
        return data.at(std::string(table)).at(index);
    }
};
```

نحوه استفاده:

```
Database database;

// Assume the "users" table has already been initialized.

int& value = database["users", 2];
```

در این مثال، دو پارامتر معنای متفاوتی دارند:

- پارامتر اول، یک بخش از داده را بر اساس نام آن مشخص می‌کند.
- پارامتر دوم، موقعیت عنصر موردنظر را تعیین می‌کند.

هدف از این پیاده‌سازی، نمایش انعطاف‌پذیری در انتخاب انواع پارامترها است. یک پیاده‌سازی واقعی از پایگاه داده، علاوه بر این موارد، به مدیریت داده‌ها، ایجاد ورودی‌ها و مدیریت خطاهای دسترسی نیز نیاز خواهد داشت.

## **مدیریت const و معناشناسی ارجاع (Managing const and Reference Semantics)**

یکی از مهم‌ترین جنبه‌های طراحی `operator[]`، مدیریت صحیح `const` و نوع مقدار بازگشتی است.

### **تعریف سربار غیر const (Defining a Non-const Overload)**

در یک کانتینر معمولی، نسخه غیر `const` باید امکان تغییر عناصر را فراهم کند.

```
class Matrix {
public:
    double& operator[](std::size_t row, std::size_t column);
};
```

بازگرداندن `double&` باعث می‌شود عبارت زیر معتبر باشد:

```
matrix[1, 2] = 100.0;
```

همچنین امکان دریافت آدرس یک عنصر را فراهم می‌کند:

```
double* pointer = &matrix[1, 2];
```

### **تعریف سربار const (Defining a const Overload)**

برای اشیای `const` نیز باید سربار مناسبی ارائه شود:

```
class Matrix {
public:
    double& operator[](std::size_t row, std::size_t column);

    const double& operator[](
        std::size_t row,
        std::size_t column
    ) const;
};
```

این کار امکان خواندن عناصر از ماتریس‌های `const` را فراهم می‌کند:

```
const Matrix& matrix = getMatrix();

double value = matrix[1, 2];
```

بازگرداندن `const double&` مانع از آن می‌شود که از طریق ارجاع بازگشتی، عنصر موردنظر مستقیماً تغییر داده شود.

### **استفاده از Deducing this برای جلوگیری از تکرار کد (Using Deducing this to Avoid Duplication)**

قابلیت Explicit Object Parameter که در C++23 و از طریق پیشنهاد `P0847R7` معرفی شده است، به توابع عضو اجازه می‌دهد پارامتر شیء خود را به‌صورت صریح دریافت کنند.

با استفاده از این قابلیت می‌توان از تکرار سربارهای `const` و غیر `const` جلوگیری کرد.

```
#include <cstddef>
#include <vector>

class Matrix {
private:
    std::size_t columns;
    std::vector<double> data;

public:
    Matrix(std::size_t rows, std::size_t cols)
        : columns(cols),
          data(rows * cols) {}

    template<typename Self>
    decltype(auto) operator[](
        this Self& self,
        std::size_t row,
        std::size_t column
    ) {
        return self.data[row * self.columns + column];
    }
};
```

در این پیاده‌سازی:

- اگر شیء غیر `const` باشد، نوع `Self` به‌صورت یک نوع غیر `const` استنباط می‌شود.
- اگر شیء `const` باشد، ویژگی `const` بودن آن حفظ می‌شود.
- نوع بازگشتی `decltype(auto)` نوع ارجاع مناسب را حفظ می‌کند.

در نتیجه، یک تابع واحد می‌تواند هر دو حالت را مدیریت کند.

این تکنیک هنگام طراحی APIهای عمومی مدرن مفید است. بااین‌حال، برای کلاس‌های ساده ممکن است تعریف دو سربار مجزا خوانایی بیشتری داشته باشد.

## **درک ارتباط با std::mdspan (Understanding the Relationship with std::mdspan)**

یکی از مهم‌ترین قابلیت‌های مرتبط با این موضوع، `std::mdspan` است.

این کلاس در کتابخانه استاندارد C++23 معرفی شده است.

`std::mdspan` یک نمای چندبعدی (Multidimensional View) روی حافظه است و مالک حافظه زیرین خود نیست.

به عبارت دیگر، این کلاس امکان دسترسی چندبعدی به داده‌هایی را فراهم می‌کند که توسط شیء دیگری ذخیره و مدیریت می‌شوند.

### **معرفی std::mdspan (Introducing std::mdspan)**

برای استفاده از این قابلیت، هدر زیر را اضافه کنید:

```
#include <mdspan>
```

یک مثال ساده:

```
#include <cstddef>
#include <iostream>
#include <mdspan>
#include <vector>

int main() {
    constexpr std::size_t rows = 3;
    constexpr std::size_t columns = 4;

    std::vector<int> data(rows * columns);

    std::mdspan<int, std::extents<
        std::size_t,
        rows,
        columns
    >> matrix(data.data());

    matrix[1, 2] = 42;

    std::cout << matrix[1, 2] << '\n';
}
```

در این مثال:

- `std::vector` مالک حافظه است.
- `std::mdspan` یک نمای چندبعدی روی آن حافظه ایجاد می‌کند.
- ابعاد ماتریس در نوع `std::mdspan` مشخص شده‌اند.
- دسترسی به عناصر با استفاده از چندین اندیس انجام می‌شود.

در واقع، `std::mdspan` یکی از مهم‌ترین کاربردهای عملی عملگر اندیس‌گذاری چندبعدی است.

### **درک مفهوم مالکیت و نماها (Understanding Ownership and Views)**

نکته مهم این است که `std::mdspan` مالک حافظه نیست.

بنابراین، طول عمر حافظه‌ای که به آن ارجاع می‌دهد باید بیشتر از طول عمر نما باشد.

برای مثال:

```
#include <mdspan>
#include <vector>

int main() {
    std::vector<double> storage(100);

    std::mdspan<double, std::extents<
        std::size_t,
        10,
        10
    >> matrix(storage.data());

    matrix[2, 3] = 12.5;
}
```

در این مثال، `storage` مدیریت حافظه را بر عهده دارد، درحالی‌که `matrix` صرفاً نمایی روی آن است.

اگر حافظه آزاد شود یا اشاره‌گر زیرین نامعتبر شود، استفاده از `mdspan` می‌تواند به رفتار تعریف‌نشده (Undefined Behavior) منجر شود.

### **استفاده از ابعاد پویا (Using Dynamic Extents)**

`std::mdspan` همچنین اجازه می‌دهد ابعاد در زمان اجرا مشخص شوند.

```
#include <cstddef>
#include <mdspan>
#include <vector>

int main() {
    std::size_t rows = 5;
    std::size_t columns = 8;

    std::vector<int> storage(rows * columns);

    using Extents = std::dextents<std::size_t, 2>;

    std::mdspan<int, Extents> matrix(
        storage.data(),
        rows,
        columns
    );

    matrix[2, 4] = 100;
}
```

در این مثال، تعداد ابعاد در زمان کامپایل مشخص است، اما اندازه هر بُعد در زمان اجرا تعیین می‌شود.

### **آشنایی با سیاست‌های چیدمان حافظه (Understanding Layout Policies)**

یکی از قابلیت‌های مهم `std::mdspan`، پشتیبانی از سیاست‌های مختلف چیدمان حافظه (Layout Policies) است.

این سیاست‌ها تعیین می‌کنند که مختصات چندبعدی چگونه به موقعیت‌های حافظه نگاشت شوند.

سه چیدمان اصلی عبارت‌اند از:

| چیدمان               | توضیحات                                                        |
| -------------------- | -------------------------------------------------------------- |
| `std::layout_right`  | بُعد آخر دارای Stride برابر یک است؛ معمولاً معادل Row-major    |
| `std::layout_left`   | بُعد اول دارای Stride برابر یک است؛ معمولاً معادل Column-major |
| `std::layout_stride` | مقادیر Stride به‌صورت صریح مشخص می‌شوند                        |

برای مثال:

```
#include <cstddef>
#include <mdspan>
#include <vector>

int main() {
    constexpr std::size_t rows = 3;
    constexpr std::size_t columns = 4;

    std::vector<int> storage(rows * columns);

    using Extents = std::extents<
        std::size_t,
        rows,
        columns
    >;

    std::mdspan<
        int,
        Extents,
        std::layout_left
    > matrix(storage.data());

    matrix[1, 2] = 50;
}
```

در این مثال، چیدمان حافظه از نوع `std::layout_left` است.

این موضوع نشان می‌دهد که استفاده از نحو دسترسی چندبعدی الزاماً به معنای ذخیره‌سازی به روش Row-major نیست.

### **ارتباط mdspan با عملگر اندیس‌گذاری چندبعدی (The Relationship Between mdspan and the Multidimensional Subscript Operator)**

باید بین دو مفهوم مجزا تمایز قائل شد:

- عملگر اندیس‌گذاری چندبعدی یک قابلیت در زبان C++23 است.
- `std::mdspan` یک مؤلفه از کتابخانه استاندارد C++23 است.

قابلیت زبانی به توسعه‌دهندگان اجازه می‌دهد `operator[]` را با چندین پارامتر تعریف کنند.

در مقابل، `std::mdspan` از این قابلیت استفاده می‌کند تا یک API استاندارد برای دسترسی چندبعدی به حافظه فراهم کند.

بنابراین، استفاده از `std::mdspan` در بسیاری از موارد نیاز به پیاده‌سازی یک کانتینر یا نمای چندبعدی سفارشی از ابتدا را برطرف می‌کند.

## **درک تغییرات رفتار عملگر کاما (Understanding the Changes to Comma Operator Behavior)**

یکی از جنبه‌های مهم و نسبتاً ظریف C++23، تغییر نحوه تفسیر کاما در عبارات اندیس‌گذاری است.

این تغییر ارتباط مستقیمی با عملگر اندیس‌گذاری چندبعدی دارد.

### **رفتار در استانداردهای قدیمی‌تر (Behavior in Older Standards)**

پیش از C++23، عبارت زیر:

```
array[i, j]
```

برای یک آرایه داخلی یا اشاره‌گر، به‌عنوان یک عملیات اندیس‌گذاری همراه با یک عبارت کاما (Comma Expression) تفسیر می‌شد.

به عبارت دیگر، ابتدا عبارت `i, j` ارزیابی می‌شد و نتیجه آن به‌عنوان اندیس مورد استفاده قرار می‌گرفت.

ازآنجاکه عملگر کاما هر دو عملوند را ارزیابی می‌کند و مقدار عملوند سمت راست را به‌عنوان نتیجه برمی‌گرداند، عبارت زیر:

```
array[i, j]
```

از نظر معنایی مشابه عبارت زیر بود:

```
array[j]
```

بااین‌حال، در C++20 استفاده از یک عبارت کامای بدون پرانتز در این موقعیت Deprecate شده بود.

### **رفتار در C++23 (Behavior in C++23)**

در C++23، نحو داخل یک عبارت اندیس‌گذاری می‌تواند شامل فهرستی از عبارات چندبعدی باشد.

بنابراین:

```
object[i, j]
```

درصورتی‌که شیء از نوع کلاس باشد و سربار مناسبی برای آن تعریف شده باشد، می‌تواند هر دو مقدار را به‌عنوان آرگومان‌های جداگانه به `operator[]` ارسال کند.

در نتیجه، این عبارت دیگر لزوماً به‌عنوان یک عبارت کاما تفسیر نمی‌شود.

### **حفظ رفتار عملگر کاما با استفاده از پرانتز (Preserving the Comma Operator with Parentheses)**

اگر هدف استفاده از عملگر کاما باشد، عبارت باید داخل پرانتز قرار بگیرد.

```
int array[10];

int i = 2;
int j = 5;

int value = array[(i, j)];
```

در این مثال، ابتدا `i` و سپس `j` ارزیابی می‌شود و مقدار `j` نتیجه عبارت کاما خواهد بود.

در نتیجه، عنصر پنجم آرایه مورد دسترسی قرار می‌گیرد.

درک این تفاوت برای حفظ سازگاری با کدهای قدیمی و جلوگیری از رفتارهای ناخواسته اهمیت دارد.

### **مقایسه دو عبارت مهم (Comparing Two Important Expressions)**

| عبارت            | معنا                                                                            |
| ---------------- | ------------------------------------------------------------------------------- |
| `object[i, j]`   | در صورت وجود سربار چندبعدی مناسب، دو آرگومان را ارسال می‌کند.                   |
| `object[(i, j)]` | نتیجه یک عبارت کاما را به‌عنوان یک آرگومان واحد ارسال می‌کند.                   |
| `array[i, j]`    | در C++23 برای یک آرایه داخلی نامعتبر است.                                       |
| `array[(i, j)]`  | با استفاده از اندیسی که از عبارت کاما به دست آمده، به آرایه دسترسی پیدا می‌کند. |

## **درک نحوه انتخاب سربار (Understanding Overload Resolution)**

مانند سایر عملگرهای بازنویسی‌شده، انتخاب `operator[]` چندبعدی مناسب نیز بر اساس قوانین معمول انتخاب سربار انجام می‌شود.

### **تعریف چندین سربار (Defining Multiple Overloads)**

می‌توان چندین تابع با تعداد یا انواع پارامترهای متفاوت تعریف کرد:

```
#include <cstddef>

class Matrix {
public:
    int& operator[](std::size_t index);

    int& operator[](
        std::size_t row,
        std::size_t column
    );
};
```

نحوه استفاده:

```
Matrix matrix;

matrix[5];

matrix[2, 3];
```

در عبارت اول، سربار دارای یک پارامتر انتخاب می‌شود.

در عبارت دوم، سربار دارای دو پارامتر انتخاب خواهد شد.

### **استفاده از انواع متفاوت پارامترها (Using Different Parameter Types)**

بازنویسی عملگر فقط به تعداد پارامترها محدود نمی‌شود.

```
#include <string_view>

class Data {
public:
    int& operator[](int index);

    int& operator[](std::string_view name, int index);
};
```

در این مثال، انواع پارامترها نیز بر انتخاب سربار تأثیر می‌گذارند.

### **استفاده از قالب‌ها (Using Templates)**

همچنین می‌توان `operator[]` را به‌صورت یک تابع قالبی (Function Template) تعریف کرد.

```
#include <concepts>
#include <cstddef>

class Matrix {
public:
    template<std::integral Row, std::integral Column>
    int& operator[](Row row, Column column);
};
```

در این مثال، `Row` و `Column` می‌توانند از انواع عدد صحیح متفاوتی باشند.

استفاده از `std::integral` باعث می‌شود الزامات نوعی پارامترها صریح‌تر باشند.

بااین‌حال، یک پیاده‌سازی واقعی باید تبدیل انواع، محدوده مقادیر و احتمال از دست رفتن اطلاعات را نیز در نظر بگیرد.

## **آشنایی با static operator[] (Understanding Static operator[])**

در C++23 امکان تعریف برخی از عملگرهای عضو به‌صورت `static` نیز فراهم شده است.

این قابلیت با پیشنهاد `P2589R1` مرتبط است و عملگرهایی مانند `operator()` و `operator[]` را شامل می‌شود.

برای مثال:

```
#include <iostream>

struct Utility {
    static int operator[](int lhs, int rhs) {
        return lhs + rhs;
    }
};

int main() {
    Utility utility;

    std::cout << utility[10, 20] << '\n';
}
```

در این مثال، `operator[]` یک تابع عضو ایستا (Static Member Function) است.

نکته مهم این است که با وجود `static` بودن، عبارت عملگر همچنان به یک شیء نیاز دارد تا عملگر روی آن فراخوانی شود.

بنابراین، نمی‌توان آن را مستقیماً به شکل زیر نوشت:

```
Utility[10, 20];
```

نام کلاس به‌تنهایی یک شیء نیست که بتوان از آن در این عبارت استفاده کرد.

همچنین `static` بودن `operator[]` به این معناست که تابع به `this` دسترسی ندارد و نمی‌تواند مستقیماً به وضعیت غیر ایستای (Non-static State) متعلق به یک نمونه از کلاس دسترسی پیدا کند.

### **موارد استفاده از static operator[] (Use Cases for Static operator[])**

این قابلیت می‌تواند هنگام طراحی اشیای کاربردی (Utility Objects)، عملیات بدون وضعیت (Stateless Operations) و APIهایی که به وضعیت داخلی یک نمونه نیاز ندارند، مفید باشد.

بااین‌حال، برای کانتینرهای معمولی که روی داده‌های متعلق به یک نمونه مشخص کار می‌کنند، استفاده از تابع عضو غیر `static` معمولاً انتخاب طبیعی‌تری است.

## **درک پارامترهای صریح شیء در C++23 (Understanding Explicit Object Parameters in C++23)**

یکی دیگر از قابلیت‌های مهم C++23، پارامترهای صریح شیء (Explicit Object Parameters) است که با نام Deducing this نیز شناخته می‌شود.

این قابلیت هنگام طراحی پیاده‌سازی‌های عمومی `operator[]` و مدیریت مشخصه‌های `const` و ارجاع می‌تواند مفید باشد.

برای مثال:

```
#include <cstddef>
#include <vector>

class Matrix {
private:
    std::size_t columns;
    std::vector<int> data;

public:
    Matrix(std::size_t rows, std::size_t cols)
        : columns(cols),
          data(rows * cols) {}

    template<typename Self>
    decltype(auto) operator[](
        this Self& self,
        std::size_t row,
        std::size_t column
    ) {
        return self.data[row * self.columns + column];
    }
};
```

در اینجا، `Self` نشان‌دهنده نوع شیئی است که تابع روی آن فراخوانی شده است.

این قابلیت اجازه می‌دهد یک پیاده‌سازی واحد، هم اشیای `const` و هم اشیای غیر `const` را مدیریت کند.

نکته مهم این است که Explicit Object Parameters و عملگر اندیس‌گذاری چندبعدی، دو قابلیت مستقل در C++23 هستند که می‌توان آن‌ها را با یکدیگر ترکیب کرد.

استفاده از Deducing this برای تمام کانتینرها ضروری نیست. تصمیم به استفاده از آن باید بر اساس این موضوع باشد که آیا باعث کاهش تکرار کد و بهبود خوانایی می‌شود یا خیر.

## **درک ترتیب ارزیابی آرگومان‌ها (Understanding Argument Evaluation Order)**

هنگام طراحی یک `operator[]` چندبعدی، ترتیب ارزیابی آرگومان‌ها اهمیت دارد.

برای مثال:

```
object[getFirstIndex(), getSecondIndex()];
```

نباید فرض کنید که `getFirstIndex()` الزاماً پیش از `getSecondIndex()` ارزیابی می‌شود.

از C++17 به بعد، آرگومان‌های یک تابع نسبت به یکدیگر دارای ترتیب ارزیابی Indeterminately Sequenced هستند. این یعنی یکی از آرگومان‌ها ابتدا ارزیابی می‌شود، اما ارزیابی آن‌ها با یکدیگر تداخل پیدا نمی‌کند.

ترتیب دقیق ارزیابی مشخص نشده است.

بنابراین، باید از نوشتن کدی که به ترتیب ارزیابی آرگومان‌ها وابسته است خودداری کرد.

برای مثال، از نوشتن عبارت‌هایی که در آن‌ها چندین آرگومان یک متغیر مشترک را به شکل‌های متعارض یا وابسته به یکدیگر تغییر می‌دهند، اجتناب کنید.

روش ایمن‌تر این است که مقادیر را پیش از فراخوانی عملگر محاسبه کنید:

```
int first = getFirstIndex();
int second = getSecondIndex();

object[first, second];
```

در این حالت، ترتیب اجرا صریح و قابل پیش‌بینی است.

## **کاربردهای عملی (Practical Applications)**

عملگر اندیس‌گذاری چندبعدی می‌تواند در انواع مختلف پروژه‌های C++ مورد استفاده قرار گیرد.

### **پردازش تصویر (Image Processing)**

در پردازش تصویر، داده‌ها معمولاً با استفاده از دو یا چند بُعد مدیریت می‌شوند.

برای مثال، دسترسی به یک پیکسل ممکن است به مختصات افقی و عمودی نیاز داشته باشد.

```
image[y, x];
```

این نحو می‌تواند در APIهای مبتنی بر آرایه، نسبت به استفاده از `image(y, x)` یا زنجیره‌کردن چندین فراخوانی `operator[]` خواناتر باشد.

### **محاسبات علمی (Scientific Computing)**

در پروژه‌های محاسبات علمی، ساختارهایی مانند ماتریس‌ها و تنسورها (Tensors) بسیار رایج هستند.

قابلیت جدید امکان طراحی APIهای ساده‌تری برای دسترسی به این داده‌ها را فراهم می‌کند.

برای مثال:

```
matrix[row, column];

tensor[batch, channel, height, width];
```

این قابلیت به‌خصوص در کتابخانه‌هایی که با داده‌های چندبعدی کار می‌کنند مفید است.

### **موتورهای بازی (Game Engines)**

در موتورهای بازی، ساختارهایی مانند Grid، نقشه‌های Voxel و ساختارهای داده مکانی ممکن است به چندین مختصات نیاز داشته باشند.

برای مثال:

```
world[x, y, z];
```

این نحو می‌تواند دسترسی به اطلاعات موجود در موقعیت‌های سه‌بعدی را خواناتر کند.

### **پردازش داده‌های چندبعدی (Multidimensional Data Processing)**

در برنامه‌هایی که داده‌ها با ابعادی مانند Batch، Channel، Height و Width سازمان‌دهی می‌شوند، این قابلیت می‌تواند یک API مناسب برای دسترسی به داده‌ها فراهم کند.

برای مثال:

```
dataset[batch, channel, row, column];
```

بااین‌حال، چنین پروژه‌هایی باید هزینه محاسبه اندیس، چیدمان حافظه و محلی‌بودن داده‌ها نسبت به کش (Cache Locality) را نیز در نظر بگیرند.

## **محدودیت‌ها و ملاحظات مهم (Limitations and Important Considerations)**

با وجود مزایای این قابلیت، تمام چالش‌های طراحی ساختارهای داده چندبعدی برطرف نمی‌شوند.

### **آرایه‌های داخلی بدون تغییر باقی می‌مانند (Built-in Arrays Remain Unchanged)**

مهم‌ترین محدودیت این است که این قابلیت فقط امکان بازنویسی `operator[]` را در انواع تعریف‌شده توسط کاربر (User-defined Types) گسترش می‌دهد.

بنابراین، عبارت زیر همچنان در C++23 نامعتبر است:

```
int matrix[3][4];

matrix[1, 2] = 10;
```

آرایه‌های داخلی زبان از بازنویسی عملگر اندیس‌گذاری چندبعدی پشتیبانی نمی‌کنند.

### **عدم ایجاد خودکار فضای ذخیره‌سازی چندبعدی (No Automatic Multidimensional Storage)**

این قابلیت نحوه ذخیره‌سازی داده‌ها در حافظه را تغییر نمی‌دهد.

توسعه‌دهندگان همچنان باید درباره موارد زیر تصمیم‌گیری کنند:

- تخصیص حافظه (Memory Allocation).
- نحوه ذخیره‌سازی عناصر.
- چیدمان داده‌ها (Data Layout).
- محاسبه Offset.
- مدیریت طول عمر (Lifetime Management).
- بررسی محدوده اندیس‌ها (Bounds Checking).

به عبارت دیگر، عملگر اندیس‌گذاری چندبعدی فقط نحو مورد استفاده برای دسترسی به داده‌ها را بهبود می‌دهد.

### **مدیریت بررسی محدوده اندیس‌ها (Managing Bounds Checking)**

عملگر اندیس‌گذاری به‌صورت خودکار بررسی محدوده انجام نمی‌دهد.

برای مثال، اگر پیاده‌سازی به اندیسی خارج از محدوده معتبر دسترسی پیدا کند، ممکن است رفتار تعریف‌نشده رخ دهد.

طراحی API باید به‌روشنی مشخص کند که دسترسی خارج از محدوده چگونه مدیریت می‌شود.

روش‌های متداول عبارت‌اند از:

- استفاده از `assert` برای اعتبارسنجی اندیس‌ها در نسخه‌های Debug.
- پرتاب استثنا در صورت نامعتبر بودن اندیس.
- ارائه یک API جداگانه برای دسترسی همراه با بررسی محدوده.
- استفاده از ابزارهایی مانند Sanitizerها برای شناسایی خطاهای حافظه در زمان توسعه.

برای مثال:

```
#include <stdexcept>
#include <vector>

class Matrix {
private:
    std::size_t rows;
    std::size_t columns;
    std::vector<int> data;

public:
    Matrix(std::size_t r, std::size_t c)
        : rows(r),
          columns(c),
          data(r * c) {}

    int& operator[](std::size_t row, std::size_t column) {
        if (row >= rows || column >= columns) {
            throw std::out_of_range("Matrix index out of range");
        }

        return data[row * columns + column];
    }
};
```

در این مثال، دسترسی خارج از محدوده با استفاده از `std::out_of_range` گزارش می‌شود.

### **احتمال سرریز هنگام محاسبه اندیس (Potential Overflow During Index Calculation)**

در آرایه‌های بزرگ، محاسبه Offset ممکن است به سرریز عدد صحیح (Integer Overflow) منجر شود.

برای مثال:

```
index = row * columns + column;
```

اگر حاصل‌ضرب `row * columns` از محدوده قابل نمایش نوع مربوطه بیشتر شود، نتیجه معتبر نخواهد بود.

بنابراین، پیاده‌سازی‌هایی که نسبت به کارایی یا ایمنی حساس هستند، باید ظرفیت فضای ذخیره‌سازی را اعتبارسنجی کنند و پیش از تخصیص حافظه و دسترسی به عناصر، احتمال سرریز را بررسی کنند.

### **انتخاب نام‌های مناسب برای پارامترها (Choosing Appropriate Parameter Names)**

ترتیب پارامترها در یک `operator[]` چندبعدی اهمیت دارد.

برای مثال، اعلان‌های زیر معانی متفاوتی دارند:

```
int& operator[](std::size_t row, std::size_t column);
```

```
int& operator[](std::size_t column, std::size_t row);
```

هر دو از نظر نحوی معتبر هستند، اما ترتیب مورد انتظار پارامترها باید به‌روشنی مستند شود.

بهتر است ترتیب پارامترها با تعریف ابعاد کانتینر و مستندات آن هماهنگ باشد.

### **در نظر گرفتن اشیای واسط (Considering Proxy Objects)**

`operator[]` همیشه مجبور نیست یک ارجاع مستقیم به عنصر را برگرداند.

برخی طراحی‌ها ممکن است به یک شیء واسط (Proxy Object) نیاز داشته باشند.

این موضوع برای ساختارهایی مانند کانتینرهای بیت و نمایش‌های فشرده داده اهمیت دارد.

برای مثال، اگر عناصر یک کانتینر در قالبی Bit-packed ذخیره شده باشند، دسترسی به یک عنصر ممکن است مستلزم محاسبه و تغییر یک بیت مشخص درون یک عدد صحیح باشد.

در چنین شرایطی، ممکن است امکان بازگرداندن یک ارجاع معمولی وجود نداشته باشد و استفاده از یک Proxy Object رفتار مناسب‌تری را فراهم کند.

چندبعدی بودن `operator[]` مانعی برای استفاده از Proxy Objectها ایجاد نمی‌کند.

### **در نظر گرفتن طول عمر ارجاع‌های بازگشتی (Considering the Lifetime of Returned References)**

اگر `operator[]` یک ارجاع برمی‌گرداند، شیئی که این ارجاع به آن اشاره می‌کند باید تا زمانی که ارجاع مورد استفاده قرار می‌گیرد معتبر باقی بماند.

برای مثال، بازگرداندن ارجاع به یک متغیر محلی اشتباه است:

```
class Matrix {
public:
    int& operator[](int row, int column) {
        int value = row + column;
        return value;
    }
};
```

این کد رفتار نامعتبر دارد، زیرا ارجاعی به یک متغیر محلی برمی‌گرداند.

بنابراین، ارجاع بازگشتی باید به داده‌ای اشاره کند که طول عمر مناسبی دارد.

## **مقایسه روش‌های مختلف دسترسی چندبعدی (Comparing Different Multidimensional Access Approaches)**

در C++ روش‌های مختلفی برای دسترسی به داده‌های چندبعدی وجود دارد.

بهترین انتخاب به API، ساختار داده و نیازمندی‌های پروژه بستگی دارد.

| روش                       | نحو          | استاندارد          | ویژگی اصلی             |
| ------------------------- | ------------ | ------------------ | ---------------------- |
| زنجیره‌کردن `operator[]`  | `a[i][j]`    | معتبر پیش از C++23 | دسترسی مرحله‌به‌مرحله  |
| استفاده از `operator()`   | `a(i, j)`    | معتبر پیش از C++23 | دریافت چندین پارامتر   |
| عملگر اندیس‌گذاری چندبعدی | `a[i, j]`    | C++23              | دسترسی مستقیم چندبعدی  |
| `std::mdspan`             | `view[i, j]` | C++23              | نمای چندبعدی استاندارد |

### **چه زمانی باید از Multidimensional operator[] استفاده کنیم؟ (When Should You Use Multidimensional operator[]?)**

این روش در شرایط زیر مناسب است:

- ساختار داده از نظر ماهیت رفتاری شبیه یک آرایه است.
- نیاز به دسترسی مستقیم به چندین بُعد وجود دارد.
- قصد دارید API خوانایی برای ماتریس‌ها یا تنسورها طراحی کنید.
- می‌خواهید در بسیاری از پیاده‌سازی‌ها از تعریف انواع واسط جلوگیری کنید.

### **چه زمانی operator() انتخاب مناسب‌تری است؟ (When Is operator() More Appropriate?)**

در برخی طراحی‌ها، همچنان `operator()` انتخاب بهتری است.

برای مثال:

- زمانی که شیء بیشتر شبیه یک تابع یا Functor رفتار می‌کند.
- زمانی که API از نظر معنایی یک عملیات محاسباتی را نمایش می‌دهد.
- زمانی که سازگاری با استانداردهای قدیمی‌تر C++ ضروری است.
- زمانی که قراردادها و الگوهای موجود در پروژه استفاده از `operator()` را ترجیح می‌دهند.

بنابراین، معرفی قابلیت جدید به معنای Deprecated شدن `operator()` نیست.

### **چه زمانی باید از std::mdspan استفاده کنیم؟ (When Should You Use std::mdspan?)**

اگر هدف اصلی ایجاد یک نمای چندبعدی روی حافظه موجود باشد، `std::mdspan` معمولاً انتخاب مناسبی است.

این کلاس امکانات استانداردی برای موارد زیر فراهم می‌کند:

- مدیریت ابعاد (Extents).
- انتخاب چیدمان حافظه (Layouts).
- دسترسی چندبعدی.
- تعریف Strideها.
- کار با داده‌های پیوسته یا داده‌هایی با چیدمان سفارشی.

بنابراین، پیش از پیاده‌سازی یک نمای چندبعدی جدید، بهتر است قابلیت‌های `std::mdspan` بررسی شوند.

## **ملاحظات سازگاری کامپایلرها (Compiler Compatibility Considerations)**

عملگر اندیس‌گذاری چندبعدی بخشی از استاندارد C++23 است، اما پشتیبانی عملی از آن به نسخه کامپایلر و میزان پیاده‌سازی قابلیت‌های C++23 در آن بستگی دارد.

کامپایلرهای مختلف ممکن است پشتیبانی از این قابلیت را در نسخه‌های متفاوتی ارائه کرده باشند.

بنابراین، صرفاً فعال‌کردن حالت زبان C++23 تضمین نمی‌کند که تمام قابلیت‌های این استاندارد در نسخه نصب‌شده کامپایلر در دسترس باشند.

### **فعال‌سازی C++23 (Enabling C++23)**

برای GCC و Clang می‌توان از گزینه کامپایلر زیر استفاده کرد:

```
g++ -std=c++23 main.cpp -o app
```

یا:

```
clang++ -std=c++23 main.cpp -o app
```

برای Microsoft Visual C++ می‌توان از گزینه زیر استفاده کرد:

```
cl /std:c++latest /EHsc main.cpp
```

گزینه `latest` جدیدترین قابلیت‌های پشتیبانی‌شده توسط نسخه نصب‌شده MSVC را فعال می‌کند. این گزینه لزوماً به معنای پیاده‌سازی کامل استاندارد C++23 نیست.

### **بررسی Feature Test Macro**

برای بررسی پشتیبانی کامپایلر از قابلیت‌های مشخص زبان می‌توان از Feature Test Macroها استفاده کرد.

ماکروی مربوط به عملگر اندیس‌گذاری چندبعدی عبارت است از:

```
__cpp_multidimensional_subscript
```

کد زیر نحوه بررسی آن را نشان می‌دهد:

```
#include <iostream>

int main() {
#ifdef __cpp_multidimensional_subscript
    std::cout << __cpp_multidimensional_subscript << '\n';
#else
    std::cout << "Multidimensional subscript is not supported\n";
#endif
}
```

این ماکرو توسط کامپایلر و به‌عنوان بخشی از پیاده‌سازی قابلیت زبان ارائه می‌شود و برای تشخیص پشتیبانی مفید است.

مقدار Feature Test Macro این قابلیت برابر با `202110L` است.

نکته مهم این است که پشتیبانی از این ماکرو به معنای پشتیبانی از تمام قابلیت‌های C++23 یا کل کتابخانه استاندارد C++23 نیست.

برای مثال، پشتیبانی از `operator[]` چندبعدی لزوماً در دسترس بودن کامل `std::mdspan` را تضمین نمی‌کند.

## **اشتباهات رایج (Common Mistakes)**

### **استفاده نادرست از آرایه‌های داخلی (Using Built-in Arrays Incorrectly)**

کد زیر همچنان در C++23 نامعتبر است:

```
int array[5][5];

array[2, 3] = 10;
```

دلیل این موضوع آن است که آرایه‌های داخلی از بازنویسی عملگر اندیس‌گذاری چندبعدی پشتیبانی نمی‌کنند.

برای این نوع آرایه باید از نحو زیر استفاده شود:

```
array[2][3] = 10;
```

### **فرض اینکه تعداد ابعاد به‌صورت خودکار تعیین می‌شود (Assuming the Number of Dimensions Is Automatically Determined)**

کامپایلر تعداد ابعاد موردنیاز را به‌صورت خودکار از ساختار داده استنباط نمی‌کند.

توسعه‌دهنده باید سربار مناسب را تعریف کند.

برای مثال:

```
class Matrix {
public:
    int& operator[](int row, int column);
};
```

فقط همین تعداد پارامتر توسط این سربار پشتیبانی می‌شود.

### **فرض اینکه این قابلیت جایگزین std::mdspan می‌شود (Assuming This Feature Replaces std::mdspan)**

این دو مفهوم در سطوح متفاوتی عمل می‌کنند.

عملگر اندیس‌گذاری چندبعدی یک قابلیت زبانی است، درحالی‌که `std::mdspan` یک مؤلفه از کتابخانه استاندارد است.

بنابراین، این دو رقیب مستقیم یکدیگر محسوب نمی‌شوند.

### **نادیده گرفتن ترتیب پارامترها (Ignoring Parameter Ordering)**

هنگام طراحی یک API، ترتیب اندیس‌ها باید به‌روشنی تعریف و مستند شود.

برای مثال، یک ماتریس ممکن است از قرارداد دسترسی زیر پیروی کند:

```
matrix[row, column];
```

اما یک تصویر ممکن است از ترتیب زیر استفاده کند:

```
image[y, x];
```

این تفاوت باید از طریق نام پارامترها و مستندات API کاملاً مشخص باشد.

### **استفاده نادرست از عملگر کاما (Misusing the Comma Operator)**

در C++23، عبارت زیر می‌تواند برای کلاسی که سربار مناسب دارد، دو آرگومان مستقل را نمایش دهد:

```
matrix[row, column];
```

اما اگر هدف استفاده از عملگر کاما باشد، استفاده از پرانتز الزامی است:

```
matrix[(row, column)];
```

این دو عبارت معنای یکسانی ندارند.

## **جمع‌بندی (Summary)**

عملگر اندیس‌گذاری چندبعدی یکی از قابلیت‌های کاربردی اضافه‌شده به C++23 است که یک محدودیت قدیمی در `operator[]` را برطرف می‌کند.

پیش از این استاندارد، `operator[]` فقط می‌توانست یک پارامتر صریح دریافت کند؛ بنابراین، دسترسی چندبعدی معمولاً مستلزم زنجیره‌کردن چندین عملگر یا استفاده از `operator()` بود.

اکنون در C++23 توسعه‌دهندگان می‌توانند یک `operator[]` عضو با قابلیت دریافت چندین پارامتر تعریف کنند و دسترسی مستقیمی به داده‌های چندبعدی فراهم آورند.

مهم‌ترین مزایای این قابلیت عبارت‌اند از:

- ساده‌تر شدن طراحی ماتریس‌ها و تنسورها.
- افزایش خوانایی نحو دسترسی چندبعدی.
- حذف نیاز به انواع واسط در بسیاری از پیاده‌سازی‌ها.
- فراهم‌شدن امکان طراحی APIهای عمومی انعطاف‌پذیرتر.
- یکپارچگی طبیعی با `std::mdspan` در C++23.

بااین‌حال، باید به خاطر داشت که این قابلیت نحوه ذخیره‌سازی داده‌ها، مدیریت حافظه یا بررسی محدوده اندیس‌ها را تغییر نمی‌دهد.

همچنین نمی‌توان از آن برای آرایه‌های داخلی زبان استفاده کرد و تنها برای انواعی در دسترس است که `operator[]` بازنویسی‌شده مناسبی تعریف کرده باشند.

در نهایت، درک تفاوت میان عملگر اندیس‌گذاری چندبعدی، `operator()`، زنجیره‌کردن `operator[]` و `std::mdspan` برای توسعه‌دهندگان C++ اهمیت زیادی دارد. انتخاب روش مناسب می‌تواند تأثیر مستقیمی بر خوانایی API، انعطاف‌پذیری و کیفیت کلی طراحی داشته باشد.

## **مراجع و منابع مطالعاتی بیشتر (References and Further Reading)**

منابع زیر برای مطالعه بیشتر و بررسی جزئیات رسمی این قابلیت پیشنهاد می‌شوند:

- [P2128R6 — Multidimensional subscript operator](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2128r6.pdf?utm_source=chatgpt.com) — پیشنهاد استانداردسازی عملگر اندیس‌گذاری چندبعدی.
- [cppreference — Operator overloading](https://en.cppreference.com/w/cpp/language/operators?utm_source=chatgpt.com) — مستندات مربوط به بازنویسی عملگرها.
- [cppreference — Array subscript operator](https://en.cppreference.com/w/cpp/language/operator_member_access?utm_source=chatgpt.com) — جزئیات مربوط به رفتار عملگر اندیس‌گذاری.
- [cppreference — std::mdspan](https://en.cppreference.com/w/cpp/container/mdspan?utm_source=chatgpt.com) — مستندات کلاس استاندارد `std::mdspan`.
- [P2589R1 — static operator()](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2589r1.pdf?utm_source=chatgpt.com) — جزئیات مربوط به عملگرهای عضو ایستا.
- [P0847R7 — Deducing this](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html?utm_source=chatgpt.com) — مستندات مربوط به Explicit Object Parameters.
- [C++23 Feature Support](https://en.cppreference.com/w/cpp/compiler_support/23?utm_source=chatgpt.com) — وضعیت پشتیبانی کامپایلرها از قابلیت‌های C++23.

**یک نکته فنی برای انتشار به‌عنوان رفرنس:** ترجمه بالا ساختار و محتوای منبع را حفظ کرده است. بااین‌حال، پیشنهاد می‌کنم پیش از انتشار نهایی، چند مورد فنی متن اصلی نیز بازبینی شوند؛ به‌خصوص تفاوت قواعد ارزیابی عبارت‌های کاما در آرایه‌های داخلی و کلاس‌ها در C++23، محدودیت‌های `static operator[]` و جزئیات پشتیبانی کامپایلرها. حفظ متن منبع نباید باعث انتقال احتمالی ابهام‌های فنی آن به رفرنس نهایی شود.

---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>