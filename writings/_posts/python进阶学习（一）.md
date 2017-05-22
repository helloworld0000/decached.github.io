# PYTHON 学习笔记

python用了差不多一年了，虽然根本的东西都了解但是感觉太粗糙了，很多细节都不是很清楚，于是想要再次重新认识一下python，希望能够学的更扎实一点。（python 3）

http://python3-cookbook.readthedocs.io  大部分都是这里学到的，所以这里才是正版。

## 数据结构和算法

#### 解压序列赋值给多个变量

```python
#以前我都是自己直接赋值的，例如 name=a[0]等等，突然发现有更好用的
data = [ 'ACME', 50, 91.1, (2012, 12, 21) ]
name, shares, price, date = data
#这样就可以赋值成功了，前提是需要个数匹配，如果个数不匹配，会产生异常

s = 'hello world'
a,b,c,d,e,f,g,h,i,j,k = s

#如果只想要其中的一部分，我们还可以这样做,使用任意变量名去占位
data = [ 'ACME', 50, 91.1, (2012, 12, 21) ]
_, shares, price, _ = data
```

实际上，这种赋值可以用在任何的可迭代对象上面，不仅仅是列表或者元组。包括字符串，文件对象，迭代器，和生成器等等。

#### 解压可迭代对象赋值给多个变量

python的星号表达式，如果我们知道前两个一定是姓名，性别，但是后面可能跟着多个不确定数量的电话号码，那么可以像下面这样：

```python
record = ('Dave', 'male', '773-555-1212', '847-555-1212')
name,sex,*phone_numbers = record
>>> name
'Dave'
>>> phone_numbers
['773-555-1212', '847-555-1212']
#对于*后面的变量都是一个列表类型，不论几个
```

扩展的迭代解压语法是专门为解压不确定个数或任意个数元素的可迭代对象而设计的。 通常，这些可迭代对象的元素结构有确定的规则（比如第1个元素后面都是电话号码）， 星号表达式让开发人员可以很容易的利用这些规则来解压出元素来。 而不是通过一些比较复杂的手段去获取这些关联的的元素值。

值得注意的是，星号表达式在迭代元素为可变长元组的序列时是很有用的。 比如，下面是一个带有标签的元组序列：

```python
records = [
    ('foo', 1, 2),
    ('bar', 'hello'),
    ('foo', 3, 4),
]
def do_foo(x, y):
    print('foo', x, y)
def do_bar(s):
    print('bar', s)
for tag, *args in records:
    if tag == 'foo':
        do_foo(*args)
    elif tag == 'bar':
        do_bar(*args)
```

星号解压语法在字符串操作的时候也会很有用，比如字符串的分割。

```python
>>> line = 'nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false'
>>> uname, *fields, homedir, sh = line.split(':')
>>> uname
'nobody'
>>> homedir
'/var/empty'
```

#### 保留最后的N个元素

如何保留有限的历史记录？保留有限历史记录正是 `collections.deque` 大显身手的时候。比如，下面的代码在多行上面做简单的文本匹配， 并返回匹配所在行的最后N行：

```python
#使用 deque(maxlen=N) 构造函数会新建一个固定大小的队列。当新的元素加入并且这个队列已满的时候， 最老的元素会自动被移除掉。

from collections import deque

def search(lines, pattern, history=5):
    previous_lines = deque(maxlen=history)
    for li in lines:
        if pattern in li:
            yield li, previous_lines
        previous_lines.append(li)
        
# Example use on a file
if __name__ == '__main__':
    with open(r'../../cookbook/somefile.txt') as f:
        for line, prevlines in search(f, 'python', 5):
            for pline in prevlines:
                print(pline, end='')
            print(line, end='')
            print('-' * 20)
```

 `deque` 类可以被用在任何你只需要一个简单队列数据结构的场合。 如果你不设置最大队列大小，那么就会得到一个无限大小队列，你可以在队列的两端执行添加和弹出元素的操作。

在队列两端插入或删除元素时间复杂度都是 `O(1)` ，而在列表的开头插入或删除元素的时间复杂度为 `O(N)` 。

#### 查找最大或者最小的N个元素

heapq模块有两个函数：`nlargest()` 和 `nsmallest()` 可以完美解决这个问题。【堆数据结构】

```python
import heapq
nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
print(heapq.nlargest(3, nums)) # Prints [42, 37, 23]
print(heapq.nsmallest(3, nums)) # Prints [-4, 1, 2]
```

```python
portfolio = [
    {'name': 'IBM', 'shares': 100, 'price': 91.1},
    {'name': 'AAPL', 'shares': 50, 'price': 543.22},
    {'name': 'FB', 'shares': 200, 'price': 21.09},
    {'name': 'HPQ', 'shares': 35, 'price': 31.75},
    {'name': 'YHOO', 'shares': 45, 'price': 16.35},
    {'name': 'ACME', 'shares': 75, 'price': 115.65}
]
#上面代码在对每个元素进行对比的时候，会以 price 的值进行比较。
cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])
```

#### 实现一个优先队列

在队列上面，每次pop操作总是会返回优先级最高的那个元素

```python
import heapq

class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0

    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, self._index, item))
        self._index += 1

    def pop(self):
        return heapq.heappop(self._queue)[-1]
```

