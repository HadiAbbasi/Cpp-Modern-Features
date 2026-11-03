<div align="center">

[🇺🇸 English](../../en/cpp20/std_make_shared_for_overwrite.md) | [🇮🇷 فارسی](./std_make_shared_for_overwrite.md)

</div>

---

# بررسی std::make_shared_for_overwrite در ++C20

## مقدمه

قابلیت `std::make_shared_for_overwrite` در استاندارد ++C20 به خانواده
ابزارهای ساخت `std::shared_ptr` اضافه شد. هدف اصلی آن ایجاد یک شیء یا
آرایه در حافظهٔ پویا، بدون انجام مقداردهی اولیهٔ صفر یا مقداردهی اولیهٔ
مقداری (`value-initialization`) است.

این تفاوت زمانی اهمیت پیدا می‌کند که برنامه قرار است بلافاصله پس از
تخصیص، تمام فضای شیء یا آرایه را با داده‌های معتبر بازنویسی کند؛ برای
نمونه، هنگام دریافت داده از فایل، شبکه، یک API سطح پایین یا یک تابع
پرکنندهٔ بافر. در چنین شرایطی، مقداردهی اولیهٔ غیرضروری می‌تواند هزینهٔ
اضافی ایجاد کند.

بااین‌حال، حذف مقداردهی اولیه مسئولیت مهمی به عهدهٔ برنامه‌نویس می‌گذارد:
هیچ مقداری که هنوز مقداردهی نشده است نباید خوانده شود. بنابراین این تابع
یک ابزار بهینه‌سازی هدفمند است، نه جایگزینی عمومی برای
`std::make_shared`.

## معرفی تابع و هدف طراحی

تابع `std::make_shared_for_overwrite` در header استاندارد `<memory>`
تعریف شده و از ++C20 در دسترس است. این تابع یک `std::shared_ptr` می‌سازد،
اما شیء یا عناصر آرایه را به‌صورت پیش‌فرض مقداردهی اولیه می‌کند، نه به‌صورت
مقداری.

تفاوت اصلی در نوع initialization است:

-   در `std::make_shared<T>()`، شیء غیرآرایه‌ای به‌صورت
    value-initialization ساخته می‌شود.
-   در `std::make_shared_for_overwrite<T>()`، شیء غیرآرایه‌ای به‌صورت
    default-initialization ساخته می‌شود.
-   برای overloadهای آرایه‌ای نیز عناصر آرایه به‌صورت
    default-initialization ساخته می‌شوند، نه value-initialization.

این تفاوت برای انواع scalar مانند `int` و `double` بسیار محسوس است.
مقداردهی مقداری یک `int` معمولاً آن را صفر می‌کند، درحالی‌که
default-initialization یک متغیر scalar را مقداردهی نمی‌کند.

## امضای تابع و overloadها

الگوی استفاده برای یک شیء غیرآرایه‌ای به شکل زیر است:

``` cpp
template<class T>
std::shared_ptr<T> make_shared_for_overwrite();
```

برای آرایهٔ دارای اندازهٔ زمان اجرا، overload زیر در دسترس است:

``` cpp
template<class T>
std::shared_ptr<T> make_shared_for_overwrite(std::size_t count);
```

همچنین می‌توان نوع آرایهٔ دارای اندازهٔ ثابت را مشخص کرد:

``` cpp
template<class T>
std::shared_ptr<T> make_shared_for_overwrite();
```

در این حالت، آرگومان قالبی می‌تواند نوعی مانند `T[N]` باشد. برای آرایهٔ با
اندازهٔ زمان اجرا، نوع قالبی به شکل `T[]` نوشته می‌شود و تعداد عناصر
به‌عنوان آرگومان تابع ارسال می‌شود.

نمونه‌های متداول:

``` cpp
auto object = std::make_shared_for_overwrite<MyType>();

auto fixed_array =
    std::make_shared_for_overwrite<int[128]>();

auto dynamic_array =
    std::make_shared_for_overwrite<int[]>(128);
```

در نمونهٔ آخر، اندازهٔ آرایه در زمان اجرا تعیین می‌شود. برای آرایهٔ ثابت،
اندازه بخشی از نوع است.

## تفاوت initializationها

### مفهوم value-initialization

در value-initialization، قواعد زبان ++C تعیین می‌کنند که شیء چگونه
مقداردهی شود. برای انواع scalar، نتیجه صفر است:

