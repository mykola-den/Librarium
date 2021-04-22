# Thread pool

https://docs.microsoft.com/en-us/archive/msdn-magazine/2010/september/concurrency-throttling-concurrency-in-the-clr-4-0-threadpool

Hill climbing algorithm

iocp threads

global vs local queue and soft starvation

preemptive workload distribution

Enqueue work item

# Task API W2L1
- https://blog.stephencleary.com/2013/08/startnew-is-dangerous.html 
- https://docs.microsoft.com/en-us/previous-versions/msp-n-p/ff963551(v=pandp.10)


task based asynchronous pattern

- don't separate task creation and task executiion
- just thread sleep -> task.delay gives way too much overhead
- task.factory.startnew 
#### Task Run
- task.run covers 99% but with limits of custom scheduler

- waiting taskX.Wait() or taskX.Result **Block THREAD**

-- faulted canceled !!completed
-- throws  AGGREGATE EXCEPTION with InnerException

-- STATIC Task Scheduler.+ unobserved exception  event listener

#### Task.factory.startnew
more options

task.continue with + continuation pattern with switch for faulted, cancelled and  completed cases 

cancellatuion Throw if cancellatuinn requested

task cancel exception != cancelled operation. only cancelled scheduling

###T ask combination
wait all <-> waitany **//AVOID THEM!!!**

TASK.Unwrap 
when using startnew with inner task lambda
built in in Run!
https://docs.microsoft.com/en-us/previous-versions/msp-n-p/ff963551(v=pandp.10)

https://blog.stephencleary.com/2013/08/startnew-is-dangerous.html

# Aync await W2L2
- async creates state machine with switch-case and enables using await
- await separates method to before and after
- purpose - scalability not performance