```python
q.push('foo', 1)
q.push('bar', 5)
q.push('spam', 4)
q.push('grok', 1)

q.pop()
‘bar’  #若优先级相同，则按照插入的顺序返回
```

#### 字典中的键映射多个值

你可以很方便的使用 `collections` 模块中的 `defaultdict` 来构造这样的字典。`defaultdict` 的一个特征是它会自动初始化每个 `key` 刚开始对应的值，所以你只需要关注添加元素操作了。比如：

```python
from collections import defaultdict

d = defaultdict(list)
d['a'].append(1)
d['a'].append(2)
d['b'].append(4)

d = defaultdict(set)
d['a'].add(1)
d['a'].add(2)
d['b'].add(4)
```

#### 字典排序

为了能控制一个字典中元素的顺序，你可以使用 `collections` 模块中的 `OrderedDict`类。 在迭代操作的时候它会保持元素被插入时的顺序。

```python
from collections import OrderedDict

d = OrderedDict()
d['foo'] = 1
d['bar'] = 2
d['spam'] = 3
d['grok'] = 4
# Outputs "foo 1", "bar 2", "spam 3", "grok 4"
for key in d:
    print(key, d[key])
```

当你想要构建一个将来需要序列化或编码成其他格式的映射的时候， `OrderedDict` 是非常有用的。 比如，你想精确控制以JSON编码后字段的顺序，你可以先使用 `OrderedDict` 来构建这样的数据。

`OrderedDict` 内部维护着一个根据键插入顺序排序的双向链表。每次当一个新的元素插入进来的时候， 它会被放到链表的尾部。对于一个已经存在的键的重复赋值不会改变键的顺序。

需要注意的是，一个 `OrderedDict` 的大小是一个普通字典的两倍，因为它内部维护着另外一个链表。 所以如果你要构建一个需要大量 `OrderedDict` 实例的数据结构的时候(比如读取100,000行CSV数据到一个 `OrderedDict` 列表中去)， 那么你就得仔细权衡一下是否使用 `OrderedDict` 带来的好处要大过额外内存消耗的影响。

#### 字典的运算

```python
prices = {
    'ACME': 45.23,
    'AAPL': 612.78,
    'IBM': 205.55,
    'HPQ': 37.20,
    'FB': 10.75
}
#为了对字典值执行计算操作，通常需要使用 zip() 函数先将键和值反转过来。 比如，下面是查找最小和最大股票价格和股票值的代码
min_price = min(zip(prices.values(), prices.keys()))
# min_price is (10.75, 'FB')
max_price = max(zip(prices.values(), prices.keys()))
# max_price is (612.78, 'AAPL')
prices_sorted = sorted(zip(prices.values(), prices.keys()))
# prices_sorted is [(10.75, 'FB'), (37.2, 'HPQ'),
#                   (45.23, 'ACME'), (205.55, 'IBM'),
#                   (612.78, 'AAPL')]
#执行这些计算的时候，需要注意的是 zip() 函数创建的是一个只能访问一次的迭代器。 比如，下面的代码就会产生错误：
prices_and_names = zip(prices.values(), prices.keys())
print(min(prices_and_names)) # OK
print(max(prices_and_names)) # ValueError: max() arg is an empty sequence
```

需要注意的是在计算操作中使用到了(值，键)对。当多个实体拥有相同的值的时候，键会决定返回结果。 比如，在执行 `min()` 和 `max()` 操作的时候，如果恰巧最小或最大值有重复的，那么拥有最小或最大键的实体会返回：

```python
>>> prices = { 'AAA' : 45.23, 'ZZZ': 45.23 }
>>> min(zip(prices.values(), prices.keys()))
(45.23, 'AAA')
>>> max(zip(prices.values(), prices.keys()))
(45.23, 'ZZZ')
>>>
```

#### 查找两个字典的相同点

一个字典就是一个键集合与值集合的映射关系。 字典的 `keys()` 方法返回一个展现键集合的键视图对象。 键视图的一个很少被了解的特性就是它们也支持集合操作，比如集合并、交、差运算。 所以，如果你想对集合的键执行一些普通的集合操作，可以直接使用键视图对象而不用先将它们转换成一个set。

字典的 `items()` 方法返回一个包含(键，值)对的元素视图对象。 这个对象同样也支持集合操作，并且可以被用来查找两个字典有哪些相同的键值对。

尽管字典的 `values()` 方法也是类似，但是它并不支持这里介绍的集合操作。 某种程度上是因为值视图不能保证所有的值互不相同，这样会导致某些集合操作会出现问题。 不过，如果你硬要在值上面执行这些集合操作的话，你可以先将值集合转换成set，然后再执行集合运算就行了。

```python
a = {
    'x' : 1,
    'y' : 2,
    'z' : 3
}

b = {
    'w' : 10,
    'x' : 11,
    'y' : 2
}

# Find keys in common
a.keys() & b.keys() # { 'x', 'y' }
# Find keys in a that are not in b
a.keys() - b.keys() # { 'z' }
# Find (key,value) pairs in common
a.items() & b.items() # { ('y', 2) }


# Make a new dictionary with certain keys removed
c = {key:a[key] for key in a.keys() - {'z', 'w'}}
# c is {'x': 1, 'y': 2}
```

