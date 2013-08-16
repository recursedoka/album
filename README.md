<img src="https://gist.github.com/recursedoka/ace6882be7a2f90180c1/raw/650cfba235c9fc2f886702078b28078f28c5f7e1/logo.svg"/>

Album (meaning list in Latin) is a Lisp dialect that attempts to gather the best
ideas and features from other Lisp dialects and bring them into a single, robust
environment that encourages continual experimentation and improvement for both
the language and applications written on the language without sacrificing
stability or performance. It is meant for both developing new, innovative
technology and bringing that technology to market under the ideology that
companies that go directly from concept to reality the fastest maintain an
advantage over their competitors.

Performance is an important part of any program, so Album is implemented with a
JIT compiler. Improvements focused on performance are greatly encouraged, and
the JIT itself is open source and bootstrapping. The same compiler that the JIT
uses is used in compiling the JIT itself. Album developers interested in helping
with the JIT should feel right at home.

Correctness is fairly subjective, but there are standards to adhere too. Things
shouldn't be "special" if they don't need to be. Special forms should be limited
and the standard library should be small. Libraries need to be developed as
modules, not part of the core library, so that we can encourage competition
between different modules. This includes things like IO, networking, and
graphics.

However, some things need to be standardized. For example, concurrency can
require special access to the VM to be implemented in some ways, and crosses
boundaries between modules frequently. Restarts are the same way. By including
these in the core language, one can be sure that everyone writes code that is
fundamentally compatible with other people's code. For example, look at how not
having a standard error handling mechanism works in C.

Core libraries can still be challenged. By creating a new module, and using the
foreign function interface, one can prove the supremacy of a new module. If it
needs to be implemented in the core VM, then a new fork is suggested. When
proves itself worthy, it can be swapped in to the core library quickly. Using
the power of homoiconicity, old code can easily be automatically updated to a
new library. Don't be afraid to challenge assumptions!

Powerful features are not restricted, and new suggestions are important to the
ecosystem. Staying on top of the latest advancements in language design is a
priority and experimentation is encouraged. Generally, several releases are
maintained at once, and production code need not update to every new version.
Stability is maintained within a single minor version. As such, git is a perfect
version control system to keep multiple running versions in the same repository.

