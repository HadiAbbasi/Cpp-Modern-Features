<div align="center">

[🇺🇸 English](./multi_dimensional_subscript_operator.md) | [🇮🇷 فارسی](../../fa/cpp23/multi_dimensional_subscript_operator.md)

</div>

---

# **Multidimensional Subscript Operator in C++23**

## **Introduction**

The subscript operator, `operator[]`, is one of the most commonly used operators in C++ for accessing elements of arrays, containers, and similar data structures.

Before C++23, an overloaded `operator[]` could not accept more than one parameter. As a result, developers had to use alternative approaches to implement multidimensional data structures, such as chaining multiple subscript operators, overloading the function call operator (`operator()`), or using techniques involving `operator,`.

C++23 removes this limitation by introducing the **Multidimensional Subscript Operator**.

Based on the standardization proposal `P2128R6`, this feature allows `operator[]` to accept multiple parameters directly.

For example, starting with C++23, the following definition is valid:

```
class Matrix {
public:
    double& operator[](std::size_t row, std::size_t column);
};
```

Elements of the matrix can now be accessed as follows:

```
Matrix matrix;

matrix[2, 3] = 10.5;
```

This feature is more than a syntactic improvement. It enables simpler, more readable, and more flexible designs for multidimensional data structures.

This document explores how the feature works, how it differs from traditional approaches, its relationship with `std::mdspan`, overload resolution, practical applications, limitations, and important design considerations.

## **Understanding the Subscript Operator Before C++23**

To better understand the new feature, it is useful to examine how the subscript operator worked in earlier C++ standards.

### **Parameter Count Restrictions**

Before C++23, an overloaded subscript operator in a class could accept only one explicit parameter.

For example, the following implementation was valid:

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

The class could be used as follows:

```
IntArray array(10);

array[3] = 42;

int value = array[3];
```

However, defining an operator with multiple parameters was not permitted before C++23:

```
class Matrix {
public:
    int& operator[](std::size_t row, std::size_t column);
};
```

This was because `operator[]` was restricted to a single explicit parameter.

### **Implementing Multidimensional Arrays Through Chained Subscript Operators**

One common approach to implementing multidimensional arrays was to chain multiple `operator[]` calls.

For example, accessing an element in a three-dimensional array could look like this:

```
array[x][y][z]
```

With this approach, each `operator[]` had to return another array or object that supported its own `operator[]`.

The following example demonstrates a simple implementation:

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

Usage:

```
Matrix matrix(3, 4);

matrix[1][2] = 100;

int value = matrix[1][2];
```

This approach remains valid and can still be useful in certain designs. However, it can introduce several challenges:

- Increased complexity when designing multidimensional classes.
- The need to define intermediate types for each dimension.
- Potentially multiple memory allocations.
- Greater difficulty managing multidimensional structures with a variable number of dimensions.
- More complicated designs for generic containers.

Note that separate allocations are not inevitable; their occurrence depends on how the container is implemented.

### **Using the Function Call Operator**

Another widely used approach before C++23 was to overload `operator()`.

For example:

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

Usage:

```
Matrix matrix(3, 4);

matrix(1, 2) = 25.5;

double value = matrix(1, 2);
```

This approach is technically valid and can still be an appropriate choice after C++23.

However, for data structures whose natural semantics resemble array access, `operator[]` can provide a more intuitive syntax.

## **Introducing the Multidimensional Subscript Operator in C++23**

C++23 allows `operator[]` to accept more than one explicit parameter.

This means that a function with the following signature can now be defined:

```
class Matrix {
public:
    double& operator[](std::size_t row, std::size_t column);
};
```

Elements can then be accessed using the following syntax:

```
matrix[1, 2]
```

For an object of type `Matrix`, this expression invokes the overloaded subscript operator with two arguments.

Conceptually, the expression is equivalent to the following function call:

```
matrix.operator[](1, 2);
```

More parameters can also be defined:

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

Usage:

```
tensor[1, 2, 3] = 15.0f;
```

