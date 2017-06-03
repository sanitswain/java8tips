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
Collecting stream elements to a `java.util.Collection` is the most widely used operation. Collectors class provide couple of methods that returns a collector which will then collect stream elements to a specific collection container.

.. list-table::

   * - Collector<T, ?, List<T>> toList()
   * - Collector<T, ?, Set<T>> toSet()
   * - Collector<T, ?, C> toCollection(Supplier<C> collectionFactory)
  
- **toList():**
    Returns a Collector that will accumulate stream elements into ArrayList in the encountered order.

    List<String> list = Stream.of("java", ".net", "python")
                .map(String::toUpperCase).collect(Collectors.toList());

- **toSet():**
    Returns a Collector that will accumulate stream elements into HashSet object.

    Set<String> set = Stream.of("java", ".net", "python")
                .map(String::toUpperCase).collect(Collectors.toSet());

- **toCollection(Supplier<C> collectionFactory):**
    The first two methods returns collectors using ArrayList and HashSet as the container, but in case you need some other Collection implementations then `toCollection` method can be helpful which accept a supplier representing the type of the container to be used for the accumulation process.
	
    TreeSet<String> set = Stream.of("java", ".net", "python").map(String::toUpperCase)
                .collect(Collectors.toCollection(TreeSet::new));


Strings concatenation
---------------------
Collectors utility class provides some of overloaded methods that concatenates stream elements into a single string either by separating them with a delimiter if provided.

.. list-table::

   * - Collector<CharSequence, ?, String> joining()
   * - joining(CharSequence delimiter)
   * - joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)
  
The default delimiter for the no argument ``joining`` method is an empty string. The three argument `joining` method takes prefix and suffix which will be joined in the front and rear end of the final concatenated string result.


.. code:: java

    Stream.of("java", ".net", "python").collect(joining(", ", "Joined String[ ", " ]"));
	
    Output: Joined String[ java, .net, python ]


Grouping elements
-----------------
A common database operation is to group records based on one or multiple columns similarly Collectors also provide factory method that accepts a classification function and returns a Collector implementing a "group by" operation on stream elements T.

The classification function derives grouping keys of type K from stream elements. The collector produces a Map<K, List<T>> whose keys are the values resulting from applying the classification function to the input elements, and values are Lists containing the input elements which map to the associated key under the classification function.

Below is the entity class definition we will be using through out the collector examples.

.. code:: java

    public class Trade {
	
        private String tradeId;
        private String trader;
        private double notional;
        private String currency;
        private String region;
		
        // getters and setters
    }

.. csv-table:: Trade deals
   :header: "Trade Id", "Trader", "Notional", "Currency", "Region"

   "T1001", "John", 540000, "USD", "NA"
   "T1002", "Mark", 10000, "SGD", "APAC"
   "T1003", "David", 120000, "USD", "NA"
   "T1004", "Peter", 4000, "USD", "NA"
   "T1005", "Mark", 300000, "SGD", "APAC"
   "T1006", "Mark", 25000, "CAD", "NA"
   "T1007", "Lizza", 285000, "EUR", "EMEA"
   "T1008", "Maria", 89000, "JPY", "EMEA"
   "T1009", "Sanit", 1000000, "INR", "APAC"

Now let's group the trade deals according to country region.

.. code:: java

    Map<String, List<Trade>> map = 
            trades.stream().collect(Collectors.groupingBy(Trade::getRegion));

    Output:
    {
       APAC: [T1002, T1005, T1009],
       EMEA: [T1007, T1008],
       NA: [T1001, T1003, T1004, T1006]
    }

In the above example we have passed ``Trade.getRegion()`` as the classification function. ``grouping`` method will apply the given classification function to every element T to derive key K and then it will place the stream element into the corresponding map bucket. The grouping operation we just perfomed is very simple and straight-forward example but Collectors also support overloaded factory methods for multi-level grouping such as grouping trade detals according to region and currency.

**groupingBy(Function<T, K> classifier, Collector<T, A, D> downstream):**
This overloaded method accepts an additional downstream collector to which value associated with a key will be supplied for further reduction. The classification function maps elements T to some key type K and generates groups of List<T>. The downstream collector will then operates on each group of elements of type T and produces a result of type D, at last collector will produces a result of Map<K, D>.

Below example shows grouping trade deals according to region and currency. The end result from this example will be ``Map<Region, Map<Currency, List<Trade>>>``.

.. code:: java

    Map<String, Map<String, List<Trade>>> map2 = trades.stream()
        .collect(Collectors.groupingBy(Trade::getRegion, 
                    Collectors.groupingBy(Trade::getCurrency)));
    System.out.println(map2);
	
    Output:
    {
       NA={CAD=[T1006], USD=[T1001, T1003, T1004]}, 
       EMEA={EUR=[T1007], JPY=[T1008]}, 
       APAC={SGD=[T1002, T1005], INR=[T1009]}
    }