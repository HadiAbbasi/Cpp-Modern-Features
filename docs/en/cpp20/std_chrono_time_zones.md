<div align="center">

[🇺🇸 English](./std_chrono_time_zones.md) | [🇮🇷 فارسی](../../fa/cpp20/std_chrono_time_zones.md)

</div>

---

# Managing Time Zones with `std::chrono` in C++20

## Introduction

Time management in real-world software is far more complex than storing a number or displaying hours and minutes. A specific instant in the world, the local time of a city, a calendar date, the offset from UTC, and daylight saving time (DST) rules are related but distinct concepts. Confusing them can lead to errors in scheduling, event logging, reporting, payments, reservation systems, and historical data processing.

The `std::chrono` library has provided facilities for representing durations and time points since C++11. However, C++20 expanded it with standardized, integrated support for calendars and time zones. The C++20 time-zone facilities make it possible to convert between system time and local time, identify ambiguous or nonexistent local times, and use a time-zone database.

These facilities are part of the standard library, not new language syntax. Their availability therefore depends on the standard library implementation and the time-zone data available at runtime. Simply enabling the `-std=c++20` compiler option does not guarantee that all time-zone APIs are available in the version of the library being used.

## The Mental Model of Time and Time Zones

### The Difference Between an Instant, Local Time, and a Duration

To design time-related functionality correctly, first distinguish between three concepts:

- Duration: The interval between two instants, such as 30 minutes or 24 hours.
- Time point: A specific instant relative to a particular clock and epoch.
- Local time: The calendar and wall-clock representation of an instant in a time zone, such as 9:00 AM in Tehran.

A duration is independent of the time zone. For example, 3,600 seconds represents the same amount of time in every region. However, the local wall-clock time can differ depending on the region and its DST rules.

In `std::chrono`, types such as `std::chrono::seconds` represent durations, while `std::chrono::time_point` represents time points. System time, local time, and UTC time also belong to different logical clock domains. They must not be treated as interchangeable merely because their numeric representations look similar.

### System Time and UTC

The type `std::chrono::sys_time<Duration>` is an alias for a `time_point` based on `std::chrono::system_clock`. In practice, it is used to represent system time relative to the epoch defined for `system_clock`. In the `chrono` time-zone model, it serves as the basis for conversion to local time.

The type `std::chrono::utc_time<Duration>` is based on `utc_clock` and models the UTC time scale. UTC is not identical to Unix time or `sys_time`: UTC accounts for leap seconds in its model, whereas `sys_time` is used for conventional system timekeeping and does not have the same countable leap-second model as UTC.

For most business applications, recording an instant using `sys_time` is a common choice. If a problem depends on the accuracy of the UTC time scale, leap seconds, or conversions between scientific time scales, the appropriate clock must be selected deliberately.

### A Region's Local Time

The type `std::chrono::local_time<Duration>` represents a local time point. By itself, it does not store a time-zone name or offset. For this reason, a `local_time` value alone is not sufficient to identify a unique instant.

For example, the expression "1:30 AM on a particular date" may occur twice or may not occur at all during a DST transition. A time zone must also be specified to perform a valid conversion.

### A Time Zone Is Not the Same as a Fixed Offset

An offset such as `+03:30` indicates only the time difference for a particular period. A region such as `Europe/Paris` represents a set of historical and future rules under which its offset and time-zone abbreviation may change on different dates.

Therefore, storing only an offset is not sufficient to reconstruct historical or future local time. If the geographical meaning and rules of a region matter, store its IANA time-zone identifier as well.

## Introduction to the Main Library Components

### Namespace and Headers

The calendar and time-zone facilities are in the `std::chrono` namespace. Their primary header is `<chrono>`.

```
#include <chrono>
#include <format>
#include <iostream>
#include <string>
```

The `<format>` header is used for formatting with `std::format`. Formatting support for `chrono` types also depends on the standard library implementation. In environments where this functionality is incomplete, other approaches can be used to display the output.

### The Time-Zone Database

The type `std::chrono::tzdb` represents a version of the time-zone database. This database is usually based on the IANA Time Zone Database, or tzdb, and provides the library with information about time zones, naming links, and leap seconds.

Important members of `tzdb` include:

- `version`: The identifier or version of the time-zone data.
- `zones`: The collection of time zones.
- `links`: Naming links between identifiers.
- `leap_seconds`: Leap-second information.