There is no language restriction limiting this feature to two or three dimensions. The number of parameters can be greater, provided that the declaration and the corresponding expression are valid.

### **An Important Difference from Built-in Arrays**

An important point is that this feature does not change the behavior of built-in arrays or turn them into multidimensional arrays with multiple independent indices.

For example, the following code is still invalid for a built-in array:

```
int array[3][4];

array[1, 2] = 10;
```

Built-in arrays do not support overloading `operator[]`.

Accessing their elements still requires the traditional syntax:

```
array[1][2] = 10;
```

However, if the object is an instance of a class that defines a multidimensional `operator[]`, the expression `object[i, j]` can be valid.

## **How It Works and Language Rules**

### **Interpreting Multidimensional Expressions**

In C++23, the following expression:

```
object[i, j]
```

invokes the overloaded operator as follows, provided that a suitable overload is selected:

```
object.operator[](i, j);
```

Similarly:

```
object[i, j, k]
```

is equivalent to:

```
object.operator[](i, j, k);
```

The compiler therefore passes the index expressions as separate arguments to the function.

### **Number of Parameters**

In C++23, `operator[]` can accept any number of explicit parameters required by the design.

For example:

```
class MultiArray {
public:
    int& operator[](int x, int y);
    int& operator[](int x, int y, int z);
    int& operator[](int x, int y, int z, int w);
};
```

These overloads can coexist in the same class.

The compiler selects the appropriate function using the usual overload resolution rules.

### **Parameter Types**

The parameters of `operator[]` are not required to be integers.

Like other member functions, the operator can accept different types:

```
#include <string>
#include <string_view>

class DataStore {
public:
    int& operator[](std::string_view category, std::size_t index);
};
```

In this example, the first parameter represents a category, while the second represents an index.

Custom types can also be used:

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

The important point is that multidimensional subscripting is not restricted to numeric indices. It allows multiple arguments of different types to be passed to the operator.

### **Explicit Parameters and the Object Parameter**

In an ordinary member function, the object itself is implicitly associated with the function.

For example:

```
class Matrix {
public:
    int& operator[](int row, int column);
};
```

In this declaration, `row` and `column` are explicit parameters, while the `Matrix` object is represented by the implicit object parameter.

Therefore:

```
matrix[1, 2]
```

has three conceptual components:

- The object representing the matrix.
- The first parameter, with the value `1`.
- The second parameter, with the value `2`.

This distinction is important when understanding overload resolution and member function qualifiers.

## **Practical Examples**

### **Implementing a Two-Dimensional Matrix**

One of the most important applications of this feature is matrix implementation.

In the following example, elements are stored contiguously in memory, and access is provided through two indices.

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

In this implementation, matrix elements are stored in a single `std::vector`.

The following formula converts two-dimensional coordinates into a one-dimensional index:

\\[ index = row \times columns + column \\]

This approach is known as row-major storage.

For example, in a matrix with four columns:

\\[ index(2,3) = 2 \times 4 + 3 = 11 \\]

Therefore, the requested element is located at index 11 in the underlying storage.

**Advantages of this design:**

- Contiguous memory storage.
- Fewer memory allocations.
- Direct element access.
- Better compatibility with data-processing algorithms.
- More effective use of the CPU cache during suitable traversal patterns.

### **Implementing a Three-Dimensional Array**

This feature is also useful for three-dimensional data structures.

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

The formula for converting three-dimensional coordinates into a one-dimensional index in this example is:

\\[ index = z \times (height \times width) + y \times width + x \\]

The dimension ordering is important in this design.

The developer must determine which dimension changes fastest and ensure that the indexing formula matches that decision.

### **Using Different Parameter Types**

As mentioned earlier, not all parameters need to have the same type.

For example, a data structure can be designed to access a value using a section name and an index.

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

Usage:

```
Database database;

// Assume the "users" table has already been initialized.

int& value = database["users", 2];
```

In this example, the two parameters have different meanings:

