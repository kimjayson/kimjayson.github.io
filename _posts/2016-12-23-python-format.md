---
layout: post
title: Python用format格式化字符串整理
categories: Python
description: 格式化字符串format。
keywords: 基础, Python, 格式化
---

format是是python2.6新增的一个格式化字符串的方法，相对于老版的%格式方法，它有很多优点:

1.单个参数可以多次输出，参数顺序可以不相同

2.填充方式非常灵活，对齐方式十分强大

3.官方推荐用的方式，%方式将被淘汰

4.pyton3.6+版本，可以使用f'{version}'的方式了，更加方便

### 例子

```
print('hello {0}'.format('world'))
```

### 参数格式

format 的格式

```
replacement_field     ::=  "{" [field_name] ["!" conversion] [":" format_spec] "}"

field_name             ::=      arg_name ("." attribute_name | "[" element_index "]")*
arg_name               ::=      [identifier | integer]
attribute_name         ::=      identifier
element_index          ::=      integer | index_string
index_string           ::=      <any source character except "]"> +
conversion             ::=      "r" | "s" | "a"
format_spec            ::=      <described in the next section>
```

format_spec 的格式

```
format_spec 　　::=  　　[[fill]align][sign][#][0][width][,][.precision][type]

fill        　　　　::=  　　<any character>
align       　　　　::=  　　"<" | ">" | "=" | "^"
sign       　　　　 ::=  　　"+" | "-" | " "
width      　　　　 ::= 　　 integer
precision   　　　　::= 　　 integer
type        　　　　::=  　　"b" | "c" | "d" | "e" | "E" | "f" | "F" | "g" | "G" | "n" | "o" | "s" | "x" | "X" | "%"
```

### 使用

#### 一. 填充

##### 1. 通过位置来填充字符串
```
print('hello {0} i am {1}'.format('Jin','Tom'))                   # hello Jin i am Tom
print('hello {} i am {}'.format('Jin','Tom'))                     # hello Jin i am Tom
print('hello {0} i am {1} . my name is {0}'.format('Jin','Tom'))  # hello Jin i am Tom . my name is Jin
```
foramt会把参数按位置顺序来填充到字符串中，第一个参数是0，然后1

也可以不输入数字，这样也会按顺序来填充

同一个参数可以填充多次，这个是format比%先进的地方

##### 2. 通过key来填充
```
print('hello {name1}  i am {name2}'.format(name1='Jin',name2='Tom')) # hello Jin i am Tom
```

##### 3. 通过下标填充
```
names=['Jin','Tom']
print('hello {names[0]}  i am {names[1]}'.format(names=names)) # hello Jin i am Tom
print('hello {0[0]}  i am {0[1]}'.format(names))               # hello Jin i am Tom
```

##### 4. 通过字典的key
```
names={'name':'Jin','name2':'Tom'}
print('hello {names[name]}  i am {names[name2]}'.format(names=names)) # hello Jin i am Tom
```
**注意**:访问字典的key，不用引号的

##### 5. 通过对象的属性
```
class Names():
    name1='Jin'
    name2='Tom'

print('hello {names.name1}  i am {names.name2}'.format(names=Names)) # hello Jin i am Tom
```

##### 6. 使用魔法参数
```
args=['Jay','Mick']
kwargs = {'name1': 'Jin', 'name2': 'Tom'}
print('hello {name1} {} {} i am {name2}'.format(*args, **kwargs))  # hello Jin Jay Mick i am Tom
```

#### 二. 格式转换

b、d、o、x分别是二进制、十进制、八进制、十六进制。

数字        | 格式    | 输出      | 描述
----------- | ------- | --------  | --------------
3.1415926   | {:.2f}  | 3.14      | 保留小数点后两位
3.1415926   | {:+.2f} | 3.14      | 带符号保留小数点后两位
-1          | {:+.2f} | -1        | 带符号保留小数点后两位
2.71828     | {:.0f}  | 3         | 不带小数
1000000     | {:,}    | 1,000,000 | 以逗号分隔的数字格式
0.25        | {:.2%}  | 25.00%    | 百分比格式
1000000000  | {:.2e}  | 1.00E+09  | 指数记法
25          | {0:b}   | 11001     | 转换成二进制
25          | {0:d}   | 25        | 转换成十进制
25          | {0:o}   | 31        | 转换成八进制
25          | {0:x}   | 19        | 转换成十六进制

#### 三. 对齐与填充

数字   | 格式   | 输出        | 描述
------ | ------ | ----------- | ---------------
5      | {:0>2} |   05        | 数字补零 (填充左边, 宽度为2)
5      | {:x<4} |   5xxx      | 数字补x (填充右边, 宽度为4)
10     | {:x^4} |   x10x      | 数字补x (填充右边, 宽度为4)
13     | {:10}  |          13 | 右对齐 (默认, 宽度为10)
13     | {:<10} | 13          | 左对齐 (宽度为10)
13     | {:^10} |      13     | 中间对齐 (宽度为10)

#### 四. 其他用法

##### 1. 转义{和}符号
```
print(" { { hello {0} } } ".format('Jin'))
```
跟%中%%转义%一样，formate中用两个大括号来转义，嵌套里面不能用两个大括号

**注意:**双大括号之间没有空格，这里因为jekyll转义问题暂时没解决。

##### 2. format作为函数
```
f = 'hello {0} i am {1}'.format    
print(f('Jin','Tom'))
```

##### 3. 格式化datetime
```
now=datetime.datetime.now()
print('{:%Y-%m-%d %X}'.format(now))
```
这个要比python的datetime,time简单的多

##### 4. {}内嵌{}
```
print('hello {0:>{1}} '.format('Jin',10))
```

##### 5. 叹号的用法

！后面可以加s r a 分别对应str() repr() ascii()

作用是在填充前先用对应的函数来处理参数

```
print("{!s}".format('2'))   # 2
print("{!r}".format('2'))   # '2'
print("{!a}".format('2'))   # '2'
print("{!a}".format('测'))  # '\u6d4b'
```
差别就是repr带有引号，str()是面向用户的，目的是可读性，repr()是面向python解析器的，返回值表示在python内部的含义

**注意:**ascii() 3.0以上才有的函数，python2使用会报错。

参考：<https://docs.python.org/3/library/string.html#grammar-token-conversion>