Time-zone data is part of the runtime environment's operational data and is not necessarily compiled into the application itself. Consequently, the database version and its update mechanism can affect the results of conversions involving historical or future dates.

### The Time-Zone Type

The class `std::chrono::time_zone` represents a time zone using an identifier such as `Europe/Paris` or `America/New_York`. Rather than constructing an object of this class directly, applications typically obtain a pointer to one through the database functions.

Its main operations include retrieving information about a `sys_time` or `local_time` and converting between the two.

### Time-Interval Information

The type `std::chrono::sys_info` provides information about a region's rules for an interval of system time. Its important members include:

- `begin` and `end`: The beginning and end of the interval during which the information is valid.
- `offset`: The difference between local time and system time during that interval.
- `save`: The amount of adjustment associated with DST.
- `abbrev`: The time-zone abbreviation for that interval, such as `CET` or `CEST`.

This information is useful for determining the effective offset at a particular instant. Do not assume that abbreviations such as `CST` are globally unique. An IANA time-zone identifier is more reliable for identifying a region.

## Finding a Time Zone

### Retrieving a Specific Time Zone

The function `std::chrono::locate_zone` accepts a time-zone identifier and returns a pointer to the corresponding object.

```
#include <chrono>
#include <iostream>

int main() {
    try {
        const auto* zone = std::chrono::locate_zone("Europe/Paris");

        std::cout << zone->name() << '\n';
    } catch (const std::chrono::runtime_error& error) {
        std::cerr << error.what() << '\n';
    }
}
```

If the identifier cannot be found in the database, the function throws a runtime error. Do not accept identifiers directly from user input without validation and error handling. A time-zone name may contain a typo or may not be available in the target environment.

### Retrieving the System's Default Time Zone

The function `std::chrono::current_zone` returns a pointer to the system's current time zone.

```
#include <chrono>
#include <iostream>

int main() {
    try {
        const auto* zone = std::chrono::current_zone();
        std::cout << zone->name() << '\n';
    } catch (const std::chrono::runtime_error& error) {
        std::cerr << error.what() << '\n';
    }
}
```

The system's default time zone is an environmental dependency. Running an application on a developer's laptop, a cloud server, or inside a container may produce different default time zones. For reproducible business logic, obtain the time zone explicitly from configuration or input data, and rely on the system setting only when displaying local time for the user.

### The Difference Between the Default and an Explicit Time Zone

Using `current_zone` is appropriate for displaying the system's local time. However, for an event associated with a particular city or organization, it is better to specify an identifier such as `Europe/Paris` explicitly. Otherwise, changing the system's time-zone setting can change the application's behavior.

## Converting Between System Time and Local Time

### Retrieving Information About an Instant

The `time_zone::get_info` function accepts a `sys_time` and returns the interval information that applies to that instant.

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const sys_seconds instant =
        sys_days{2026y / October / 25} + 1h + 30min;

    const sys_info info = zone->get_info(instant);

    std::cout << "Offset: " << info.offset.count() << " seconds\n";
    std::cout << "Abbreviation: " << info.abbrev << '\n';
}
```

In this example, `sys_days` converts a calendar day into a system time point, after which a duration is added to it. The result of `get_info` provides the offset and abbreviation that are valid for that particular instant.

### Converting from System Time to Local Time

The `time_zone::to_local` function converts a `sys_time` to the local time in the corresponding time zone. This direction of conversion is unambiguous for every system-time instant.

```
#include <chrono>
#include <format>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Asia/Tehran");
    const sys_seconds instant = sys_seconds{sys_days{2026y / November / 13}}
                              + 12h + 0min;

    const local_seconds local = zone->to_local(instant);

    std::cout << std::format("{:%F %T}", local) << '\n';
}
```

If the implementation supports `chrono` formatting, the `"%F %T"` format displays the date and time. Formatting a local time displays only the local value. For output that is clear and unambiguous, it is usually a good idea to print the time-zone name or offset alongside it.

### Converting from Local Time to System Time

The reverse direction is not always straightforward. The `get_info(local_time)` function determines whether the requested local time is unique, nonexistent, or ambiguous. After identifying the situation, it can be converted to system time using `to_sys`.

## Handling Unique, Nonexistent, and Ambiguous Times

### The Unique Case

Under normal circumstances, a local time corresponds to exactly one system-time instant. In this case, `local_info::result` is equal to `local_info::unique`, and the corresponding information is available in the `first` member.

### The Nonexistent Case

When DST begins, the wall clock moves forward in some regions. As a result, a range of local times never occurs. For example, the clock might jump directly from 01:59:59 to 03:00:00. In that case, 02:30 does not exist.

This situation is identified by `local_info::nonexistent`. An application should not convert such a time into an arbitrary instant without an explicit policy. Depending on the use case, it can reject the input, move it to the first valid time afterward, or ask the user to choose another time.

### The Ambiguous Case

When DST ends, the wall clock moves backward, causing a range of local times to occur twice. In this case, one `local_time` corresponds to two system-time instants, and the result is `local_info::ambiguous`.

To choose between the two instants, the `std::chrono::choose` enum provides the values `earliest` and `latest`. The choice must be consistent with the application's meaning. For example, an attendance-tracking system may need a different choice from a meeting-scheduling system.

### Explicitly Checking the Local-Time Status

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const local_seconds local =
        local_seconds{local_days{2026y / October / 25}} + 2h + 30min;

    const local_info info = zone->get_info(local);

    switch (info.result) {
    case local_info::unique:
        std::cout << "The local time is unique\n";
        break;
    case local_info::nonexistent:
        std::cout << "The local time does not exist\n";
        break;
    case local_info::ambiguous:
        std::cout << "The local time is ambiguous\n";
        break;
    }
}
```

