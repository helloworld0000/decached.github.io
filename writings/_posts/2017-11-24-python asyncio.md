---
typora-copy-images-to: ../../images/writings
---

# **Python with asyncio**

### yeild

要了解python的协程，首先需要了了解``  yeild from``的用法，我再稍微提及一点，``yeild``的用法应该并不会陌生

首先Generator 的作用其实是实现了懒执行 (lazy evalution) ，即在真正需要某个值的时候才真正去计算这个值。因此，更进一步，Generator 其实是返回了控制流。当一个 generator 执行到 yeild 语句时，它便保存当前的状态，返回所给的结果（也可以没有），并将当前的执行流还给调用它的函数，而当再次调用它时，Generator 就从上次 yield 的位置继续执行。

```python
def generator():
    print('before')
    yield            # break 1
    print('middle')
    yield            # break 2
    print('after')

x = generator()
next(x)
#=> before
next(x)
#=> middle
next(x)
#=> after
#=> exception StopIteration
```

### yield from

考虑我们有多个 generator 并想把 generator 组合起来，如：

```python
def odds(n):
    for i in range(n):
        if i % 2 == 1:
            yield i

def evens(n):
    for i in range(n):
        if i % 2 == 0:
            yield i

def odd_even(n):
    for x in odds(n):
        yield x
    for x in evens(n):
        yield x

for x in odd_even(6):
    print(x)
#=> 1, 3, 5, 0, 2, 4
```

```python
def odd_even(n):
    yield from odds(n)
    yield from evens(n)
#这样会清晰很多 yield from 语法可以用来方便地组合不同的 generator。
```

### 多线程

CPU 的执行是顺序的，线程是操作系统提供的一种机制，允许我们在操作系统的层面上实现“并行”。而协程则可以认为是应用程序提供的一种机制（用户或库来完成），允许我们在应用程序的层面上实现“并行”。

### 协程

![0-s1GH0YO9ZNdEEDxo](../../images/writings/0-s1GH0YO9ZNdEEDxo.jpg)

1. 消息循环在线程中执行
2. 从队列中取得任务
3. 每个任务在协程中执行下一步动作
4. 如果在一个协程中调用另一个协程（await），会触发上下文切换，挂起当前协程，并保存现场环境（变量，状态），然后载入被调用协程。
5. 如果协程的执行到阻塞部分（阻塞IO，Sleep），当前协程就会被挂起，并将控制权返回到线程的消息循环中，然后消息循环继续从队列中执行下一个任务。





```python
import threading
import asyncio

@asyncio.coroutine
def hello():
    print('Hello world! (%s)' % threading.currentThread())
    yield from asyncio.sleep(1)
    print('Hello again! (%s)' % threading.currentThread())

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()

Hello world! (<_MainThread(MainThread, started 139931946989312)>)
Hello world! (<_MainThread(MainThread, started 139931946989312)>)
Hello again! (<_MainThread(MainThread, started 139931946989312)>)
Hello again! (<_MainThread(MainThread, started 139931946989312)>)
```



```python
import asyncio

@asyncio.coroutine
def wget(host):
    print('wget %s...' % host)
    connect = asyncio.open_connection(host, 80)
    reader, writer = yield from connect
    header = 'GET / HTTP/1.0\r\nHost: %s\r\n\r\n' % host
    writer.write(header.encode('utf-8'))
    yield from writer.drain()
    while True:
        line = yield from reader.readline()
        if line == b'\r\n':
            break
        print('%s header > %s' % (host, line.decode('utf-8').rstrip()))
    # Ignore the body, close the socket
    writer.close()

loop = asyncio.get_event_loop()
tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

```python
wget www.sina.com.cn...
wget www.sohu.com...
wget www.163.com...
www.sina.com.cn header > HTTP/1.1 200 OK
www.sina.com.cn header > Server: nginx
www.sina.com.cn header > Date: Thu, 23 Nov 2017 06:49:56 GMT
www.sina.com.cn header > Content-Type: text/html
www.sina.com.cn header > Content-Length: 602623
www.sina.com.cn header > Connection: close
www.sina.com.cn header > Last-Modified: Thu, 23 Nov 2017 06:48:10 GMT
www.sina.com.cn header > Vary: Accept-Encoding
www.sina.com.cn header > Expires: Thu, 23 Nov 2017 06:50:04 GMT
www.sina.com.cn header > Cache-Control: max-age=60
www.sina.com.cn header > X-Powered-By: shci_v1.03
www.sina.com.cn header > Age: 53
www.sina.com.cn header > Via: http/1.1 cnc.beixian.ha2ts4.205 (ApacheTrafficServer/6.2.1 [cHs f ]), http/1.1 cern.beijing.ha2ts4.209 (ApacheTrafficServer/6.2.1 [cHs f ])
www.sina.com.cn header > X-Via-Edge: 1511419796303c84376cacdd4cd3a3b4c7983
www.sina.com.cn header > X-Cache: HIT.209
www.sina.com.cn header > X-Via-CDN: f=edge,s=cern.beijing.ha2ts4.205.nb.sinaedge.com,c=202.118.67.200;f=Edge,s=cern.beijing.ha2ts4.209,c=58.205.212.205
www.163.com header > HTTP/1.0 302 Moved Temporarily
www.163.com header > Server: Cdn Cache Server V2.0
www.163.com header > Date: Thu, 23 Nov 2017 06:49:56 GMT
www.163.com header > Content-Length: 0
www.163.com header > Location: http://www.163.com/special/0077jt/error_isp.html
www.163.com header > Connection: close
www.sohu.com header > HTTP/1.1 200 OK
www.sohu.com header > Content-Type: text/html;charset=UTF-8
www.sohu.com header > Connection: close
www.sohu.com header > Server: nginx
www.sohu.com header > Date: Thu, 23 Nov 2017 06:49:54 GMT
www.sohu.com header > Cache-Control: max-age=60
www.sohu.com header > X-From-Sohu: X-SRC-Cached
www.sohu.com header > Content-Encoding: gzip
www.sohu.com header > FSS-Cache: HIT from 7965678.14519288.8719442
www.sohu.com header > FSS-Proxy: Powered by 2788255.4164521.3541940
```