``` cpp
auto a = std::make_shared<int>();
auto b = std::make_shared<double>();

// Both objects are value-initialized.
```

در این مثال، مقدار اولیهٔ `*a` برابر صفر و مقدار اولیهٔ `*b` برابر `0.0`
است.

### مفهوم default-initialization

در default-initialization، رفتار به نوع شیء بستگی دارد:

-   برای نوع scalar مانند `int`، هیچ مقداردهی‌ای انجام نمی‌شود و مقدار شیء
    نامعین خواهد بود.
-   برای نوع class، constructor مناسب اجرا می‌شود.
-   برای آرایه، هر عنصر به‌صورت جداگانه default-initialize می‌شود.

بنابراین عبارت «بدون initialization» توصیف دقیقی برای تمام انواع نیست.
تابع برای اشیای class همچنان constructor را اجرا می‌کند؛ چیزی که حذف
می‌شود مقداردهی مقداری و صفرسازی ضمنیِ ناشی از آن است.

### مقایسهٔ عملی

``` cpp
#include <memory>

struct Record {
    int id;
    double score;

    Record() : id(0), score(0.0) {}
};

int main() {
    auto initialized = std::make_shared<int>();
    auto overwritten = std::make_shared_for_overwrite<int>();

    auto record = std::make_shared_for_overwrite<Record>();

    // initialized is zero.
    // overwritten must be assigned before it is read.
    // Record's constructor runs and initializes its members.
}
```

در این مثال، `initialized` مقدار صفر دارد. مقدار `overwritten` تا پیش از
نوشتن، قابل استفاده نیست. در مقابل، constructor نوع `Record` اجرا می‌شود
و اعضای آن را مقداردهی می‌کند.

## رفتار برای انواع class و struct

استفاده از `std::make_shared_for_overwrite` به معنای دور زدن constructor
نیست. اگر نوع موردنظر یک class باشد، قواعد default-initialization زبان
اعمال می‌شوند و constructor پیش‌فرض آن اجرا می‌شود.

نکتهٔ مهم این است که اجرای constructor لزوماً به معنای مقداردهی تمام اعضا
نیست:

``` cpp
#include <memory>

struct Packet {
    int size;
    int flags;

    Packet() : size(0) {}
};

int main() {
    auto packet = std::make_shared_for_overwrite<Packet>();

    // packet->size is initialized to zero.
    // packet->flags is not initialized by the constructor.
}
```

در این مثال، `size` مقداردهی شده است، اما `flags` چنین نیست. در نتیجه
نباید مقدار `flags` خوانده شود، مگر آنکه پیش از آن مقدار معتبری به آن
اختصاص داده شود.

این موضوع برای classهایی که constructor پیش‌فرض آن‌ها بعضی اعضا را
مقداردهی نمی‌کند اهمیت ویژه‌ای دارد. هنگام بازبینی چنین کدی باید تمام
مسیرهای مقداردهی اعضا بررسی شوند.

## ساخت آرایه‌ها

یکی از قابلیت‌های مهم این API پشتیبانی مستقیم از آرایه‌ها است. این قابلیت
برای ساخت bufferهایی با مالکیت اشتراکی مفید است.

### آرایه با اندازهٔ ثابت

``` cpp
#include <memory>

int main() {
    auto values =
        std::make_shared_for_overwrite<int[256]>();

    for (std::size_t i = 0; i < 256; ++i) {
        values[i] = static_cast<int>(i);
    }
}
```

عناصر آرایه از نوع `int` هستند و default-initialization آن‌ها مقداری
تولید نمی‌کند. حلقه پیش از خواندن عناصر، همهٔ آن‌ها را می‌نویسد.

### آرایه با اندازهٔ زمان اجرا

``` cpp
#include <cstddef>
#include <memory>

int main() {
    const std::size_t count = 4096;

    auto buffer =
        std::make_shared_for_overwrite<std::byte[]>(count);

    for (std::size_t i = 0; i < count; ++i) {
        buffer[i] = std::byte{0};
    }
}
```

در این نمونه یک آرایهٔ پویا از `std::byte` ساخته شده است. هر بایت پیش از
استفاده مقداردهی می‌شود. در کد واقعی، نوشتن داده می‌تواند به‌جای حلقه از یک
API دریافت داده یا عملیات ورودی/خروجی انجام شود.

### آرایه‌های چندبعدی

