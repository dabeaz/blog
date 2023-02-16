# A Different Refactoring

Author: David Beazley ([@dabeaz](https://www.dabeaz.com)),
February 15, 2023

In part of my [Advanced Programming with Python](https://www.dabeaz.com/advprog.html) course we
spend a day implementing state machines. Part of that project involves dealing with
"stateful" objects.   Here's a very simple example, taken from the [Python Cookbook](https://dabeaz.com/cookbook.html):

```python
class Connection:
    def __init__(self):
        self.state = 'CLOSED'
	    self.bytes_sent = 0

    def open(self):
        if self.state == 'OPEN':
            raise RuntimeError('Connection already open')
        elif self.state == 'CLOSED':
            self.state = 'OPEN'

    def close(self):
        if self.state == 'OPEN':
            self.state = 'CLOSED'
        elif self.state == 'CLOSED':
            raise RuntimeError('Connection already closed')

    def receive(self):
        if self.state == 'OPEN':
            print('Receiving')
        elif self.state == 'CLOSED':
            raise RuntimeError('Connection closed')

    def send(self, data):
        if self.state == 'OPEN':
            print('Sending')
	    self.bytes_sent += len(data)
        elif self.state == 'CLOSED':
            raise RuntimeError('Connection closed')
```

Part of the problem with a stateful object is that the control flow gets
messy--often populated with a lot of `if-else` statements.  A common
design pattern for dealing with this is to split the class into separate
classes--each representing a different state.  You then stitch everything
back together again.  Here is an example:

```python
class Connection:
    def __init__(self):
        self.state = ClosedConnection
        self.bytes_sent = 0

    def open(self):
        return self.state.open(self)

    def close(self):
        return self.state.close(self)

    def receive(self):
        return self.state.receive(self)

    def send(self, data):
        return self.state.send(self, data)

class OpenConnection:
    @staticmethod
    def open(conn):
        raise RuntimeError('Connection already open')

    @staticmethod
    def close(conn):
        self.state = ClosedConnection
        
    @staticmethod
    def receive(conn):
        print('Receiving')

    @staticmethod        
    def send(conn, data):
        conn.bytes_sent += len(data)
        print('Sending')

class ClosedConnection:
    @staticmethod    
    def open(conn):
        self.state = OpenConnection

    @staticmethod        
    def close(conn):
        raise RuntimeError('Connection already closed')

    @staticmethod    
    def receive(conn):
        raise RuntimeError('Connection closed')

    @staticmethod    
    def send(conn, data):
        raise RuntimeError('Connection closed')
```

The general idea is that the top level `Connection` class keeps an internal
variable `state` that points to a class implementing the various methods.
The methods on `Connection` then delegate their operation to the `state` class.
By changing the `state` variable to point to different classes, you can
effectively swap in new methods when the object changes.  It's a neat trick.

## A Different Strategy

Anyways, I'm staring at this code the other day and a stray thought occurs to me--"why does
the code have to be refactored like that?"  Instead of decomposing the code
based on the state, maybe you could decompose it by method instead.  Like this:

```python
class Connection:
    def __init__(self):
        self.state = 'CLOSED'
        self.bytes_sent = 0
        
    def open(self):
        return getattr(Open, self.state)(self)

    def close(self):
        return getattr(Close, self.state)(self)        

    def receive(self):
        return getattr(Receive, self.state)(self)

    def send(self, data):
        return getattr(Send, self.state)(self, data)

class Open:
    @staticmethod
    def OPEN(conn):
        raise RuntimeError('Connection already open')

    @staticmethod        
    def CLOSED(conn):
        conn.state = 'OPEN'

class Close:
    @staticmethod    
    def OPEN(conn):
        conn.state = 'CLOSED'

    @staticmethod        
    def CLOSED(conn):
        raise RuntimeError('Connection already closed')

class Receive:
    @staticmethod    
    def OPEN(conn):
        print('Receiving')

    @staticmethod        
    def CLOSED(conn):
        raise RuntimeError('Connection closed')

class Send:
    @staticmethod    
    def OPEN(conn, data):
        conn.bytes_sent += len(data)
        print('Sending')

    @staticmethod        
    def CLOSED(conn, data):
        raise RuntimeError('Connection closed')
```

Yeah. Okay!   I somehow suspect that your coworkers will now be too angry to
even notice the clever use of `getattr()`.

## Discussion

No comments.  Want to make a comment?  Edit this page. Then submit a pull request. 



