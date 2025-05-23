# AtomicArena
[![Go Report Card](https://goreportcard.com/badge/github.com/Raezil/atomicarena)](https://goreportcard.com/report/github.com/Raezil/atomicarena)

`AtomicArena` is a lock-free, thread-safe generic arena allocator for Go. It provides a fixed-capacity container of atomic pointers, allowing you to allocate objects of any type **T** up to a maximum element count without locks or garbage collection overhead.

## Features

- **Generic**: Works with any Go type `T` whose size is known at compile time.
- **Lock-Free**: Uses atomic operations (`atomic.Uintptr`, `atomic.Pointer[T]`) for high concurrency without mutexes.
- **Fixed Capacity**: Pre-allocates a buffer for `maxElems` objects to avoid slice growth and dynamic allocations after initialization.
- **Resettable**: Clear the arena in constant time, reusing all slots immediately.
- **Bulk Allocation**: Easily append multiple values at once using `AppendSlice`.

## Installation

```bash
go get github.com/Raezil/atomicarena
```

## Usage

```go
package main

import (
    "fmt"
    "github.com/Raezil/atomicarena"
)

func main() {
    // Create an arena for 100 integers
    arena := atomicarena.NewAtomicArena[int](100)

    // Allocate a single value
    ptr, err := arena.Alloc(42)
    if err != nil {
        panic(err)
    }
    fmt.Println("Allocated:", *ptr)

    // Bulk allocate a slice of values
    inputs := []int{1, 2, 3, 4, 5}
    ptrs, err := arena.AppendSlice(inputs)
    if err != nil {
        panic(err)
    }
    for i, p := range ptrs {
        fmt.Printf("Value %d allocated: %d\n", i, *p)
    }

    // Reset to reuse all slots
    arena.Reset()
}
```

## API

### `NewAtomicArena[T any](maxElems uintptr) *AtomicArena[T]`
Creates a new arena capable of holding up to `maxElems` values of type `T`.

### `(a *AtomicArena[T]) Alloc(obj T) (*T, error)`
Atomically reserves a slot and stores `obj`. Returns an error if capacity is exhausted.

### `(a *AtomicArena[T]) AppendSlice(objs []T) ([]*T, error)`
Atomically reserves slots for each element in `objs`, storing them in the arena. Returns a slice of pointers to the stored values in the same order. If there is insufficient capacity to store all elements, no values are stored and an error is returned.

### `(a *AtomicArena[T]) Reset(release bool)`
Clears all allocations, setting the element count back to zero.
If release is true, Reset additionally zeroes out the raw storage (memset-style) before resetting the count and pointers.

## Example: Structs

```go
// Define a struct
 type Point struct { X, Y float64 }

// Arena for 10 points
a := atomicarena.NewAtomicArena[Point](10)

// Allocate two points
p1, _ := a.Alloc(Point{1, 2})
p2, _ := a.Alloc(Point{3, 4})

fmt.Println(*p1, *p2)
```

## Testing

A comprehensive test suite covers:

- Allocating basic types (`int`, `string`, etc.) and structs
- Bulk allocations via `AppendSlice`
- Error on exceeding `maxElems`
- `Reset()` correctness
- High-concurrency allocations (data-race free)

Run tests with:

```bash
go test --bench=. --cover --race
```

![Screenshot from 2025-04-29 17-34-57](https://github.com/user-attachments/assets/9d263d80-8519-4118-be5d-1e7a53828f59)
![output](https://github.com/user-attachments/assets/9fee7895-bd30-40b3-be3a-6af614c341e9)

