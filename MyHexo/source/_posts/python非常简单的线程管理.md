title: python非常简单的线程管理
author: tr
tags:
  - Python
categories:
  - Python
date: 2019-07-10 16:40:00
---
### 问题

多线程爬虫，带一个网页后台，调用接口传递json即可多线程爬虫。因为爬虫时间长需要提供接口取消线程。（每个搜索主题对应一个线程）


### 代码

```python
import threading                                                                                                                                                   
--import sys
  import time
  
  
--def thread(d_input, pool, thread_id):
--    for i in range(6):
          if pool.pool[thread_id] == -1:
              exit()
--        print('thread run '+str(thread_id)+'****'+str(d_input))
          time.sleep(2)
  
  
--class ThreadPool(object):
      _pool = {}
      _count = 0
  
      @property
--    def pool(self):
          return ThreadPool._pool
  
      @property
--    def count(self):
          return ThreadPool._count
  
      @count.setter
--    def count(self, count):
          ThreadPool._count = count
  
      @pool.setter
--    def pool(self, pool):
          ThreadPool._pool = pool
  
  
--def mainThread():
      # prepare the thread pool
      pool = ThreadPool()
      # start up ten thread
      for i in range(10):
          # start up a thread and pass the thread id and pool object
          timer = threading.Timer(1, thread, [i, pool, pool.count])
          # store the pool id
          pool.pool[pool.count] = 1 
          pool.count += 1
          timer.start()
  
      # start for a while
      time.sleep(4)
      # end one thread
      pool.pool[0] = -1
      pool.pool[1] = -1
      pool.pool[2] = -1
      pool.pool[3] = -1
      pool.pool[4] = -1
      pool.pool[5] = -1
  
  
  mainThread()        
```