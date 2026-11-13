<div align="center">

[🇺🇸 English](./fallthrough.md) | [🇮🇷 فارسی](../../fa/cpp17/fallthrough.md)

</div>

----

# `[[fallthrough]]` in C++17

Introduced in **C++17** via proposal **P0188R1** ("Wording for `[[fallthrough]]` attribute"), `[[fallthrough]]` is a standard statement attribute that explicitly documents and permits an intentional fallthrough between consecutive `case` or `default` labels inside a `switch` statement.

Inherited from C, the default semantic of `switch` statements allows execution to cascade from one label to the next unless interrupted by a control-flow jump (such as `break`, `return`, `throw`, or `goto`). While intentional fallthrough is a powerful technique for implementing state machines, byte parsing, and Duff's Device algorithms, accidental fallthrough (often caused by a forgotten `break`) is a notorious source of severe logic errors. Compilers diagnose unannotated fallthrough under flags like `-Wimplicit-fallthrough` / `/we5262`. The `[[fallthrough]]` attribute eliminates these warnings by explicitly certifying programmer intent at compile time with zero runtime cost.

```cpp
enum class PacketType { CompressedHeader, RawPayload, Heartbeat };

void route_packet(PacketType type, Buffer& stream) {
    switch (type) {
        case PacketType::CompressedHeader:
            stream.decompress_in_place();
            [[fallthrough]]; // Explicitly intentional: falls into RawPayload processing
            
        case PacketType::RawPayload:
            stream.parse_records();
            break;

        case PacketType::Heartbeat:
            stream.update_keepalive();
            break;
    }
}
```

---

## Table of Contents

