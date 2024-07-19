# How to Put Everything in `__init__.py`

Author: David Beazley ([@dabeaz](https://www.dabeaz.com))  
Date: July 18, 2024

Inspired by
[this](https://mastodon.social/@AlSweigart/112802553108554869) brief
exchange with Al Sweigert, I got to thinking about the problem of how
you could put all of your code in `__init__.py` and get away with it.

## Starting Out

Naturally, you need to have a cool name for your package. So, how
about `ipyp`?  What's cool about that?  It's simply "pypi" reversed,
which is, admittedly, not THAT cool, but it does seem have the feature of not
presently being taken.

Naturally, you need to put all of your code in `__init__.py` as you
usually do. For example, let's assume you've got some classes and a
function:

```python
# ipyp/__init__.py

class A:
    def yow(self):
        print('A.yow')
        
class B:
    def spam(self):
        print('B.spam')
        
def blah():
    print('blah')
```

## Reversing the Imports

A common use of `__init__.py` is to consolidate the contents of different files
into a central location. For example, it's common to see code like this:

```python
# __init__.py

from .a import *
from .b import *
...
```

As an example, you can look at [`__init__.py`](https://github.com/python/cpython/blob/main/Lib/asyncio/__init__.py) from `asyncio`.

However, that's not what we're doing here.  Instead of `__init__.py`
importing definitions from a submodule, I only want to make it *appear*
as if code is properly spread out across submodules when, in fact, it is
not.   To do this, I want to create a kind of "fake" submodule that actually
imports its definitions from `__init__.py`.  Since that's a slightly different
idea, I'm going to call such submodules a "cosubmodule."

To do that, make your cosubmodule look like this:

```python
# ipyp/a.py

__getattr__ = lambda name: (
    getattr(__import__('sys').modules[__package__], name)
    )
```

With this, you'll find that you can load anything in `__init__.py` using 
a more specific submodule import.  Like this:

```python
>>> from ipyp.a import A
>>> a = A()
>>> a.yow()
A.yow
>>>
```

Is the class `A` actually defined in `ipyp/a.py`?  Does it matter?

## Wishful Thinking

In a very large project, you'll probably want to have a lot of
cosubmodules in the spirit of teamwork.  So, the easiest thing to do
is to simply step away from the keyboard for a moment and reflect upon
the structure you WISH your project had.  Break up the code in your mind and
create some files.  Or better yet, just some symbolic links:

```
bash % cd ipyp
bash % ln -s a.py b.py
bash % ln -s a.py funcs.py
bash % ln -s a.py util.py
```

In this example, the file `a.py` would be called a terminal cosubmodule
since that's where everything ultimately links.

You'll now be able to import things exactly as you wished it worked:

```python
>>> from ipyp.a import A
>>> from ipyp.b import B
>>> from ipyp.funcs import blah
>>> a = A()
>>> a.yow()
A.yow
>>> b.spam()
B.spam
>>> blah()
blah
>>>
```

At some point, your coworker will probably complain "I thought that
`blah` was defined in `util`?"  Yes.

```python
>>> from ipyp.util import blah
>>> blah()
blah
>>>
```

## Being Sneaky

Just to emphasize, all of your code is still in `__init__.py`.  Anyone who
looks in there is going to see it.   It's hard to avoid that, but you
might be able to trick people.   Change your `__init__.py` file so that
it has the usual submodule import trick at the top.  However, insert a few hundred
blank lines afterwards so that the real code remains hidden from view 
if loaded into an editor.

```
# __init.py

from .a import *
from .b import *
from .funcs import *
from .util import *











(... several hundred blank lines continue ...)






# Now the real code
class A:
    def yow(self):
        print('A.yow')
        
class B:
    def spam(self):
        print('B.spam')
        
def blah():
    print('blah')
```

## Other Benefits

By having all of your code in one file, things will probably load a
bit faster.  And goodbye circular imports!

It's also easier to refactor the code.  Want to move a definition to a
different file?  Just change your import statement and create a new
symbolic link (if needed).  It doesn't get any easier.

## Downsides

I honestly can't think of any, but if pressed, just tell people that you're
using cosubmodules.

"Is that something related to coroutines?"

"Yes. For performance."

"Oh!"


















	
