<div align="center">

[🇺🇸 English](./std_chrono_calendar.md) | [🇮🇷 فارسی](../../fa/cpp20/std_chrono_calendar.md)

</div>

---

# A Comprehensive Guide to Calendar in `std::chrono` in C++20

## Introduction

The `std::chrono` library in C++20 provides a collection of standard
tools for representing and performing calculations with time, durations,
calendar dates, and, at a more advanced level, time-zone-aware times.
The Calendar portion of the library makes it possible to represent the
components of a Gregorian date, validate them, perform monthly and
yearly calculations, and convert dates into a day-based numeric
representation.

Before C++20, the standard facilities for working with calendar dates
were limited. Types such as `std::time_t` and the `std::tm` structure
were primarily designed around the C calendar-time model and functions
such as `std::localtime` and `std::gmtime`. These APIs have limitations
when it comes to precise calendar calculations, thread safety, and
explicit time-zone management. C++20's `std::chrono` library provides a
more type-safe and composable model.

These facilities are defined in the `<chrono>` header and the
`std::chrono` namespace. Using them requires C++20 support in the
standard library; enabling C++20 language mode alone does not guarantee
that every library component, particularly the Time Zone Database, is
available in the implementation being used.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, February, 29d};

    if (date.ok()) {
        std::cout << date << '\\n';
    }
}
```

## Calendar Dates and Time Points

Understanding the data model of `std::chrono` is the most important step
toward using Calendar correctly. A calendar date and a point on the
timeline are related, but distinct, concepts.

### Calendar Dates

A calendar date consists of components such as a year, month, and day.
For example, `2024-02-29` is a date in the Gregorian calendar. By
itself, this date does not specify an hour, minute, second, or time
zone.

The `year_month_day` type is designed specifically for this model. It
stores the date components and provides a way to check their validity.

### Time Points

A `time_point` represents a point relative to the Epoch of a Clock. For
example, `sys_time` represents a time relative to the system clock's
Epoch. A time point can have a precision such as seconds, milliseconds,
or nanoseconds.

Converting a date to `sys_days` turns it into a day-precision time point
on the system timeline. This conversion does not determine a local time
of day or a time zone; it simply provides a day-based representation of
the date.

### Durations

A `duration` represents a time interval, not a date or a specific
instant. Types such as `days`, `months`, and `years` are involved in
Calendar calculations, but they do not have the same meaning. A day is a
fixed-length unit in this model, whereas months and years are
variable-length calendar units.

## Core Calendar Components

All Calendar types in C++20 are in the `std::chrono` namespace and are
defined in the `<chrono>` header.

### The `day` Type

The `day` type represents a day of the month. Its value is usually
between 1 and 31, but its constructor does not necessarily constrain the
value to a valid range.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr day first{1};
    constexpr day thirty_first{31};
    constexpr day invalid{0};

    std::cout << first.ok() << '\\n';
    std::cout << thirty_first.ok() << '\\n';
    std::cout << invalid.ok() << '\\n';
}
```

The `ok()` member function checks whether a `day` value is within the
type's general valid range. It does not determine whether that day
exists in a particular month. For example, `day{31}` is valid as a
`day`, but April 31 is not a valid date.

### The `month` Type

The `month` type represents a month of the Gregorian year. Its valid
values range from January through December, numbered from 1 to 12.

The library provides named constants for the months:

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month m1 = January;
    constexpr month m2 = February;
    constexpr month m3 = December;

    static_assert(m1.ok());
    static_assert(m2.ok());
    static_assert(m3.ok());
}
```

Numeric initialization is also possible:

``` cpp
constexpr std::chrono::month march{3};
```

The type supports increment and decrement operators, as well as addition
and subtraction with `months`, allowing month-by-month traversal. Month
arithmetic operates at the Calendar level and should not be confused
with adding a fixed number of days.

### The `year` Type

The `year` type represents a Gregorian calendar year. Its representable
range is limited by the standard; in C++20, year values in the range
`[-32767, 32767]` can be represented. This type represents Gregorian
years and is not, by itself, a type for other calendars such as the
Persian or Islamic calendars.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr year y{2024};
    static_assert(y.ok());

    constexpr auto next = y + years{1};
    static_assert(next == year{2025});
}
```

