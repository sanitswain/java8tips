Quick Introduction
==================
Java 8 released on March 18, 2014 and it became one of the favourite release for most of the java programmers. It has touched almost all part the java: memory management, runtime, syntax parser etc. It will improve your coding experience with different way of writing codes using features like declarative programming style, passing code as an argument etc. Every new releases has its own pros and cons and java 8 has too. When it comes to adopting a new language, the first thing we look for is benifits of it. To be honest some of Java 8 features are little confusing but ultimately they are of the most powerful. Soon we will have more clarity on them but prior to that let's see the main goal of it. Java 8 expects you to write codes that will be more precise, objective driven and highly readable.

Passing methods as parameter removes verbosity from code and in fact increases reusability, Streams helps in writing SQL like syntaxes, parallelization that is almost free- speed of the execution with efficient use of modern computers having multicore processors, handling nullable values using Optional and many more. Initially stuffs will be little confusing but once you used to it, you will be reluctant to write code with out using it. Let's look into the below usecase and understand why java 8 is different then all the releases.

Suppose we are trying to find the highest salary paid in each technology of a XYZ company. Before Java 8 the typical implementation could be
	
::
	
    public Map<String, Double> method2(List<Employee> list) {
        Map<String, List<Employee>> temp = new HashMap<>();
        for (Employee e : list) {
            temp.putIfAbsent(e.getTechnology(), new ArrayList<>());
            temp.get(e.getTechnology()).add(e);
        }

        Map<String, Double> map = new HashMap<>();
        for (Entry<String, List<Employee>> ent : temp.entrySet()) {
            double max = 0;
            for (Employee e2 : ent.getValue()) {
                max = Double.max(max, e2.getSalary());
            }
            map.put(ent.getKey(), max);
        }
        return map;
    }

	
Let's rewrite this code snippet in Java 8 way.

.. code:: java

    Map<String, Double> map = list.stream().collect(
            groupingBy(Employee::getTechnology,                            -- Grouping on technology
                mapping(Employee::getSalary,                               -- Scale to salary from Employee object
                    collectingAndThen(maxBy(Comparator.naturalOrder()),    -- Find the maximum among them
                        Optional::get))));

Isn't it great. I just said group on technologies then extract salary from the employee object and finally get me the highest value from each group. Here my code is objective oriented and easy understandable. If you look into the first approch we are using a temporary intermediate map just to keep grouped data and then process it to find the desired result. Every time you implement this kind of funtionality, you will write these boilerplate codes, but now java does these extra coding and returns result to you. You still might be thinking older approach is good because of the confusions and we are not ready to think in functional programming way.

I am excited to walk you through other features in detail and hope you are too, so let's get started...


.. note:: It's always a best practice to read the official doc so i would suggest you to visit the official java site for more inforrmation.



The topic we are going to cover are:

* What is a functional programming and Functinal interface?
* java.util.stream.Stream and its operations
* Collector and Collectors
* Forkjoin Pool and Spliterators
* How to use parallel streams?
* Nullable values with Optional
* java.util.time package
* Repeatable annotations