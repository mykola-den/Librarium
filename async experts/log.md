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