#### 删除序列相同的元素并保持顺序

```python
def dedupe(items, key=None):
    seen = set()
    for item in items:
        val = item if key is None else key(item)
        if val not in seen:
            yield item
            seen.add(val)
```

#### 命名切片

你的程序已经出现一大堆已无法直视的硬编码切片下标，然后你想清理下代码。

假定你有一段代码要从一个记录字符串中几个固定位置提取出特定的数据字段(比如文件或类似格式)：

```python
###### 0123456789012345678901234567890123456789012345678901234567890'
record = '....................100 .......513.25 ..........'
cost = int(record[20:23]) * float(record[31:37])
#这样代码的可读性会很弱，而且可能就忘记了到底是什么
#可以改进为命名切片
SHARES = slice(20, 23)
PRICE = slice(31, 37)
cost = int(record[SHARES]) * float(record[PRICE])
```

内置的 `slice()` 函数创建了一个切片对象，可以被用在任何切片允许使用的地方。

如果你有一个切片对象a，你可以分别调用它的 `a.start` , `a.stop` , `a.step` 属性来获取更多的信息。比如：

```python
>>> a = slice(5, 50, 2)
>>> a.start
5
>>> a.stop
50
>>> a.step
2
>>>

#另外，你还能通过调用切片的 indices(size) 方法将它映射到一个确定大小的序列上， 这个方法返回一个三元组 (start, stop, step) ，所有值都会被合适的缩小以满足边界限制， 从而使用的时候避免出现 IndexError 异常。比如：
>>> s = 'HelloWorld'
>>> a.indices(len(s))
(5, 10, 2)
>>> for i in range(*a.indices(len(s))):
... print(s[i])
...
W
r
d
```

#### 序列中出现次数最多的元素

`collections.Counter` 类就是专门为这类问题而设计的， 它甚至有一个有用的 `most_common()` 方法直接给了你答案。

```python
words = [
    'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
    'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not', 'around', 'the',
    'eyes', "don't", 'look', 'around', 'the', 'eyes', 'look', 'into',
    'my', 'eyes', "you're", 'under'
]
from collections import Counter
word_counts = Counter(words)
# 出现频率最高的3个单词
top_three = word_counts.most_common(3)
print(top_three)
# Outputs [('eyes', 8), ('the', 5), ('look', 4)]
```

作为输入， `Counter` 对象可以接受任意的由可哈希(`hashable`)元素构成的序列对象。 在底层实现上，一个 `Counter` 对象就是一个字典，将元素映射到它出现的次数上。

`Counter` 实例一个鲜为人知的特性是它们可以很容易的跟数学运算操作相结合。比如：

```python
>>> a = Counter(words)
>>> b = Counter(morewords)
>>> a
Counter({'eyes': 8, 'the': 5, 'look': 4, 'into': 3, 'my': 3, 'around': 2,
"you're": 1, "don't": 1, 'under': 1, 'not': 1})
>>> b
Counter({'eyes': 1, 'looking': 1, 'are': 1, 'in': 1, 'not': 1, 'you': 1,
'my': 1, 'why': 1})
>>> # Combine counts
>>> c = a + b
>>> c
Counter({'eyes': 9, 'the': 5, 'look': 4, 'my': 4, 'into': 3, 'not': 2,
'around': 2, "you're": 1, "don't": 1, 'in': 1, 'why': 1,
'looking': 1, 'are': 1, 'under': 1, 'you': 1})
>>> # Subtract counts
>>> d = a - b
>>> d
Counter({'eyes': 7, 'the': 5, 'look': 4, 'into': 3, 'my': 2, 'around': 2,
"you're": 1, "don't": 1, 'under': 1})
>>>

```

毫无疑问， `Counter` 对象在几乎所有需要制表或者计数数据的场合是非常有用的工具。 在解决这类问题的时候你应该优先选择它，而不是手动的利用字典去实现。

#### 通过某个关键字排序一个字典列表

通过使用 `operator` 模块的 `itemgetter` 函数，可以非常容易的排序这样的数据结构。 假设你从数据库中检索出来网站会员信息列表，并且以下列的数据结构返回：

```python
rows = [
    {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003},
    {'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
]

from operator import itemgetter
rows_by_fname = sorted(rows, key=itemgetter('fname'))
rows_by_uid = sorted(rows, key=itemgetter('uid'))
print(rows_by_fname)
print(rows_by_uid)

#itemgetter() 函数也支持多个keys，比如下面的代码
rows_by_lfname = sorted(rows, key=itemgetter('lname','fname'))
print(rows_by_lfname)

#itemgetter() 有时候也可以用 lambda 表达式代替，但是，使用 itemgetter() 方式会运行的稍微快点。因此，如果你对性能要求比较高的话就使用 itemgetter() 方式。

rows_by_fname = sorted(rows, key=lambda r: r['fname'])
rows_by_lfname = sorted(rows, key=lambda r: (r['lname'],r['fname']))
```

