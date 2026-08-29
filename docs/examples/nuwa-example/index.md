# Nuwa Example

Example projects demonstrating how to use Nuwa tools.

## Overview

This repository contains example projects showing how to build Python extensions with Nim using the Nuwa toolchain.

## example_project

A comprehensive example project that demonstrates all the key features of building Python extensions with Nim using Nuwa.

### Key Features Demonstrated

- **Zero-configuration build system** - No setup.py or custom build scripts needed
- **Automatic type stubs** (`.pyi` files) - Full IDE autocomplete and type checking
- **Build profiles** - Debug, release, and benchmarking configurations
- **Editable installs** - Rapid development workflow
- **NumPy integration** - Zero-copy array access using the buffer protocol
- **GIL release** - Performance optimization for CPU-intensive operations
- **Python callbacks** - Call Python functions from Nim
- **GitHub Actions CI/CD** - Automated multi-platform wheel building and PyPI publishing
- **pytest integration** - Complete test suite for Nim extensions

### Example Functions

The project includes examples of common Nim-Python interop patterns:

#### Basic Types

```nim
proc greet(name: string): string {.nuwa_export.} =
  return "Hello, " & name & "!"

proc add(a: int, b: int): int {.nuwa_export.} =
  return a + b

proc is_even(n: int): bool {.nuwa_export.} =
  return n mod 2 == 0
```

```python
from example_project import greet, add, is_even

greet("World")  # "Hello, World!"
add(2, 3)       # 5
is_even(4)      # True
```

#### Error Handling

```nim
proc safe_divide(a: float, b: float): float {.nuwa_export.} =
  if b == 0.0:
    raise newException(ValueError, "Cannot divide by zero")
  return a / b
```

```python
from example_project import safe_divide

safe_divide(10.0, 2.0)  # 5.0
safe_divide(10.0, 0.0)  # Raises ValueError: Cannot divide by zero
```

#### Validation Functions

```nim
proc validate_email(email: string): bool {.nuwa_export.} =
  if email.len == 0:
    raise newException(ValueError, "Email cannot be empty")
  if "@" notin email:
    raise newException(ValueError, "Email must contain @ symbol")
  if "." notin email.split("@")[^1]:
    raise newException(ValueError, "Email domain must contain a dot")
  return true
```

```python
from example_project import validate_email

validate_email("user@example.com")  # True
validate_email("invalid")            # Raises ValueError
```

#### Optional Types / Default Values

```nim
proc get_with_default(data: Table[string, int], key: string, default: int): int {.nuwa_export.} =
  if data.hasKey(key):
    return data[key]
  return default
```

```python
from example_project import get_with_default

config = {"timeout": 30}
get_with_default(config, "timeout", 60)  # 30
get_with_default(config, "debug", 0)     # 0
```

#### Enums

Enums in Nim are exposed through accessor functions:

```nim
type
  StatusCode* = enum
    scOk = 200
    scNotFound = 404
    scServerError = 500

proc status_ok*(): int {.nuwa_export.} = int(StatusCode.scOk)
proc status_not_found*(): int {.nuwa_export.} = int(StatusCode.scNotFound)

proc get_status_message(code: int): string {.nuwa_export.} =
  let status = StatusCode(code)
  case status
  of StatusCode.scOk: return "Success"
  of StatusCode.scNotFound: return "Not Found"
  of StatusCode.scServerError: return "Internal Server Error"
```

```python
from example_project import status_ok, get_status_message

get_status_message(status_ok())  # "Success"
```

#### Variadic Functions (Sequences)

```nim
proc sum_all*(numbers: seq[int]): int {.nuwa_export.} =
  result = 0
  for num in numbers:
    result += num

proc concatenate_all*(parts: seq[string]): string {.nuwa_export.} =
  result = ""
  for i, part in parts.pairs:
    if i > 0:
      result.add(" ")
    result.add(part)
```

```python
from example_project import sum_all, concatenate_all

sum_all([1, 2, 3, 4, 5])           # 15
concatenate_all(["Hello", "world"])  # "Hello world"
```

