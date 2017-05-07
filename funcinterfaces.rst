Functional Interfaces
=====================
In java 8 context, `functional interface` is an interface having exactly one abstract method called `functional method` to which the lambda expression's parameter and return types are matched. Functional interface provides target types for lambda expressions and method references.

The `java.util.function <http://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html>`_ contains general purpose functional interfaces used by JDK and also available for end users like us. While they are not the complete set of funtional interfaces to which lambda expressions might be applicable, but they provide enough to cover common requirements. You are free to create your own functional interfaces whenever existing set are not enough.

The interfaces defined in the this package are annotated with `FunctionalInterface <http://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html>`_. This annotation is not the requirement for the java compiler to determine the interface is an `functional interface` but it helps the compiler to identify the accidental violation of the design intent. Basically I would say this annotation will be very much useful for us while creating our custom functional interfaces. 


@FunctionalInterface rules
--------------------------
As discussed `@FunctionalInterface <http://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html>`_ is a runtime annotation that is used to verify the interface follows all of the rules that can make this interface as `functional interface`. Below are some of the rules from them:

    - Interface must have exactly one abstract method.
	
    - It can have any number of `default methods <http://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Method.html#isDefault-->`_  because they are not abstract and implementation is already provided by same.

    - Interface can declares an abstract method overriding one of the public method from ``java.lang.Object``, that still can be considered as `functional interface`. The reason is any implementation class to this interface will have implementation for this abstract method either from super class (bare minimum java.lang.Object) class or defined by implementation class it self. In the below example ``toString()`` method declared as abstract which will be implemented in its concrete implementation class or at last derived from ``java.lang.Object`` class.

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


Function<T, R>
--------------
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

            toMap(employeeList, empPrimaryId);
            toMap(deptList, deptPrimaryId);
        }

        static <T, R> Map<T, R> toMap(List<T> list, Function<T, R> func) {
            Map<T, R> result = new HashMap<>();
            for (T t : list) {
                result.put(t, func.apply(t));
            }
            return result;
        }
    }

`Function` has couple of default and static methods:

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
     - Returns a function that always returns its input argument. Basically it is a helper method that used in Collector implementation that we will look later.

Below code snippet shows an example of composed function ``andThen()``.

.. code:: java

    public class ComposedFunctionExample {

        /** 
         *  Find the Addrees of given employee from database and return pincode
         */
        public static void main(String[] args) {
            Function<String, Address> first = empid -> EmployeeService.getEmployeesData().get(empid);
            Function<Address, Integer> second = addr -> addr.pincode;
            extract("E101", first, second);
        }

        static <T, R, U> U extract(T input, Function<T, R> first, Function<R, U> second) {
            return first.andThen(second).apply(input);
        }
    }
	
It has two subclasses whose type of operand and return types are of same type.
	
- **UnaryOperator<T>:**
	This represents an operation on a single operand that produces a result of the same type as its operand. The simple usecase could be calculating square of a number.

	*Function descriptor signature:* ``T apply(T t)``
	
	*Example:* UnaryOperator<Integer> square = (Integer in) -> in * in;


- **BinaryOperator<T>:**
	This represents an operation upon two operands of the same type, producing a result of the same type as the operands. The simple usecase could be calculating sum of two numbers.

	*Function descriptor signature:* ``T apply(T t1, T t2)``
	
	*Example:* BinaryOperator<Integer> sum = (i1, i2) -> i1 + i2;

	
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


There is another variant of functional interfaces that starts with **Bi**: BiConsumer, BiFunction, BiPredicate etc which accept two input arguments of same or different reference types. These are helper interfaces used when working with tasks expecting two input arguments as an example ``list.add(element)``. There is no functional interfaces which accepts more than two input parameters, but still you can deal with such problems by wrapping all inputs to a single container.

.. hint:: Suppose you want to replace a CharSequence with another CharSequence within a string. Here you have three input parameters: `original string, search string, replace string`. So you can write them in following ways.

	- Function<String[], String> f1 = arr -> arr[0].replaceAll(arr[1], arr[2]);
	- BiFunction<String, String[], String> f2 = (str, arr) -> str.replaceAll(arr[0], arr[1]);


Primitive Functional Interfaces
-------------------------------
We visited couple of functional interfaces which are defined as generic types. Generic types are always reference type which has extra cost associated with it called `Boxing` and `Unboxing`. Reference types are generally a wrapper around primitive types and stored in heap. Therefore, takes extra space. You might not bother about more space taking though cost of hardware is decreased a lot in last decade, but what about the execution time. When you operate on primitive types, your input and expected return type both are primitives but internally due to generics it boxes your input, does the operation then unboxes the result and returns it. So here the boxing and unboxing is an extra effort that takes phenomenon time which is useless for your purpose. Let's see an example.