Numeric literals with the `y` suffix, such as `2024y`, are provided in
C++20 to construct a `year` value.

### The `weekday` Type

The `weekday` type represents a day of the week in the seven-day
Gregorian cycle. The constants `Sunday`, `Monday`, `Tuesday`,
`Wednesday`, `Thursday`, `Friday`, and `Saturday` are available.

In this type's numeric representation, Sunday has the value zero and
Saturday has the value six. This convention differs from APIs that
consider Monday the first day of the week, so the convention must be
checked explicitly when converting data from other systems.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr weekday wd{Monday};

    std::cout << wd.c_encoding() << '\\n';
    std::cout << wd.iso_encoding() << '\\n';
}
```

The `c_encoding()` function returns the numeric representation in which
Sunday is zero. The `iso_encoding()` function uses the ISO convention,
in which Monday is one and Sunday is seven.

### The `weekday_indexed` Type

The `weekday_indexed` type stores a day of the week and its occurrence
index within a month. For example, "the second Tuesday of the month" can
be represented with this model.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr weekday_indexed second_tuesday = Tuesday[2];
    static_assert(second_tuesday.ok());
}
```

The occurrence index ranges from 1 to 5. A valid combination does not
mean that the specified occurrence necessarily exists in every
particular month; this type does not yet identify a year and month.

### The `weekday_last` Type

The `weekday_last` type represents the last occurrence of a particular
weekday in a month. For example, "the last Friday of the month" can be
represented with `Friday[last]`.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr weekday_last final_friday = Friday[last];
    static_assert(final_friday.ok());
}
```

### The `last` Constant

The `last` constant has type `last_spec` and is used to express the last
day of a month or the last occurrence of a weekday in a month. It is not
a date by itself; it acquires meaning when combined with Calendar types.

## Constructing Dates with the Slash Operator

One useful Calendar feature is the `/` operator. It makes it possible to
construct calendar combinations in a readable order.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr auto date1 = 2024y / February / 29d;
    constexpr auto date2 = April / 2025;
    constexpr auto date3 = June / 15;
}
```

The resulting type depends on the components used in the expression. For
example, combining a year and a month produces a `year_month`, while
combining a month and a day produces a `month_day`. When all three
components are present, the result is a `year_month_day`.

In production code, keep the resulting type with `auto` or an explicit
type such as `year_month_day`, and prefer typed values to avoid
ambiguity. Using raw integers in some combinations is permitted, but the
`y` and `d` suffixes and named constants improve readability and type
clarity.

## Composite Date Types

### The `month_day` Type

The `month_day` type stores a month and a day without specifying a year.
This model is useful for dates such as an anniversary or New Year's Day
whose month and day are fixed while the year varies.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month_day new_year = January / 1;
    constexpr month_day halloween = October / 31;

    static_assert(new_year.ok());
    static_assert(halloween.ok());
}
```

The `ok()` function on `month_day` can check whether the day is within
the acceptable range for the month. However, because no year is
specified, it cannot determine whether February 29 is valid in the
intended year.

### The `month_day_last` Type

The `month_day_last` type represents the last day of a month without
requiring the number of days in that month for a particular year to be
known.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month_day_last end_of_february = February / last;
}
```

To obtain an actual date, a year must also be specified;
`year_month_day_last` serves this purpose.

### The `year_month` Type

The `year_month` type stores a specific year and month. It is useful for
monthly calculations such as moving to the next or previous month.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr year_month current = 2024y / December;
    constexpr year_month next = current + months{1};

    static_assert(next == 2025y / January);
}
```

### The `year_month_day` Type

The `year_month_day` type is the most common type for representing a
complete Gregorian date. It stores the year, month, and day components.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr year_month_day leap_day{2024y, February, 29d};
    static_assert(leap_day.ok());

    constexpr year_month_day invalid_date{2023y, February, 29d};
    static_assert(!invalid_date.ok());
}
```

