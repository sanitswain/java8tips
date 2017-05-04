Functional Interfaces
=====================
In java 8 context, `functional interface` is an interface having exactly one abstract method called `functional method` to which the lambda expression's parameter and return types are matched. Functional interface provides target types for lambda expressions and method references.

The `java.util.function <http://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html>`_ contains general purpose functional interfaces used by JDK and also available for end users like us. While they are not the complete set of funtional interfaces to which lambda expressions might be applicable, but they provide enough to cover common requirements. You are free to create your own functional interfaces whenever existing set are not enough.

The interfaces defined in the this package are annotated with `FunctionalInterface <http://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html>`_. This annotation is not the requirement for the java compiler to determine the interface is an `functional interface` but it helps the compiler to identify the accidental violation of the design intent. Basically I would say this annotation will be very much useful for us while creating our custom functional interfaces. 


Functional Interface rules
--------------------------
As discussed `@FunctionalInterface <http://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html>`_ is a runtime annotation that is used to verify the interface follows all of the rules that can make this interface as `functional interface`. Below are some of the rules from them:

    - Interface must have exactly one abstract method.
	
    - It can have any number of `default methods <http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#isDefault-->`_  because they are not abstract and implementation is already provided by same.

    - Interface can declares an abstract method overriding one of the public method from ``java.lang.Object``, that still can be considered as `functional interface`. The reason is any implementation class for this interface will have implementation for this abstract method either from super class (bare minimum java.lang.Object) class or defined by implementation class it self.

Below code snippet is a simple example of functinal interface.

.. code:: java
    
    @FunctionalInterface
    public interface MyFunctionalInterface {

        public abstract void execute();

        @Override
        String toString();

        default void beforeTask() {
            System.out.println("beforeTask... ");
        }

        default void afterTask() {
            System.out.println("afterTask... ");
        }
    }


Enough prose here, now see some of the basic functional interfaces defined in this package.

Predicate<T>
------------
`java.util.function.Predicate <http://docs.oracle.com/javase/8/docs/api/java/util/function/Predicate.html>`_ has a boolean-valued function that takes an argument and returns boolean value.

+---------------------------------------------------+ 
|     Class definition                              | 
+===================================================+ 
|  public interface Predicate<T> {                  |
|                                                   |
|    boolean test(T t);  // functional descriptor   |
|                                                   |
|  }                                                |
+---------------------------------------------------+

| Simple usecase of Predicate can be identifying all odd numbers from a given set or finding java employees from list of employees etc.

Example:

.. code:: java

    public class PredicateTest {

        public static void main(String[] args) {
            Predicate<Integer> oddNums = (num -> num % 2 == 0);
            Predicate<Integer> positiveNums = (num -> num > 0);

            Integer[] array = IntStream.rangeClosed(-10, 10).boxed().toArray(Integer[]::new);

            filter(array, oddNums);
            filter(array, positiveNums);
        }

        public static <T> List<T> filter(T[] array, Predicate<T> predicate) {
            List<T> result = new ArrayList<>();
            for (T t : array) {
                if (predicate.test(t))
                    result.add(t);
            }
            return result;
        }
    }

Here if you see `filter` method accepts a Predicate which is calling its test() method to extract the desired result. Later if you want find all primary numbers then you prepare another predicate and pass it to filter method.

It has couple of default methods which you can use it:

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Method
     - Description
     - Example

   * - and(Predicate<? super T> other) 
     - Returns a composite predicate that represents logical AND of two predicates (P1 AND P2)
     - Predicate<Integer> positiveOdd = positiveNums.and(oddNums)

   * - or(Predicate<? super T> other)
     - Returns a composite predicate that represents logical OR of two predicates (P1 OR P2)
     - Predicate<Integer> positiveOrOdd = positiveNums.or(oddNums)

   * - negate()
     - Returns a predicate that represents the logical negation of this predicate.
     - Predicate<Integer> negative = positiveNums.negate();
	


