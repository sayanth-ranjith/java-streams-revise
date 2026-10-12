# Question 3 — Second highest (and Nth highest) salary

**One of the most asked Java Streams questions** — it checks `distinct()`, reverse sorting, `skip()`,
`Optional`, and whether you handle **duplicate salaries** correctly.

## Question
Given a list of employees, find:
1. The **second-highest distinct salary**
2. **Who** earns it
3. The **Nth highest** salary (generalise it)
4. Bonus: the second-highest salary **in each department**

Input (note: `70000` appears twice on purpose):

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

public class SecondHighestSalary {

    record Employee(int id, String name, String dept, long salary) {}

    public static void main(String[] args) {
        List<Employee> employees = List.of(
                new Employee(1, "Sayanth", "IT", 50000),
                new Employee(2, "Sourav", "IT", 70000),
                new Employee(3, "Dhoni", "Sales", 80000),
                new Employee(4, "Sonu", "Sales", 60000),
                new Employee(5, "Virat", "IT", 70000),   // duplicate salary on purpose
                new Employee(6, "Rohit", "HR", 45000));

        // 1. Second-highest DISTINCT salary
        Optional<Long> secondHighest = employees.stream()
                .map(Employee::salary)
                .distinct()                                   // 70000 appears twice
                .sorted(Comparator.reverseOrder())            // highest first
                .skip(1)                                      // drop the top one
                .findFirst();                                 // Optional: list may have < 2 salaries

        // 2. Employees earning that salary
        List<String> earners = secondHighest
                .map(s -> employees.stream()
                        .filter(e -> e.salary() == s)
                        .map(Employee::name)
                        .toList())
                .orElse(List.of());

        // 3. Generalised: Nth highest
        int n = 3;
        Optional<Long> nthHighest = employees.stream()
                .map(Employee::salary)
                .distinct()
                .sorted(Comparator.reverseOrder())
                .skip(n - 1)
                .findFirst();

        // 4. Bonus: second-highest salary in EACH department
        Map<String, Optional<Long>> secondPerDept = employees.stream()
                .collect(Collectors.groupingBy(
                        Employee::dept,
                        TreeMap::new,                         // sorted keys for stable output
                        Collectors.collectingAndThen(
                                Collectors.mapping(Employee::salary, Collectors.toSet()),
                                set -> set.stream()
                                        .sorted(Comparator.reverseOrder())
                                        .skip(1)
                                        .findFirst())));

        System.out.println("Second highest salary : " + secondHighest.orElse(null));
        System.out.println("Earned by             : " + earners);
        System.out.println(n + "rd highest salary    : " + nthHighest.orElse(null));
        System.out.println("Second highest by dept: " + secondPerDept);
    }
}
```

## Output
```
Second highest salary : 70000
Earned by             : [Sourav, Virat]
3rd highest salary    : 60000
Second highest by dept: {HR=Optional.empty, IT=Optional[50000], Sales=Optional[60000]}
```

## Key points to say in the interview
- **`distinct()` is the trap.** Without it, the answer here would be `70000` by luck, but the 3rd highest
  would wrongly be `70000` again instead of `60000`. Always clarify: "distinct salary or second employee?"
- `sorted(Comparator.reverseOrder())` for descending order — for objects use
  `Comparator.comparing(Employee::salary).reversed()`.
- `skip(n - 1).findFirst()` generalises to the Nth highest and returns `Optional` — HR has only one
  salary, so it gets `Optional.empty` instead of an exception.
- Alternative for "second employee": `.sorted(comparing(Employee::salary).reversed()).skip(1).findFirst()`
  — returns an `Optional<Employee>` but ignores ties.
- Per-department: `groupingBy` + `collectingAndThen(mapping(..., toSet()), finisher)` — `toSet()` does the
  distinct step, the finisher does sort/skip/findFirst.
- Complexity: O(n log n) because of sorting. For a single pass O(n), track top-two values with
  `reduce` or a simple loop — mention it if the interviewer asks to optimise.
- SQL follow-up often comes next: `SELECT MAX(salary) FROM emp WHERE salary < (SELECT MAX(salary) FROM emp)`
  or `DENSE_RANK()`.
