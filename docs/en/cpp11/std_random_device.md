<div align="right">

[🇺🇸 English](./std_random_device.md) | [🇮🇷 فارسی](../../en/cpp11/std_random_device.md)

</div>

---

# Random Number Generation in C++ with `std::random_device`, `std::mt19937`, and Distributions

The C++ standard library provides a flexible framework for generating random numbers through the `<random>` header. Introduced in C++11, this framework offers a more powerful and structured alternative to older techniques such as `std::rand()` and `std::srand()`.

Modern C++ random number generation is built around three main components:

- Entropy sources: Such as `std::random_device`, which may rely on operating-system randomness or an implementation-specific alternative.
- Random number engines: Such as `std::mt19937`, which generate sequences of pseudorandom numbers.
- Random number distributions: Such as `std::uniform_int_distribution`, which transform engine output into values following a specified statistical distribution.

Understanding how these components work together is essential for writing reliable simulations, games, tests, and statistical applications.

## Understanding the Difference Between Engines and Distributions

Random number engines and distributions serve different purposes. An engine generates values from an internal pseudorandom sequence, while a distribution maps those values to the statistical behavior required by an application.

For example, if you want to simulate a six-sided die, using an engine alone is not the best approach. Instead, use a uniform integer distribution to generate values in the range from 1 to 6.

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{std::random_device{}()};
    std::uniform_int_distribution<int> distribution{1, 6};

    for (int i = 0; i < 10; ++i) {
        std::cout << distribution(engine) << '\n';
    }
}
```

In this example, `std::mt19937` generates the pseudorandom sequence, and `std::uniform_int_distribution` produces integers between 1 and 6, inclusive.

## Introduction to `std::random_device`

The `std::random_device` class provides a source of random values for applications. Depending on the implementation, it may obtain values from a nondeterministic source or use an alternative pseudorandom mechanism.

It has been available in the C++ standard library since C++11.

### The Primary Purpose of `std::random_device`

One common use of this class is to provide a seed for a random number engine such as `std::mt19937`. The goal is to obtain different initial states across program executions under typical conditions.

```
#include <iostream>
#include <random>

int main() {
    std::random_device rd;

    std::cout << rd() << '\n';
    std::cout << rd() << '\n';
    std::cout << rd() << '\n';
}
```

In this program, each call to `rd()` returns a value of type `std::random_device::result_type`.

### Important Limitations of `std::random_device`

You should not assume that this class always uses hardware randomness or a cryptographically secure random source on every platform.

- Some implementations obtain randomness from the operating system.
- Other implementations may use a pseudorandom generator under certain conditions.
- Calling it may be more expensive than generating values with a conventional random number engine.
- Its exact behavior depends on the standard library implementation and the execution environment.

Therefore, `std::random_device` is a common choice for seeding a random number engine, but its use alone does not guarantee cryptographic security.

### Understanding `entropy()`

The `entropy()` member function returns an estimate of the source's entropy. This value can provide information about the source, but it does not independently guarantee that the output is suitable for cryptographic purposes.

```
#include <iostream>
#include <random>

int main() {
    std::random_device rd;

    std::cout << "Entropy estimate: "
              << rd.entropy()
              << '\n';
}
```

The reported value should be interpreted according to the implementation's documentation. A value of zero does not necessarily mean that the output is entirely devoid of randomness in every circumstance.

## Introduction to `std::mt19937`

The `std::mt19937` engine is one of the best-known pseudorandom number generators in the C++ standard library. It implements the Mersenne Twister algorithm and has been available since C++11.

Its name refers to a parameter of the algorithm: its period is \\(2^{19937}-1\\).

### Key Characteristics of `std::mt19937`

- It has an extremely long period.
- It is useful for simulations, game development, software testing, and statistical sampling.
- It is generally efficient for generating large numbers of values compared with many operating-system randomness sources.
- It supports reproducible sequences when initialized with the same seed.
- It is not designed for direct use in cryptographic applications.

### Initializing the Engine

You can initialize the engine with a fixed seed:

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{12345};

    for (int i = 0; i < 5; ++i) {
        std::cout << engine() << '\n';
    }
}
```

With the same seed and compatible implementation, the engine produces a reproducible sequence. This property is especially useful for software testing and debugging.

