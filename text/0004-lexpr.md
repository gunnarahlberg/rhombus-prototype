
XXX Numbers

```lexpr
1
```
```sexpr
((1))
```

XXX Symbols

```lexpr
x
```
```sexpr
((x))
```

XXX Characters

```lexpr
#\

```
```sexpr
((#\newline))
```

```lexpr
#\a
```
```sexpr
((#\a))
```

XXX Sequences

```lexpr
x y
```
```sexpr
((x y))
```

XXX Dots

```lexpr
x.y
```
```sexpr
(((#%dot x y)))
```

```lexpr
x.y.z
```
```sexpr
(((#%dot x y z)))
```

```lexpr
x.y z
```
```sexpr
(((#%dot x y) z))
```

XXX Applications

```lexpr
f(x)
```
```sexpr
(((#%fun-app f (x))))
```

```lexpr
f(x, y)
```
```sexpr
(((#%fun-app f (x y))))
```

XXX Member

```lexpr
f[x, y]
```
```sexpr
(((#%member f (x y))))
```

XXX Param

```lexpr
f<x, y>
```
```sexpr
(((#%param f (x y))))
```

```lexpr
x < y > z
```
```sexpr
((x < y > z))
```

```lexpr
f<A, B>(x, y)[1, 2]
```
```sexpr
(((#%member (#%fun-app (#%param f (A B)) (x y)) (1 2))))
```

XXX Grouping

```lexpr
x + 6 * y
```
```sexpr
((x + 6 * y))
```

```lexpr
(x + 6) * y
```
```sexpr
(((x + 6) * y))
```

XXX Quotation

```lexpr
x + '(y * 6) + z
```
```sexpr
((x + (#%quote (y * 6)) + z))
```

```lexpr
x + 'x.y + z
```
```sexpr
((x + (#%quote (#%dot x y)) + z))
```

```lexpr
x + 'f(x) + z
```
```sexpr
((x + (#%quote (#%fun-app f (x))) + z))
```
 
XXX Text quotation

```lexpr
{Hello World!}
```
```sexpr
(((#%text (("Hello World!")))))
```

```lexpr
{Hello @(1 + 2)!}
```
```sexpr
(((#%text (("Hello " (#%text-esc (1 + 2)) "!")))))
```

```lexpr
{Hello

 World!}
```
```sexpr
(((#%text (("Hello") () (" World!")))))
```

```lexpr
{}
```
```sexpr
(((#%text (()))))
```

```lexpr
{@1 + 2!}
```
```sexpr
(((#%text (((#%text-esc 1) " + 2!")))))
```

```lexpr
{This is a { embedded brace! } }
```
```sexpr
(((#%text (("This is a " "{" " embedded brace! " "}" " ")))))
```

XXX Line follower: \n

```lexpr
foo bar
zig zag
```
```sexpr
((foo bar) (zig zag))
```

```lexpr
foo bar

zig zag
```
```sexpr
((foo bar) (zig zag))
```

XXX Line follower: \

```lexpr
foo \
  bar
zig zag
```
```sexpr
((foo bar) (zig zag))
```

```lexpr
foo \
  bar baz
zig zag
```
```sexpr
((foo bar baz) (zig zag))
```

```lexpr
foo \
  bar
  baz
zig zag
```
```sexpr
((foo bar baz) (zig zag))
```

```lexpr
foo \
  bar \
    baz
zig zag
```
```sexpr
((foo bar baz) (zig zag))
```

XXX Line follower: :

```lexpr
if x < y :
  f(x)
else :
  g(y)
```
```sexpr
((if x < y 
   (#%indent (((#%fun-app f (x)))))
  else 
   (#%indent (((#%fun-app g (y)))))))
```

```lexpr
a :
  b \
    c
  d
f
g
```
```sexpr
((a (#%indent ((b c) (d))) f) (g))
```

XXX Line follower: @

```lexpr
foo bar @
  This is just some text!
  It can have many lines
  And @(4 - 3) quote!
baz
```
```sexpr
((foo
  bar
  (#%text (("This is just some text!")
           ("It can have many lines")
           ("And " (#%text-esc (4 - 3)) " quote!")))
  baz))
```

XXX Line follower: |

XXX Line quotation

```lexpr
foo bar [zig zag]
```
```sexpr
((foo bar (#%quote-line (zig zag))))
```

XXX More line quotation

XXX

Line expressions (`lexpr`) are primarily line-oriented, but they embed
two other kinds of expressions: individual and sequence expressions,
both which are made up of atomic expressions.

