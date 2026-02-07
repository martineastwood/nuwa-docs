# GIL Release with withNogil

The `withNogil` template allows you to release Python's Global Interpreter Lock (GIL) during performance-critical Nim code, enabling true parallelism in multi-threaded Python applications.

## Overview

Python's GIL prevents multiple threads from executing Python bytecode simultaneously. The `withNogil` template releases the GIL so other Python threads can run while your Nim code executes.

## Basic Usage

```nim
import nuwa_sdk

proc computePi(iterations: int): float {.nuwa_export.} =
  ## Compute Pi using Monte Carlo method (GIL-released)
  var count = 0
  withNogil:
    # Pure Nim code runs without Python interpreter interference
    # Other Python threads can run concurrently
    for i in 0..<iterations:
      let x = rand(1.0)
      let y = rand(1.0)
      if x*x + y*y <= 1.0:
        count += 1
  return 4.0 * float(count) / float(iterations)
```

## When to Use withNogil

### Good Use Cases ✅

- **CPU-bound computations**: Mathematical operations, numeric algorithms
- **Batch processing**: Processing large datasets without Python interaction
- **Parallel workloads**: Allow other Python threads to run while Nim computes

```nim
# Image processing - can run in parallel
proc processImage(data: seq[uint8]): seq[uint8] {.nuwa_export.} =
  var result = newSeq[uint8](data.len)
  withNogil:
    for i in 0..<data.len:
      result[i] = data[i] xor 255  # Invert colors
  return result
```

### Bad Use Cases ❌

- **Python API calls**: Any code that calls Python objects or nimpy functions
- **Data transfer**: Converting between Nim and Python types
- **I/O operations**: File/network operations that release GIL automatically

```nim
# DON'T DO THIS - will crash!
proc badExample(data: PyObject): void {.nuwa_export.} =
  withNogil:
    # Error: Can't call Python functions without GIL!
    data.someMethod()
```

## How It Works

The `withNogil` template:

1. **Saves thread state** - Calls `PyEval_SaveThread()` to release the GIL
2. **Executes Nim code** - Runs your code block without Python interference
3. **Restores thread state** - Calls `PyEval_RestoreThread()` to reacquire the GIL
4. **Exception safety** - Ensures GIL is always reacquired even if an exception occurs

Equivalent to:
- Cython's `with nogil:` block
- CPython's `Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS`

## Example: Parallel Processing

```nim
import nuwa_sdk

proc expensiveComputation(n: int): int {.nuwa_export.} =
  ## Perform an expensive computation with GIL released
  var result = 0
  withNogil:
    for i in 0..<n:
      result += i * i
  return result
```

Python usage:

```python
import my_extension
from concurrent.futures import ThreadPoolExecutor

def process(n):
    return my_extension.expensive_computation(n)

# Run multiple computations in parallel
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(process, [10000, 20000, 30000, 40000]))

print(results)  # All computations run in parallel
```

## Technical Details

### Lazy Initialization

The `withNogil` template dynamically loads Python C API functions using `GetProcAddress()` on Windows or `dlsym()` on Unix-like systems. This happens once on first use.

### Thread Safety

The template ensures:
- GIL is always reacquired before returning to Python
- Exceptions during GIL release still trigger GIL reacquisition
- Multiple threads can safely execute Nim code concurrently

### Performance Considerations

- **GIL release overhead**: ~1-2 microseconds per release/acquire cycle
- **Worth it for**: Operations taking > 10 microseconds
- **Not worth it for**: Very simple operations (use normal Nim code)

## Best Practices

1. **Minimize GIL-held sections**: Do data conversion before/after `withNogil` block
2. **Keep blocks large**: One large `withNogil` block is better than many small ones
3. **Test carefully**: GIL bugs can cause segfaults - test with actual threaded Python code
4. **Profile first**: Make sure GIL contention is actually your bottleneck

```nim
proc goodPattern(data: seq[float]): float {.nuwa_export.} =
  ## Good: Convert data once, then compute
  var sum = 0.0
  withNogil:
    # All computation done in one block
    for x in data:
      sum += x * x
  return sqrt(sum)
```

```nim
proc badPattern(data: seq[float]): float {.nuwa_export.} =
  ## Bad: Multiple GIL releases - slower!
  var sum = 0.0
  for x in data:
    withNogil:  # Don't do this!
      sum += x * x
  return sqrt(sum)
```