在上面例子中， `rows` 被传递给接受一个关键字参数的 `sorted()` 内置函数。 这个参数是 `callable` 类型，并且从 `rows` 中接受一个单一元素，然后返回被用来排序的值。`itemgetter()` 函数就是负责创建这个 `callable` 对象的。

`operator.itemgetter()` 函数有一个被 `rows` 中的记录用来查找值的索引参数。可以是一个字典键名称， 一个整形值或者任何能够传入一个对象的 `__getitem__()` 方法的值。 如果你传入多个索引参数给 `itemgetter()` ，它生成的 `callable` 对象会返回一个包含所有元素值的元组， 并且 `sorted()` 函数会根据这个元组中元素顺序去排序。 但你想要同时在几个字段上面进行排序(比如通过姓和名来排序，也就是例子中的那样)的时候这种方法是很有用的。



####  排序不支持原生比较的对象处理

如果我们想要对类型相同的对象进行排序，但是他们不支持原生的比较操作，我们需要怎么办？

内置的 `sorted()` 函数有一个关键字参数 `key` ，可以传入一个 `callable` 对象给它， 这个 `callable` 对象对每个传入的对象返回一个值，这个值会被 `sorted` 用来排序这些对象。 比如，如果你在应用程序里面有一个 `User` 实例序列，并且你希望通过他们的 `user_id` 属性进行排序， 你可以提供一个以 `User` 实例作为输入并输出对应 `user_id`值的 `callable` 对象。比如：

callable 是指一个可调用的对象！例如函数等等 itemgetter() 函数就可以创建可调用对象。

```python
class User:
    def __init__(self, user_id):
        self.user_id = user_id

    def __repr__(self):
        return 'User({})'.format(self.user_id)


def sort_notcompare():
    users = [User(23), User(3), User(99)]
    print(users)
    print(sorted(users, key=lambda u: u.user_id))

#或者使用之前提到过的 operator.attrgetter() 来代替lambda
from operator import attrgetter
sorted(users, key=attrgetter('user_id'))
```

#### 如何通过某个字段将记录分组

`itertools.groupby()` 函数对于这样的数据分组操作非常实用。 为了演示，假设你已经有了下列的字典列表：

```python
rows = [
    {'address': '5412 N CLARK', 'date': '07/01/2012'},
    {'address': '5148 N CLARK', 'date': '07/04/2012'},
    {'address': '5800 E 58TH', 'date': '07/02/2012'},
    {'address': '2122 N CLARK', 'date': '07/03/2012'},
    {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'},
    {'address': '1060 W ADDISON', 'date': '07/02/2012'},
    {'address': '4801 N BROADWAY', 'date': '07/01/2012'},
    {'address': '1039 W GRANVILLE', 'date': '07/04/2012'},
]
from operator import itemgetter
from itertools import groupby

# Sort by the desired field first
rows.sort(key=itemgetter('date'))
# Iterate in groups
for date, items in groupby(rows, key=itemgetter('date')):
    print(date)
    for i in items:
        print(' ', i)
 
#结果
07/01/2012
  {'address': '5412 N CLARK', 'date': '07/01/2012'}
  {'address': '4801 N BROADWAY', 'date': '07/01/2012'}
07/02/2012
  {'address': '5800 E 58TH', 'date': '07/02/2012'}
  {'address': '5645 N RAVENSWOOD', 'date': '07/02/2012'}
  {'address': '1060 W ADDISON', 'date': '07/02/2012'}
07/03/2012
  {'address': '2122 N CLARK', 'date': '07/03/2012'}
07/04/2012
  {'address': '5148 N CLARK', 'date': '07/04/2012'}
  {'address': '1039 W GRANVILLE', 'date': '07/04/2012'}

#groupby() 函数扫描整个序列并且查找连续的相同值（或者根据指定的key函数返回值相同）的元素序列
#在每次迭代的时候，都会返回一个值和一个迭代器对象，这个迭代器对象可以生成元素值全部等于上边那个值的组中的所有的对象。


```

一个非常重要的准备步骤是要根据指定的字段将数据排序。 因为 `groupby()` 仅仅检查连续的元素，如果事先并没有排序完成的话，分组函数将得不到想要的结果。

如果你仅仅只是想根据 `date` 字段将数据分组到一个大的数据结构中去，并且允许随机访问， 那么你最好使用 `defaultdict()` 来构建一个多值字典。比如：

```python
from collections import defaultdict
rows_by_date = defaultdict(list)
for row in rows:
    rows_by_date[row['date']].append(row)
>>> for r in rows_by_date['07/01/2012']:
... print(r)
...
{'date': '07/01/2012', 'address': '5412 N CLARK'}
{'date': '07/01/2012', 'address': '4801 N BROADWAY'}
>>>
#在上面这个例子中，我们没有必要先将记录排序。因此，如果对内存占用不是很关心， 这种方式会比先排序然后再通过 groupby() 函数迭代的方式运行得快一些。

```

#### 过滤序列元素

假如有一个数据序列想利用一些规则从中提取出需要的值或者是缩短序列

最简单的过滤序列元素的方法就是使用列表推导。比如：

