# java-streams-revise
```java
//Find the highest salary in each dept.
//A list of Employee class is given to you
@Data
class Employee {
    private int id;
    private String name;
    private String dept;
    private Long salary;
}

class StreamExample {

    public void example() {

        List<Employee> employees = List.of(
            new Employee(1, "Sayanth", "IT", 50000),
            new Employee(2, "Sourav", "IT", 70000),
            new Employee(3, "Dhoni", "Sales", 80000),
            new Employee(4, "Sonu", "Sales", 60000)
        );

        // answer
        // group by department, compare by salary
        employees.stream().
                    collect(Collectors.groupingBy (Employee::getDepartment), 
                    Collectors.maxBy(Comparator.comparing(Employee::getSalary)))
    }
}
```