White space is a sequence of space characters and comments. Comments
are either line oriented with `//`, block oriented with `/*` and
`*/`, or expressions which start with `#;` and consume an individual
expression.

An atomic expression is a single base object, like a symbol or
number.

An operator is a sequence of special operator characters.

A text expression is like the Racket @-reader, except that it is
indexed by an ignored prefix of characters, typically a sequence of
spaces.

Individual expressions (`iexpr`) are white-space-free prefixes of
characters (modulo comments), which may embed sequence
expressions. They can also include text expressions with `{`

```bnf
iexpr := atom
       | operator
       | iexpr '.' atom
       | iexpr '(' qexpr ')'
       | iexpr '[' qexpr ']'
       | iexpr '<' qexpr '>'
       | '(' qexpr ')'
       | '{' text[ϵ] '}'
```

XXX Single (`'`) and line (`[]`) quotations

Sequence expressions (`qexpr`) are sequences of individual expression
separated by whitespace. Operators effect the parsing of sequence
expressions.

```bnf
qexpr  := ϵ
        | iexpr q*expr

q*expr := ϵ
        | WS qexpr
```

Line expressions are indexed by a prefix which they must present and a
prefix that must occur in embedded expressions. They are a sequence of
this prefix, an individual expression, a sequence expression, and a
line tail. The body of a module is parsed as a sequence of line
expressions with no prefix.

```bnf
lexpr[pre, tailpre] := pre iexpr q*expr ltail[tailpre]
```

There are a variety of line tails:

A newline, which terminates a line expression.

```bnf
ltail[tailpre] := .... | '\n'
```

A `:`, which visibly embeds a sequence of line expressions at one
higher level of indentation, followed by an optional line expression
at the same level of indentation.

```bnf
ltail[tailpre] := .... | ':' WS* '\n' lexpr[tailpre SP SP, tailpre] * lexpr[tailpre, tailpre]?
```

For example,

```lexpr1
if x < y :
  "Left"
else :
  "Right"
```

is parsed as

```sexpr
(if (x < y) (: ("Left")) else (: ("Right")))
```

A `&`, which invisibly embeds a sequence of line expressions at the
same level of indentation.

```bnf
ltail[tailpre] := .... | '&' WS* '\n' lexpr[tailpre, tailpre] *
```

For example,

```lexpr1
begin &
a
b
c
```

is parsed as

```sexpr
(begin (a) (b) (c))
```

A `\`, which invisibly continues the line expression at one
high level of indentation.

XXX update BNF

```bnf
ltail[tailpre] := .... | '\' WS* '\n' lexpr[pre SP SP, tailpre] *
```

For example,

```lexpr1
begin \
  a
  b
  c
```

is parsed as

```sexpr
(begin a b c)
```

A `|`, which visibly embeds a sequence of line expressions with
aligned `|`s. (The notation in the following BNF is lacking.)

```bnf
ltail[tailpre] := .... | '|'@col lexpr[ϵ, tailpre']? lexpr[pre, tailpre'] *
  where      pre = SP{col-1} '|'
        tailpre' = SP{col}
```

For example,

```lexpr1
data List | Empty
          | Cons(a, b)
```

is

```sexpr
(data List (#%bar (Empty) (#%app Cons (#%comma a b))))
```

and

```lexpr1
define length(l) :
  match l with \
    | Empty => 0
    | Cons(a, b) => 1 + length(b)
```

is parsed as

```sexpr
(define (#%app length (l))
  (: (match l with 
       (#%bar (Empty => 0)
          ((#%app Cons (#%comma a b)) => 1 + (#%app length b))))))
```

A `@`, which embeds an indented text block.

```bnf
ltail[tailpre] := .... | '@' WS* '\n' text[tailpre]
```

For example,

```lexpr1
datalog @
  import "family.log"
  add(X, @5, Y)?
```

is parsed as

```sexpr
(datalog (#%text (list (list "import \"family.log\"")
                       (list "add(X, " (#%text-esc 5) ", Y)?"))))
```

A `@{`, which embeds an indented text block that is
terminated with `}` and continues the line expression.

```bnf
ltail[tailpre] := .... | '@{' WS* '\n' text[tailpre] `}` lexpr[tailpre, tailpre]
```

For example,

```lexpr1
c @{
  double log2(double x) {
   return log(x) / log(2); }
} with \
  "-lmath"
```

parses as

```sexpr
(c (#%text (list (list "double log2(double x) {")
                 (list " return log(x) / log(2); }")))
   with "-lmath")
```

XXX lots of overlap with `#lang something`: https://github.com/tonyg/racket-something