```python
pos = (n for n in mylist if n > 0) 

有时候，过滤规则比较复杂，不能简单的在列表推导或者生成器表达式中表达出来。 比如，假设过滤的时候需要处理一些异常或者其他复杂情况。这时候你可以将过滤代码放到一个函数中， 然后使用内建的 filter() 函数。示例如下：

values = ['1', '2', '-3', '-', '4', 'N/A', '5']
def is_int(val):
    try:
        x = int(val)
        return True
    except ValueError:
        return False
ivals = list(filter(is_int, values))
print(ivals)
# Outputs ['1', '2', '-3', '4', '5']
```

另外一个值得关注的过滤工具就是 `itertools.compress()` ， 它以一个 `iterable` 对象和一个相对应的 `Boolean` 选择器序列作为输入参数。 然后输出 `iterable` 对象中对应选择器为 `True` 的元素。 当你需要用另外一个相关联的序列来过滤某个序列的时候，这个函数是非常有用的。 比如，假如现在你有下面两列数据：

```python
addresses = [
    '5412 N CLARK',
    '5148 N CLARK',
    '5800 E 58TH',
    '2122 N CLARK',
    '5645 N RAVENSWOOD',
    '1060 W ADDISON',
    '4801 N BROADWAY',
    '1039 W GRANVILLE',
]
counts = [ 0, 3, 10, 4, 1, 7, 6, 1]
现在你想将那些对应 count 值大于5的地址全部输出，那么你可以这样做：
>>> from itertools import compress
>>> more5 = [n > 5 for n in counts]
>>> more5
[False, False, True, False, False, True, True, False]
>>> list(compress(addresses, more5))
['5800 E 58TH', '1060 W ADDISON', '4801 N BROADWAY']
>>>

```

这里的关键点在于先创建一个 `Boolean` 序列，指示哪些元素符合条件。 然后 `compress()` 函数根据这个序列去选择输出对应位置为 `True` 的元素。

和 `filter()` 函数类似， `compress()` 也是返回的一个迭代器。因此，如果你需要得到一个列表， 那么你需要使用 `list()` 来将结果转换为列表类型。

#### 从字典中提取子集

想构造一个字典，它是另外一个字典的子集。

最简单的方式是使用字典推导。比如：

```python
prices = {
    'ACME': 45.23,
    'AAPL': 612.78,
    'IBM': 205.55,
    'HPQ': 37.20,
    'FB': 10.75
}
# Make a dictionary of all prices over 200
p1 = {key: value for key, value in prices.items() if value > 200}
# Make a dictionary of tech stocks
tech_names = {'AAPL', 'IBM', 'HPQ', 'MSFT'}
p2 = {key: value for key, value in prices.items() if key in tech_names}
```

#### 映射名称到序列元素

你有一段通过下标访问列表或者元组中元素的代码，但是这样有时候会使得你的代码难以阅读， 于是你想通过名称来访问元素。

`collections.namedtuple()` 函数通过使用一个普通的元组对象来帮你解决这个问题。 这个函数实际上是一个返回Python中标准元组类型子类的一个工厂方法。 你需要传递一个类型名和你需要的字段给它，然后它就会返回一个类，你可以初始化这个类，为你定义的字段传递值等。 代码示例：

```python
>>> from collections import namedtuple
>>> Subscriber = namedtuple('Subscriber', ['addr', 'joined'])
>>> sub = Subscriber('jonesy@example.com', '2012-10-19')
>>> sub
Subscriber(addr='jonesy@example.com', joined='2012-10-19')
>>> sub.addr
'jonesy@example.com'
>>> sub.joined
'2012-10-19'
>>>
```

命名元组另一个用途就是作为字典的替代，因为字典存储需要更多的内存空间。 如果你需要构建一个非常大的包含字典的数据结构，那么使用命名元组会更加高效。 但是需要注意的是，不像字典那样，一个命名元组是不可更改的。比如：

```python
>>> s = Stock('ACME', 100, 123.45)
>>> s
Stock(name='ACME', shares=100, price=123.45)
>>> s.shares = 75
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
>>>
```

#### 转换并同时计算数据

你需要在数据序列上执行聚集函数(比如 `sum()` , `min()` , `max()` )， 但是首先你需要先转换或者过滤数据

一个非常优雅的方式去结合数据计算与转换就是使用一个生成器表达式参数。 比如，如果你想计算平方和，可以像下面这样做：

```python
nums = [1, 2, 3, 4, 5]
s = sum(x * x for x in nums)
```

上面的示例向你演示了当生成器表达式作为一个单独参数传递给函数时候的巧妙语法(你并不需要多加一个括号)。 比如，下面这些语句是等效的：

```python
s = sum((x * x for x in nums)) # 显示的传递一个生成器表达式对象
s = sum(x * x for x in nums) # 更加优雅的实现方式，省略了括号
```

#### 合并多个字典或映射

假如你有如下两个字典:

```python
a = {'x': 1, 'z': 3 }
b = {'y': 2, 'z': 4 }
```

现在假设你必须在两个字典中执行查找操作(比如先从 `a` 中找，如果找不到再在 `b` 中找)。 一个非常简单的解决方案就是使用 `collections` 模块中的 `ChainMap` 类。比如：

```python
from collections import ChainMap
c = ChainMap(a,b)
print(c['x']) # Outputs 1 (from a)
print(c['y']) # Outputs 2 (from b)
print(c['z']) # Outputs 3 (from a)
```

