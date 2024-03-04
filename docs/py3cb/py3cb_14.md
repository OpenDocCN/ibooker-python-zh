# 第十二章：并发编程

对于并发编程, Python 有多种长期支持的方法, 包括多线程, 调用子进程, 以及各种各样的关于生成器函数的技巧.这一章将会给出并发编程各种方面的技巧, 包括通用的多线程技术以及并行计算的实现方法.

像经验丰富的程序员所知道的那样, 大家担心并发的程序有潜在的危险.因此, 本章的主要目标之一是给出更加可信赖和易调试的代码.

# 12.1 启动与停止线程

## 问题

You want to create and destroy threads for concurrent execution of code.

## 解决方案

The threading library can be used to execute any Python callable in its own thread. Todo this, you create a Thread instance and supply the callable that you wish to executeas a target. Here is a simple example:

# Code to execute in an independent threadimport timedef countdown(n):

> while n > 0:print(‘T-minus', n)n -= 1time.sleep(5)

# Create and launch a threadfrom threading import Threadt = Thread(target=countdown, args=(10,))t.start()

When you create a thread instance, it doesn’t start executing until you invoke its start()method (which invokes the target function with the arguments you supplied).Threads are executed in their own system-level thread (e.g., a POSIX thread or Windowsthreads) that is fully managed by the host operating system. Once started, threads runindependently until the target function returns. You can query a thread instance to seeif it’s still running:

if t.is_alive():print(‘Still running')else:print(‘Completed')
You can also request to join with a thread, which waits for it to terminate:

> t.join()

The interpreter remains running until all threads terminate. For long-running threadsor background tasks that run forever, you should consider making the thread daemonic.For example:

t = Thread(target=countdown, args=(10,), daemon=True)t.start()

Daemonic threads can’t be joined. However, they are destroyed automatically when themain thread terminates.Beyond the two operations shown, there aren’t many other things you can do withthreads. For example, there are no operations to terminate a thread, signal a thread,adjust its scheduling, or perform any other high-level operations. If you want thesefeatures, you need to build them yourself.If you want to be able to terminate threads, the thread must be programmed to poll forexit at selected points. For example, you might put your thread in a class such as this:

class CountdownTask:def **init**(self):self._running = Truedef terminate(self):self._running = Falsedef run(self, n):while self._running and n > 0:print(‘T-minus', n)n -= 1time.sleep(5)
c = CountdownTask()t = Thread(target=c.run, args=(10,))t.start()...c.terminate() # Signal terminationt.join() # Wait for actual termination (if needed)

Polling for thread termination can be tricky to coordinate if threads perform blockingoperations such as I/O. For example, a thread blocked indefinitely on an I/O operationmay never return to check if it’s been killed. To correctly deal with this case, you’ll needto carefully program thread to utilize timeout loops. For example:

class IOTask:def terminate(self):self._running = Falsedef run(self, sock):

# sock is a socketsock.settimeout(5) # Set timeout periodwhile self._running:

> > # Perform a blocking I/O operation w/ timeouttry:
> > 
> > data = sock.recv(8192)break

except socket.timeout:continue> # Continued processing...

# Terminatedreturn

## 讨论

Due to a global interpreter lock (GIL), Python threads are restricted to an executionmodel that only allows one thread to execute in the interpreter at any given time. Forthis reason, Python threads should generally not be used for computationally intensivetasks where you are trying to achieve parallelism on multiple CPUs. They are muchbetter suited for I/O handling and handling concurrent execution in code that performsblocking operations (e.g., waiting for I/O, waiting for results from a database, etc.).Sometimes you will see threads defined via inheritance from the Thread class. Forexample:

from threading import Thread

class CountdownThread(Thread):def **init**(self, n):super().**init**()self.n = 0def run(self):
while self.n > 0:

> print(‘T-minus', self.n)self.n -= 1time.sleep(5)

c = CountdownThread(5)c.start()

Although this works, it introduces an extra dependency between the code and thethreading library. That is, you can only use the resulting code in the context of threads,whereas the technique shown earlier involves writing code with no explicit dependencyon threading. By freeing your code of such dependencies, it becomes usable in othercontexts that may or may not involve threads. For instance, you might be able to executeyour code in a separate process using the multiprocessing module using code like this:

import multiprocessingc = CountdownTask(5)p = multiprocessing.Process(target=c.run)p.start()...

Again, this only works if the CountdownTask class has been written in a manner that isneutral to the actual means of concurrency (threads, processes, etc.).

# 12.2 判断线程是否已经启动

## 问题

You’ve launched a thread, but want to know when it actually starts running.

## 解决方案

A key feature of threads is that they execute independently and nondeterministically.This can present a tricky synchronization problem if other threads in the program needto know if a thread has reached a certain point in its execution before carrying outfurther operations. To solve such problems, use the Event object from the threadinglibrary.Event instances are similar to a “sticky” flag that allows threads to wait for somethingto happen. Initially, an event is set to 0\. If the event is unset and a thread waits on theevent, it will block (i.e., go to sleep) until the event gets set. A thread that sets the eventwill wake up all of the threads that happen to be waiting (if any). If a thread waits on anevent that has already been set, it merely moves on, continuing to execute.Here is some sample code that uses an Event to coordinate the startup of a thread:

from threading import Thread, Eventimport time

# Code to execute in an independent threaddef countdown(n, started_evt):

> > print(‘countdown starting')started_evt.set()while n > 0:
> > 
> > print(‘T-minus', n)n -= 1time.sleep(5)

# Create the event object that will be used to signal startupstarted_evt = Event()

# Launch the thread and pass the startup eventprint(‘Launching countdown')t = Thread(target=countdown, args=(10,started_evt))t.start()

# Wait for the thread to startstarted_evt.wait()print(‘countdown is running')

When you run this code, the “countdown is running” message will always appear afterthe “countdown starting” message. This is coordinated by the event that makes the mainthread wait until the countdown() function has first printed the startup message.

## 讨论

Event objects are best used for one-time events. That is, you create an event, threadswait for the event to be set, and once set, the Event is discarded. Although it is possibleto clear an event using its clear() method, safely clearing an event and waiting for itto be set again is tricky to coordinate, and can lead to missed events, deadlock, or otherproblems (in particular, you can’t guarantee that a request to clear an event after settingit will execute before a released thread cycles back to wait on the event again).If a thread is going to repeatedly signal an event over and over, you’re probably betteroff using a Condition object instead. For example, this code implements a periodic timerthat other threads can monitor to see whenever the timer expires:

import threadingimport time

class PeriodicTimer:def **init**(self, interval):self._interval = intervalself._flag = 0self._cv = threading.Condition()def start(self):
t = threading.Thread(target=self.run)t.daemon = True

t.start()

def run(self):
‘''Run the timer and notify waiting threads after each interval‘''while True:

> > time.sleep(self._interval)with self._cv:
> > 
> > self._flag ^= 1self._cv.notify_all()

def wait_for_tick(self):
‘''Wait for the next tick of the timer‘''with self._cv:

> > last_flag = self._flagwhile last_flag == self._flag:
> > 
> > self._cv.wait()

# Example use of the timerptimer = PeriodicTimer(5)ptimer.start()

# Two threads that synchronize on the timerdef countdown(nticks):

> while nticks > 0:ptimer.wait_for_tick()print(‘T-minus', nticks)nticks -= 1

def countup(last):
n = 0while n < last:

> ptimer.wait_for_tick()print(‘Counting', n)n += 1

threading.Thread(target=countdown, args=(10,)).start()threading.Thread(target=countup, args=(5,)).start()

A critical feature of Event objects is that they wake all waiting threads. If you are writinga program where you only want to wake up a single waiting thread, it is probably betterto use a Semaphore or Condition object instead.For example, consider this code involving semaphores:

# Worker threaddef worker(n, sema):

> > # Wait to be signaledsema.acquire()
> 
> # Do some workprint(‘Working', n)

# Create some threadssema = threading.Semaphore(0)nworkers = 10for n in range(nworkers):

> t = threading.Thread(target=worker, args=(n, sema,))t.start()

If you run this, a pool of threads will start, but nothing happens because they’re allblocked waiting to acquire the semaphore. Each time the semaphore is released, onlyone worker will wake up and run. For example:

```py
>>> sema.release()
Working 0
>>> sema.release()
Working 1
>>>

```

Writing code that involves a lot of tricky synchronization between threads is likely tomake your head explode. A more sane approach is to thread threads as communicatingtasks using queues or as actors. Queues are described in the next recipe. Actors aredescribed in Recipe 12.10.

# 12.3 线程间的通信

## 问题

You have multiple threads in your program and you want to safely communicate orexchange data between them.

## 解决方案

Perhaps the safest way to send data from one thread to another is to use a Queue fromthe queue library. To do this, you create a Queue instance that is shared by the threads.Threads then use put() or get() operations to add or remove items from the queue.For example:

from queue import Queuefrom threading import Thread

# A thread that produces datadef producer(out_q):

> while True:# Produce some data...out_q.put(data)

# A thread that consumes datadef consumer(in_q):

> while True:# Get some datadata = in_q.get()# Process the data...

# Create the shared queue and launch both threadsq = Queue()t1 = Thread(target=consumer, args=(q,))t2 = Thread(target=producer, args=(q,))t1.start()t2.start()

Queue instances already have all of the required locking, so they can be safely shared byas many threads as you wish.When using queues, it can be somewhat tricky to coordinate the shutdown of the pro‐ducer and consumer. A common solution to this problem is to rely on a special sentinelvalue, which when placed in the queue, causes consumers to terminate. For example:

from queue import Queuefrom threading import Thread

# Object that signals shutdown_sentinel = object()

# A thread that produces datadef producer(out_q):

> while running:# Produce some data...out_q.put(data)> # Put the sentinel on the queue to indicate completionout_q.put(_sentinel)

# A thread that consumes datadef consumer(in_q):

> while True:> # Get some datadata = in_q.get()
> 
> # Check for terminationif data is _sentinel:
> 
> > in_q.put(_sentinel)break
> 
> # Process the data...

A subtle feature of this example is that the consumer, upon receiving the special sentinelvalue, immediately places it back onto the queue. This propagates the sentinel to otherconsumers threads that might be listening on the same queue—thus shutting them alldown one after the other.Although queues are the most common thread communication mechanism, you canbuild your own data structures as long as you add the required locking and synchroni‐zation. The most common way to do this is to wrap your data structures with a conditionvariable. For example, here is how you might build a thread-safe priority queue, asdiscussed in Recipe 1.5.

import heapqimport threading

class PriorityQueue:def **init**(self):self._queue = []self._count = 0self._cv = threading.Condition()def put(self, item, priority):with self._cv:heapq.heappush(self._queue, (-priority, self._count, item))self._count += 1self._cv.notify()def get(self):with self._cv:while len(self._queue) == 0:self._cv.wait()
return heapq.heappop(self._queue)[-1]

Thread communication with a queue is a one-way and nondeterministic process. Ingeneral, there is no way to know when the receiving thread has actually received amessage and worked on it. However, Queue objects do provide some basic completionfeatures, as illustrated by the task_done() and join() methods in this example:

from queue import Queuefrom threading import Thread

# A thread that produces datadef producer(out_q):

> while running:# Produce some data...out_q.put(data)

# A thread that consumes datadef consumer(in_q):

> while True:> # Get some datadata = in_q.get()
> 
> # Process the data...# Indicate completionin_q.task_done()

# Create the shared queue and launch both threadsq = Queue()t1 = Thread(target=consumer, args=(q,))t2 = Thread(target=producer, args=(q,))t1.start()t2.start()

# Wait for all produced items to be consumedq.join()

If a thread needs to know immediately when a consumer thread has processed a par‐ticular item of data, you should pair the sent data with an Event object that allows theproducer to monitor its progress. For example:

from queue import Queuefrom threading import Thread, Event

# A thread that produces datadef producer(out_q):

> while running:# Produce some data...# Make an (data, event) pair and hand it to the consumerevt = Event()out_q.put((data, evt))...# Wait for the consumer to process the itemevt.wait()

# A thread that consumes datadef consumer(in_q):

> while True:# Get some datadata, evt = in_q.get()# Process the data...# Indicate completionevt.set()

## 讨论

Writing threaded programs based on simple queuing is often a good way to maintainsanity. If you can break everything down to simple thread-safe queuing, you’ll find thatyou don’t need to litter your program with locks and other low-level synchronization.Also, communicating with queues often leads to designs that can be scaled up to otherkinds of message-based communication patterns later on. For instance, you might be

able to split your program into multiple processes, or even a distributed system, withoutchanging much of its underlying queuing architecture.One caution with thread queues is that putting an item in a queue doesn’t make a copyof the item. Thus, communication actually involves passing an object reference betweenthreads. If you are concerned about shared state, it may make sense to only pass im‐mutable data structures (e.g., integers, strings, or tuples) or to make deep copies of thequeued items. For example:from queue import Queuefrom threading import Threadimport copy

# A thread that produces datadef producer(out_q):

> while True:# Produce some data...out_q.put(copy.deepcopy(data))

# A thread that consumes datadef consumer(in_q):

> while True:# Get some datadata = in_q.get()# Process the data...

Queue objects provide a few additional features that may prove to be useful in certaincontexts. If you create a Queue with an optional size, such as Queue(N), it places a limiton the number of items that can be enqueued before the put() blocks the producer.Adding an upper bound to a queue might make sense if there is mismatch in speedbetween a producer and consumer. For instance, if a producer is generating items at amuch faster rate than they can be consumed. On the other hand, making a queue blockwhen it’s full can also have an unintended cascading effect throughout your program,possibly causing it to deadlock or run poorly. In general, the problem of “flow control”between communicating threads is a much harder problem than it seems. If you everfind yourself trying to fix a problem by fiddling with queue sizes, it could be an indicatorof a fragile design or some other inherent scaling problem.Both the get() and put() methods support nonblocking and timeouts. For example:

import queueq = queue.Queue()

try:data = q.get(block=False)except queue.Empty:...try:q.put(item, block=False)except queue.Full:...try:data = q.get(timeout=5.0)except queue.Empty:...
Both of these options can be used to avoid the problem of just blocking indefinitely ona particular queuing operation. For example, a nonblocking put() could be used witha fixed-sized queue to implement different kinds of handling code for when a queue isfull. For example, issuing a log message and discarding:

def producer(q):
...try:

> q.put(item, block=False)

except queue.Full:log.warning(‘queued item %r discarded!', item)
A timeout is useful if you’re trying to make consumer threads periodically give up onoperations such as q.get() so that they can check things such as a termination flag, asdescribed in Recipe 12.1.

_running = True

def consumer(q):while _running:try:item = q.get(timeout=5.0)# Process item...except queue.Empty:pass
Lastly, there are utility methods q.qsize(), q.full(), q.empty() that can tell you thecurrent size and status of the queue. However, be aware that all of these are unreliablein a multithreaded environment. For example, a call to q.empty() might tell you thatthe queue is empty, but in the time that has elapsed since making the call, another threadcould have added an item to the queue. Frankly, it’s best to write your code not to relyon such functions.

# 12.4 给关键部分加锁

## 问题

Your program uses threads and you want to lock critical sections of code to avoid raceconditions.

## 解决方案

To make mutable objects safe to use by multiple threads, use Lock objects in the threading library, as shown here:

import threading

class SharedCounter:
‘''A counter object that can be shared by multiple threads.‘''def **init**(self, initial_value = 0):

> self._value = initial_valueself._value_lock = threading.Lock()

def incr(self,delta=1):
‘''Increment the counter with locking‘''with self._value_lock:

> self._value += delta

def decr(self,delta=1):
‘''Decrement the counter with locking‘''with self._value_lock:

> self._value -= delta

A Lock guarantees mutual exclusion when used with the with statement—that is, onlyone thread is allowed to execute the block of statements under the with statement at atime. The with statement acquires the lock for the duration of the indented statementsand releases the lock when control flow exits the indented block.

## 讨论

Thread scheduling is inherently nondeterministic. Because of this, failure to use locksin threaded programs can result in randomly corrupted data and bizarre behaviorknown as a “race condition.” To avoid this, locks should always be used whenever sharedmutable state is accessed by multiple threads.

In older Python code, it is common to see locks explicitly acquired and released. Forexample, in this variant of the last example:

import threading

class SharedCounter:
‘''A counter object that can be shared by multiple threads.‘''def **init**(self, initial_value = 0):

> self._value = initial_valueself._value_lock = threading.Lock()

def incr(self,delta=1):‘''Increment the counter with locking‘''self._value_lock.acquire()self._value += deltaself._value_lock.release()def decr(self,delta=1):‘''Decrement the counter with locking‘''self._value_lock.acquire()self._value -= deltaself._value_lock.release()
The with statement is more elegant and less prone to error—especially in situationswhere a programmer might forget to call the release() method or if a program happensto raise an exception while holding a lock (the with statement guarantees that locks arealways released in both cases).To avoid the potential for deadlock, programs that use locks should be written in a waysuch that each thread is only allowed to acquire one lock at a time. If this is not possible,you may need to introduce more advanced deadlock avoidance into your program, asdescribed in Recipe 12.5.In the threading library, you’ll find other synchronization primitives, such as RLockand Semaphore objects. As a general rule of thumb, these are more special purpose andshould not be used for simple locking of mutable state. An RLock or re-entrant lockobject is a lock that can be acquired multiple times by the same thread. It is primarilyused to implement code based locking or synchronization based on a construct knownas a “monitor.” With this kind of locking, only one thread is allowed to use an entirefunction or the methods of a class while the lock is held. For example, you could im‐plement the SharedCounter class like this:

import threading

class SharedCounter:
‘''A counter object that can be shared by multiple threads.‘''_lock = threading.RLock()def **init**(self, initial_value = 0):

> self._value = initial_value

def incr(self,delta=1):
‘''Increment the counter with locking‘''with SharedCounter._lock:

> self._value += delta

def decr(self,delta=1):
‘''Decrement the counter with locking‘''with SharedCounter._lock:

> self.incr(-delta)

In this variant of the code, there is just a single class-level lock shared by all instancesof the class. Instead of the lock being tied to the per-instance mutable state, the lock ismeant to synchronize the methods of the class. Specifically, this lock ensures that onlyone thread is allowed to be using the methods of the class at once. However, unlike astandard lock, it is OK for methods that already have the lock to call other methods thatalso use the lock (e.g., see the decr() method).One feature of this implementation is that only one lock is created, regardless of howmany counter instances are created. Thus, it is much more memory-efficient in situa‐tions where there are a large number of counters. However, a possible downside is thatit may cause more lock contention in programs that use a large number of threads andmake frequent counter updates.A Semaphore object is a synchronization primitive based on a shared counter. If thecounter is nonzero, the with statement decrements the count and a thread is allowed toproceed. The counter is incremented upon the conclusion of the with block. If thecounter is zero, progress is blocked until the counter is incremented by another thread.Although a semaphore can be used in the same manner as a standard Lock, the addedcomplexity in implementation negatively impacts performance. Instead of simple lock‐ing, Semaphore objects are more useful for applications involving signaling betweenthreads or throttling. For example, if you want to limit the amount of concurrency in apart of code, you might use a semaphore, as follows:

from threading import Semaphoreimport urllib.request

# At most, five threads allowed to run at once_fetch_url_sema = Semaphore(5)

def fetch_url(url):with _fetch_url_sema:return urllib.request.urlopen(url)
If you’re interested in the underlying theory and implementation of thread synchroni‐zation primitives, consult almost any textbook on operating systems.

# 12.5 防止死锁的加锁机制

## 问题

You’re writing a multithreaded program where threads need to acquire more than onelock at a time while avoiding deadlock.

## 解决方案

In multithreaded programs, a common source of deadlock is due to threads that attemptto acquire multiple locks at once. For instance, if a thread acquires the first lock, butthen blocks trying to acquire the second lock, that thread can potentially block theprogress of other threads and make the program freeze.One solution to deadlock avoidance is to assign each lock in the program a uniquenumber, and to enforce an ordering rule that only allows multiple locks to be acquiredin ascending order. This is surprisingly easy to implement using a context manager asfollows:

import threadingfrom contextlib import contextmanager

# Thread-local state to stored information on locks already acquired_local = threading.local()

@contextmanagerdef acquire(*locks):

> > # Sort locks by object identifierlocks = sorted(locks, key=lambda x: id(x))
> 
> # Make sure lock order of previously acquired locks is not violatedacquired = getattr(_local,'acquired',[])if acquired and max(id(lock) for lock in acquired) >= id(locks[0]):
> 
> > raise RuntimeError(‘Lock Order Violation')
> 
> # Acquire all of the locksacquired.extend(locks)_local.acquired = acquired

try:for lock in locks:lock.acquire()> yield

finally:> # Release locks in reverse order of acquisitionfor lock in reversed(locks):

> > lock.release()
> 
> del acquired[-len(locks):]

To use this context manager, you simply allocate lock objects in the normal way, but usethe acquire() function whenever you want to work with one or more locks. Forexample:

import threadingx_lock = threading.Lock()y_lock = threading.Lock()

def thread_1():while True:with acquire(x_lock, y_lock):print(‘Thread-1')def thread_2():while True:with acquire(y_lock, x_lock):print(‘Thread-2')
t1 = threading.Thread(target=thread_1)t1.daemon = Truet1.start()

t2 = threading.Thread(target=thread_2)t2.daemon = Truet2.start()

If you run this program, you’ll find that it happily runs forever without deadlock—eventhough the acquisition of locks is specified in a different order in each function.The key to this recipe lies in the first statement that sorts the locks according to objectidentifier. By sorting the locks, they always get acquired in a consistent order regardlessof how the user might have provided them to acquire().The solution uses thread-local storage to solve a subtle problem with detecting potentialdeadlock if multiple acquire() operations are nested. For example, suppose you wrotethe code like this:

import threadingx_lock = threading.Lock()y_lock = threading.Lock()

def thread_1():

> while True:with acquire(x_lock):with acquire(y_lock):print(‘Thread-1')

def thread_2():while True:with acquire(y_lock):with acquire(x_lock):print(‘Thread-2')
t1 = threading.Thread(target=thread_1)t1.daemon = Truet1.start()

t2 = threading.Thread(target=thread_2)t2.daemon = Truet2.start()

If you run this version of the program, one of the threads will crash with an exceptionsuch as this:

Exception in thread Thread-1:Traceback (most recent call last):

> File “/usr/local/lib/python3.3/threading.py”, line 639, in _bootstrap_innerself.run()File “/usr/local/lib/python3.3/threading.py”, line 596, in runself._target(*self._args, **self._kwargs)File “deadlock.py”, line 49, in thread_1with acquire(y_lock):File “/usr/local/lib/python3.3/contextlib.py”, line 48, in **enter**return next(self.gen)File “deadlock.py”, line 15, in acquireraise RuntimeError(“Lock Order Violation”)

RuntimeError: Lock Order Violation>>>

This crash is caused by the fact that each thread remembers the locks it has alreadyacquired. The acquire() function checks the list of previously acquired locks and en‐forces the ordering constraint that previously acquired locks must have an object IDthat is less than the new locks being acquired.

## 讨论

The issue of deadlock is a well-known problem with programs involving threads (aswell as a common subject in textbooks on operating systems). As a rule of thumb, aslong as you can ensure that threads can hold only one lock at a time, your program willbe deadlock free. However, once multiple locks are being acquired at the same time, allbets are off.

Detecting and recovering from deadlock is an extremely tricky problem with few elegantsolutions. For example, a common deadlock detection and recovery scheme involvesthe use of a watchdog timer. As threads run, they periodically reset the timer, and aslong as everything is running smoothly, all is well. However, if the program deadlocks,the watchdog timer will eventually expire. At that point, the program “recovers” bykilling and then restarting itself.Deadlock avoidance is a different strategy where locking operations are carried out ina manner that simply does not allow the program to enter a deadlocked state. Thesolution in which locks are always acquired in strict order of ascending object ID canbe mathematically proven to avoid deadlock, although the proof is left as an exercise tothe reader (the gist of it is that by acquiring locks in a purely increasing order, you can’tget cyclic locking dependencies, which are a necessary condition for deadlock to occur).As a final example, a classic thread deadlock problem is the so-called “dining philoso‐pher’s problem.” In this problem, five philosophers sit around a table on which thereare five bowls of rice and five chopsticks. Each philosopher represents an independentthread and each chopstick represents a lock. In the problem, philosophers either sit andthink or they eat rice. However, in order to eat rice, a philosopher needs two chopsticks.Unfortunately, if all of the philosophers reach over and grab the chopstick to their left,they’ll all just sit there with one stick and eventually starve to death. It’s a gruesomescene.Using the solution, here is a simple deadlock free implementation of the dining philos‐opher’s problem:

import threading

# The philosopher threaddef philosopher(left, right):

> while True:with acquire(left,right):print(threading.currentThread(), ‘eating')

# The chopsticks (represented by locks)NSTICKS = 5chopsticks = [threading.Lock() for n in range(NSTICKS)]

# Create all of the philosophersfor n in range(NSTICKS):

> t = threading.Thread(target=philosopher,args=(chopsticks[n],chopsticks[(n+1) % NSTICKS]))> t.start()

Last, but not least, it should be noted that in order to avoid deadlock, all locking oper‐ations must be carried out using our acquire() function. If some fragment of codedecided to acquire a lock directly, then the deadlock avoidance algorithm wouldn’t work.

# 12.6 保存线程的状态信息

## 问题

You need to store state that’s specific to the currently executing thread and not visibleto other threads.

## 解决方案

Sometimes in multithreaded programs, you need to store data that is only specific tothe currently executing thread. To do this, create a thread-local storage object usingthreading.local(). Attributes stored and read on this object are only visible to theexecuting thread and no others.As an interesting practical example of using thread-local storage, consider the LazyConnection context-manager class that was first defined in Recipe 8.3\. Here is a slightlymodified version that safely works with multiple threads:

from socket import socket, AF_INET, SOCK_STREAMimport threading

class LazyConnection:def **init**(self, address, family=AF_INET, type=SOCK_STREAM):self.address = addressself.family = AF_INETself.type = SOCK_STREAMself.local = threading.local()def **enter**(self):if hasattr(self.local, ‘sock'):raise RuntimeError(‘Already connected')
self.local.sock = socket(self.family, self.type)self.local.sock.connect(self.address)return self.local.sock

def **exit**(self, exc_ty, exc_val, tb):self.local.sock.close()del self.local.sock
In this code, carefully observe the use of the self.local attribute. It is initialized as aninstance of threading.local(). The other methods then manipulate a socket that’sstored as self.local.sock. This is enough to make it possible to safely use an instanceof LazyConnection in multiple threads. For example:

from functools import partialdef test(conn):

> with conn as s:> s.send(b'GET /index.html HTTP/1.0rn')s.send(b'Host: www.python.orgrn')
> 
> s.send(b'rn')resp = b'‘.join(iter(partial(s.recv, 8192), b'‘))
> 
> print(‘Got {} bytes'.format(len(resp)))

if **name** == ‘**main**':
conn = LazyConnection((‘www.python.org', 80))

t1 = threading.Thread(target=test, args=(conn,))t2 = threading.Thread(target=test, args=(conn,))t1.start()t2.start()t1.join()t2.join()

The reason it works is that each thread actually creates its own dedicated socket con‐nection (stored as self.local.sock). Thus, when the different threads perform socketoperations, they don’t interfere with one another as they are being performed on dif‐ferent sockets.

## 讨论

Creating and manipulating thread-specific state is not a problem that often arises inmost programs. However, when it does, it commonly involves situations where an objectbeing used by multiple threads needs to manipulate some kind of dedicated systemresource, such as a socket or file. You can’t just have a single socket object shared byeveryone because chaos would ensue if multiple threads ever started reading and writingon it at the same time. Thread-local storage fixes this by making such resources onlyvisible in the thread where they’re being used.In this recipe, the use of threading.local() makes the LazyConnection class supportone connection per thread, as opposed to one connection for the entire process. It’s asubtle but interesting distinction.Under the covers, an instance of threading.local() maintains a separate instancedictionary for each thread. All of the usual instance operations of getting, setting, anddeleting values just manipulate the per-thread dictionary. The fact that each thread usesa separate dictionary is what provides the isolation of data.

# 12.7 创建一个线程池

## 问题

You want to create a pool of worker threads for serving clients or performing other kindsof work.

## 解决方案

The concurrent.futures library has a ThreadPoolExecutor class that can be used forthis purpose. Here is an example of a simple TCP server that uses a thread-pool to serveclients:

from socket import AF_INET, SOCK_STREAM, socketfrom concurrent.futures import ThreadPoolExecutor

def echo_client(sock, client_addr):
‘''Handle a client connection‘''print(‘Got connection from', client_addr)while True:

> > msg = sock.recv(65536)if not msg:
> > 
> > break
> 
> sock.sendall(msg)

print(‘Client closed connection')sock.close()

def echo_server(addr):
pool = ThreadPoolExecutor(128)sock = socket(AF_INET, SOCK_STREAM)sock.bind(addr)sock.listen(5)while True:

> client_sock, client_addr = sock.accept()pool.submit(echo_client, client_sock, client_addr)

echo_server((‘',15000))

If you want to manually create your own thread pool, it’s usually easy enough to do itusing a Queue. Here is a slightly different, but manual implementation of the same code:

from socket import socket, AF_INET, SOCK_STREAMfrom threading import Threadfrom queue import Queue

def echo_client(q):
‘''Handle a client connection‘''sock, client_addr = q.get()print(‘Got connection from', client_addr)while True:

> > msg = sock.recv(65536)if not msg:
> > 
> > break
> 
> sock.sendall(msg)

print(‘Client closed connection')

sock.close()

def echo_server(addr, nworkers):

# Launch the client workersq = Queue()for n in range(nworkers):

> t = Thread(target=echo_client, args=(q,))t.daemon = Truet.start()

# Run the serversock = socket(AF_INET, SOCK_STREAM)sock.bind(addr)sock.listen(5)while True:

> client_sock, client_addr = sock.accept()q.put((client_sock, client_addr))

echo_server((‘',15000), 128)

One advantage of using ThreadPoolExecutor over a manual implementation is that itmakes it easier for the submitter to receive results from the called function. For example,you could write code like this:

from concurrent.futures import ThreadPoolExecutorimport urllib.request

def fetch_url(url):u = urllib.request.urlopen(url)data = u.read()return data
pool = ThreadPoolExecutor(10)# Submit work to the poola = pool.submit(fetch_url, ‘[`www.python.org`](http://www.python.org)‘)b = pool.submit(fetch_url, ‘[`www.pypy.org`](http://www.pypy.org)‘)

# Get the results backx = a.result()y = b.result()

The result objects in the example handle all of the blocking and coordination neededto get data back from the worker thread. Specifically, the operation a.result() blocksuntil the corresponding function has been executed by the pool and returned a value.

## 讨论

Generally, you should avoid writing programs that allow unlimited growth in the num‐ber of threads. For example, take a look at the following server:

from threading import Threadfrom socket import socket, AF_INET, SOCK_STREAM

def echo_client(sock, client_addr):
‘''Handle a client connection‘''print(‘Got connection from', client_addr)while True:

> > msg = sock.recv(65536)if not msg:
> > 
> > break
> 
> sock.sendall(msg)

print(‘Client closed connection')sock.close()

def echo_server(addr, nworkers):

# Run the serversock = socket(AF_INET, SOCK_STREAM)sock.bind(addr)sock.listen(5)while True:

> client_sock, client_addr = sock.accept()t = Thread(target=echo_client, args=(client_sock, client_addr))t.daemon = Truet.start()

echo_server((‘',15000))

Although this works, it doesn’t prevent some asynchronous hipster from launching anattack on the server that makes it create so many threads that your program runs outof resources and crashes (thus further demonstrating the “evils” of using threads). Byusing a pre-initialized thread pool, you can carefully put an upper limit on the amountof supported concurrency.You might be concerned with the effect of creating a large number of threads. However,modern systems should have no trouble creating pools of a few thousand threads.Moreover, having a thousand threads just sitting around waiting for work isn’t going tohave much, if any, impact on the performance of other code (a sleeping thread does justthat—nothing at all). Of course, if all of those threads wake up at the same time andstart hammering on the CPU, that’s a different story—especially in light of the GlobalInterpreter Lock (GIL). Generally, you only want to use thread pools for I/O-boundprocessing.One possible concern with creating large thread pools might be memory use. For ex‐ample, if you create 2,000 threads on OS X, the system shows the Python process usingup more than 9 GB of virtual memory. However, this is actually somewhat misleading.When creating a thread, the operating system reserves a region of virtual memory tohold the thread’s execution stack (often as large as 8 MB). Only a small fragment of thismemory is actually mapped to real memory, though. Thus, if you look a bit closer, youmight find the Python process is using far less real memory (e.g., for 2,000 threads, only

70 MB of real memory is used, not 9 GB). If the size of the virtual memory is a concern,you can dial it down using the threading.stack_size() function. For example:

import threadingthreading.stack_size(65536)

If you add this call and repeat the experiment of creating 2,000 threads, you’ll find thatthe Python process is now only using about 210 MB of virtual memory, although theamount of real memory in use remains about the same. Note that the thread stack sizemust be at least 32,768 bytes, and is usually restricted to be a multiple of the systemmemory page size (4096, 8192, etc.).

# 12.8 简单的并行编程

## 问题

You have a program that performs a lot of CPU-intensive work, and you want to makeit run faster by having it take advantage of multiple CPUs.

## 解决方案

The concurrent.futures library provides a ProcessPoolExecutor class that can beused to execute computationally intensive functions in a separately running instance ofthe Python interpreter. However, in order to use it, you first need to have some com‐putationally intensive work. Let’s illustrate with a simple yet practical example.Suppose you have an entire directory of gzip-compressed Apache web server logs:

logs/20120701.log.gz20120702.log.gz20120703.log.gz20120704.log.gz20120705.log.gz20120706.log.gz...
Further suppose each log file contains lines like this:

124.115.6.12 - - [10/Jul/2012:00:18:50 -0500] “GET /robots.txt ...” 200 71210.212.209.67 - - [10/Jul/2012:00:18:51 -0500] “GET /ply/ ...” 200 11875210.212.209.67 - - [10/Jul/2012:00:18:51 -0500] “GET /favicon.ico ...” 404 36961.135.216.105 - - [10/Jul/2012:00:20:04 -0500] “GET /blog/atom.xml ...” 304 -...

Here is a simple script that takes this data and identifies all hosts that have accessed therobots.txt file:

# findrobots.py

import gzipimport ioimport glob

def find_robots(filename):
‘''Find all of the hosts that access robots.txt in a single log file‘''robots = set()with gzip.open(filename) as f:

> for line in io.TextIOWrapper(f,encoding='ascii'):> fields = line.split()if fields[6] == ‘/robots.txt':
> 
> > robots.add(fields[0])

return robots

def find_all_robots(logdir):
‘''Find all hosts across and entire sequence of files‘''files = glob.glob(logdir+'/*.log.gz')all_robots = set()for robots in map(find_robots, files):

> all_robots.update(robots)

return all_robots

if **name** == ‘**main**':
robots = find_all_robots(‘logs')for ipaddr in robots:

> print(ipaddr)

The preceding program is written in the commonly used map-reduce style. The functionfind_robots() is mapped across a collection of filenames and the results are combinedinto a single result (the all_robots set in the find_all_robots() function).Now, suppose you want to modify this program to use multiple CPUs. It turns out tobe easy—simply replace the map() operation with a similar operation carried out on aprocess pool from the concurrent.futures library. Here is a slightly modified versionof the code:

# findrobots.py

import gzipimport ioimport globfrom concurrent import futures

def find_robots(filename):
‘''Find all of the hosts that access robots.txt in a single log file

‘''robots = set()with gzip.open(filename) as f:

> for line in io.TextIOWrapper(f,encoding='ascii'):> fields = line.split()if fields[6] == ‘/robots.txt':
> 
> > robots.add(fields[0])

return robots

def find_all_robots(logdir):
‘''Find all hosts across and entire sequence of files‘''files = glob.glob(logdir+'/*.log.gz')all_robots = set()with futures.ProcessPoolExecutor() as pool:

> for robots in pool.map(find_robots, files):all_robots.update(robots)

return all_robots

if **name** == ‘**main**':
robots = find_all_robots(‘logs')for ipaddr in robots:

> print(ipaddr)

With this modification, the script produces the same result but runs about 3.5 timesfaster on our quad-core machine. The actual performance will vary according to thenumber of CPUs available on your machine.

## 讨论

Typical usage of a ProcessPoolExecutor is as follows:from concurrent.futures import ProcessPoolExecutor

with ProcessPoolExecutor() as pool:...do work in parallel using pool...
Under the covers, a ProcessPoolExecutor creates N independent running Python in‐terpreters where N is the number of available CPUs detected on the system. You canchange the number of processes created by supplying an optional argument to ProcessPoolExecutor(N). The pool runs until the last statement in the with block is executed,at which point the process pool is shut down. However, the program will wait until allsubmitted work has been processed.Work to be submitted to a pool must be defined in a function. There are two methodsfor submission. If you are are trying to parallelize a list comprehension or a map()operation, you use pool.map():

# A function that performs a lot of workdef work(x):

> ...return result

# Nonparallel coderesults = map(work, data)

# Parallel implementationwith ProcessPoolExecutor() as pool:

> results = pool.map(work, data)

Alternatively, you can manually submit single tasks using the pool.submit() method:

# Some functiondef work(x):

> ...return result

with ProcessPoolExecutor() as pool:
...# Example of submitting work to the poolfuture_result = pool.submit(work, arg)

# Obtaining the result (blocks until done)r = future_result.result()...

If you manually submit a job, the result is an instance of Future. To obtain the actualresult, you call its result() method. This blocks until the result is computed and re‐turned by the pool.Instead of blocking, you can also arrange to have a callback function triggered uponcompletion instead. For example:

def when_done(r):print(‘Got:', r.result())with ProcessPoolExecutor() as pool:future_result = pool.submit(work, arg)future_result.add_done_callback(when_done)
The user-supplied callback function receives an instance of Future that must be usedto obtain the actual result (i.e., by calling its result() method).Although process pools can be easy to use, there are a number of important consider‐ations to be made in designing larger programs. In no particular order:

*   This technique for parallelization only works well for problems that can be trivially

decomposed into independent parts.

*   Work must be submitted in the form of simple functions. Parallel execution of

instance methods, closures, or other kinds of constructs are not supported.

*   Function arguments and return values must be compatible with pickle. Work is

carried out in a separate interpreter using interprocess communication. Thus, dataexchanged between interpreters has to be serialized.

*   Functions submitted for work should not maintain persistent state or have side

effects. With the exception of simple things such as logging, you don’t really haveany control over the behavior of child processes once started. Thus, to preserve yoursanity, it is probably best to keep things simple and carry out work in pure-functionsthat don’t alter their environment.

*   Process pools are created by calling the fork() system call on Unix. This makes a

clone of the Python interpreter, including all of the program state at the time of thefork. On Windows, an independent copy of the interpreter that does not clone stateis launched. The actual forking process does not occur until the first pool.map()or pool.submit() method is called.

*   Great care should be made when combining process pools and programs that use

threads. In particular, you should probably create and launch process pools priorto the creation of any threads (e.g., create the pool in the main thread at programstartup).

# 12.9 Python 的全局锁问题

## 问题

You’ve heard about the Global Interpreter Lock (GIL), and are worried that it might beaffecting the performance of your multithreaded program.

## 解决方案

Although Python fully supports thread programming, parts of the C implementationof the interpreter are not entirely thread safe to a level of allowing fully concurrentexecution. In fact, the interpreter is protected by a so-called Global Interpreter Lock(GIL) that only allows one Python thread to execute at any given time. The most no‐ticeable effect of the GIL is that multithreaded Python programs are not able to fullytake advantage of multiple CPU cores (e.g., a computationally intensive applicationusing more than one thread only runs on a single CPU).

Before discussing common GIL workarounds, it is important to emphasize that the GILtends to only affect programs that are heavily CPU bound (i.e., dominated by compu‐tation). If your program is mostly doing I/O, such as network communication, threadsare often a sensible choice because they’re mostly going to spend their time sittingaround waiting. In fact, you can create thousands of Python threads with barely a con‐cern. Modern operating systems have no trouble running with that many threads, soit’s simply not something you should worry much about.For CPU-bound programs, you really need to study the nature of the computation beingperformed. For instance, careful choice of the underlying algorithm may produce a fargreater speedup than trying to parallelize an unoptimal algorithm with threads. Simi‐larly, given that Python is interpreted, you might get a far greater speedup simply bymoving performance-critical code into a C extension module. Extensions such asNumPy are also highly effective at speeding up certain kinds of calculations involvingarray data. Last, but not least, you might investigate alternative implementations, suchas PyPy, which features optimizations such as a JIT compiler (although, as of this writing,it does not yet support Python 3).It’s also worth noting that threads are not necessarily used exclusively for performance.A CPU-bound program might be using threads to manage a graphical user interface, anetwork connection, or provide some other kind of service. In this case, the GIL canactually present more of a problem, since code that holds it for an excessively long periodwill cause annoying stalls in the non-CPU-bound threads. In fact, a poorly written Cextension can actually make this problem worse, even though the computation part ofthe code might run faster than before.Having said all of this, there are two common strategies for working around the limi‐tations of the GIL. First, if you are working entirely in Python, you can use the multiprocessing module to create a process pool and use it like a co-processor. For example,suppose you have the following thread code:

# Performs a large calculation (CPU bound)def some_work(args):

> ...return result

# A thread that calls the above functiondef some_thread():

> while True:...r = some_work(args)...

Here’s how you would modify the code to use a pool:

# Processing pool (see below for initiazation)pool = None

# Performs a large calculation (CPU bound)def some_work(args):

> ...return result

# A thread that calls the above functiondef some_thread():

> while True:...r = pool.apply(some_work, (args))...

# Initiaze the poolif **name** == ‘**main**':

> import multiprocessingpool = multiprocessing.Pool()

This example with a pool works around the GIL using a neat trick. Whenever a threadwants to perform CPU-intensive work, it hands the work to the pool. The pool, in turn,hands the work to a separate Python interpreter running in a different process. Whilethe thread is waiting for the result, it releases the GIL. Moreover, because the calculationis being performed in a separate interpreter, it’s no longer bound by the restrictions ofthe GIL. On a multicore system, you’ll find that this technique easily allows you to takeadvantage of all the CPUs.The second strategy for working around the GIL is to focus on C extension program‐ming. The general idea is to move computationally intensive tasks to C, independent ofPython, and have the C code release the GIL while it’s working. This is done by insertingspecial macros into the C code like this:

# include “Python.h”...

PyObject *pyfunc(PyObject *self, PyObject *args) {...Py_BEGIN_ALLOW_THREADS// Threaded C code...Py_END_ALLOW_THREADS...
}

If you are using other tools to access C, such as the ctypes library or Cython, you maynot need to do anything. For example, ctypes releases the GIL when calling into C bydefault.

## 讨论

Many programmers, when faced with thread performance problems, are quick to blamethe GIL for all of their ills. However, doing so is shortsighted and naive. Just as a real-

world example, mysterious “stalls” in a multithreaded network program might be causedby something entirely different (e.g., a stalled DNS lookup) rather than anything relatedto the GIL. The bottom line is that you really need to study your code to know if theGIL is an issue or not. Again, realize that the GIL is mostly concerned with CPU-boundprocessing, not I/O.If you are going to use a process pool as a workaround, be aware that doing so involvesdata serialization and communication with a different Python interpreter. For this towork, the operation to be performed needs to be contained within a Python functiondefined by the def statement (i.e., no lambdas, closures, callable instances, etc.), and thefunction arguments and return value must be compatible with pickle. Also, the amountof work to be performed must be sufficiently large to make up for the extra communi‐cation overhead.Another subtle aspect of pools is that mixing threads and process pools together can bea good way to make your head explode. If you are going to use both of these featurestogether, it is often best to create the process pool as a singleton at program startup,prior to the creation of any threads. Threads will then use the same process pool for allof their computationally intensive work.For C extensions, the most important feature is maintaining isolation from the Pythoninterpreter process. That is, if you’re going to offload work from Python to C, you needto make sure the C code operates independently of Python. This means using no Pythondata structures and making no calls to Python’s C API. Another consideration is thatyou want to make sure your C extension does enough work to make it all worthwhile.That is, it’s much better if the extension can perform millions of calculations as opposedto just a few small calculations.Needless to say, these solutions to working around the GIL don’t apply to all possibleproblems. For instance, certain kinds of applications don’t work well if separated intomultiple processes, nor may you want to code parts in C. For these kinds of applications,you may have to come up with your own solution (e.g., multiple processes accessingshared memory regions, multiple interpreters running in the same process, etc.). Al‐ternatively, you might look at some other implementations of the interpreter, such asPyPy.See Recipes 15.7 and 15.10 for additional information on releasing the GIL in Cextensions.

# 12.10 定义一个 Actor 任务

## 问题

You’d like to define tasks with behavior similar to “actors” in the so-called “actor model.”

## 解决方案

The “actor model” is one of the oldest and most simple approaches to concurrency anddistributed computing. In fact, its underlying simplicity is part of its appeal. In a nutshell,an actor is a concurrently executing task that simply acts upon messages sent to it. Inresponse to these messages, it may decide to send further messages to other actors.Communication with actors is one way and asynchronous. Thus, the sender of a messagedoes not know when a message actually gets delivered, nor does it receive a responseor acknowledgment that the message has been processed.Actors are straightforward to define using a combination of a thread and a queue. Forexample:

from queue import Queuefrom threading import Thread, Event

# Sentinel used for shutdownclass ActorExit(Exception):

> pass

class Actor:def **init**(self):self._mailbox = Queue()def send(self, msg):‘''Send a message to the actor‘''self._mailbox.put(msg)def recv(self):
‘''Receive an incoming message‘''msg = self._mailbox.get()if msg is ActorExit:

> raise ActorExit()

return msg

def close(self):‘''Close the actor, thus shutting it down‘''self.send(ActorExit)def start(self):
‘''Start concurrent execution‘''self._terminated = Event()t = Thread(target=self._bootstrap)

t.daemon = Truet.start()

def _bootstrap(self):try:self.run()except ActorExit:passfinally:self._terminated.set()def join(self):self._terminated.wait()def run(self):
‘''Run method to be implemented by the user‘''while True:

> msg = self.recv()

# Sample ActorTaskclass PrintActor(Actor):

> def run(self):while True:msg = self.recv()print(‘Got:', msg)

# Sample usep = PrintActor()p.start()p.send(‘Hello')p.send(‘World')p.close()p.join()

In this example, Actor instances are things that you simply send a message to usingtheir send() method. Under the covers, this places the message on a queue and handsit off to an internal thread that runs to process the received messages. The close()method is programmed to shut down the actor by placing a special sentinel value(ActorExit) on the queue. Users define new actors by inheriting from Actor and re‐defining the run() method to implement their custom processing. The usage of theActorExit exception is such that user-defined code can be programmed to catch thetermination request and handle it if appropriate (the exception is raised by the get()method and propagated).If you relax the requirement of concurrent and asynchronous message delivery, actor-like objects can also be minimally defined by generators. For example:

def print_actor():
while True:

> try:msg = yield # Get a messageprint(‘Got:', msg)except GeneratorExit:print(‘Actor terminating')

# Sample usep = print_actor()next(p) # Advance to the yield (ready to receive)p.send(‘Hello')p.send(‘World')p.close()

## 讨论

Part of the appeal of actors is their underlying simplicity. In practice, there is just onecore operation, send(). Plus, the general concept of a “message” in actor-based systemsis something that can be expanded in many different directions. For example, you couldpass tagged messages in the form of tuples and have actors take different courses ofaction like this:

class TaggedActor(Actor):def run(self):while True:tag, *payload = self.recv()getattr(self,'do_‘+tag)(*payload)

# Methods correponding to different message tagsdef do_A(self, x):

> print(‘Running A', x)

def do_B(self, x, y):print(‘Running B', x, y)

# Examplea = TaggedActor()a.start()a.send((‘A', 1)) # Invokes do_A(1)a.send((‘B', 2, 3)) # Invokes do_B(2,3)

As another example, here is a variation of an actor that allows arbitrary functions to beexecuted in a worker and results to be communicated back using a special Result object:

from threading import Eventclass Result:

> def **init**(self):self._evt = Event()self._result = Nonedef set_result(self, value):> self._result = value
> 
> self._evt.set()

def result(self):self._evt.wait()return self._result

class Worker(Actor):def submit(self, func, *args, **kwargs):r = Result()self.send((func, args, kwargs, r))return rdef run(self):while True:func, args, kwargs, r = self.recv()r.set_result(func(*args, **kwargs))

# Example useworker = Worker()worker.start()r = worker.submit(pow, 2, 3)print(r.result())

Last, but not least, the concept of “sending” a task a message is something that can bescaled up into systems involving multiple processes or even large distributed systems.For example, the send() method of an actor-like object could be programmed to trans‐mit data on a socket connection or deliver it via some kind of messaging infrastructure(e.g., AMQP, ZMQ, etc.).

# 12.11 实现消息发布/订阅模型

## 问题

You have a program based on communicating threads and want them to implementpublish/subscribe messaging.

## 解决方案

To implement publish/subscribe messaging, you typically introduce a separate “ex‐change” or “gateway” object that acts as an intermediary for all messages. That is, insteadof directly sending a message from one task to another, a message is sent to the exchangeand it delivers it to one or more attached tasks. Here is one example of a very simpleexchange implementation:

from collections import defaultdict

class Exchange:def **init**(self):self._subscribers = set()def attach(self, task):self._subscribers.add(task)def detach(self, task):self._subscribers.remove(task)def send(self, msg):for subscriber in self._subscribers:subscriber.send(msg)

# Dictionary of all created exchanges_exchanges = defaultdict(Exchange)

# Return the Exchange instance associated with a given namedef get_exchange(name):

> return _exchanges[name]

An exchange is really nothing more than an object that keeps a set of active subscribersand provides methods for attaching, detaching, and sending messages. Each exchangeis identified by a name, and the get_exchange() function simply returns the Exchange instance associated with a given name.Here is a simple example that shows how to use an exchange:

# Example of a task. Any object with a send() method

class Task:
...def send(self, msg):

> ...

task_a = Task()task_b = Task()

# Example of getting an exchangeexc = get_exchange(‘name')

# Examples of subscribing tasks to itexc.attach(task_a)exc.attach(task_b)

# Example of sending messagesexc.send(‘msg1')exc.send(‘msg2')

# Example of unsubscribingexc.detach(task_a)exc.detach(task_b)

Although there are many different variations on this theme, the overall idea is the same.Messages will be delivered to an exchange and the exchange will deliver them to attachedsubscribers.

## 讨论

The concept of tasks or threads sending messages to one another (often via queues) iseasy to implement and quite popular. However, the benefits of using a public/subscribe(pub/sub) model instead are often overlooked.First, the use of an exchange can simplify much of the plumbing involved in setting upcommunicating threads. Instead of trying to wire threads together across multiple pro‐gram modules, you only worry about connecting them to a known exchange. In somesense, this is similar to how the logging library works. In practice, it can make it easierto decouple various tasks in the program.Second, the ability of the exchange to broadcast messages to multiple subscribers opensup new communication patterns. For example, you could implement systems with re‐dundant tasks, broadcasting, or fan-out. You could also build debugging and diagnostictools that attach themselves to exchanges as ordinary subscribers. For example, here isa simple diagnostic class that would display sent messages:

class DisplayMessages:def **init**(self):self.count = 0def send(self, msg):self.count += 1print(‘msg[{}]: {!r}'.format(self.count, msg))
exc = get_exchange(‘name')d = DisplayMessages()exc.attach(d)

Last, but not least, a notable aspect of the implementation is that it works with a varietyof task-like objects. For example, the receivers of a message could be actors (as describedin Recipe 12.10), coroutines, network connections, or just about anything that imple‐ments a proper send() method.One potentially problematic aspect of an exchange concerns the proper attachment anddetachment of subscribers. In order to properly manage resources, every subscriber thatattaches must eventually detach. This leads to a programming model similar to this:

exc = get_exchange(‘name')exc.attach(some_task)try:

> ...

finally:exc.detach(some_task)
In some sense, this is similar to the usage of files, locks, and similar objects. Experiencehas shown that it is quite easy to forget the final detach() step. To simplify this, youmight consider the use of the context-management protocol. For example, adding asubscribe() method to the exchange like this:

from contextlib import contextmanagerfrom collections import defaultdict

class Exchange:def **init**(self):self._subscribers = set()def attach(self, task):self._subscribers.add(task)def detach(self, task):self._subscribers.remove(task)
@contextmanagerdef subscribe(self, *tasks):

> for task in tasks:self.attach(task)try:yieldfinally:for task in tasks:self.detach(task)

def send(self, msg):for subscriber in self._subscribers:subscriber.send(msg)

# Dictionary of all created exchanges_exchanges = defaultdict(Exchange)

# Return the Exchange instance associated with a given namedef get_exchange(name):

> return _exchanges[name]

# Example of using the subscribe() methodexc = get_exchange(‘name')with exc.subscribe(task_a, task_b):

> ...exc.send(‘msg1')exc.send(‘msg2')...

# task_a and task_b detached here

Finally, it should be noted that there are numerous possible extensions to the exchangeidea. For example, exchanges could implement an entire collection of message channels

or apply pattern matching rules to exchange names. Exchanges can also be extendedinto distributed computing applications (e.g., routing messages to tasks on differentmachines, etc.).

# 12.12 使用生成器代替线程

## 问题

You want to implement concurrency using generators (coroutines) as an alternative tosystem threads. This is sometimes known as user-level threading or green threading.

## 解决方案

To implement your own concurrency using generators, you first need a fundamentalinsight concerning generator functions and the yield statement. Specifically, the fun‐damental behavior of yield is that it causes a generator to suspend its execution. Bysuspending execution, it is possible to write a scheduler that treats generators as a kindof “task” and alternates their execution using a kind of cooperative task switching.To illustrate this idea, consider the following two generator functions using a simpleyield:

# Two simple generator functionsdef countdown(n):

> while n > 0:print(‘T-minus', n)yieldn -= 1> print(‘Blastoff!')

def countup(n):
x = 0while x < n:

> print(‘Counting up', x)yieldx += 1

These functions probably look a bit funny using yield all by itself. However, considerthe following code that implements a simple task scheduler:

from collections import deque

class TaskScheduler:def **init**(self):self._task_queue = deque()def new_task(self, task):
‘''Admit a newly started task to the scheduler

‘''self._task_queue.append(task)

def run(self):
‘''Run until there are no more tasks‘''while self._task_queue:

> > task = self._task_queue.popleft()try:
> > 
> > # Run until the next yield statementnext(task)self._task_queue.append(task)

except StopIteration:# Generator is no longer executingpass

# Example usesched = TaskScheduler()sched.new_task(countdown(10))sched.new_task(countdown(5))sched.new_task(countup(15))sched.run()

In this code, the TaskScheduler class runs a collection of generators in a round-robinmanner—each one running until they reach a yield statement. For the sample, theoutput will be as follows:

T-minus 10T-minus 5Counting up 0T-minus 9T-minus 4Counting up 1T-minus 8T-minus 3Counting up 2T-minus 7T-minus 2...

At this point, you’ve essentially implemented the tiny core of an “operating system” ifyou will. Generator functions are the tasks and the yield statement is how tasks signalthat they want to suspend. The scheduler simply cycles over the tasks until none are leftexecuting.In practice, you probably wouldn’t use generators to implement concurrency for some‐thing as simple as shown. Instead, you might use generators to replace the use of threadswhen implementing actors (see Recipe 12.10) or network servers.

The following code illustrates the use of generators to implement a thread-free versionof actors:

from collections import deque

class ActorScheduler:def **init**(self):self._actors = { } # Mapping of names to actorsself._msg_queue = deque() # Message queuedef new_actor(self, name, actor):‘''Admit a newly started actor to the scheduler and give it a name‘''self._msg_queue.append((actor,None))self._actors[name] = actordef send(self, name, msg):
‘''Send a message to a named actor‘''actor = self._actors.get(name)if actor:

> self._msg_queue.append((actor,msg))

def run(self):
‘''Run as long as there are pending messages.‘''while self._msg_queue:

> > actor, msg = self._msg_queue.popleft()try:
> > 
> > actor.send(msg)

except StopIteration:pass

# Example useif **name** == ‘**main**':

> def printer():while True:msg = yieldprint(‘Got:', msg)def counter(sched):while True:> # Receive the current countn = yieldif n == 0:
> 
> > break
> 
> # Send to the printer tasksched.send(‘printer', n)# Send the next count to the counter task (recursive)
> 
> sched.send(‘counter', n-1)
> 
> sched = ActorScheduler()# Create the initial actorssched.new_actor(‘printer', printer())sched.new_actor(‘counter', counter(sched))
> 
> # Send an initial message to the counter to initiatesched.send(‘counter', 10000)sched.run()

The execution of this code might take a bit of study, but the key is the queue of pendingmessages. Essentially, the scheduler runs as long as there are messages to deliver. Aremarkable feature is that the counter generator sends messages to itself and ends upin a recursive cycle not bound by Python’s recursion limit.Here is an advanced example showing the use of generators to implement a concurrentnetwork application:

from collections import dequefrom select import select

# This class represents a generic yield event in the schedulerclass YieldEvent:

> def handle_yield(self, sched, task):passdef handle_resume(self, sched, task):pass

# Task Schedulerclass Scheduler:

> def **init**(self):self._numtasks = 0 # Total num of tasksself._ready = deque() # Tasks ready to runself._read_waiting = {} # Tasks waiting to readself._write_waiting = {} # Tasks waiting to write> # Poll for I/O events and restart waiting tasksdef _iopoll(self):
> 
> > rset,wset,eset = select(self._read_waiting,self._write_waiting,[])for r in rset:evt, task = self._read_waiting.pop(r)evt.handle_resume(self, task)for w in wset:evt, task = self._write_waiting.pop(w)evt.handle_resume(self, task)

def new(self,task):> ‘''Add a newly started task to the scheduler‘'‘

> self._ready.append((task, None))self._numtasks += 1

def add_ready(self, task, msg=None):‘''Append an already started task to the ready queue.msg is what to send into the task when it resumes.‘''self._ready.append((task, msg))> # Add a task to the reading setdef _read_wait(self, fileno, evt, task):

> > self._read_waiting[fileno] = (evt, task)
> 
> # Add a task to the write setdef _write_wait(self, fileno, evt, task):
> 
> > self._write_waiting[fileno] = (evt, task)

def run(self):> ‘''Run the task scheduler until there are no tasks‘''while self._numtasks:

> > if not self._ready:self._iopoll()> > task, msg = self._ready.popleft()try:
> > 
> > > > > > # Run the coroutine to the next yieldr = task.send(msg)if isinstance(r, YieldEvent):
> > > > 
> > > > r.handle_yield(self, task)

else:raise RuntimeError(‘unrecognized yield event')

except StopIteration:self._numtasks -= 1

# Example implementation of coroutine-based socket I/Oclass ReadSocket(YieldEvent):

> def **init**(self, sock, nbytes):self.sock = sockself.nbytes = nbytesdef handle_yield(self, sched, task):sched._read_wait(self.sock.fileno(), self, task)def handle_resume(self, sched, task):data = self.sock.recv(self.nbytes)sched.add_ready(task, data)

class WriteSocket(YieldEvent):def **init**(self, sock, data):self.sock = sockself.data = data
def handle_yield(self, sched, task):

> sched._write_wait(self.sock.fileno(), self, task)

def handle_resume(self, sched, task):nsent = self.sock.send(self.data)sched.add_ready(task, nsent)class AcceptSocket(YieldEvent):def **init**(self, sock):self.sock = sockdef handle_yield(self, sched, task):sched._read_wait(self.sock.fileno(), self, task)def handle_resume(self, sched, task):r = self.sock.accept()sched.add_ready(task, r)

# Wrapper around a socket object for use with yieldclass Socket(object):

> def **init**(self, sock):self._sock = sockdef recv(self, maxbytes):return ReadSocket(self._sock, maxbytes)def send(self, data):return WriteSocket(self._sock, data)def accept(self):return AcceptSocket(self._sock)def **getattr**(self, name):return getattr(self._sock, name)

if **name** == ‘**main**':
from socket import socket, AF_INET, SOCK_STREAMimport time

# Example of a function involving generators. This should# be called using line = yield from readline(sock)def readline(sock):

> > chars = []while True:
> > 
> > > > c = yield sock.recv(1)if not c:
> > > 
> > > break
> > 
> > chars.append(c)if c == b'n':
> > 
> > > break
> 
> return b'‘.join(chars)

# Echo server using generatorsclass EchoServer:

> def **init**(self,addr,sched):self.sched = schedsched.new(self.server_loop(addr))def server_loop(self,addr):> s = Socket(socket(AF_INET,SOCK_STREAM))
> 
> s.bind(addr)s.listen(5)while True:
> 
> > c,a = yield s.accept()print(‘Got connection from ‘, a)self.sched.new(self.client_handler(Socket(c)))

def client_handler(self,client):while True:> line = yield from readline(client)if not line:

> > break
> 
> line = b'GOT:' + linewhile line:
> 
> > nsent = yield client.send(line)line = line[nsent:]
> 
> client.close()print(‘Client closed')

sched = Scheduler()EchoServer((‘',16000),sched)sched.run()

This code will undoubtedly require a certain amount of careful study. However, it isessentially implementing a small operating system. There is a queue of tasks ready torun and there are waiting areas for tasks sleeping for I/O. Much of the scheduler involvesmoving tasks between the ready queue and the I/O waiting area.

## 讨论

When building generator-based concurrency frameworks, it is most common to workwith the more general form of yield:

def some_generator():...result = yield data...
Functions that use yield in this manner are more generally referred to as “coroutines.”Within a scheduler, the yield statement gets handled in a loop as follows:

f = some_generator()

# Initial result. Is None to start since nothing has been computedresult = Nonewhile True:

> try:data = f.send(result)result = ... do some calculation ...except StopIteration:break

The logic concerning the result is a bit convoluted. However, the value passed to send()defines what gets returned when the yield statement wakes back up. So, if a yield isgoing to return a result in response to data that was previously yielded, it gets returnedon the next send() operation. If a generator function has just started, sending in a valueof None simply makes it advance to the first yield statement.In addition to sending in values, it is also possible to execute a close() method on agenerator. This causes a silent GeneratorExit exception to be raised at the yield state‐ment, which stops execution. If desired, a generator can catch this exception and per‐form cleanup actions. It’s also possible to use the throw() method of a generator to raisean arbitrary execution at the yield statement. A task scheduler might use this to com‐municate errors into running generators.The yield from statement used in the last example is used to implement coroutinesthat serve as subroutines or procedures to be called from other generators. Essentially,control transparently transfers to the new function. Unlike normal generators, a func‐tion that is called using yield from can return a value that becomes the result of theyield from statement. More information about yield from can be found in PEP 380.Finally, if programming with generators, it is important to stress that there are somemajor limitations. In particular, you get none of the benefits that threads provide. Forinstance, if you execute any code that is CPU bound or which blocks for I/O, it willsuspend the entire task scheduler until the completion of that operation. To work aroundthis, your only real option is to delegate the operation to a separate thread or processwhere it can run independently. Another limitation is that most Python libraries havenot been written to work well with generator-based threading. If you take this approach,you may find that you need to write replacements for many standard library functions.As basic background on coroutines and the techniques utilized in this recipe, see PEP342 and “A Curious Course on Coroutines and Concurrency”.PEP 3156 also has a modern take on asynchronous I/O involving coroutines. In practice,it is extremelyunlikely that you will write a low-level coroutine scheduler yourself.However, ideas surrounding coroutines are the basis for many popular libraries, in‐cluding gevent, greenlet, Stackless Python, and similar projects.

# 12.13 多个线程队列轮询

## 问题

You have a collection of thread queues, and you would like to be able to poll them forincoming items, much in the same way as you might poll a collection of network con‐nections for incoming data.

## 解决方案

A common solution to polling problems involves a little-known trick involving a hiddenloopback network connection. Essentially, the idea is as follows: for each queue (or anyobject) that you want to poll, you create a pair of connected sockets. You then write onone of the sockets to signal the presence of data. The other sockect is then passed toselect() or a similar function to poll for the arrival of data. Here is some sample codethat illustrates this idea:

import queueimport socketimport os

class PollableQueue(queue.Queue):def **init**(self):
super().**init**()# Create a pair of connected socketsif os.name == ‘posix':

> self._putsocket, self._getsocket = socket.socketpair()

else:# Compatibility on non-POSIX systemsserver = socket.socket(socket.AF_INET, socket.SOCK_STREAM)server.bind((‘127.0.0.1', 0))server.listen(1)self._putsocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)self._putsocket.connect(server.getsockname())self.*getsocket,* = server.accept()server.close()def fileno(self):return self._getsocket.fileno()def put(self, item):super().put(item)self._putsocket.send(b'x')def get(self):self._getsocket.recv(1)return super().get()
In this code, a new kind of Queue instance is defined where there is an underlying pairof connected sockets. The socketpair() function on Unix machines can establish suchsockets easily. On Windows, you have to fake it using code similar to that shown (itlooks a bit weird, but a server socket is created and a client immediately connects to itafterward). The normal get() and put() methods are then redefined slightly to performa small bit of I/O on these sockets. The put() method writes a single byte of data to oneof the sockets after putting data on the queue. The get() method reads a single byte ofdata from the other socket when removing an item from the queue.

The fileno() method is what makes the queue pollable using a function such as select(). Essentially, it just exposes the underlying file descriptor of the socket used bythe get() function.Here is an example of some code that defines a consumer which monitors multiplequeues for incoming items:

import selectimport threading

def consumer(queues):
‘''Consumer that reads data on multiple queues simultaneously‘''while True:

> > can*read,* , _ = select.select(queues,[],[])for r in can_read:
> > 
> > item = r.get()print(‘Got:', item)

q1 = PollableQueue()q2 = PollableQueue()q3 = PollableQueue()t = threading.Thread(target=consumer, args=([q1,q2,q3],))t.daemon = Truet.start()

# Feed data to the queuesq1.put(1)q2.put(10)q3.put(‘hello')q2.put(15)...

If you try it, you’ll find that the consumer indeed receives all of the put items, regardlessof which queues they are placed in.

## 讨论

The problem of polling non-file-like objects, such as queues, is often a lot trickier thanit looks. For instance, if you don’t use the socket technique shown, your only option isto write code that cycles through the queues and uses a timer, like this:

import timedef consumer(queues):

> while True:for q in queues:if not q.empty():item = q.get()print(‘Got:', item)> # Sleep briefly to avoid 100% CPUtime.sleep(0.01)

This might work for certain kinds of problems, but it’s clumsy and introduces otherweird performance problems. For example, if new data is added to a queue, it won’t bedetected for as long as 10 milliseconds (an eternity on a modern processor).You run into even further problems if the preceding polling is mixed with the pollingof other objects, such as network sockets. For example, if you want to poll both socketsand queues at the same time, you might have to use code like this:

import select

def event_loop(sockets, queues):while True:

# polling with a timeoutcan*read,* , _ = select.select(sockets, [], [], 0.01)for r in can_read:

> handle_read(r)

for q in queues:if not q.empty():item = q.get()print(‘Got:', item)
The solution shown solves a lot of these problems by simply putting queues on equalstatus with sockets. A single select() call can be used to poll for activity on both. It isnot necessary to use timeouts or other time-based hacks to periodically check. More‐over, if data gets added to a queue, the consumer will be notified almost instantaneously.Although there is a tiny amount of overhead associated with the underlying I/O, it oftenis worth it to have better response time and simplified coding.

# 12.14 在 Unix 系统上面启动守护进程

## 问题

You would like to write a program that runs as a proper daemon process on Unix orUnix-like systems.

## 解决方案

Creating a proper daemon process requires a precise sequence of system calls and carefulattention to detail. The following code shows how to define a daemon process alongwith the ability to easily stop it once launched:

# !/usr/bin/env python3# daemon.py

import osimport sys

import atexitimport signal

def daemonize(pidfile, *, stdin='/dev/null',> stdout='/dev/null',stderr='/dev/null'):

if os.path.exists(pidfile):raise RuntimeError(‘Already running')

# First fork (detaches from parent)try:

> if os.fork() > 0:raise SystemExit(0) # Parent exit

except OSError as e:raise RuntimeError(‘fork #1 failed.')
os.chdir(‘/')os.umask(0)os.setsid()# Second fork (relinquish session leadership)try:

> if os.fork() > 0:raise SystemExit(0)

except OSError as e:raise RuntimeError(‘fork #2 failed.')

# Flush I/O bufferssys.stdout.flush()sys.stderr.flush()

# Replace file descriptors for stdin, stdout, and stderrwith open(stdin, ‘rb', 0) as f:

> os.dup2(f.fileno(), sys.stdin.fileno())

with open(stdout, ‘ab', 0) as f:os.dup2(f.fileno(), sys.stdout.fileno())with open(stderr, ‘ab', 0) as f:os.dup2(f.fileno(), sys.stderr.fileno())

# Write the PID filewith open(pidfile,'w') as f:

> print(os.getpid(),file=f)

# Arrange to have the PID file removed on exit/signalatexit.register(lambda: os.remove(pidfile))

# Signal handler for termination (required)def sigterm_handler(signo, frame):

> raise SystemExit(1)

signal.signal(signal.SIGTERM, sigterm_handler)

def main():
import timesys.stdout.write(‘Daemon started with pid {}n'.format(os.getpid()))while True:

> sys.stdout.write(‘Daemon Alive! {}n'.format(time.ctime()))time.sleep(10)

if **name** == ‘**main**':
PIDFILE = ‘/tmp/daemon.pid'

if len(sys.argv) != 2:print(‘Usage: {} [start|stop]'.format(sys.argv[0]), file=sys.stderr)raise SystemExit(1)if sys.argv[1] == ‘start':try:daemonize(PIDFILE,stdout='/tmp/daemon.log',stderr='/tmp/dameon.log')except RuntimeError as e:print(e, file=sys.stderr)raise SystemExit(1)
main()

elif sys.argv[1] == ‘stop':if os.path.exists(PIDFILE):with open(PIDFILE) as f:os.kill(int(f.read()), signal.SIGTERM)else:print(‘Not running', file=sys.stderr)raise SystemExit(1)else:print(‘Unknown command {!r}'.format(sys.argv[1]), file=sys.stderr)raise SystemExit(1)
To launch the daemon, the user would use a command like this:

bash % daemon.py startbash % cat /tmp/daemon.pid2882bash % tail -f /tmp/daemon.logDaemon started with pid 2882Daemon Alive! Fri Oct 12 13:45:37 2012Daemon Alive! Fri Oct 12 13:45:47 2012...

Daemon processes run entirely in the background, so the command returns immedi‐ately. However, you can view its associated pid file and log, as just shown. To stop thedaemon, use:

bash % daemon.py stopbash %

## 讨论

This recipe defines a function daemonize() that should be called at program startup tomake the program run as a daemon. The signature to daemonize() is using keyword-only arguments to make the purpose of the optional arguments more clear when used.This forces the user to use a call such as this:

daemonize(‘daemon.pid',stdin='/dev/null,stdout='/tmp/daemon.log',stderr='/tmp/daemon.log')
As opposed to a more cryptic call such as:# Illegal. Must use keyword argumentsdaemonize(‘daemon.pid',

> ‘/dev/null', ‘/tmp/daemon.log','/tmp/daemon.log')

The steps involved in creating a daemon are fairly cryptic, but the general idea is asfollows. First, a daemon has to detach itself from its parent process. This is the purposeof the first os.fork() operation and immediate termination by the parent.After the child has been orphaned, the call to os.setsid() creates an entirely newprocess session and sets the child as the leader. This also sets the child as the leader ofa new process group and makes sure there is no controlling terminal. If this all soundsa bit too magical, it has to do with getting the daemon to detach properly from theterminal and making sure that things like signals don’t interfere with its operation.The calls to os.chdir() and os.umask(0) change the current working directory andreset the file mode mask. Changing the directory is usually a good idea so that thedaemon is no longer working in the directory from which it was launched.The second call to os.fork() is by far the more mysterious operation here. This stepmakes the daemon process give up the ability to acquire a new controlling terminal andprovides even more isolation (essentially, the daemon gives up its session leadershipand thus no longer has the permission to open controlling terminals). Although youcould probably omit this step, it’s typically recommended.Once the daemon process has been properly detached, it performs steps to reinitializethe standard I/O streams to point at files specified by the user. This part is actuallysomewhat tricky. References to file objects associated with the standard I/O streams arefound in multiple places in the interpreter (sys.stdout, sys.**stdout**, etc.). Simplyclosing sys.stdout and reassigning it is not likely to work correctly, because there’s noway to know if it will fix all uses of sys.stdout. Instead, a separate file object is opened,and the os.dup2() call is used to have it replace the file descriptor currently being used

by sys.stdout. When this happens, the original file for sys.stdout will be closed andthe new one takes its place. It must be emphasized that any file encoding or text handlingalready applied to the standard I/O streams will remain in place.A common practice with daemon processes is to write the process ID of the daemon ina file for later use by other programs. The last part of the daemonize() function writesthis file, but also arranges to have the file removed on program termination. The atexit.register() function registers a function to execute when the Python interpreterterminates. The definition of a signal handler for SIGTERM is also required for a gracefultermination. The signal handler merely raises SystemExit() and nothing more. Thismight look unnecessary, but without it, termination signals kill the interpreter withoutperforming the cleanup actions registered with atexit.register(). An example ofcode that kills the daemon can be found in the handling of the stop command at theend of the program.More information about writing daemon processes can be found in Advanced Pro‐gramming in the UNIX Environment, 2nd Edition, by W. Richard Stevens and StephenA. Rago (Addison-Wesley, 2005). Although focused on C programming, all of the ma‐terial is easily adapted to Python, since all of the required POSIX functions are availablein the standard library.