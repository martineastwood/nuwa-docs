# nuwa_export Macro

The `{.nuwa_export.}` pragma is the primary way to export Nim functions to Python with automatic type stub generation.

## Basic Usage

```nim
import nuwa_sdk

proc add(a: int, b: int): int {.nuwa_export.} =
  ## Add two integers together
  return a + b

proc greet(name: string): string {.nuwa_export.} =
  ## Greet a person by name
  return "Hello, " & name

proc isPositive(x: int): bool {.nuwa_export.} =
  ## Check if a number is positive
  return x > 0
```

## Docstrings

Use Nim docstring comments (##) - they will be included in the generated `.pyi` files:

```nim
proc calculate(x: float, y: float): float {.nuwa_export.} =
  ## Calculate the hypotenuse of a right triangle
  ##
  ## This function uses the Pythagorean theorem to compute
  ## the length of the hypotenuse given the two other sides.
  return sqrt(x*x + y*y)
```

## How It Works

The `nuwa_export` macro:

1. **Inspects your function at compile time**
   - Extracts parameter names and types
   - Extracts return type
   - Extracts docstring comments

2. **Maps Nim types to Python types**
   - `int` → `int`
   - `float` → `float`
   - `string` → `str`
   - `bool` → `bool`
   - `void` → `None`

3. **Emits metadata to compiler output**
   - Outputs JSON as `NUWA_STUB:` lines
   - Includes function signature and docstring
   - Captured by `nuwa build` process

4. **Generates `.pyi` files**
   - Creates type stub in your package directory
   - Enables IDE autocomplete and type checking
   - Follows PEP 484 type hint standards

## Generated Type Stubs

For this Nim code:

```nim
proc add(a: int, b: int): int {.nuwa_export.} =
  ## Add two integers together
  return a + b
```

Nuwa generates this `.pyi` file:

```python
def add(a: int, b: int) -> int:
    """Add two integers together"""
    ...
```

## Default Parameters

You can use Nim's default parameters:

```nim
proc greet(name: string; greeting: string = "Hello"): string {.nuwa_export.} =
  ## Greet someone with a custom greeting
  return greeting & ", " & name
```

## Varargs

For variable arguments, use `openArray`:

```nim
proc sum(numbers: varargs[int]): int {.nuwa_export.} =
  ## Sum all provided numbers
  result = 0
  for n in numbers:
    result += n
```

## Common Patterns

### Multiple Return Values

Use tuples to return multiple values:

```nim
proc divide(a: float, b: float): tuple[quotient: float, remainder: float] {.nuwa_export.} =
  ## Divide two numbers and return quotient and remainder
  let q = a / b
  let r = a mod b
  return (quotient: q, remainder: r)
```

### Optional Parameters

Use `Option` from nim's standard library (requires manual handling):

```nim
import std/options

proc findUser(id: int): Option[string] {.nuwa_export.} =
  ## Find a user by ID, returns None if not found
  # Implementation here
  if id == 1:
    return some("Alice")
  else:
    return none(string)
```

### Exporting Procedures from Different Modules

When using multi-file projects with `include`, all included procedures marked with `{.nuwa_export.}` will be exported:

```nim
# nim/my_lib.nim
import nuwa_sdk
include helpers

proc mainFunction(): int {.nuwa_export.} =
  return helperFunction()

# nim/helpers.nim
proc helperFunction(): int {.nuwa_export.} =
  return 42
```

Both `mainFunction` and `helperFunction` will be exported to Python.
