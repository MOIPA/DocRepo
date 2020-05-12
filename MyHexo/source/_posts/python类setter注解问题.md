title: python类setter注解问题
author: tr
tags:
  - Python
categories:
  - Python
date: 2019-07-10 11:16:00
---
### 概述

配置爬虫线程池时出现的问题：定义一个线程类，在类内方法外声明变量，按照python规则，这个是非实例所能拥有的静态变量。

变量为`dict`类型`{}`。且为这个变量设定了setter和getter。为了能够让实例也能访问类变量使用了注解`property`。类代码如下

```python
def thread(input):
    for i in range(6):
        print('hello+'+str(input))
        time.sleep(2)
        if i == 4:
            print('stop')
  
class ThreadPool(object):
    _pool = {}
    
    @property
    def pool(self):
        return ThreadPool._pool

	@pool.setter
    def pool(self, pool):
        ThreadPool._pool = pool
 
```

### 问题

以下是使用方式：第三行为什么可以直接使用dict的方法新增？就像是直接访问变量而不是通过setter和getter一样，待解决。

```python
tp = ThreadPool()
timer = threading.Timer(1, thread, [1])
tp.pool[timer.name]=timer
print(tp.pool)
timer.start()
```