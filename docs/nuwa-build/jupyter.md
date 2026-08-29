# Jupyter Notebook Support

Nuwa-Build includes a Jupyter magic command for compiling Nim code directly in notebooks.

Install it with `pip install "nuwa-build[notebook]"`.

## Usage

```python
# Load the extension
%load_ext nuwa_build.magic

# Compile Nim code in a cell
%%nuwa
proc add(a, b: int): int {.nuwa_export.} =
    return a + b

# Use the function immediately (no import needed!)
add(1, 2)  # Output: 3
```

## Caching

Nuwa automatically caches compiled modules in `.nuwacache/`:

- **Cache hits**: Re-running cells uses cached compilation (fast!)
- **Cache misses**: New code or different flags trigger recompilation
- **Persistent**: Cache survives kernel restarts

### Cache Management

```python
# Show cache statistics
%nuwa_cache_info
# Output: 📊 Cache: 3 modules, 123.4 KB
#         📍 Location: /path/to/.nuwacache

# Clear cache
%nuwa_clean
# Output: 🧹 Cleared cache: .nuwacache
```

Or manually delete the `.nuwacache/` folder.

### .gitignore

Add to your `.gitignore`:
```
.nuwacache/
```

## Compiler Flags

Pass Nim compiler flags on the magic line:

```python
%%nuwa -d:release --opt:speed
proc optimized_func(n: int): int {.nuwa_export.} =
    # Optimized implementation
    ...
```

Different flags create different cache entries (code hash includes flags).

## Example

```python
# Cell 1: Load extension
%load_ext nuwa_build.magic

# Cell 2: Compile simple function
%%nuwa
proc greet(name: string): string {.nuwa_export.} =
    return "Hello, " & name & "!"

# Cell 3: Use immediately
print(greet("World"))  # Output: Hello, World!

# Cell 4: Multiple functions
%%nuwa
proc add(a, b: int): int {.nuwa_export.} = a + b
proc multiply(a, b: int): int {.nuwa_export.} = a * b

print(add(5, 3))      # 8
print(multiply(4, 7)) # 28

# Cell 5: With compiler flags
%%nuwa -d:release
proc fibonacci(n: int): int {.nuwa_export.} =
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(10))  # 55 (optimized)
```
