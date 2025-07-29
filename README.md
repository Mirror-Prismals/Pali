
---

# Pali Specification v0.1

## 1 Vision & Design Pillars

Pali is a **general‑purpose, systems‑level** language that targets C/C++/Rust‑class performance while enforcing *total semantic clarity*:

* **High performance, slow compiler.** Heavy optimisation is embraced; compile‑time is traded for predictable run‑time.
* **Safety without a GC.** Memory and thread safety are achieved through explicit ownership / lifetimes, never through a garbage collector.
* **Easy to grasp, hard to master.** Newcomers can read Pali quickly; mastery demands discipline—“like learning guitar.”
* **No foot‑guns.** Constructs that routinely cause inefficiency (*e.g.* generic `for` loops) are *omitted* or replaced with problem‑specific forms.
* **Everything explicit.** Nothing happens unless you *say* it happens.

---

## 2 Source‑Layer Architecture

| Layer         | Extension | Purpose                                                 | Mirror Rule                                         |
| ------------- | --------- | ------------------------------------------------------- | --------------------------------------------------- |
| **Core Pali** | `.core`   | Low‑level, performance‑critical truth.                  | *May* mirror High but is not required.              |
| **High Pali** | `.high`   | Ergonomic façade that always reflects Core definitions. | **Must** mirror a named `.core` file via `$mirror`. |

```pali
// pali.high
$define pali.high
$mirror pali.core

main = pali
print(main)

// pali.core
$define pali.core

$main()                 // entry‑point – must be first line
#main" = $str pali
$print(main)
```

---

## 3 Lexical & Visual Grammar

| Sigil      | Meaning                                                             |
| ---------- | ------------------------------------------------------------------- |
| `$`        | Prefix for **keywords / types** (`$let`, `$int`)                    |
| `"`        | Trailing marker for **user names** (`main"`, `x"`)                  |
| `#`        | Marks a **callable** reference (`#double"`)                         |
| `#@`       | Same as `#` but **private** (`#@helper"`)                           |
| `<~*^`     | Directional *return* arrow, used inside types or expressions        |
| `f*`, `+-` | Optional visual cues for advanced signatures (not required in v0.1) |

> **Rule of thumb:** if it starts with `$`, the compiler owns it; if it ends in `"`, *you* own it; if it starts with `#`, you can **call** it.

---

## 4 Program Structure

* **Entry‑point:** the first line of a `.core` file **must** be

  ```pali
  $main()
  ```

  which immediately begins execution.
  `main"()` (with a trailing quote) is just another function.

* Execution flows **top‑down**—there is no implicit REPL‑style evaluation.

---

## 5 Types

* Types are ordinary names prefixed with `$`: `$int`, `$str`, `$float`, `$bool`, `$void`, etc.
* A *return type* is written with the arrow suffix:

```pali
$int<~*^          // “value of type int returning out of here”
```

---

## 6 Variables & Assignment

```pali
$let x" = 7                     // binds the literal 7 to name x"
$let frag" = \n x * 2; <~*^ \n  // stores an unevaluated expression
```

* `=` **never executes**; it merely binds.
* Writing a bare name (**no #**) has zero side‑effects.
* To bind a callable with preset arguments:

```pali
$let bomb_ref"(x = 1, y = 2) = #bomb"
```

---

## 7 Functions & Callability

### 7.1 Declaration (Core Pali)

```pali
#double"($int y) = $int<~*^ y * 2;
```

* `#` makes the name callable.
* Parameters are typed; parameter identifiers **do not** carry the trailing quote.
* The `$int<~*^` immediately before the body declares the return type.

### 7.2 Calling

* Direct call: `#double"(5)`
* **Variables are never callable.**

  ```pali
  $let f" = #double"
  f"(5)          // ERROR – variables cannot be invoked
  ```

---

## 8 Callability Namespace Rules

| Kind             | Example     | Callable? | Mutable? | Notes           |
| ---------------- | ----------- | --------- | -------- | --------------- |
| Function def     | `double"`   | ✗         | ✓        | Plain symbol    |
| Callable ref     | `#double"`  | ✓         | ✗        | Use `#` to call |
| Private ref      | `#@helper"` | ✓ (scope) | ✗        | File‑private    |
| Variable         | `x"`        | ✗         | ✓        | Never callable  |

---

## 9 Modules & Mirroring

```pali
$pali *.high = math
$pali *.core = math
$mirror math.core      // required in math.high
```

* A `.high` file **must** mirror exactly one `.core` file.
* Attempting to mirror a wildcard produces a compile‑time error.

---

## 10 Forbidden & Deferred Constructs

* Generic `for` / `while` loops are **omitted** in v0.1.
  Expect domain‑specific iterators like `$search`, `$map`, `$fold`.
* Macros, generics, and module linkage tooling (`$mirror` granularity, `$link`) are drafted but **not yet specified**.

---

## 11 Minimum “Hello, Pali!”

```pali
// hello.core
$define hello.core

$main()
#main" = $str<~*^ "Hello, Pali!";
$print(#main"())
```

---

**End of v0.1 spec** – ready for critique, expansion, or prototype work.
