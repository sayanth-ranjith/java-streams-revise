# Question 2 — First non-repeating character in a String

**Asked very often in interviews** (TCS, Infosys, IBM, Accenture, product companies) because it checks
`chars()`, `groupingBy`, `counting()`, and — the trap — **preserving insertion order**.

## Question
Given a String, find the **first character that does not repeat**.
Bonus: also find the **first character that repeats**.

Input: `"swiss programming"`

## Solution
```java
import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;

public class FirstNonRepeating {
    public static void main(String[] args) {
        String input = "swiss programming";

        // 1. Build a frequency map, keeping insertion order
        Map<Character, Long> freq = input.chars()               // IntStream of char codes
                .mapToObj(c -> (char) c)                        // box to Character
                .collect(Collectors.groupingBy(
                        Function.identity(),                    // key = the character
                        LinkedHashMap::new,                     // KEEP ORDER (HashMap would break this)
                        Collectors.counting()));                // value = count

        // 2. First entry with count == 1
        Optional<Character> firstNonRepeating = freq.entrySet().stream()
                .filter(e -> e.getValue() == 1)
                .map(Map.Entry::getKey)
                .findFirst();

        // Bonus: first entry with count > 1
        Optional<Character> firstRepeating = freq.entrySet().stream()
                .filter(e -> e.getValue() > 1)
                .map(Map.Entry::getKey)
                .findFirst();

        System.out.println("Frequency map      : " + freq);
        System.out.println("First non-repeating: " + firstNonRepeating.orElse(null));
        System.out.println("First repeating    : " + firstRepeating.orElse(null));
    }
}
```

## Output
```
Frequency map      : {s=3, w=1, i=2,  =1, p=1, r=2, o=1, g=2, a=1, m=2, n=1}
First non-repeating: w
First repeating    : s
```

## Key points to say in the interview
- `groupingBy(classifier, mapFactory, downstream)` — the **3-arg overload** lets you pass `LinkedHashMap::new`.
  With the default `HashMap`, order is not guaranteed and `findFirst()` may return the wrong character.
- `String.chars()` returns an `IntStream`, so you must `mapToObj(c -> (char) c)` before grouping.
- `counting()` returns `Long`, not `Integer`.
- Return `Optional` — the string may have no non-repeating character (e.g. `"aabb"`).
- Case-insensitive variant: `input.toLowerCase().chars()...`; to ignore spaces add `.filter(c -> c != ' ')`.
- Complexity: O(n) time, O(k) space (k = distinct characters).
