<div align="center">

[🇺🇸 English](../../en/cpp20/std_to_address.md) | [🇮🇷 فارسی](./std_to_address.md)

</div>

---

# راهنمای جامع `std::to_address` در ++C20

## مقدمه

در ++C20 تابع `std::to_address` به‌عنوان یک ابزار استاندارد در کتابخانهٔ `<memory>` معرفی شد تا بتوان آدرس خام یک ناحیهٔ حافظه یا شیء را از یک pointer-like type به دست آورد. اهمیت این تابع فقط در تبدیل یک نوع اشاره‌گر به `T*` خلاصه نمی‌شود؛ هدف اصلی آن فراهم‌کردن یک مسیر استاندارد و قابل‌گسترش برای کار با انواع اشاره‌گر سفارشی، allocatorها و iteratorهای پیوسته است.

در ++C، همهٔ انواعی که رفتار اشاره‌گر دارند الزاماً از نوع `T*` نیستند. برای نمونه، یک allocator ممکن است نوع اشاره‌گر اختصاصی خود را برگرداند؛ این نوع که معمولاً fancy pointer نامیده می‌شود، می‌تواند اطلاعاتی مانند offset نسبت به یک حافظهٔ مشترک، handle یا metadata را در خود نگه دارد. در چنین شرایطی، کد عمومی نباید فرض کند که می‌تواند مستقیماً از `operator*` یا `operator->` برای رسیدن به آدرس خام استفاده کند.

تابع `std::to_address` این تفاوت را پنهان می‌کند و در صورت پیاده‌سازی صحیح pointer-like type، آدرس خام متناظر را بدون نیاز به ایجاد reference از طریق dereference در اختیار قرار می‌دهد.

## مسئله‌ای که این تابع حل می‌کند

### تفاوت آدرس‌گیری با dereference

روش متداول برای به‌دست‌آوردن آدرس یک شیء موجود، استفاده از `std::addressof(*p)` است. این روش ابتدا اشاره‌گر را dereference می‌کند و سپس آدرس شیء حاصل را می‌گیرد. اما این الگو در همهٔ وضعیت‌های حافظه معتبر نیست.

برای مثال، allocator ممکن است فضایی را رزرو کرده باشد که هنوز هیچ شیئی در آن ساخته نشده است. در این وضعیت، آدرس storage وجود دارد، اما dereference کردن pointer برای ساختن reference به یک شیء زنده و معتبر مجاز نیست. تابع `std::to_address` برای چنین موقعیت‌هایی نیز طراحی شده است: می‌تواند آدرس storage را بدون تشکیل reference از طریق `*p` استخراج کند.

```cpp
#include <memory>

int main() {
    std::allocator<int> alloc;

    int* p = alloc.allocate(4);

    // The storage is allocated, but no int object has been constructed yet.
    int* raw = std::to_address(p);

    alloc.deallocate(p, 4);
}
```

در مثال بالا، `std::allocator<int>::allocate` حافظهٔ لازم را فراهم می‌کند، اما به‌تنهایی چهار شیء `int` را مقداردهی و ایجاد نمی‌کند. فراخوانی `std::to_address` برای گرفتن آدرس خام به dereference نیاز ندارد.

### پشتیبانی از fancy pointer

در طراحی allocatorها، نوع `pointer` لزوماً `T*` نیست. یک نوع اشاره‌گر سفارشی می‌تواند نمایش داخلی متفاوتی داشته باشد و تنها راه معتبر دسترسی به آدرس خام را از طریق customization مخصوص خود ارائه کند.

بدون یک customization point استاندارد، کتابخانه‌های عمومی ناچار می‌شدند برای هر نوع اشاره‌گر سفارشی منطق جداگانه بنویسند یا به جزئیات پیاده‌سازی آن وابسته شوند. `std::to_address` این نیاز را با سازوکاری مبتنی بر `std::pointer_traits` و `operator->` برطرف می‌کند.

