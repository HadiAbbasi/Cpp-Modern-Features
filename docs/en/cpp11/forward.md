<div align="right">

[🇺🇸 English](./function.md) | [🇮🇷 فارسی](../../fa/cpp11/function.md)

</div>

In C++11 we got a new reference syntax: &&. You famously know it as an rvalue reference. But in certain contexts, T&& can act as a forwarding reference (also called a universal reference) - a pattern that can bind to both lvalues and rvalues, adapting depending on what you pass to it.

A universal reference or Forwarding reference is a type of reference pattern that can bind to both lvalues and rvalues — and adapts depending on what you pass to it.

Before we talk about universal references, let’s start with something familiar — passing objects to functions. We’ll look at a few common patterns that seem fine at first but quickly break down in real-world code. These problems will naturally guide us to the idea of universal references and why C++11 introduced them.


Lets say i have a function upload that takes a Photo object as parameter and do further operation using the photo. The operation can be just upload the photo or edit the photo and upload or something else.

```cpp
void upload(Photo& p) {
    std::cout << "upload(Photo&) called\n";
    // Some operation
}

int main() {
    Photo selfie = Photo{1,2};
    upload(selfie);
}
```

Ok great its working fine. Now lets say in main function the Photo object are generated on the fly without storing in some named variable so it would be like this:
```cpp
void upload(Photo& p) {
    std::cout << "upload(Photo&) called\n";
    // Some operation
}

int main() {
    Photo selfie = Photo{1,2};
    upload(selfie);
    upload(Photo{1,2});
}

```

**faile compiler**
`error: candidate function not viable: expects lvalue as 1st argument`

This will fail because Photo{1,2} is an rvalue (temporary), but Photo& only binds to lvalues! No issues we can fix this if we can write an overload of upload function that does the same thing but accepts a rvalue reference like below:

```cpp
void upload(Photo& p) {
    std::cout << "upload(Photo&) called\n";
}

void upload(Photo&& p) {
    std::cout << "upload(Photo&&) called\n";
}

int main() {
    Photo selfie = Photo{1,2};
    upload(selfie); // calls upload(Photo&)
    upload(Photo{1,2}); // // calls upload(Photo&&)
}
```


## std::forward
In C++, std::forward() is a template function used for achieving perfect forwarding of arguments to functions so that it's lvalue or rvalue is preserved. It basically forwards the argument while preserving the value type of it.

std::forward() was introduced in C++ 11 as the part of `<utility>` header file.

```cpp
std::forward<T> (arg)
```



---
## 🤝 Contributors
<div align="center">

| GitHub | LinkedIn | Email | Site | Telegram |
|--------|----------|-------|------|----------|
| [mbr](https://github.com/mbr1376) | [mbr](https://www.linkedin.com/in/mbr1376/) | [mbr](m.roodsarabi76@gmail.com) | | [mbr](@ad1mi2n) |

</div>