قواعد آرایه‌های تو‌در‌تو نیز از قواعد عادی initialization پیروی می‌کنند.
برای مثال، نوع آرایهٔ دوبعدی ثابت را می‌توان به‌عنوان نوع قالبی مشخص کرد:

``` cpp
#include <memory>

int main() {
    auto matrix =
        std::make_shared_for_overwrite<int[16][16]>();

    for (int row = 0; row < 16; ++row) {
        for (int column = 0; column < 16; ++column) {
            matrix[row][column] = row + column;
        }
    }
}
```

تمام عناصر باید پیش از خواندن مقداردهی شوند. در آرایه‌های بزرگ، بهتر است
تعداد عناصر و مسیرهای نوشتن به‌صورت روشن و قابل بررسی مدیریت شوند.

## کاربرد در bufferهای ورودی و خروجی

یکی از مناسب‌ترین کاربردهای این تابع، تخصیص حافظه‌ای است که قرار است یک
عملیات دیگر تمام یا بخش مشخصی از آن را پر کند.

برای مثال، فرض کنید یک API داده را در buffer مقصد می‌نویسد و تعداد
بایت‌های نوشته‌شده را برمی‌گرداند:

``` cpp
#include <cstddef>
#include <memory>

std::size_t read_data(std::byte* destination,
                      std::size_t capacity);

int main() {
    constexpr std::size_t capacity = 8192;

    auto buffer =
        std::make_shared_for_overwrite<std::byte[]>(capacity);

    const std::size_t bytes_read =
        read_data(buffer.get(), capacity);

    // Only the first bytes_read bytes contain received data.
}
```

در این مثال فقط محدودهٔ `[0, bytes_read)` دارای دادهٔ دریافت‌شده است.
خواندن بخش باقی‌ماندهٔ buffer مجاز نیست، مگر آنکه جداگانه مقداردهی شده
باشد.

در طراحی چنین APIهایی باید قرارداد تابع پرکننده مشخص کند که: - چه تعداد
بایت ممکن است نوشته شود؛ - آیا در صورت خطا بخشی از buffer تغییر می‌کند؛ -
آیا تمام محدودهٔ اعلام‌شده واقعاً مقداردهی می‌شود؛ - و مصرف‌کننده چگونه طول
دادهٔ معتبر را دریافت می‌کند.

### نگهداری buffer با مالکیت اشتراکی

اگر چند بخش برنامه باید طول عمر buffer را به‌صورت اشتراکی مدیریت کنند،
`std::shared_ptr` می‌تواند مناسب باشد:

``` cpp
#include <cstddef>
#include <memory>

using ByteBuffer = std::shared_ptr<std::byte[]>;

ByteBuffer create_buffer(std::size_t size) {
    return std::make_shared_for_overwrite<std::byte[]>(size);
}
```

با این حال، مالکیت اشتراکی به‌خودی‌خود دسترسی هم‌زمان امن ایجاد نمی‌کند. اگر
چند thread به buffer دسترسی دارند و دست‌کم یکی از آن‌ها می‌نویسد،
هماهنگ‌سازی مناسب همچنان ضروری است.

## مقایسه با std::make_shared

انتخاب بین دو تابع باید بر اساس نیاز واقعی به مقدار اولیه انجام شود.

  ----------------------------------------------------------------------------------
  ویژگی                   `std::make_shared`      `std::make_shared_for_overwrite`
  ----------------------- ----------------------- ----------------------------------
  استاندارد               ++C11                   ++C20

  initialization شیء      value-initialization    default-initialization
  غیرآرایه‌ای                                      

  مقدار اولیهٔ نوع scalar  معمولاً صفر              مقدار نامعین

  اجرای constructor کلاس  بله                     بله، در صورت نیاز طبق قواعد
                                                  default-initialization

  کاربرد معمول            ساخت شیء آمادهٔ استفاده  ساخت شیء یا buffer برای بازنویسی
                                                  فوری

  ریسک خواندن مقدار       کمتر                    بیشتر و نیازمند دقت صریح
  مقداردهی‌نشده                                    
  ----------------------------------------------------------------------------------

برای مثال، در کد زیر استفاده از `make_shared` انتخاب طبیعی‌تری است:

``` cpp
auto counter = std::make_shared<int>();
++(*counter);
```

مقدار اولیهٔ counter صفر است و افزایش آن معتبر خواهد بود.

