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
\
>Performance best practices in C#

>Issue Add non-generic TaskCompletionSource 37452

>Performance best practices in C#

>The danger of TaskCompletionSource class

## Awaitables

- **Awaiter** type -> has GetAwaiter + INotifyCompletion

- unravels isCompleted >> getResult() + oncompleted event
 
 - await Task.Yield

 ## Locals
 - [THreadStatic] attrivute

 - static get init once => first thread wins

 - **ThreadLocal\<T\>** for thread
     - threadsafe random //TODO: check with sd

    - trackallvalues:true -> threadlocal.Values

- **AsyncLocal\<T>** for as\aw 

- Lazy\<T> AsyncLazy\<T> AsyncEx
????
- just pass around context
