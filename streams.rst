Streams
=======
Streams are one of the best feature of java 8 that let you write codes in declarative programming style rather than typical imperative programming technique. Declarative programming expect you to mention what you want not how to achieve them. Many of the technologies like unix, database etc are already working on this fashion. In database we write ``SELECT technology, max(salary) from employee group by salary`` and it returns highest salary paid in each technology. In case of unix we just combine group of commands `(ls -l | grep "search string" | sort)` and ask unix to execute the operations.

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

Technically stream is a sequence of elements from a source. Source can be anything like collections, arrays, generator functions or I/O resources etc.


Stream vs Collection
--------------------
Most of the time collections are one of the main source for stream to act on. Stream and collection are used togather, they don't replace each other. Streams differ from collection in several ways:

- **No storage:** Collections are typically physical set of data where as streams are a logial view that will be supplied to a pipeline of operations. Collections are about data and streams are about computations.

- **Functional in nature:** An operation on a stream produces a result, but does not modify its source. For example, if we call filtering on a stream it will return a new stream rather than removing them from the original collection.

- **Lazyness execution:** Many of stream operation like filtering, mapping etc are chained togather and executed in one shot using a terminal operation. This technique helps to create optimized execution strategy to process the operations. For example, to find first three odd numbers from a stream it doesn't go through the complete data set and halts the execution once three values found.

- **Possibly unbounded:** While collections have a finite size, streams need not. Short-circuiting operations such as limit(n) or findFirst() can allow computations on infinite streams to complete in finite time.

- **Consumable:** The elements of a stream are only visited once during the life of a stream. Like an Iterator, a new stream must be generated to revisit the same elements of the source.