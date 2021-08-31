# Why TODO might be better than an Abstract Base Class

Author: David Beazley ([@dabeaz](https://www.dabeaz.com)),
August 31, 2021

A highly underrated programming technique is the power of wishful thinking.  Too often, we're inclined to solve problems using only the tools at hand.  Thus, when the kids ask you for a puppy, they get disappointed when you hand them ZipPandaTurtle and say "look, it's fast, furry and it can turn left."

No, in such situations, it's often more empowering to just make stuff up.  If you want a puppy, make one:

```
class Puppy:
    def sit(self):
        ...
    
    def stay(self):
        ...

    def drop(self):
        ...

    def come(self):
        ...
```

Of course, the devil is in the details.  What do you mean just "make one?"  I have no idea how to make a puppy.  That's a fair point, but I might have some idea about the commands that I'd actually give a puppy.  Call this the puppy "interface" if you wish.

Aha!  Interfaces!  Clearly, this is the right place to introduce an abstract base class:

```
from abc import ABC, abstactmethod

class AbstractPuppy(ABC):
    @abstractmethod
    def sit(self):
        pass

    @abstractmethod
    def stay(self):
        pass

    @abstractmethod
    def drop(self):
        pass

    @abstractmethod
    def come(self):
        pass
```

No! No! No! WHAT are you doing here?   The minute you do this, you're opening yourself up to all sorts of horrible dystopian futures.  For example, are we now talking about the possibility of having multiple puppies? (HELL NO).   Also, if some lunatic walks in off the street with a duck that can carry out all of the required commands, are we going to allow them to claim that it's as good as a puppy by using `AbstractPuppy.register()`?  No, we're not.  Stop it!

But doesn't an abstract base class give you some kind of extra protection?   For example, if you don't implement all of the required methods, you can't even create an instance.  So, naturally you're only going to get a puppy if it's born into this world already knowing all of those commands.  "Sorry kids, that's just how it works." 

Maybe a more sensible approach is to simply accept the reality of puppies.  Start with a basic class like this:

```
class Puppy:
    def sit(self):
        TODO
    
    def stay(self):
        TODO

    def drop(self):
        TODO

    def come(self):
        TODO
```

Wait, what is that `TODO` just stuck in there?  That's a programming error!  If you create a `Puppy` and try one of the commands your puppy will crash.  

```
>>> spot = Puppy()
>>> spot.sit()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "puppy.py", line 3, in sit
    TODO
NameError: name 'TODO' is not defined
>>> 
```

Perfect.  That's exactly what I want.  A crash--it gets my attention. So does the fact that `TODO` is an undefined variable.  It lights up code linters and IDEs like a colorful holiday display.  A not-so-gentle reminder that you'd better get around to implementing "sit" and "stay" sooner rather than later.  

As you work on your puppy, maybe you realize that you could just give your puppy a nice name.  Let's call her "Mabel."

```
class Mabel:
    def sit(self):
        ...
    
    def stay(self):
        ...

    def drop(self):
        ...

    def come(self):
        ...
```

Heck, that even sounds pretty good as a type-hint:

```
def walk(what: Mabel):
    ...
```

Or better yet, as a descriptive argument name:

```
def walk(mabel):
    ...
```

Now, if you come home and find your whole code base covered with the former stuffing from a donut plush toy, you'll know who did it.  "Mabel! Bad dog!"

Anyways, the idea of just sticking a `TODO` in a plain class is something that a student suggested to me recently.   It's since grown on me.   In many cases, the whole reason why you've written a class is that you're trying to work out some problem/design details.  Maybe it makes sense to have a puppy--a singular puppy, not a whole framework of puppies.  Thus, why make the code much more complicated than it needs to be?

I concur. In the end, you're probably not even going to need that abstract base class.   Should the situation change, you can always add it later.  In the meantime, go outside and play frisbee with Mabel. 

## Discussion

No comments.  Want to make a comment?  Edit this page. Then submit a pull request. 