## Async eliding: ~~async~~ - > return Task
 [async eliding blog](https://blog.stephencleary.com/2016/12/eliding-async-await.html)
 - omit async and return task to the upper level
 - **using** will dispose while we use async operation //**using != async eliding**
 - benefit perf  by reduce state machine
 - changing exception handling 
    - the exception is raised dicetly rather than to the task
 - for oneliner methods

## ValueTask
- for 2 paths sync happy path and async path
- cache task Task.fromResult(X)

```csharp
GC.GetAllocatedBytesForCurrentThread() // measure asynce
```
**VT** is a [struct](https://www.c-sharpcorner.com/blogs/difference-between-struct-and-class-in-c-sharp)
stric

```
return new ValueTask<T>(xT)
```
**Can't await twice or .AsTask()**

> ValueTask pooling and ivaluetasksource with cache in .net5.0 that reduces allocations

### TODO: Check Websockets change for .net5.0 on value tasks (huge perf difference)

[Understanding the Whys, Whats, and Whens of ValueTask](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/)

Prefer ValueTask to Task, always; and don't await twice
Async ValueTask Pooling in .NET 5
.NET 5 and pooling for ValueTasks
Proposal: ValueTask.WhenAll #23625
"Determine what to do with "async ValueTask" caching" #13633

## task api
factory.startnew -> needs unwrap (even with async->await sign with async lambdas
- use 2 awaits outer and inner
-starnew + longrunning + await = switch back to thread pull after first await
solution 
- abandon async- await
- use task run ((!!))
- write new single thread syncContext
/// only bad solutions

### TODO: check for startnew

- as/aw is better than continue with. don't mix. switch cw ->

- hot|cold tasks. need to .Start() or Task.Run() (cold tasks are bad design)

# Async 2 W4

## Sync Context
- winforms have ui thread that has context
- capture and release context
- post method and send method
- async void == antipattern

- asp.net each request must be processed on 1 particular thread(thread affinity)

- asp.net core -> abandoned thread affinity. just default task scheduler

.Result is *BAD* we're blocking thread


.result + congifdurawait = better

.configureawait false for reusable library //todo: check in sd
.configureawait creates new task

#### Execution context
flows with async logical flow

## Sync vs Async
????


## Fire and forget
- async void is bad -  crashes the process UNh ex.
    - depends on sync.Context it receives ex from asvoid

    - **TODO check SD for async void**

- async task is better. returns exception

- _ + task. discard result

**Exception**
- extension method of async void with try catch on task and ConfigureAwait(false)
with logging
- use it to wrap into static void function without asybc



## TaskCompletionSource
factory to produce tasks

inarrivalorder fix

-runcontinuationsAsynchronously
- net5 non-generic will be introduced

**EAP = Event Asynchronous pattern**

Additional materials:

>Performance best practices in C#

>Issue Add non-generic TaskCompletionSource 37452

>Performance best practices in C#

>The danger of TaskCompletionSource class

## Awaitables

- **Awaiter** type -> has GetAwaiter + INotifyCompletion

- unravels isCompleted >> getResult() + oncompleted event
 
 - await Task.Yield

 ## Locals
 - [THreadStatic] attribute

 - static get init once => first thread wins

 - **ThreadLocal\<T\>** for thread
     - threadsafe random //TODO: check with sd

    - trackallvalues:true -> threadlocal.Values

- **AsyncLocal\<T>** for as\aw 

- Lazy\<T> AsyncLazy\<T> AsyncEx
????
- just pass around context


///TODO: Threadcontention count
https://stackoverflow.com/questions/1970345/what-is-thread-contention/7064008

# Week 4
### Task when all /any
- when all for parrallelizm
- for batching
- unwrap inner task
#### when any -returns first COMPLETED|FAULTED|CANCELED 
-do  await await Task.whenany(tasks) - to handle excheptions in inner tasks

- don't forget to cancel all tasks after taskwhenany  via cts.Cansel()

-extension for timeout

await whenany(task + delay) -> realtask==task -> cancel timer-> inner await)

-Task.delay(-1) 


- pattern for canceling awaiting of operation by Fauler
    register cancellation 
- processing in arrival order
    - list of task + while + remove from list
    - select with await inside
- NET5 task.whenany 2 introduces new 2 task promise with much cheaper promise

### IASYNCDISPOSABLE
- uses ValueTask for performance
await using var s = new SomeDisposable
s.UseMe()

asynchronous dispose

- disposable implementation

suppressfinalizer( -true -> remove from GC finalizer queue

- async disposable pattern
finalizing is antipattern - > don't use them
use explicit cleaunp

//TODO: deadlock vs tcs
// automation framework search for async disposables

https://bitbucket.sbtech.com/plugins/servlet/search?q=project%3ASD%20IAsyncDisposable


# Week 5 Low level concurrency

## Hardware 

#### CPU registers - cpu like variables
jit responsible for register allocation
register -level 0
**cpu cache** - level 1 stores copy of data fro cpu
- there are separate insttructions cache and data cache
-  has also several levels
- access speed = depends. from 0 to tens of cpu cycles
- has trait of associativity if can map address to N places

**cache miss** - long we go to main memory
that intros **prefetcher** to get either data or instructions

**RAM and NonVolatileMemory**
- speed 100+ cpu cycles
- power outage damages may be treated by db-like commit structure

**CPU pipelines** allows to parralel

**memory model**
- set of rules for shared data
- weak <===> strong mem model
+ 1 tool **memory barrier** < - hardware dependant  (may be emulated|| skipped||expressed)
in C#  its **VOLATILE**'❤❤❤🛠💔💔 keyword
prohibits set of reordering and local optimizations

### Barriers🔥🔥🔥
STORE LOAD => ss| sl |ls |ll =>
ls || ll - >> Acquire semantics prohibits reordering before first load

ss||sl ->release semantic -> no ops are moved after store.

### VOLATILE
- thread.volatileread + thread.volatilewrite \\dont use it
- thread.memorybarrier - > heavt

-**Volatile.read** ->read with acquire semantic -> ls ll
>guarantee that nothing will be ordered above the read
- **volatile.write** ->write with release semantic -> sl ss
> write guarantees that nothing will be ordered after the write

x86 strong model.- evaporates volatile
arm7 flooded with full memory barrier
arm8  will use barrier properly

#### Volatile example 
= Cancellationtoken source

**Volatile = eventually consistemt programming model for CPU**

### Interlocked
basic

lock(sharedCounter){
    sharedcounter++
}

Interlocked.Increment(ref sharedcounter)
#### API Interlock
increment\decrement + ref
add
read + ref long location
- exchange
- compareexchange returns old values
- memorybarrier
//TODO: scan repos for interlock exchange

**implemented with compiler intrinsic**

lockfree code comes with costs of GC
they are foundation fo **lock-free** and **wait-free**

**http://deadlockempire.github.io**

# Week 6 Synchronizarion
## lock
exclusive locking  => private lock
> protects code not an object itself
> treat only aas flag
- has thread affinity
- invalid in async-await that can break thread affinity
- lock syntactic sugar around concept of monitor
```
lock(){} -> monitor.Enter(ref) + bool -> monitor.exit()locker
```
## monitor
- thread affine
- syntactic sugared lock
- monitor exit must be always called
- lock1(lock()) - deadlock nest in exactly same order

- monitor.tryenter(ref)+timneout = no locks but can be worse
- disposable timedlock example for tryenter-exit
- useful for implementing concurrent queeue

- lock striping + mutex + spinlock

## non-exclusive locks
### sempapthore ()
- old 
- sync Api WaitOne
### semaphoreslim()
- no thread affinity
- can use in async waitasync
- ctor (number) wait() release() => must be matched(dev responsibility)

### task throtling
create semaphore. extension method RunThrottledAsync(collection of factories of tasks (ie async()=>{await task}) lambda)

//analogy krippled channels or TPL

### readerwriterlockslim()
- many readers little writers
> reader starvation
- enter readlock
- enter write locks
//TODO: check string intern pool

### custom string internpool
string.itnern caveat -> enter writelock inside readlock
> proper way = add exit consequently and add more checks

> lock-free takes more checks

### upgrade reader to writer
- EnterUpgradeableReadlock only for 1 thread

### Signalining AutoResetEvent
- waithandle /base class for waiting
- a lot of classes implement it
- api: signal waitall\any
- idisposable
> example AutoResetEvent remember to cleanup .close\dispose
- old manualresetevent\eventslim
- countdownevent //todo: maybe for tests
- high level barrier (exotic)

## Async lock
- semaphore slim wait with limit 1
- AsyncLock (AsyncEx library)
- asyncmanualresetevetnt
    == task completion source
- problem reset 
    > mutate solution => only completed, orphan awaits
- asyncautroresetevent
  > has concurrentqueue

  both present in asyncex library


// use case for volatile



# Week 7 Concurrent DAta Structures
> BC |CQ |CD | CB

## Foundations 1/10
- spsc | mpsc |spmc | mpmc
- what if data structure is single ended and writes go to only one end
    single-ended | multi-ended
- volatile -half barrier(eventually consistent)
- interloced -full barier **atomic operation** (will lock the whole cpu cache line)

- single-end -> volatile
- multi-end -> at least 1 interlocked

### bounded vs unbounded collections
- bounded -> faster
- unbounded -> group of boundede e.g concurrentqueue with concurrentqueue segment

IProducerConsumerCollection: ienumerable icollection
    void copyto
    bool tryadd
    bool trytake
    T toarray

## BlockingCollection<T>
- mpmc
- optional\bound
-Add\Take -blocking add\remove
- optional timeout
- cts

**no asyncstpport**
replacable by:
TPL Dataflow alternative BufferBlock<T>
Channels

blockingcollection(stack) -> lifo


## ConcurrentStack<T> 5/10

head -> [v-> v-> null]
Stack implementation
ABA-PROBLEM (Lost C in stack)
    - dont reuse nodes **used by concurrent stack**
        -allocate new node on every push
        - interlocked.CEx works ok with new refs
    
    - use taggedreference
        -reference + counter (doesnpt exis
    - use intermediate nodes | hazard pointers -> exotic

Summary
- threadsafe
- single linked list of elements
- lockfree adding
- pushrange \trypoprange optimized


## ConcurrentQueue
- design
    queue = linked list of segments
    segment - bounded mpmc queue
        - has padded allocation for HEAD and TAIL to separate on CPU cache lines (for interlocked)
    slot - minimal cell with data and sequence number

HEAD for dequeue
TAIL for enqueue

### **ENQUEUE** /dequeue
- q.enqueue -> q.segment.Enqueue
- s.Enqueue uses Interlocked to atomically reserve a slot
return bool
    while true
        volatile read tail
        find index and volatile read sequence number of slot
        check consistency of segment tail and queue tail
        volatile write next item to sequence //default on dequeeu

### Count  / isempty
- multisegment count locks
- isempty much faster

### Enumeration
    - lock - > snapshot -> preserveforobservation -> yield return
http://www.1024cores.net/

## ConcurrentQueue in Kestrel
memory pool
    memory poolblock
        return -> concurrentqueue '<'memoryblock/>

## Concurrent Dictionary

design
    buckets -> node ->kvp
        buckets -grown when needed
        node - key+value+hash(compute-heavue)+_next(ref to a next node)

add or mod process:

    -1. run facktory method
    0. lock bucket
    add -replace remove
    release lock bucket

getting value :
[volatile.read ](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/volatile)
returns snapshot(might be stale immediatley)

#### Locking bucket
+ additional [] object _locks
+ can be tweaked by ctor (concurrencylevel, int capacity)
+ lock(_lock[i])
+ single lock object is used for multiple buckets. we may lock several buckets

## CDict Demo GetOrAdd
- wrapper for async task. Valuetasks to make it async ready

## Concurrent bag
thread safe unordered set