一个 `ChainMap` 接受多个字典并将它们在逻辑上变为一个字典。 然后，这些字典并不是真的合并在一起了， `ChainMap` 类只是在内部创建了一个容纳这些字典的列表 并重新定义了一些常见的字典操作来遍历这个列表。大部分字典操作都是可以正常使用的，比如：

```
>>> len(c)
3
>>> list(c.keys())
['x', 'y', 'z']
>>> list(c.values())
[1, 2, 3]
>>>

```

如果出现重复键，那么第一次出现的映射值会被返回。 因此，例子程序中的 `c['z']`总是会返回字典 `a` 中对应的值，而不是 `b` 中对应的值。

对于字典的更新或删除操作总是影响的是列表中第一个字典。比如：

```
>>> c['z'] = 10
>>> c['w'] = 40
>>> del c['x']
>>> a
{'w': 40, 'z': 10}
>>> del c['y']
Traceback (most recent call last):
...
KeyError: "Key not found in the first mapping: 'y'"
>>>

```

`ChainMap` 对于编程语言中的作用范围变量(比如 `globals` , `locals` 等)是非常有用的。 事实上，有一些方法可以使它变得简单：

```
>>> values = ChainMap()
>>> values['x'] = 1
>>> # Add a new mapping
>>> values = values.new_child()
>>> values['x'] = 2
>>> # Add a new mapping
>>> values = values.new_child()
>>> values['x'] = 3
>>> values
ChainMap({'x': 3}, {'x': 2}, {'x': 1})
>>> values['x']
3
>>> # Discard last mapping
>>> values = values.parents
>>> values['x']
2
>>> # Discard last mapping
>>> values = values.parents
>>> values['x']
1
>>> values
ChainMap({'x': 1})
>>>

```

作为 `ChainMap` 的替代，你可能会考虑使用 `update()` 方法将两个字典合并。比如：

```
>>> a = {'x': 1, 'z': 3 }
>>> b = {'y': 2, 'z': 4 }
>>> merged = dict(b)
>>> merged.update(a)
>>> merged['x']
1
>>> merged['y']
2
>>> merged['z']
3
>>>

```

这样也能行得通，但是它需要你创建一个完全不同的字典对象(或者是破坏现有字典结构)。 同时，如果原字典做了更新，这种改变不会反应到新的合并字典中去。比如：

```
>>> a['x'] = 13
>>> merged['x']
1

```

`ChainMap` 使用原来的字典，它自己不创建新的字典。所以它并不会产生上面所说的结果，比如：

```
>>> a = {'x': 1, 'z': 3 }
>>> b = {'y': 2, 'z': 4 }
>>> merged = ChainMap(a, b)
>>> merged['x']
1
>>> a['x'] = 42
>>> merged['x'] # Notice change to merged dicts
42
>>>
```

对于字典的更新或删除操作总是影响的是列表中第一个字典。



### 字符串和文本

#### 使用多个界定符分割字符串

将一个字符串分割为多个字段，但是分隔符(还有周围的空格)并不是固定的。

`string` 对象的 `split()` 方法只适应于非常简单的字符串分割情形， 它并不允许有多个分隔符或者是分隔符周围不确定的空格。 当你需要更加灵活的切割字符串的时候，最好使用 `re.split()` 方法：

```python
>>> line = 'asdf fjdk; afed, fjek,asdf, foo'
>>> import re
>>> re.split(r'[;,\s]\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
```

需要特别注意的是正则表达式中是否包含一个括号捕获分组。 如果使用了捕获分组，那么被匹配的文本也将出现在结果列表中。比如，观察一下这段代码运行后的结果：

```python
>>> fields = re.split(r'(;|,|\s)\s*', line)
>>> fields
['asdf', ' ', 'fjdk', ';', 'afed', ',', 'fjek', ',', 'asdf', ',', 'foo']
```

如果你不想保留分割字符串到结果列表中去，但仍然需要使用到括号来分组正则表达式的话， 确保你的分组是非捕获分组，形如 `(?:...)` 。比如：

```
>>> re.split(r'(?:,|;|\s)\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
>>>

```

如果你不想保留分割字符串到结果列表中去，但仍然需要使用到括号来分组正则表达式的话， 确保你的分组是非捕获分组，形如 `(?:...)` 。比如：

```python
>>> re.split(r'(?:,|;|\s)\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
>>>
```

#### 字符串匹配

检查字符串开头或结尾的一个简单方法是使用 `str.startswith()` 或者是 `str.endswith()` 方法。

如果你想检查多种匹配可能，只需要将所有的匹配项放入到一个元组中去， 然后传给 `startswith()` 或者 `endswith()` 方法：

```python
>>> import os
>>> filenames = os.listdir('.')
>>> filenames
[ 'Makefile', 'foo.c', 'bar.py', 'spam.c', 'spam.h' ]
>>> [name for name in filenames if name.endswith(('.c', '.h')) ]
['foo.c', 'spam.c', 'spam.h'
>>> any(name.endswith('.py') for name in filenames)
True
>>>
```