#### Tuples

```nim
proc get_coordinates(): tuple[x: float64, y: float64] {.nuwa_export.} =
  return (x: 10.5, y: 20.3)

proc divide_with_remainder(a: int, b: int): tuple[quotient: int, remainder: int] {.nuwa_export.} =
  if b == 0:
    raise newException(ValueError, "Cannot divide by zero")
  let quotient = a div b
  let remainder = a mod b
  return (quotient: quotient, remainder: remainder)
```

```python
from example_project import get_coordinates, divide_with_remainder

get_coordinates()                  # (10.5, 20.3)
divide_with_remainder(17, 5)       # (3, 2)
```

#### Bytes Handling

```nim
import std/base64

proc reverse_bytes(data: seq[byte]): seq[byte] {.nuwa_export.} =
  result = newSeq[byte](data.len)
  for i in 0..<data.len:
    result[i] = data[data.len - 1 - i]

proc bytes_to_base64(data: seq[byte]): string {.nuwa_export.} =
  return base64.encode(data)

proc base64_to_bytes(encoded: string): seq[byte] {.nuwa_export.} =
  let decoded = base64.decode(encoded)
  result = newSeq[byte](decoded.len)
  for i, b in decoded.pairs:
    result[i] = b.uint8
```

```python
from example_project import reverse_bytes, bytes_to_base64, base64_to_bytes

reverse_bytes(b"hello")                      # b'olleh'
bytes_to_base64(b"Hello, World!")            # "SGVsbG8sIFdvcmxkIQ=="
base64_to_bytes("SGVsbG8sIFdvcmxkIQ==")      # b'Hello, World!'
```

#### Python Set Operations

```nim
proc get_unique_ints*(items: seq[int]): Table[int, bool] {.nuwa_export.} =
  result = initTable[int, bool]()
  for item in items:
    result[item] = true

proc set_union_ints*(a: seq[int], b: seq[int]): seq[int] {.nuwa_export.} =
  var seen = initTable[int, bool]()
  result = @[]
  for item in a:
    if not seen.hasKey(item):
      seen[item] = true
      result.add(item)
  for item in b:
    if not seen.hasKey(item):
      seen[item] = true
      result.add(item)
```

```python
from example_project import get_unique_ints, set_union_ints

get_unique_ints([1, 2, 2, 3, 3, 3])        # {1: True, 2: True, 3: True}
set_union_ints([1, 2, 3], [3, 4, 5])       # [1, 2, 3, 4, 5]
```

#### String Operations

```nim
proc reverse_string(s: string): string {.nuwa_export.} =
  result = newStringOfCap(s.len)
  for i in countdown(s.high, 0):
    result.add(s[i])

proc count_words(text: string): int {.nuwa_export.} =
  let words = text.splitWhitespace()
  return words.len
```

```python
from example_project import reverse_string, count_words

reverse_string("hello")     # "olleh"
count_words("hello world")  # 2
```

#### Collections

```nim
proc calculate_average(numbers: seq[float]): float {.nuwa_export.} =
  if numbers.len == 0:
    raise newException(ValueError, "Cannot calculate average of empty sequence")
  var sum = 0.0
  for num in numbers:
    sum += num
  return sum / float(numbers.len)
```

```python
from example_project import calculate_average

calculate_average([1.0, 2.0, 3.0, 4.0, 5.0])  # 3.0
```

#### Default Parameters

```nim
proc power(base: float, exponent: float = 2.0): float {.nuwa_export.} =
  return pow(base, exponent)
```

```python
from example_project import power

power(5.0)       # 25.0 (default exponent is 2)
power(2.0, 10.0) # 1024.0
```

#### Dictionary / Table Conversions

```nim
proc count_characters(text: string): Table[string, int] {.nuwa_export.} =
  result = initTable[string, int]()
  for ch in text:
    let key = $ch
    if result.hasKey(key):
      result[key] += 1
    else:
      result[key] = 1

proc merge_configs(configs: seq[Table[string, int]]): Table[string, int] {.nuwa_export.} =
  result = initTable[string, int]()
  for config in configs:
    for key, value in config.pairs:
      result[key] = value
```

