# Question 6 — `Collectors.toMap` with duplicate keys and a merge function

**Asked to check if you know why `toMap` blows up on duplicate keys** — a very common follow-up after `groupingBy`,
because many candidates reach for `toMap` and forget the merge function, the map supplier, and the null trap.

## Question
Using the employee list below, with `Collectors.toMap`:
1. Build `id -> name` (keys are unique)
2. Try `dept -> name` — what happens and why?
3. Fix it by keeping the **first** employee seen per department
4. **Concatenate** all names per department
5. **Sum** salaries per department
6. Find the **highest-paid employee per department** (no `groupingBy`)
7. Keep the departments in **insertion order** (4-arg `toMap`)
8. What happens if the value mapper returns **null**?
9. Bonus: word frequency with `toMap` instead of `groupingBy` + `counting()`

| id | name    | dept  | salary |
|----|---------|-------|--------|
| 1  | Sayanth | IT    | 90000  |
| 2  | Sourav  | IT    | 75000  |
| 3  | Dhoni   | Sales | 60000  |
| 4  | Sonu    | Sales | 82000  |
| 5  | Virat   | IT    | 95000  |
| 6  | Rohit   | HR    | 50000  |

## Solution
```java
import java.util.*;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.stream.Collectors;

public class ToMapMerge {

    record Employee(int id, String name, String dept, double salary) {}

    public static void main(String[] args) {
        List<Employee> employees = List.of(
                new Employee(1, "Sayanth", "IT", 90000),
                new Employee(2, "Sourav", "IT", 75000),
                new Employee(3, "Dhoni", "Sales", 60000),
                new Employee(4, "Sonu", "Sales", 82000),
                new Employee(5, "Virat", "IT", 95000),
                new Employee(6, "Rohit", "HR", 50000));

        // 1. Unique key: id -> name works fine (2-arg toMap)
        Map<Integer, String> idToName = employees.stream()
                .collect(Collectors.toMap(Employee::id, Employee::name));
        System.out.println("1. id -> name        : " + idToName);

        // 2. Trap: dept is NOT unique -> IllegalStateException
        try {
            employees.stream()
                    .collect(Collectors.toMap(Employee::dept, Employee::name));
        } catch (IllegalStateException ex) {
            System.out.println("2. dept -> name      : " + ex.getMessage());
        }

        // 3. Merge function: keep the first value seen
        Map<String, String> firstPerDept = employees.stream()
                .collect(Collectors.toMap(Employee::dept, Employee::name,
                        (oldVal, newVal) -> oldVal));
        System.out.println("3. first per dept    : " + new TreeMap<>(firstPerDept));

        // 4. Merge function: concatenate names
        Map<String, String> namesPerDept = employees.stream()
                .collect(Collectors.toMap(Employee::dept, Employee::name,
                        (a, b) -> a + ", " + b));
        System.out.println("4. names per dept    : " + new TreeMap<>(namesPerDept));

        // 5. Merge function: sum salaries per dept
        Map<String, Double> totalPerDept = employees.stream()
                .collect(Collectors.toMap(Employee::dept, Employee::salary, Double::sum));
        System.out.println("5. total per dept    : " + new TreeMap<>(totalPerDept));

        // 6. Highest-paid employee per dept: BinaryOperator.maxBy as merge
        Map<String, Employee> topPerDept = employees.stream()
                .collect(Collectors.toMap(Employee::dept, Function.identity(),
                        BinaryOperator.maxBy(Comparator.comparingDouble(Employee::salary))));
        topPerDept.entrySet().stream()
                .sorted(Map.Entry.comparingByKey())
                .forEach(e -> System.out.println("6. top in " + e.getKey() + "\t: " + e.getValue().name()));

        // 7. 4-arg toMap: keep insertion order with LinkedHashMap
        Map<String, Double> ordered = employees.stream()
                .collect(Collectors.toMap(Employee::dept, Employee::salary, Double::sum,
                        LinkedHashMap::new));
        System.out.println("7. insertion order   : " + ordered);

        // 8. Trap: null value -> NullPointerException (HashMap.merge rejects null)
        try {
            List.of("a", "bb", "ccc").stream()
                    .collect(Collectors.toMap(Function.identity(),
                            s -> s.length() == 2 ? null : s.length()));
        } catch (NullPointerException ex) {
            System.out.println("8. null value        : NullPointerException");
        }

        // 9. Word frequency with toMap (same as groupingBy + counting)
        String text = "java stream java map stream java";
        Map<String, Integer> freq = Arrays.stream(text.split(" "))
                .collect(Collectors.toMap(w -> w, w -> 1, Integer::sum, TreeMap::new));
        System.out.println("9. word frequency    : " + freq);
    }
}
```

## Output
```
1. id -> name        : {1=Sayanth, 2=Sourav, 3=Dhoni, 4=Sonu, 5=Virat, 6=Rohit}
2. dept -> name      : Duplicate key IT (attempted merging values Sayanth and Sourav)
3. first per dept    : {HR=Rohit, IT=Sayanth, Sales=Dhoni}
4. names per dept    : {HR=Rohit, IT=Sayanth, Sourav, Virat, Sales=Dhoni, Sonu}
5. total per dept    : {HR=50000.0, IT=260000.0, Sales=142000.0}
6. top in HR	: Rohit
6. top in IT	: Virat
6. top in Sales	: Sonu
7. insertion order   : {IT=260000.0, Sales=142000.0, HR=50000.0}
8. null value        : NullPointerException
9. word frequency    : {java=3, map=1, stream=2}
```

## Key points to say in the interview
- **Three overloads**: `toMap(keyMapper, valueMapper)`, `toMap(key, value, mergeFunction)`, `toMap(key, value, merge, mapSupplier)`.
- **Duplicate keys**: the 2-arg version throws `IllegalStateException` (Java 9+ message shows the key and both values). Always ask "can keys repeat?" — if yes, pass a merge function.
- **Merge function** is a `BinaryOperator<V>` called as `(existing, incoming)`: `(a, b) -> a` keeps first, `(a, b) -> b` keeps last, `Integer::sum` / `Double::sum` aggregates, `BinaryOperator.maxBy(comparator)` keeps the max.
- **Order**: default is a `HashMap` (no order guarantee). Use the 4-arg form with `LinkedHashMap::new` for encounter order or `TreeMap::new` for sorted keys.
- **Null values**: `toMap` uses `HashMap.merge`, which throws `NullPointerException` on a null value (even with unique keys). Filter nulls first, wrap in `Optional`, or collect with `groupingBy` / a manual `forEach(map::put)`.
- **`toMap` vs `groupingBy`**: `groupingBy` gives `Map<K, List<V>>` (or a downstream result); `toMap` gives one value per key. Highest-per-dept with `toMap` + `maxBy` avoids the `Optional` that `groupingBy(..., maxBy(...))` returns.
- **Immutable result**: Java 10+ `Collectors.toUnmodifiableMap(...)` — also rejects duplicates (unless a merge function is given) and nulls.
- **Parallel streams**: `toConcurrentMap` avoids merging per-thread maps, but loses ordering.
- **Complexity**: O(n) with a `HashMap`; O(n log k) with `TreeMap` where k is the number of distinct keys.
