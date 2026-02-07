# Type Mappings

Nuwa SDK automatically maps Nim types to Python type annotations in generated `.pyi` files.

## Supported Types

| Nim Type                      | Python Type  | Notes                          |
| ----------------------------- | ------------ | ------------------------------ |
| `int`                         | `int`        | Standard integer               |
| `int32`                       | `int`        | 32-bit integer                 |
| `int64`                       | `int`        | 64-bit integer                 |
| `float`                       | `float`      | Standard float                 |
| `float32`                     | `float`      | 32-bit float                   |
| `float64`                     | `float`      | 64-bit float                   |
| `string`                      | `str`        | String                         |
| `bool`                        | `bool`       | Boolean                        |
| `void`                        | `None`       | No return value                |
| `seq[T]`                      | `list[T]`    | List of type T                 |
| `array[N, T]`                 | `list[T]`    | Fixed-size array → list        |
| Other types                   | `Any`        | Fallback for unsupported types |

## Examples

### Basic Types

```nim
proc intExample(x: int): int {.nuwa_export.} =
  return x * 2
# Generated: def int_example(x: int) -> int

proc floatExample(x: float): float {.nuwa_export.} =
  return x * 1.5
# Generated: def float_example(x: float) -> float

proc stringExample(s: string): string {.nuwa_export.} =
  return "Hello, " & s
# Generated: def string_example(s: str) -> str

proc boolExample(flag: bool): bool {.nuwa_export.} =
  return not flag
# Generated: def bool_example(flag: bool) -> bool

proc voidExample(): void {.nuwa_export.} =
  echo "Side effect only"
# Generated: def void_example() -> None
```

### Collection Types

```nim
proc listExample(numbers: seq[int]): int {.nuwa_export.} =
  ## Sum a list of integers
  result = 0
  for n in numbers:
    result += n
# Generated: def list_example(numbers: list[int]) -> int

proc arrayExample(arr: array[5, float]): float {.nuwa_export.} =
  ## Average of 5 floats
  var sum = 0.0
  for x in arr:
    sum += x
  return sum / 5.0
# Generated: def array_example(arr: list[float]) -> float
```

### Complex Types (Fallback to Any)

```nim
import std/tables

proc tableExample(data: Table[string, int]): int {.nuwa_export.} =
  ## Complex types map to Any
  return data.len
# Generated: def table_example(data: Any) -> int

type
  CustomObj = object
    x: int
    y: int

proc customExample(obj: CustomObj): int {.nuwa_export.} =
  ## Custom types map to Any
  return obj.x + obj.y
# Generated: def custom_example(obj: Any) -> int
```

## Type Conversion with nimpy

While `nuwa_export` generates type stubs, the actual type conversion is handled by the `nimpy` library. nimpy supports additional conversions not reflected in stubs:

### nimpy Supported Conversions

| Nim Type             | Python Type | nimpy Support |
| -------------------- | ----------- | ------------- |
| `int`                | `int`       | ✅            |
| `float`              | `float`     | ✅            |
| `string`             | `str`       | ✅            |
| `bool`               | `bool`      | ✅            |
| `seq[T]`             | `list`      | ✅            |
| `Table[K, V]`        | `dict`      | ✅            |
| `tuple`              | `tuple`     | ✅            |
| `object` types       | `object`    | ✅            |

For complex types, you may need to manually add type annotations in your `.pyi` files:

```python
# Manual type hint for complex types
from typing import Dict

def process_data(data: Dict[str, int]) -> int:
    ...
```

## Naming Conventions

Nuwa automatically converts `camelCase` Nim function names to `snake_case` Python names:

```nim
proc myFunction(x: int): int {.nuwa_export.} =
  return x
# Generated: def my_function(x: int) -> int
```

## Generics

Generic types are supported but map to `Any` in Python:

```nim
proc genericExample[T](x: T): T {.nuwa_export.} =
  return x
# Generated: def generic_example(x: Any) -> Any
```