This type can also store an invalid date in its components. Therefore,
successfully constructing an object does not mean that the date is
valid. Always check `ok()` at input boundaries, when processing external
data, or after calculations that might move the day outside the target
month's range.

### The `year_month_day_last` Type

The `year_month_day_last` type represents the last day of a particular
month in a particular year. It is suitable for calculating the end of a
month without hard-coding the number of days.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr year_month_day_last end_of_february{
        2024y, February / last
    };

    const year_month_day date{end_of_february};

    std::cout << date << '\\n';
}
```

Converting it to `year_month_day` produces the actual date. February's
last day is 29 in a leap year and 28 in a common year.

### The `month_weekday` Type

The `month_weekday` type stores a month and a specified occurrence of a
weekday, but no year. For example, "the fourth Thursday of November" can
be represented as a monthly pattern.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month_weekday thanksgiving_pattern =
        November / Thursday[4];
}
```

This pattern is not, by itself, a definite date. A year must also be
supplied to determine the actual day.

### The `month_weekday_last` Type

The `month_weekday_last` type stores a month and the last occurrence of
a weekday in that month. It also does not include a year.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    constexpr month_weekday_last last_monday_pattern =
        February / Monday[last];
}
```

### The `year_month_weekday` Type

The `year_month_weekday` type specifies a year, month, and the nth
occurrence of a weekday. It is suitable for rules such as "the second
Monday of October."

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr year_month_weekday date_pattern =
        October / Monday[2] / 2024y;

    const sys_days point = date_pattern;
    const year_month_day date{point};

    std::cout << date << '\\n';
}
```

This model is useful for calculating floating holidays and recurring
schedules.

### The `year_month_weekday_last` Type

The `year_month_weekday_last` type specifies a year, month, and the last
occurrence of a weekday. For example, "the last Friday of November 2024"
can be represented with this type.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr year_month_weekday_last pattern =
        Friday[last] / November / 2024y;

    const year_month_day date{sys_days{pattern}};
    std::cout << date << '\\n';
}
```

## Date Validation

### Checking Validity with `ok()`

Calendar types that store calendar values provide an `ok()` member
function. It checks the validity of the value or combination represented
by that particular type.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, February, 30d};

    if (!date.ok()) {
        std::cout << "Invalid date\\n";
    }
}
```

Validation must be performed at the appropriate type level. For example,
`day{31}` is valid within the range of the `day` type, but `April / 31`
is not a valid date. Also, `month_day` has no year and therefore cannot
determine whether February 29 is valid for a particular year.

### Invalid Values and Normalization

The `year_month_day` type does not necessarily correct its input
components when it is constructed. For example, a date such as April 31
can be stored in an object, but `ok()` returns false.

When the year and month are valid, converting such a value to `sys_days`
or `local_days` can interpret the day arithmetically relative to the
first day of the month, according to the standard conversion rules. This
behavior is different from rejecting the input or clamping it to the
last day of the month.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day input{2024y, April, 31d};

    const sys_days normalized_point{input};
    const year_month_day normalized_date{normalized_point};

    std::cout << input.ok() << '\\n';
    std::cout << normalized_date << '\\n';
}
```

In this example, April 31 is interpreted as the day after April 30,
namely May 1. If the application's policy is to reject invalid input,
check `ok()` before performing the conversion. Do not use calendar
conversion as a substitute for input validation.

### Year Range Limitations

The `year` type has a defined numeric range. In applications involving
very distant historical dates, astronomical data, or unbounded input,
account for the type's representable range in the API design.

## Converting Dates to a Day-Based Representation

### The `sys_days` Type

The `sys_days` type is an alias for a `time_point` based on
`system_clock` with day precision. Its value represents the number of
days elapsed since the system clock's Epoch, 1970-01-01.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr year_month_day date{2024y, May, 20d};
    constexpr sys_days point{date};

    std::cout << point.time_since_epoch().count() << '\\n';
}
```

A day-based representation is suitable for comparing dates, sorting
them, traversing days, and performing calculations based on whole days.

### The `local_days` Type

