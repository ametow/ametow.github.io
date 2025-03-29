---
title: "Golang slices "
date: "2025-03-10"
description: "Slices in Go, internals of usage"
summary: "Slices in Go, internals of usage"
tags: ["golang", "slices"]
categories: ["golang"]
ShowToc: true
TocOpen: true
---

### Introduction

Go has a built-in dynamic array type called slice. It's the most common data structure in the language and extremely powerful. This article explores what slices are and how to use them.

### Arrays

Slices are an abstraction layer built on top of arrays in Go. To understand slices, we must first understand arrays.

An array is a fixed-size data structure that stores elements of the same type in contiguous memory, providing constant-time access through indices. Arrays are defined as follows:

```go
var nums [4]int
```

The variable `nums` is of type `[4]int`. An array's size is part of its type, so `[6]int` is an entirely different type.

We set elements using indices:

```go
nums[0] = 11
nums[3] = 12

i := nums[3] // i = 12
```

If we try to assign outside of the bounds, we get a compile-time error:

```go
nums[5] = 23 // invalid argument: index 5 out of bounds [0:4]
```

Arrays don't need to be initialized explicitly; the zero value of an array is a ready-to-use array whose elements are zeroed:

```go
fmt.Println(nums) // [11 0 0 12]
```

The representation of this array is simply four integer values laid out sequentially:

![slice-array.png](/images/slice-array.png)

Unlike in `C`, arrays in Go are values, not pointers to the first element. This means when you assign or pass an array, you make a copy of its contents.

```go
nums := [3]int{12,14,16}
processArray(nums)
// nums[0] = 12 still
}

func processArray(nums [3]int) {
	nums[0] = 15
}
```

To avoid copying, you can pass a *pointer* to the array, though this makes it a pointer to an array rather than an array itself:

```go
processArray(&nums)
}

func processArray(nums *[3]int) {
...
}
```

### Slices

Arrays are inflexible and therefore uncommon in Go code. Slices, being dynamically resizable, build on arrays to provide greater convenience. You can think of a slice as a struct with three fields:

```go
// imaginary type
type Slice struct {
	data *[10]int
	length int
	capacity int
}
```

The `data` field is a pointer to the underlying array. `length` is the number of elements the slice currently holds, and `capacity` defines how many elements the slice can hold before increasing the underlying array's size. A slice can be created with the built-in function `make`:

```go
// func make([]T, length, capacity) []T
mySlice := make([]int, 10, 10) // a slice with len = 10 and cap = 10
print(mySlice, len(mySlice), cap(mySlice)) // [0 0 0 0 0 0 0 0 0 0], 10, 10
```

Here, T represents the element type of the slice to be created. The `capacity` argument is optional—if omitted, it defaults to the specified `length`. You can inspect a slice's length and capacity using the built-in `len` and `cap` functions.

When called, `make` allocates an array and returns a slice that refers to that array.

A slice literal is declared like an array literal, but without the element count:

```go
letters := []string{"a", "b", "c", "d"}
```

The zero value of a slice is nil:

```go
var nums []int
print(nums == nil) // true
print(len(nums), cap(nums)) // 0, 0
```

You can also create a slice by "slicing" an existing slice or array using a half-open range with two indices separated by a colon:

```go
b := []int{1,2,3,4,5,6}
c := b[1:3] // []int{2,3}
```

Variable `c` is a new slice that refers to the same storage as `b` with length 2 and capacity 5.

By default, `capacity` is the number of elements in the underlying array (beginning at the element referred to by the slice pointer). However, you can specify the capacity explicitly with this syntax:

```go
c := b[1:3:3] // []int{2,3}, len = 2, cap = 3
```

The third number specifies the capacity of the newly created slice. A slice cannot be grown beyond its capacity, appending to it with `append` will allocate a new underlying array. Similarly it cannot be re-sliced below zero to access earlier elements in the array.

The start and end indices are optional—they default to zero and the length of the slice respectively:

```go
c := b[:] // == b
// c := b[2:] == []int{3,4,5,6}
// c := b[:2] == []int{1,2}
```

Since slices share the underlying array, changing values in one slice is reflected on the others:

```go
 c[0] = 77
 print(b) // [1,77,3,4,5,6]
 print(c) // [77, 3]
```

This is also the syntax to create a slice from an array:

```go
nums := [4]int{12,13,14,15}
numSlice := nums[2:] // new slice refers to nums array, len = 2, cap = 2
```