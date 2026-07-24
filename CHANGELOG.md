# Flow — Changelog

## Bug fix: recursion and forward-referenced closures

**Recursion was completely broken.** A function calling itself — even simple
factorial — crashed with `Runtime error: Undefined variable 'fact'`. The same
bug affected any function that referenced a sibling function or variable
declared *after* it in the file.

Root cause: `Env` (the variable scope) was a plain `HashMap` that got
*cloned* every time a function's closure was captured. Because Flow
registers `put` functions in a first pass, a function's closure was
snapshotted before its own name (or any function defined later) had been
inserted — so it could never see itself.

Fix: `Env` is now a reference-counted, interior-mutable scope
(`Rc<RefCell<Scope>>`). A closure captures a *handle* to its enclosing scope
rather than a copy, so later writes to that scope — including registering
sibling functions — are visible through every closure that captured it. This
is the standard approach used by most tree-walking interpreters and is what
makes recursion, mutual recursion, and forward references work correctly.

As a side effect, this also fixed a **variable-scoping bug** where
`>> x = value;` inside a function or block could leak into and overwrite a
variable of the same name in an outer scope. `>> name = value` now always
declares (or shadows) in the current scope; a plain `name = value` (no `>>`)
still walks up to the nearest existing binding, so mutating an outer
variable from inside a loop or `if` block continues to work as before.

Because recursion now actually works, the interpreter runs on a dedicated
thread with a 256MB stack (instead of the default ~8MB) so moderately deep
recursive Flow programs don't overflow the native call stack.

## 1. User input

```
>> name = ask("What is your name?");
"Hello, " + name << show;
```

`ask()` (no prompt) also works — it just reads a line without printing
anything first.

## 2. File I/O

```
write("notes.txt", "some text");     ~~ overwrite ~~
append("output.txt", "more text");   ~~ append ~~
>> text = read("notes.txt");
exists("notes.txt") << show;         ~~ true / false ~~
delete_file("notes.txt");
```

`read`/`write`/`append`/`exists`/`delete_file` resolve relative paths
against the process's current working directory (same as most languages'
file APIs). This is deliberately different from `import`, see below.

## 3. Standard library

```
length(x)        ~~ works on letters, boxes, and maps ~~
upper(word) / lower(word)
sqrt(n) / abs(n)
min(a, b, ...) / max(a, b, ...)   ~~ also accepts min(box [...]) ~~
round(n) / floor(n) / ceil(n)
random()          ~~ float 0.0–1.0 ~~
random(lo, hi)     ~~ random integer, inclusive ~~
type(x)           ~~ "number" | "letter" | "bool" | "box" | "map" | "function" | "unit" ~~
to_number(x) / to_letter(x)
```

## 4. Array methods

```
push(arr, value)
pop(arr)
remove(arr, index)
insert_at(arr, index, value)
contains(arr, value)
length(arr)
keys(map) / values(map)
```

These mutate the array/map in place (boxes and maps are now reference
types under the hood — see "Boxes and maps are now shared" below).

## 5. String functions

```
length(word)
slice(word, start, end)
contains(word, "sub")
replace(word, old, new)
split(word, ",")
join(arr, ",")
trim(word)
```

## 6. Collection iteration — `each`

```
each value in numbers {
    value << show;
}

each key, value in person {
    key + ": " + value << show;
}

each i, value in numbers {       ~~ index + value, for arrays ~~
    i << show;
}

each c in "flow" {               ~~ works over letters too ~~
    c << show;
}
```

`break` works inside `each`, same as `loop`/`again`/`aslong`.

**Known limitation:** `index` and `letter` are reserved keywords (used for
`arr index i` and `letter >> expr`), so they can't be used as the loop
variable name in `each`. Use `i`/`idx` or `c`/`ch` instead — a small
consequence of Flow's existing keyword-as-syntax design, not something new.

## 7. Modules — `import`

```
import "mathutils.flow";
square(6) << show;
```

Imported files run in the importing file's top-level scope (like a textual
include), so `put` functions and `always` constants defined in the module
become directly available — no namespacing yet (e.g. no `math.square(...)`).
Relative import paths resolve against the *importing file's own directory*,
not the current working directory, so a project's files can `import` each
other regardless of where you run `flow` from. Importing the same file twice,
or in a cycle, is safe — it's a no-op after the first load.

There's no `math`/`random` built-in module namespace since the equivalent
functions (`sqrt`, `random`, etc.) are already available globally as
built-ins — write your own `.flow` module and `import` it for reusable
project code.

## 8. Constants — `always`

```
always PI = 3.14159;
PI << show;
PI = 4;   ~~ Runtime error: Cannot reassign constant 'PI' ~~
```

## 9. Error handling — `attempt` / `catch`

```
attempt {
    >> result = 10 / 0;
} catch err {
    "Something went wrong: " + err << show;
}
```

`catch` can also omit the variable (`catch { ... }`) if you don't need the
error message. Errors thrown anywhere in the `attempt` block — including
deep inside nested function calls — are caught. `take` inside `attempt`/
`catch` still works as a function return.

## Other changes

- **`and` / `or`** are now real, working, short-circuiting operators. They
  existed in the AST before but nothing in the lexer or parser ever produced
  them, so `and`/`or` used to be a parse error.
- **Boxes (arrays) and maps are now shared/reference types**, not
  copy-on-write values. Passing a box into a function and mutating it there
  now affects the original — needed for `push`/`pop`/`remove`/field
  mutation to work at all, and matches how arrays/objects behave in most
  scripting languages.
- **`arr index i = value;`** and **`m.field = value;`** now work as
  assignment targets, for in-place mutation without rebuilding the whole
  array or map.
- Equality (`==`/`!=`) on boxes and maps now compares contents, not
  reference identity.

## Deliberately not added (per discussion)

Classes, generics, static typing, operator overloading, and macros were
intentionally left out to keep Flow's syntax surface small — the standard
library, array/string methods, and `each` loops give most of that value
without the complexity.