想使用 **Unix Shell** 中常用的通配符(比如 `*.py` , `Dat[0-9]*.csv` 等)去匹配文本字符串

`fnmatch` 模块提供了两个函数—— `fnmatch()` 和 `fnmatchcase()` ，可以用来实现这样的匹配。用法如下：

```python
>>> from fnmatch import fnmatch, fnmatchcase
>>> fnmatch('foo.txt', '*.txt')
True
>>> fnmatch('foo.txt', '?oo.txt')
True
>>> fnmatch('Dat45.csv', 'Dat[0-9]*')
True
>>> names = ['Dat1.csv', 'Dat2.csv', 'config.ini', 'foo.py']
>>> [name for name in names if fnmatch(name, 'Dat*.csv')]
['Dat1.csv', 'Dat2.csv']
>>>
>>> fnmatchcase('foo.txt', '*.TXT')
False
```

re模块

在定义正则式的时候，通常会利用括号去捕获分组。比如：

```python
>>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
>>>
>>> m = datepat.match('11/27/2012')
>>> m
<_sre.SRE_Match object at 0x1005d2750>
>>> # Extract the contents of each group
>>> m.group(0)
'11/27/2012'
>>> m.group(1)
'11'
>>> m.group(2)
'27'
>>> m.group(3)
'2012'
>>> m.groups()
('11', '27', '2012')
>>> month, day, year = m.groups()
>>>
>>> # Find all matches (notice splitting into tuples)
>>> text
'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> datepat.findall(text)
[('11', '27', '2012'), ('3', '13', '2013')]
>>> for month, day, year in datepat.findall(text):
... print('{}-{}-{}'.format(year, month, day))
...
2012-11-27
2013-3-13
>>>
```

#### 字符串的搜索和替换

对于简单的字面模式，直接使用 `str.repalce()` 方法即可

对于复杂的模式，请使用 `re` 模块中的 `sub()` 函数。 为了说明这个，假设你想将形式为 `11/27/2012` 的日期字符串改成 `2012-11-27` 。示例如下：

```python
>>> text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> import re
>>> re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text)
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>>
#事先定义好这种模式，先编译好来提升性能 
#sub() 函数中的第一个参数是被匹配的模式，第二个参数是替换模式。反斜杠数字比如 \3 指向前面模式的捕获组号。

>>> import re
>>> datepat = re.compile(r'(\d+)/(\d+)/(\d+)')
>>> datepat.sub(r'\3-\1-\2', text)

#对于更加复杂的替换，可以传递一个替换回调函数来代替
#一个替换回调函数的参数是一个 match 对象，也就是 match() 或者 find() 返回的对象。 使用 group() 方法来提取特定的匹配部分。回调函数最后返回替换字符串。


>>> from calendar import month_abbr
>>> def change_date(m):
... mon_name = month_abbr[int(m.group(1))]
... return '{} {} {}'.format(m.group(2), mon_name, m.group(3))
...
>>> datepat.sub(change_date, text)
'Today is 27 Nov 2012. PyCon starts 13 Mar 2013.'

#如果除了替换后的结果外，你还想知道有多少替换发生了，可以使用 re.subn() 来代替。比如：
>>> newtext, n = datepat.subn(r'\3-\1-\2', text)
>>> newtext
'Today is 2012-11-27. PyCon starts 2013-3-13.'
>>> n
2
>>>
```

#### 审查并清理文本字符串

```python
>>> s = 'pýtĥöñ\fis\tawesome\r\n'

>>> remap = {
...     ord('\t') : ' ',
...     ord('\f') : ' ',
...     ord('\r') : None # Deleted
... }
>>> a = s.translate(remap)
>>> a
'pýtĥöñ is awesome\n'
>>>
#可以以这个表格为基础进一步构建更大的表格。比如，让我们删除所有的和音符：
>>> import unicodedata
>>> import sys
>>> cmb_chrs = dict.fromkeys(c for c in range(sys.maxunicode)
...                         if unicodedata.combining(chr(c)))
...
>>> b = unicodedata.normalize('NFD', a)
>>> b
'pýtĥöñ is awesome\n'
>>> b.translate(cmb_chrs)
'python is awesome\n'
>>>
#另一种清理文本的技术涉及到I/O解码与编码函数。这里的思路是先对文本做一些初步的清理， 然后再结合 encode() 或者 decode() 操作来清除或修改它。比如：
>>> a
'pýtĥöñ is awesome\n'
>>> b = unicodedata.normalize('NFD', a)
>>> b.encode('ascii', 'ignore').decode('ascii')
'python is awesome\n'
>>>
```

#### 字符串对齐

对于基本的字符串对齐操作，可以使用字符串的 `ljust()` , `rjust()` 和 `center()` 方法。比如：

```python
>>> text = 'Hello World'
>>> text.ljust(20)
'Hello World         '
>>> text.rjust(20)
'         Hello World'
>>> text.center(20)
'    Hello World     '
>>>
```

所有这些方法都能接受一个可选的填充字符。比如：

```python
>>> text.rjust(20,'=')
'=========Hello World'
>>> text.center(20,'*')
'****Hello World*****'
>>>
```