The `local_days` type is also a day-precision `time_point`, but it is
based on `local_t` rather than the system timeline. It is used for local
time representations that have not yet been mapped to a time zone.

The difference between `sys_days` and `local_days` is subtle but
important. Both can represent a calendar date at day precision, but
their meanings and underlying clocks differ. `sys_days` is generally
used to convert a calendar date into a point on the system timeline.
`local_days` is appropriate for modeling a local date or date-time that
has not yet been mapped to a time zone.

### Bidirectional Conversion with `year_month_day`

Conversion between `year_month_day` and `sys_days` is used to switch
between a component-based representation and a day-based representation.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, July, 4d};

    const sys_days serial_date{date};
    const year_month_day recovered{serial_date};

    std::cout << recovered << '\\n';
}
```

For a valid date, converting to `sys_days` and back to `year_month_day`
reconstructs the same date. The reverse conversion for invalid values
follows its own rules, so do not assume it is an automatic way to repair
data without first checking validity.

## Calendar Arithmetic

### Adding and Subtracting Days

To add or subtract a number of days, first convert the date to
`sys_days`. Then add or subtract `days` from the day-based time point.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day start{2024y, March, 1d};
    const sys_days start_point{start};

    const year_month_day after_ten_days{start_point + days{10}};

    std::cout << after_ten_days << '\\n';
}
```

This approach naturally crosses month and year boundaries and correctly
accounts for month lengths and leap years.

### Adding and Subtracting Months

Use `months` for month-based calculations. The `year_month` and
`year_month_day` types provide the operators needed for month-by-month
movement.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month current{2024y, January};
    const year_month next_month = current + months{1};

    std::cout << next_month << '\\n';
}
```

Month addition to a date that includes a day requires care. For example,
adding one month to January 31 does not necessarily mean the last day of
February; the intermediate result may contain a day that does not exist
in the target month. In such cases, check `ok()` or implement the
desired policy explicitly with `year_month_day_last`.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    year_month_day date{2024y, January, 31d};
    date += months{1};

    std::cout << date.ok() << '\\n';
}
```

If the business requirement is "the same day number in the next month,"
the result may be invalid. If the requirement is "the last day of the
target month when that day number does not exist," implement that policy
explicitly.

### Adding and Subtracting Years

Use `years` to move by calendar years. The important edge case is
February 29. Adding one year to February 29 can produce an invalid date
in the target year.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    year_month_day leap_day{2024y, February, 29d};
    leap_day += years{1};

    std::cout << leap_day.ok() << '\\n';
}
```

In financial, reservation, or subscription systems, define whether the
anniversary of a leap-day date in a non-leap year should be February 28,
March 1, or some other date. The library cannot infer the business
policy.

### Calculating the Last Day of a Month

The combination of `year_month_day_last` and `last` calculates the last
day of any month without requiring a manually maintained table of month
lengths.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day_last end_of_month{
        2025y, February / last
    };

    const year_month_day date{end_of_month};

    std::cout << date << '\\n';
}
```

This approach is useful for end-of-month calculations, monthly due
dates, and financial reports.

### Calculating the First Day of the Next Month

A straightforward way to find the first day of the next month is to work
with `year_month`.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month current{2024y, December};
    const year_month next = current + months{1};
    const year_month_day first_day{next / 1};

    std::cout << first_day << '\\n';
}
```

### Calculating the Nth Weekday of a Month

To find the nth occurrence of a weekday, use `weekday_indexed` and
`year_month_weekday`.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_weekday second_monday =
        Monday[2] / February / 2025y;

    const year_month_day date{sys_days{second_monday}};

    std::cout << date << '\\n';
}
```

This pattern is useful for floating holidays, recurring meetings, and
calendar rules such as "the second Monday of the month."

### Calculating the Last Weekday of a Month

To find the last occurrence of a weekday, use `weekday_last`.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_weekday_last last_friday =
        Friday[last] / November / 2025y;

    const year_month_day date{sys_days{last_friday}};

    std::cout << date << '\\n';
}
```

### Calculating the Difference Between Two Dates

To calculate the number of days between two dates, convert both to
`sys_days` and subtract them.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day first{2024y, January, 1d};
    const year_month_day second{2024y, December, 31d};

    const days difference = sys_days{second} - sys_days{first};

    std::cout << difference.count() << '\\n';
}
```

