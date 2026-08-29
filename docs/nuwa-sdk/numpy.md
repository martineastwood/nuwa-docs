# NumPy / buffer views

Nuwa SDK wraps **existing** Python objects that export [PEP 3118](https://peps.python.org/pep-3118/) (NumPy ndarrays, many memoryviews). It does not allocate or return `ndarray`s. To give Python a new array, copy into a `seq` (or build an array in Python).

## Read and write

```nim
import nuwa_sdk

proc sumInt64(arr: PyObject): int64 {.nuwa_export.} =
  var view = asNumpyArray(arr, int64)
  result = 0
  for x in view:
    result += x

proc scaleInPlace(arr: PyObject, s: float64) {.nuwa_export.} =
  var view = asNumpyArrayWrite(arr, float64)
  for x in mitems(view):
    x = x * s
```

`asStridedArray` is the same constructor under another name.

## Types

Use a Nim element type that matches the array dtype: `int8`–`int64`, `uint8`–`uint64`, `float32`, `float64`, `bool`, plus platform `int` / `uint`. Wrong dtype raises `TypeError`. Byteswapped (non-native endian) multi-byte dtypes are rejected.

Complex, datetime, and object arrays are not supported.

## Layout

| API | Meaning |
| --- | --- |
| `isContiguous` | C-order (row-major) only |
| `isFortranContiguous` | Column-major |
| `data` | Pointer to the contiguous block if C- **or** Fortran-contiguous. Fortran `data[i]` is memory (column) order. |
| `items` | Logical **C-order** walk, including for Fortran arrays |
| `arr[i]` | **1D only**. 2D+ uses `arr[i, j, ...]` |

Non-contiguous views (slices, some transposes) use strided indexing. `data` raises `LayoutError`; copy in Python with `np.ascontiguousarray` / `np.asfortranarray` if you need a pointer.

## GIL

Acquire the view **with the GIL held**, then take `data` / `len` and run pure Nim inside `withNogil`. Do not call Python or nimpy inside that block.

## Cleanup

The wrapper releases the `Py_buffer` when it goes out of scope. `close()` is optional and needs a `var` binding.

Read-only NumPy arrays (`setflags(write=False)`) work with `asNumpyArray` and fail with `asNumpyArrayWrite`.