- The first parameter identifies a data section by name.
- The second parameter specifies the element's position.

This implementation is intended to demonstrate the flexibility of parameter types. A real database implementation would also need to handle data management, entry creation, and access errors.

## **Managing const and Reference Semantics**

One of the most important aspects of designing `operator[]` is correctly handling `const` and the return type.

### **Defining a Non-const Overload**

In a typical container, the non-`const` version should allow elements to be modified.

```
class Matrix {
public:
    double& operator[](std::size_t row, std::size_t column);
};
```

Returning `double&` makes the following expression valid:

```
matrix[1, 2] = 100.0;
```

It also allows the address of an element to be obtained:

```
double* pointer = &matrix[1, 2];
```

### **Defining a const Overload**

A suitable overload should be provided for `const` objects:

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

This allows elements to be read from `const` matrices:

```
const Matrix& matrix = getMatrix();

double value = matrix[1, 2];
```

Returning `const double&` prevents the returned reference from being used to modify the element directly.

### **Using Deducing this to Avoid Duplication**

The Explicit Object Parameter feature, introduced in C++23 through proposal `P0847R7`, allows member functions to receive their object parameter explicitly.

This can be used to avoid duplicating `const` and non-`const` overloads.

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

In this implementation:

- If the object is non-`const`, `Self` is deduced as a non-`const` type.
- If the object is `const`, its `const` qualification is preserved.
- The `decltype(auto)` return type preserves the appropriate reference type.

Consequently, a single function can handle both cases.

This technique is useful when designing modern generic APIs. However, for simple classes, defining two separate overloads may be more readable.

## **Understanding the Relationship with std::mdspan**

One of the most important related features is `std::mdspan`.

This class was introduced into the C++23 standard library.

`std::mdspan` is a multidimensional view over memory and does not own the underlying memory.

In other words, it provides multidimensional access to data stored and managed by another object.

### **Introducing std::mdspan**

To use this feature, include the following header:

```
#include <mdspan>
```

A simple example:

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

In this example:

- `std::vector` owns the memory.
- `std::mdspan` creates a multidimensional view over that memory.
- The matrix dimensions are specified in the `std::mdspan` type.
- Elements are accessed using multiple indices.

In fact, `std::mdspan` is one of the most important practical applications of the Multidimensional Subscript Operator.

### **Understanding Ownership and Views**

An important point is that `std::mdspan` does not own memory.

Therefore, the lifetime of the memory it references must exceed the lifetime of the view.

For example:

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

In this example, `storage` manages the memory, while `matrix` is simply a view over it.

If the memory is released or the underlying pointer becomes invalid, using the `mdspan` can result in undefined behavior.

### **Using Dynamic Extents**

`std::mdspan` also allows dimensions to be specified at runtime.

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

In this example, the number of dimensions is known at compile time, while the size of each dimension is determined at runtime.

### **Understanding Layout Policies**

One important feature of `std::mdspan` is its support for different layout policies.

These policies determine how multidimensional coordinates are mapped to memory locations.

The three main layouts are:

| Layout               | Description                                                     |
| -------------------- | --------------------------------------------------------------- |
| `std::layout_right`  | The last dimension has a stride of one; typically row-major     |
| `std::layout_left`   | The first dimension has a stride of one; typically column-major |
| `std::layout_stride` | Strides are specified explicitly                                |

For example:

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

In this example, the layout is `std::layout_left`.

This demonstrates that multidimensional access syntax does not necessarily imply row-major storage.

### **The Relationship Between mdspan and the Multidimensional Subscript Operator**

It is important to distinguish between two separate concepts:

- The Multidimensional Subscript Operator is a C++23 language feature.
- `std::mdspan` is a C++23 standard library component.

The language feature allows developers to define `operator[]` with multiple parameters.

In contrast, `std::mdspan` uses this feature to provide a standard API for multidimensional memory access.

Therefore, using `std::mdspan` often eliminates the need to implement a custom multidimensional container or view from scratch.

