# Question 7 — Sort by multiple fields with `Comparator` chaining

**Asked to check if you can build a multi-key sort cleanly** — interviewers love the "dept ascending, salary descending"
twist because most candidates put `.reversed()` in the wrong place and flip the whole sort.

## Question
Using the employee list below:
1. Sort by **salary descending**
2. Sort by **dept ascending**, then **salary descending**, then **name ascending** (tie-breaker)
3. What goes wrong if you write `comparing(dept).thenComparing(salary).reversed()`?
4. Sort by **id descending** without boxing
5. Sort by **age** where age can be **null** (nulls at the end)
6. Group by dept (depts in alphabetical order), with names inside each dept **highest-paid first**
7. **Top 3** earners, ties broken by name

| id | name    | dept  | salary | age  |
|----|---------|-------|--------|------|
| 1  | Sayanth | IT    | 90000  | 26   |
| 2  | Sourav  | IT    | 75000  | 29   |
| 3  | Dhoni   | Sales | 60000  | 42   |
| 4  | Sonu    | Sales | 82000  | 31   |
| 5  | Virat   | IT    | 90000  | 36   |
| 6  | Rohit   | HR    | 50000  | null |

## Solution
```java
import java.util.*;
import java.util.stream.Collectors;

public class MultiFieldSort {

    record Employee(int id, String name, String dept, double salary, Integer age) {}

    public static void main(String[] args) {
        List<Employee> employees = List.of(
                new Employee(1, "Sayanth", "IT", 90000, 26),
                new Employee(2, "Sourav", "IT", 75000, 29),
                new Employee(3, "Dhoni", "Sales", 60000, 42),
                new Employee(4, "Sonu", "Sales", 82000, 31),
                new Employee(5, "Virat", "IT", 90000, 36),
                new Employee(6, "Rohit", "HR", 50000, null));

        // 1. Single field: salary descending
        List<String> bySalaryDesc = employees.stream()
                .sorted(Comparator.comparingDouble(Employee::salary).reversed())
                .map(Employee::name)
                .toList();
        System.out.println("1. salary desc            : " + bySalaryDesc);

        // 2. Multiple fields: dept ASC, then salary DESC, then name ASC (tie-breaker)
        Comparator<Employee> byDeptThenSalaryDescThenName =
                Comparator.comparing(Employee::dept)
                        .thenComparing(Employee::salary, Comparator.reverseOrder())
                        .thenComparing(Employee::name);
        employees.stream()
                .sorted(byDeptThenSalaryDescThenName)
                .forEach(e -> System.out.println("2. " + e.dept() + " | " + e.salary() + " | " + e.name()));

        // 3. TRAP: .reversed() at the end reverses the WHOLE chain, not just the last key
        List<String> wrong = employees.stream()
                .sorted(Comparator.comparing(Employee::dept)
                        .thenComparingDouble(Employee::salary)
                        .reversed())                        // dept is now DESC too!
                .map(e -> e.dept() + ":" + e.name())
                .toList();
        System.out.println("3. reversed() whole chain : " + wrong);

        // 4. Primitive-specialised comparators avoid boxing: comparingDouble / thenComparingInt
        List<String> byIdDesc = employees.stream()
                .sorted(Comparator.comparingInt(Employee::id).reversed())
                .map(e -> e.id() + "-" + e.name())
                .toList();
        System.out.println("4. id desc (comparingInt) : " + byIdDesc);

        // 5. Null-safe key: age is null for Rohit -> nullsLast keeps it at the end
        List<String> byAgeNullsLast = employees.stream()
                .sorted(Comparator.comparing(Employee::age,
                        Comparator.nullsLast(Comparator.naturalOrder())))
                .map(e -> e.name() + "(" + e.age() + ")")
                .toList();
        System.out.println("5. age, nulls last        : " + byAgeNullsLast);

        // 6. Sort within each group: highest-paid first per dept, depts in TreeMap order
        Map<String, List<String>> sortedWithinDept = employees.stream()
                .sorted(Comparator.comparingDouble(Employee::salary).reversed()
                        .thenComparing(Employee::name))
                .collect(Collectors.groupingBy(Employee::dept, TreeMap::new,
                        Collectors.mapping(Employee::name, Collectors.toList())));
        System.out.println("6. per dept, salary desc  : " + sortedWithinDept);

        // 7. Top 3 by salary, ties broken by name
        List<String> top3 = employees.stream()
                .sorted(Comparator.comparingDouble(Employee::salary).reversed()
                        .thenComparing(Employee::name))
                .limit(3)
                .map(Employee::name)
                .toList();
        System.out.println("7. top 3 salary           : " + top3);
    }
}
```

