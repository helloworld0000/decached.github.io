# Python Super 继承

```` python
>>> class A:
...     def __init__(self):
...             self.n = 2
...     def add(self,m):
...             print("self is {0} @A.add".format(self))
...             self.n += m
...
>>> class B(A):
...     def __init__(self):
...             self.n = 3
...     def add(self,m):
...             print("self is {0} @B.add".format(self))
...             super().add(m)
...             self.n += 3
...
>>>
>>> B.mro()
[<class '__main__.B'>, <class '__main__.A'>, <class 'object'>]
>>> A.mro()
[<class '__main__.A'>, <class 'object'>]
>>> class C(A):
...      def __init__(self):
...             self.n = 4
...      def add(self,m):
...              print("self is {0} @C.add".format(self))
...              super().add(m)
...              self.n += 4
...
>>> C.mro()
[<class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
>>> class D(B,C):
...       def __init__(self):
...             self.n = 5
...       def add(self,m):
...               print("self is {0} @D.add".format(self))
...               super().add(m)
...               self.n += 5
...
>>>
>>> D.mro()
[<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
>>> d = D()
>>> d.n = 5
>>> d.add(2)
self is <__main__.D object at 0x10e2fc5f8> @D.add
self is <__main__.D object at 0x10e2fc5f8> @B.add
self is <__main__.D object at 0x10e2fc5f8> @C.add
self is <__main__.D object at 0x10e2fc5f8> @A.add
>>> d.n
19
>>>
````