## معرفی رابط استاندارد

### سرآیند و امضای تابع

این تابع در سرآیند `<memory>` تعریف شده و از ++C20 در دسترس است.

```cpp
#include <memory>

template<class T>
constexpr T* to_address(T* p) noexcept;

template<class Ptr>
constexpr auto to_address(const Ptr& p) noexcept;
```

دو overload اصلی وجود دارد:

- overload نخست برای raw pointerها مانند `int*`، `const Widget*` و `void*` است.
- overload دوم برای pointer-like typeها و انواعی مانند fancy pointerها طراحی شده است که می‌توانند آدرس خام خود را از طریق customization یا `operator->` ارائه کنند.

نوع بازگشتی overload مربوط به pointer-like type از expression مورد استفاده برای استخراج آدرس استنتاج می‌شود. در کاربرد معمول، نتیجه یک raw pointer به نوع عنصر خواهد بود.

### رفتار overload مربوط به raw pointer

برای ورودی از نوع `T*`، تابع همان pointer ورودی را بازمی‌گرداند. بنابراین این فراخوانی هیچ تبدیل یا dereference اضافی انجام نمی‌دهد.

```cpp
#include <memory>
#include <type_traits>

int main() {
    int value = 42;
    int* p = &value;

    int* raw = std::to_address(p);

    static_assert(std::is_same_v<decltype(raw), int*>);
    return raw == &value ? 0 : 1;
}
```

نوع `T` نباید function type باشد. در نتیجه، استفاده از این تابع برای function pointerها ill-formed است؛ این تابع برای آدرس اشیاء و storage آن‌ها تعریف شده است، نه تبدیل function pointerها.

### رفتار overload مربوط به pointer-like type

برای overload دوم، استاندارد دو مسیر را مشخص می‌کند:

- اگر expression زیر معتبر باشد، همان مسیر استفاده می‌شود:

  `std::pointer_traits<Ptr>::to_address(p)`

- در غیر این صورت، تابع به‌صورت بازگشتی از مسیر زیر استفاده می‌کند:

  `std::to_address(p.operator->())`

بنابراین وجود یک عضو به نام `operator->` به‌تنهایی تنها راه پشتیبانی نیست؛ specialization مناسب از `std::pointer_traits` می‌تواند رفتار استخراج آدرس را سفارشی کند.

## سازوکار `std::pointer_traits`

### نقش pointer traits

قالب `std::pointer_traits` از ++C11 وجود دارد و برای ارائهٔ اطلاعات استاندارد دربارهٔ pointer-like typeها طراحی شده است. این قالب می‌تواند اطلاعاتی مانند `element_type`، `difference_type` و قالب بازتولیدشدهٔ اشاره‌گر یعنی `rebind` را فراهم کند.

در ++C20، امکان تعریف عضو اختیاری `to_address` در specializationهای برنامه‌نویس نیز به این سازوکار افزوده شد. `std::to_address` این عضو را به‌عنوان customization بررسی می‌کند.

### پیاده‌سازی specialization سفارشی

فرض کنید یک نوع اشاره‌گر سفارشی، آدرس خام را در یک عضو داخلی نگه می‌دارد و نمی‌خواهد یا نمی‌تواند `operator->` معمولی ارائه کند. می‌توان specialization مربوط به `std::pointer_traits` را تعریف کرد.

```cpp
#include <cstddef>
#include <memory>

template<class T>
class offset_like_pointer {
public:
    using element_type = T;

    explicit offset_like_pointer(T* ptr) noexcept : ptr_(ptr) {}

private:
    T* ptr_;

    template<class>
    friend struct std::pointer_traits;
};

namespace std {
    template<class T>
    struct pointer_traits<offset_like_pointer<T>> {
        using pointer = offset_like_pointer<T>;
        using element_type = T;
        using difference_type = ptrdiff_t;

        template<class U>
        using rebind = offset_like_pointer<U>;

        static element_type* to_address(pointer p) noexcept {
            return p.ptr_;
        }
    };
}

int main() {
    int value = 10;
    offset_like_pointer<int> p{&value};

    int* raw = std::to_address(p);
    return *raw == 10 ? 0 : 1;
}
```

