## Features of the Flow Programming Language

Flow is a **dynamically typed**, **interpreted** scripting language with a
unique, minimal syntax, designed to be friendly for beginners without
copying the look of Python, JavaScript, or any other existing language.

This document covers everything Flow supports, including the new additions
below.

---

## What's new

- **Real block scoping.** Variables created inside a `check`/`loop`/`again`/
  `aslong`/`group` block now disappear when the block ends, while
  assignments to a variable that already exists *outside* the block still
  update that outer variable. (Previously, blocks didn't have their own
  scope at all — this also happened to fix a bug where a function couldn't
  see a variable declared after the function itself.)
- **String indexing.** The same `index` word you already use for arrays now
  works on words (strings) too: `"hello" index 0` → `"h"`. Indexing is by
  character, so it works correctly with emoji and other multi-byte
  characters, not just plain ASCII.
- **Chained dot access.** `a.b.c.d` and mixing dots with `index` (e.g.
  `(party index 0).name`) both work for arbitrarily nested maps.
- **Random numbers.** A new `roll` keyword: `roll()` gives a random decimal
  between 0 and 1, and `roll(low, high)` gives a random whole number between
  `low` and `high`, inclusive — handy for dice, loot tables, and games.
- **Much better error messages.** Every error now names the exact line (and
  for syntax errors, the column) it happened on, shows the source line with
  a caret pointing at the problem, and — for undefined variables, unknown
  functions, and missing map fields — offers a "did you mean '...'?"
  suggestion when there's an obvious typo.
