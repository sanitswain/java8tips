Lambdas
=======
In previous chapter we were expecting the ``GroupByExperience`` class should be removed and only the method body should be given to our *group()* function, Lambdas are the concise way of writing them. It can be considered as anonymous function which doesn't have any function name but has list of parameters, function body, returns result and even throws exceptions. So let's rewrite the ``GroupByExperience``.

.. code:: java

	(Employee e) -> { return e.yearsOfExpr >= 7 ? "Expert" : e.yearsOfExpr >= 3 ? "Intermediet" : "Fresher"; }

Basically *Lambda* has 3 parts.

#. **A list of parameters** : In above example "Employee e"
#. **Function body**		: The behavior
#. **An arrow**				: It's a separator between parameter list and function body

.. note:: Lambda syntax follows some of the below rules.

	* Parameter types are optional, but if you mention them then brackets are mandatory.
	* If you have single parameter then both parameter type and brackets are optional.
	* If you have multiple parameters, then brackets require.
	* For multiple statement function body, courly braces required.
	* If courly braces present then ``return`` keyward is mandated incase your behavior return something.

With applying above rules our ``GroupByExperience`` class can be writtten as following ways.

	* e -> { return e.yearsOfExpr >= 7 ? "Expert" : e.yearsOfExpr >= 3 ? "Intermediet" : "Fresher"; }
	* e -> e.yearsOfExpr >= 7 ? "Expert" : e.yearsOfExpr >= 3 ? "Intermediet" : "Fresher"

|
|
	Below are some more examples of lambda expressions.
	
	* ``BiConsumer<List, Integer> addIntoList =`` (List list, Integer element) -> list.add(element);
			Adding an element to a given list.
	* ``Predicate<Employee> javaEmp =`` e -> "Java".equals(e.technology);
			Checking an employee is is from Java technology.
	* ``Supplier<Integer> uniqueKey =`` () -> new Random().nextInt();
			Generate a unique number with the help of generator.
	
We saw couple of more lambda expressions above but what are these left hand side classes (BiConsumer, Predicate etc). If you remember in behavior parameterization chapter I mentioned, what if ``Groupable`` interface were given by java it self then we don't have to write our own interfaces or abstract classes to give different different implementations. Java 8 has already identified set of these functional interfaces and included as part of JDK which we are going to cover in next chapter.