This example checks the status and deliberately avoids assuming that a local time is always valid and unique. The exact result depends on the region's rules and the version of tzdb available in the runtime environment.

### Using `choose` for an Ambiguous Time

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const local_seconds local =
        local_seconds{local_days{2026y / October / 25}} + 2h + 30min;

    const sys_seconds earlier = zone->to_sys(local, choose::earliest);
    const sys_seconds later   = zone->to_sys(local, choose::latest);

    std::cout << (later - earlier).count() << " seconds\n";
}
```

The `choose` parameter resolves the ambiguity between two valid instants. It should not be considered a general solution for nonexistent times. A nonexistent time has no corresponding instant, so it requires a separate policy.

## Using `zoned_time`

### The Role of `zoned_time`

The `std::chrono::zoned_time` template associates a time zone with a system-time point. It provides a convenient representation for a time when both the actual instant and the time zone used to display it matter.

In a typical design, `zoned_time` stores the instant using the `sys_time` scale and uses the time zone for conversion and display. Therefore, it is suitable for representing an instant in a specified region, not for storing an ambiguous local time without resolving the ambiguity.

### Constructing `zoned_time` with a Specific Time Zone

```
#include <chrono>
#include <format>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Asia/Tehran");
    const sys_seconds instant = floor<seconds>(system_clock::now());

    const zoned_time<seconds> zoned{zone, instant};

    std::cout << std::format("{:%F %T %Z}", zoned) << '\n';
}
```

The `%Z` specifier is used to display the time-zone name or abbreviation in `chrono` formatting. The exact abbreviation in the output depends on the time-zone data and the implementation.

### Constructing with a Time-Zone Identifier

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const zoned_time<seconds> zoned{
        "Europe/London",
        floor<seconds>(system_clock::now())
    };

    std::cout << zoned.get_time_zone()->name() << '\n';
}
```

The `zoned_time` constructors support both a time-zone pointer and a time-zone identifier. When using an identifier, the same considerations for handling a time zone that cannot be found still apply.

### Retrieving Local Time and System Time

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const zoned_time<seconds> zoned{
        "America/New_York",
        floor<seconds>(system_clock::now())
    };

    const local_seconds local = zoned.get_local_time();
    const sys_seconds system = zoned.get_sys_time();

    std::cout << local.time_since_epoch().count() << '\n';
    std::cout << system.time_since_epoch().count() << '\n';
}
```

The two values above use different time scales. Directly comparing their counts or interpreting both as Unix timestamps would be incorrect.

### Choosing the `Duration` Type

`zoned_time` has a template parameter that determines the duration precision. Choosing `seconds` is sufficient for many event-logging applications. However, if subsecond precision is required, an appropriate precision such as `milliseconds` or `microseconds` should be selected. Match the storage precision to the actual precision of the time source and the requirements of the problem domain.

## Getting to Know the `chrono` Calendar

### Calendar Dates

The C++20 calendar facilities complement time zones. Types such as `year`, `month`, `day`, `year_month_day`, and `weekday` provide explicit representations of the components of a date.

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2026y / November / 13};

    std::cout << static_cast<int>(date.year()) << '-'
              << static_cast<unsigned>(date.month()) << '-'
              << static_cast<unsigned>(date.day()) << '\n';
}
```

