# python 学习笔记（二）

## 迭代器与生成器

#### 遍历迭代器

1. 使用for循环

2. 使用 `next()` 函数并在代码中捕获 `StopIteration` 异常。

   ```python
   def manual_iter():
       with open('/etc/passwd') as f:
           try:
               while True:
                   line = next(f)
                   print(line, end='')
           except StopIteration:
               pass
   ```

#### 代理迭代

构建了一个自定义容器对象，里面包含有列表、元组或其他可迭代对象。 你想直接在你的这个新容器对象上执行迭代操作。

实际上你只需要定义一个 `__iter__()` 方法，将迭代操作代理到容器内部的对象上去。比如：

```python
class Node:
    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        return iter(self._children)

# Example
if __name__ == '__main__':
    root = Node(0)
    child1 = Node(1)
    child2 = Node(2)
    root.add_child(child1)
    root.add_child(child2)
    # Outputs Node(1), Node(2)
    for ch in root:
        print(ch)
```

root本身是不可迭代的，但是python的迭代器协议需要`__iter__()` 方法返回一个实现了 `__next__()` 方法的迭代器对象。这里的 `iter()` 函数的使用简化了代码， `iter(s)` 只是简单的通过调用 `s.__iter__()`方法来返回对应的迭代器对象， 就跟 `len(s)` 会调用 `s.__len__()` 原理是一样的。

#### 使用生成器创建新的迭代模式

假如想要实现一种新的迭代模式，使用一个生成器函数来定义例如

```python
def floatrange(start,stop,increment):
    x = start
    while x<stop:
        yield x
        x+=increment
list(floatrange(0,5,0.5))
[0, 0.5, 1.0, 1.5, 2.0, 2.5, 3.0, 3.5, 4.0, 4.5]
```

```python
def tryhha(n):
    while n>0:
        yield n
        n-=1
c =tryhha(2)
next(c)
...
next(c)
StopIteration                             Traceback (most recent call last)
<ipython-input-38-73b012f9653f> in <module>()
----> 1 next(c)

StopIteration: 
```

#### 使用生成器做深度优先搜索

```python
class Node:
    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        return iter(self._children)

    def depth_first(self):
        yield self
        for c in self:
            yield from c.depth_first()

# Example
if __name__ == '__main__':
    root = Node(0)
    child1 = Node(1)
    child2 = Node(2)
    root.add_child(child1)
    root.add_child(child2)
    child1.add_child(Node(3))
    child1.add_child(Node(4))
    child2.add_child(Node(5))

    for ch in root.depth_first():
        print(ch)
    # Outputs Node(0), Node(1), Node(3), Node(4), Node(2), Node(5)
```

#### 反向迭代

1. 使用内置的 `reversed()` 函数，反向迭代仅仅当对象的大小可预先确定或者对象实现了 `__reversed__()` 的特殊方法时才能生效。 如果两者都不符合，那你必须先将对象转换为一个列表才行

   ```python
   >>> a = [1, 2, 3, 4]
   >>> for x in reversed(a):
   ...     print(x)
   ```

2. 在自定义类上实现 `__reversed__()` 方法来实现反向迭代。

   ```python
   class Countdown:
       def __init__(self, start):
           self.start = start

       # Forward iterator
       def __iter__(self):
           n = self.start
           while n > 0:
               yield n
               n -= 1

       # Reverse iterator
       def __reversed__(self):
           n = 1
           while n <= self.start:
               yield n
               n += 1

   for rr in reversed(Countdown(30)):
       print(rr)
   for rr in Countdown(30):
       print(rr)
   ```

#### 排列组合的迭代

itertools模块提供了三个函数来解决这类问题。 其中一个是 `itertools.permutations()`， 它接受一个集合并产生一个元组序列，每个元组由集合中所有元素的一个可能排列组成。 也就是说通过打乱集合中元素排列顺序生成一个元组，比如：

