# Specification

## Table of Contents

- [Introduction](#introduction)
- [Data Types](data-types.md)
- [Standard Form](standard-form.md)
- [Special Forms](special-forms.md)
- [Standard Definitions](standard-definitions.md)

## Introduction

## Some syntax stuff

```
1 => 1
2/5 => 2/5
-4/3 => -4/3
4+2/3i => 4+2/3i
12.045 => 12.045
748.93S => 748.93S ;(single precision, defaults to double)
1e10 => 10000000000
x4B => 75
\c => c
\6D => m
(let b 2 b) => 2
'x -> (quote x) => x
:key => :key
nil -> '() => ()
"hello world" => (\h \e \l \l \o \space \w \o \r \l \d) => hello world
"hello ~"world~"" -> (\h \e \l \l \o \space \" \w \o \r \l \d \")
a:b:c -> (fn args (a (b (apply c args))))
-.6.3.1 -> (- 6 3 1) => 2
(let b 2 list!a.b!c) -> (let b 2 (list 'a b 'c)) => (a 2 c)
$fut -> (deref fut)
^4 -> (fn (n) (expt n 4))
;comment ->
(let |'woah| 2 |'woah|) => 2
(io#print "hi") -> (io 'print) => nil ; prints hi

(func-or-macro args ...) => calls func with args or expands macro with args
[+ _ 2] -> (fn (_) (+ _ 2))
{4 + 2 3} -> (+ 4 2 3) => 9
(:key value :key2 value2) -> (hash :key value :key2 value2)
'(x) -> (quote (x))
(let (x 2 y '(3 4)) `(+ ,x ,@y)) -> (+ 2 3 4)
(let x '(3 4) (+ @x)) => 7 ; notice no expansion -> @ is an important thing
%(:weak)(list 12 14) => (12 14) ; metadata is hidden 
```

THERE'S NOTHING LEFT!!!!

## Rationale 

Functions with unevaluated arguments are preferred over macros because they are
more hygienic. The return values from macros are executed in the environment
that the macro was called, which is preferably in some situations, but allows
any bindings to override the very meaning of a carelessly created macro. A
significant performance loss is not realized with unevaluated functions either,
since macros are evaluated at runtime anyways in a JIT environment. In some
cases macros can actually be slower than a function with unevaluated arguments,

Every function being generic doesn't mean the implementation has to be in place
for functions that aren't generic. Also, since the JIT is compiling code at
runtime, checking types at compile time are no better than checking them at
runtime, so a static solution doesn't solve the problem. Type checking could be
ignored entirely, but the guarantees they provide can help prevent bugs. Type
checking can also be ignored explicitly by using the most generic type
available, but this should be done explicitly since it adds an inherent risk of
developing a bug.

Lists and vectors don't both need to be implemented. The important semantics are
the same - you have a collection of ordered values. How they are stored in
memory is irrelevant to their usage, and the implementation can likely choose
the correct memory storage better than the programmer with a simple analysis of
how the list is used on a case-by-case basis. It is a JIT compiler after all.

Adding powerful but dangerous features to a language doesn't preclude stability.
With the power of homoiconicity, code to warn when certain functions are used
incorrectly or frequently is easy to write. Therefore, anyone concerned about
the use of these functions can easily verify that they are used wisely, so it
isn't a problem.

# Notes

*What's left:*
- lists
- numbers
- characters
- keywords
- packages
- structs
- restarts
- concurrency

<!--%a-%t
s/[^A-Za-z0-9\s]//g
s/\s+/\-/g-->