### Initializing with `std::random_device`

When exact reproducibility is not required, you can initialize the engine using a value obtained from `std::random_device`:

```
#include <random>

int main() {
    std::random_device rd;
    std::mt19937 engine{rd()};

    return static_cast<int>(engine() & 1U);
}
```

This is a simple approach, but it uses only one seed value. If you want to initialize an engine using multiple seed values, you can use `std::seed_seq`.

### Understanding `std::seed_seq`

The `std::seed_seq` class transforms a collection of input seed values into data suitable for initializing a random number engine.

```
#include <iostream>
#include <random>
#include <array>

int main() {
    std::random_device rd;

    std::array<std::random_device::result_type, 8> seed_data{};

    for (auto& value : seed_data) {
        value = rd();
    }

    std::seed_seq seed_sequence(seed_data.begin(), seed_data.end());
    std::mt19937 engine(seed_sequence);

    std::cout << engine() << '\n';
}
```

This approach is useful when initializing an engine from multiple seed values. However, using more seed values does not, by itself, guarantee sufficient source entropy or cryptographic security.

## Understanding Random Number Distributions

Distributions determine how an engine's output is transformed into values required by an application. The standard library provides several distributions for integral values, floating-point values, and common statistical models.

### Uniform Integer Distribution

The `std::uniform_int_distribution` distribution generates integers within a closed interval, meaning that both the lower and upper bounds are included.

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{42};
    std::uniform_int_distribution<int> distribution{10, 20};

    for (int i = 0; i < 10; ++i) {
        std::cout << distribution(engine) << '\n';
    }
}
```

In this example, every generated value is between 10 and 20, inclusive.

This distribution is suitable for selecting random indices, simulating dice rolls, and generating integers within a specified range.

### Uniform Real Distribution

The `std::uniform_real_distribution` distribution generates floating-point values with a uniform distribution.

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{42};
    std::uniform_real_distribution<double> distribution{0.0, 1.0};

    for (int i = 0; i < 5; ++i) {
        std::cout << distribution(engine) << '\n';
    }
}
```

The distribution uses the interval \\([a,b)\\), which includes the lower bound but excludes the upper bound.

This distribution is suitable for simulations, probabilistic selection, and generating floating-point values within a specified range.

### Normal Distribution

The `std::normal_distribution` distribution generates values following a normal distribution with a specified mean and standard deviation.

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{42};
    std::normal_distribution<double> distribution{100.0, 15.0};

    for (int i = 0; i < 5; ++i) {
        std::cout << distribution(engine) << '\n';
    }
}
```

In this example, the distribution has a mean of 100 and a standard deviation of 15.

Generated values are not restricted to a specific interval and may deviate substantially from the mean. If you need to constrain the output to a range, you should choose an appropriate statistical approach. Simply clamping values with `std::clamp` does not necessarily preserve the intended normal distribution.

### Bernoulli Distribution

The `std::bernoulli_distribution` distribution is suitable for events with two possible outcomes, such as success and failure.

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{42};
    std::bernoulli_distribution distribution{0.7};

    for (int i = 0; i < 10; ++i) {
        std::cout << (distribution(engine) ? "Success" : "Failure")
                  << '\n';
    }
}
```

In this example, the probability of obtaining `true` is 0.7.

### Poisson Distribution

The `std::poisson_distribution` distribution is used to model the number of occurrences within an interval under the appropriate assumptions of a Poisson process.

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{42};
    std::poisson_distribution<int> distribution{4.0};

    for (int i = 0; i < 10; ++i) {
        std::cout << distribution(engine) << '\n';
    }
}
```

The distribution parameter represents the expected number of occurrences and must be a valid, nonnegative value.

### Exponential Distribution

The `std::exponential_distribution` distribution is useful for modeling the time between events in appropriate stochastic processes.

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{42};
    std::exponential_distribution<double> distribution{2.0};

    for (int i = 0; i < 5; ++i) {
        std::cout << distribution(engine) << '\n';
    }
}
```

The parameter of this distribution is the rate \\(\lambda\\), and the mean of the generated values is \\(1/\lambda\\).

## Managing Engine State and Reproducibility

One of the most important characteristics of pseudorandom number engines is their internal state. Each time an engine is invoked, its state changes and the next value in the sequence is generated.

