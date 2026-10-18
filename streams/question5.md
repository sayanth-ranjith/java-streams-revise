# Question 5 — Flatten employee skills with `flatMap`

**Asked to check if you know `map` vs `flatMap`** — the most common "explain the difference" Streams question,
usually followed by "now find the unique skills / count per skill".

## Question
Each employee has a **list of skills**. Using streams, find:
1. Why `map(Employee::skills)` is wrong (what it returns) vs `flatMap`
2. All skills in one flat list
3. **Unique** skills, sorted
4. **How many employees** have each skill
5. Reverse index: **skill → employee names**
6. **Unique skills per department** (Java 9+ `Collectors.flatMapping`)
7. Warm-ups: sum a `List<List<Integer>>`, and get distinct words from sentences

| id | name    | dept  | skills               |
|----|---------|-------|----------------------|
| 1  | Sayanth | IT    | Java, Spring, SQL    |
| 2  | Sourav  | IT    | Java, Kafka          |
| 3  | Dhoni   | Sales | Excel, SQL           |
| 4  | Sonu    | Sales | *(none)*             |
| 5  | Virat   | IT    | Java, Spring, Docker |
| 6  | Rohit   | HR    | Excel                |

## Solution
```java
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class FlatMapSkills {

    record Employee(int id, String name, String dept, List<String> skills) {}

    public static void main(String[] args) {
        List<Employee> employees = List.of(
                new Employee(1, "Sayanth", "IT", List.of("Java", "Spring", "SQL")),
                new Employee(2, "Sourav", "IT", List.of("Java", "Kafka")),
                new Employee(3, "Dhoni", "Sales", List.of("Excel", "SQL")),
                new Employee(4, "Sonu", "Sales", List.of()),
                new Employee(5, "Virat", "IT", List.of("Java", "Spring", "Docker")),
                new Employee(6, "Rohit", "HR", List.of("Excel")));

        // Trap: map gives a Stream<List<String>> -> list of lists, not flat
        List<List<String>> nested = employees.stream()
                .map(Employee::skills)
                .collect(Collectors.toList());

        // 1. flatMap flattens each inner list into one Stream<String>
        List<String> allSkills = employees.stream()
                .flatMap(e -> e.skills().stream())
                .collect(Collectors.toList());

        // 2. Unique skills, sorted
        List<String> uniqueSkills = employees.stream()
                .flatMap(e -> e.skills().stream())
                .distinct()
                .sorted()
                .collect(Collectors.toList());

        // 3. How many employees have each skill (TreeMap for sorted keys)
        Map<String, Long> skillCount = employees.stream()
                .flatMap(e -> e.skills().stream())
                .collect(Collectors.groupingBy(s -> s, TreeMap::new, Collectors.counting()));

        // 4. Reverse index: skill -> employee names (flatMap to (skill, name) pairs)
        Map<String, List<String>> namesBySkill = employees.stream()
                .flatMap(e -> e.skills().stream()
                        .map(s -> Map.entry(s, e.name())))
                .collect(Collectors.groupingBy(Map.Entry::getKey, TreeMap::new,
                        Collectors.mapping(Map.Entry::getValue, Collectors.toList())));

        // 5. Unique skills per department (Java 9+ flatMapping downstream collector)
        Map<String, Set<String>> skillsByDept = employees.stream()
                .collect(Collectors.groupingBy(Employee::dept, TreeMap::new,
                        Collectors.flatMapping(e -> e.skills().stream(),
                                Collectors.toCollection(TreeSet::new))));

        // 6. Classic warm-up: flatten List<List<Integer>> and sum
        List<List<Integer>> matrix = List.of(List.of(1, 2), List.of(3, 4, 5), List.of(6));
        int sum = matrix.stream()
                .flatMap(List::stream)
                .mapToInt(Integer::intValue)
                .sum();

        // 7. Split sentences into distinct words
        List<String> words = Stream.of("java streams are fun", "streams are lazy")
                .flatMap(line -> Arrays.stream(line.split("\\s+")))
                .distinct()
                .collect(Collectors.toList());

        System.out.println("map (nested)  : " + nested);
        System.out.println("flatMap (flat): " + allSkills);
        System.out.println("Unique skills : " + uniqueSkills);
        System.out.println("Skill count   : " + skillCount);
        System.out.println("Names by skill: " + namesBySkill);
        System.out.println("Skills by dept: " + skillsByDept);
        System.out.println("Matrix sum    : " + sum);
        System.out.println("Distinct words: " + words);
    }
}
```

## Output
```
map (nested)  : [[Java, Spring, SQL], [Java, Kafka], [Excel, SQL], [], [Java, Spring, Docker], [Excel]]
flatMap (flat): [Java, Spring, SQL, Java, Kafka, Excel, SQL, Java, Spring, Docker, Excel]
Unique skills : [Docker, Excel, Java, Kafka, SQL, Spring]
Skill count   : {Docker=1, Excel=2, Java=3, Kafka=1, SQL=2, Spring=2}
Names by skill: {Docker=[Virat], Excel=[Dhoni, Rohit], Java=[Sayanth, Sourav, Virat], Kafka=[Sourav], SQL=[Sayanth, Dhoni], Spring=[Sayanth, Virat]}
Skills by dept: {HR=[Excel], IT=[Docker, Java, Kafka, SQL, Spring], Sales=[Excel, SQL]}
Matrix sum    : 21
Distinct words: [java, streams, are, fun, lazy]
```

## Key points to say in the interview
- `map` is **one-to-one**: `Function<T, R>` → `map(Employee::skills)` gives `Stream<List<String>>` (a list of lists).
- `flatMap` is **one-to-many**: `Function<T, Stream<R>>` → each element becomes a stream and all of them are
  concatenated into one `Stream<String>`. The mapper must return a **Stream**, not a List — hence `e.skills().stream()`
  or `List::stream`.
- An **empty list** (Sonu) simply contributes nothing — no null checks needed. But a **`null` list** would throw NPE;
  guard with `Stream.ofNullable(e.skills()).flatMap(List::stream)` (Java 9+) or default to `List.of()` in the model.
- When you need data from the **outer** element after flattening (skill → name), map inside the flatMap to a pair:
  `e.skills().stream().map(s -> Map.entry(s, e.name()))`.
- `Collectors.flatMapping(mapper, downstream)` (Java 9+) is the downstream version — use it inside `groupingBy`.
- Primitive variants: `flatMapToInt`, `flatMapToLong`, `flatMapToDouble`; Java 16 adds `mapMulti` as an alternative.
- `Arrays.stream(line.split("\\s+"))` inside `flatMap` is the classic "sentences → words" pattern (leads into word frequency).
- Complexity: O(total number of inner elements); `flatMap` is lazy like other intermediate ops.
