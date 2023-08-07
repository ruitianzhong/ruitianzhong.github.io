---
title: "Python Notes"
date: 2023-08-07T20:10:00+08:00
summary: "Python Cheatsheets"
showtoc: false
categories:
- Python
---

## Regular Expression

```python
"""
Regular Expression Playground
"""

import re

print(re.match('www', 'www.google.com').span())

print(re.search('www', 'www.google.com').span())

pattern = re.compile(r'\d+')
print(pattern.match('123456 12345').group())
print(pattern.findall('123456 12345'))
num = re.sub(r'\D', "", "2000-01-01")
print(num)


def double(matched):
    value = int(matched.group('value'))
    return str(value * 2)


print(re.sub(r'(?P<value>\d+)', double, '2222'))
s = "1102231990xxxxxx"
res = re.search(r'(?P<province>\d{3})(?P<city>\d{3})(?P<born_year>\d{4})', s)
print(res.groups())
print(res.groupdict())
# 分组匹配
r"""
re.l 忽略大小写
re.M 多行模式
re.U 和Unicode字符属性数据库相关
^ 开头
$ 结尾
.匹配任意字符，除了换行符
[...]
[^...]
re*
re+
re?
re{n}
re(n,}
re{n,m}
a | b
(re) group
\w 字母数字+下划线
\w 非字母数 + 下划线
\d
\D
\A
\Z
\b
\B
\S 任意非空字符
\s
\1 \9
\n \t
"""

```

## Loop

+ while
+ for

```python
l1 = [1, 2, 3, 4, 5]

for i in l1:
    print(i)

for index in range(len(l1)):
    print(l1[index])
else:
    print("no break")
```

## is & ==

```python
a = b = 100
if a is b:
    print('same memory area')
```

## Basic Data Structure

```python
# complex
complex_v = 10 + 2j
print(complex_v)

# List
l1 = ['Tim Cook', 100, 2.33, 'john']
# Dictionary
dict1 = {'one': 1, 'two': 'two'}
print(dict1['one'])
dict1['two'] = 2
print(dict1.keys())
print(dict1.values())


```

## Pass

No operation

## Exception

```python
def raiseException():
    raise Exception(100)


class NetworkError(RuntimeError):
    def _init__(self, arg):
        self.args = arg


try:
    raise NetworkError("Bad hostname")
except NetworkError as e:
    print(e)

```

```python
try:
    fh = open('test.txt', 'w')
    fh.write("hello world")
except(SystemError, SystemExit):
    print("")
except IOError:
    print("IO error")
else:
    print("succeed")
finally:
    print("finally")

fh.close()
```

## OOP Related

```python{linenos=true,style = "github"}
class Animal:
    """This is a animal class"""
    count = 0  # shared
    __private_var = 0
    _protected = 0

    def __init__(self, name, age):
        self.name = name
        self.age = age
        Animal.count += 1

    def displayName(self):
        self._protected += 1
        print(self.name)

    def displayAge(self):
        print(self.age)

    def getName(self):
        return self.name

    def __del__(self):
        print('delete ' + self.name)


dog = Animal("Dog", 1)
cat = Animal("Cat", 2)
assert Animal.count == 2
assert "Dog" == dog.getName()
assert dog._Animal__private_var == 0
print(dog.__doc__)
print(dog.__dict__.keys())
del cat


class Dog(Animal):

    def run(self):
        Animal.displayName(self)
        print("Dog " + self.name + "is running")

    def getName(self):
        self._protected += 2
        print("this is dog")


# Python 总是首先查找对应类型的方法，
# 如果它不能在派生类中找到对应的方法，
# 它才开始到基类中逐个查找。
# （先在本类中查找调用的方法，找不到才去基类中找）。
dog = Dog("dog 1", 100)


class LittleDog(Dog):
    def __init__(self):
        print("Little Dog")
        self.name = "Little"
        self.age = 0


dog = LittleDog()
dog.getName()

```
