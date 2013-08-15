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
{:key value :key2 value2} -> (hash :key value :key2 value2)
'(x) -> (quote (x))
(let (x 2 y '(3 4)) `(+ ,x @y)) -> (+ 2 3 4)
%{:weak}(list 12 14) => (12 14) ; metadata is hidden 
```

Too many special forms?

## Features

- Restarts
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