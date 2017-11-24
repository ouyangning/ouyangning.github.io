---
title: 求N以内质数的Python实现
date: 2017-11-18 17:09:24
tags:
- 质数
- 算法
categories:
- 数据结构与算法
---

## 质数的定义

质数（prime number）又称素数，有无限个。除了1和它本身以外不再有其他的除数整除。

### Normal

首先来看一个最`normal`的实现方式

```Python


def prime_1(n):
    if n <= 1:
        return 0
    for i in range(2, n):
        # 要判断n是不是一个质数 就要看能不能在2到n之间找到一个可以被n整除的除数
        if n % i == 0:
            return 0
    return 1


def primes_3(n):
    for i in range(2, n+1):
        if prime(i):
            pass

```

![Normal](http://op7aviu2v.bkt.clouddn.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/image/png/Normal.png)

这是求N以内的质数最简单的一种方式，所消耗的时间也很多。那么有没有更快的方法呢？

### 埃氏筛

[埃拉托斯特尼](https://baike.baidu.com/item/%E5%9F%83%E6%8B%89%E6%89%98%E6%96%AF%E7%89%B9%E5%B0%BC)筛法，简称埃氏筛或爱氏筛，是一种由希腊数学家埃拉托斯特尼所提出的一种简单检定质数的算法。

给出要筛数值的范围n，找出以内的质数。先用2去筛，即把2留下，把2的倍数剔除掉；再用下一个质数，也就是3筛，把3留下，把3的倍数剔除掉；接下去用下一个质数5筛，把5留下，把5的倍数剔除掉；不断重复下去......

```python


def primes_1(n):
    p = []
    f = []
    for i in range(n + 1):
        if i > 2 and i % 2 == 0:
            f.append(1)
        else:
            f.append(0)
    # 筛去所有大于2的偶数
    i = 3
    while i * i <= n:
        # 在3到根号n之间寻找能整除3到n之间的奇数的除数
        # 用这些除数筛掉奇数中的合数 得到的就是质数
        if f[i] == 0:
            j = i * i
            while j <= n:
                f[j] = 1
                j += 2 * i
                # 在筛去一轮偶数之后剩下的都是奇数 如果这个奇数是合数
                # 则该奇数的两个除数也应该都是奇数
                # i是奇数 i*i也是奇数 同样的j = i*i + 偶数 * i也应该是奇数
        i += 2

    p.append(2)
    for x in range(3, n + 1, 2):
        if f[x] == 0:
            p.append(x)

    return p

```

![Normal_埃氏筛](http://op7aviu2v.bkt.clouddn.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/image/png/Normal_%E5%9F%83%E6%B0%8F%E7%AD%9B.png)

那么还有没有更快的方法呢？

### 开根号法

```python


def prime_2(n):
    if n <= 1:
        return 0
    for i in range(2, int(math.sqrt(n)+1)):
        # 要判断n是不是一个质数 就要看能不能在2到n之间找到一个可以被n整除的除数
        # 假设存在这样一个除数a 使得a * b = n 那么a b之中必有一个数大于根号n
        # 因此 只要小于或等于根号n的数（1除外）不能整除n 则n一定是质数
        if n % i == 0:
            return 0
    return 1


def primes_4(n):
    for i in range(2, n+1):
        if prime_2(i):
            pass
```

![Normal_埃氏筛_开根号法](http://op7aviu2v.bkt.clouddn.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/image/png/Normal_%E5%9F%83%E6%B0%8F%E7%AD%9B_%E5%BC%80%E6%A0%B9%E5%8F%B7%E6%B3%95.png)

那么如果埃氏筛也使用了开根号法，会不会更快呢？

### 埃氏筛加开根号法

```Python


def primes_1(n):
    p = []
    f = []
    for i in range(n + 1):
        if i > 2 and i % 2 == 0:
            f.append(1)
        else:
            f.append(0)
    # 筛去所有大于2的偶数
    i = 3
    while i * i <= n:
        # 在3到根号n之间寻找能整除3到n之间的奇数的除数
        # 用这些除数筛掉奇数中的合数 得到的就是质数
        if f[i] == 0:
            j = i * i
            while j <= n:
                f[j] = 1
                j += 2 * i
                # 在筛去一轮偶数之后剩下的都是奇数 如果这个奇数是合数
                # 则该奇数的两个除数也应该都是奇数
                # i是奇数 i*i也是奇数 同样的j = i*i + 偶数 * i也应该是奇数
        i += 2

    p.append(2)
    for x in range(3, n + 1, 2):
        if f[x] == 0:
            p.append(x)

    return p

```

![Normal_埃氏筛_开根号法_埃氏筛加开根号法](http://op7aviu2v.bkt.clouddn.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/image/png/Normal_%E5%9F%83%E6%B0%8F%E7%AD%9B_%E5%BC%80%E6%A0%B9%E5%8F%B7%E6%B3%95_%E5%9F%83%E6%B0%8F%E7%AD%9B%E5%8A%A0%E5%BC%80%E6%A0%B9%E5%8F%B7%E6%B3%95.png)