These types make calendar calculations safer and more readable than manually manipulating numeric day and month values.

### Converting a Date to System Days

The type `sys_days` represents a time point with day-level precision based on `system_clock`. It is useful for date calculations and constructing time points based on calendar days.

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2026y / November / 13};
    const sys_days day_point{date};

    std::cout << day_point.time_since_epoch().count() << '\n';
}
```

### Validating a Date

The `year_month_day` type can represent dates that are not valid according to the calendar. Use the `ok` function to check whether a date is valid.

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day valid{2024y / February / 29};
    const year_month_day invalid{2025y / February / 29};

    std::cout << valid.ok() << '\n';
    std::cout << invalid.ok() << '\n';
}
```

Validate dates before converting external input into time values or using them in business logic.

## Formatting and Parsing Time

### Formatting with `std::format`

C++20 `chrono` formatting supports format specifiers for dates, times, and time zones. This approach is generally more readable and consistent than manually constructing strings.

```
#include <chrono>
#include <format>
#include <iostream>

int main() {
    using namespace std::chrono;

    const zoned_time<seconds> zoned{
        "Europe/Paris",
        floor<seconds>(system_clock::now())
    };

    std::cout << std::format("{:%Y-%m-%d %H:%M:%S %Z}", zoned) << '\n';
}
```

Some commonly used format specifiers include:

- `%Y`: Four-digit year.
- `%m`: Two-digit month.
- `%d`: Day of the month.
- `%H`: Hour in 24-hour format.
- `%M`: Minute.
- `%S`: Second.
- `%F`: Date in the form `YYYY-MM-DD`.
- `%T`: Time in the form `HH:MM:SS`.
- `%Z`: Time-zone name or abbreviation.
- `%z`: Numeric offset from UTC.

Full support for `chrono` formatting has not been consistent across all standard library versions and environments. Test the functionality with the actual compiler and standard library versions used in the target environment.

### Parsing Input with `parse`

The `std::chrono::parse` facility is provided for reading dates and times from input streams using a specified format. This API is useful for parsing textual data, but parsing a string alone does not replace semantic time-zone validation or DST ambiguity handling.

```
#include <chrono>
#include <iomanip>
#include <iostream>
#include <sstream>

int main() {
    using namespace std::chrono;

    std::istringstream input{"2026-11-13 09:45:00"};
    local_seconds local;

    input >> parse("%F %T", local);

    if (input.fail()) {
        std::cerr << "Invalid date-time input\n";
        return 1;
    }

    std::cout << local.time_since_epoch().count() << '\n';
}
```

In this example, the string is converted to `local_seconds`. Because the string does not specify a time zone, a time zone and a policy for handling ambiguous or nonexistent times must still be selected before converting the value into a unique instant.

## Managing the Time-Zone Database

### Retrieving the Current Version

The function `std::chrono::get_tzdb` returns a reference to the current version of the time-zone database.

```
#include <chrono>
#include <iostream>

int main() {
    const auto& db = std::chrono::get_tzdb();
    std::cout << db.version << '\n';
    std::cout << db.zones.size() << '\n';
}
```

Recording the tzdb version in diagnostic logs can help reproduce differences in results between environments.

### Inspecting Database Versions

The function `std::chrono::get_tzdb_list` provides a list of loaded versions in the form of a `tzdb_list`. This list can be used to inspect available versions and manage previous versions.

```
#include <chrono>
#include <iostream>

int main() {
    const auto& databases = std::chrono::get_tzdb_list();

    for (const auto& db : databases) {
        std::cout << db.version << '\n';
    }
}
```

### Reloading the Data

The function `std::chrono::reload_tzdb` attempts to update the time-zone database from the implementation's data source and returns a reference to the new version.

```
#include <chrono>
#include <iostream>

int main() {
    try {
        const auto& db = std::chrono::reload_tzdb();
        std::cout << db.version << '\n';
    } catch (const std::exception& error) {
        std::cerr << error.what() << '\n';
    }
}
```