این نمونه صرفاً یک نمایش ساده برای توضیح customization است؛ pointer واقعی مبتنی بر offset معمولاً باید قواعد تبدیل، اعتبارسنجی، عمر حافظه و محدودیت‌های مربوط به segment یا mapping را نیز مدیریت کند. همچنین specializationهای برنامه‌نویس برای قالب‌های استاندارد باید مطابق قواعد مجاز استاندارد تعریف شوند؛ افزودن specialization به `std` برای یک نوع برنامه‌تعریف‌شده در این چارچوب انجام می‌شود، نه برای تغییر رفتار نوع‌های استاندارد.

### انتخاب میان `pointer_traits::to_address` و `operator->`

وجود هر دو مسیر انعطاف طراحی را بیشتر می‌کند:

- اگر نوع اشاره‌گر به یک روش ویژه برای تبدیل نیاز دارد، `pointer_traits::to_address` انتخاب مناسب‌تری است.
- اگر `operator->` به‌درستی آدرس خام یا pointer-like type دیگری را ارائه می‌کند، مسیر fallback کافی است.
- اگر هر دو مسیر وجود داشته باشند و expression مربوط به `pointer_traits<Ptr>::to_address(p)` معتبر باشد، مسیر `pointer_traits` اولویت دارد.

پیاده‌سازی سفارشی باید `noexcept` باشد و pointer خامی را برگرداند که واقعاً همان location مورد نظر pointer-like object را نشان می‌دهد. بازگرداندن آدرس نامعتبر یا آدرسی با معنای متفاوت، قرارداد مورد انتظار تابع را نقض می‌کند.

## ارتباط با iteratorها

### پشتیبانی از contiguous iterator

یکی از کاربردهای مهم `std::to_address` در ++C20، کار با iteratorهایی است که مفهوم `std::contiguous_iterator` را برآورده می‌کنند. این مفهوم از iteratorهای random-access محدودتر است و تضمین می‌کند عناصر بازه در حافظه به‌صورت پیوسته قرار گرفته‌اند.

نمونه‌های رایج contiguous iterator شامل pointer خام و iteratorهای `std::vector` در شرایط معمول هستند. در مقابل، iteratorهای `std::deque` contiguous نیستند؛ زیرا عناصر آن ممکن است در چند بلوک جداگانه ذخیره شوند.

```cpp
#include <iterator>
#include <memory>
#include <type_traits>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30};

    auto it = values.begin();

    static_assert(std::contiguous_iterator<decltype(it)>);

    int* raw = std::to_address(it);

    return raw[1] == 20 ? 0 : 1;
}
```

مفهوم `std::contiguous_iterator` علاوه بر random-access بودن، الزامات نوع reference و رابطهٔ آدرس عناصر متوالی را نیز مشخص می‌کند. این مفهوم تضمین می‌کند آدرس عنصر بعدی با arithmetic روی pointer متناظر سازگار باشد؛ همین تضمین است که تبدیل iterator به pointer را معتبر می‌سازد.

### چرا هر iterator را نمی‌توان به آدرس خام تبدیل کرد؟

قابلیت random access به معنی contiguous بودن نیست. یک iterator ممکن است از `it + n`، تفریق iteratorها و دسترسی با اندیس پشتیبانی کند، اما عناصرش در یک بلوک پیوسته از حافظه قرار نگرفته باشند.

برای مثال، iterator مربوط به `std::deque` معمولاً random-access است، ولی تبدیل آن به یک pointer خام و سپس فرض‌کردن اینکه `raw + n` به عنصر متناظر می‌رسد، صحیح نیست. در نتیجه، نباید `std::to_address` را برای هر iterator دلخواه به کار برد؛ کاربرد آن برای iteratorهای contiguous و pointer-like typeهای دارای قرارداد مناسب است.

