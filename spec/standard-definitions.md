# Standard Definitions

Standard definitions are essentially in between special forms and non-special
forms. They are all defined as functions and macros, but they are allowed to
access the internal read macro interface, so they can have shortcut syntax. None
of them are defined with foreign functions, and all definitions are provided.
The main goal of the standard definitions is to help develop clear yet succinct
code fast and easily.

## Quoting

### Quote
```
'(a b c)
;->
(quote (a b c))

'other
;->
(quote other)

(def quote (fn (x) :evaluate nil x))
```

Prevents the specified list or atom from having any semantics. For example,
`'(a b c)` prevents the list `(a b c)` from being evaluated as a form. Note that
the syntax with the preceding `'` are transformed into the second syntax with
the surrounding `(quote)` form. Whenever a `(quote)` form is executed, it simply
returns it's argument as the value.

Note that this is only really a special form because of the special syntax. It
is actually defined as an unevaluated function and an internal read macro.

### Quasiquote
```
`(a ,b c)
;->
(list 'a b 'c)
```

## Let Forms

### No Variable
```
(def begin (macro (fn (&body)
  `(,fn ()
      ,@body))))
```

### Single Variable
```
(def let (macro (fn (symbol var
                     generic value
                     symbol (keyword name :named :passed name-passed?)
                     &body)
  (if name-passed?
    `(binding ,name (fn (,(type value) var) ,@body)
       (,name ,value))
    `((fn (,(type value) ,var) ,@body) ,value)))))
```

### Multiple Variables
```
(def let (macro (fn (list bindings
                     symbol (keyword name :named :passed name-passed?)
                     &body)
  (if name-passed?
    `(let ,(car bindings) ,(cadr bindings) :named ,name
       (let ,(cddr bindings) ,@body))
    `(let ,(car bindings) ,(cadr bindings)
       (let ,(cddr bindings) ,@body))))))
```

