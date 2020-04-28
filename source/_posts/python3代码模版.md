---
title: python3代码模版
date: 2020-04-15 19:56:46
tags: [python]
---

各种语法糖，简化写代码
<!--more-->

# 基础

```python
import collections

# 新建类语法糖
Card = collections.namedtuple('Card', ['rank', 'suit'])
# filter in list
[str(n) for n in range(2, 11)]

# 随机选取
from random import choice
choice(list)

# 倒序访问
for a in reversed(list):
    print(a)

# tuple比较的时候比较的是数值
('a', 1) == ('a', 1) # True

# 排序方法
list0.sort() #在原列表排序


def comp(card):
    return card.val
list1 = sorted(list0, key=comp) #生成一个新的list，并且按照key方法返回的数值排序 还有一个reversed参数控制倒序

#输出字符串 -- old school
name = "Eric"
age = 74
"Hello, %s. You are %s." % (name, age)

# 输出字符串 -- f string
name = "Eric"
age = 74
f"Hello, {name}. You are {age}."
message = (
    f"Hi {name}. "
    f"You are a {profession}. "
    f"You were in {affiliation}."
) # 它会连起来

# python3.7之后 dict 类型的key值是ordered，意思是插入的顺序是固定的。
```

+ bin 用法

```python
# binary 用法
print(bin(5))
print(bin(5).count('1'))
print(bin(5).count('01'))

# 输出
0b101
2
1
```

## [collections](https://docs.python.org/3/library/collections.html?highlight=counter#collections.Counter)

+ heapq 用法

默认维护的是小顶堆。如果要搞大顶堆，加个负号就可以。
https://docs.python.org/3/library/heapq.html

```python
import heapq
heapq.heappush(heap, item)
heapq.heappop(heap)
heapq.heappushpop(heap, item)
heapq.heapify(x)
heapq.nlargest(n, iterable, key=None)
heapq.nsmallest(n, iterable, key=None)
```

+ collections.Counter 用法

counter 相当于对一个iteratalble 进行计数的dict。方括号跟dict一样使用的方法。语法糖？

```python
c = collections.Counter([1,1,1,2,3,3])
c = collections.Counter('aaabbbaaa')
sorted(c.elements()) # [a,a,a,a,a,b,b,b]
Counter('abracadabra').most_common(3) # [('a', 5), ('b', 2), ('r', 2)]

```

+ collections.deque 用法

deque是 队列 和 栈 的实现。

```python
d = deque([iterable[, maxlen])
d.append(x)
d.pop()

d.appendleft(x)
d.popleft()

d.clear()
d.copy()

d.extend()
d.extendleft()

d.insert(i, x)

d.reverse()
d.rotate(n=1) # n > 0, then d.appendleft(d.pop()); n < 0, then d.append(d.popleft())

```

# Sequences
+ ```list, tuple, and collections.deque``` can hold items of different types, including nested containers.
+ ```str, bytes, bytearray, memoryview, and array.array ```hold items of one simple type.
+ 不可变序列 ```tuple, str, and bytes```

## list
```python
# 构建list
symbols = '$¢£¥€¤'
codes = [ord(symbol) for symbol in symbols]
beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]

# 下面代码涉及了，filter(function, list) --> filter object
# lambda arg_list: expression
# map(function, list)
beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols))) # filter/map

# 两个for循环建立数组
colors = ['black', 'white']
sizes = ['S', 'M', 'L']
tshirts = [(color, size) for color in colors for size in sizes]
```

常用方法
```python
s.__add__(s2) #s + s2—concatenation
s.__iadd__(s2) #s += s2—in-place concatenation
s.append(e)
s.copy() #Shallow copy of the list
s.count(e) # Count occurrences of an element
s.extend(it) # Append items from iterable it
s.index(e) # Find position of first occurrence of e
s.insert(p, e) # Insert element e before the item at position p
s.pop([p]) # Remove and return item at position p (default: last)
s.remove(e) # Remove first occurrence of element e by value
s.reverse() # Reverse the order of the items in place
s.__reversed__() # 配合for使用的
s.sort([key], [reverse]) # Sort items in place with optional keyword arguments key and reverse
```

## tuple

+ tuple list可以被排序，顺序按照第一个值来
+ tuple list可以被for进行轮询

```python
for country, _ in traveler_ids:
    print(country)

lax_coordinates = (33.9425, -118.408056)
latitude, longitude = lax_coordinates  # unpacking
b, a = a, b # unpacking
*(1, 2) # unpacking *号

a, b, *rest = range(5)
(0, 1, [2, 3, 4])
a, *body, c, d = range(5)
(0, [1, 2], 3, 4)

hash(someTuple) #不会抛出异常，但是如果里面有list就会抛出异常
```

## slice
stride seq[start:stop:step]

```python
>>> s = 'bicycle'
>>> s[::3]
'bye'
>>> s[::-1]
'elcycib'
>>> s[::-2]
'eccb'
```