```
>>> items = ['a', 'b', 'c']
>>> from itertools import permutations
>>> for p in permutations(items):
...     print(p)
...
('a', 'b', 'c')
('a', 'c', 'b')
('b', 'a', 'c')
('b', 'c', 'a')
('c', 'a', 'b')
('c', 'b', 'a')
>>>
#如果你想得到指定长度的所有排列，你可以传递一个可选的长度参数。
for p in permutations(items, 2):
```

使用 `itertools.combinations()` 可得到输入集合中元素的所有的组合。比如：

```
>>> from itertools import combinations
>>> for c in combinations(items, 3):
...     print(c)
...
('a', 'b', 'c')

>>> for c in combinations(items, 2):
...     print(c)
...
('a', 'b')
('a', 'c')
('b', 'c')

>>> for c in combinations(items, 1):
...     print(c)
...
('a',)
('b',)
('c',)
>>>
```

函数 `itertools.combinations_with_replacement()` 允许同一个元素被选择多次，比如：

```
>>> for c in combinations_with_replacement(items, 3):
...     print(c)
...
('a', 'a', 'a')
('a', 'a', 'b')
('a', 'a', 'c')
('a', 'b', 'b')
('a', 'b', 'c')
('a', 'c', 'c')
('b', 'b', 'b')
('b', 'b', 'c')
('b', 'c', 'c')
('c', 'c', 'c')
>>>
```

#### 序列带索引值迭代

```python
>>> my_list = ['a', 'b', 'c']
>>> for idx, val in enumerate(my_list):
...     print(idx, val)
...
0 a
1 b
2 c
```

#### 同时迭代两个序列

```python
>>> for i in zip_longest(a, b, fillvalue=0):
...     print(i)
```

#### 不同集合上元素的迭代

想在多个对象执行相同的操作，但是这些对象在不同的容器中，你希望代码在不失可读性的情况下避免写重复的循环。

`itertools.chain()` 方法可以用来简化这个任务。 它接受一个可迭代对象列表作为输入，并返回一个迭代器，有效的屏蔽掉在多个容器中迭代细节。 为了演示清楚，考虑下面这个例子：

```python
>>> from itertools import chain
>>> a = [1, 2, 3, 4]
>>> b = ['x', 'y', 'z']
>>> for x in chain(a, b):
... print(x)
...
1
2
3
4
x
y
z
```

#### 展开嵌套的序列

可以写一个包含 `yield from` 语句的递归生成器来轻松解决这个问题。比如：

```python
from collections import Iterable

def flatten(items, ignore_types=(str, bytes)):
    for x in items:
        if isinstance(x, Iterable) and not isinstance(x, ignore_types):
            yield from flatten(x)
        else:
            yield x

items = [1, 2, [3, 4, [5, 6], 7], 8]
# Produces 1 2 3 4 5 6 7 8
for x in flatten(items):
    print(x)
```

在上面代码中， `isinstance(x, Iterable)` 检查某个元素是否是可迭代的。 如果是的话， `yield from` 就会返回所有子例程的值。最终返回结果就是一个没有嵌套的简单序列了。

额外的参数 `ignore_types` 和检测语句 `isinstance(x, ignore_types)` 用来将字符串和字节排除在可迭代对象外，防止将它们再展开成单个的字符。 这样的话字符串数组就能最终返回我们所期望的结果了。

#### 迭代器代替while无限循环

你在代码中使用 `while` 循环来迭代处理数据，因为它需要调用某个函数或者和一般迭代模式不同的测试条件。 能不能用迭代器来重写这个循环呢？

一个常见的IO操作程序可能会想下面这样：

```
CHUNKSIZE = 8192

def reader(s):
    while True:
        data = s.recv(CHUNKSIZE)
        if data == b'':
            break
        process_data(data)

```

这种代码通常可以使用 `iter()` 来代替，如下所示：

```
def reader2(s):
    for chunk in iter(lambda: s.recv(CHUNKSIZE), b''):
        pass
        # process_data(data)
```