- **Bug fixes:**
  - Functions can now see variables declared anywhere before they're
    *called* (not just before they're *defined*), which fixes recursive
    functions that rely on a sentinel value declared later in the file.
  - `%` (remainder) now works with decimal numbers, not just whole numbers.
  - `true`/`false` can now be cast with `number >> ...` (`true` → `1`,
    `false` → `0`).

---

### Output & Formatting
- `value << show;` – prints value and appends a newline.
- `value << show:` – prints value without a newline.
- Chaining with spacing helpers: `Add.space`, `Add.tab`, `Add.newline`.
  - You can specify a count: `Add.space.3`, `Add.tab.2`, `Add.newline.2`.
- Combine multiple outputs on one line:
  `"Name:" << show: Add.tab << "Freddy" << show;`

### Comments
- Single‑line: `~~ comment text`
- Multi‑line:
  ```
  ~~~
  everything here is ignored
  ~~~
  ```

### Variables & Assignment
- `>> name = value;` – creates or updates a variable.
- The `>>` prefix is optional but idiomatic.
- **Scoping:** a variable created inside `{ }` (in a `check`, `loop`,
  `again`, `aslong`, or `group` block) only exists inside that block.
  Assigning to a name that already exists in an *outer* scope updates that
  outer variable instead of creating a new local one — so loops can freely
  accumulate into a variable declared before the loop starts.

### Data Types
- **Numbers**: integers (`123`) and decimals (`3.14`). Arithmetic always
  produces numbers (division returns a float).
- **Words (strings)**: text enclosed in double quotes (`"hello"`).
- **Booleans**: `true` and `false` (result of comparisons).
- **Arrays**: `box [1, 2, 3]` – accessed with `index`: `arr index 0`.
- **Maps**: `map { key: "value", age: 25 }` – fields accessed with dot:
  `person.name`. Dot access chains: `a.b.c`.
- **Unit**: the empty value returned by some statements (not directly
  writable).

### Indexing
- Arrays: `arr index 0`
- Words (strings), by character: `"hello" index 0` → `"h"`
- Indexing chains with dot access: `(people index 0).name`

### Type Casts
- `number >> expression` – forces the result to be treated as a number
  (e.g., `number >> "123"` → `123`; `number >> true` → `1`).
- `letter >> expression` – forces the result to be treated as a string
  (e.g., `letter >> 789` → `"789"`).

### Arithmetic & String Concatenation
- Operators: `+`, `-`, `*`, `/`, `%` (remainder — works on whole numbers and
  decimals).
- Mixing a string and a number with `+` automatically converts the number to
  a string and concatenates: `"apple" + 5` → `"apple5"`.

### Comparison & Logic
- Comparison: `==`, `!=`, `<`, `>`, `<=`, `>=`. Return `true` or `false`.
- Logical: `and`, `or` – though not yet fully wired in the parser/interpreter
  (reserved for future).

### Conditionals
```
check condition then {
    ...
} else {
    ...
}
```
- Single‑line branches work: `check x > 2 then "big" << show; else "small" << show;`

### Loops
- **Infinite loop with break**:
  ```
  loop {
      ...
      check condition then break;
  }
  ```
- **Fixed repetition**: `again n { ... }` – runs the block `n` times.
- **Conditional loop**: `aslong condition { ... }` – repeats while the
  condition is `true`.

### Functions
- Defined with `put` and parameters in parentheses:
  ```
  put add(a, b) {
      take a + b;
  }
  ```
- Return a value with `take expression`.
- Calls: `add(3, 4)`
- Function parameters and any variables created inside the function body are
  local to that call and never leak into the caller's scope.

### Random Numbers
- `roll()` – a random decimal between 0 (inclusive) and 1 (exclusive).
- `roll(low, high)` – a random whole number between `low` and `high`,
  inclusive. `low` must be less than or equal to `high`.

### Groups
- `group.name { ... }` – a block of code that executes immediately when
  encountered. Useful for organisation. Like other blocks, it has its own
  scope.

### Error Handling
- The interpreter reports runtime errors like undefined variables (with
  "did you mean" suggestions), type mismatches in arithmetic, division by
  zero, index out of bounds, missing map fields, and incorrect use of `take`
  outside a function — each tagged with the line it happened on.
- The lexer/parser detect syntax errors (unterminated strings, unexpected
  tokens, missing semicolons, unmatched brackets) and print the offending
  source line with a caret pointing at the exact spot.

### What Flow Does **Not** Have (Yet)
These are intentionally deferred, not forgotten:
- File I/O (`read`, `write`) — planned, would make Flow practical for real
  scripts.
- User input (`ask`) — planned, for interactive scripts.
- `slice` / `glue` (split/join for strings and arrays) — planned.
- Each‑style loops, string interpolation, and a package/standard-library
  system are deliberately left out for now to keep Flow small, minimal, and
  distinct from mainstream languages.
- Static typing (all checks happen at runtime).
- Multi‑threading or asynchronous code.

---

### Quick Reference Card

| Feature              | Syntax Example                                  |
|-----------------------|-------------------------------------------------|
| Print                 | `"Hello" << show;`                              |
| Print no newline      | `"Hello" << show:`                              |
| Add space              | `"Hi" << show: Add.space << "there!" << show;`  |
| Variable               | `>> x = 10;`                                    |
| Conditional             | `check x > 5 then { … } else { … }`             |
| Repeat block            | `again 3 { "Hi" << show; }`                     |
| While loop               | `aslong i < 10 { … }`                           |
| Infinite loop            | `loop { … ; check done then break; }`           |
| Function                 | `put double(x) { take x * 2; }`                 |
| Function call            | `double(7) << show;`                            |
| Array                    | `>> arr = box [1, 2, 3];`                       |
| Access array             | `arr index 0 << show;`                          |
| String index (new)       | `"hello" index 0 << show;`                      |
| Map                      | `>> obj = map { a: 1, b: "two" };`              |
| Access field              | `obj.b << show;`                                |
| Chained dot access (new)  | `obj.a.b.c << show;`                            |
| Random float (new)         | `roll() << show;`                               |
| Random int in range (new)  | `roll(1, 6) << show;`                           |
| Cast to number            | `number >> "42" << show;`                       |
| Cast to string             | `letter >> 100 << show;`                        |
| Comments                   | `~~ single`  /  `~~~ multi ~~~`                  |

---

Flow is small but expressive – perfect for learning how languages work or
for embedding simple scripting into a larger project. If you dream up a new
feature, the interpreter is modular and can be extended with ease.