<div align="right">

[🇺🇸 English](../../en/cpp11/unique_ptr.md) | [🇮🇷 فارسی](./unique_ptr.md)

</div>

----

# Complete Guide to `std::unique_ptr` in C++11: Unique Ownership and Smart Memory Management

## Table of Contents

- [Introduction](#Introduction)
- [What Was the Problem Before C++11?](#مشکل-قبل-از-c11-چه-بود)
- [Exceptions Made the Problem Worse](#isثناها-مشکل-را-جدیتر-میکردند)
- [What Is the Solution?](#راهحل-چیست)
- [What Is unique_ptr?](#unique_ptr-چیست)
- [Why Is It Called unique?](#چرا-نام-آن-unique-is)
- [Transferring Ownership with move](#انتقال-مالکیت-با-move)
- [Why Do We Have move but Not copy?](#چرا-copy-نداریم-ولی-move-داریم)
- [What Is reset?](#reset-چیست)
- [reset with a New Object](#reset-با-یک-شیء-جدید)
- [Can We Empty a unique_ptr?](#آیا-میتوانیم-uniqueptr-را-خالی-کنیم)
- [Using unique_ptr in a Setter](#isفاده-از-uniqueptr-در-setter)
- [Why Can Taking unique_ptr by Value Be a Good Design?](#چرا-گرفتن-uniqueptr-با-Value-میتواند-طراحی-خوبی-باشد)
- [How Should a getter for unique_ptr Be Designed?](#getter-برای-uniqueptr-چگونه-باید-باشد)
- [What If We Only Want to Access the Object?](#اگر-فقط-میخواهیم-به-شیء-دسترسی-بدهیم-چه-کنیم)
- [A Better getter for an Object That Always Exists](#getter-بهتر-برای-شیئی-که-همیشه-وجود-دارد)
- [getter for Read-Only Access](#getter-برای-دسترسی-فقط-خواندنی)
- [Is Returning a raw pointer from a getter Dangerous?](#آیا-برگرداندن-raw-pointer-از-getter-خطرناک-is)
- [Does get Cause a Memory Leak?](#آیا-get-باعث-memory-leak-میشود)
- [How Can release Cause a Memory Leak?](#release-چگونه-میتواند-باعث-memory-leak-شود)
- [Do Not Confuse get and release](#get-و-release-را-با-هم-اشتباه-نگیریم)
- [Passing unique_ptr to a Function](#انتقال-uniqueptr-به-یک-تابع)
- [What If the Function Only Wants to Use the Object?](#اگر-تابع-فقط-میخواهد-از-شیء-isفاده-کند-چه-کنیم)
- [Using unique_ptr in a Class](#isفاده-از-uniqueptr-در-یک-کلاس)
- [unique_ptr and destructor](#uniqueptr-و-destructor)
- [Transferring Ownership Between Classes](#انتقال-مالکیت-بین-کلاسها)
- [Can unique_ptr Be Returned from a getter?](#آیا-میتوان-uniqueptr-را-از-getter-برگرداند)
- [Choosing the Right getter Based on the Requirement](#getter-مناسب-بر-اساس-نیاز)
- [Why Is It Better Not to Expose unique_ptr from a Class?](#چرا-بهتر-is-uniqueptr-را-از-کلاس-بیرون-ندهیم)
- [unique_ptr and const](#uniqueptr-و-const)
- [unique_ptr and polymorphism](#uniqueptr-و-polymorphism)
- [unique_ptr for Arrays](#uniqueptr-برای-آرایه)
- [Does unique_ptr Itself Consume a Lot of Memory?](#آیا-uniqueptr-itself-حافظه-زیادی-مصرف-میکند)
- [What Is a custom deleter?](#custom-deleter-چیست)
- [One Very Important Point: make_unique](#یک-نکته-بسیار-مهم-make_unique)
- [A Common Mistake: Creating Multiple unique_ptrs from One raw Pointer](#یک-اشتباه-رایج-ساختن-چند-uniqueptr-از-یک-raw-pointer)
- [Common Mistake: Deleting the Result of get](#اشتباه-رایج-delete-کردن-نتیجه-get)
- [Common Mistake: Keeping a raw Pointer for Too Long](#اشتباه-رایج-نگه-داشتن-raw-pointer-برای-مدت-طولانی)
- [When Should We Use unique_ptr?](#چه-زمانی-از-uniqueptr-isفاده-کنیم)
- [Difference Between unique_ptr and shared_ptr](#تفاوت-uniqueptr-و-sharedptr)
- [An Important Rule in Modern C++ Design](#قانون-مهم-در-طراحی-مدرن-c)
- [A Complete Example](#یک-نمونه-کامل)
- [Final Summary](#جمعبندی-نهایی)
- [Contributions](#مشارکت-ها)

---

## Introduction

The `std::unique_ptr` smart pointer is one of the most important memory-management tools in modern C++, introduced with the C++11 standard.

`unique_ptr` is designed for situations where an object in dynamic memory has **exactly one owner**.

The core behavior of this smart pointer is simple: as long as the `unique_ptr` is alive, it owns the target object. When the `unique_ptr` is destroyed, it automatically releases and destroys the object it owns.

---

## What Was the Problem Before C++11?

Memory management in older C++ code was commonly done using `new` and `delete`.

عبارت زیر نمونهٔ ساده‌ای از این روش is:

```cpp
MyClass* ptr = new MyClass();

// use ptr

delete ptr;
```

The main problem with this approach is that the programmer is fully responsible for releasing the memory.

عبارت زیر اگر فراموش شود، باعث نشت حافظه می‌شود:

```cpp
MyClass* ptr = new MyClass();

// use ptr

// delete ptr; has been forgotten
```

Another problem arises when a function has multiple exit paths.

عبارت زیر را در نظر بگیرید:

```cpp
void process()
{
    MyClass* ptr = new MyClass();

    if (someCondition())
        return;

    doSomething();

    delete ptr;
}
```

The problem in this example is that if `someCondition()` returns `true`, the function exits before reaching `delete`.

As a result, the memory allocated with `new` is not released, causing a memory leak.

---

## Exceptions Made the Problem Worse

Another important problem is the presence of exceptions.

عبارت زیر را ببینید:

```cpp
void process()
{
    MyClass* ptr = new MyClass();

    doSomething(); // may throw an exception

    delete ptr;
}
```

The problem is that if `doSomething()` throws an exception, normal function execution is interrupted and `delete ptr` may never be executed.

The result can be a memory leak.

---

# What Is the Solution?

RAII is one of the most important principles in C++.

RAII stands for `Resource Acquisition Is Initialization`.

The idea behind RAII is to tie resource ownership to the lifetime of an object.

Simply put:

> When the object that owns a resource is created, it acquires the resource; when the object is destroyed, it releases the resource.

اشاره گر `std::unique_ptr` دقیقاً بر همین اساس کار می‌کند.

---

# اشاره گر What Is unique_ptr?

`std::unique_ptr<T>` is a smart pointer that owns an object of type `T`.

عبارت زیر یک `unique_ptr` ایجاد می‌کند:

```cpp
#include <memory>

std::unique_ptr<MyClass> ptr(new MyClass());
```

However, the recommended approach in C++14 and later is to use `std::make_unique`:

```cpp
auto ptr = std::make_unique<MyClass>();
```

An important point is that `unique_ptr` is responsible for deleting the object itself.

عبارت زیر causes زمانی که `ptr` از scope خارج شود، شیء نیز آزاد شود:

```cpp
void process()
{
    auto ptr = std::make_unique<MyClass>();

    // use ptr
}
```

The point of this code is that we no longer need to write `delete` manually.

---

# Why Is It Called unique?

The word `unique` refers to unique ownership. The following is not allowed:

```cpp
auto ptr1 = std::make_unique<MyClass>();

auto ptr2 = ptr1; // خطا
```

The reason is that if two `unique_ptr` objects owned the same object, it would be unclear which one should destroy it.

---

# Transferring Ownership with move

A `unique_ptr` cannot be copied, but it can be moved.

عبارت زیر مالکیت را از `ptr1` به `ptr2` منتقل می‌کند:

```cpp
auto ptr1 = std::make_unique<MyClass>();

auto ptr2 = std::move(ptr1);
```

The important point is that after this operation, `ptr2` owns the object.

`ptr1` is no longer the owner of the object and will normally contain `nullptr`.

عبارت زیر برای بررسی آن مناسب is:

```cpp
if (ptr1 == nullptr)
{
    // ptr1 no longer owns the object
}
```

---

# Why Do We Have move but Not copy?

Copying means that after the copy, both objects remain independent and valid.

عبارت زیر برای `unique_ptr` معنی درستی ندارد و باعث خطای کامپایلر می شود:

```cpp
auto ptr2 = ptr1;
```

متد move متفاوت is؛ در move مالکیت از یک شیء به شیء دیگر منتقل می‌شود.

عبارت isاندارد آن چنین is:

```cpp
auto ptr2 = std::move(ptr1);
```

---

# متد What Is reset?

The `reset()` method is used to replace ownership or release the currently owned object.

عبارت زیر شیء فعلی را آزاد می‌کند:

```cpp
ptr.reset();
```

This operation means that if `ptr` owns an object, that object is destroyed and `ptr` becomes empty.

عبارت بررسی وضعیت آن چنین is:

```cpp
if (!ptr)
{
    // ptr is empty 
}
```

---

# عملیات reset with a New Object

`reset` can also make the `unique_ptr` take ownership of a new object.

عبارت نمونه چنین is:

```cpp
ptr.reset(new MyClass());
```

This code means that if `ptr` already owns another object, the old object is released first and then `ptr` takes ownership of the new object.

In modern C++, it is better to use `make_unique` whenever possible.

عبارت مناسب‌تر برای ایجاد اولیه چنین is:

```cpp
auto ptr = std::make_unique<MyClass>();
```

---

# Can We Empty a unique_ptr?

A `unique_ptr` can be empty and own no object at any time.

عبارت زیر یک `unique_ptr` خالی ایجاد می‌کند:

```cpp
std::unique_ptr<MyClass> ptr;
```

بررسی خالی بودن آن نیز ساده is:

```cpp
if (!ptr)
{
    // ptr خالی است
}
```

---

# Using unique_ptr in a Setter

A setter that accepts a `unique_ptr` should be designed around ownership semantics.

A common and explicit way to express ownership transfer is to accept `std::unique_ptr&&`, but the preferred setter design here is to take the `unique_ptr` by value, as follows:

```cpp
class Owner
{
private:
    std::unique_ptr<MyClass> value;

public:
    void setValue(std::unique_ptr<MyClass> ptr)
    {
        value = std::move(ptr);
    }
};
```

This design means that the caller must explicitly transfer ownership:

```cpp
auto obj = std::make_unique<MyClass>();

owner.setValue(std::move(obj));
```

The important point is that after this transfer, `owner` owns the object and `obj` no longer does.

---

# Why Can Taking unique_ptr by Value Be a Good Design?

Taking a `unique_ptr` by value in a setter has the advantage of clearly communicating that:

> the function receives ownership of an object.

پس یک Setter درست به این شکل is:

```cpp
void setValue(std::unique_ptr<MyClass> ptr)
{
    value = std::move(ptr);
}
```

Another benefit is that if the caller wants to transfer ownership, it must explicitly use `std::move`.

بنابراین عبارت زیر کاملاً گویis:

```cpp
owner.setValue(std::move(obj));
```

---

# عملیات How Should a getter for unique_ptr Be Designed?

A getter is an important part of class design.

The important point is that we should not expose the `unique_ptr` itself simply because the class member happens to be a `unique_ptr`.

عبارت زیر معمولاً انتخاب مناسبی is not:

```cpp
std::unique_ptr<MyClass>& getValue()
{
    return value;
}
```

The problem with this design is that the caller can modify the class's internal ownership.

For example, the following could happen:

```cpp
owner.getValue().reset();
```

This operation allows the caller to destroy the object owned by the class.

---

# What If We Only Want to Access the Object?

A better approach is to **keep ownership inside the class and expose access to the object only.**

عبارت زیر برای یک عضو nullable انتخاب مناسبی is:

```cpp
MyClass* getValue()
{
    return value.get();
}
```

`get()` can return the raw pointer to the object, but it does not transfer ownership.

عملکرد caller می‌تواند چنین باشد:

```cpp
MyClass* ptr = owner.getValue();

if (ptr)
{
    ptr->doSomething();
}
```

مهم این is که caller مالک `ptr` is not و نباید آن را با `delete` آزاد کند.

---

# متد A Better getter for an Object That Always Exists

If the class guarantees that the object always exists, returning a reference can be a better design. For example:

```cpp
MyClass& getValue()
{
    return *value;
}
```

مفهوم isفاده از آن نیز ساده is:

```cpp
owner.getValue().doSomething();
```

The advantage is that the caller does not have to deal with `nullptr`.

The important requirement is that `value` must really always be valid.

---

# متد getter for Read-Only Access

If we do not want the caller to modify the internal object, `const` is an appropriate choice.

عبارت نمونه چنین is:

```cpp
const MyClass& getValue() const
{
    return *value;
}
```

This design allows the caller to read the object but not modify it.

---

# Is Returning a raw pointer from a getter Dangerous?

A raw pointer is not inherently a problem.

The problem occurs when the caller assumes that the raw pointer owns the object. عبارت خطرناک زیر نباید انجام شود:

```cpp
MyClass* ptr = owner.getValue();

delete ptr;
```

Here, `ptr` does not own the object, while the `unique_ptr` still considers itself the owner.

The result can be a `double delete`—deleting the heap object twice—and therefore undefined behavior.

---

# خطر مهم‌تر raw pointer؛ dangling pointer

Another danger occurs when we keep a raw pointer after the original owner has destroyed the object. عبارت نمونه:

```cpp
MyClass* ptr = owner.getValue();

owner.reset();

ptr->doSomething(); // خطرناک
```

After `reset`, the object has most likely been destroyed, and `ptr` now points to memory that is no longer associated with that object. Such a pointer is called a `dangling pointer`.

---

# Does get Cause a Memory Leak?

The important point is that `get()` itself does not cause a memory leak. It only gives the caller a non-owning raw pointer:

```cpp
MyClass* ptr = value.get();
```

A memory leak usually occurs when ownership is improperly detached from the `unique_ptr`.

```cpp
MyClass* ptr = value.release();
```

`release()` and `get()` are fundamentally different. `get()` means:

> Give me only the address; keep ownership.

`release()` means:

> Release ownership and give me the raw pointer.

---

# متد How Can release Cause a Memory Leak?

After `release()`, the `unique_ptr` no longer owns the object. Therefore, the following is dangerous:

```cpp
auto ptr = std::make_unique<MyClass>();

MyClass* raw = ptr.release();

// Raw now

// delete raw has been forgotten
```

In this example, the `unique_ptr` no longer owns the object and `delete raw` was never called, so the memory leaks.

If you must use `release()`, you must clearly establish the new ownership responsibility:

```cpp
MyClass* raw = ptr.release();

delete raw;
```

An important recommendation is to use `release()` only when you genuinely need to transfer ownership to a legacy API or another system.

---

# متدهای Do Not Confuse get and release

The difference between these two functions is very important.

| تابع        |           مالکیت منتقل می‌شود؟ | raw pointer برمی‌گرداند؟ |
| ----------- | -----------------------------: | -----------------------: |
| `get()`     |                            No |                      Yes |
| `release()` | Yes، از unique_ptr خارج می‌شود |                      Yes |
| `reset()`   |        مالکیت قبلی آزاد می‌شود |                      No |

خلاصه شرح این موارد:

```cpp
ptr.get();      // فقط مشاهده
ptr.release();  // رها کردن مالکیت
ptr.reset();    // حذف شیء فعلی
```

---

# Passing unique_ptr to a Function

If a function needs to take ownership of an object, we move the `unique_ptr` into it.

عبارت نمونه:

```cpp
void process(std::unique_ptr<MyClass> obj)
{
    obj->doSomething();
}
```

فراخوانی آن چنین is:

```cpp
auto obj = std::make_unique<MyClass>();

process(std::move(obj));
```

After this call, the function owns the object.

---

# What If the Function Only Wants to Use the Object?

If a function does not take ownership, it is usually better not to pass a `unique_ptr` to it at all.

عبارت مناسب برای شیئی که باید وجود داشته باشد:

```cpp
void process(const MyClass& obj)
{
    obj.doSomething();
}
```

فراخوانی آن با `unique_ptr` نیز ساده is:

```cpp
process(*obj);
```

اگر وجود شیء اختیاری is، می‌توان raw pointer غیرمالک دریافت کرد:

```cpp
void process(const MyClass* obj)
{
    if (obj)
    {
        obj->doSomething();
    }
}
```

This design makes the function API explicit about whether it takes ownership or merely uses the object.

---

# Using unique_ptr in a Class

One of the most common uses of `unique_ptr` is storing it as a private class member.

عبارت نمونه:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;

public:
    Car()
        : engine(std::make_unique<Engine>())
    {
    }
};
```

In this example, `Car` owns the `Engine`.

When a `Car` is destroyed, the `Engine` is also released automatically.

---

# ارتباط unique_ptr and destructor

One important advantage of this design is that we usually do not need a custom destructor.

عبارت زیر کافی is:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;
};
```

When `Car` is destroyed, the `unique_ptr` destructor runs and releases the `Engine`.

عبارت این اصل را نشان می‌دهد:

```text
Car
 └── unique_ptr
      └── Engine
```

مالکیت در این ساختار کاملاً مشخص is.

---

# Transferring Ownership Between Classes

One advantage of `unique_ptr` is that ownership can be transferred between objects.

عبارت نمونه:

```cpp
class A
{
public:
    std::unique_ptr<MyClass> create()
    {
        return std::make_unique<MyClass>();
    }
};
```

The caller can take ownership of the result:

```cpp
std::unique_ptr<MyClass> obj = a.create();
```

Here, ownership transfer is explicit and no manual `delete` is required.

---

# Can unique_ptr Be Returned from a getter?

Yes, but note that this normally means **transferring ownership**.

عبارت نمونه:

```cpp
std::unique_ptr<MyClass> takeValue()
{
    return std::move(value);
}
```

The name `takeValue` is intentionally chosen to indicate that the caller receives ownership.

عبارت isفاده چنین is:

```cpp
auto obj = owner.takeValue();
```

After this operation, the class is no longer the owner of the object.

---

# متد Choosing the Right getter Based on the Requirement

We can use a simple rule for designing getters:

```text
فقط خواندن:
const T&

خواندن و تغییر بدون انتقال مالکیت:
T&

دسترسی اختیاری:
T*

دسترسی اختیاری و فقط خواندنی:
const T*

انتقال مالکیت:
std::unique_ptr<T>

unique_ptr دسترسی مستقیم به خود:
std::unique_ptr<T>&
```

The final choice should be based on the API contract, not merely on the type of the internal member.

---

# Why Is It Better Not to Expose unique_ptr from a Class?

If a class owns a resource, it is generally better to keep ownership inside that class.

عبارت نامناسب چنین چیزی is:

```cpp
std::unique_ptr<MyClass>& getValue();
```

This API exposes the class's internal memory-management details to its users.

عبارت طراحی بهتر این is:

```cpp
MyClass* getValue();
```

یا اگر nullable is not:

```cpp
MyClass& getValue();
```

در این حالت کلاس می‌گوید:

> من the owner of the object هستم، اما اجازه می‌دهم از آن isفاده کنی.

---

# همراهی unique_ptr and const

کلمه کلیدی `const` را می‌توان در چند سطح مختلف isفاده کرد.

عبارت زیر یعنی خود شیء قابل تغییر is not:

```cpp
std::unique_ptr<const MyClass> ptr;
```

عبارت زیر با مورد بالا متفاوت is:

```cpp
const std::unique_ptr<MyClass> ptr;
```

در حالت دوم خود `unique_ptr` قابل تغییر یا انتقال is not، اما شیء `MyClass` می‌تواند قابل تغییر باشد.

This distinction is an important C++ concept and the two should not be confused.

---

# همراهی unique_ptr and polymorphism

`unique_ptr` is also well suited for storing polymorphic objects.

عبارت نمونه:

```cpp
class Base
{
public:
    virtual ~Base() = default;
};

class Derived : public Base
{
};
```

حالا می‌توانیم بنویسیم:

```cpp
std::unique_ptr<Base> obj =
    std::make_unique<Derived>();
```

A very important point is that if a derived object is going to be destroyed through `Base`, the base-class destructor must be `virtual`.

عبارت مناسب چنین is:

```cpp
virtual ~Base() = default;
```

---

# اشاره گر unique_ptr for Arrays

`unique_ptr` also has a specialization for arrays.

عبارت نمونه:

```cpp
std::unique_ptr<int[]> values =
    std::make_unique<int[]>(10);
```

اشاره گر `unique_ptr` هنگام نابودی، آرایه را به شکل مناسب آزاد می‌کند.

عبارت دسترسی به عنصر نیز مانند آرایه معمولی is:

```cpp
values[0] = 10;
values[1] = 20;
```

In modern code, `std::array` is usually preferable for fixed-size arrays and `std::vector` for variable-size collections, unless this specific form of dynamic ownership is actually needed.

---

# Does unique_ptr Itself Consume a Lot of Memory?

Normally, `unique_ptr<T>` is conceptually about the size of a raw pointer.

The important point is that `unique_ptr` is a small object that manages ownership and responsibility for releasing the resource.

If a custom deleter is used, the size of `unique_ptr` may be different.

---

# What Is a custom deleter?

`unique_ptr` is not limited to deleting objects created with `new`.

می‌توانیم مشخص کنیم منبع چگونه آزاد شود:

```cpp
struct FileDeleter
{
    void operator()(FILE* file) const
    {
        if (file)
            fclose(file);
    }
};

using FilePtr = std::unique_ptr<FILE, FileDeleter>;
```

This capability also makes `unique_ptr` useful for resources other than memory, such as file handles or resources that must be released with a specific function. Here, we use a functor to define how the file should be released.

---

# One Very Important Point: make_unique

`std::make_unique` was added in C++14.

عبارت پیشنهادی چنین is:

```cpp
auto ptr = std::make_unique<MyClass>();
```

The advantage is that this approach is simpler and is the preferred way to create a `unique_ptr` in modern C++.

If the project really targets C++11, `make_unique` is not available in the standard library. You can use `std::unique_ptr<T>(new T(...))` or define an appropriate C++11 helper.

---

# A Common Mistake: Creating Multiple unique_ptrs from One raw Pointer

We must not give the same raw pointer to multiple `unique_ptr` objects.

عبارت بسیار خطرناک:

```cpp
MyClass* raw = new MyClass();

std::unique_ptr<MyClass> p1(raw);
std::unique_ptr<MyClass> p2(raw);
```

There are now two independent owners, both of which believe they should eventually delete the same object.

The result can be a `double delete` and undefined behavior.

---

# Common Mistake: Deleting the Result of get

The pointer returned by `get()` is not owned by the caller.

عبارت زیر غلط is:

```cpp
auto ptr = std::make_unique<MyClass>();

MyClass* raw = ptr.get();

delete raw;
```

`ptr` still owns the object and will later attempt to delete it again.

---

# Common Mistake: Keeping a raw Pointer for Too Long

A raw pointer obtained from `get()` is valid only while the object owned by the `unique_ptr` still exists.

عبارت خطرناک:

```cpp
MyClass* raw = ptr.get();

ptr.reset();

// raw is no longer valid
```

Therefore, if the raw pointer outlives its owner, it can become a `dangling pointer`.

---

# When Should We Use unique_ptr?

When an object has exactly one owner, `unique_ptr` is usually an excellent choice.

عبارت‌های رایج isفاده عبارت‌اند از:

```text
مالکیت یک عضو داخل کلاس
factory ایجاد شیء در 
انتقال مالکیت بین توابع
مدیریت منابع پویا
custom deleter مدیریت منابع با 
polymorphism پیاده‌سازی 
```

If multiple independent owners are required, we should consider `std::shared_ptr`.

If shared ownership is not actually required, using `shared_ptr` merely for convenience is usually not a good choice.

---

# Difference Between unique_ptr and shared_ptr

تفاوت اصلی این دو در مدل مالکیت is.

| ویژگی         | `unique_ptr` | `shared_ptr` |
| ------------- | ------------ | ------------ |
| مالکیت        | Unique         | Shared        |
| Copy          | No          | Yes          |
| Move          | Yes          | Yes          |
| هزینه مدیریتی | Low           | Higher        |
| کنترل مالکیت  | Simple and clear  | More complex    |
| کاربرد اصلی   | One owner      | Multiple owners     |

یک قاعده ساده این is:

> اگر نمی‌دانید از کدام isفاده کنید، ابتدا بررسی کنید آیا واقعاً به مالکیت Shared نیاز دارید یا No.

---

# An Important Rule in Modern C++ Design

One very useful rule is:

> Express ownership with a smart pointer, and express non-owning access with a reference or raw pointer.

عبارت نمونه:

```cpp
std::unique_ptr<Engine> engine;
```

This member means that `Car` owns the `Engine`.

عبارت زیر:

```cpp
Engine* getEngine();
```

This getter means that the caller can access the `Engine` but does not own it.

عبارت زیر:

```cpp
std::unique_ptr<Engine> takeEngine();
```

This API means that the caller takes ownership.

---

# A Complete Example

یک کلاس واقعی می‌تواند به شکل زیر طراحی شود:

```cpp
class Car
{
private:
    std::unique_ptr<Engine> engine;

public:
    Car()
        : engine(std::make_unique<Engine>())
    {
    }

    Engine* getEngine()
    {
        return engine.get();
    }

    const Engine* getEngine() const
    {
        return engine.get();
    }

    void setEngine(std::unique_ptr<Engine> newEngine)
    {
        engine = std::move(newEngine);
    }

    std::unique_ptr<Engine> takeEngine()
    {
        return std::move(engine);
    }

    void resetEngine()
    {
        engine.reset();
    }
};
```

This design provides several clearly defined ownership behaviors.

`getEngine()` only provides access.

`setEngine()` receives new ownership.

`takeEngine()` transfers ownership.

`resetEngine()` releases the resource.

As a result, the class API makes its ownership semantics very clear.

---

# Final Summary

`std::unique_ptr` is a way to express **unique ownership of a resource**.

The important point is that `unique_ptr` is not merely a raw pointer with a different name.

Its core concept is **managing resource ownership and lifetime**.

The key operations to remember are:

```cpp
auto ptr = std::make_unique<T>();
```

The statement above creates an object and gives ownership to `ptr`.

```cpp
ptr.get();
```

The statement above retrieves only a non-owning raw pointer.

```cpp
ptr.reset();
```

The statement above releases the current object and makes the `unique_ptr` empty.

```cpp
ptr.reset(new T());
```

The statement above releases the current object and makes the `unique_ptr` the owner of a new object; however, `make_unique` is preferred for initial construction.

```cpp
ptr.release();
```

The statement above releases ownership from the `unique_ptr` and returns the raw pointer.

```cpp
auto other = std::move(ptr);
```
The statement above transfers ownership from `ptr` to `other`.

```cpp
delete ptr.get();
```

The statement above violates the ownership contract of the `unique_ptr` and can lead to undefined behavior.

The final point is that whenever we use `unique_ptr`, we should always ask ourselves:

> ****Who owns this object?****

If the answer is “this class or variable alone,” `unique_ptr` is usually an appropriate choice.

If the answer is “multiple entities share ownership,” we should consider another ownership model such as `shared_ptr`.

If nobody owns the object and we only need temporary access to an existing object, a reference or non-owning raw pointer is usually more appropriate.

Ultimately, the most important benefit of `unique_ptr` is not that we no longer have to write `delete`; its main benefit is that **resource ownership is expressed explicitly, clearly, and automatically in the structure of the program.**

---
## 🤝 Contributions

<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-programmer/) | [Hadi Abbasi](hadi.abbasi.programmer@gmail.com) | [Hiens.org](https://hiens.org) | [Hadi Abbasi](@Hadi_Abbasi_Programmer) |

</div>