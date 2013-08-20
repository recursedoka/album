# Special Forms

The following special forms take precedence over the last section. Each section
starts with the syntax, followed by a one line summary, followed by a
detailed description of the semantics.

Special forms require access to the underlying VM, and cannot exist without
special allowances. They cannot be defined with functions or macros, and
sometimes are the forms that directly correlate to assembly code. These are the
basic building blocks of the language.


## Functions and Macros

### Functions
```
(fn (parameter ...) :case guard :evaluate nil :dynamic 't body...)
parameter := type symbol
           | type (symbol keyword :default value :passed symbol)
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

If the binding specifier is a list, then a keyword parameter is specified,
meaning this parameter should be specified by an optional key-value pair. The
first item in the list is an unquoted symbol to bind the value after the keyword
to, if it was specified. The second item in the list is the keyword for which
the value should be found in the argument list. Keyword parameters need no
order, since they are specified by a key anywhere in the argument list. This
form has a number of options, as follows.

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

The `:evaluate` option specifies whether the function arguments will be
evaluated or not. The default value is 't, but if the value of this key is nil,
the arguments will not be evaluated when the function is called. This allows you
to create a function that acts a lot like a macro, but is more hygienic.

When the function is run, the body forms are run in the closure created by
binding the arguments to the parameters, and the final form's value is
returned as the value of running the function. (Not the same as the :case
closure).

The function that was created from the parameters, options, and body is the
value of this special form.

### Source Code Access
```
(source func)
```

Returns the source form that created this function.

### Generics
```
(extend function new-function)
```

Adds a new dispatch to a function. Initially a function is defined with a single
dispatch. After adding the second dispatch to a function, they are sorted by how
specific they are. Functions with more specific types and parameters are more
specific, and therefore are closer to the top of the list. Each function's
parameter list is matched with the argument list, until a successful match is
found. When that happens, the matched dispatch is run, and the function returns
its value.

### Macros
```
(macro function)
```

Returns a macro. When executed, a macro calls the passed function with the
arguments passed to the macro. The return value of the function is then replaced
with the call to the macro like it was the original call.

This is very similar to a unevaluated function with dynamic binding. The
difference is that the return value of the function has dynamic binding at the
place where the macro was executed. This makes it so that the macro's local
variables aren't accessible to any code run in the return value. If you don't
need this, use unevaluated functions. They are more hygienic.

### Evaluation
```
(eval form)
```

Evaluates the specified form in the current environment. This is the same as if
the specified form had been evaluated at the position of the eval.

## Gensym
```
(gensym)
```

Generates a unique symbol. Especially useful for making macros more hygienic.

## Splicing
```
(let list '(1 2 3)
  (+ @list))
```

Takes the value of the list and splices into the current form. Allows you to do
extremely cool things. It's apply for pros.

## Control Flow

### Conditional Branch
```
(if condition first second)
```

Branches to either the first or second branch depending on the condition.

Evaluates the condition. If it is not nil, it evaluates the first form,
returning it's value, otherwise it evaluates the second form, returning that
form's value.

## Lexical Variables
```
(def var value)
```

Binds the symbol `var` to `value`. This occurs in the closure that define was
executed in, so it can be shadowed by defines lower in the chain. This should
not be used for global variables; see the section on dynamic variables.

This can be shadowed in closures lower in the chain.

## Global Variables
```
(global var value)
```

Creates a global variable. Global variables have dynamic scoping, and can be
shadowed by local dynamic bindings. 

## Dynamic Bindings
```
(binding var value body...)
```

Creates a local dynamic variable. If there is a global dynamic variable with
the same name, then the local variable shadows it.

## Assignment 
```
(set! var value)
```

Changes the value of the symbol `var` to `value`. This changes the value in the
closure that `var` is bound in, so it does not shadow any bindings. This
assignment is generic and can reach into data structures to change their values.

Some examples:

```
(def x '(1 2 3))
(set! (x 1) 5)
x ;=> (1 5 3)

(define t (:first-value 5 :other-value 7))
(set! (t :other-value) 8)
t ;-> (:first-value 5 :other-value 8)
```

## Structs

```
(struct 'name (data...)
  :inherit nil
  )
data := type name
      | type (name
               :accessor ('t 't) ; (read write)
               :construct 't ; include in the constructor?
               :default 213 ; default value
               :keyword 't) ; keyword in constructor
```

## Packages



## Foreign Functions

### Importing
```
(import "libc.so" 'libc)
```

Imports the shared library into a namespace.

### Calling
```
; Continued from above
(call 'libc/printf "%d" 15)
```

Calls the library function from a namespace. Triggers a restart if the types of
the parameters can't be coerced to the C parameter types.

## Sandboxing
```
(sandbox form-list :exclude exclude-list body...)
```

Executes the body, limiting what functions and forms can be used. The form-list
specifies which packages and forms can be used. The list can contain both
packages and specific symbols to allow the body to access. To exclude specific
symbols from access by the body, an `:exclude` option allows you to specify a
list of symbols and packages to exclude. This is useful when you have code to
run from a source outside of the program text, like an extension script.