### Fixed Seeds Versus Variable Seeds

Using a fixed seed is useful for repeatable tests and simulations. Conversely, using a variable initial source can produce different sequences across program executions.

```
#include <iostream>
#include <random>

int main() {
    std::mt19937 engine{2026};
    std::uniform_int_distribution<int> distribution{1, 100};

    for (int i = 0; i < 5; ++i) {
        std::cout << distribution(engine) << '\n';
    }
}
```

For automated testing, a fixed seed is generally a good choice because it makes results reproducible.

### Saving and Restoring Engine State

Standard engines such as `std::mt19937` support stream operators for saving and restoring their state.

```
#include <iostream>
#include <random>
#include <sstream>

int main() {
    std::mt19937 engine{42};

    engine();

    std::stringstream state;
    state << engine;

    const auto expected = engine();

    std::mt19937 restored;
    state >> restored;

    std::cout << (restored() == expected) << '\n';
}
```

In this example, the engine state is saved after one invocation. The original engine then generates another value, and the restored engine generates the same next value.

This capability is useful for reproducing failures and resuming simulations. To reproduce an entire process, you must also consider the state of distributions that maintain internal state and any other factors that affect the computation.

## Practical Application: Random Selection from a Collection

To select a random element from a collection, you can use `std::uniform_int_distribution`. The standard library also provides the `std::sample` algorithm, introduced in C++17, for sampling elements from a sequence.

```
#include <algorithm>
#include <iostream>
#include <random>
#include <vector>

int main() {
    std::vector<int> values{10, 20, 30, 40, 50};
    std::mt19937 engine{42};

    auto selected = values.begin();

    std::advance(
        selected,
        std::uniform_int_distribution<std::size_t>{
            0, values.size() - 1
        }(engine)
    );

    std::cout << *selected << '\n';
}
```

This example assumes that the collection is not empty, so the index range is valid. If the collection might be empty, check for emptiness before constructing the distribution or selecting an index.

## Important Considerations for Multithreaded Applications

Random number engines maintain mutable state. Concurrently accessing the same engine without proper synchronization can result in a data race.

In multithreaded applications, it is generally preferable to give each thread its own engine or protect access to a shared engine with an appropriate synchronization mechanism.

Additionally, initializing multiple independent engines with the same seed can produce identical sequences. For parallel simulations, the seeding strategy should be designed with the requirements for reproducibility and sequence independence in mind.

## Common Mistakes in Random Number Generation

### Using `%` to Restrict the Output Range

Using the modulo operator to map an engine's output to a desired range can introduce modulo bias, particularly when the number of possible engine outputs is not divisible by the number of values in the target range.

The recommended approach is to use the standard distribution appropriate for the intended range and value type.

### Reseeding on Every Iteration

Constructing a new engine on every iteration of a loop, especially when using a source such as `std::random_device`, generally introduces unnecessary overhead and makes reproducibility harder to manage.

Instead, create the engine once at an appropriate scope and reuse it across multiple calls.

### Assuming Cryptographic Security

The `std::mt19937` engine is designed for general statistical applications, not for generating cryptographic keys, security tokens, or other secret values. Its long period does not imply cryptographic security.

For security-sensitive applications, use secure random APIs provided by the operating system or a reputable cryptographic library.

### Assuming Raw Engine Output Is Uniform Over the Desired Range

Raw engine output is not necessarily directly suitable for the range required by an application. Distributions are designed to transform engine output into values with the required statistical properties.

## Summary

The `<random>` library in C++ provides a flexible framework for generating and using random numbers. Choosing the appropriate components depends on the application's requirements:

- For initialization in typical applications, `std::random_device` is a common choice, although its exact properties depend on the implementation.
- For efficient and reproducible pseudorandom sequences, `std::mt19937` is a widely used option.
- To transform engine output into values with specific statistical properties, choose an appropriate distribution.
- For testing and debugging, a fixed seed is generally useful.
- For cryptographic security, do not rely on `std::mt19937`.

Ultimately, designing a sound random number generation strategy involves more than selecting an engine. Initialization, distribution selection, state management, statistical requirements, and security considerations must all be taken into account.

---

## 🤝 Contributers

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |
