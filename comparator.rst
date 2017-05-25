Comparator
==========
Comparator is there since jdk 1.2 and almost all of us know how important it is. Java-8 came up with couple of updates to the Comparator operations given below.

- Additionl default and static methods added into ``Comparator`` interface to support pipeline of stream operations.
- ``String`` class added with ``CaseInsensitiveComparator`` to sort by ignoring the case.
- A new utility class ``Comparators`` is bundled with jdk-8 to support natural ordered sorting and handling ``null`` values while sorting.


Comparators
-----------
Comparators is a helper class that provides new Comparator implmentations in the following cases.

	- To impose natural ordrered sorting on elements of Comparable types
	- Sorting on the collections that mixed with null values.

**Natural ordering:**

.. code:: java

    NaturalOrderComparator implements Comparator<Comparable<Object>> {

        @Override
        public int compare(Comparable<Object> c1, Comparable<Object> c2) {
            return c1.compareTo(c2);
        }
    }

``Comparator`` interface contains a static method called ``naturalOrder`` which returns a ``NaturalOrderComparator`` that imposes sorting on elements implementing ``Comparable``. As you know all wrapper classes for primitive types implements Comparable interface so this natural ordered sorting can be applicable to all of them.  


**Handling null elements:**

Usually comparators throws ``NullPointerException`` if null elements found while performing sorting operation. ``Comparator`` contains two methods `nullsFirst` and `nullsLast` that takes a comparator as an argument and returns another null-friendly comparator by wrapping the given comparator. It will arrange null elements at the begining or end depending on the operation you called. For non-null elements it will sort them using the comparator passed initially.



Updates in Comparator
---------------------
Comparator interface contains some of static methods that returns another comparator implementations described below.

- **comparing(Function<T,U> keyExtractor)**
    This method uses the given key extracting function that applies on T type elements to generate U type comparable sort keys. To compare two elements of type T, it first apply the function to both the elements and then performs the sorting operation on the resulted keys.

.. code:: java

        // Sorting words based on word lengths
        Stream.of("java-8", "is", "great", "technology")
            .sorted(Comparator.comparing(String::length))
            .forEach(System.out::println);


- **comparing(Function<T,U> keyExtractor, Comparator<U> keyComparator)**
    In the first ``comparing`` method, key extracting function returns sorting keys of ``Comparable`` type so it doesn't need additional Comparator object to perform sorting. But in this ``comparing`` function it first uses the key extracting function to generate key and then performs sorting based on the given comparator.

.. code:: java

    Stream.of("java-8", "is", "great")
        .sorted(Comparator.comparing(String::length, Comparator.reverseOrder()))
            .forEach(System.out::println);


- **comparingXXX(ToXXXFunction<T> keyExtractor)**
    Comparator interface provides three primitive comparing functions: `comparingInt, comparingDouble and comparingLong` to sort the elements based on the primitive keys. It accepts ToXXXFunction functional interface which returns primitive values that avoid unnecessary boxing-unboxing costs while doing sorting.

.. code:: java

   // Natural order sorting by ignoring the sign.
   Stream.of(-10, 31, 16, -5, 2)
       .sorted(Comparator.comparingInt(i -> Math.abs(i)))
       .forEach(System.out::println);


- **thenComparing(Comparator<T> other)**
    It is very much possible that two elements will be equal according to the given comparator. In such cases the other comprator decides the sorting order. Below code snippet shows example of sorting employee objects based on employee's salary and then uses name if two salaries are equal.

.. code:: 

    List<Employee> employees = Application.getEmployees();
    employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary).thenComparing(Employee::getName))
        .forEach(System.out::println);