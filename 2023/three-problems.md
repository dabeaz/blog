# Now You Have Three Problems

Author: David Beazley ([@dabeaz](https://www.dabeaz.com))  
Date: March 3, 2023

There's a popular programming trope that if the solution to your
problem involves parsing text with a regular expression, you now have
two problems.  Some programmers read this and think to try a different
approach.  Perhaps you don't actually need a regex.  Maybe the problem
can be solved with a simple string split or something.  However,
others might think a bit harder and wonder "what if I did something so
audacious that it resulted in three problems?"  This post is in the
spirit of that! 

The code that follows is Python, but it could be adapted to any
language with support for higher-order functions.

## Elementary Particles

If you wander out into the forest and think hard about parsing, you'll
realize that a big part of it involves consuming input. Let's write a
function to do that:

```python
def shift(inp):
    return bool(inp) and (inp[0], inp[1:])
```

Given an input sequence `inp`, this returns the first item `inp[0]` and all of
the remaining items `inp[1:]`.   Alternatively, it returns `False` if there's no
input at all.   It looks weird--almost like a strange string split.  Here's
how it might be used to step through a string:

```pycon
>>> shift('bar')
('b', 'ar')
>>> shift('ar')
('a', 'r')
>>> shift('r')
('r', '')
>>> shift('')
False
>>>
```

For every function, it's always a good idea to have some kind of
opposing function. At least, that's what I learned working on physics
software. What is the opposite of consuming input?  Not consuming
input of course.  Let's write that:

```python
def nothing(inp):
    return (None, inp)
```

`nothing()` doesn't advance. It returns `None` from
any input. It also returns the same input as it received (unmodified).

```pycon
>>> nothing('bar')
(None, 'bar')
>>>
```

`nothing()` is different than having no input available.  It just
means that you're choosing **NOT** to do anything with the input
that's there.

Both of these functions are instances of what I'll call a "parser."  A
parser is a function defined by its calling signature and return
convention.  Specifically, a parser is any function that accepts some input (`inp`) and on
success, returns a tuple `(value, remaining)` where `value` is some
value of interest and `remaining` represents all of the remaining
input still to be parsed. On failure, a parser returns `False`.

Although both of these functions are already short, you can make them
even shorter using `lambda` like this:

```python
shift   = lambda inp: bool(inp) and (inp[0], inp[1:])
nothing = lambda inp: (None, inp)
```

`lambda` has the benefit of making the code compact and foreboding.
Plus, it prevents people from trying to add type-hints to the
beautiful thing that is about to unfold.

By the way, `lambda` is the third problem.  Shifting nothingness and
lambda--the three problems.  Let's proceed.

## A Parsing System

What we have here is a system for parsing.  As with any system, there are
rules.  In this system, you are only allowed to use the two parsers
provided (`shift` and `nothing`).  You are also allowed to use
`lambda` to create new parsers from existing parsers.  That's it. 

## Parser Modifiers

Let's write a rule that applies a predicate to the result of a parser:

```python
filt = lambda predicate: (
         lambda parser:
           lambda inp: (m:=parser(inp)) and predicate(m[0]) and m)
```

Three lambdas?!? The walrus operator? Short-circuit evaluation? What
fresh hell?  Yes, I could have used four lambdas instead:

```python
filt = lambda predicate: (
         lambda parser:
           lambda inp:
             (lambda m: m and predicate(m[0]) and m)(parser(inp)))
```

However, readability is important.  Besides, good things come in
threes. Three problems. Three lambdas. Three tusks. Not four lambdas
and no tusks.

The `filt()` function takes a predicate and a parser as input and
combines them together to form a new parser.  It looks a little weird,
but it works kind of like a decorator. Here's an example of how you use
it:

```pycon
>>> digit = filt(str.isdigit)(shift)
>>> letter = filt(str.isalpha)(shift)
>>> digit('456')
('4', '56')
>>> letter('456')
False
>>>
```

The return value of `False` in this case means that a parse didn't
work. Just as `shift()` returns `False` when there is no more input, the
parser created by `filt()` returns `False` if the value produced 
doesn't match the given predicate.

The funny formulation of `filt()` allows you to easily make other
kinds of useful filters by only supplying custom predicates.  Here's a
filter for exactly matching a literal:

```python
literal = lambda value: filt(lambda v: v == value)
```

Here's a filter where values must come from a predefined set of
acceptable values:

```python
memberof = lambda values: filt(lambda v: v in values)
```

Here's are some examples of applying these filters to an existing
parser:

```pycon
>>> dot = literal('.')(shift)
>>> even = memberof('02468')(digit)   # Yes, digit, not shift.
>>> dot('.456')
('.', '456')
>>> dot('45.6')
False
>>> even('456')
('4', '56')
>>> even('345')
False
>>>
```

Just as the purpose of `filt()` is to ignore things, the opposite of
ignoring something is to do something.  Hence, we need an
opposing force to keep the code balanced.  Let's call that
operation `fmap()`:

```python
fmap = lambda func: (
         lambda parser:
           lambda inp: (m:=parser(inp)) and (func(m[0]), m[1]))
```

`fmap()` takes a function and parser as input and creates a new parser
wherein the supplied function is applied to a successful parse.  You
use `fmap()` to transform values. For example:

```pycon
>>> ndigit = fmap(int)(digit)
>>> ndigit('456')
(4, '56')
>>> tenx = fmap(lambda x: 10*x)
>>> tenx(ndigit)('456')
(40, '56')
>>> tenx(digit)('456')
('4444444444', '56')
>>>
```

*Digression:* `map` and `filter` are the names of built-in functions
that Python uses to operate on iterables.  I would have used those,
if not for the resulting naming confusion.  Hence, I've chosen `fmap`
and `filt` instead. Conceptually, our functions serve a 
semantically similar purpose however.

## Repetition

So far, our parsers only work with a single input.  To do more
interesting things, you would want to match multiple inputs.  For
example, multiple digits or multiple letters.  It would be great to be
able to define something like this:

```python
digits = one_or_more(digit)
```

To do this, we could dive into some functional techniques involving
recursion.  However, let's have some straight talk--Python sucks at
recursion for various "reasons", the least of which is its internal
recursion limit.  You don't want to do that.  So, we're going take a
bit of inspiration from theater and "break the fourth wall" where we
turn to the audience, wink, and acknowledge that we're still actually
coding in Python.  Ok, fine:

```python
def one_or_more(parser):
    def parse(inp):
        result = [ ]
        while (m:=parser(inp)):
            value, inp = m
            result.append(value)
        return bool(result) and (result, inp)
    return parse
```

Yes, a `while` loop doesn't fit into our whole system of
lambdas. However, one could argue that it's "pythonic" solely for the
fact that it doesn't blow the recursion limit. So, let's roll with
it. Like our other functions, `one_or_more()` accepts a parser as
input and creates a new parser as output. It calls the supplied parser
repeatedly until no more matches can be made.  A list is produced.

```pycon
>>> digit = filt(str.isdigit)(shift)
>>> digits = one_or_more(digit)
>>> digits('456')
(['4','5','6'], '')
>>> digits('1abc')
(['1'], 'abc')
>>> digits('abc')
False
>>>
```

If you don't like the digits separated into a list, use `fmap` to
combine them back together:

```pycon
>>> digits = fmap(''.join)(one_or_more(digit))
>>> digits('456')
('456', '')
>>>
```

If you want a numeric value instead, add one more `fmap` to the affair:

```pycon
>>> value = fmap(int)(digits)
>>> value('456')
(456, '')
>>>
```

*Exercise:* Rewrite `one_or_more()` using nothing more
 than lambda and recursion.

## Sequencing

Sometimes you want to parse one thing after another.  You can write a
sequencing operator like this:

```python
def seq(*parsers):
    def parse(inp):
        result = [ ]
        for p in parsers:
            if not (m:=p(inp, n)):
                return False
            value, inp = m
            result.append(value)
        return (result, inp)
    return parse
```

`seq()` takes an arbitrary number of parsers as input.  It then creates
a new parser where they are sequenced, one after the other.  All parsers must
succeed to have a successful parse.  Here's an example:

```pycon
>>> seq(letter, digit, letter)('a4x')
(['a', '4', 'x'], '')
>>> seq(letter, digit, letter)('abc')
False
>>> seq(letter, fmap(''.join)(one_or_more(digit)))('x12345')
(['x', '12345'], '')
>>> 
```

*Exercise:* Write a version of `seq()` that uses recursion and lambda.

## Choice

Sequencing requires all parsers to match.  What is the opposing operation?
Perhaps it's a function that only requires one of the given parsers to
match.  Let's call this operation `either()`:

```python
either = lambda p1, p2: (lambda inp: p1(inp) or p2(inp))
```

Here is an example:

```pycon
>>> alnum = either(letter, digit)
>>> alnum('4a')
('4', 'a')
>>> alnum('a4')
('a', '4')
>>> alnum('$4')
False
>>>
```

`either()` allows you to build optionals and finally gives you a
chance to use `nothing`.  For example:

```python
maybe = lambda parser: either(parser, nothing)
```

For example:

```pycon
>>> maybe(digit)('456')
('4', '56')
>>> maybe(digit)('abc')
(None, 'abc')
>>>
```

You can also use it to implement `zero_or_more()`:

```python
zero_or_more = lambda parser: either(one_or_more(parser), seq())
```

```pycon
>>> zero_or_more(digit)('456')
(['4','5','6'], '')
>>> zero_or_more(digit)('abc')
([], 'abc')
>>>
```

Last, but not least, you can use `either()` to build a more capable
`choice()` function that allows you to choose between any number
of supplied parsers.

```python
choice = lambda parser, *parsers: (
           either(parser, choice(*parsers)) if parsers else parser)
```

*Exercise:* Rewrite `choice()` to not use recursion.

*Exercise:* Do you actually need `nothing`?  Can you build `nothing` from something?

## Example: Numbers

Let's look at the problem of parsing numbers.  Suppose that numbers
come in 2 different varieties. Integers such as `1234` and decimals
such as `12.34`.  Decimals, however, are a bit more complicated
because they can be written with a trailing decimal like `12.` or with
a leading decimal like `.34`.  Suppose that you want to convert
integers to a Python integer and decimals to a Python float.  How
would you parse any number? Here's how you might do it:

```python
dot = literal('.')(shift)
digit = filt(str.isdigit)(shift)
digits = fmap(''.join)(one_or_more(digit))
decdigits = fmap(''.join)(choice(
               seq(digits, dot, digits),
               seq(digits, dot),
               seq(dot, digits)))

integer = fmap(int)(digits)
decimal = fmap(float)(decdigits)
number = choice(decimal, integer)
```

Let's try our `number()` function.

```pycon
>>> number('1234')
(1234, '')
>>> number('12.3')
(12.3, '')
>>> number('.123')
(0.123, '')
>>> number('123.')
(123.0, '')
>>> number('.xyz')
False
>>>
```

## Example: Key-value pairs

Suppose you want to parse key-value pairs of the form `name=value;` where
`name` consists of letters and `value` is any numeric value.  Further
suppose that there could be arbitrary whitespace around any of the
parts (which should be ignored).  Here's how you might do it:

```python
letter = filt(str.isalpha)(shift)
letters = fmap(''.join)(one_or_more(letter))
whitespace = zero_or_more(filt(str.isspace)(shift))
eq = literal('=')(shift)
semi = literal(';')(shift)
ws = lambda parser: (
        fmap(lambda p: p[1])
        (seq(whitespace, parser, whitespace)))
name = ws(letters)
value = ws(number)
keyvalue = fmap(lambda p: (p[0], p[2]))(seq(name, eq, value, semi))
```

Let's try it out:

```
>>> keyvalue('xyz=123;')
(('xyz', 123), '')
>>> keyvalue('   pi = 3.14  ;')
(('pi', 3.14), '')
>>>
```

The handling of whitespace might require a bit of study. The key
is the `ws()` function that accepts a parser and creates a new
parser that accepts and discards any leading/trailing whitespace.

## Example: Building a dictionary

Suppose you want to extend your parser so that it converts an
arbitrary number of key-pair pairs written as `key1=value1;
key2=value2; key3=value3;` into a Python dictionary with the same keys
and values. Here's how to do it:

```python
keyvalues = fmap(dict)(zero_or_more(keyvalue))
```

Example:

```pycon
>>> keyvalues('x=2; y=3.4; z=.789;')
({'x': 2, 'y': 3.4, 'z': 0.789}, '')
>>> keyvalues('')
({}, '')
>>>
```

## Example: Validating dictionary keys

Suppose you want to write a parser that only accepts dicts with keys `x` and `y`.
You can use `filt()` to check like this:

```python
xydict = filt(lambda d: d.keys() == {'x', 'y'})(keyvalues)
```

Example:

```pycon
>>> xydict('x=4;y=5;')
({'x': 4, 'y': 5}, '')
>>> xydict('y=5;x=4;')
({'y': 5, 'x': 4}, '')
>>> xydict('x=4;y=5;z=6;')
False
>>>
```

This example illustrates how features compose together in interesting
ways.  Earlier, the `filt()` function was used to filter individual
characters, but now it's being applied to dictionaries.

## Discussion: Composability

The essential feature that makes everything work is an attention to
composibility. At the core, this is the interface to a parser:

```python
def parser(inp):
    ...
    if success:
        return (value, remaining)
    else:
        return False
```

Everything else builds around this.  All of the different functions
such as `filt()`, `fmap()`, `zero_or_more()`, `seq()`, and `choice()`
create new parsers that have an identical interface.  As such,
everything works with everything everywhere all at once.  Perhaps the main
area of concern would be `fmap()`.  Since that applies a user-defined function
to the parsed value, the supplied function would obviously have to be
compatible with that.

## Discussion: Decomposition of Concepts

Consider the formulation of `filt()` for a moment.  When you used
`filt()`, you probably thought that it looked a little funny. Like this:

```python
digit = filt(str.isdigit)(shift)
```

Why is `shift` on the outside like that?  Besides, isn't that more
of an internal implementation detail?  Couldn't we just hide
it like this:

```python
filt = lambda predicate: (
         lambda inp: (m:=shift(inp)) and predicate(m[0]) and m)

digit = filt(str.isdigit)
```

Yes, this could be done, but moving it inside limits the utility of
`filt()` to single characters.  I'd much prefer a `filt()` that's
dangerously flexible.  The original formulation allows a predicate to
be applied to *ANY* parser whatsoever--even more complex ones that are
returning data structures.  That's cool.  You can't do that if the
choice of parser is pulled inside.

Another question about `filt()` (and related functions) relates to its
strange calling convention.  Why are the input predicate and parser
arguments handled via separate function calls instead of being passed
together to a single function?  For example, why not this?

```python
filt = lambda predicate, parser: (
         lambda inp: (m:=parser(inp)) and predicate(m[0]) and m)

digit = filt(str.isdigit, shift)
```

Formulating the function in this way makes it more clunky to define
useful variants such as `literal`.  Previously, `literal` was
defined by only concerning itself with the predicate part.  Like this:

```python
literal = lambda value: filt(lambda v: v == value)
```

This is focused and elegant. However, if `filt()` required an
additional argument, that argument spills into outer functions,
forcing you to write code like this:

```python
literal = lambda value, parser: filt(lambda v: v == value, parser)
```

That's ugly.  The original approach doesn't require us to know any
further details about `filt()`.

## Magic

Let's discuss the `shift()` function for a moment.  As originally
formulated, it splits the input string into its first character and
all of the remaining text.  Here's the original code again:

```python
shift = lambda inp: bool(inp) and (inp[0], inp[1:])
```

And here's how it worked:

```pycon
>>> shift('hello world')
('h', 'ello world')
>>>
```

This is not an efficient way to process text in Python. In fact, it's
probably the worst way to process text that you could devise.  When
running a test on my machine, parsing a string with 100000 key-value pairs
into a dictionary takes more than 2 minutes!

The central problem is the memory copy that takes place when computing
`inp[1:]`.  In fact, every call to `shift()` makes a nearly
complete copy of the input text. Is it possible to avoid that?

An astute observer will note that in all of the code presented,
nothing is ever done with the input value `inp` except for passing it
along elsewhere.  The only code that ever looks at it is the `shift()`
function!  Moreover, no code ever looks at the value of the remaining
text either. As such, we could choose to change the data
representation of these parts to something else entirely.  Instead of
representing the input as a string, perhaps we could use a tuple
`(text, n)` where `n` is an integer representing the current position.
Let's try rewriting `shift()` like this:

```python
def shift(inp):
    text, n = inp
    return n < len(text) and (text[n], (text, n+1))
```

Here's how this new version works:

```pycon
>>> shift(('abc', 0))     # Note the use a tuple now
('a', ('abc', 1))
>>> shift(('abc', 1))
('b', ('abc', 2))
>>> shift(('abc', 2))
('c', ('abc' 3))
>>> shift(('abc', 3))
False
>>>
```

Notice how the input string never changes from step-to-step.
There is no copying and no slicing.  Python will efficiently pass the
string around as a reference. The only changing value is the integer
index.

It is not necessary to change *ANY* other code.  You can verify that
everything still works perfectly--as long as you provide your input in
the expected format.
 
```pycon
>>> keyvalues(('x=2; y=3.4; z=.789;', 0))    # Note: tuple here
({'x': 2, 'y': 3.4, 'z': 0.789}, ('x=2; y=3.4; z=.789;', 19))
>>>
```

To hide some of the input details, I might prefer to introduce a special
`Input()` function to convert the user-supplied input into my internal
format.  For example:

```python
Input = lambda inp: (inp, 0)

# Example
result = keyvalues(Input('x=2; y=3.4; z=.789'))
```

I've upper-cased `Input` to keep my options open.  Maybe it's something
that I would later change into class.  Maybe I'm just doing that
as a way to say "Phffffft!!!" to PEP-8.  Who is to say?

Nevertheless, when tested on the same 100000 key-value pair input as before,
the parsing time drops from more than 2 minutes to 2 seconds.
That's pretty amazing.   We solved our performance problem by changing the
input representation and adjusting only one line of code.

Why did this work?  I think it works because all of the of
functionality we wrote is based not on direct manipulation of input
data, but on function composition.  Changing the data representation
had no effect on the composition of parts.

## More Magic

Although we made a great improvement in performance, we're still performing
a lot of low-level single-character manipulation.  Perhaps it would make
sense to use a more proper tokenizer.   For example, maybe we could use my
[SLY](https://github.com/dabeaz/sly) tool to write a lexer like this:

```python
from sly import Lexer

class KVLexer(Lexer):
    tokens = { EQ, SEMI, NAME, INTEGER, FLOAT }
    ignore = ' \t\n'
    FLOAT = r'(\d+\.\d+)|(\d+\.)|(\.\d+)'
    INTEGER = r'\d+'
    NAME = r'[a-zA-Z]+'    
    EQ = r'='
    SEMI = r';'
```

A lexer produces tokens, not characters.  For example:

```pycon
>>> lexer = KVLexer()
>>> list(lexer.tokenize("x=2;"))
[Token(type='NAME', value='x', lineno=1, index=0, end=1),
 Token(type='EQ', value='=', lineno=1, index=1, end=2),
 Token(type='INTEGER', value='2', lineno=1, index=2, end=3),
 Token(type='SEMI', value=';', lineno=1, index=3, end=4)]
>>>
```

Could we use our parsing framework with such a tokenizer?  Certainly!
To do this, we'll replace the low-level character handling to use
tokens, but other keep the rest of the parser intact.  Here's
a new parser:

```python
expect = lambda ty:\
           fmap(lambda tok: tok.value)(filt(lambda tok: tok.type == ty)(shift))
name = expect('NAME')
integer = fmap(int)(expect('INTEGER'))
decimal = fmap(float)(expect('FLOAT'))
value = choice(decimal, integer)
keyvalue = fmap(lambda p: (p[0], p[2]))\
           (seq(name, expect('EQ'), value, expect('SEMI')))
keyvalues = fmap(dict)(zero_or_more(keyvalue))

Input = lambda inp: (list(KVLexer().tokenize(inp)), 0)
```

Let's verify that it works:

```pycon
>>> r = keyvalues(Input('x=2; y=3.4; z=.789;'))
>>> r[0]
{'x': 2, 'y': 3.4, 'z': 0.789}
>>>
```

When run on my large test input, this version runs in about 0.8
seconds, about 2.5 times faster than it did before.

## Kaboom!

I got to thinking about the countless hours I spent micro-optimizing
the LALR(1) parser in some of my other tools like
[PLY](https://github.com/dabeaz/ply) and
[SLY](https://github.com/dabeaz/sly).   Seriously, I spent a **LOT**
of time staring at that code trying to remove every last bit
of performance overhead I could think to identify.  As such, these
tools have long been one of the fastest pure-Python parser implementations around.
How would this new approach compare to THAT?

To test it out, I specified a similiar KV-pair parser in SLY:

```python
from sly import Parser

class KVParser(Parser):
    tokens = KVLexer.tokens

    @_('{ keyvalue }')
    def keyvalues(self, p):
        return dict(p.keyvalue)

    @_('NAME EQ value SEMI')
    def keyvalue(self, p):
        return (p.NAME, p.value)

    @_('INTEGER')
    def value(self, p):
        return int(p.INTEGER)

    @_('FLOAT')
    def value(self, p):
        return float(p.FLOAT)
```

Here's how you use it to parse our little example:

```pycon
>>> lexer = KVLexer()
>>> parser = KVParser()
>>> tokens = lexer.tokenize('x=2; y=3.4; z=.789;')
>>> parser.parse(tokens)
{'x': 2, 'y': 3.4, 'z': 0.789}
>>>
```

I'll now try it with my input of 100000 key-value pairs.  It takes 2.3 seconds.
It's three times slower than the last test--which used the exact same
token stream!  It's even slower than the original "magic" version that
simply worked with individual characters.  How can this be?

This was not a result I was expecting. An LALR(1) parser is driven
entirely by table-lookup and a state machine.  There is no
backtracking nor does it involve a deep stack of composed functions.

## Digression: Iteration

In Python, the concept of iteration is well defined.  It is reasonable
to ask why not use an iterator or a generator function for this?  Could
`shift()` be rewritten like this instead?

```python
shift = lambda inp: (x:=next(inp, False)) and (x, inp)
```

An experiment seems to suggest that it might work:

```pycon
>>> inp = iter('abc')
>>> shift(inp)
('a', <str_iterator object at 0x10983e140>)
>>> shift(inp)
('b', <str_iterator object at 0x10983e140>)
>>> shift(inp)
('c', <str_iterator object at 0x10983e140>)
>>> shift(inp)
False
>>>
```

Unfortunately, it doesn't actually work because our parsing framework
involves backtracking--especially when making decisions in the
`either()`, `maybe()`, and `choice()` functions.  When processing
`either()`, parsing may proceed succesfully for some time and then
suddenly fail.  When this happens, everything rewinds and a different
parsing branch is attempted.

There's no mechanism to rewind a generic Python iterator.  Although it
might be possible to copy an iterator or to use some magic from
`itertools`, doing so seems tricky. For now, I'll leave this as an exercise.

## The Complete Code

Here's the complete implementation of the basic parsing framework.
I thought it'd be interesting just to see it all in one place.

```python

# -- Parsing framework

def shift(inp):
    text, n = inp
    return n < len(text) and (text[n], (text, n+1))

nothing = lambda inp: (None, inp)

filt = lambda predicate: (
         lambda parser:
           lambda inp: (m:=parser(inp)) and predicate(m[0]) and m)

literal = lambda value: filt(lambda v: v==value)

fmap = lambda func: (
         lambda parser:
           lambda inp: (m:=parser(inp)) and (func(m[0]), m[1]))

either = lambda p1, p2: (lambda inp: p1(inp) or p2(inp))
maybe  = lambda parser: either(parser, nothing)
choice = lambda parser, *parsers: either(parser, choice(*parsers)) if parsers else parser

def seq(*parsers):
    def parse(inp):
        result = [ ]
        for p in parsers:
            if not (m:=p(inp)):
                return False
            value, inp = m
            result.append(value)
        return (result, inp)
    return parse

def one_or_more(parser):
    def parse(inp):
        result = [ ]
        while (m:=parser(inp)):
            value, inp = m
            result.append(value)
        return bool(result) and (result, inp)
    return parse

zero_or_more = lambda parser: either(one_or_more(parser), seq())

Input = lambda inp: (inp, 0)

# -- Example: Convert "key1=value1; key2=value2; ..." into a dict

# literals
dot = literal('.')(shift)
eq = literal('=')(shift)
semi = literal(';')(shift)

# numbers and values
digit = filt(str.isdigit)(shift)
digits = fmap(''.join)(one_or_more(digit))
decdigits = fmap(''.join)(choice(
               seq(digits, dot, digits),
               seq(digits, dot),
               seq(dot, digits)))

integer = fmap(int)(digits)
decimal = fmap(float)(decdigits)
number = choice(decimal, integer)

# names
letter = filt(str.isalpha)(shift)
letters = fmap(''.join)(one_or_more(letter))

# Ignore whitespace
whitespace = zero_or_more(filt(str.isspace)(shift))
ws = lambda parser: (
        fmap(lambda p: p[1])
        (seq(whitespace, parser, whitespace)))

# Name and value tokes (removed whitespace)
name = ws(letters)
value = ws(number)

# Single key=value; pair
keyvalue = fmap(lambda p: (p[0], p[2]))(seq(name, eq, value, semi))

# Multiple key-values
keyvalues = fmap(dict)(zero_or_more(keyvalue))

# Example
result, remaining = keyvalues(Input("x=2; y=3.4; z=.789;"))
print(result)

```

## Related work:

If you're looking at this whole thing with a blown mind--you should
look further into [parser
combinators](https://en.wikipedia.org/wiki/Parser_combinator).
Programming with combinators is not the most common thing in Python,
but it can be interesting way to achieve an unusual kind of extreme
flexibility.

Back in the Python world,
[PyParsing](https://pyparsing-docs.readthedocs.io/en/latest/) is based
on a similar set of concepts.  Although not rooted in functional
programming, PyParsing provides objects, operators, and primitives
that allow parsers to be built within a very similar conceptual
framework.

## Various Thoughts

Over the last twenty years, I've done a lot with parsing in Python.
This has included the development of several LALR(1) parser generator
packages and the teaching of a [compilers course](https://www.dabeaz.com/compiler.html)
where I usually have students write a recursive descent parser.  Although I had heard the
words "parser combinator" uttered before, it's not something I had any
first-hand experience with until recently.

This past January, I decided that I'd try to implement my [compilers
class](https://www.dabeaz.com/compiler.html) project in Haskell as a
side project. Having never programmed Haskell before, I had to
entirely rewire part of my brain.  To help, I started working from the 
"Programming in Haskell" book by Graham Hutton.  As it turns out,
Graham has significant experience with monadic parser
combinators (see [this paper](https://www.cs.nott.ac.uk/~pszgmh/monparsing.pdf) for instance)
and many of the examples in his book were focused on this technique.

It had never occurred to me to write a parser in this way.  Thus, most of
what you see in this post is a Python "remix" of those ideas.  To
be sure, I've taken wide liberty in utilizing Python idioms and
have put things together in a slightly different way.  However, the
code still embodies the general big idea.  I've also written the
Python code in a largely "equational style" that reflects the emphasis
on function composition as opposed to function implementation.

As an aside, sometimes people ask me "what can I do to improve my Python
skills?"  Much to their surprise, I often suggest doing a project in a
completely different language or outside of their area of expertise.
You'll almost always walk away with new ideas when you do that.

## Final Words: Would I Actually Use This?

As noted, I learned of this technique by way of Haskell.  However, I
later went on to apply it to the Python implementation of my compiler
class project.  The resulting parser was about half the size of a
hand-written recursive descent parser and involved fewer concepts.
The code also almost directly reflected the language grammar which
had been specified using a PEG.  Last, but not least, the new
implementation turned out to have a number of desirable properties
related to error handling and error messages.  In the end, I liked
it a lot more.

A more complete discussion of writing a full programming language
parser using combinators will have to wait until another time.
However, in closing--yes, I think I might actually use a technique
like this to build a parser for something real.

Feel free to leave a comment by submitting a pull request.  If you
want to learn something completely different, consider taking one
of my [courses](https://www.dabeaz.com/courses.html).

-- Dave