```python
from example_project import count_characters, merge_configs

count_characters("hello")  # {'h': 1, 'e': 1, 'l': 2, 'o': 1}
merge_configs([{"timeout": 30}, {"timeout": 60, "debug": 1}])  # {'timeout': 60, 'debug': 1}
```

#### Object Types / Records

```nim
type
  Point* = object
    x*: float64
    y*: float64

  Rectangle* = object
    topLeft*: Point
    bottomRight*: Point

proc create_point(x: float64, y: float64): Point {.nuwa_export.} =
  return Point(x: x, y: y)

proc point_distance(p1: Point, p2: Point): float64 {.nuwa_export.} =
  let dx = p2.x - p1.x
  let dy = p2.y - p1.y
  return sqrt(dx * dx + dy * dy)
```

```python
from example_project import create_point, point_distance

p1 = create_point(0.0, 0.0)
p2 = create_point(3.0, 4.0)
point_distance(p1, p2)  # 5.0
```

#### Python Callable Callbacks

```nim
proc apply_callback(values: seq[int], callback: PyObject): seq[int] {.nuwa_export.} =
  result = newSeq[int](values.len)
  for i in 0..<values.len:
    let transformed = callback.callObject(values[i])
    result[i] = transformed.to(int)

proc filter_with_predicate(values: seq[int], predicate: PyObject): seq[int] {.nuwa_export.} =
  result = @[]
  for val in values:
    if predicate.callObject(val).to(bool):
      result.add(val)
```

```python
from example_project import apply_callback, filter_with_predicate

apply_callback([1, 2, 3, 4, 5], lambda x: x * 2)  # [2, 4, 6, 8, 10]
filter_with_predicate([1, 2, 3, 4, 5, 6], lambda x: x % 2 == 0)  # [2, 4, 6]
```

#### NumPy Arrays (Zero-Copy)

Reading NumPy arrays without copying:

```nim
proc numpy_array_sum(arr: PyObject): int64 {.nuwa_export.} =
  var npArr = asNumpyArray(arr, int64)
  result = 0
  for val in items(npArr):
    result += val
```

```python
import numpy as np
from example_project import numpy_array_sum

arr = np.array([1, 2, 3, 4, 5], dtype=np.int64)
numpy_array_sum(arr)  # 15
```

#### NumPy with GIL Release

Optimized operations that release the Python GIL:

```nim
proc numpy_array_sum_fast(arr: PyObject): int64 {.nuwa_export.} =
  var npArr = asNumpyArray(arr, int64)
  let n = npArr.len
  let data = npArr.data

  withNogil:
    var sum = 0'i64
    for i in 0..<n:
      sum += data[i]
    return sum
```

```python
import numpy as np
from example_project import numpy_array_sum_fast

arr = np.arange(1000000, dtype=np.int64)
numpy_array_sum_fast(arr)  # 499999500000
```

#### In-Place NumPy Modification

```nim
proc numpy_array_multiply_in_place(arr: PyObject, scalar: float64) {.nuwa_export.} =
  var npArr = asNumpyArrayWrite(arr, float64)
  for val in mitems(npArr):
    val = val * scalar
```

```python
import numpy as np
from example_project import numpy_array_multiply_in_place

arr = np.array([1.0, 2.0, 3.0], dtype=np.float64)
numpy_array_multiply_in_place(arr, 2.0)
print(arr)  # [2.0, 4.0, 6.0]
```

#### Multi-Dimensional Arrays