### iterator انتهایی

در مدل contiguous iterator، `std::to_address` برای iteratorی که به موقعیت انتهایی بازه اشاره می‌کند نیز می‌تواند معنا داشته باشد، حتی اگر آن iterator قابل dereference نباشد. بااین‌حال، pointer حاصل از موقعیت انتهایی فقط برای نمایش مرز بازه و عملیات مجاز pointer arithmetic مناسب است و نباید dereference شود.

در طراحی الگوریتم‌ها، باید میان «آدرس شروع بازه»، «آدرس موقعیت انتهایی» و «آدرس یک شیء قابل دسترسی» تفاوت گذاشت.

## کاربرد در allocatorها و مدیریت حافظه

### استخراج آدرس برای ساخت شیء

یکی از انگیزه‌های مهم این API، ساده‌سازی کد allocator-generic است. نوع `allocator_traits<A>::pointer` ممکن است raw pointer یا fancy pointer باشد. کد عمومی برای دسترسی به storage خام می‌تواند از `std::to_address` استفاده کند.

```cpp
#include <memory>
#include <new>

template<class Alloc>
auto create_one(Alloc& alloc) {
    using traits = std::allocator_traits<Alloc>;
    using value_type = typename traits::value_type;

    auto p = traits::allocate(alloc, 1);
    value_type* raw = std::to_address(p);

    try {
        traits::construct(alloc, raw);
    } catch (...) {
        traits::deallocate(alloc, p, 1);
        throw;
    }

    return p;
}

template<class Alloc>
void destroy_one(Alloc& alloc,
                 typename std::allocator_traits<Alloc>::pointer p) {
    using traits = std::allocator_traits<Alloc>;
    auto* raw = std::to_address(p);

    traits::destroy(alloc, raw);
    traits::deallocate(alloc, p, 1);
}
```

در این الگو، handle یا pointer اصلی allocator برای `deallocate` حفظ می‌شود؛ آدرس خام صرفاً برای عملیاتی مانند ساخت و تخریب شیء استفاده می‌شود. این تفکیک مهم است، زیرا allocator سفارشی ممکن است برای آزادسازی به همان نوع pointer اختصاصی خود نیاز داشته باشد.

### تفاوت storage و lifetime شیء

گرفتن آدرس storage به معنی آغاز lifetime یک شیء نیست. `std::to_address` حافظه تخصیص نمی‌دهد، constructor اجرا نمی‌کند، شیء نمی‌سازد و lifetime را پایان نمی‌دهد.

برای ساخت شیء باید از مکانیزم مناسب مانند `allocator_traits::construct` یا ابزارهای lifetime و construction متناسب با طراحی استفاده کرد. برای پایان‌دادن به lifetime نیز باید destructor یا سازوکار متناظر اجرا شود. در نهایت، حافظه باید با همان allocator و قواعد تخصیص اولیه آزاد شود.

## تفاوت با ابزارهای مشابه

### مقایسه با `std::addressof`

تابع `std::addressof` آدرس یک object یا function را که از طریق reference در دسترس است به دست می‌آورد و از overload احتمالی `operator&` جلوگیری می‌کند. اما برای استفاده از آن روی pointer معمولاً باید ابتدا dereference انجام شود:

```cpp
auto* raw = std::addressof(*p);
```

در مقابل، `std::to_address(p)` مستقیماً آدرس را از pointer-like object استخراج می‌کند و نیازی به ساخت reference از طریق dereference ندارد. به همین دلیل برای storage تخصیص‌یافته اما هنوز بدون object ساخته‌شده مناسب است.

### مقایسه با `operator->`

عملگر `operator->` بخشی از رابط یک کلاس است و می‌تواند به pointer خام یا یک proxy pointer-like type دیگر برسد. فراخوانی مستقیم آن، کد را به جزئیات نوع اشاره‌گر وابسته می‌کند.

