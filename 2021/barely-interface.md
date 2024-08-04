# Barely an Interface

Author: David Beazley ([@dabeaz](https://www.dabeaz.com)),
September 3, 2021

In a [recent installment](todo-abc.md), I talked about why you might not want to use an Abstract Base Class.  However, sometimes you really do want to define an interface.  Thus, it might make sense to define an ABC.  For example:

```python

from abc import ABC, abstractmethod

class AbstractStream(ABC):
    @abstractmethod
    def send(self, data):
        pass

    @abstractmethod
    def receive(self):
        pass
```

An abstract base class guarantees that instances of subclasses implement a set of required methods.  For example, if you make a typo, you'll get an informative error:

```python
class Stream(AbstractStream):
    def send(self, data):
        pass

    def recv(self):
        pass

>>> s = Stream()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't instantiate abstract class Stream with abstract method receive
>>> 
```

However, what constitutes the parts of an "interface?"   Surely, the method names are important, but what about the parameter names?  For example, what if you define this class?

```python
class Stream(AbstractStream):
    def send(self, stuff):      # Note: renamed parameter
        pass

    def receive(self):
        pass
```

In this case, it's perfectly legal to create a `Stream`.  However, the interface is broken if you ever try to use a keyword argument:

```python
>>> s = Stream()
>>> s.send(data='hello')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: send() got an unexpected keyword argument 'data'
>>> 
```

Are keyword arguments part of the "contract" that should enforced by an interface?  I'd claim that most programmers would say "yes."  It's the kind of thing that a compiler might find.  Indeed, if you run a separate tool like `pylint` you'll get a warning:

```
W0237: Parameter 'data' has been renamed to 'stuff' in overridden 'Stream.send' method (arguments-renamed)
```

Getting back to Python itself though, many people are surprised to learn that abstract base classes don't enforce anything other than the mere existence of names.  Thus, you can define a class like this and it will pass the abstract base class runtime "test":

```python
class Stream(AbstractStream):
    send = None
    receive = None

s = Stream()     # Works
s.send('hello')  # Fails (obviously)
```

Needless to say, there isn't much to the concept of an "interface" as far as ABCs are concerned.  If your view of an interface is that it defines a kind of contract related to usage, you'll find that ABCs fall pretty far short.

ABCs also fall short if you try to use them with more unusual kinds of programming patterns--especially those involving class or static methods.   To illustrate, here is a more complex example involving an object with different runtime states:

```python
class Stream:
    def __init__(self):
        ClosedState.enter(self)

    def open(self):
        return self.mode.open(self)

    def close(self):
        return self.mode.close(self)

    def send(self, data):
        return self.mode.send(self, data)

    def receive(self):
        return self.mode.receive(self)

# -- Interface for different Stream states

class StreamState:
    @staticmethod
    def enter(stream):
        raise NotImplementedError()

    @staticmethod
    def open(stream):
        raise NotImplementedError()

    @staticmethod
    def close(stream):
        raise NotImplementedError()

    @staticmethod
    def send(stream, data):
        raise NotImplementedError()
    
    @staticmethod
    def receive(stream):
        raise NotImplementedError()

# -- Open mode implementation

class OpenState(StreamState):
    @staticmethod
    def enter(stream):
        stream.mode = OpenState

    @staticmethod
    def open(stream):
        raise RuntimeError('Already open')

    @staticmethod
    def close(stream):
        ClosedState.enter(stream)

    @staticmethod
    def send(stream, data):
        print('Sending', data)

    @staticmethod
    def receive(stream):
        print('Receiving')

# -- Closed mode implementation

class ClosedState(StreamState):
    @staticmethod
    def enter(stream):
        stream.mode = ClosedState

    @staticmethod
    def open(stream):
        OpenState.enter(stream)

    @staticmethod
    def close(stream):
        raise RuntimeError('Already closed')

    @staticmethod
    def send(stream, data):
        raise RuntimeError('Not open')

    @staticmethod
    def receive(stream):
        raise RuntimeError('Not open')
```

In this example, the `StreamState` class is serving as an interface.  You might be inclined to make it an abstract base class.  However, doing so has no useful effect at all.  The extra checks that an ABC provide only take place at the time instances are created.  In this case, there are no instances--it's all static methods.  So, you're out of luck.

There is a potential fix if you define `StreamState` with an extra `__init_subclass__()` method like this:

```python
import inspect

class StreamState:
    @classmethod
    def __init_subclass__(cls):
        for name in ['enter','open','close','send','receive']:
            assert (getattr(cls, name) is not getattr(StreamState, name) and 
                    inspect.signature(getattr(cls, name)) == 
                    inspect.signature(getattr(StreamState, name)))

    @staticmethod
    def enter(stream):
        pass

    @staticmethod
    def open(stream):
        pass

    @staticmethod
    def close(stream):
        pass

    @staticmethod
    def send(stream, data):
        pass
    
    @staticmethod
    def receive(stream):
        pass
```

As it turns out, this is a pretty strong check--much stronger than an abstract base class.  It checks that all of the required methods have been defined and it makes sure that their calling signatures match exactly.   Moreover, these checks occur at the time of class definition--not instance creation.  Thus, your code won't even import or run unless it's defined correctly.

Obviously, you could probably do a bit more to clean up the whole `__init_subclass__()` hook used in this example (better error messages, etc.).

## Is it actually worth it?

I think it's valid to ask if defining a special kind of abstract base class is even worth the extra ceremony involved.   First, what is the overall purpose of defining such a class in the first place?   If the goal is merely organizational, then defining a normal top-level class conveys the same intent and involves a lot less to think about (e.g., no extra imports, decorators, or hidden metaclasses). 

```python
class AbstractStream:
    def send(self, data):
        raise NotImplementedError()

    def receive(self):
        raise NotImplementedError()

class Stream(AbstractStream):
    def send(self, data):
        ...

    def receive(self):
        ...
```

If the goal is to have extra error checking, does providing "early" runtime error detection actually provide much benefit beyond simply raising a runtime exception and letting bad code crash?  Or is it any better than using a code-linter which still reports a suitable warning for the above code even when you don't define it as a proper ABC?  Plus, it seems unlikely that someone would write code and never test it by, well, actually running it.  So, perhaps the purported benefits of using an ABC are more theoretical than practical.  

If anything, keeping things simple is often a good policy.  If you start off with a plain interface class it still conveys your intent.  If needed, it can always be upgraded to a ABC later.  Or not. 

## Discussion

@icing: the dimension in code most overlooked is time. the interface/baseclass will need to evolve in ways
inforeseen today. If it is 'private', not visible outside your module/package, you retain every freedom to
modify both in any way.

If your interface/baseclass is visible to the outside, the interface is orders of magnitude less dangerous
to change than the 'abstract' base class. Especially when the abstract baseclass has any 'default' implementation
of a method (the 'code reuse' pit). These default implementation parts are never documented, and yet are part
of the contract to any inheritor of the base class.

The most brittle is the initialisation phase. When the constructors run top to bottom, it is often assumed
that all methods function that are part of the interface. Most implementation inheritancers become very surprised
when their class methods get called before the constructor has run.

Lots of pitfalls in your own code, but when the hierarchy is split among release lines, the behaviour
of the "other side" can change any time and break you. Not worth it.


Want to make a comment?  Edit this page. Then submit a pull request. 



