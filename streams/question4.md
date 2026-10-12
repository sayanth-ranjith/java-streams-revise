# Question 4 — Split employees into two groups with `partitioningBy`

**Asked to check if you know `partitioningBy` vs `groupingBy`** and how downstream collectors
(`mapping`, `counting`, `averagingLong`) plug into it.

## Question
Using the same employee list, split employees into **salary > 60000** and **salary ≤ 60000**, and find:
1. The **names** in each group
2. The **count** in each group
3. The **average salary** of each group
4. Show what happens when one side is **empty** (salary > 100000) — `partitioningBy` vs `groupingBy`

| id | name    | dept  | salary |
|----|---------|-------|--------|
| 1  | Sayanth | IT    | 50000  |
| 2  | Sourav  | IT    | 70000  |
| 3  | Dhoni   | Sales | 80000  |
| 4  | Sonu    | Sales | 60000  |
| 5  | Virat   | IT    | 70000  |
| 6  | Rohit   | HR    | 45000  |

## Solution
```java
import java.util.*;
import java.util.stream.Collectors;

public class PartitionEmployees {

    record Employee(int id, String name, String dept, long salary) {}

    public static void main(String[] args) {
        List<Employee> employees = List.of(
                new Employee(1, "Sayanth", "IT", 50000),
                new Employee(2, "Sourav", "IT", 70000),
                new Employee(3, "Dhoni", "Sales", 80000),
                new Employee(4, "Sonu", "Sales", 60000),
                new Employee(5, "Virat", "IT", 70000),
                new Employee(6, "Rohit", "HR", 45000));

        long threshold = 60000;

        // 1. Split into two groups: salary > threshold (true) and the rest (false)
        Map<Boolean, List<String>> namesBySalary = employees.stream()
                .collect(Collectors.partitioningBy(
                        e -> e.salary() > threshold,
                        Collectors.mapping(Employee::name, Collectors.toList())));

        // 2. Count in each partition (downstream collector)
        Map<Boolean, Long> countBySalary = employees.stream()
                .collect(Collectors.partitioningBy(
                        e -> e.salary() > threshold,
                        Collectors.counting()));

        // 3. Average salary of each partition
        Map<Boolean, Double> avgBySalary = employees.stream()
                .collect(Collectors.partitioningBy(
                        e -> e.salary() > threshold,
                        Collectors.averagingLong(Employee::salary)));

        // 4. Trap: partitioningBy ALWAYS has both keys, even if one side is empty
        Map<Boolean, List<Employee>> above100k = employees.stream()
                .collect(Collectors.partitioningBy(e -> e.salary() > 100000));

        // 5. Same predicate with groupingBy: the empty key is MISSING
        Map<Boolean, List<Employee>> groupedAbove100k = employees.stream()
                .collect(Collectors.groupingBy(e -> e.salary() > 100000));

        System.out.println("Names  : " + namesBySalary);
        System.out.println("Count  : " + countBySalary);
        System.out.println("Average: " + avgBySalary);
        System.out.println("partitioningBy keys (>100000): " + above100k.keySet()
                + ", true side = " + above100k.get(true));
        System.out.println("groupingBy keys     (>100000): " + groupedAbove100k.keySet()
                + ", true side = " + groupedAbove100k.get(true));
    }
}
```

## Output
```
Names  : {false=[Sayanth, Sonu, Rohit], true=[Sourav, Dhoni, Virat]}
Count  : {false=3, true=3}
Average: {false=51666.666666666664, true=73333.33333333333}
partitioningBy keys (>100000): [false, true], true side = []
groupingBy keys     (>100000): [false], true side = null
```

## Key points to say in the interview
- `partitioningBy(Predicate)` returns `Map<Boolean, List<T>>` — exactly **two** groups, keyed `true` / `false`.
- **Both keys are always present**, even when a side is empty (`true side = []`). With `groupingBy` on the
  same predicate the missing key gives `null` → possible `NullPointerException`. This is the #1 follow-up.
- 2-arg overload `partitioningBy(predicate, downstream)` works like `groupingBy`'s downstream:
  `mapping(...)`, `counting()`, `averagingLong(...)`, `summingLong(...)`, `maxBy(...)`.
- Use `partitioningBy` for a yes/no split (pass/fail, even/odd, active/inactive); use `groupingBy` when
  there are more than two categories (dept, city).
- `Sonu` (exactly 60000) lands in `false` because the predicate is `>` — clarify boundary conditions out loud.
- Classic variant: split a `List<Integer>` into even and odd → `partitioningBy(n -> n % 2 == 0)`.
- Complexity: O(n), single pass.
