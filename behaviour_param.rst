Behavior Parameterization
==========================
Generally speaking behavior parameterization is the ability of a method to receive multiple different behavior as its parameter and use them internally to accomplish the task. It let you to make your code more adaptive to changing requirements and saves the engineering effort from writing similar piece of code here and there. 

If you have come across some of the behavioral design patterns like *Strategy Pattern* then you know we create set of similar algorithms and choose required one at run time to deal with a certain problem scenario. This type of techniques facilitate us to add new behaviors in future. Let's look into a problem statement to understand more on this.

Suppose your company is trying to group its employees based on certain criterias like proficiency level, technology type, gender etc or new criterias can be added in future. So to solve this problem we have to create family of grouping algorithms as describing below.

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

Great!!! We have solved our proble, as and when new usecases comes we just have to implement another bahavior that's all. But from begining we are talking our ne of the main goal is to remove verbosity from the code as well as make it understandable.