Predefined Collectors
=====================
In the previous chapter you got an overal idea on how does collector works and how to implement your own collector. Java-8 has introduced a new utilility class called ``java.util.stream.Collectors`` containing many factory methods which provides most commonly used ``Collector`` implentations. Collectors mainly offers the below functionalities:

- Collecting elements to a `java.util.Collection`
- Joining String elements to a single String
- Grouping elements by custom grouping key
- Partitioning elements into TRUE FALSE group
- Reducing operations
- Summerizing elements

These factory methods can also be combined to generate nested Collector that we will see while moving deeper.

Collecting as collections
-------------------------
Collecting stream elements to a `java.util.Collection` is the most widely used usecase. Collectors class provide couple of methods that returns a collector which will then collect stream elements to a specific collection container.

.. list-table::

   * - Collector<T, ?, List<T>> toList()
   * - Collector<T, ?, Set<T>> toSet()
   * - Collector<T, ?, C> toCollection(Supplier<C> collectionFactory)
  
- **toList():**
    Returns a Collector that will accumulate stream elements into ArrayList in the encountered order.

    List<String> list = Stream.of("java", ".net", "python").map(String::toUpperCase).collect(Collectors.toList());

- **toSet():**
    Returns a Collector that will accumulate stream elements into HashSet object.

    Set<String> set = Stream.of("java", ".net", "python").map(String::toUpperCase).collect(Collectors.toSet());

- **toCollection(Supplier<C> collectionFactory):**
    The above two methods returns only ArrayList and HashSet type collectors, but you might encounter situtaions where your need could be of some other types of Collection. In such situation `toCollection` method can be used which accept a supplier representing the type of the container used for th acculumation process.
	
    TreeSet<String> set = Stream.of("java", ".net", "python").map(String::toUpperCase)
                .collect(Collectors.toCollection(TreeSet::new));
	