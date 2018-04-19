.. raw:: html

    <embed>
		<link rel="stylesheet" href="_static/mystyle.css">
    </embed>

Handling nulls with Optional
============================
Tony Hoare-one of the giants of computer science once told, "`I call it my` **billion-dollar mistake**. `It was the invention of the null reference in 1965. I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement.`" The null reference is the source of many problems because it is often used to denote the absence of a value. As a java developer you would have felt the pain of getting NullPointerException. Millions of projects are running using Java and guess the total amount of dollars spent on fixing those issues.

Imagine a job portal maintaining candidate database with the following nested object structure.

.. code:: java

   class Candidate{
      String name;
      String spouse;
	  
      jobProfile: class Job{
          int yearsOfExpr;
		  
          uiExperience: class Framework{
             String name;
             String proficiency;
			 
             certification: class Certification{
                String type;
                double score;
             }
          }
      }
   }

The structure looks pretty reasonable, but a candidate can be a recent college passout for whom job-profile could be missing, or for a candidate doing job might not have worked on any UI (User Interface) frameworks or even worked on might not be certified. All of these scenarios are proven to generate NullPointerException if not carefully handled in the code.

The simplest solutions depelopers are adopting is the defensive approach of having ``null`` checks.

.. code:: java

   String uiProficiency(Candidate candidate){
       String proficiency = null;
       Job job = candidate.getJobProfile();
       if(job != null){
           if(job.getUiExperience() != null){
               String expr = job.getUiExperience().proficiency;
               proficiency = min(expr, "Intermediet");   
               Certification cert = ui.certification;
               if(cert != null && cert.score >= 60){
                  proficiency = "Expert";
               }
           }
       }
       return proficiency;
   }

No doubt that code will run without any issues but it still has following problems: 

- It looks wierd and looses the readability.
- Repeated low levels codes such as ``if`` conditions.
- Maintainability will be difficult as nesting levels goes on.

Java8 has introduced ``java.util.Optional`` class to deal with such problems and additionally provides some utility methods that can be used in certain common scnearios. `Optional` is a container or a wrapper class that represents value might or might not exist for a variable. When value present you can use ``get`` method to fetch the value or on absent it just behaves as an empty container. We get exception when we directly operate on the null instances so Optional promotes to use its utility methods to perform operations. With keeping things in mind we can reimplement the original 'Candidate' model class given below.

.. code:: java

   class Candidate {
      String name;
      Optional<String> spouce;
      Optional<Job> jobProfile;	
   }

   class Job {
      int yearsOfExpr;
      Optional<Framework> ui;
   }

   class Framework {
      String name;
      String proficiency;
      Optional<Certification> certification;
   }

   class Certification {
      String type;
      double score;
   }

The benifits over using Optional are:

- No need to document separetely to represent nullable members, model class Optional types are self documentary. As an example `spouse` and `jobProfile` clearly mentions that they can be null.
- No need to write null checks explicitely, operations will be performed only if value is present.


Optional Construction
---------------------
Creating optional objects are damn easy, it provides following factory methods to create Optionals.

- **Empty Optional:**

 ``Optional.empty()`` gets you an hold of empty optional object. The default values for the nullable members of an object can be of this type which passed to some other code won't through NullPointerException and will supress any operation performed on it. Even though ``Option.empty() == Option.empty()`` returns true, Optional promotos to use ``isPresent`` method to perform the equility operation.

 ``Optional<Job> optJob = Optional.empty();``

 .. note:: Optional.empty() always returns a singleton Optional instance so equility check will always derive true.
 
- **Optional from nullable value:**
 
 You can create an optional object from a nullable value using the static factoy method ``Optional.ofNullable``. The advantage over using this method is if the given value is null then it returns an empty optional and rest of the operations performed on it will be supressed.
 
 ``Optional<Job> optJob = Optional.ofNullable(candidate.getJobProfile());``


- **Optional from non-null value:**

 You can also create an optional object from a non-null value using the static factoy method ``Optional.of``. Use this method if you are sure given value is not null otherwise it will immidietely throw NullPointerException.
 
 ``Optional<Job> optJob = Optional.of(candidate.getJobProfile());``
 
 There is no other difference in using ``Optional.of`` or ``Optional.ofNullable`` except `of()` methods creates the perception that given value is mandatory field and passing null is the unaccepted criteria.

 
 .. note:: Most of languages has concept of missing values and they handle it in different ways. Scala has a safe way to navigate through values, Google's Guava library and Groovy language has same construct as Java Optional, so we can say java Optional can be inspired from them.

 
Operating on Optionals
----------------------
Optional provides three basic methods: `map, flatMap` and `filter` to perform any kind of common task. Like Streams these operations can also be chained togather to perform composite tasks.