This difference counts the number of day boundaries between the two
dates. If "the number of days including both the start and end dates" is
intended (an inclusive count), you may need to add one to the result.
Make the interval's definition explicit in the application logic.

## Weekday Calculations

### Getting the Weekday from a Date

After converting a date to `sys_days`, construct the corresponding
`weekday`.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, January, 1d};
    const weekday day_of_week{sys_days{date}};

    std::cout << day_of_week << '\\n';
}
```

### Comparing Weekdays

`weekday` values can be compared. For conditions such as weekends or
business days, define the business-day convention in the application;
the standard does not determine which days are holidays in a particular
country or organization.

``` cpp
#include <chrono>

bool is_weekend(std::chrono::weekday wd) {
    using namespace std::chrono;
    return wd == Saturday || wd == Sunday;
}
```

This function implements one common assumption about weekends. For
international systems, business days and holidays should be configured
according to the relevant region or business rules.

### C and ISO Encodings

For `weekday`, `c_encoding()` treats Sunday as zero, whereas
`iso_encoding()` treats Monday as one and Sunday as seven. This
difference can cause weekday shifts when exchanging data with other
APIs.

## Getting the Current Date

To obtain the current system date, use `system_clock::now()` and round
it down to day precision.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto now = system_clock::now();
    const auto today = floor<days>(now);
    const year_month_day date{today};

    std::cout << date << '\\n';
}
```

Using `floor<days>` avoids incorrect conversion behavior. For times
before the Epoch, the difference between `floor` and a simple conversion
to an integer can matter: `floor` rounds toward negative infinity, not
toward zero.

An important point is that `system_clock` represents the system time
scale, and this example extracts the date corresponding to that scale.
For a user's local date, the time zone must be taken into account; the
system clock's time scale is not necessarily the same as the intended
local time zone.

## Combining a Date with a Time of Day

Calendar models only the date portion. C++20 introduces `hh_mm_ss` for
representing the time of day. This type decomposes a `duration` into
hours, minutes, seconds, and a fractional part.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    constexpr auto elapsed = 13h + 42min + 18s;
    const hh_mm_ss time_of_day{elapsed};

    std::cout << time_of_day << '\\n';
}
```

`hh_mm_ss` is a decomposed representation of a duration; it does not, by
itself, create a date or a time associated with a time zone. To
construct a complete local date-time, a `local_days` value can be
combined with a duration since midnight.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    const local_days date = local_days{days{20000}};
    const local_time<seconds> local_datetime =
        date + 13h + 42min + 18s;
}
```

This model still does not specify a time zone. Mapping this local time
to system time requires time-zone rules.

## Time Zones and Local Time

### The Difference Between `local_time` and `sys_time`

The `sys_time` type represents a time point on the system timeline. It
is independent of the local representation used by time zones. The
`local_time` type represents a local time that has not yet been
associated with a specific time zone.

For example, `2025-10-26 02:30` may be ambiguous in a particular time
zone because the clock is moved backward at the end of Daylight Saving
Time, causing that local time to occur twice. In spring, some local
clock times may not exist at all.

### Introducing `time_zone` and `zoned_time`

C++20 standardizes facilities for working with a Time Zone Database,
including `time_zone`, `time_zone_link`, `tzdb`, `tzdb_list`,
`zoned_time`, `sys_info`, and `local_info`.

The `zoned_time` type stores a time point together with a time zone and
is useful for converting between system time and its local
representation.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const zoned_time local{zone, system_clock::now()};

    std::cout << local << '\\n';
}
```

The time-zone identifier must exist in the Time Zone Database provided
by the implementation. Availability of the database and support for the
time-zone APIs can differ among standard library implementations; C++20
language support alone does not guarantee that time-zone data is
available.

### Converting Local Time to System Time

Converting from system time to local time with a specified time zone is
unambiguous. However, converting from local time to system time may be
ambiguous or impossible.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const local_time<minutes> local =
        local_days{year{2025}/October/26} + 2h + 30min;

    const local_info info = zone->get_info(local);

    if (info.result == local_info::unique) {
        const zoned_time result{zone, local};
        std::cout << result << '\\n';
    } else if (info.result == local_info::ambiguous) {
        std::cout << "Ambiguous local time\\n";
    } else {
        std::cout << "Nonexistent local time\\n";
    }
}
```