اما در نمونهٔ زیر، استفاده از `make_shared_for_overwrite` می‌تواند مناسب
باشد، چون تمام عناصر پیش از خواندن نوشته می‌شوند:

``` cpp
auto samples =
    std::make_shared_for_overwrite<float[]>(sample_count);

fill_samples(samples.get(), sample_count);
process_samples(samples.get(), sample_count);
```

در این مثال، معتبر بودن `process_samples` به قرارداد `fill_samples`
وابسته است؛ تابع پرکننده باید تمام عناصری را که تابع پردازش می‌خواند
مقداردهی کرده باشد.

## مقایسه با تخصیص مستقیم حافظه

در ++C ابزارهای دیگری نیز برای تخصیص حافظه وجود دارند؛ از جمله `new`،
`std::unique_ptr` و allocatorها. انتخاب ابزار مناسب به مدل مالکیت و
نیازهای عمر شیء بستگی دارد.

### مقایسه با new

عبارت `new T` برای یک شیء غیرآرایه‌ای معمولاً default-initialization را
انجام می‌دهد. بنابراین از نظر نوع initialization می‌تواند به رفتار موردنظر
`make_shared_for_overwrite` نزدیک باشد:

``` cpp
int* raw = new int;
```

اما این روش مالکیت اشتراکی و مدیریت خودکار طول عمر را فراهم نمی‌کند.
همچنین باید آزادسازی حافظه و مدیریت exception به‌درستی انجام شود.

در مقابل، `std::make_shared_for_overwrite<T>()` یک `shared_ptr<T>`
برمی‌گرداند و مدیریت مالکیت را در اختیار کتابخانهٔ استاندارد قرار می‌دهد.

### مقایسه با std::unique_ptr

اگر مالکیت انحصاری کافی است، `std::unique_ptr` معمولاً انتخاب ساده‌تر و
کم‌هزینه‌تری از نظر مدل مالکیت است. برای مثال:

``` cpp
#include <memory>

auto buffer =
    std::make_unique_for_overwrite<std::byte[]>(4096);
```

تابع `std::make_unique_for_overwrite` نیز در ++C20 معرفی شده است و از
نظر هدف initialization مشابه است، اما مالکیت انحصاری را فراهم می‌کند.

قاعدهٔ طراحی این است که اگر واقعاً به چند مالک مستقل نیاز ندارید، صرفاً
برای راحتی از `shared_ptr` استفاده نکنید. هزینهٔ control block و عملیات
reference counting ممکن است غیرضروری باشد.

### مقایسه با allocatorها

خانوادهٔ allocatorها امکان کنترل بیشتری بر تخصیص حافظه و ساخت اشیا فراهم
می‌کنند. در ++C20 تابع `std::allocate_shared_for_overwrite` نیز برای ساخت
`shared_ptr` با allocator مشخص وجود دارد.

این تابع زمانی مفید است که برنامه از allocator سفارشی، memory resource
یا سیاست تخصیص خاصی استفاده می‌کند. در مقابل، `make_shared_for_overwrite`
برای حالت معمولی که allocator سفارشی لازم نیست، رابط ساده‌تری ارائه
می‌دهد.

## ارتباط با std::allocate_shared_for_overwrite

تابع `std::allocate_shared_for_overwrite` هم‌خانوادهٔ allocator-aware این
قابلیت است و در ++C20 ارائه شده است. تفاوت اصلی آن با
`make_shared_for_overwrite` این است که allocator را به‌عنوان آرگومان
دریافت می‌کند.

نمونهٔ استفاده:

``` cpp
#include <memory>

int main() {
    std::allocator<int> allocator;

    auto value =
        std::allocate_shared_for_overwrite<int>(allocator);

    *value = 42;
}
```

انتخاب این تابع زمانی منطقی است که سیاست تخصیص حافظه بخشی از طراحی سیستم
باشد. در غیر این صورت، استفاده از `make_shared_for_overwrite` معمولاً
خواناتر است.

## نکات مربوط به طول عمر و کنترل مالکیت

مقدار بازگشتی از نوع `std::shared_ptr` است؛ بنابراین قواعد معمول مالکیت
اشتراکی همچنان برقرار هستند:

-   شمارندهٔ مالکیت با کپی شدن `shared_ptr` افزایش می‌یابد.
-   با از بین رفتن آخرین مالک اشتراکی، شیء یا آرایه تخریب می‌شود.
-   استفاده از `std::weak_ptr` مالکیت را افزایش نمی‌دهد.
-   عملیات reference counting، ایمنی دسترسی هم‌زمان به خود شیء را تضمین
    نمی‌کند.

