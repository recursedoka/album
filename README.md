# Album

Album (meaning list in Latin) is a Lisp dialect that attempts to gather the best
ideas and features from other Lisp dialects and bring them into a single, robust
environment that encourages continual experimentation and improvement for both
the language and applications written on the language without sacrificing
stability or performance. It is meant for both developing new, innovative
technology and bringing that technology to market under the ideology that
companies that go directly from concept to concrete ideas the fastest maintain a
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
a:b:c -> (lambda args (a (b (apply c args))))
-.6.3.1 -> (- 6 3 1) => 2
(let b 2 list!a.b!c) -> (let b 2 (list 'a b 'c)) => (a 2 c)
@fut -> (deref fut)
^4 -> (lambda (n) (expt n 4))
;comment ->
(let |'woah| 2 |'woah|) => 2
(let a {:b 6} a/b) -> (let a {:b 6} (a :b)) => 6

(func args ...) => calls func with args
[+ _ 2] -> (lambda (_) (+ _ 2))
{:key value :key2 value2} -> (hash :key value :key2 value2)
'(x) -> (quote (x))
(let (x 2 y '(3 4)) `(+ ,x @y)) -> (+ 2 3 4)
```