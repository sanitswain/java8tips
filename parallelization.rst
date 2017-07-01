Parallel Data Processing
========================
Compare to earlier days, cost of the hardwares have been reduced and the number of processors (cores) in modern computers are also increased. Each core has ability to perform operations independently. Before Java 7, processing a collection of data in parallel was extreamly cumbersome. First you have to split the complete data set into sub parts and asks the threads to execute them parallelly. In the last chapter we saw how ForkJoinPool perform these operations more consistently and in less error-prone way. To gain better understanding of prallel processing it is important to know how parallel stream works internally. I will strongly recommend you to go through `fork-join-pool <forkjoin.html>`__ chapter if you have missed it.


Parallel Streams
----------------
A `parallel` Stream is a stream that splits its elements into multiple chunks, process each chunk with different thread. Thus you can automatically partition the workload of a given operation on all the cores of your multicore processor and keep all of them equally busy. Getting parallel stream is very easy, calling ``parallelStream()`` method on collection classes or ``parallel()`` method on sequential stream returns a parallel stream as demonstrated below.

::

  List<String> list = getDataSet();
  list.parallelStream().forEach(System.out::println);
  
  int[] array = {1, 2, 3, 4, 5};
  int sum = Arrays.stream(arr).parallel().sum();

Similarly stream also has ``sequential()`` method that converts parallel stream into sequential stream. In reality stream class maintains an internal boolean state to identify the stream is a parallel stream. Due to this calling `parallel()` and `sequential()` methods multiple times on a stream will not throw any exception. In the below example the last call parallel() will win the priority.

::

  stream.parallel()
     .filter(...)
     .sequential()
     .map(...)
     .parallel()
     .sum();

By now you already have idea that tasks are divided and processed individually in parallel stream. Now it is the time to see how parallel stream internally works. To understand it better we will see following example to find largest element in an array.

.. code:: java
			
    int max = numbers.parallelStream().reduce(0, Integer::max, Integer::max);
    System.out.println("Parallel: " + max);

Here to the reduce method we are passing a BiFunction (2nd argument) which denominates the task to be performed when the task become too small and can not be splitted again. The last argument is a BinaryOperator shows the operation to be perfomed on the two partial results collected from sub tasks. If you want to know about `Stream.reduce` method please visit the `Stream <streamsapi.html#stream-reduction>`__ chapter. Below is the call stack of parallelStream() method.

|
|     parallelStream()
|        StreamSupport.stream(spliterator(), true);
|	        ArrayList.spliterator()
|                ArrayListSpliterator<>();
		

Parallelstream() calls ``spliterator()`` on the collection object which returns a Spliterator implementation that provides the logic of splitting a task. Every source or collection has their own spliterator implementations. Using these spliterators, parallel stream splits the task as long as possible and finally when the task becomes too small it executes it sequentially and merges partial results from all the sub tasks.
	
Spliterator
-----------


Conclusion
----------