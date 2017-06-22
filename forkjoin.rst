ForkJoinPool
============
In the begining of the tutorial we said- parallelization is almost free, with a small change you can enjoy the benifit of parallel stream processing. The complete parallel stream processing is based on ForkJoinPool concept. It was delivered with jdk7 which provides a highly specialized ``ExecutorService``. If you can recall, we submit multiple indepenedent tasks to ExecutorService which are then executed by thread-pool threads. In case of parallel stream we don't have multiple tasks where as we have a single task of larger size. To execute this big task we have to perform following actions explicitly.

- Divide the tasks into smaller chunks
- Process all tasks independently
- Join the partial results from each task

ForkJoinPool internally does all these steps for you. We will typically submit a single task to ForkJoinPool and awaits its completion. The ForkJoinPool and the task itself work togather to divide and conquer the problem. Any problems that can be recursively divided can be a candidature for Fork-Join.


ForkJoinPool creation
---------------------
Creating ForkJoinPool is simple, call its no-arg constructor to create an instance that will use ``Runtime.availableProcessors()`` method to determine number of worker threads to be used by the pool. It also provides `ForkJoinPool(int parallelism)` constructor that allows you to override the number of threads that will be used. You shouldn't override the number of threads unless you have a valid reason to do it.

Though it internally uses the number of processors (cores) available to create the worker threads, we should always create a single instance of ForkJoinPool and different kinds of tasks should be submitted to the same pool.


.. note:: The level of parallelism can also be constrolled by setting ``java.util.concurrent.ForkJoinPool.common.parallelism`` system property which will affect every ForkJoinPool creation.


ForkJoinTask
------------
As like Callable and Runnable in ThreadPoolExecutor, fork-join accepts a type of ``ForkJoinTask`` instance for the execution. The abstract class ForkJoinTask is an implementation of ``java.util.concurrecnt.Future`` that provides common functionalities to its subclasses. It has again two abstract subclasses:- `RecursiveAction` and `RecursiveTask` which has only one abstract method called "compute()". There is no difference between these two sub-classes except ``RecursiveTask`` can return result where as ``RecursiveAction`` not.