یک نکتهٔ پیاده‌سازی مهم نیز به تخصیص مشترک object و control block مربوط
است. پیاده‌سازی‌های متداول `make_shared` و هم‌خانواده‌های آن، شیء و control
block را در یک تخصیص قرار می‌دهند. این روش معمولاً تعداد تخصیص‌ها را کاهش
می‌دهد. بااین‌حال، اگر پس از نابودی آخرین `shared_ptr` هنوز `weak_ptr`
زنده‌ای وجود داشته باشد، حافظهٔ تخصیص‌یافته برای control block تا پایان عمر
آخرین `weak_ptr` قابل آزادسازی کامل نیست. در پیاده‌سازی‌های دارای تخصیص
مشترک، این موضوع می‌تواند آزادسازی فضای شیء را نیز به تأخیر بیندازد.

این رفتار در bufferهای بسیار بزرگ اهمیت دارد. اگر طول عمر `weak_ptr`ها
ممکن است از طول عمر مالکان اصلی بسیار بیشتر باشد، اثر مصرف حافظه را در
طراحی در نظر بگیرید.

## محدودیت‌ها و موارد نامناسب

استفاده از این تابع در شرایط زیر معمولاً انتخاب خوبی نیست:

-   زمانی که مقدار اولیهٔ صفر یا مقدار پیش‌فرض مشخصی برای منطق برنامه لازم
    است؛
-   زمانی که ممکن است بعضی اعضا یا عناصر پیش از خواندن مقداردهی نشوند؛
-   زمانی که چند مسیر خطا وجود دارد و تضمین مقداردهی کامل buffer دشوار
    است؛
-   زمانی که مالکیت اشتراکی لازم نیست و `std::unique_ptr` مناسب‌تر است؛
-   زمانی که هدف صرفاً کوتاه‌تر کردن کد است و هیچ هزینهٔ قابل‌اندازه‌گیری از
    initialization وجود ندارد.

همچنین این تابع ابزار ساخت یک شیء خام بدون اجرای constructor نیست. برای
انواع class، قواعد عادی ساخت شیء پابرجا هستند.

## نکات مهم دربارهٔ مقدارهای نامعین

برای نوع scalar که با default-initialization ساخته شده است، نباید مقدار
پیش از نوشتن خوانده شود. چنین کاری می‌تواند به رفتار تعریف‌نشده منجر شود؛
قواعد دقیق استاندارد دربارهٔ indeterminate value و موارد خاص آن به نوع و
زمینهٔ استفاده وابسته‌اند. رویکرد امن و قابل اتکا این است که مقداردهی را
پیش‌شرط هر نوع خواندن قرار دهید.

برای آرایه‌ها نیز مقداردهی جزئی باید با طول دادهٔ معتبر همراه باشد. صرف
داشتن یک buffer با ظرفیت مشخص به این معنا نیست که تمام عناصر آن دادهٔ
معتبر دارند.

یک الگوی مناسب، نگهداری ظرفیت و اندازهٔ واقعی داده به‌صورت جداگانه است:

``` cpp
#include <cstddef>
#include <memory>

struct Buffer {
    std::shared_ptr<std::byte[]> storage;
    std::size_t capacity = 0;
    std::size_t size = 0;
};

Buffer make_buffer(std::size_t capacity) {
    return {
        std::make_shared_for_overwrite<std::byte[]>(capacity),
        capacity,
        0
    };
}
```

در این طراحی، `size` باید فقط پس از نوشتن موفق داده افزایش یابد.
مصرف‌کننده نیز باید فقط محدودهٔ معتبر را پردازش کند.

## ملاحظات کارایی و اندازه‌گیری

مزیت بالقوهٔ این تابع حذف هزینهٔ مقداردهی اولیهٔ غیرضروری است. برای
bufferهای بزرگ، صفر کردن تمام حافظه و سپس بازنویسی آن می‌تواند پهنای باند
حافظه را بی‌دلیل مصرف کند.

بااین‌حال، بهبود کارایی تضمین‌شده نیست. نتیجه به عوامل متعددی وابسته است:

