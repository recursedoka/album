# Album

Album (meaning list in Latin) is a Lisp dialect that attempts to gather the best
ideas and features from other Lisp dialects and bring them into a single, robust
environment that encourages continual experimentation and improvement for both
the language and applications written on the language without sacrificing
stability or performance. It is meant for both developing new, innovative
technology and bringing that technology to market under the ideology that
companies that go directly from concept to reality the fastest maintain an
advantage over their competitors.

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
@fut -> (deref fut)
^4 -> (fn (n) (expt n 4))
;comment ->
(let |'woah| 2 |'woah|) => 2
(io#print "hi") -> (io 'print) => nil ; prints hi

(func-or-macro args ...) => calls func with args or expands macro with args
[+ _ 2] -> (fn (_) (+ _ 2))
(:key value :key2 value2) -> (hash :key value :key2 value2)
'(x) -> (quote (x))
(let (x 2 y '(3 4)) `(+ ,x @y)) -> (+ 2 3 4)
$(:weak)(list 12 14) => (12 14) ; metadata is hidden 
```

Too many special forms?

## Features

- Restarts
- Strings are just lists of characters
- Generic set
- CL-style macros as values
- Racket modules with a touch of Clojure and Node.js
- Scheme pairs/lists
- Every function is generic
- Arc-esque type annotation mixed with structs
- Clojure concurrency
- Arc brevity including function composition
- Type matching and arity matching
- Clojure hash tables and meta data
- Prefix read macros
- Tail recursion
- Minimalism Node.js-style
- JIT compiler built on custom static compiler
- Compiles to optimized bytecode
- Support for ARM with NEON, x86/64 with AVX

# Specification

## Non-special Forms

A non-special form has the following syntax:

```
(func args...)
```

Evaluates the variable with the given arguments.

If the symbol is one of the special forms that follow this section, none of the
following happen, and the respective special form semantics are followed.

Firstly, func is evaluated to retrieve a value. The action that occurs depends
on the type of the result.

If it is a lambda, each argument is first evaluated. The results are then bound
in a new closure, where the function body is evaluated. The value of the final
form of the function body is returned.

If it is a macro, then the arguments are bound to a new closure, without being
evaluated. The macro body is then run in the closure, with the value of the
final form being called in the place of the macro, as if the value of the final
form replaced the original macro call.

If it is a list, then the there can only be one argument, which, when evaluated,
must be  an integer value. The value at that index of the list is returned.

If it is a hash table, then there can be only one argument, which, when
evaluated, must be a keyword value.  The value of that key in the hash table is
returned if it exists, otherwise nil is returned.

If it is a package, then there can only be one argument, which, when evaluated,
must be  a symbol value. The value of that symbol in the package is returned if
it exists, otherwise nil is returned. 

If it is a number, a list is created, with the the first value being the number
itself. Each additional argument is evaluated, the results of which are appended
to the list. The return value is that list.

If it is a character, a list is created, with the the first value being the
character itself. Each additional argument is evaluated, the results of which
are appended to the list. The return value is that list.

If it is a keyword, a hash table is created, the keyword being the first key of
the table. If no arguments are given, then the value of this key is the symbol
t. If arguments are given, the first argument is the value of the first key, and
the additional arguments are key value pairs. The return value is that hash
table.

If it is a symbol, including nil, no arguments are allowed, and the symbol
itself is returned.

## Special Forms

The following special forms take precedence over the last section. Each section
starts with the syntax, followed by a one line summary, followed by a
detailed description of the semantics.

### Table of Contents

- [Lambda](#lambda)
- [Let Forms](#let-forms)
  - [Single Variable](#single-variable)
- [Conditional Branch](#conditional-branch)

### Lambda
```
(fn (parameter ...) body...)
parameter := type symbol
           | type (optional symbol :default value :passed symbol :guard predicate?)
           | type (keyword  symbol keyword :default value  :passed symbol :guard predicate?)
           | &symbol
```

Creates a function that accepts the arguments to be bound to the given
parameters, and executes the given forms in a closure with the arguments bound
to their respective parameters, returning the value of the last form of the body
as the result.

When called, the first thing the returned lambda does is bind the arguments to
their specified parameters in a new closure. Each parameter form can be
specified in several different ways.

Most forms consist of two seperate parts, the type specifier and the binding
specifier. The type specifier is evaluated, the value of which is the type
expected of the matched argument. The binding specified can be many things, as
follows.

The most basic form of the binding specifier is a single unquoted symbol. This
is the symbol that the matched argument should be bound to. This form of
parameter must be matched, and they are matched in the order they are defined in
the parameter list. These must precede all optional parameters.

The binding specifier may be a list. If it is, the first value of the list can
either be the unquoted symbol `optional` or `keyword`.

If the first item in the list is the unquoted symbol `optional`, then an
optional argument is specified, meaning this parameter does not need to be
matched in the argument list.  The second symbol is the name to bind the
matched argument to, if it was matched. Then any number of key value pairs can
be specified as optionals. All optional parameters must follow nonoptional
parameters. These parameter forms are matched in the order they are defined in
the parameter list.

If the first item in the list is the unquoted symbol `keyword`, then a keyword
argument is specified, meaning this parameter should be specified by an optional
key-value pair. The second item in the list is an unquoted symbol to bind the
value afer the keyword to, if it was specified. Keyword parameters need no order,
since they are specified by a key anywhere in the argument list. The third item
in the list is the keyword for which the value should be found in the argument
list.

Both of these binding specified share a number of options, as follows.

The `:default` option specifies a value to bind to the symbol if it was not
matched with a value in the argument list.

The `:passed` option specifies a symbol that will be bound to a value
representing whether the parameter was matched with an argument or not. If the
parameter was matched, the symbol will be bound to 't in the function body,
otherwise it will be bound to nil.

The `:guard` option specifies a predicate function to call with a single
argument that is the value matched in the argument list. If the value of calling
this function is nil, then the argument will not be matched, and the whole
function evaluation will not succeed.

The body forms are then run in the created closure. The final form's value is
returned as the value of evaluating the returned lambda.

### Let Forms

## Single Variable
```
(let var value body...)
```

### Conditional Branch
```
(if condition first second)
```

Branches to either the first or second branch depending on the condition.

Evaluates the condition. If it is not nil, it evaluates the first form,
returning it's value, otherwise it evaluates the second form, returning that
form's value.