## **Understanding the Changes to Comma Operator Behavior**

One important and relatively subtle aspect of C++23 is the change in how commas are interpreted inside subscript expressions.

This change is directly related to the Multidimensional Subscript Operator.

### **Behavior in Older Standards**

Before C++23, the following expression:

```
array[i, j]
```

was interpreted as a subscript operation with a comma expression for a built-in array or pointer.

In other words, the expression `i, j` was evaluated first, and its result was used as the index.

Since the comma operator evaluates both operands and yields the value of the right operand, the following expression:

```
array[i, j]
```

was semantically similar to:

```
array[j]
```

However, in C++20, using an unparenthesized comma expression in this context was deprecated.

### **Behavior in C++23**

In C++23, the syntax inside a subscript expression can contain a multidimensional expression list.

Therefore:

```
object[i, j]
```

can pass both values as separate arguments to an overloaded `operator[]` when the object is a class type with a suitable overload.

Consequently, this expression is no longer necessarily interpreted as a comma expression.

### **Preserving the Comma Operator with Parentheses**

If the intention is to use the comma operator, the expression must be enclosed in parentheses.

```
int array[10];

int i = 2;
int j = 5;

int value = array[(i, j)];
```

In this example, `i` is evaluated first, followed by `j`, whose value becomes the result of the comma expression.

Consequently, the fifth element is accessed.

This distinction is important for maintaining compatibility with older code and avoiding unintended behavior.

### **Comparing Two Important Expressions**

| Expression       | Meaning                                                            |
| ---------------- | ------------------------------------------------------------------ |
| `object[i, j]`   | Passes two arguments to a multidimensional overload, if one exists |
| `object[(i, j)]` | Passes the result of a comma expression as a single argument       |
| `array[i, j]`    | Invalid for a built-in array in C++23                              |
| `array[(i, j)]`  | Accesses an array using the index produced by a comma expression   |

## **Understanding Overload Resolution**

Like other overloaded operators, selecting the appropriate multidimensional `operator[]` involves the usual overload resolution rules.

### **Defining Multiple Overloads**

Multiple functions with different parameter counts or types can be defined:

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

Usage:

```
Matrix matrix;

matrix[5];

matrix[2, 3];
```

In the first expression, the single-parameter overload is selected.

In the second expression, the two-parameter overload is selected.

### **Using Different Parameter Types**

Overloading is not limited to the number of parameters.

```
#include <string_view>

class Data {
public:
    int& operator[](int index);

    int& operator[](std::string_view name, int index);
};
```

In this example, the parameter types also influence overload selection.

### **Using Templates**

`operator[]` can also be defined as a function template.

```
#include <concepts>
#include <cstddef>

class Matrix {
public:
    template<std::integral Row, std::integral Column>
    int& operator[](Row row, Column column);
};
```

In this example, `Row` and `Column` can be different integral types.

Using `std::integral` makes the type requirements more explicit.

However, a real implementation should also consider type conversions, value ranges, and potential information loss.

## **Understanding Static operator[]**

C++23 also allows certain member operators to be declared as `static`.

This capability is associated with proposal `P2589R1` and applies to operators including `operator()` and `operator[]`.

For example:

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

In this example, `operator[]` is a static member function.

An important detail is that, despite being static, the operator expression still requires an object on which the operator is invoked.

Therefore, it cannot be written directly as:

```
Utility[10, 20];
```

The class name alone is not an object that can be used in this expression.

Also, making `operator[]` static means that the function has no access to `this` and cannot directly access non-static state belonging to an instance.

### **Use Cases for Static operator[]**

This feature can be useful when designing utility objects, stateless operations, and APIs that do not require internal instance state.

However, for ordinary containers that operate on data belonging to a particular instance, a non-static member function is generally the more natural choice.

## **Understanding Explicit Object Parameters in C++23**

Another important C++23 feature is Explicit Object Parameters, also known as Deducing this.

This feature can be useful when designing generic `operator[]` implementations and managing `const` and reference qualifiers.