## Output
```
1. salary desc            : [Sayanth, Virat, Sonu, Sourav, Dhoni, Rohit]
2. HR | 50000.0 | Rohit
2. IT | 90000.0 | Sayanth
2. IT | 90000.0 | Virat
2. IT | 75000.0 | Sourav
2. Sales | 82000.0 | Sonu
2. Sales | 60000.0 | Dhoni
3. reversed() whole chain : [Sales:Sonu, Sales:Dhoni, IT:Sayanth, IT:Virat, IT:Sourav, HR:Rohit]
4. id desc (comparingInt) : [6-Rohit, 5-Virat, 4-Sonu, 3-Dhoni, 2-Sourav, 1-Sayanth]
5. age, nulls last        : [Sayanth(26), Sourav(29), Sonu(31), Virat(36), Dhoni(42), Rohit(null)]
6. per dept, salary desc  : {HR=[Rohit], IT=[Sayanth, Virat, Sourav], Sales=[Sonu, Dhoni]}
7. top 3 salary           : [Sayanth, Virat, Sonu]
```

## Key points to say in the interview
- **Build the chain left to right**: `Comparator.comparing(primary).thenComparing(secondary).thenComparing(tieBreaker)`.
  Each `thenComparing` only kicks in when everything before it is equal.
- **Reverse just one key** with the 2-arg overload: `thenComparing(Employee::salary, Comparator.reverseOrder())`.
  Calling `.reversed()` at the end of a chain reverses **every** key (see output 3 — dept became `Sales, IT, HR`).
- `.reversed()` is fine on a **single** key comparator: `comparingDouble(Employee::salary).reversed()`.
- **Type inference trap**: `Comparator.comparing(e -> e.dept()).reversed()` won't compile — with a lambda the type is
  inferred as `Object`. Use a method reference (`Employee::dept`) or an explicit type `(Employee e) -> e.dept()`.
- Prefer `comparingInt` / `comparingLong` / `comparingDouble` (and `thenComparingInt` …) for primitives — no boxing.
- **Nulls**: a plain `comparing(Employee::age)` throws `NullPointerException` on a null key. Wrap the key comparator:
  `comparing(Employee::age, Comparator.nullsLast(Comparator.naturalOrder()))` (or `nullsFirst`).
  If the **element itself** can be null, use `Comparator.nullsLast(yourComparator)` on the whole thing.
- `sorted()` on a sequential stream is **stable**: Sayanth and Virat both earn 90000, so without a tie-breaker they stay
  in encounter order. Add `thenComparing(Employee::name)` when the order must be deterministic regardless of input.
- `sorted()` with no argument needs `Comparable` elements, otherwise `ClassCastException` at runtime.
- Sort **before** `groupingBy` to get sorted lists inside each group — `groupingBy` + `toList()` preserves encounter order.
  Use `TreeMap::new` as the map supplier if the **keys** must be sorted too.
- `sorted()` is a **stateful, full-barrier** operation: O(n log n), it buffers all elements. `sorted().limit(k)` still
  sorts everything — for huge inputs a `PriorityQueue` of size k is O(n log k).
- Same comparators work outside streams: `list.sort(cmp)`, `Collections.sort(list, cmp)`, `TreeMap(cmp)`, `max(cmp)`.
