Predefined Collectors
=====================
In the previous chapter you got an overal idea on how does collector works and how to implement your own collector. Java-8 has introduced a new utilility class called ``java.util.stream.Collectors`` containing many factory methods which provides most commonly used ``Collector`` implentations. Collectors mainly offers the below functionalities:

- Collecting elements to a `java.util.Collection`
- Joining String elements to a single String
- Grouping elements by custom grouping key
- Partitioning elements into TRUE FALSE group
- Some of reducing operations
- Summerizing elements

These factory methods can also be combined to generate nested Collector that we will see while moving deeper.