تابع `std::to_address` یک لایهٔ استاندارد بالاتر فراهم می‌کند: ابتدا customization مربوط به `pointer_traits` را بررسی می‌کند و سپس در صورت نبود آن از زنجیرهٔ `operator->` استفاده می‌کند. بنابراین برای کد generic، معمولاً انتخاب مناسب‌تری است.

### مقایسه با `std::pointer_traits<Ptr>::pointer_to`

عضو `pointer_to` در `std::pointer_traits` از یک reference به شیء، pointer-like object متناظر را می‌سازد. عضو اختیاری `to_address` جهت معکوس این مسیر را برای انواعی که آن را پیاده‌سازی می‌کنند فراهم می‌کند.

این دو تابع الزاماً در همهٔ انواع pointer-like به‌صورت یک جفت کامل وجود ندارند؛ `to_address` عضو اختیاری specialization است و نباید فرض کرد هر نوعی که `pointer_to` دارد، حتماً `to_address` سفارشی نیز دارد.

## ویژگی‌های مهم و محدودیت‌ها

### تضمین `noexcept`

هر دو overload استاندارد `std::to_address` با `noexcept` تعریف شده‌اند. در نتیجه، عملیات استخراج آدرس از دید رابط استاندارد نباید exception را به فراخواننده منتقل کند. اگر customization سفارشی درون خود عملیات بالقوه throwing داشته باشد، چنین طراحی‌ای با قرارداد `noexcept` سازگار نیست و در صورت خروج exception از تابع `noexcept`، برنامه با `std::terminate` مواجه می‌شود.

### محدودیت function pointer

overload مربوط به raw pointer برای function type مجاز نیست. بنابراین `std::to_address` ابزار عمومی برای تبدیل هر نوع pointer به آدرس نیست و نباید آن را برای function pointerها به کار برد.

### نیاز به specialization معتبر

در مسیر fancy pointer، اگر instantiation مربوط به `std::pointer_traits<Ptr>` خودش نامعتبر باشد، ممکن است برنامه با خطای سخت زمان کامپایل مواجه شود. این وضعیت لزوماً در context فوری یک requires-expression قابل تبدیل به نتیجهٔ سادهٔ false نیست؛ بنابراین نباید فرض کرد هر نوع دلخواهی را می‌توان بدون داشتن ساختار pointer-like مناسب به `std::to_address` داد.

### آدرس خام، مالکیت را منتقل نمی‌کند

pointer خام حاصل از `std::to_address` مالکیت حافظه یا object را به دست نمی‌آورد. اعتبار آن به lifetime، وضعیت تخصیص، mapping و قواعد invalidation مربوط به pointer-like object اصلی وابسته است.

اگر container یا allocator حافظه را جابه‌جا کند، آن را آزاد کند یا mapping مربوط به آن تغییر کند، pointer خام قبلی ممکن است نامعتبر شود. بنابراین نباید آن را بیش از مدت اعتبار pointer اصلی نگه داشت، مگر آنکه قرارداد نوع مربوطه چنین کاری را تضمین کند.

### تابع هیچ اعتبارسنجی عمومی انجام نمی‌دهد

این تابع بررسی نمی‌کند که pointer به حافظهٔ زنده اشاره می‌کند، dereference آن مجاز است، محدودهٔ دسترسی معتبر است یا object مورد انتظار ساخته شده است. مسئولیت این تضمین‌ها بر عهدهٔ نوع pointer، allocator و کد فراخواننده است.

## خطاهای رایج در استفاده