如果你怀疑它到底能不能正常工作，可以试验下一个简单的例子。比如：

```python
>>> import sys
>>> f = open('/etc/passwd')
>>> for chunk in iter(lambda: f.read(10), ''):
...     n = sys.stdout.write(chunk)
```

`iter` 函数一个鲜为人知的特性是它接受一个可选的 `callable` 对象和一个标记(结尾)值作为输入参数。 当以这种方式使用的时候，它会创建一个迭代器， 这个迭代器会不断调用 `callable` 对象直到返回值和标记值相等为止。

这种特殊的方法对于一些特定的会被重复调用的函数很有效果，比如涉及到I/O调用的函数。 举例来讲，如果你想从套接字或文件中以数据块的方式读取数据，通常你得要不断重复的执行 `read()` 或 `recv()` ， 并在后面紧跟一个文件结尾测试来决定是否终止。这节中的方案使用一个简单的 `iter()` 调用就可以将两者结合起来了。 其中 `lambda` 函数参数是为了创建一个无参的 `callable` 对象，并为 `recv` 或 `read()` 方法提供了 `size` 参数。

## 文件与IO

1. with语句给被使用到的文件创建了一个上下文环境， 但 `with` 控制块结束时，文件会自动关闭。你也可以不使用 `with` 语句，但是这时候你就必须记得手动关闭文件

2. 想将 `print()` 函数的输出重定向到一个文件中去

   ```python
   with open('d:/work/test.txt', 'wt') as f:
       print('Hello World!', file=f)
   ```

3. ```python
   print('ACME', 50, 91.5, sep=',')   # join需要是字符串，sep不需要更方便
   ```

4. 定义当文件不存在时才可以写入，不能替换可以在 `open()` 函数中使用 `x` 模式来代替 `w` 模式的方法来解决这个问题

5. 读取写入字节数据时 使用b，注意解码和编码的问题