- **map(Function<T, U> mapper):**

 If a value is present, apply the provided mapping function to it, and return an Optional describing the result, otherwise return an empty Optional. Similar to `Stream.map` method, this is also commonly used as transformation function.

 This method supports post-processing on optional values, without the need to explicitly check for a return status. For example, the following code snippet traverses a stream of trades, selects first APAC trade encountered, and then returns the trade id, returning an Optional<String>: 

 .. code:: java
 
      Optional<String> opt = trades.stream()
          .filter(trade -> "APAC".equals(trade.getRegion()))
          .findFirst()
          .map(Trade:::getTradeId);


- **flatMap(Function<T, Optional<U>> mapper):**
 
 Before getting through `flatMap` method let's try an example to find the UI certification done by a candidate who is having a job.
 
 .. code:: java
 
   String certificationName = candidate.getJobProfile()
        .map(Job::getFramework)
        .map(Framework::getCertification)
        .orElse(null);
 
 Yeah pretty easy, but unfortunately this code will not compile at all. The reason is ``Job.getFramework()`` returns ``Optional<Framework>`` type so the first `map` method will produce Optional<Optional<Framework>>. This return value will again input for the second map method and we know that ``getCertification`` method exist in 'Framework' class not in 'Optional<Framework>'.
 
 .. figure:: _static/mapissue.png
   :align: center
   :width: 600px
   :height: 200px

 The solution here is to replace the map methods with `flatMap` method. This method is similar to map method but flatMap expects the mapper function return type already to be Optional which will be directly returned as the final result. If you notice map & flatMap methods internal implementations, map method will wrap mapper calculated result inside a Optional object where as flatMap method will not.
 
  .. figure:: _static/flatmap1.png
   :width: 600px
   :height: 170px

 Below snippet is the correct solution for our earlier example.
 
 .. code:: java
 
    String certificationName = candidate.getJobProfile()
        .flatMap(Job::getFramework)
        .flatMap(Framework::getCertification)
        .orElse(null);
 
 We saw map and flatMap methods in details, now I will show you a nice usage by combining both methods that will be used often. Imagine there is a external service which calculates the reimbursement amount.
 
 .. code:: java
 
    double calculate(Optional<Framework> optFrm, Optional<Certification> optCert) {
        return optFrm.flatMap(framwork -> 
            optCert.map(certification -> reimburse(framwork, certification)))
            .get();
    }
	
 Here the map method is called inside flatMap just for the availability of framework value to invoke `reimburse`. Originally reimbursement will be executed by map method and flatMap will just return calculated result.
 
 
- **filter(Predicate<T> predicate):**

 If the value matches the given predicate, then the Optional containing the value will be returned, otherwise an empty Optional.

 .. code:: java
 
   boolean isCertified = candidate.getJobProfile()
        .flatMap(Job::getFramework)
        .flatMap(Framework::getCertification)
        .filter(certification -> certification.score >= 60)
        .isPresent();

		
Retrieving from Optionals
-------------------------
Optional provides following methods to retrive values from optional object.

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Method
     - Description

   * - get()
     - Returns the value wrapped by the Optional or throws NoSuchElementException 
       if doesn't contain data. Use this method if you are sure optinal holding data.
	 
       ``int years = Optional.of(job).map(Job::getYearsOfExpr).get()``

	   
   * - orElse(default_value)
     - Return the value if present, otherwise default_value.
       This method is the safest way to get the value.
	 
       ``String spouse = Optional.of(candidate).map(Candidate::getSpouse).orElse(null)``

	   
   * - orElseGet(Supplier<T> other)
     - Return the value if present, otherwise retrived from supplier.
       This is the lazy way to retrive value. As an example call an external service in case value not exist in optional.
       If you use `orElse(external_service)` then the service will be executed irrespective of the original value exist which can impose additional cost.
	 
       ``Optional.of(trade).map(Trade::getId).orElseGet(UUID::randomUUID)``

	   
   * - orElseThrow(Supplier<Throwable> exception)
     - Return the value if present, otherwise throw supplied exception.
       Using `get` method will always return NoSuchElementException but custom exceptions could be returned using this.
	 
       ``String spouse = Optional.of(candidate).orElse(YourException::new)``	   

	   
   * - isPresent()
     - Returns true if value present Otherwise false.
	 
       ``boolean cond = Optional.of(job).isPresent()``

	   
   * - ifPresent(Consumer<T> consumer)
     - If a value is present, invoke the specified consumer with the value, otherwise do nothing.
	 
       ``Optional.of(job).ifPresent(System.out::println)``
	   
	   
Miscellaneous
-------------
**Primitive Optionals:**

As like streams, Optionals also have primitive flavours- OptionalInt, OptionalLong and OptionalDouble. These primitives should be used when operating on streams otherwise its usage is discouraged. Stream can contain huge number of elements where using primitives can save time as well as space but an Optional can have at most single value and primitive optional will not employ much difference in fact it will stop you to use common methods like map, filter etc.


**Optionals can't be serialized:**

Optionals were designed to handle missing values. These were not intended for use as a field type so it doesn't implement Serializable. In case you need to have a serializable domain model, implement getter methods returning optionals given below.

.. code:: java

   class Candidate {
      String name;
      String spouce;
      
      public Optional<String> getSpouse(){
         return Optional.ofNullable(spouse);
      }
   }