- **اشتباه گرفتن random-access با contiguous:** پشتیبانی از جمع و تفریق iteratorها تضمین نمی‌کند که عناصر در حافظه پیوسته باشند.
- **استفاده از pointer خام برای deallocate:** اگر allocator نوع pointer اختصاصی دارد، باید همان pointer اصلی را به `deallocate` بدهید؛ raw pointer حاصل از `std::to_address` جایگزین عمومی آن نیست.
- **فرض ساخته‌شدن object پس از allocate:** تخصیص storage به‌تنهایی به معنی ساخت object نیست.
- **نگهداری طولانی‌مدت raw pointer:** pointer خام ممکن است با آزادسازی، جابه‌جایی یا تغییر mapping نامعتبر شود.
- **استفاده برای function pointer:** این کاربرد با الزامات overload خام سازگار نیست.
- **نوشتن customization ناسازگار:** تابع سفارشی باید آدرس متناظر با pointer را برگرداند و قرارداد `noexcept` را رعایت کند.

## نکات طراحی برای نویسندگان کتابخانه

### طراحی fancy pointer

اگر در حال طراحی fancy pointer هستید، ابتدا مشخص کنید آیا `operator->` می‌تواند به‌صورت صحیح و پایدار آدرس خام را ارائه کند یا نوع شما به customization اختصاصی نیاز دارد. برای pointerهایی مانند offset pointer که آدرس وابسته به base address یا mapping جاری است، تبدیل باید در زمان فراخوانی و با توجه به وضعیت معتبر حافظه انجام شود.

### حفظ نوع pointer اصلی

در APIهای allocator-aware، نوع pointer اصلی را برای عملیات مالکیت و آزادسازی حفظ کنید. raw pointer را تنها در مرزهایی استخراج کنید که API پایین‌دستی واقعاً به آدرس خام نیاز دارد؛ مانند placement construction یا تعامل با یک API مبتنی بر `T*`.

### محدودکردن وابستگی به آدرس خام

در طراحی‌های قابل‌حمل، تا جای ممکن abstraction اصلی pointer را حفظ کنید. تبدیل زودهنگام به raw pointer می‌تواند مزیت‌هایی مانند relocatability، shared-memory addressing یا کنترل دسترسی را از بین ببرد.

### استفاده از concept مناسب

اگر یک template واقعاً به حافظهٔ پیوسته نیاز دارد، constraint آن را با `std::contiguous_iterator` بیان کنید، نه صرفاً `std::random_access_iterator`. این کار نیاز الگوریتم را شفاف می‌کند و از پذیرش iteratorهایی جلوگیری می‌کند که تنها قابلیت دسترسی تصادفی دارند.

## نسخه‌های استاندارد و سازگاری

تابع `std::to_address` از ++C20 بخشی از کتابخانهٔ استاندارد است. قالب `std::pointer_traits` از ++C11 وجود داشته، اما عضو اختیاری `pointer_traits::to_address` نیز از ++C20 معرفی شده است. مفهوم `std::contiguous_iterator` هم متعلق به ++C20 است.

برای تشخیص پشتیبانی کتابخانهٔ استاندارد می‌توان از feature-test macro زیر استفاده کرد:

```cpp
#include <memory>

#if defined(__cpp_lib_to_address) && __cpp_lib_to_address >= 201711L
// std::to_address is available.
#endif
```

مقدار feature-test macro مربوط به این قابلیت `201711L` است. وجود این macro به پشتیبانی کتابخانهٔ استاندارد مربوط است و نباید آن را با صرف پشتیبانی frontend کامپایلر از زبان ++C20 یکسان دانست؛ نسخهٔ کتابخانهٔ استاندارد مورد استفاده نیز اهمیت دارد.

در پروژه‌هایی که باید با استانداردهای پیش از ++C20 ساخته شوند، `std::to_address` در دسترس نیست. در چنین پروژه‌هایی می‌توان برای مجموعهٔ محدود و مشخصی از pointer typeها helper سازگار با نسخه‌های قدیمی‌تر نوشت، اما پیاده‌سازی عمومی و معادل آن ساده نیست؛ به‌ویژه اگر fancy pointerهای سفارشی یا iteratorهای متفاوت در میان باشند. بهتر است از helper اختصاصی و مستندشده استفاده شود و از فرض‌کردن اینکه هر iterator دارای `operator->` الزاماً contiguous است، پرهیز شود.

