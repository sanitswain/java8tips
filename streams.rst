..  _stream_basics:

Introduction to Streams
=======================
Streams are one of the bigwig among java8's released features that let you write codes in declarative style rather than typical imperative programming technique. Declarative programming expect you to mention what you want not how to achieve them. Many of the technologies like unix, database etc are already working on this fashion. In database we write ``SELECT technology, max(salary) from employee group by salary`` and it returns highest salary paid in each technology. In case of unix we just combine group of commands `(ls -l | grep "search string" | sort)` and ask unix to execute the operations.

Just look into the below example.

.. code:: java

    public static void main(String[] args) {
        List<Trade> trades = TradeData.allTrades();
        Comparator<Trade> comparator = Comparator.comparing(Trade::getNotional);
        List<String> naTrades = trades.stream()
            .filter(trade -> Region.NA.equals(trade.getRegion()))
            .sorted(comparator)
            .map(Trade::getTradeId)
            .collect(toList());
        System.out.println(naTrades);
    }

In the above code snippet we just created a pipeline of tasks and java 8 will prepare the execution strategy internally to process it. Here we didn't write any external ``foreach`` loop to traverse through all the elements and it will be internally taken care. If you wish to process trades parallely no need to write any extra milti-threaded code to do it, just replacing the `stream()` method with `parallelStream()` will handle the whole parallelism process. Don't wory about parallelism now, we will look into it later.

.. note:: Technically stream is a sequence of elements from a source. Source can be anything like collections, arrays, generator functions or I/O resources etc.


Stream vs Collection
--------------------
Most of the time collections are one of the main source for stream to act on. Stream and collection are used togather, they don't replace each other. Streams differ from collection in several ways:

- **No storage:** Collections are typically physical set of data where as streams are a logial view that will be supplied to a pipeline of operations. Collections are about data and streams are about computations.

- **Functional in nature:** An operation on a stream produces a result, but does not modify its source. For example, if we call filtering on a stream it will return a new stream rather than removing them from the original collection.

- **Lazyness execution:** Many of stream operation like filtering, mapping etc are chained togather and executed in one shot using a terminal operation. This technique helps to create optimized execution strategy to process the operations. For example, to find first three odd numbers from a stream it doesn't go through the complete data set and halts the execution once three values found.

- **Possibly unbounded:** While collections have a finite size, streams need not. Short-circuiting operations such as limit(n) or findFirst() can allow computations on infinite streams to complete in finite time.

- **Consumable:** The elements of a stream are only visited once during the life of a stream. Like an Iterator, a new stream must be generated to revisit the same elements of the source. If the source is FileInputStream etc, then you are out of luck because inputstream will be closed once consumed and you cann't regenerate the stream.

Stream sources
--------------
In above section we saw collections and InputStream are two sources for streams. There are numerous other sources as well from where you can generate the stream.

* From a Collection via the `stream()` and `parallelStream()` methods;
* From an array via `Arrays.stream(T[])`;
* From static factory methods on the stream classes, such as `Stream.of(T[]), IntStream.range(int, int) or Stream.iterate(T, UnaryOperator)`;
* The lines of a file can be obtained from `BufferedReader.lines()`;
* Streams of file paths can be obtained from methods in `Files`;
* Streams of random numbers can be obtained from `Random.ints()`;

Apart from all of these predefined sources, you can also generate stream from your custom source using StreamSupport class. Example:

.. code:: java

    public class TradePool {
        List<Trade> list;

        public Stream<Trade> stream() {
            return StreamSupport.stream(list.spliterator(), false);
        }
    }

StreamSupport has some low-level methods which expects you to provide a spliterator that will generate stream. As of now don't worry about this spliterator, just think it is an iterator we will cover this spliterator once we are ready to go for parallelization because you need to understand ForkJoinPool better to know about it.

Stream Operations
-----------------
Stream operations are broadly divided into intermediate and terminal operations that are combined to form pipeline. A stream pipeline consists of a source (such as a Collection, an array, a generator function, or an I/O channel); followed by zero or more intermediate operations such as Stream.filter or Stream.map; and a terminal operation such as Stream.forEach or Stream.reduce.

.. image:: _static/stream_ops.png
   :align: center
   :alt: Stream Operations


- **Intermediate Operations:** Intermediate operations helps the stream pipeline to build the execution strategy. These are lazy in nature, they don't execute until any terminal operations are invoked. They don't modify the original stream, everytime they return a new stream. Intermediate operations can again divided into stateless and stateful operations.
	
    - `Stateless` operations such as filter, map are processed independently of operations on other elements
    - `Stateful` operations such as sorted, distinct require to rememeber the result of operations on already seen elements to calculate the result for next element. They execute the entire input before producing the final result.

- **Terminal Operation:** Terminal operation traverse the stream and execute the pipeline of intermediate operations to produce the result. They are eager in nature. After the terminal operation is performed, the stream pipeline is considered consumed, and can no longer be used. A stream implementation may throw IllegalStateException if it detects that the stream is being reused.

Streams are also generated from infinite dataset. Some of the stream operations can be tagged as `short-circuting operations` which acts on these infinite stream or data. An intermediate operation is said to be short-circuting if applying on infinite stream should produce finite stream. As an example ``new Random().ints().limit(5)`` will return only 5 random numbers. A terminal operation is short-circuting if, when applying on infinite set of input should produce result in finite time. As an example ``new Random().ints().filter(no -> no % 10 == 0).findAny()`` will return any one random number divisible by 10.