For an ambiguous time, the application must decide which occurrence to
select. For a nonexistent time, it must define a policy such as
rejecting the input or shifting it to the next valid time. The
`zoned_time` constructors support the `choose::earliest` and
`choose::latest` policies for resolving ambiguity.

``` cpp
#include <chrono>

int main() {
    using namespace std::chrono;

    const auto* zone = locate_zone("Europe/Paris");
    const local_time<minutes> local =
        local_days{year{2025}/October/26} + 2h + 30min;

    const zoned_time resolved{zone, local, choose::earliest};
}
```

The selection policy applies to ambiguous times; choosing `earliest` or
`latest` does not generally solve the problem of a nonexistent local
time. Define the expected application behavior explicitly.

### Retrieving Time-Zone Information

The `get_info` function retrieves information for a `sys_time` or
`local_time` from a time zone. Information such as the offset from UTC,
the time-zone abbreviation, and the interval for which the information
is valid is provided through `sys_info` or `local_info`.

This information is useful for displaying offsets, detecting
daylight-saving transitions, and analyzing local times. Do not assume
that a time zone's offset is constant throughout the year or across
historical years.

### Updating the Time-Zone Database

Time-zone rules can change as a result of legislation. The Time Zone
Database provides a version of the data to the library and exposes APIs
for accessing it. Systems that depend on the correctness of future or
historical times should account for their time-zone data update policy.

## Formatting and Parsing Dates

### Output with `operator<<`

Calendar types support stream insertion. This approach is useful for
quick display and debugging.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, August, 17d};
    std::cout << date << '\\n';
}
```

### Formatting with `std::format`

C++20 introduces formatting support for time-related types through
`std::format` and the corresponding `chrono` formatters. However,
standard library implementations may provide different levels of support
in different versions.

``` cpp
#include <chrono>
#include <format>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month_day date{2024y, August, 17d};

    std::cout << std::format("{:%Y-%m-%d}", date) << '\\n';
}
```

Common format specifiers include `%Y` for the year, `%m` for the month,
and `%d` for the day. When using `chrono` formatters, check the
documentation for the implementation and standard library version in
use.

### Parsing Input with `std::chrono::from_stream`

C++20 provides `from_stream` for parsing calendar values from a stream.
This API uses time-format specifiers.

``` cpp
#include <chrono>
#include <iostream>
#include <sstream>

int main() {
    using namespace std::chrono;

    std::istringstream input{"2024-02-29"};
    year_month_day date;

    from_stream(input, "%F", date);

    if (!input.fail() && date.ok()) {
        std::cout << date << '\\n';
    } else {
        std::cout << "Invalid input\\n";
    }
}
```

For external input, successful parsing alone is not sufficient; also
validate the calendar date with `ok()`. The input format must match the
application's data contract. For example, `%F` represents the ISO
year-month-day date format.

## Practical Example: Calculating a Month's Range

In reporting and database queries, it is often necessary to obtain the
beginning of a month and the beginning of the following month. Using a
half-open interval `[start, end)` generally avoids errors involving the
last instant of a day.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month month_to_report{2024y, February};

    const sys_days start{month_to_report / 1};
    const sys_days end{(month_to_report + months{1}) / 1};

    std::cout << start << '\\n';
    std::cout << end << '\\n';
}
```

With this model, every time point at or after the beginning of the month
and before the beginning of the next month belongs to the reporting
interval. For timestamp data, day boundaries must be constructed in the
intended time zone and then converted to system time; using UTC directly
for a local report may assign records near midnight to the wrong day or
month.

## Practical Example: Generating the Days of a Month

To iterate over the days of a month, converting the beginning and end of
the month to `sys_days` is a simple and reliable approach.

