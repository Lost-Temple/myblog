---
title: Python中装饰器的wraps的作用
categories:
  - Python
  - 基础
cover: 'https://s2.loli.net/2023/07/14/7mGfZnoCvybLQNc.jpg'
abbrlink: 19804
date: 2023-07-14 15:50:52
tags:
---

# 装饰器
`Python`装饰器`decorator`在实现的时候，被装饰后的函数其实已经是另外一个函数了（函数名等函数属性会发生改变），就相当于产生了`副作用`，`Python`的`functools`包中提供了一个叫`wraps`的`decorator`来消除这样的`副作用`。写一个`decorator`的时候，最好在实现之前加上`functools`的`wrap`，它能保留原有函数的`__name__`和`__doc__`。

# 示例1（副作用）
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        """decorator"""
        print('Calling decorated function...')
        return func(*args, **kwargs)

    return wrapper


@my_decorator
def example():
    """DocString"""
    print('Called example function.')


print(example.__name__, example.__doc__)

if __name__ == '__main__':
    example()

```
> 打印现来的是：wrapper decorator 而不是 example DocString

# 示例2（消除副作用）
```python
from functools import wraps


def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        """Decorator"""
        print('Calling decorated function...')
        return func(*args, **kwargs)

    return wrapper


@my_decorator
def example():
    """DocString"""
    print('Called example function')


if __name__ == '__main__':
    example()

```

> 无毒副作用, 这就是`@wraps`的作用