The update mechanism, data location, network or file access, and availability of reloading depend on the implementation. An application should not assume that calling this function will always retrieve new data or that updating the database is possible in every environment.

### Remote Version

The function `std::chrono::remote_version` is intended to retrieve the remote version of the time-zone database in implementations that provide this functionality. This operation may depend on external infrastructure and access. Therefore, it should not be relied upon in a critical execution path without appropriate error handling.

### The Lifetime of Time-Zone Pointers

Functions such as `locate_zone` and `current_zone` return pointers to time-zone objects. When working with reload operations and multiple database versions, account for the lifetime and ownership of the data according to the library's contract. Avoid storing pointers or references to data whose lifetime you do not understand in long-lived structures. Document the database version and reload timing when designing services that retain time-related data over extended periods.

## Leap Seconds and Other Clocks

### Leap Seconds

Leap seconds are used to keep UTC synchronized with the Earth's rotation. The associated data can be inspected through `tzdb::leap_seconds`, and the type `std::chrono::leap_second` represents information about a leap-second event.

Many everyday software systems do not need to manage leap seconds directly. However, time-sensitive financial systems, telecommunications, astronomy, measurement systems, and applications that record precise events should examine the clock model and the runtime environment's policies.

### The UTC Clock

The type `std::chrono::utc_clock` models the UTC clock. The `to_utc` and `from_utc` functions are provided in the clock-related API for conversions between system time and UTC.

```
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const sys_seconds system_time = floor<seconds>(system_clock::now());
    const utc_time<seconds> utc = utc_clock::from_sys(system_time);
    const sys_seconds back = utc_clock::to_sys(utc);

    std::cout << (back - system_time).count() << '\n';
}
```

Around leap seconds, conversions between `sys_time` and `utc_time` can involve subtleties. The example above demonstrates the conversion path; it should not be taken as proof that the two time scales are identical at every instant.

### TAI and GPS Clocks

The `chrono` library also defines clocks such as `tai_clock` and `gps_clock`. These clocks are useful for applications that depend on more precise time scales or specialized standards. They should be selected according to the data contract and the requirements of the problem domain. Simply replacing `system_clock` with one of these clocks without examining the epoch and differences between the time scales is not correct.

## Practical Applications in Software Design

### Scheduling an Event in a City

For a meeting that must take place at the local time of a particular city, accept the input as a date, time, and time-zone identifier. Then use `get_info` to check the local-time status. If the time is ambiguous or nonexistent, apply a defined policy, such as rejecting the input or asking the user to choose again.

For future events, storing only a `sys_time` may not fully preserve the business meaning. If the event rule is "9:00 AM Paris time," also retain the time-zone identifier and the intended local time. A change to DST legislation may change the corresponding instant in the future.

### Event Logging

For recording technical events, storing the system-time instant as UTC or as a standard timestamp is usually more appropriate than storing local time. The user's time zone can be stored separately for display purposes. This separation avoids ambiguity caused by repeated wall-clock times.

### Daily Reporting

A "day" in a time zone is not necessarily equivalent to a 24-hour interval. A day on which a DST transition occurs may have 23 or 25 hours. For daily reports, first determine the calendar-day boundaries in the target time zone, then convert those boundaries to system time. Do not add a fixed 24 hours to reach the next day.

### Data Storage

Depending on the requirements of the problem domain, consider storing the following information for time-related data:

- The absolute instant, usually as a UTC-based timestamp or `sys_time`.
- The IANA time-zone identifier, if the geographical meaning matters.
- The local time and the user's original input, if the initial decision needs to be reproducible.
- The tzdb version or audit information, if reproducing historical results is important.

Not every system needs to store all of these values simultaneously. Design the data model around the question of whether the goal is to preserve a definitive instant or a local appointment that must remain consistent with the region's future rules.

## Implementation Limitations and Dependencies

### Compiler and Standard Library Support

The `chrono` time-zone facilities are part of the C++20 standard library. In practice, their support depends on the version of the standard library distributed with the compiler. A compiler may accept C++20 language mode while some of the time-zone APIs or `chrono` formatting facilities in its accompanying library remain incomplete.

For a real project, create a small test program for the APIs you need and compile and run it with every target toolchain. Do not rely on the compiler version alone. The version of `libstdc++`, `libc++`, or whichever standard library implementation is being used also matters.

### Dependence on tzdb Data

