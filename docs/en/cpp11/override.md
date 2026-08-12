<div align="right">

[🇺🇸 English](./override.md) | [🇮🇷 فارسی](../../fa/cpp11/override.md)

</div>

---
# The `override` Directive in `C++11` and Later

Table of Contents

- [General Concept](#general-concept)
- [What the `override` Directive Is](#what-the-override-directive-is)
- [Where `override` Really Saves Us](#where-override-really-saves-us)
- [`Override` Mechanism vs `Overload`](#override-mechanism-vs-overload)
- [The Nature of `Override`](#the-nature-of-override)
- [A Very Important Comparison](#a-very-important-comparison)
- [`Overload` Structure](#overload-structure)
- [`Override` Structure](#override-structure)
- [An Example Showing Both Together](#an-example-showing-both-together)
- [A Very Common Mistake](#a-very-common-mistake)
- [Even `const` Matters](#even-const-matters)
- [What `Override` Actually Does](#what-override-actually-does)
- [A Very Short Mental Sentence](#a-very-short-mental-sentence)
- [Contributions](#contributions)

## General Concept

I am intentionally overriding a `virtual function` from the base class, so if such a function with this `signature` does not exist in the `base`, it should produce an error.

## What the `override` Directive Is

Suppose we have a parent class like the following:

```cpp
class Animal {
public:
    virtual void speak() {
        std::cout << "Animal speaks\n";
    }
};
```

And the child class:

```cpp
class Dog : public Animal {
public:
    void speak() override {
        std::cout << "Dog barks\n";
    }
};
```

Here:

```cpp
void speak() override
```

means:

I expect `speak` to be exactly a `virtual function` in the base class, and this function overrides that version in the child class.

What happens if we do not use `override`? We could write:

```cpp
class Dog : public Animal {
public:
    void speak() {
        std::cout << "Dog barks\n";
    }
};
```

This code is also completely valid. Because `speak()` in `Animal` is `virtual`, the `Dog::speak()` function also overrides it. So this:

```cpp
void speak()
```

and this:

```cpp
void speak() override
```

have no difference in `runtime` behavior.

The main difference is that `override` lets the compiler detect our mistake.

## Where `override` Really Saves Us

Suppose we mistakenly write:

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

Note:

```cpp
speak()   // starts with lowercase
Speak()   // starts with uppercase
```

These two are different!  
`C++` is case-sensitive.

Without `override`, the compiler may say:

No problem; `Dog` has a new function named `Speak`.

In other words, it is not even an `overload`; here we have created a new and `unrelated` function.

But if we write:

```cpp
class Dog : public Animal {
public:
    void Speak() override {
        std::cout << "Dog barks\n";
    }
};
```

The compiler says:

```text
error: function marked 'override' but does not override
```

## `Override` Mechanism vs `Overload`

`Override` means:

A child class replaces the `implementation` of a `virtual function` from the parent class.

For example:

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

Both of these classes have the following function with the same signature!

```cpp
speak()
```

So:

```text
Animal
   │
   │ virtual speak()
   ▼
Dog
   │
   └── override speak()
```

## The Nature of `Override`

In `Overload`, we have several functions with the same name but different `signature`s.

For example:

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

Here:

```cpp
add(int, int)
add(double, double)
add(int, int, int)
```

All of them are named `add`, but they have different `signature`s.

This is called:

`Function Overloading`

and it has nothing to do with `inheritance` or `virtual`.

## A Very Important Comparison

### `Overload` Structure

```cpp
void foo(int);
void foo(double);
```

Meaning:

```text
A class
   │
   ├── foo(int)
   └── foo(double)
```

Purpose:

Several versions of one `function` with different input parameters.

### `Override` Structure

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

Meaning:

```text
Parent
   │
   └── virtual foo(int)
             ▲
             │ override
             │
Child ───────┘
```

Purpose:

Changing the `implementation` of the parent's `virtual` function in the child class.

## An Example Showing Both Together

Here the issue becomes very interesting:

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

Here `Dog` has two functions:

```cpp
speak(int)
speak(string)
```

These two are `overload`s of each other.

But:

```cpp
speak(int)
```

is an `override` relative to `Animal::speak(int)`.

So a function can simultaneously be part of an `overload set` in one `context` and, in relation to the `base class`, be considered an `override`.

## A Very Common Mistake

In this code:

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

A programmer might think:

“Well, I changed the parent's `speak`.”

But that has not happened!

Because:

```cpp
speak(int)
```

and:

```cpp
speak(double)
```

are not the same.

So here we do not have an `override`.

In fact, `Dog` has a new function.

If we write:

```cpp
void speak(double x) override {}
```

The compiler immediately finds the mistake.

## Even `const` Matters

For example:

```cpp
class Animal {
public:
    virtual void speak() const {
    }
};
```

If we write in the child class:

```cpp
class Dog : public Animal {
public:
    void speak() {
    }
};
```

This is not an `override`.

Because:

```cpp
speak() const
```

and:

```cpp
speak()
```

do not have the same `signature` required for `override`.

However:

```cpp
class Dog : public Animal {
public:
    void speak() const override {
    }
};
```

is correct.

And this is one of the best reasons to use `override`:

You do not need to visually check these details yourself every time; the compiler checks them.

## What `Override` Actually Does

Burn this sentence into your mind:

`override` tells the compiler that I intentionally intend to override a `virtual function` from the base class; therefore, if my function is not actually a `virtual` function that can be overridden from the `base`, stop the program with a `compile` error.

And the main difference:

```text
OVERLOAD
────────
Same name
Different parameters
Usually same class
No inheritance required
No virtual required
```

In contrast:

```text
OVERRIDE
────────
Same function signature
Base → Derived
Requires a virtual function in the base
Used for runtime polymorphism
override keyword helps the compiler catch mistakes
```

## A Very Short Mental Sentence

`Overload = same name, different signature.`  
`Override = same virtual function, new implementation in the derived class.`

And `override` itself does not perform `polymorphism`; it is `virtual` in the `base` that provides `virtual dispatch`. `override` mostly acts as a `compile-time safety check`.

Just as when using Git with the following command:

```bash
git push origin devs
```

the last two words are there to ensure that if we are on a branch other than `devs`, it gives an error, `Override` behaves similarly: if we make a mistake while overriding and rewriting a parent function, the compiler gives us an appropriate error!

🤝

## Contributions

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>