一波对slice的修改方法

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20, 30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> del l[5:7] # del 操作
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[3::2] = [11, 22]
>>> l
[0, 1, 20, 11, 5, 22, 9]
>>> l[2:5] = 100  1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
>>> l[2:5] = [100] # 删除并且替换了部分
>>> l
[0, 1, 100, 22, 9]
```

## 多维数组
```python
>>> board = [['_'] * 3 for i in range(3)]  1
>>> board
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
>>> board[1][2] = 'X'  2
>>> board
[['_', '_', '_'], ['_', '_', 'X'], ['_', '_', '_']]
```

注意 * 和 + 如果作用到不可变类型，那么id(变量)是会变的

## bisect
用于二分查找和插入
bisect.bisect , bisect.bisect_left 和 bisect.bisect_right (遇到相等的是插入左边还是右边)

```python
bisect.bisect(a, x, lo=0, hi=len(a))
```

```python
bisect.insort(seq, item)
```

## array
更加节省内存，支持 insert pop extend 等操作
```pyton
floats = array('d', (random() for i in range(10**7)))
```

## deque

```python
>>> from collections import deque
>>> dq = deque(range(10), maxlen=10)  1
>>> dq
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.rotate(3)  2
>>> dq
deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)
>>> dq.rotate(-4)
>>> dq
deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
>>> dq.appendleft(-1)  3
>>> dq
deque([-1, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.extend([11, 22, 33])  4
>>> dq
deque([3, 4, 5, 6, 7, 8, 9, 11, 22, 33], maxlen=10)
>>> dq.extendleft([10, 20, 30, 40])  5
>>> dq
deque([40, 30, 20, 10, 3, 4, 5, 6, 7, 8], maxlen=10)
dq.popLeft()
```

+ 其他queue。SimpleQueue, Queue, LifoQueue, and PriorityQueue
+ heapq  提供 heappush  heappop 两种方法。类似优先队列

# dict and set
键值需要是 hashable 的

## dict

初始化
```python
>>> a = dict(one=1, two=2, three=3)
>>> b = {'three': 3, 'two': 2, 'one': 1}
>>> c = dict([('two', 2), ('one', 1), ('three', 3)])
>>> d = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
>>> e = dict({'three': 3, 'one': 1, 'two': 2})
>>> a == b == c == d == e
True


{code: country.upper()                                          3
...     for country, code in sorted(country_dial.items())
...     if code < 70}
```

方法
```python
list(m.keys())
m.popitem()
d.fromkeys(it, [initial]) # New mapping from keys in iterable, with optional initial value (defaults to None)
d.get(k, [default]) # Get item with key k, return default or None if missing
d.keys() 
d.values()
d.items()
d.pop(k, [default])
d.setdefault(k, [default]) # If k in d, return d[k]; else set d[k] = default and return it
```
其中 setdefault 和 get 很好用
除了 dict 还有 defaultdict 和 OrderedDict

```python
index = collections.defaultdict(list)
index[word].append(location) # 返回默认的list
```

## set
初始化
```python
 a = {1, 2, 3}
 a = set([1,2,3])
 {chr(i) for i in range(32, 256) if 'SIGN' in name(chr(i),'')} # setcomps
```

交集 &
```python
found = len(needles & haystack) # set的交集
found = len(set(needles).intersection(haystack))
```

并集 |
```python
a | b
```

差 -
```python
s - z # Relative complement or difference between s and z
```

异或 ^
```python
s ^ z # (s | z) - (s & z)
```

包含
```python
e in s
s <= z
s < z
```

基本方法
```python
s.discard(e) # Remove element e from s if it is present
s.remove(e) # Remove element e from s, raising KeyError if e not in s
```

# 字符串 略

python中没有string buffer，如果效率高的话，使用以下
```python
''.join(['abc' for _ in range(loop_count)])
```

# object references, mutalbility and recycling

copy
```python
import copy
bus1 = Bus(['Alice', 'Bill', 'Claire', 'David'])
bus2 = copy.copy(bus1)
bus3 = copy.deepcopy(bus1)
```

# python tips

## pythonic thinking
+ 变量、函数名、属性命名方式为 lowercase_underscore
+ tuple进行unpacking使用命名的方式，而不是indexing

```python
item = ('Peanut butter', 'Jelly')
first, second = item # Unpacking
```

+ 循环的时候，使用 enumerate 代替 range
```python
for i, flavor in enumerate(flavor_list):
    print(f'{i + 1}: {flavor}')
```

+ 并行计算的时候可以使用zip函数，可以把两个iterator连接起来

```python
for name, count in zip(names, counts):
    if count > max_count:
        longest_name = name
        max_count = count

# 如果不一样长
import itertools
for name, count in itertools.zip_longest(names, counts):
    print(f'{name}: {count}')
```

+ 使用 := 进行赋值并返回

普通用法
```python
if (count := fresh_fruit.get('apple', 0)) >= 4:
    make_cider(count)
else:
    out_of_stock()
```

代替 do-while
```python
while fresh_fruit := pick_fruit():
    for fruit, count in fresh_fruit.items():
        batch = make_juice(fruit, count)
        bottles.extend(batch)
```

## lists and dicts

+ slice会忽略 out of bounds
+ 给slice赋值一个list，原来slice的位置会被替代
+ stride 正负决定了slice起点
```python
x = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
x[::2]   # ['a', 'c', 'e', 'g']
x[::-2]  # ['h', 'f', 'd', 'b']
```
+ 排序 使用key 
```python
places.sort(key=lambda x: x.lower())
```

+ 使用get 而并非 in进行默认值选取
```python
s.get(key, 0)
```

+ 使用defaultdict 代替 setDefault
```python
from collections import defaultdict
class Visits:
    def __init__(self):
       self.data = defaultdict(set)

    def add(self, country, city):
       self.data[country].add(city)
```

## functions
+ 不要返回超过3个变量
+ 使用可变长参数代替list
```python
def log(message, *values): # The only difference
```

+ 注意使用keyword argument的方法

keyword argument方法要在positional方法后面

```python
my_kwargs = {
    'number': 20,
    'divisor': 7,
}
assert remainder(**my_kwargs) == 6
```