Consumer<T>
-----------
`java.util.function.Consumer <http://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html>`_ accepts an argument and returns no result.

+----------------------------------------+ 
|     Class definition                   | 
+========================================+ 
|  public interface Consumer<T> {        |
|                                        |
|    void accept(T t);                   |
|                                        |
|  }                                     |
+----------------------------------------+

| Simple usecase can be persisting elements of a collection into DB or serializing them or printing on the console.

.. code:: java

    public class ConsumerTest {
	
        public static void main(String[] args) {
            Consumer<Employee> printOnConsole = (e -> System.out.print(e));
            Consumer<Employee> storeInDB = (e -> DaoUtil.save(e));
			
			forEach(empList, printOnConsole);
			forEach(empList, storeInDB);
			forEach(empList, printOnConsole.andThen(storeInDB));
        }

        static <T> void forEach(List<T> list, Consumer<T> consumer) {
            int nullCount = 0;
            for (T t : list) {
                if (t != null) {
                    consumer.accept(t);
                } else {
                    nullCount++;
                }
            }
            System.out.printf("%d null entries found in the list.\n", nullCount);
        }
    }

Consumer has also one default method called `andThen(Consumer<? super T> after)` which returns a composite consumer where second consumer will be executed after execution of first one. If the first consumer throws any exception then the second consumer will not be executed because non of the functional interfaces provided by JDK handles any exception.


Function<T,R>
-------------
`java.util.function.Function <http://docs.oracle.com/javase/8/docs/api/java/util/function/Function.html>`_ accepts an argument and returns result.

+----------------------------------------+ 
|     Class definition                   | 
+========================================+ 
|  public interface Function<T, R> {     |
|                                        |
|    R apply(T t);                       |
|                                        |
|  }                                     |
+----------------------------------------+

A usecase of `Function` can be extracting employee name from Employee class or deriving primary ids from given object etc.

.. code:: java

    public class FunctionTest {

        public static void main(String[] args) {
            Function<Employee, String> empPrimaryId = (emp -> emp.getEmployeeId());
            Function<Department, String> deptPrimaryId = (dept -> dept.getLocationCode() + dept.getName());

            map(employeeList, empPrimaryId);
            map(deptList, deptPrimaryId);
        }

        static <T, R> Map<T, R> map(List<T> list, Function<T, R> func) {
            Map<T, R> result = new HashMap<>();
            for (T t : list) {
                result.put(t, func.apply(t));
            }
            return result;
        }
    }

It has couple of other methods:

.. list-table::
   :widths: 35 65
   :header-rows: 1

   * - Method
     - Description

   * - compose(Function<? super V, ? extends T> before) 
     - Returns a composed function that first applies the before function to its input, and then applies this function to the result.

   * - andThen(Function<? super R, ? extends V> after)
     - Returns a composed function that first applies this function to its input, and then applies the after function to the result.

   * - static <T> Function<T, T> identity()
     - Returns a function that always returns its input argument.
	
Supplier<T>
-----------
`java.util.function.Supplier <http://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html>`_ doesn't accept any argument but returns a result.

+------------------------------------+ 
|     Class definition               | 
+====================================+ 
|  public interface Supplier<R> {    |
|                                    |
|    R get();                        |
|                                    |
|  }                                 |
+------------------------------------+

A simple usecase of Supplier can be generating unique numbers using various algorithms.

.. code:: java

    public class SupplierTest {

        public static void main(String[] args) {
            Supplier<Long> randomId = () -> new Random().nextLong();
            Supplier<UUID> uuid = () -> UUID.randomUUID();

            Trade trade = new Trade();
            populate(trade, randomId);
            populate(trade, uuid);
        }

        static <R> void populate(Trade t, Supplier<R> supplier) {
            t.tradeDate = new Date();
            t.tradeId = (String) supplier.get();
            t.location = "XYZ Hub";
        }

        static class Trade {
            String tradeId;
            Date tradeDate;
            String location;
        }
    }