函数 `format()` 同样可以用来很容易的对齐字符串。 你要做的就是使用 `<,>` 或者 `^`字符后面紧跟一个指定的宽度。比如：

```python
>>> format(text, '>20')
'         Hello World'
>>> format(text, '<20')
'Hello World         '
>>> format(text, '^20')
'    Hello World     '
>>>
#如果你想指定一个非空格的填充字符，将它写到对齐字符的前面即可：不仅字符串，数字类型也同样适用
>>> format(text, '*^20s')
'****Hello World*****'
```

#### 合并或者拼接字符串

如果你想要合并的字符串是在一个序列或者 `iterable` 中，那么最快的方式就是使用 `join()` 方法。

```python
>>> parts = ['Is', 'Chicago', 'Not', 'Chicago?']
>>> ' '.join(parts)
'Is Chicago Not Chicago?'
>>> ','.join(parts)
'Is,Chicago,Not,Chicago?'
>>> ''.join(parts)
'IsChicagoNotChicago?'
>>>

>>> data = ['ACME', 50, 91.1]
>>> ','.join(str(d) for d in data)
'ACME,50,91.1'
>>>
```

同样还得注意不必要的字符串连接操作。有时候程序员在没有必要做连接操作的时候仍然多此一举。比如在打印的时候：

```python
print(a + ':' + b + ':' + c) # Ugly
print(':'.join([a, b, c])) # Still ugly
print(a, b, c, sep=':') # Better
```

#### 字符串中插入变量

你想创建一个内嵌变量的字符串，变量被它的值所表示的字符串替换掉。

Python并没有对在字符串中简单替换变量值提供直接的支持。 但是通过使用字符串的 `format()` 方法来解决这个问题。比如：

```python
>>> s = '{name} has {n} messages.'
>>> s.format(name='Guido', n=37)
'Guido has 37 messages.'
>>>
#如果要被替换的变量能在变量域中找到， 那么你可以结合使用 format_map() 和 vars()
>>> name = 'Guido'
>>> n = 37
>>> s.format_map(vars())
'Guido has 37 messages.'
>>>
#vars() 还有一个有意思的特性就是它也适用于对象实例。
>>> class Info:
...     def __init__(self, name, n):
...         self.name = name
...         self.n = n
...
>>> a = Info('Guido',37)
>>> s.format_map(vars(a))
'Guido has 37 messages.'
>>>
#注意不能处理变量缺失的情况！
#一种避免这种错误的方法是另外定义一个含有 __missing__() 方法的字典对象，就像下面这样：
class safesub(dict):
"""防止key找不到"""
def __missing__(self, key):
    return '{' + key + '}'
#可以利用这个类包装输入后传递给 format_map() ：
>>> s.format_map(safesub(vars()))
'Guido has {n} messages.'
```

#### 以指定列宽格式化字符串

```python
>>> print(textwrap.fill(s, 40, initial_indent='    '))
    Look into my eyes, look into my
eyes, the eyes, the eyes, the eyes, not
around the eyes, don't look around the
eyes, look into my eyes, you're under.

>>> print(textwrap.fill(s, 40, subsequent_indent='    '))
Look into my eyes, look into my eyes,
    the eyes, the eyes, the eyes, not
    around the eyes, don't look around
    the eyes, look into my eyes, you're
    under.
#如果要自动匹配终端大小的话
>>> import os
>>> os.get_terminal_size().columns
80
>>>
```

## 数字和日期

#### 随机选择

`random` 模块有大量的函数用来产生随机数和随机选择元素。 比如，要想从一个序列中随机的抽取一个元素，可以使用 `random.choice()` ：

```
>>> import random
>>> values = [1, 2, 3, 4, 5, 6]
>>> random.choice(values)
2
>>> random.choice(values)
3
>>> random.choice(values)
1
>>> random.choice(values)
4
>>> random.choice(values)
6
>>>
```

如果你仅仅只是想打乱序列中元素的顺序，可以使用 `random.shuffle()` ：

```
>>> random.shuffle(values)
>>> values
[2, 4, 6, 5, 3, 1]
>>> random.shuffle(values)
>>> values
[3, 5, 2, 1, 6, 4]
>>>
```

生成随机整数，请使用 `random.randint()` ：

```
>>> random.randint(0,10)
2
>>> random.randint(0,10)
5
>>> random.randint(0,10)
0
>>> random.randint(0,10)
7
>>> random.randint(0,10)
10
>>> random.randint(0,10)
3
>>>
```

为了生成0到1范围内均匀分布的浮点数，使用 `random.random()` ：

```
>>> random.random()
0.9406677561675867
>>> random.random()
0.133129581343897
>>> random.random()
0.4144991136919316
>>>
```

`random` 模块使用 *Mersenne Twister* 算法来计算生成随机数。这是一个确定性算法， 但是你可以通过 `random.seed()` 函数修改初始化种子。比如：

```
random.seed() # Seed based on system time or os.urandom()
random.seed(12345) # Seed based on integer given
random.seed(b'bytedata') # Seed based on byte data

```

除了上述介绍的功能，random模块还包含基于均匀分布、高斯分布和其他分布的随机数生成函数。 比如， `random.uniform()` 计算均匀分布随机数， `random.gauss()` 计算正态分布随机数。