-   نوع داده و اندازهٔ تخصیص؛
-   رفتار allocator و کتابخانهٔ استاندارد؛
-   نحوهٔ استفاده از حافظه و الگوی دسترسی؛
-   اینکه تابع بعدی واقعاً تمام محدوده را بازنویسی می‌کند یا نه؛
-   بهینه‌سازی‌های compiler و ویژگی‌های runtime.

در نتیجه، انتخاب این تابع باید بر اساس profiling و اندازه‌گیری در
workload واقعی انجام شود. اگر داده‌ها فقط بخشی از buffer را پر می‌کنند،
مزیت احتمالی باید در برابر پیچیدگی مدیریت محدودهٔ معتبر سنجیده شود.

## اشتباهات رایج

### فرض صفر بودن عناصر

اشتباه رایج این است که برنامه‌نویس تصور کند آرایهٔ ساخته‌شده مانند آرایهٔ
حاصل از `make_shared` صفر شده است. چنین فرضی برای عناصر scalar نادرست
است.

### خواندن buffer پیش از پر شدن

دریافت یک pointer معتبر از `shared_ptr` به معنای مقداردهی شدن محتوای
حافظه نیست. معتبر بودن آدرس و معتبر بودن مقدار ذخیره‌شده دو مفهوم
متفاوت‌اند.

### نادیده گرفتن مقداردهی جزئی

بعضی APIها فقط بخشی از buffer را پر می‌کنند. مصرف کل ظرفیت به‌جای طول
واقعی داده می‌تواند باعث خواندن مقادیر مقداردهی‌نشده شود.

### فرض اینکه constructor اجرا نمی‌شود

این تابع constructor پیش‌فرض class را حذف نمی‌کند. اگر هدف برنامه جلوگیری
از اجرای constructor یا مدیریت دستی عمر اشیا باشد، این API ابزار مناسبی
نیست.

### انتخاب shared_ptr بدون نیاز واقعی

اگر buffer فقط یک مالک دارد، استفاده از `make_unique_for_overwrite`
معمولاً طراحی مناسب‌تری است. مالکیت اشتراکی باید بازتاب نیاز واقعی برنامه
باشد.

## راهنمای انتخاب

برای تصمیم‌گیری سریع می‌توان از قواعد زیر استفاده کرد:

  --------------------------------------------------------------------------
  نیاز برنامه                         انتخاب پیشنهادی
  ----------------------------------- --------------------------------------
  ساخت شیء با مقداردهی اولیهٔ معمول    `std::make_shared`

  ساخت شیء با مالکیت اشتراکی و        `std::make_shared_for_overwrite`
  بازنویسی فوری                       

  ساخت buffer با مالکیت انحصاری و     `std::make_unique_for_overwrite`
  بازنویسی فوری                       

  ساخت شیء با allocator سفارشی و      `std::allocate_shared_for_overwrite`
  مالکیت اشتراکی                      

  نیاز به buffer صفرشده               `std::make_shared` برای نوع آرایه‌ای
                                      مناسب
  --------------------------------------------------------------------------

## جمع‌بندی

تابع `std::make_shared_for_overwrite` در ++C20 راهی استاندارد برای ساخت
شیء یا آرایه با مالکیت اشتراکی، بدون value-initialization فراهم می‌کند.
این قابلیت به‌ویژه برای bufferهایی مفید است که بلافاصله با دادهٔ واقعی پر
می‌شوند و صفرسازی اولیهٔ آن‌ها هزینهٔ غیرضروری دارد.

مهم‌ترین نکتهٔ استفاده از این API درک تفاوت default-initialization و
value-initialization است. برای انواع scalar و عناصر آرایه، مقدار اولیه‌ای
تضمین نمی‌شود؛ بنابراین هر مقدار باید پیش از خواندن نوشته شود. برای انواع
class نیز constructor پیش‌فرض طبق قواعد زبان اجرا می‌شود و اعضایی که
constructor مقداردهی نمی‌کند همچنان ممکن است مقدار نامعین داشته باشند.

در نهایت، این تابع زمانی انتخاب مناسبی است که برنامه به مالکیت اشتراکی
نیاز دارد، تمام مسیرهای خواندن پس از مقداردهی معتبر قرار می‌گیرند و
اندازه‌گیری نشان می‌دهد حذف initialization هزینهٔ قابل‌توجهی را کاهش می‌دهد.
در سایر موارد، `std::make_shared` یا ابزارهای مالکیت انحصاری می‌توانند
انتخاب ساده‌تر و امن‌تری باشند.


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>