``` cpp
#include <chrono>
#include <iostream>

int main() {
    using namespace std::chrono;

    const year_month selected_month{2024y, February};
    const sys_days first{selected_month / 1};
    const sys_days next_month{(selected_month + months{1}) / 1};

    for (sys_days current = first; current < next_month; current += days{1}) {
        const year_month_day date{current};
        const weekday wd{current};

        std::cout << date << " weekday=" << wd << '\\n';
    }
}
```

This approach naturally handles the number of days in each month and
leap years, avoiding the need to build a list of days manually.

## Practical Example: Calculating a Monthly Due Date

In financial applications, adding one month to a due date may need to
follow a specific policy. The following example implements the policy
"if the day does not exist in the target month, choose the last day of
the target month."

``` cpp
#include <chrono>
#include <iostream>

std::chrono::year_month_day add_month_clamped(
    std::chrono::year_month_day date,
    std::chrono::months delta
) {
    using namespace std::chrono;

    const year_month target_month = date.year() / date.month() + delta;
    const year_month_day candidate{target_month / date.day()};

    if (candidate.ok()) {
        return candidate;
    }

    return year_month_day{target_month / last};
}

int main() {
    using namespace std::chrono;

    const year_month_day due_date{2024y, January, 31d};
    const auto next_due_date = add_month_clamped(due_date, months{1});

    std::cout << next_due_date << '\\n';
}
```

This function implements one example policy; it is not the default
behavior of all month-based calculations. In financial systems, the
business contract may require a different policy, such as preserving a
day-based interval or moving to the next business day.

## Calendar Compared with Older APIs

### Comparison with `std::tm`

The `std::tm` structure comes from the C time API and has fields for the
year, month, day, hour, minute, and second. In this structure, the year
is measured relative to 1900 and the month is zero-based; these
conventions are prone to off-by-one errors.

By contrast, `year_month_day` uses typed values and represents months
with values from 1 to 12. Calendar types are also designed for
validation and calendar arithmetic.

### Comparison with `std::chrono::system_clock::to_time_t`

The `to_time_t` function converts a time point to `time_t`. This
conversion is useful for interacting with legacy operating-system or C
APIs, but it is not the preferred choice for modern Calendar
calculations. Keep the internal parts of an application in `std::chrono`
types whenever possible, and convert only at compatibility boundaries.

### Comparison with External Date Libraries

External libraries may provide non-Gregorian calendars, advanced
parsing, broader locale support, or specialized time-zone capabilities.
C++20's standard Calendar is based on the Gregorian calendar and is not
a complete replacement for specialized internationalization or
multi-calendar libraries.

## Limitations and Design Considerations

### Focus on the Gregorian Calendar

The `year`, `month`, `day`, and composite Calendar types are designed
for the Gregorian calendar. For the Persian, Islamic, Hebrew, or other
calendars, a specialized model or library is required. Calendar
conversion must be performed using well-defined and reliable rules.

### No Built-In Holiday or Business-Day Concept

The library calculates weekdays and dates, but it does not maintain
lists of public holidays, organization-specific holidays, bank business
days, or regional rules. This information must be managed in the
application's domain layer.

### Calendar Days Are Not Necessarily 24 Hours

Adding `days{1}` to a time point on the system timeline means adding
exactly 24 hours. However, adding one calendar day to a local time in a
region that observes daylight-saving transitions can correspond to an
interval of 23 or 25 hours on the system timeline.

For local scheduling, first determine whether the requirement is "24
hours from now" or "the same local time on the next calendar day." These
are different requirements.

### Define an Explicit Policy for Invalid Dates

Do not silently normalize an invalid date unless that behavior is part
of the API contract. For user input, files, and network messages,
rejecting invalid values and reporting an error is usually the safer
choice.

### Define an Explicit Policy for Time-Zone Ambiguities

For local times that are repeated or skipped because of official clock
changes, the application must define explicit behavior. Choosing the
first or last occurrence, rejecting the value, or shifting it to the
next valid time are domain-level decisions.

### Implementation Differences