6. 读写压缩文件 gzip` 和 `bz2` 模块可以很容易的处理这些文件。 两个模块都为 `open()` 函数提供了另外的实现来解决这个问题。

7. 文件路径名的操作

   ```python
   >>> import os
   >>> path = '/Users/beazley/Data/data.csv'

   >>> # Get the last component of the path
   >>> os.path.basename(path)
   'data.csv'

   >>> # Get the directory name
   >>> os.path.dirname(path)
   '/Users/beazley/Data'

   >>> # Join path components together
   >>> os.path.join('tmp', 'data', os.path.basename(path))
   'tmp/data/data.csv'

   >>> # Expand the user's home directory
   >>> path = '~/Data/data.csv'
   >>> os.path.expanduser(path)
   '/Users/beazley/Data/data.csv'

   >>> # Split the file extension
   >>> os.path.splitext(path)
   ('~/Data/data', '.csv')
   >>>
   ```

8. 测试文件是否存在 os.path.exists('/etc/passwd')

   ```python
   >>> # Is a regular file
   >>> os.path.isfile('/etc/passwd')
   True

   >>> # Is a directory
   >>> os.path.isdir('/etc/passwd')
   False

   >>> # Is a symbolic link
   >>> os.path.islink('/usr/local/bin/python3')
   True

   >>> # Get the file linked to
   >>> os.path.realpath('/usr/local/bin/python3')
   '/usr/local/bin/python3.3'
   >>>
   >>> os.path.getsize('/etc/passwd')
   3669
   >>> os.path.getmtime('/etc/passwd')
   1272478234.0
   ```

   ​

## 函数

1. 可接受任意数量参数的函数

   为了能让一个函数接受任意数量的位置参数，可以使用一个*参数。例如：

   ```python
   def avg(first, *rest):
       return (first + sum(rest)) / (1 + len(rest))

   # Sample use
   avg(1, 2) # 1.5
   avg(1, 2, 3, 4) # 2.5

   #为了接受任意数量的关键字参数，使用一个以**开头的参数。attrs是一个包含所有被传入进来的关键字参数的字典。

   def make_element(name, value, **attrs):
       keyvals = [' %s="%s"' % item for item in attrs.items()]
       attr_str = ''.join(keyvals)
       element = '<{name}{attrs}>{value}</{name}>'.format(
                   name=name,
                   attrs=attr_str,
                   value=html.escape(value))
       return element

   # Example
   # Creates '<item size="large" quantity="6">Albatross</item>'
   make_element('item', 'Albatross', size='large', quantity=6)
   ```

2. 只接受关键字参数的函数

   希望函数的某些参数强制使用关键字参数传递

   ```python
   def mininum(*values, clip=None):
       m = min(values)
       if clip is not None:
           m = clip if clip > m else m
       return m

   minimum(1, 5, 2, -5, 10) # Returns -5
   minimum(1, 5, 2, -5, 10, clip=0) # Returns 0
   ```

3. 函数注解

   ```python
   #函数注解只存储在函数的 __annotations__ 属性中
   def add(x:int, y:int) -> int:
       return x + y
       
   >>> help(add)
   Help on function add in module __main__:
   add(x: int, y: int) -> int
   >>>
   ```

4.  返回多个值，return一个元祖即可

5. 定义匿名函数或内联函数

   当一些函数很简单，仅仅只是计算一个表达式的时候，就可以使用lambda表达式了

   ```python
   >>> add = lambda x, y: x + y
   >>> add(2,3)
   5
   >>> add('hello', 'world')
   'helloworld'

   #lambada 典型的使用场景是排序或数据reduce等

   >>> x = 10
   >>> a = lambda y: x + y
   >>> x = 20
   >>> b = lambda y: x + y
   >>>

   >>> a(10)
   30
   >>> b(10)
   30
   >>>
   #这其中的奥妙在于lambda表达式中的x是一个自由变量， 在运行时绑定值，而不是定义时就绑定，这跟函数的默认值参数定义是不同的。 因此，在调用这个lambda表达式的时候，x的值是执行时的值。
   #如果你想让某个匿名函数在定义时就捕获到值，可以将那个参数值定义成默认参数即可，就像下面这样：
   >>> x = 10
   >>> a = lambda y, x=x: x + y
   >>> x = 20
   >>> b = lambda y, x=x: x + y
   >>> a(10)
   20
   >>> b(10)
   30
   ```

6. 减少可调用对象的参数个数

   如果需要减少某个函数的参数个数，你可以使用 `functools.partial()` 。 `partial()`函数允许你给一个或多个参数设置固定的值，减少接下来被调用时的参数个数。

   ```python
   def spam(a, b, c, d):
       print(a, b, c, d)
   >> from functools import partial
   >>> s1 = partial(spam, 1) # a = 1
   >>> s1(2, 3, 4)
   1 2 3 4
   >>> s1(4, 5, 6)
   1 4 5 6
   #根据点和基点之间的距离来排序所有的这些点
   >>> pt = (4, 3)
   >>> points.sort(key=partial(distance,pt))
   >>> points
   [(3, 4), (1, 2), (5, 6), (7, 8)]
   >>>
   ```

7. 任何时候只要你碰到需要给某个函数增加额外的状态信息的问题，都可以考虑使用闭包。 相比将你的函数转换成一个类而言，闭包通常是一种更加简洁和优雅的方案。

8. 带额外状态信息的回调函数

9. 闭包

   通常来讲，闭包的内部变量对于外界来讲是完全隐藏的。 但是，你可以通过编写访问函数并将其作为函数属性绑定到闭包上来实现这个目的。例如：

   ```python
   def sample():
       n = 0
       # Closure function
       def func():
           print('n=', n)

       # Accessor methods for n
       def get_n():
           return n

       def set_n(value):
           nonlocal n
           n = value

       # Attach as function attributes
       func.get_n = get_n
       func.set_n = set_n
       return func
   ```

10. ​

