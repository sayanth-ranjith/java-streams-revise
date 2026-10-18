# Java Streams Revision Repo

This repository is a quick Java Streams revision note for interview preparation.

As of today, it is meant for:
- revising common Java Stream questions before interviews
- quickly looking at practice problems and examples
- improving confidence by solving and understanding stream patterns

## Contents
- [streams/question1.md](streams/question1.md) — example question and solution around grouping and max salary in a department
- [streams/question2.md](streams/question2.md) — first non-repeating (and first repeating) character in a String using `groupingBy` + `LinkedHashMap` + `counting()`
- [streams/question3.md](streams/question3.md) — second / Nth highest salary (with duplicates) using `distinct()` + `sorted(reverseOrder())` + `skip()`, plus per-department variant
- [streams/question4.md](streams/question4.md) — split employees into two groups with `partitioningBy` + downstream collectors, and why it differs from `groupingBy`
- [streams/question5.md](streams/question5.md) — `map` vs `flatMap`: flatten employee skills, unique skills, skill → names index, and `Collectors.flatMapping` per department

## Purpose
Use this repo for fast revision, repeated practice, and interview recall before facing Java stream questions.