A feature's presence in the standard does not mean that it is fully
available in every version of a compiler and standard library. Calendar,
formatting, and Time Zone Database support may differ among versions of
GCC/libstdc++, Clang/libc++, and MSVC STL. For a real project, maintain
a small compile-and-run test and a CI matrix covering the target
toolchains.

## Common Mistakes

-   **Assuming a date is valid after construction:** Constructing
    `year_month_day` does not guarantee validity; check `ok()`.
-   **Treating `months` as a fixed number of days:** Month lengths vary;
    use `months` for month-based calculations.
-   **Adding 24 hours instead of one local calendar day:**
    Daylight-saving transitions can change the length of a local day.
-   **Assuming weekday numbering is universal:** The conventions used by
    `c_encoding()` and `iso_encoding()` differ.
-   **Ignoring leap years:** February 29 and yearly calculations require
    an explicit policy.
-   **Assuming the Time Zone Database is available in every
    implementation:** Check library and time-zone data support in the
    target toolchain.
-   **Automatically converting invalid dates into valid dates:** Such a
    conversion may move the date into the next month and differ from the
    expected clamping policy.
-   **Using UTC dates for local reports:** Local day and month
    boundaries must be calculated in the target time zone.

## Choosing the Appropriate Type

  -----------------------------------------------------------------------
  Requirement                         Recommended Type or Facility
  ----------------------------------- -----------------------------------
  Represent a year                    `year`

  Represent a month                   `month`

  Represent a day of the month        `day`

  Represent a weekday                 `weekday`

  Represent a complete date           `year_month_day`

  Represent a year and month          `year_month`

  Represent the last day of a month   `year_month_day_last`

  Represent the nth weekday of a      `year_month_weekday`
  month                               

  Represent the last weekday of a     `year_month_weekday_last`
  month                               

  Calculate a number of days          `sys_days` and `days`

  Represent a local date without a    `local_days` or `local_time`
  time zone                           

  Represent a time with a time zone   `zoned_time`

  Split a duration into time-of-day   `hh_mm_ss`
  components                          

  Format dates and times              `std::format` with `chrono`
                                      formatters

  Parse a date from a stream          `std::chrono::from_stream`
  -----------------------------------------------------------------------

## Summary

The Calendar facilities in C++20's `std::chrono` provide a standard,
type-safe, and composable model for Gregorian dates. Component types
such as `year_month_day` are suitable for representing and validating
dates; pattern types such as `year_month_weekday` are useful for
floating calendar rules; and `sys_days` enables precise day-based
calculations.

For professional use, keep three decisions separate: date validity,
monthly and yearly arithmetic policies, and the mapping of local time to
system time. Also remember that the standard Calendar models the
Gregorian calendar; holiday rules, business days, and regional policies
must be implemented in other application layers.

## Further Reading

-   [cppreference: Date and time
    library](https://en.cppreference.com/w/cpp/chrono)
-   [cppreference: Header
    `<chrono>`](https://en.cppreference.com/w/cpp/header/chrono)
-   [cppreference:
    `std::chrono::year_month_day`](https://en.cppreference.com/w/cpp/chrono/year_month_day)
-   [cppreference:
    `std::chrono::year_month_day_last`](https://en.cppreference.com/w/cpp/chrono/year_month_day_last)
-   [Microsoft Learn:
    `year_month_day`](https://learn.microsoft.com/en-us/cpp/standard-library/year-month-day-class)
-   [Working Draft, Standard for Programming Language
    C++](https://eel.is/c++draft/time)


---

## 🤝 Contributors

<div align="center">

| `GitHub` | `LinkedIn` | `Email` | `Site` | `Telegram` |
| --- | --- | --- | --- | --- |
| [HadiAb basi](https://github.com/HadiAbbasi) | [Hadi Abbasi](https://www.linkedin.com/in/hadi-abbasi-pro) | [hadi.abbasi.programmer@gmail.com](mailto:hadi.abbasi.programmer@gmail.com) | [hiens.org](https://hiens.org) | [Hadi Abbasi_Pro](https://t.me/HadiAbbasi_Programmer) |

</div>