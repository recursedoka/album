# Standard Form

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

If it is a symbol, then the there can only be one argument, which, when evaluated,
must be an integer value. The character at that index in the symbol is returned.

If it is a hash table, then there can be only one argument, which, when
evaluated, must be a keyword value.  The value of that key in the hash table is
returned if it exists, otherwise nil is returned.

If it is a package, then there can only be one argument, which, when evaluated,
must be a symbol value. The value of that symbol in the package is returned if
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

If it is nil, nil is returned.