For example:

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

Here, `Self` represents the type of the calling object.

This feature allows a single implementation to handle both `const` and non-`const` objects.

It is important to understand that Explicit Object Parameters and the Multidimensional Subscript Operator are independent C++23 features that can be combined.

Using Deducing this is not necessary for every container. Its adoption should depend on whether it reduces code duplication and improves readability.

## **Understanding Argument Evaluation Order**

When designing a multidimensional `operator[]`, the evaluation order of its arguments is important.

For example:

```
object[getFirstIndex(), getSecondIndex()];
```

You should not assume that `getFirstIndex()` is necessarily evaluated before `getSecondIndex()`.

Since C++17, function arguments are indeterminately sequenced relative to one another. This means that one argument is evaluated first, but their evaluations do not interleave.

The exact order is unspecified.

Therefore, code that depends on argument evaluation order should be avoided.

For example, avoid writing expressions in which multiple arguments modify the same variable in conflicting or dependent ways.

A safer approach is to calculate the values before invoking the operator:

```
int first = getFirstIndex();
int second = getSecondIndex();

object[first, second];
```

In this case, the execution order is explicit and predictable.

## **Practical Applications**

The Multidimensional Subscript Operator can be used in various types of C++ projects.

### **Image Processing**

In image processing, data is commonly managed using two or more dimensions.

For example, accessing a pixel may require horizontal and vertical coordinates.

```
image[y, x];
```

This syntax can be more readable for array-oriented APIs than using `image(y, x)` or chaining multiple `operator[]` calls.

### **Scientific Computing**

In scientific computing projects, structures such as matrices and tensors are extremely common.

The new feature allows simpler APIs for accessing this data.

For example:

```
matrix[row, column];

tensor[batch, channel, height, width];
```

This is particularly useful in libraries that work with multidimensional data.

### **Game Engines**

In game engines, structures such as grids, voxel maps, and spatial data structures may require multiple coordinates.

For example:

```
world[x, y, z];
```

This syntax can make accessing information at three-dimensional positions more readable.

### **Multidimensional Data Processing**

In applications where data is organized using dimensions such as batch, channel, height, and width, this feature can provide a suitable access API.

For example:

```
dataset[batch, channel, row, column];
```

However, such projects should also consider index calculation costs, memory layout, and cache locality.

## **Limitations and Important Considerations**

Despite its advantages, this feature does not eliminate all the design challenges associated with multidimensional data structures.

### **Built-in Arrays Remain Unchanged**

The most important limitation is that this feature only extends the ability to overload `operator[]` in user-defined types.

Therefore, the following expression is still invalid in C++23:

```
int matrix[3][4];

matrix[1, 2] = 10;
```

Built-in arrays do not support multidimensional operator overloading.

### **No Automatic Multidimensional Storage**

This feature does not change how data is stored in memory.

Developers must still make decisions about:

- Memory allocation.
- Element storage.
- Data layout.
- Offset calculation.
- Lifetime management.
- Bounds checking.

In other words, the Multidimensional Subscript Operator improves only the syntax used to access data.

### **Managing Bounds Checking**

The subscript operator does not automatically perform bounds checking.

For example, if the implementation accesses an index outside the valid range, undefined behavior may occur.

The API design should clearly specify how out-of-range access is handled.

Common approaches include:

- Using `assert` to validate indices in debug builds.
- Throwing exceptions when an index is invalid.
- Providing a separate checked-access API.
- Using tools such as sanitizers to detect memory errors during development.

For example:

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

In this example, out-of-range access is reported using `std::out_of_range`.

### **Potential Overflow During Index Calculation**

In large arrays, calculating an offset can result in integer overflow.

For example:

```
index = row * columns + column;
```

If the product `row * columns` exceeds the representable range of the corresponding type, the result will not be valid.

Therefore, performance-sensitive or safety-critical implementations should validate storage capacity and check for potential overflow before allocation and access.

### **Choosing Appropriate Parameter Names**