Since Album is such a fast moving environment with multiple versions to maintain
and many improvements to review and merge, maintainers are always in need. The
best way to become a maintainer is to start contributing to the code itself and
commenting on other pull requests. If you work with a business and have
significant experience, feel free to email me directly to talk about maintainer
access.

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
(:key value :key2 value2) -> (hash :key value :key2 value2)
'(x) -> (quote (x))
(let (x 2 y '(3 4)) `(+ ,x ,@y)) -> (+ 2 3 4)
(let x '(3 4) (+ @x)) => 7 ; notice no expansion -> @ is an important thing
%(:weak)(list 12 14) => (12 14) ; metadata is hidden 
```

THERE'S NOTHING LEFT!!!!

## Features

- Restarts
- Strings are just lists of characters
- Generic set
- Dynamic binding
- Fancy functions replace macros 
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

## Value Types

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

If it is a function, each argument is evaluated if the function hasn't set the
`:evaluate` key to nil. Then the argument list uses the parameter list to bind
arguments to parameter symbols in a new closure. The body of the function is
then called in the specified closure. If the `:dynamic` key has been set to a
value that isn't nil, then the specified closure inherits from the closure where
the function was called, otherwise the closure inherits from the closure where
the function was defined.

If it is a list, then the there can only be one argument, which, when evaluated,
must be an integer value. The value at that index of the list is returned.

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

- [Function Definition](#function-definition)
- [Conditional Branch](#conditional-branch)

### Function Definition
```
(fn (parameter ...) :case guard :evaluate nil :dynamic 't body...)
parameter := type symbol
           | type (optional symbol :default value :passed symbol)
           | type (keyword  symbol keyword :default value  :passed symbol)
           | &symbol
```

Creates a function that accepts some arguments to be bound to the given
parameters, and executes the given body forms in a closure with the arguments
bound to their respective parameters, returning the value of the last form of
the body as the result.

When called, the first thing the returned function does is bind the arguments to
their specified parameters in a new closure. Each parameter form can be
specified in several different ways.

Most forms consist of two seperate parts, the type specifier and the binding
specifier. The type specifier is evaluated, the value of which is the type
expected of the matched argument. The binding specifier can be many things,
depending on the type of binding you wish to create, but it is responsible for
defining how the parameter should be bound to an argument.

An exception to this rule is the rest form. If this is specified, then any
arguments that were not bound to a parameter are instead bound to the value of
the symbol. The rest symbol must start with an &, and the following characters
are the characters of the symbol to bind the list of unbound arguments to. For
example, if you specify `&rlist`, then the symbol `rlist` will be bound to the
list of unbound arguments.

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

After the parameter list, several options can be specified. These should be
specified before the body of the lambda. These options follow.

The `:case` option specifies a single form to evaluate (with the parameters
and arguments bound to a new, different closure from the body) that determines
whether the function should continue to be called. If the value of the form is
nil, the function will not be executed, and an error will be triggered.

The `:dynamic` option specifies whether the function will be evaluated with a
dynamic scope or a lexical scope. If you don't know what you're doing, you
should leave this option alone. This is mostly useful for macro-like functions.
If the value of this option is `nil`, the binding will be lexical, which is the
default value, but if you set it to something else, the function will be scoped
dynamically, meaning it is evaluated in the environment it is called, rather
than the environment in which it is defined. This is especially useful when you
set the `:evaluate` option.

The `:evaluate` option specifies whether the function arguments will be
evaluated or not. The default value is 't, but if the value of this key is nil,
the arguments will not be evaluated when the function is called. This allows you
to create a macro-esque function.

When the function is run, the body forms are run in the closure created by
binding the arguments to the parameters, and the final form's value is
returned as the value of running the function. (Not the same as the :case
closure).

The function that was created from the parameters, options, and body is the
value of this special form.

### Conditional Branch
```
(if condition first second)
```

Branches to either the first or second branch depending on the condition.

Evaluates the condition. If it is not nil, it evaluates the first form,
returning it's value, otherwise it evaluates the second form, returning that
form's value.

### Chaining Evaluation
```
(begin body...)
```

Evaluates each body in order in the current closure, returning the value of the
last form. This means all definitions contine to occur where the body was
called.

### Application
```
(apply function list)
```

Calls the function with the specified list as the argument list. The function
can actually be anything that can be applied with non-special forms or a special
form. Therefore, anything you call like a normal function can be applied.

### Sandboxing
```
(sandbox form-list :exclude exclude-list body...)
```

Executes the body, limiting what functions and forms can be used. The form-list
specifies which packages and forms can be used. The list can contain both
packages and specific symbols to allow the body to access. To exclude specific
symbols from access by the body, an `:exclude` option allows you to specify a
list of symbols and packages to exclude. This is useful when you have code to
run from a source outside of the program text, like an extension script.


## Standard Definitions

### Let Forms

#### Single Variable
```
(let var value body...)
```

((lambda (var) body...) value)


## Rationale 

Functions with unevaluated arguments and dynamic binding are preferred over
macros because they permit more flexibility. A significance performance loss is
not realized since macros are evaluated at runtime anyways in a JIT environment.
This also takes away one more special form.

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

Read macros don't affect anything outside of the package they are defined in by
default to prevent collisions. They can be imported explicitly into a package if
desired. Since they can't be namespaced in the same way that other language
features can, they are essentially global. We should therefore treat them very
carefully, and being explicit is the best way to do that. However, although
removal could be an option, it is important to be able to create shortcuts in
the name of brevity.

Changing functions is a dangerous thing to add to the language, and it is
suggested that it is used sparingly. There are other things like this available
to make the language more powerful, like read macros, which allow you to change
the syntax, albeit in a limited way. Adding powerful features to a language
doesn't preclude stability. With the power of homoiconicity, code to warn when
certain functions are used incorrectly or frequently is easy to write.
Therefore, anyone concerned about the use of these functions can easily verify
that they are used wisely, and so it isn't a problem.

# Notes

*What's left:*
- foreign functions
- assignment
- dynamic/globals
- quoting
- lists
- numbers
- characters
- keywords
- packages
- structs
- arity matching
- restarts
- read macros
- concurrency

<!--%a-%t
s/[^A-Za-z0-9\s]//g
s/\s+/\-/g-->
