# Java Stream API Guide

The **Stream API**, introduced in **Java 8**, is one of the most powerful additions to the language. It provides a high-level, declarative way to process data, making code more concise, expressive, and maintainable. Instead of focusing on *how* to iterate, filter, or transform collections, Streams let you focus on *what* you want to achieve.

---

## Table of Contents

1. [Introduction to Streams](#1-introduction-to-streams)
2. [Creating Streams](#2-creating-streams)
3. [Intermediate Operations](#3-intermediate-operations)
4. [Terminal Operations](#4-terminal-operations)
5. [Primitive Streams](#5-primitive-streams)
6. [Parallel Streams](#6-parallel-streams)
7. [Collectors](#7-collectors)
8. [Optional and Streams](#8-optional-and-streams)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Best Practices](#10-best-practices)
11. [Advanced Grouping & Partitioning Cases](#11-advanced-grouping--partitioning-cases)

---

## 1. Introduction to Streams

* A **Stream** is a sequence of elements supporting sequential and parallel operations.
* Streams do **not** store data; they operate on a source such as a `Collection`, array, or I/O channel.
* A Stream is **consumed once**; it cannot be reused.

**Key ideas:**

* *Functional style*: Focus on operations, not iteration.
* *Lazy evaluation*: Intermediate operations are executed only when a terminal operation is invoked.
* *Pipelining*: Combine operations into a chain for readability and performance.

---

## 2. Creating Streams

```java
// From a collection
List<String> list = List.of("a", "b", "c");
Stream<String> stream1 = list.stream();

// From an array
String[] arr = {"x", "y", "z"};
Stream<String> stream2 = Arrays.stream(arr);

// Using Stream.of()
Stream<Integer> stream3 = Stream.of(1, 2, 3);

// Infinite streams
Stream<Double> randoms = Stream.generate(Math::random);
Stream<Integer> naturals = Stream.iterate(0, n -> n + 1);
```

---

## 3. Intermediate Operations

Intermediate operations transform a stream into another stream. They are **lazy**.

* **Filtering**

```java
list.stream()
    .filter(s -> s.startsWith("a"))
    .forEach(System.out::println);
```

* **Mapping**

```java
list.stream()
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

* **FlatMap**

```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4));
nested.stream()
      .flatMap(List::stream)
      .forEach(System.out::println); // 1,2,3,4
```

* **Sorting**

```java
list.stream()
    .sorted()
    .forEach(System.out::println);
```

* **Distinct**

```java
Stream.of(1, 2, 2, 3)
      .distinct()
      .forEach(System.out::println);
```

* **Limit & Skip**

```java
Stream.iterate(1, n -> n + 1)
      .skip(5)
      .limit(5)
      .forEach(System.out::println); // 6..10
```

---

## 4. Terminal Operations

Terminal operations trigger execution.

* **forEach**

```java
list.stream().forEach(System.out::println);
```

* **collect**

```java
List<String> upper = list.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

* **reduce**

```java
int sum = Stream.of(1, 2, 3, 4)
    .reduce(0, Integer::sum); // 10
```

* **min / max**

```java
Optional<String> min = list.stream().min(String::compareTo);
```

* **count**

```java
long count = list.stream().filter(s -> s.length() > 2).count();
```

* **anyMatch / allMatch / noneMatch**

```java
boolean anyStartsWithA = list.stream().anyMatch(s -> s.startsWith("a"));
```

* **findFirst / findAny**

```java
Optional<String> first = list.stream().findFirst();
```

---

## 5. Primitive Streams

Java provides specialized streams for primitives: `IntStream`, `LongStream`, `DoubleStream`.

```java
IntStream.range(1, 5).forEach(System.out::println); // 1..4
DoubleStream.of(1.0, 2.0).average().ifPresent(System.out::println);
```

---

## 6. Parallel Streams

Parallel streams split the source into multiple **chunks**, process them on different threads, and then merge the results. This is built on top of the **ForkJoinPool.commonPool()**, which uses a **work-stealing algorithm** to balance tasks across available threads.

### How Parallel Streams Work

* A parallel stream **divides the data source** into substreams.
* Each substream is processed by a worker thread in the **ForkJoinPool**.
* Results are **combined** into the final output by the terminal operation.
* The pool size (number of threads) is usually equal to the number of available CPU cores.

```java
List<String> list = List.of("alpha", "beta", "gamma", "delta");

list.parallelStream()
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

⚠️ **Note**:

* Parallel streams are **not magic**: splitting, synchronization, and merging results have overhead.
* For small collections, sequential streams are usually faster.

---

### Controlling Parallelism

By default, parallel streams use the **common ForkJoinPool**, which may be shared with other parallel tasks in the JVM. You can control parallelism in two ways:

1. **Set parallelism level for common pool**

```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "8");
```

2. **Use a custom ForkJoinPool**

```java
ForkJoinPool customPool = new ForkJoinPool(4);

customPool.submit(() ->
    list.parallelStream()
        .map(String::toUpperCase)
        .forEach(System.out::println)
).join();
```

---

### When to Use Parallel Streams ✅

Use parallel streams when:

* The dataset is **large** (tens of thousands of elements or more).
* The operation is **CPU-intensive**, stateless, and independent (e.g., numerical calculations, transformations).
* The environment has **multiple cores** available.
* You don’t need strict ordering in results, or you can tolerate `forEachOrdered()` overhead.

Examples:

```java
// Good candidate: heavy computation on large dataset
long count = LongStream.range(0, 10_000_000)
    .parallel()
    .map(n -> expensiveComputation(n))
    .filter(x -> x > 1000)
    .count();
```

---

### When NOT to Use Parallel Streams ❌

Avoid parallel streams when:

* The dataset is **small** (overhead > gain).
* Operations are **I/O-bound** (disk, network, DB). Thread contention will hurt performance.
* The task involves **synchronization** or shared mutable state.
* Ordering is critical and must be preserved.
* Running inside a container or server with **thread pool constraints** (e.g., application servers, web apps) — using the common pool may cause contention with request-handling threads.

Bad example:

```java
// Not a good candidate: small list, trivial operation
List<String> words = List.of("a", "b", "c");
words.parallelStream()
     .map(String::toUpperCase) // too lightweight for parallelism
     .forEach(System.out::println);
```

---

### Rules of Thumb

* **Sequential first**: Start with sequential streams.
* **Measure performance**: Only switch to parallel if profiling shows real benefit.
* **CPU-bound workloads** benefit most.
* **I/O-bound workloads** should use async I/O, not parallel streams.

---

## 7. Collectors

`Collectors` provide flexible reduction and grouping mechanisms.

* **To List/Set**

```java
Set<String> set = list.stream()
    .collect(Collectors.toSet());
```

* **Joining**

```java
String result = list.stream()
    .collect(Collectors.joining(", "));
```

* **Grouping**

```java
Map<Integer, List<String>> grouped = list.stream()
    .collect(Collectors.groupingBy(String::length));
```

* **Partitioning**

```java
Map<Boolean, List<String>> partitioned = list.stream()
    .collect(Collectors.partitioningBy(s -> s.startsWith("a")));
```

* **Summarizing**

```java
IntSummaryStatistics stats = IntStream.of(1, 2, 3, 4)
    .summaryStatistics();
```

---


### Grouping and Partitioning


Java `Collectors` provide powerful tools to **reduce**, **aggregate**, and **transform** stream elements. Two commonly confused operations are:

1. **Grouping (`groupingBy`)**

    * Divides elements into **multiple groups** based on a classifier function.
    * Returns a `Map<K, List<T>>` (or other downstream collector).
    * Can have **nested grouping** for multi-level categorization.

2. **Partitioning (`partitioningBy`)**

    * Divides elements into **two groups only**: `true` or `false` based on a predicate.
    * Returns a `Map<Boolean, List<T>>`.
    * Essentially a special case of grouping by a boolean condition.

**Difference in short:**

| Feature  | Grouping       | Partitioning               |
| -------- | -------------- | -------------------------- |
| Keys     | Any type       | Boolean                    |
| Map size | 0..N           | Always 2                   |
| Use case | Categorization | Yes/No conditions, filters |

---

### Examples

#### 1. Simple Grouping

Group words by their length:

```java
List<String> words = List.of("apple", "bat", "car", "banana", "ant");

Map<Integer, List<String>> grouped = words.stream()
    .collect(Collectors.groupingBy(String::length));

System.out.println(grouped);
// Output: {3=[bat, car, ant], 5=[apple], 6=[banana]}
```

---

#### 2. Grouping with Counting

Count how many words have each length:

```java
Map<Integer, Long> countByLength = words.stream()
    .collect(Collectors.groupingBy(String::length, Collectors.counting()));

System.out.println(countByLength);
// Output: {3=3, 5=1, 6=1}
```

---

#### 3. Nested Grouping (Multi-level)

Group people by **city** and then **age group**:

```java
record Person(String name, String city, int age) {}
List<Person> people = List.of(
    new Person("Alice", "NY", 23),
    new Person("Bob", "NY", 30),
    new Person("Charlie", "LA", 23),
    new Person("Dave", "LA", 30)
);

Map<String, Map<String, List<Person>>> nested = people.stream()
    .collect(Collectors.groupingBy(
        Person::city,
        Collectors.groupingBy(p -> p.age < 25 ? "Young" : "Adult")
    ));

System.out.println(nested);
/* Output:
{
  NY={Young=[Alice], Adult=[Bob]},
  LA={Young=[Charlie], Adult=[Dave]}
}
*/
```

---

#### 4. Partitioning

Partition numbers into even and odd:

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6);

Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

System.out.println(partitioned);
// Output: {false=[1, 3, 5], true=[2, 4, 6]}
```

---

#### 5. Partitioning with Downstream Collector

Find **max in each partition**:

```java
Map<Boolean, Optional<Integer>> maxPartition = numbers.stream()
    .collect(Collectors.partitioningBy(
        n -> n % 2 == 0,
        Collectors.maxBy(Integer::compare)
    ));

System.out.println(maxPartition);
// Output: {false=Optional[5], true=Optional[6]}
```

---

#### 6. Interview-style Challenge: Grouping and Summarizing

Suppose you have a list of transactions and want **total amount per type**:

```java
record Transaction(String type, double amount) {}
List<Transaction> txs = List.of(
    new Transaction("Food", 20),
    new Transaction("Food", 30),
    new Transaction("Travel", 100)
);

Map<String, Double> totalByType = txs.stream()
    .collect(Collectors.groupingBy(
        Transaction::type,
        Collectors.summingDouble(Transaction::amount)
    ));

System.out.println(totalByType);
// Output: {Food=50.0, Travel=100.0}
```

---

## 8. Optional and Streams

Streams often return `Optional<T>` for operations like `findFirst`, `min`, `max`.

```java
Optional<String> result = list.stream()
    .filter(s -> s.startsWith("z"))
    .findFirst();

result.ifPresentOrElse(
    System.out::println,
    () -> System.out.println("No match")
);
```

---

## 9. Common Pitfalls

1. **Reusing Streams**

```java
Stream<String> s = list.stream();
s.forEach(System.out::println);
s.forEach(System.out::println); // IllegalStateException
```

2. **Heavy Operations in Parallel Streams**
   Avoid shared mutable state; it breaks parallelism guarantees.

3. **forEach vs collect**
   Prefer `collect` when building results, not `forEach` with side effects.

---

## 10. Best Practices

* Prefer **method references** (`String::toUpperCase`) for clarity.
* Avoid **stateful lambda expressions** inside streams.
* Use **collectors** instead of mutating external collections.
* Leverage **primitive streams** to avoid boxing overhead.
* Use **parallel streams** only when:

    * The dataset is large.
    * The operation is CPU-intensive and stateless.
    * The environment has enough cores.
* Keep pipelines **readable**: one operation per line.
* Remember: **Streams are not always faster** — they’re about clarity and maintainability.

---

## 11. Advanced Grouping & Partitioning Cases

### 1. Partitioning + Grouping

Combine a boolean partition with secondary grouping:

```java
record Person(String name, int age) {}
List<Person> people = List.of(
    new Person("Alice", 23),
    new Person("Bob", 30),
    new Person("Charlie", 25),
    new Person("Dave", 19)
);

// Partition adults (age >= 25) and group by age bracket
Map<Boolean, Map<String, List<Person>>> result = people.stream()
    .collect(Collectors.partitioningBy(
        p -> p.age >= 25,
        Collectors.groupingBy(p -> {
            if (p.age < 20) return "Teen";
            else if (p.age < 30) return "Young Adult";
            else return "Adult";
        })
    ));

System.out.println(result);
/* Output:
{
  false={Teen=[Dave], Young Adult=[Alice]},
  true={Young Adult=[Charlie], Adult=[Bob]}
}
*/
```

---

### 2. Top-N elements per group

Find top 1 oldest person per city:

```java
record Person(String name, String city, int age) {}
List<Person> people = List.of(
    new Person("Alice", "NY", 23),
    new Person("Bob", "NY", 30),
    new Person("Charlie", "LA", 25),
    new Person("Dave", "LA", 40)
);

Map<String, Optional<Person>> topOldest = people.stream()
    .collect(Collectors.groupingBy(
        Person::city,
        Collectors.maxBy(Comparator.comparingInt(Person::age))
    ));

System.out.println(topOldest);
// Output: {NY=Optional[Bob], LA=Optional[Dave]}
```

---

### 3. Counting elements per partition

Count even vs odd numbers:

```java
List<Integer> nums = List.of(1,2,3,4,5,6,7,8,9);

Map<Boolean, Long> countPartition = nums.stream()
    .collect(Collectors.partitioningBy(
        n -> n % 2 == 0,
        Collectors.counting()
    ));

System.out.println(countPartition);
// Output: {false=5, true=4}
```

---

### 4. Mapping + Grouping

Get **names only** per age group:

```java
List<Person> people = List.of(
    new Person("Alice", 23),
    new Person("Bob", 30),
    new Person("Charlie", 23)
);

Map<Integer, List<String>> namesByAge = people.stream()
    .collect(Collectors.groupingBy(
        Person::age,
        Collectors.mapping(Person::name, Collectors.toList())
    ));

System.out.println(namesByAge);
// Output: {23=[Alice, Charlie], 30=[Bob]}
```

---

### 5. Partitioning + Summing

Sum amounts in a list of transactions by **positive vs negative**:

```java
record Transaction(double amount) {}
List<Transaction> txs = List.of(
    new Transaction(100),
    new Transaction(-50),
    new Transaction(200),
    new Transaction(-20)
);

Map<Boolean, Double> sumBySign = txs.stream()
    .collect(Collectors.partitioningBy(
        t -> t.amount > 0,
        Collectors.summingDouble(t -> t.amount)
    ));

System.out.println(sumBySign);
// Output: {false=-70.0, true=300.0}
```

---

### 6. Multi-level Partitioning

Partition numbers by even/odd and then by >5 / ≤5:

```java
List<Integer> numbers = List.of(1,2,3,6,7,8);

Map<Boolean, Map<Boolean, List<Integer>>> multiPartition = numbers.stream()
    .collect(Collectors.partitioningBy(
        n -> n % 2 == 0,                 // even?
        Collectors.partitioningBy(n -> n > 5) // greater than 5?
    ));

System.out.println(multiPartition);
/* Output:
{
  false={false=[1, 3], true=[7]},
  true={false=[2], true=[6, 8]}
}
*/
```