The order of parameters in a multidimensional `operator[]` is important.

For example, the following declarations have different meanings:

```
int& operator[](std::size_t row, std::size_t column);
```

```
int& operator[](std::size_t column, std::size_t row);
```

Both are syntactically valid, but the expected parameter ordering should be clearly documented.

It is best to align parameter ordering with the container's dimension definitions and documentation.

### **Considering Proxy Objects**

`operator[]` does not always need to return a direct reference to an element.

Some designs may require a Proxy Object.

This is important for structures such as bit containers and compressed data representations.

For example, if elements of a container are stored in a bit-packed format, accessing an element may require calculating and modifying a single bit within an integer.

In such cases, returning a normal reference may not be possible, and a Proxy Object can provide more appropriate behavior.

The multidimensional nature of `operator[]` does not prevent the use of Proxy Objects.

### **Considering the Lifetime of Returned References**

If `operator[]` returns a reference, the referenced object must remain alive for as long as the reference is used.

For example, returning a reference to a local variable is incorrect:

```
class Matrix {
public:
    int& operator[](int row, int column) {
        int value = row + column;
        return value;
    }
};
```

This code has invalid behavior because it returns a reference to a local variable.

Therefore, the returned reference must refer to data with an appropriate lifetime.

## **Comparing Different Multidimensional Access Approaches**

There are several ways to access multidimensional data in C++.

The best choice depends on the API, data structure, and project requirements.

| Approach                      | Syntax       | Standard           | Main Characteristic            |
| ----------------------------- | ------------ | ------------------ | ------------------------------ |
| Chained `operator[]`          | `a[i][j]`    | Valid before C++23 | Step-by-step access            |
| Using `operator()`            | `a(i, j)`    | Valid before C++23 | Accepts multiple parameters    |
| Multidimensional `operator[]` | `a[i, j]`    | C++23              | Direct multidimensional access |
| `std::mdspan`                 | `view[i, j]` | C++23              | Standard multidimensional view |

### **When Should You Use Multidimensional operator[]?**

This approach is suitable when:

- The data structure naturally behaves like an array.
- Direct access to multiple dimensions is required.
- You want to design a readable API for matrices or tensors.
- You want to avoid intermediate types in many implementations.

### **When Is operator() More Appropriate?**

In some designs, `operator()` remains a better choice.

For example:

- When the object behaves more like a function or functor.
- When the API semantically represents a computational operation.
- When compatibility with older C++ standards is required.
- When the existing project conventions favor `operator()`.

Therefore, introducing the new feature does not make `operator()` deprecated.

### **When Should You Use std::mdspan?**

If the primary goal is to create a multidimensional view over existing memory, `std::mdspan` is often an appropriate choice.

It provides standard facilities for:

- Managing extents.
- Selecting layouts.
- Multidimensional access.
- Defining strides.
- Working with contiguous or custom-layout data.

Therefore, before implementing a new multidimensional view, it is worth examining the capabilities of `std::mdspan`.

## **Compiler Compatibility Considerations**

The Multidimensional Subscript Operator is part of the C++23 standard, but practical support depends on the compiler version and its level of C++23 implementation.

Different compilers may have introduced support for this feature in different releases.

Therefore, simply enabling the C++23 language mode does not guarantee that every C++23 feature is available in the installed compiler version.

### **Enabling C++23**

For GCC and Clang, the following compiler option can be used:

```
g++ -std=c++23 main.cpp -o app
```

Or:

```
clang++ -std=c++23 main.cpp -o app
```

For Microsoft Visual C++, the following option can be used:

```
cl /std:c++latest /EHsc main.cpp
```

The `latest` option enables the latest features supported by the installed MSVC version. It does not necessarily mean that the entire C++23 standard has been implemented.

### **Checking the Feature Test Macro**

Feature test macros can be used to check compiler support for specific language features.

The macro for the Multidimensional Subscript Operator is:

```
__cpp_multidimensional_subscript
```

The following code demonstrates how to check it:

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

