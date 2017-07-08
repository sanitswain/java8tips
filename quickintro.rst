Quick Introduction
==================
Java 8 launched on 18th March 2014 and it was a next major release after jdk5. It came up with large set of scintillating features that has won the attention of most of the java programmers. It has enhanced various java components like runtime environment, compiler, lexical parser, memory management, command line tools and many more. Java 8 will improve programmer's coding experience with its enticed features of declarative programming, passing code as an argument, method reference, optional for handling null etc. It will assure you to write codes that will be more precise, objective driven and highly readable.

Passing methods as parameter removes verbosity from the code and in fact increases reusability, Streams helps in writing SQL like syntaxes, parallelization that is almost free:- `speed of the execution with efficient use of modern computers having multicore processors`, handling nullable values using Optional and many more. Initially stuffs will be little confusing but once you used to it, you will be reluctant to write code with out using it. Let's look into the below usecase and understand why java 8 is different then all the releases.

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

Isn't it great. I just said "group on technologies" then extract salary from the employee object and finally get me the highest value from each group. Here my code is objective oriented and easy understandable. If you look into the first approch we are using a temporary intermediate map just to keep grouped data and then process it to find the desired result. Every time you implement this kind of funtionality, you will write these boilerplate codes, but now java does these extra coding and returns result to you. You still might be thinking older approach is good because of the confusions and we are not ready to think in functional programming way.

I am excited to walk you through other features in details and hope you too, so let's get started. Below are the topics we will cover in this tutorial.

* What is a functional programming and Functinal interface?
* java.util.stream.Stream and its operations
* Collector and Collectors
* Forkjoin Pool and Spliterators
* How to use parallel streams?
* Nullable values with Optional
* java.util.time package
* Repeatable annotations