Conversion results for historical and future dates depend on the database version. Governments may change official time rules at short notice. If the environment's tzdb data is outdated, conversions for future times may no longer match the latest legal rules.

### Differences Between Environments

Containers, minimal servers, and embedded systems may not have time-zone data installed, or they may provide it through a different mechanism. The presence of an API at compile time does not guarantee that the required data will be available at runtime.

### Cost and Performance

Time conversion is generally not an expensive operation. However, looking up a time zone, accessing its rule data, and formatting values in frequently executed code paths can introduce overhead. If optimization is necessary, measure actual performance. Caching results is appropriate only when validity intervals, database reloads, and concurrency are handled correctly.

## Common Mistakes and Design Considerations

### Assuming the Offset Is Constant

Do not assume that a region's offset remains constant throughout the year. For date-dependent conversions, use `time_zone` rather than adding a fixed number to a timestamp.

### Assuming Local Time Is Always Unique

Do not convert local time to system time without checking `local_info`. Nonexistent and ambiguous times are a natural part of DST behavior.

### Using an Abbreviation Instead of a Time-Zone Identifier

Abbreviations such as `CST` or `IST` may refer to multiple regions. Prefer an IANA identifier such as `Asia/Tehran` when storing or exchanging time-zone information.

### Using Local Time to Order Events

Determine event ordering using `sys_time` or another appropriate absolute time scale, not a local-time string. During a repeated hour, two different events may have identical local-time representations.

### Adding 24 Hours to Reach the Next Day

Use calendar operations to move to the next calendar day in a region. Adding `24h` to an instant on a DST transition day does not necessarily produce the same local time on the following day.

### Ignoring Runtime Errors

Functions that look up a time zone by name or load database data may fail. Design input boundaries, configuration handling, and runtime behavior with clear error-handling policies.

### Relying on the Development Machine's Local Behavior

Tests that use `current_zone` may produce different results on different machines. For deterministic tests, select an explicit time zone and include dates close to DST transitions in the test suite.

## Recommended Testing Strategy

To ensure the correctness of time-related logic, do not limit tests to ordinary dates. Cover the following cases:

- A normal local time that corresponds to exactly one instant.
- A nonexistent local time at the start of DST.
- An ambiguous local time at the end of DST.
- Instants immediately before and after an offset change.
- Leap dates and month and year boundaries.
- An invalid time-zone identifier.
- Missing tzdb data or an error while loading it.
- Application behavior after the database is updated.
- Test execution across multiple time zones and standard library versions.

For tests that depend on tzdb, record the data version or run the test environment with a specific, controlled data set. Otherwise, a database update may change the expected test results.

## Comparison with Older Approaches

### Comparison with `time_t` and C Functions

Functions such as `std::time`, `std::localtime`, and `std::gmtime` belong to the older time interfaces in the C library. These interfaces are still encountered when maintaining legacy code and interacting with operating-system APIs, but they do not provide the type-safe model and composability of `chrono`.

Some of these functions use static structures or process-dependent local state, which can cause problems in multithreaded applications or applications that manage multiple time zones simultaneously. For new C++ code, the type-safe `chrono` model is a clearer choice. However, conversions may still be necessary at boundaries where the application interacts with older APIs.

### Comparison with Third-Party Libraries

Libraries such as Howard Hinnant's date library and specialized time libraries may be useful in projects that use older toolchains or require capabilities beyond the standard API. Some aspects of the design of the C++20 standard calendar and time-zone library were influenced by earlier work in this area.

A third-party library should be selected based on platform support, tzdb update policies, parsing and formatting capabilities, licensing, and compatibility requirements. Avoid combining multiple time models without clearly defining their boundaries.

## Summary

The time-zone facilities in `std::chrono` introduced in C++20 provide a more standardized way to model local time, convert it to system time, and use regional time-zone rules. The main components of this model include `time_zone`, `tzdb`, `sys_info`, `local_info`, and `zoned_time`.

The most important design principle is not to treat an absolute instant, local time, and time zone as a single concept. Converting from system time to local time is unambiguous for a specified region, but the reverse conversion may encounter nonexistent or ambiguous times. An application must handle these situations deliberately.

For professional software, specify the time zone explicitly when the problem domain requires it, account for the version and availability of tzdb at runtime, test DST boundaries, and use an appropriate absolute time scale for recording events. These principles help prevent many subtle and difficult errors in time-sensitive systems.

---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAbbasi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>