.. code:: java

    public class PrimitiveFunc {

        public static void main(String[] args) {
            int[] arr = IntStream.range(1, 50000).toArray();
            BinaryOperator<Integer> f1 = (i1, i2) -> i1 + i2;
    	    IntBinaryOperator f2 = (i1, i2) -> i1 + i2;

    	    RunningTime.calculate((Consumer<Void>) v -> reduce1(arr, f1));
    	    RunningTime.calculate((Consumer<Void>) v -> reduce2(arr, f2));
    	}

    	static int reduce1(int[] arr, BinaryOperator<Integer> operator) {  
    	    int result = arr[0];
    	    for (int i = 1; i < arr.length; i++) {
    	        result = operator.apply(result, arr[i]);  // Boxing and Unboxing here
    	    }
    	    return result;
    	}

        static int reduce2(int[] arr, IntBinaryOperator operator) {
    	    int result = arr[0];
    	    for (int i = 1; i < arr.length; i++) {
    		    result = operator.applyAsInt(result, arr[i]);
    	    }
    	    return result;
        }
    }

    Output:
    reduce1() execution time: 0.006 secs
    reduce2() execution time: 0.002 secs

In the above example `reduce` methods calculating sum of a given array of numbers and output section shows their running times. ``reduce2()`` is 3 times faster than ``reduce1()`` method because it uses ``IntBinaryOperator`` which avoids unnecessary boxing and unboxing operations.

Java8 brings a bundle of primitive functional interfaces that deals with only three primitive types i.e. int, long and double. Basically it follows a naming conventions to identify as them:

- **XXX:** Examples are IntPredicate, IntFunction, DoubleFunction, LongFunction etc. They accept primitive inputs and returns reference type results.
- **ToXXX:** Examples are ToLongFunction, ToIntFunction etc. They accept reference type as input and returns primitive types.
- **XXXToYYY:** IntToDoubleFunction, DoubleToLongFunction are some examples of this. They accept primitive type and also return primitive types.


.. note:: There are little caveats in above rules:
	
	- In case of `Supplier`, XXX type returns primitive type because Supplier doesn't accept any input.
	
	- ToXXX and XXXToYYY are only applicable to them who returns something. Functional interfaces like `Predicate` doesn't have flavours of ToIntPredicate or LongToDoublePredicate because its return type is always boolean.


Method References
------------------
We have learnt enough to build lambda expressions to create anonymous methods. You might come across the scenarios where your lambda expression can contain just one line of code that calls an existing method. In such scenario lambda expressions will look like:

- Function<String, Integer> func = str -> str.length();
- Supplier<Address> sup = () -> emp.getAddress();

Though java8 talks about removing boilerplace codes, there is an efficient way called `method references` to build these lambdas which will be more clear and readable. If we rewrite above two lambda expressions using method reference technique then the representations will be ``String::length`` and ``emp::getAddress``. These representation clearly says we are trying to call length method of a string in first case and getAddess in the second.

**Syntax**: <target reference>::<method name>

Above is the syntax for creating method references where the target reference will be placed before the delimeter :: and then the name of method. There are three kinds of method references exists.

- Reference to static method:
    ``Consumer<List<Integer>> c = Collections::sort;`` is an example of method reference for static methods. Compiler will automatically consider it as ``(list) -> Collections.sort(list)``. Here the target type will be the class name that contains the static method.

- Reference to an instance method of a particular object:
	If you have an object reference then you can call its method like ``list::add`` which is very similar to ``(list, ele) -> list.add(ele)``. Here the target type will be object reference.

- Reference to an instance method of an arbitrary object of a particular type:
	This type of method references are little confusing. If you look into the previous example ``String::length``, usually length() method is called on a string reference but we have written class name "String" as like it is a static method. When we use method references they also go through similar checks as lambda expression goes. Compiler will try to match the method reference with any of functional descriptor syntax and if matches then passes on.

Below table shows some of method references and equal lambda expressions.

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Method Reference
     - Lambda Expression

   * - Integer::parseInt 
     - ToIntFunction<String> f = (str) -> Integer.parseInt(str)
	 
   * - Collections::sort
     - BiFunction<List, Comparator<Trade>> f = (list, comp) -> Collections.sort(list, comp)

   * - String::toUpperCase
     - UnaryOperator<String> f = (str) -> str.toUpperCase()

   * - UUID::randomUUID
     - Supplier<UUID> f = () -> UUID.randomUUID()
	 
   * - empDao::getEmployee
     - Function<String, Employee> f = (empid) -> empDao.getEmployee(empid)


.. important::  There are two things you should be aware of before using method references.

	#. Method reference should not contain paranthesis after method name otherwise it will represent a method invocation that wwill lead to compilation error.
	#. It is difficult to wrte meethod signature until and unless you know the signature of the method for which writing method reference.