## نمونهٔ کاربردی کامل

نمونهٔ زیر یک تابع generic را نشان می‌دهد که یک allocator را می‌پذیرد، یک واحد storage رزرو می‌کند، آدرس خام آن را می‌گیرد و object را با سازوکار allocator می‌سازد. در صورت شکست construction، storage آزاد می‌شود.

```cpp
#include <memory>
#include <utility>

template<class Alloc, class... Args>
auto allocate_and_construct(Alloc& alloc, Args&&... args) {
    using traits = std::allocator_traits<Alloc>;
    using pointer = typename traits::pointer;
    using value_type = typename traits::value_type;

    pointer p = traits::allocate(alloc, 1);
    value_type* raw = std::to_address(p);

    try {
        traits::construct(alloc, raw, std::forward<Args>(args)...);
    } catch (...) {
        traits::deallocate(alloc, p, 1);
        throw;
    }

    return p;
}

template<class Alloc>
void destroy_and_deallocate(
    Alloc& alloc,
    typename std::allocator_traits<Alloc>::pointer p) {

    using traits = std::allocator_traits<Alloc>;
    using value_type = typename traits::value_type;

    value_type* raw = std::to_address(p);

    traits::destroy(alloc, raw);
    traits::deallocate(alloc, p, 1);
}
```

این نمونه یک الگوی آموزشی برای یک object است. در کد production باید علاوه بر آن، رفتار allocator در صورت خطا، تعداد عناصر، alignment، قواعد ownership، و نیازهای exception-safety کل container یا abstraction بررسی شود.

## جمع‌بندی

تابع `std::to_address` در ++C20 راه استانداردی برای تبدیل raw pointer یا pointer-like type به آدرس خام متناظر فراهم می‌کند. این تابع بدون dereference کردن pointer، آدرس را استخراج می‌کند و به همین دلیل در کد allocator-generic و موقعیت‌هایی که storage هنوز object زنده‌ای در خود ندارد، اهمیت ویژه‌ای دارد.

برای fancy pointerها، ابتدا customization اختیاری `std::pointer_traits<Ptr>::to_address` بررسی می‌شود و در صورت نبود آن، زنجیرهٔ `operator->` دنبال می‌شود. همچنین این تابع برای iteratorهایی که واقعاً مفهوم `std::contiguous_iterator` را مدل می‌کنند قابل استفاده است؛ اما random-access بودن به‌تنهایی کافی نیست.

قاعدهٔ عملی این است: هر زمان یک API واقعاً به raw pointer نیاز دارد و ورودی ممکن است pointer-like type استاندارد یا سفارشی باشد، `std::to_address` را به‌عنوان نقطهٔ تبدیل استاندارد در نظر بگیرید. در عین حال، مالکیت، lifetime، اعتبار حافظه و نوع pointer مورد نیاز برای deallocation را جداگانه مدیریت کنید.

## منابع و مراجع

- [cppreference: `std::to_address`](https://en.cppreference.com/w/cpp/memory/to_address)
- [cppreference: `std::pointer_traits`](https://en.cppreference.com/w/cpp/memory/pointer_traits)
- [cppreference: `std::pointer_traits::to_address`](https://en.cppreference.com/w/cpp/memory/pointer_traits/to_address)
- [Working Draft: Pointer conversion](https://eel.is/c++draft/pointer.conversion)
- [Working Draft: Pointer traits](https://eel.is/c++draft/pointer.traits)
- [Working Draft: `contiguous_iterator`](https://eel.is/c++draft/iterator.concept.contiguous)
- [WG21 P3349R1: Converting contiguous iterators to pointers](https://www.open-std.org/JTC1/SC22/WG21/docs/papers/2025/p3349r1.html)


---

## 🤝 مشارکت ها

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>