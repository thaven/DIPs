# Mixed pointer sizes

| Field           | Value                                                           |
|-----------------|-----------------------------------------------------------------|
| DIP:            |                                                                 |
| Review Count:   | 0                                                               |
| Author:         | Harry T. Vennik <htvennik@gmail.com>                            |
| Implementation: | (none yet)                                                      |
| Status:         |                                                                 |

## Abstract

This DIP proposes a syntax to explicitly specify the size of a pointer, as well
as semantic rules regarding the mixed use of pointer sizes.

Two new keywords are added: `__ptr32` and `__ptr64`, which are used as type
constructors to create pointer types.

Also the semantical definition of `*` in its use to denote a pointer type is
modified to define the way it interoperates with the use of mixed pointer sizes.

### Reference

Optional links to reference material such as existing discussions, research papers
or any other supplementary materials.

## Contents
* [Rationale](#rationale)
* [Description](#description)
* [Breaking Changes and Deprecations](#breaking-changes-and-deprecations)
* [Review](#review)

## Rationale

On 64-bit architectures it is still possible to use 32-bit pointers if the OS is
capable of providing a suitable memory layout. Usage of 32-bit pointers may
reduce memory usage significantly, depending on the workload. Some applications
may benefit from using mostly 32-bit pointers, but still use 64-bit pointers
to address data caches placed high in memory.

Currently D does not have any way to specify pointer size, but instead it is
fixed at the default pointer size of the compilation target system. The intent
of this DIP is to provide D programmers with opt-in control over pointer size.

## Description

D grammar is modified as follows: Two additional alternatives are added to the
_`TypeCtor`_ production: __`__ptr32`__ and __`__ptr64`__.

A version identifier `D_MixedPointers` is added to indicate whether the compiler
is able to generate code using non-native pointer size for the compilation
target.

### Formal definition of semantics

Given any type `T`, `__ptr32 T` denotes a 32-bit pointer type, pointing to a
value of type `T`.

Given any type `T`, `__ptr64 T` denotes a 64-bit pointer type, pointing to a
value of type `T`.

The existing syntax `T*` denotes a native size pointer type, pointing to a
value of type `T`. If the native pointer size for the compilation target is
32-bit, `T*` is an alias of `__ptr32 T`. If the native pointer size for the
compilation target is 64-bit, `T*` is an alias of `__ptr64 T`.

A 32-bit pointer is implicitly convertible to a 64-bit pointer. Conversion of
a 64-bit pointer to a 32-bit pointer, called a _pointer truncation_, requires
a cast and is disallowed in `@safe` code.

Whether pointers are zero-extended or sign-extended when converting from
32-bit to 64-bit, depends on what is natural to the compilation target
architecture. Given a 32-bit pointer `p`, it is required that `*p` and
`*(cast(__ptr64) p)` will read from the same position in memory.

### Target dependent limitations

Any attempt to dereference a pointer of a type that is not supported by the
compilation target architecture is a compile-time error.

### Examples
```
__ptr32 int p;      // p is a 32-bit pointer to int
__ptr64 int q;      // q is a 64-bit pointer to int

q = p;              // OK: pointer is extended
p = q;              // ERROR: Assignment truncates pointer
p = cast(__ptr32 int) q; // OK, but disallowed in @safe code
```

As with all type constructors, the referenced type may be enclosed in
parentheses: e.g. `__ptr32(float)` is identical to just `__ptr32 float`.

## Breaking Changes and Deprecations

No breaking changes to the D language syntax and semantics are proposed, but a
potentially breaking ABI change is required to implement this DIP: name mangling
needs to be changed to distinguish between pointer sizes.

## Copyright & License

Copyright (c) 2018 by the D Language Foundation

Licensed under [Creative Commons Zero 1.0](https://creativecommons.org/publicdomain/zero/1.0/legalcode.txt)

## Review

The DIP Manager will supplement this section with a summary of each review stage
of the DIP process beyond the Draft Review.