1. [Motivation & The Dangers of Implicit Cascades](#1-motivation--the-dangers-of-implicit-cascades)
2. [Language Mechanics & Placement Rules](#2-language-mechanics--placement-rules)
3. [Systems Use Cases & Idiomatic Patterns](#3-systems-use-cases--idiomatic-patterns)
   - [A. Cascading Tiered Operations & Pipeline Processing](#a-cascading-tiered-operations--pipeline-processing)
   - [B. High-Performance Packet Parsing & Protocol State Machines](#b-high-performance-packet-parsing--protocol-state-machines)
   - [C. High-Throughput Unrolling & Duff's Device](#c-high-throughput-unrolling--duffs-device)
4. [Assembly & Code Generation: Zero-Cost Proof](#4-assembly--code-generation-zero-cost-proof)
5. [Feature-Test Macros & Legacy Workarounds](#5-feature-test-macros--legacy-workarounds)
6. [Compilation Traps, Constraints, and Anti-Patterns](#6-compilation-traps-constraints-and-anti-patterns)
7. [Contributors](#7-contributors)

---

## 1. Motivation & The Dangers of Implicit Cascades

In classic C and earlier C++, distinguishing between intentional fallthrough and an accidental omitted `break` was impossible for static analyzers without heuristic comment parsing (such as `/* FALLTHROUGH */`).

```cpp
// Pre-C++17 Dilemma:
switch (status) {
    case State::Pending:
        initialize_worker();
        // Was 'break;' forgotten by mistake, or is this cascade intentional?
    case State::Ready:
        dispatch_job();
        break;
}
```

Modern compilers enable rigorous diagnostics to catch forgotten break statements (`-Wimplicit-fallthrough` in GCC/Clang, warning level 4 / `C5262` in MSVC). Without a standardized attribute, developers were forced to rely on compiler-specific macros or magic comments:
- GCC/Clang: `__attribute__((fallthrough));`
- Regex-based comments: `// [[fallthrough]]` or `/* FALLTHROUGH */` (fragile and easily broken by formatting tools)

`[[fallthrough]]` establishes a standard-conforming, portable keyword recognized by the compiler front end.

---

## 2. Language Mechanics & Placement Rules

The C++ standard imposes strict syntactic constraints on where `[[fallthrough]];` can appear:

1. **Only Within a `switch` Statement**: It cannot be used in `if`, `while`, or standalone scopes.
2. **Applies to Null Statements**: Syntactically, `[[fallthrough]]` is an attribute attached to an empty statement (null statement terminating in a semicolon: `[[fallthrough]];`).
3. **Must Precede a `case` or `default` Label**: It must sit on a path of execution that immediately leads into the subsequent label.
4. **Cannot Be the Final Statement**: It is ill-formed to place `[[fallthrough]];` after the final `case` or `default` label of a `switch` block because there is no subsequent label to fall through to.

```text
+-----------------------------------------------------------------------------------------+
|                               [[fallthrough]] Placement Flow                            |
+-----------------------------------------------------------------------------------------+
|   switch (condition) {                                                                  |
|       case A:                                                                           |
|           action_a();                                                                   |
|           [[fallthrough]];  // OK: Sits at end of block, followed by label              |
|       case B:                                                                           |
|           action_b();                                                                   |
|           break;                                                                        |
|       case C:                                                                           |
|           action_c();                                                                   |
|           [[fallthrough]];  // ILL-FORMED: Last case in switch; nothing to fall into!   |
|   }                                                                                     |
+-----------------------------------------------------------------------------------------+
```

### Ill-Formed Example:
```cpp
switch (val) {
    case 1:
        if (check()) {
            [[fallthrough]]; // ERROR: May not lead directly to case 2 if condition is false
        }
        do_fallback();
    case 2:
        break;
}
```

---

## 3. Systems Use Cases & Idiomatic Patterns

### A. Cascading Tiered Operations & Pipeline Processing
When handling hierarchy levels (such as permission escalation or logging levels), subsequent lower levels often require all the actions of the preceding tiers:

```cpp
enum class Severity { Emergency, Alert, Notice };

void log_event(Severity level, const std::string& msg) {
    switch (level) {
        case Severity::Emergency:
            page_oncall_engineer(msg);
            [[fallthrough]]; // Emergencies must also trigger alerting and audit records
        case Severity::Alert:
            broadcast_pager_duty(msg);
            [[fallthrough]]; // Alerts also append to the audit log
        case Severity::Notice:
            append_disk_audit_log(msg);
            break;
    }
}
```

---

### B. High-Performance Packet Parsing & Protocol State Machines
Network frames or protocol decoders (e.g., HTTP/2 frame framing or TLS records) often transition through consecutive parser phases with minimal branching latency:

```cpp
enum class DecoderState { ReadEnvelope, DecryptBody, DecompressPayload };

void stream_decode(DecoderState state, PacketContext& ctx) {
    switch (state) {
        case DecoderState::ReadEnvelope:
            ctx.parse_header();
            [[fallthrough]];
            
        case DecoderState::DecryptBody:
            ctx.decrypt_aes_gcm();
            [[fallthrough]];
            
        case DecoderState::DecompressPayload:
            ctx.lz4_decompress();
            break;
    }
}
```

---

### C. High-Throughput Unrolling & Duff's Device
Serial-to-memory copies or unrolled hardware registers frequently use Duff's Device patterns to handle leftover count residues (`count % 8`):

```cpp
void send_to_device(volatile uint32_t* register_port, const uint32_t* data, size_t count) {
    size_t n = (count + 7) / 8;
    switch (count % 8) {
        case 0: do { *register_port = *data++; [[fallthrough]];
        case 7:      *register_port = *data++; [[fallthrough]];
        case 6:      *register_port = *data++; [[fallthrough]];
        case 5:      *register_port = *data++; [[fallthrough]];
        case 4:      *register_port = *data++; [[fallthrough]];
        case 3:      *register_port = *data++; [[fallthrough]];
        case 2:      *register_port = *data++; [[fallthrough]];
        case 1:      *register_port = *data++;
                } while (--n > 0);
    }
}
```

---

## 4. Assembly & Code Generation: Zero-Cost Proof

`[[fallthrough]]` is strictly an AST/semantic attribute for compiler static analyzers. It generates **no runtime code, no jump instructions, and no branch penalties**.

Let us examine GCC 13 (`-O2`):

```cpp
void process_step(int step, volatile int* target) {
    switch (step) {
        case 1:
            *target += 10;
            [[fallthrough]];
        case 2:
            *target += 20;
            break;
        default:
            *target = 0;
            break;
    }
}
```

### Disassembly (x86-64):
```assembly
process_step(int, int volatile*):
    cmp     edi, 1
    je      .L_STEP1
    cmp     edi, 2
    je      .L_STEP2
    mov     DWORD PTR [rsi], 0
    ret
.L_STEP1:
    mov     eax, DWORD PTR [rsi]
    add     eax, 10
    mov     DWORD PTR [rsi], eax
    ; Direct linear fallthrough without extra jmp:
.L_STEP2:
    mov     eax, DWORD PTR [rsi]
    add     eax, 20
    mov     DWORD PTR [rsi], eax
    ret
```

The compiler avoids generating a `jmp` instruction between `.L_STEP1` and `.L_STEP2`. The attribute cleanly suppresses the warning while allowing the machine instructions to cascade linearly.

---

## 5. Feature-Test Macros & Legacy Workarounds

When maintaining cross-standard or legacy C++11/C++14 codebases alongside C++17, portability can be ensured via `__has_cpp_attribute`:

```cpp
#if defined(__has_cpp_attribute)
    #if __has_cpp_attribute(fallthrough) >= 201603L
        #define FALLTHROUGH [[fallthrough]]
    #endif
#endif

#ifndef FALLTHROUGH
    #if defined(__GNUC__) || defined(__clang__)
        #define FALLTHROUGH __attribute__((fallthrough))
    #else
        #define FALLTHROUGH ((void)0)
    #endif
#endif

void execute_step(int code) {
    switch (code) {
        case 1:
            initiate();
            FALLTHROUGH;
        case 2:
            finalize();
            break;
    }
}
```

---

## 6. Compilation Traps, Constraints, and Anti-Patterns

### 1. The Trailing Semicolon Requirement
Unlike function attributes, `[[fallthrough]]` behaves like an isolated statement and **must be followed by a semicolon**:

```cpp
case 1:
    action();
    [[fallthrough]]  // SYNTAX ERROR: Expected ';' after attribute [[fallthrough]]
case 2:
```

### 2. Not Allowed on Empty Consecutive Cases
When cases are stacked together without any executable statements between them, no warning is generated, and `[[fallthrough]];` is redundant:

```cpp
switch (val) {
    case 1:
    case 2: // Perfectly valid; no warning generated for consecutive empty case labels.
        handle_one_or_two();
        break;
}
```

### 3. Misplacement inside Conditional Blocks
Placing `[[fallthrough]];` where the path of execution does not directly hit the label is diagnosed as ill-formed:

```cpp
switch (mode) {
    case 10:
        if (is_valid()) {
            run_step();
            [[fallthrough]]; // ILL-FORMED: If is_valid() is false, control skips this statement!
        }
        break;
    case 20:
        run_fallback();
        break;
}
```

---

## 7. Contributors

| GitHub | LinkedIn | Email | Site | Telegram |
|---|---|---|---|---|
| [Ordikhani](https://github.com/Ordikhani) | | [Ordikhani](mailto:ordikhanifateme@gmail.com) | | [Ordikhani](https://t.me/OrdikhaniFateme) |