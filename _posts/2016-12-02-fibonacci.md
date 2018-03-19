---
layout: post
title: 青蛙跳台阶的问题——Fibonacci
categories: Algorithm
description: 使用 Fibonacci 来解决青蛙跳台阶的问题。
keywords: 算法，Fibonacci
---

这几天正在复习算法，今天在看一篇文章时偶然看到这个题目，想了一下居然没什么思路……（抱歉，实在太菜。)，文章中提示了一个关键词：Fibonacci 数列。然后我又小百度了一下，找了一个具体分析实例，结合两处，这才理清了思路。（汗啊……基础全忘光了，这以后咋办啊……深感担忧……)

### 问题描述

一只青蛙一次可以跳上 1 级台阶，也可以跳上 2 级台阶，请问这只青蛙跳上 n 级的台阶总共有多少种跳法？

### 问题分析

设青蛙跳上 n 级台阶的跳法为 f(n) 种。

设 Fibonacci 数列的第 x 项值为 fibo(x)。

1. 当 n=1 时，f(n)=1=fibo(2)
2. 当 n=2 时，f(n)=2=fibo(3)
3. 当 n>2 时，分析可知，在跳上第 n 级台阶前一步，必然是在第 (n-1) 或 (n-2) 级台阶，故有 f(n) = f(n-1) + f(n-2); 依此类推……

则有：

<del>f(n)</del>  
<del>= f(n-1) + f(n-2)</del>  
<del>= 2f(n-2) + f(n-3)</del>  
<del>= 3f(n-3) + 2f(n-4)</del>  
<del>= 5 f(n-4) + 3f(n-5)</del>  
<del>= 8f(n-5) + 5f(n-6)</del>  
<del>= ...</del>  
<del>= fibo(x+1)f(n-x)+fibo(x)f(n-(x+1))</del>  
<del>=...</del>  
<del>= fibo(n-1)f(n-(n-2)) + fibo(n-2)f(n-(n-1))</del>  
<del>= fibo(n-1)f(2) + fibo(n-2)f(1)</del>  

f(n) 的规律符合 Fibonacci 数列的规律，它与 Fibonacci 的区别是 Fibonacci 的前两个元素是 1，1，而 f(n) 的规律是 1，2，即可知有 `f(n)=fibo(n+1)`。

### 简单的 Python 实现

```
# 初级 普通函数 树形递归
def fib1(n):
    if n<=2 :
        return 1
    else:
        return fib1(n-1)+fib1(n-2)


# 中级 匿名函数 树形递归
fib2=lambda n:1 if n<=2 else fib2(n-1)+fib2(n-2)


# 高级 迭代
def fib3(n):
    x, y = 0, 1
    while(n):
        x, y, n = y, x+y, n-1 # 元组赋值
    return x


# 高级 尾递归
def fib4(n):
    def fib_iter(n,x,y):
        if n==0 : return x
        else : return fib_iter(n-1,y,x+y)

    return fib_iter(n,0,1)


# 高级 线性代数 二元向量[x,y]T [[1,0],[1,1]]*[x,y]T=[y,x+y]T 复杂度n
def fib5(n):
    def m1(a,b):
        m = [
            [a[0][0] * b[0][0] + a[0][1] * b[1][0],
             a[0][0] * b[0][1] + a[0][1] * b[1][1]
             ],
            [a[1][0] * b[0][0] + a[1][1] * b[1][0],
             a[1][0] * b[1][0] + a[1][1] * b[1][1]
             ]
        ]
        return m
    def m2(a,b):
        m = [
            a[0][0] * b[0][0] + a[0][1] * b[1][0],
            a[1][0] * b[0][0] + a[1][1] * b[1][0]
        ]
        return m
    return m2(reduce(m1,[[[0,1],[1,1]] for i in range(n)]),[[0],[1]])[0]


a = fib5(10)
print(a)


# 线性代数 logN
def fib(n):
    lhm = [[0, 1], [1, 1]]
    rhm = [[0], [1]]
    em = [[1, 0], [0, 1]]

    # multiply two matrixes
    def matrix_mul(lhm, rhm):
        # initialize an empty matrix filled with zero
        result = [[0 for i in range(len(rhm[0]))] for j in range(len(rhm))]
        print(result)
        # multiply loop
        for i in range(len(lhm)):
            for j in range(len(rhm[0])):
                for k in range(len(rhm)):
                    result[i][j] += lhm[i][k] * rhm[k][j]
        return result

    def matrix_square(mat):
        return matrix_mul(mat, mat)

    # quick transform 类比自然数的快速求幂
    def fib_iter(mat, n):
        if not n:
            return em
        elif (n % 2):
            return matrix_mul(mat, fib_iter(mat, n - 1))
        else:
            return matrix_square(fib_iter(mat, n / 2))

    return matrix_mul(fib_iter(lhm, n), rhm)[0][0]

```
