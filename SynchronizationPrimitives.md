[toc]



### Synchronization Primitives

[原文件](https://docs.python.org/zh-cn/3/library/asyncio-sync.html)

asyncio synchronization primitives are designed to be similar to those of the threading module with two important caveats:

> asyncio 同步基元被设计为类似于线程模块，两个重要的注意事项：

- asyncio primitives are not thread-safe, therefore they should not be used for OS thread synchronization (use threading for that);

> asyncio 基元不是安全的线程，因此他们不应该使用于OS线程同步（使用线程）

- methods of these synchronization primitives do not accept the *timeout* argument; use the asyncio.wait_for() function to perform operations with timeouts.

> 这些同步基元的方法不接受”timeout“参数；使用asyncio.wait_for()函数来执行超时操作

asyncio has the following basic synchronization primitives:

> asyncio 有如下基本的同步基元：

- Lock

- Event
- Condition
- Semaphore
- BoundedSemaphore

> * 锁
> * 事件
> * 条件
> * 信号量
> * 有界的信号量

### Lock

#### class asyncio.Lock(*, *loop=None*)

Implements a mutex lock for asyncio tasks. Not thread-safe.

> asyncio任务实现量异步锁。非安全线程

An asyncio lock can be used to guarantee exclusive access to a shared resource.

> 

The preferred way to use a Lock is an async with statement:

```python
lock = asyncio.Lock()

# ... later
async with lock:
    # access shared state
```

which is equivalent to:

```python
lock = asyncio.Lock()

# ... later
await lock.acquire()
try:
    # access shared state
finally:
    lock.release()
```

**coroutine acquire()**

Acquire the lock.

This method waits until the lock is *unlocked*, sets it to *locked* and returns True.

When more than one coroutine is blocked in acquire() waiting for the lock to be unlocked, only one coroutine eventually proceeds.

Acquiring a lock is *fair*: the coroutine that proceeds will be the first coroutine that started waiting on the lock.

**release()**

Release the lock.

When the lock is *locked*, reset it to *unlocked* and return.

If the lock is *unlocked*, a RuntimeError is raised.

**locked()**

Return True if the lock is locked.

### Event

#### *class* asyncio.Event(***, *loop=None*)

An event object. Not thread-safe.

An asyncio event can be used to notify multiple asyncio tasks that some event has happened.

An Event object manages an internal flag that can be set to *true* with the set() method and reset to *false* with the clear() method. The wait() method blocks until the flag is set to *true*. The flag is set to *false* initially.

示例:

```python
async def waiter(event):
    print('waiting for it ...')
    await event.wait()
    print('... got it!')

async def main():
    # Create an Event object.
    event = asyncio.Event()

    # Spawn a Task to wait until 'event' is set.
    waiter_task = asyncio.create_task(waiter(event))

    # Sleep for 1 second and set the event.
    await asyncio.sleep(1)
    event.set()

    # Wait until the waiter task is finished.
    await waiter_task

asyncio.run(main())
```

***coroutine* wait()**

Wait until the event is set.

If the event is set, return True immediately. Otherwise block until another task calls set().

**set()**

Set the event.

All tasks waiting for event to be set will be immediately awakened.

**clear()**

Clear (unset) the event.

Tasks awaiting on wait() will now block until the set() method is called again.

**is_set()**

Return True if the event is set

### Condition

#### *class* asyncio.Condition(lock=None, *, loop=None)

A Condition object. Not thread-safe.

An asyncio condition primitive can be used by a task to wait for some event to happen and then get exclusive access to a shared resource.

In essence, a Condition object combines the functionality of an Event and a Lock. It is possible to have multiple Condition objects share one Lock, which allows coordinating exclusive access to a shared resource between different tasks interested in particular states of that shared resource.

The optional *lock* argument must be a Lock object or None. In the latter case a new Lock object is created automatically.

*Deprecated since version 3.8, will be removed in version 3.10:* *loop* 形参。

The preferred way to use a Condition is an async with statement:

```python
cond = asyncio.Condition()

# ... later
async with cond:
    await cond.wait()
```

which is equivalent to:

```python
cond = asyncio.Condition()

# ... later
await cond.acquire()
try:
    await cond.wait()
finally:
    cond.release()
```

**coroutine acquire()**

Acquire the underlying lock.

This method waits until the underlying lock is *unlocked*, sets it to *locked* and returns True.

**notify(n=1)**

Wake up at most *n* tasks (1 by default) waiting on this condition. The method is no-op if no tasks are waiting.

The lock must be acquired before this method is called and released shortly after. If called with an *unlocked* lock a RuntimeError error is raised.

**locked()**

Return True if the underlying lock is acquired.

**notify_all()**

Wake up all tasks waiting on this condition.

This method acts like notify(), but wakes up all waiting tasks.

The lock must be acquired before this method is called and released shortly after. If called with an *unlocked* lock a RuntimeError error is raised.

**release()**

Release the underlying lock.

在未锁定的锁调用时，会引发 RuntimeError 异常。

**coroutine wait()**

Wait until notified.

If the calling task has not acquired the lock when this method is called, a RuntimeError is raised.

This method releases the underlying lock, and then blocks until it is awakened by a notify() or notify_all() call. Once awakened, the Condition re-acquires its lock and this method returns True.

**coroutine wait_for(predicate)**

Wait until a predicate becomes *true*.

The predicate must be a callable which result will be interpreted as a boolean value. The final value is the return value.

### Semaphore

#### *class* asyncio.Semaphore(value=1, *, loop=None)

A Semaphore object. Not thread-safe.

A semaphore manages an internal counter which is decremented by each acquire() call and incremented by each release() call. The counter can never go below zero; when acquire() finds that it is zero, it blocks, waiting until some task calls release().

The optional *value* argument gives the initial value for the internal counter (`1` by default). If the given value is less than `0` a ValueError is raised.

*Deprecated since version 3.8, will be removed in version 3.10:* *loop* 形参。

The preferred way to use a Semaphore is an async with statement:

```python
sem = asyncio.Semaphore(10)

# ... later
async with sem:
    # work with shared resource
```

which is equivalent to:

```python
sem = asyncio.Semaphore(10)

# ... later
await sem.acquire()
try:
    # work with shared resource
finally:
    sem.release()
```

**coroutine acquire()**

获取一个信号量。

If the internal counter is greater than zero, decrement it by one and return `True` immediately. If it is zero, wait until a release() is called and return `True`.

**locked()**

Returns `True` if semaphore can not be acquired immediately.

**release()**

Release a semaphore, incrementing the internal counter by one. Can wake up a task waiting to acquire the semaphore.

Unlike BoundedSemaphore, Semaphore allows making more `release()` calls than `acquire()` calls.

### BoundedSemaphore

#### *class* asyncio.BoundedSemaphore(value=1, *, loop=None)

A bounded semaphore object. Not thread-safe.

Bounded Semaphore is a version of Semaphore that raises a ValueError in release() if it increases the internal counter above the initial *value*.

*Deprecated since version 3.8, will be removed in version 3.10:* *loop* 形参。

