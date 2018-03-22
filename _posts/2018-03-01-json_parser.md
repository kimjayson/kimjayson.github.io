---
layout: post
title: 简易json解析器
categories: Python
description: python 语法解析器。
keywords: 基础, Python, json, 解析器
---
之前面试的时候被问到json解析器的思路，当时只是回答了一下，并没有写，现在写个简单的解析器练练手。

### JSON的数据类型

对象，数组，布尔，null，字符串，数值

### 思路

json的设计你可以读第一个字符就能知道后面要如何解析，具体来讲：

- 一个左花括号（{）时，你知道你需要从当前字符开始，解析出一个对象，而解析对象的过程则是解析出多对 key/value，每解析完一对，如果你遇到了右花括号（}），则对这个对象的解析就结束了，而如果遇到的不是右花括号，则你需要解析下一对 key/value，直到遇见右花括号。而解析出一对 key/value，则是必须先解析出一个字符串，然后一个冒号（:），然后是这个 key 对应的值。
- 同理，当你遇到一个左方括号（[）时，你需要从这个字符开始解析出一个数组；而解析数组，则是解析出多个由逗号分隔的值，直到解析完一个值后遇到的不是逗号而是右方括号（]），则这个数组解析完毕。
- 而当你遇到一个双引号（"）时，则需要从当前位置开始解析出一个字符串。
- 遇到字母 “n” 的话，则是从当前位置开始往后读 4 个字符，且读到的 4 个字符组成的字符串必须是“null”，否则就应该报错。
- 遇到字母 “t” 的话，则是从当前位置开始往后读 4 个字符，且读到的 4 个字符组成的字符串必须是“true”，否则就应该报错。
- 遇到字母 “f” 的话，则是从当前位置开始往后读 5 个字符，且读到的 5 个字符组成的字符串必须是“false”，否则就应该报错。
- 剩下的就是数值了。

代码逻辑：递归调用

### 代码逻辑

先定义解析值的函数

```
# 先定义大的框架
def parseValue(string):
    if string[i] == '{':
        return parseObject(string)
    elif string[i] == '[':
        return parseArray(string)
    elif string[i] == 'n':
        return parseNull(string)
    elif string[i] == 't':
        return parseTrue(string)
    elif string[i] == 'f':
        return parseFalse(string)
    elif string[i] == '"':
        return parseString(string)
    else:
        return parseNumber(string)
```
然后只要一步一步实现调用的函数就好了

```
# 之后一步步实现每个函数
# 所有的函数都是从i位置开始解析出一个对应类型的值，同时把i移动到解析完成后的下一位
def parseString(string):
    res = ''
    global i
    i += 1 # 开始解析之前，i是指向字符开始的双引号，但字符不能包括双引号
    while string[i] != '"':
        res += string[i]
        i += 1
    i += 1 # 解析完移动到下一个位置
    return res

def parseNull(string): # 简单粗暴，直接往后读4个字符出来
    global i

    content = string[i:i+4]
    if content == 'null':
        i += 4
        return None
    else:
        raise Exception('Unexpected char at pos: {}'.format(i))

def parseFalse(string):
    global i

    content = string[i:i+5]
    if content == 'false':
        i += 5
        return False
    else:
        raise Exception('Unexpected char at pos: {}'.format(i))

def parseTrue(string):
    global i
    content = string[i:i+4]
    if content == 'true':
        i += 4
        return True
    else:
        raise Exception('Unexpected char at pos: {}'.format(i))

def parseNumber(string):
    global i
    '''
    // 本函数的实现并没有考虑内容格式的问题，实际上JSON中的数值需要满足一个格式
    // 不过好在这个格式基本可以用正则表达出来，不过这里就不写了
    // 想写的话对着官网的铁路图写一个出来就行了
    // 并且由于最后调用了parseFloat，所以如果格式不对，还是会报错的
    '''
    numStr = '' # -2e+8 此处只要判断i位置还是数字字符，就继续读 ;为了方便，写了另一个helper函数
    while isNumberChar(string[i]):
        numStr += string[i]
        i += 1
    return numStr

# 判断字母C是否是组成数值的符号
def isNumberChar(c):
    chars = {'-':True,'+':True,'e':True,'E':True,'.':True}
    if chars.get(c):
        return True
    if c >= '0' and c <= '9':
        return True
    return False

# 解析数组,掐头去尾，一个值一个逗号，如果解析完一个值没有逗号，说明完成了
def parseArray(string):
    global i

    i += 1
    result = [] # [123,"asdf",true,false]
    while string[i] != ']':
        result.append(parseValue(string))
        if string[i] == ',':
            i += 1
    i += 1
    return result

# 解析对象，一如既往，掐头去尾，一个值可能是任意类型，调用parseValue，逗号下一组，没有解析完毕
def parseObject(string):
    global i

    i += 1
    result = {} # {"a":1,"b":"c"}
    while string[i] != '}':
        key = parseString(string)
        i += 1 # 直接跳过冒号：
        value = parseValue(string)
        result[key] = value
        if string[i] == ',':
            i += 1
    i += 1
    return result
```

最后调用parser函数

```
def parse(jsonStr):
    string = jsonStr
    return parseValue(string)

```



### 后续改进

> 1. 封装类
> 2. 处理空白，支持转义字符，unnicode转义符\u6211
> 3. 做json序列器
> 4. 做json格式化输出器


代码地址：<https://github.com/kimjayson/anyParser>

[转载请注明作者]
