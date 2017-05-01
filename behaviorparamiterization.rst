Behavior Parameterization
==========================
`Behavior parameterization` is the ability of a method to receive multiple different behavior as its parameter and use them internally to accomplish the task. It let you to make your code more adaptive to changing requirements and saves the engineering effort from writing similar piece of code here and there. 

If you have come across some of the behavioral design patterns like *Strategy Pattern* then you know we create set of similar algorithms and choose required one at run time to deal with a certain problem scenario. This type of techniques facilitate to add new behaviors in future. Let's look into a problem statement to understand better.

Suppose a company XYZ is trying to group its employees based on certain criterias like proficiency level, technology type, gender etc or new criterias can be added in future. So to solve this problem we will create family of grouping algorithms as described below.

.. code:: java

    interface Groupable {
        public String findGroup(Employee e);
    }

    class GroupByExperience implements Groupable {
	
        @Override
        public String findGroup(Employee e) {
            return e.yearsOfExpr >= 7 ? "Expert" : 
                e.yearsOfExpr >= 3 ? "Intermediet" : "Fresher";
        }
    }

    class GroupByTechnology implements Groupable {

        @Override
        public String findGroup(Employee e) {
            Map<String, List<String>> mapping = new HashMap<String, List<String>>() {
                {
                    put("Front-end", Arrays.asList("AngularJS", "ExtJS"));
                    put("Middleware", Arrays.asList("Java", ".Net"));
                    put("Back-end", Arrays.asList("Oracle", "MySQL", "PostgreSQL"));
                }
            };

            for (Entry<String, List<String>> entry : mapping.entrySet()) {
                if (entry.getValue().contains(e.technology)) {
                    return entry.getKey();
                }
            }
            return "Others";
        }
    }

Based on our purpose we are passing the required behaviors to the grouping function which is just creating groups.
	
.. code:: java

    public Map<String, List<String>> group(List<Employee> list, Groupable behavior) {
        Map<String, List<String>> map = new HashMap<>();
        for (Employee e : list) {
            String group = behavior.findGroup(e);
            map.putIfAbsent(group, new ArrayList<>());
            map.get(group).add(e.name);
        }
        return map;
    }

Great... We solved the problem, as and when new requirements comes we just need to provide some other implementation classes. But from begining we are talking, one of our main objective is to remove verbosity from the code as well as maintain the understandability. If you look into the ``GroupByExperience`` class, the behaviour is of one liner but still complete class has been written. Another way can be writting Anonymous classes which some what reduces these boilerplate codes but not to the great extent.

Just think, if the interface ``Groupable`` was given by java SDK itself and we were written only the method and passed to the grouping function, then the code will be clearer and more flexible. Some of the interpreted langauges like python, JavaScript etc support passing of method as parameter to the calling function, similarly Java 8 has also started supporting it with the help of *Functional Interfaces* and *Lambdas*. Most of us already aware of Lambdas which is a very well-known concept that exist from the begining of languages like python. Don't worry about them now, we will slowly have deep drive into it. 