```nim
proc numpy_matrix_multiply(a: PyObject, b: PyObject): seq[seq[float64]] {.nuwa_export.} =
  var matA = asStridedArray(a, float64)
  var matB = asStridedArray(b, float64)

  let rowsA = matA.shape[0]
  let colsA = matA.shape[1]
  let colsB = matB.shape[1]

  if matB.shape[0] != colsA:
    raise newException(ValueError, "Matrix dimensions mismatch for multiplication")

  result = newSeq[seq[float64]](rowsA)
  for i in 0..<rowsA:
    result[i] = newSeq[float64](colsB)
    for j in 0..<colsB:
      var sum = 0.0
      for k in 0..<colsA:
        sum += matA[i, k] * matB[k, j]
      result[i][j] = sum
```

```python
import numpy as np
from example_project import numpy_matrix_multiply

a = np.array([[1, 2], [3, 4]], dtype=np.float64)
b = np.array([[5, 6], [7, 8]], dtype=np.float64)
numpy_matrix_multiply(a, b)  # [[19.0, 22.0], [43.0, 50.0]]
```

#### Creating NumPy Arrays from Nim

```nim
proc create_range_array(start: int64, stop: int64, step: int64 = 1): PyObject {.nuwa_export.} =
  if step == 0:
    raise newException(ValueError, "Step cannot be zero")
  let size = (stop - start + step - (if step > 0: 1 else: -1)) div step
  if size <= 0:
    raise newException(ValueError, "Invalid range: size must be positive")

  let np = pyImport("numpy")
  result = np.arange(start, stop, step)

proc create_zeros_array(size: int): PyObject {.nuwa_export.} =
  let np = pyImport("numpy")
  result = np.zeros(size)
```

```python
import numpy as np
from example_project import create_range_array, create_zeros_array

create_range_array(0, 10, 2)  # np.array([0, 2, 4, 6, 8])
create_zeros_array(5)         # np.array([0.0, 0.0, 0.0, 0.0, 0.0])
```

### Quick Start

```bash
# Clone and navigate to the example
cd nuwa-example/example_project

# Build the extension (debug mode)
nuwa develop

# Run the example script
python example.py

# Run tests
pytest
```

### Project Structure

```
example_project/
├── .github/
│   └── workflows/
│       └── publish.yml          # PyPI publishing workflow
├── nim/                          # Nim source files
│   ├── example_project_lib.nim   # Main entry point
│   └── helpers.nim               # Additional modules
├── example_project/              # Python package
│   ├── __init__.py              # Package wrapper
│   ├── example_project_lib.so   # Compiled extension (generated)
│   └── example_project_lib.pyi  # Type stubs (generated)
├── tests/                       # Pytest tests
│   └── test_example_project.py
├── example.py                   # Usage examples
└── pyproject.toml               # Project configuration
```

### Configuration

The project uses `pyproject.toml` for all configuration:

```toml
[build-system]
requires = ["nuwa-build"]
build-backend = "nuwa_build"

[tool.nuwa]
nim-source = "nim"
module-name = "example_project"
lib-name = "example_project_lib"
entry-point = "example_project_lib.nim"
nimble-deps = ["nimpy@0.2.1", "nuwa_sdk@0.4.3"]

# Build profiles
[tool.nuwa.profiles.dev]
nim-flags = ["-d:debug", "--debugger:native", "--linenos:on"]

[tool.nuwa.profiles.release]
nim-flags = ["-d:release", "--opt:speed"]
```

### Adding New Functions

1. Edit `nim/example_project_lib.nim`:
```nim
import nuwa_sdk

proc my_function(x: int, y: int): string {.nuwa_export.} =
  ## Your function documentation
  return "Result: " & $(x + y)
```

2. Rebuild:
```bash
nuwa develop
```

3. Use from Python (with full autocomplete):
```python
from example_project import my_function

# IDE shows: my_function(x: int, y: int) -> str
result = my_function(5, 10)
```

### Publishing to PyPI

The project includes a GitHub Actions workflow for automated publishing:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The workflow automatically builds wheels for:

- **Platforms**: Linux, macOS, Windows
- **Python versions**: 3.10, 3.11, 3.12, 3.13, 3.14 (regular CPython)

### Source

Repository: [github.com/martineastwood/nuwa-example](https://github.com/martineastwood/nuwa-example)
