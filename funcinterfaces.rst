Functional Interfaces
=====================
In java 8 context, `functional interface` is an interface having exactly one abstract method called `functional method` to which the lambda expression's parameter and return types are matched. Functional interface provides target types for lambda expressions and method references.

The `java.util.function <http://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html>`_ contains general purpose functional interfaces used by JDK and also available for end users like us. While they are not the complete set of funtional interfaces to which lambda expressions might be applicable, but they provide enough to cover common requirements. You are free to create your own functional interfaces whenever existing set are not enough.

The interfaces defined in the this package are annotated with `FunctionalInterface <http://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html>`_. This annotation is not the requirement for the java compiler to determine an interface is an `functional interface` but it helps the compiler to identify the accidental violation of the design intent. Basically I would say this annotation will be very much useful for us while creating our own functional interfaces. 


Functional Interface rules
--------------------------


Enough prose here, now see some of the basic functional interface defined in this package.

Predicate<T>
------------


Consumer<T>
-----------


Function<T,R>
-------------


Supplier<T>
-----------