This macro is provided by the compiler as part of the language feature implementation and is useful for detecting support.

The feature test macro value for this feature is `202110L`.

An important point is that support for this macro does not imply support for every C++23 feature or the entire C++23 standard library.

For example, support for multidimensional `operator[]` does not guarantee full availability of `std::mdspan`.

## **Common Mistakes**

### **Using Built-in Arrays Incorrectly**

The following code is still invalid in C++23:

```
int array[5][5];

array[2, 3] = 10;
```

This is because built-in arrays do not support multidimensional operator overloading.

For this type of array, the following syntax must be used:

```
array[2][3] = 10;
```

### **Assuming the Number of Dimensions Is Automatically Determined**

The compiler does not automatically infer the required number of dimensions from the data structure.

The developer must define the appropriate overload.

For example:

```
class Matrix {
public:
    int& operator[](int row, int column);
};
```

Only this number of parameters is supported by this overload.

### **Assuming This Feature Replaces std::mdspan**

These concepts operate at different levels.

The Multidimensional Subscript Operator is a language feature, while `std::mdspan` is a standard library component.

Therefore, they are not direct competitors.

### **Ignoring Parameter Ordering**

When designing an API, the order of indices should be clearly defined and documented.

For example, a matrix may follow this access convention:

```
matrix[row, column];
```

An image, however, may use the following ordering:

```
image[y, x];
```

This distinction should be made clear through parameter names and API documentation.

### **Misusing the Comma Operator**

In C++23, the following expression can represent two independent arguments for a class with a suitable overload:

```
matrix[row, column];
```

However, if the intention is to use the comma operator, parentheses are required:

```
matrix[(row, column)];
```

These two expressions do not have the same meaning.

## **Summary**

The Multidimensional Subscript Operator is a practical addition to C++23 that removes a longstanding limitation of `operator[]`.

Before this standard, `operator[]` could accept only one explicit parameter, so multidimensional access generally required chaining multiple operators or using `operator()`.

C++23 now allows developers to define a multidimensional member `operator[]` and provide direct access to multidimensional data.

The main advantages of this feature include:

- Simplifying the design of matrices and tensors.
- Improving the readability of multidimensional access syntax.
- Eliminating the need for intermediate types in many implementations.
- Enabling more flexible generic APIs.
- Providing natural integration with `std::mdspan` in C++23.

However, it is important to remember that this feature does not change data storage, memory management, or bounds checking.

It also cannot be used with built-in arrays and is available only for types that define an appropriate overloaded `operator[]`.

Ultimately, understanding the differences between the Multidimensional Subscript Operator, `operator()`, chained `operator[]`, and `std::mdspan` is important for C++ developers. Choosing the appropriate approach can have a direct impact on API readability, flexibility, and overall design quality.

## **References and Further Reading**

The following resources are recommended for further study and for examining the official details of this feature:

- [P2128R6 — Multidimensional subscript operator](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2128r6.pdf?utm_source=chatgpt.com) — The standardization proposal for the Multidimensional Subscript Operator.
- [cppreference — Operator overloading](https://en.cppreference.com/w/cpp/language/operators?utm_source=chatgpt.com) — Documentation covering operator overloading.
- [cppreference — Array subscript operator](https://en.cppreference.com/w/cpp/language/operator_member_access?utm_source=chatgpt.com) — Details about subscript operator behavior.
- [cppreference — std::mdspan](https://en.cppreference.com/w/cpp/container/mdspan?utm_source=chatgpt.com) — Documentation for the standard `std::mdspan` class.
- [P2589R1 — static operator()](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2589r1.pdf?utm_source=chatgpt.com) — Details about static member operators.
- [P0847R7 — Deducing this](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0847r7.html?utm_source=chatgpt.com) — Documentation about Explicit Object Parameters.
- [C++23 Feature Support](https://en.cppreference.com/w/cpp/compiler_support/23?utm_source=chatgpt.com) — Compiler support status for C++23 features.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>