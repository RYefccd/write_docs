
# 并行编程

[TOC]

- **Processes** speed up Python operations that are CPU intensive because they benefit from multiple cores and avoid the GIL.
- **Threads** are best for IO tasks or tasks involving external systems because threads can combine their work more efficiently. Processes need to pickle their results to combine them which takes time.
- **Threads** provide no benefit in python for CPU intensive tasks because of the GIL.

## 进程

## 线程（threading）

```
├── threading.py
```

类：

函数：

1. **Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)**

   线程对象。

   target：为将被 run() 方法调用的可调用对象。如果是 None，那就意味着什么也不做

   1. `start()`：开始线程的执行。
      本方法至多只能调用一次。它会在独立线程中执行 run() 方法。

   2. `run()`：start() 方法是启动一个子线程。run() 方法并不启动一个新线程，就是在主线程中调用了一个普通函数而已。

      本方法可以在子类中覆盖。否则默认调用传给构造器的 target 参数（和 args，kwargs 参数）

   3. `join(timeout=None)`：被调用 join() 方法的线程会一直阻塞调用者的线程，直到自己结束（正常结束，或引发未处理异常），或超出 timeout 的时间。

   4. `name()`、`ident()`：获取用于鉴别进程的一个字符串、获取本线程的“thread identifier”。

   5. `is_alive()`、`daemon()`：返回本进程是否是 alive 状态、返回本进程是否为守护进程。

2. **Event()**

   在初始情况下，event 对象中的信号标志(flag)被设置为假。最好单次使用。

   如果有线程等待一个 event 对象，而这个 event 对象的标志为假，那么这个线程将会被一直阻塞直至该标志为真。

   一个线程如果将一个 event 对象的信号标志设置为真，它将唤醒所有等待这个 event 对象的线程。

   1. `is_set()`：返回flag是否为True。
   2. `set()`：将flag设置为True。所有等待的线程将被唤醒。
   3. `clear()`：将 flag 重置为 False。
   4. `wait(timeout=None)`：阻塞线程直到 flag 被置为 True，或超时。

3. **Condition(lock=None)**

   用于实现条件变量对象。条件变量对象允许多条线程保持等待状态直到接收另一条线程的通知。

   1. `acquire(*args)`、`release()`：获取、释放锁。

   2. `wait(timeout=None)`、`wait_for(predicate,timeout=None)`：等待（predicate）通知或者超时。

      wait_for(predicate,timeout=None)在忽略超时参数时，约等于如下：

      ```python
      while not predicate():
          cv.wait()
      ```

   3. `notify(n=1)`、`notify_all()`：唤醒处于等待本条件变量的线程。

4. **Semaphore(value=1)**

   Semaphore 在内部管理着一个计数器（value）。

   调用 acquire() 会使这个计数器 -1，release() 则是 +1。

   计数器的值永远不会小于 0，当计数器到 0 时，再调用 acquire() 就会阻塞，直到其他线程来调用 release()。

   1. `acquire(blocking=True, timeout=None)`：本方法用于获取 Semaphore。

      如果 blocking=False，则不阻塞，但若获取失败的话，返回 False。

   2. `release()`：释放 Semaphore，给内部计数器 +1，可以唤醒处于等待状态的线程。

其他：

threading模块中所有包含`acquire()`和`release()`方法的对象都可以当做上下文管理器（`with`）使用。

`___enter__`时调用acquire()方法获得锁，`__exit__`时调用`release()`方法释放锁。

```python
with some_lock:
    # do something...
```

is equivalent to:

```python
some_lock.acquire()
try:
    # do something...
finally:
    some_lock.release()
```



##`concurrent.futures` 

```
├──concurrent
│	├── futures
│   │   ├── __init__.py
│   │   ├── _base.py
│   │   ├── thread.py
│   │   └── process.py
```

类：

1. **Executor**

   提供异步操作的抽象类。（可以调用with语句）

   1. `submit(fn, *args, **kwargs)`：提交单个任务，返回future对象。

   2. `map(func, *iterables, timeout=None, chunksize=1)`：提交多个命令，返回一个组成元素为future对象的生成器（按照输入的顺序）。

      timeout：超时，future对象仍未完成抛异常。

   3. `shutdown(wait=True)`：通知executor释放所有正在使用的资源。

      `__exit__`中调用了改方法。

      wait：为true时，等待所有未完成的future实例完成。

2. **ThreadPoolExecutor(*max_workers=None*, *thread_name_prefix=''*, *initializer=None*, *initargs=()*)**

   线程池

3. **ProcessPoolExecutor(*max_workers=None*, *mp_context=None*, *initializer=None*, *initargs=()*)**

   进程池

4. **Future()**

   1. `cancelled()`、`running()`、`done()`：返回状态。

   2. `cancel()`：取消future实例

   3. `result(timeout=None)`、`exception(timeout=None)`：返回future实例的结果或异常。

      会阻塞进程直到结果被返回来。

   4. `add_done_callback(fn)`：增加回调函数。

函数：

1. **as_completed**

   输入：(*fs*, *timeout=None*)，future实例

   返回：生成器（future实例完成时返回的结果，按照完成的顺序）

2. **wait**

   输入：(*fs*, *timeout=None*, *return_when=ALL_COMPLETED*)，

   return_when：FIRST_COMPLETED，FIRST_EXCEPTION，ALL_COMPLETED

   输出：named 2-tuple of sets：(done, not_done)。根据timeout、return_when参数等待fs完成

示例：

```python
import concurrent.futures

def when_done(r):
    print('Got:', r.result())

# 回调函数
with concurrent.futures.ThreadPoolExecutor() as executor:
    future_result = executor.submit(work, arg)
    future_result.add_done_callback(when_done)

def f(x):
    return x

# submit提交单个命令 + as_completed；返回结果的顺序为实际完成顺序
# 返回：20413(每次运行结果不同)
with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
    f_list = [executor.submit(f, i) for i in range(5)]
    for future in concurrent.futures.as_completed(f_list):
        data = future.result()
        print(data)

# map；返回结果顺序与输入相同
# 返回：01234
with concurrent.futures.ProcessPoolExecutor() as executor:
    for res in executor.map(f, [i for i in range(5)]):
        print(res)
```

## 协程

 Coroutines run synchronously until they hit an `await` and then they pause, give up control to the event loop, and something else can happen.

直接调用异步函数不会返回结果，而是返回一个coroutine对象

async 定义一个协程，await 用来挂起阻塞方法的执行
