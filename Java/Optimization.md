#  Performance Optimization

## 1. Loop Hoisting (Invariant Code Motion)

Move computations that do not depend on the loop iteration outside the loop.

```java
// Bad
int sum = 0;
for (int i = 0; i < 1_000_000; i++) {
    int x = 5 * 10; // constant inside loop
    sum += x + i;
}

// Good
int x = 5 * 10;
int sum = 0;
for (int i = 0; i < 1_000_000; i++) {
    sum += x + i;
}


// Bad
for (int i = 0; i < list.size(); i++) {
    //...
}

// Good
int listSize= = list.size();
for (int i = 0; i < listSize; i++) {
    //...
}
```

---

## 2. Inlining (Reducing Method Call Overhead)

Small methods are routinely inlined by the JIT, reducing call overhead.

```java
// Before inlining
private int add(int a, int b) { return a + b; }

int sum = 0;
for (int i = 0; i < 1_000_000; i++) {
    sum += add(i, 1);
}

// Conceptually after inlining
int sum2 = 0;
for (int i = 0; i < 1_000_000; i++) {
    sum2 += i + 1;
}
```

**Note:** Don’t inline manually unless profiling proves the method is a bottleneck.

---

## 3. Lock Coarsening

It is known that Hotspot does lock coarsening optimizations that can effectively merge several adjacent locking blocks, thus reducing the locking overhead. It effectively converts this:

```java
synchronized (obj) {
    // statements 1
}
synchronized (obj) {
    // statements 2
}
```
into:
```java
synchronized (obj) {
    // statements 1
    // statements 2
}
```

Merge adjacent synchronized blocks on the same lock.

```java
// Inefficient
for (...) {
    synchronized (lock) {
        doSomething();
    }
    synchronized (lock) {
        doSomethingElse();
    }
}

// Improved
synchronized (lock) {
    for (...) {
        doSomething();
        doSomethingElse();
    }
}
```

---

## 4. Dead Code Elimination

Remove calculations or branches that don’t affect output.

```java
public int compute(int x) {
    int unused = x * 100; // unused
    return x + 5;
}
```

Compilers remove useless variables, but you should too.

---

## 5. Redundant Store Elimination

Avoid assignments that are overwritten before use.

```java
int x = 0;
x = 5;
x = 10; // 5 is never used
```

---

## 6. Loop Unswitching

Hoist invariant conditionals outside the loop.

```java
// Before
for (int i = 0; i < 1000; i++) {
    if (flag) doA();
    else doB();
}

// After
if (flag) {
    for (int i = 0; i < 1000; i++) doA();
} else {
    for (int i = 0; i < 1000; i++) doB();
}
```

---

## 7. Loop Peeling

Handle the first/last iteration separately to simplify loop logic.

```java
doFirstIteration();
for (int i = 1; i < 1000; i++) {
    doNormalIteration(i);
}
```

---

## 8. Algorithm & Data Structure Choice

Often the biggest performance gains come from choosing the right structure.

| Data Structure | Access | Search | Insertion  | Deletion   |
| -------------- | ------ | ------ | ---------- | ---------- |
| Array          | O(1)   | O(n)   | O(n)       | O(n)       |
| HashSet        | N/A    | O(1)   | O(1)       | O(1)       |
| LinkedList     | O(n)   | O(n)   | O(1) front | O(1) front |

**Use `HashSet` instead of `List.contains()` for fast membership checks.**

---

## 9. Avoid Unnecessary Object Creation

Reuse heavy objects to reduce Garbage Collector pressure.

```java
// Bad
public List<User> getUsers() {
    ObjectMapper mapper = new ObjectMapper();
    return mapper.readValue(...);
}

// Good
private static final ObjectMapper MAPPER = new ObjectMapper();

public List<User> getUsers() {
    return MAPPER.readValue(...);
}
```

---

## 10. Efficient String Handling

Avoid string concatenation in loops.

```java
// Bad
String result = "";
for (String s : list) {
    result += s;
}

// Good
StringBuilder sb = new StringBuilder();
for (String s : list) sb.append(s);
String result = sb.toString();
```

---

## 11. Concurrency Optimization

Use thread pools instead of creating new threads per task.

```java
// Bad
for (Task t : tasks) {
    new Thread(() -> process(t)).start();
}

// Good
ExecutorService pool = Executors.newFixedThreadPool(10);
for (Task t : tasks) {
    pool.submit(() -> process(t));
}
pool.shutdown();
```

Prefer concurrent collections (ConcurrentHashMap, CopyOnWriteArrayList)

---

## 12. I/O Optimization (Streaming & Buffering)

Use buffered I/O for large files or high-frequency operations.

```java
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        process(line);
    }
}
```

---

## 13. Caching

Cache expensive operations.

```java
@Cacheable("users")
public User getUserById(Long id) {
    // Expensive DB/API call
}
```

---

## 14. Lazy Initialization

Create heavy objects only when needed.

```java
@Bean
@Lazy
ExpensiveToCreateBean lazy() {
    return new ExpensiveToCreateBean();
}

@Component
@Lazy
public class TimeConsumingBean {
    // ...
}
```

---

## 15. JDBC Optimization

Use connection pools and batch updates.

```java
try (Connection conn = dataSource.getConnection();
     PreparedStatement ps = conn.prepareStatement(
         "INSERT INTO table(col) VALUES (?)"
     )) {

    for (int i = 0; i < batchSize; i++) {
        ps.setInt(1, i);
        ps.addBatch();
    }

    ps.executeBatch();
}
```

---

## 16. Hibernate Performance

Avoid N+1 query patterns using `JOIN FETCH`.

```java
// N+1
List<User> users =
    em.createQuery("SELECT u FROM User u", User.class)
      .getResultList();

for (User u : users) {
    u.getOrders().size();
}

// Optimized
List<User> users =
    em.createQuery(
        "SELECT u FROM User u JOIN FETCH u.orders WHERE u.active = true",
        User.class
    ).getResultList();
```

## 17. Escape Analysis & Scalar Replacement

JIT can remove object allocations if the object **never escapes the method**.

```java
class Point { int x, y; }

int calc() {
    Point p = new Point(); // may be removed by JIT
    p.x = 5;
    p.y = 10;
    return p.x + p.y;
}
```

JIT transforms it into:

```
int x = 5;
int y = 10;
return x + y;
```

**Effect:**
- zero allocations
- zero GC
- full stack allocation / scalarization

---

## 18. Escape Analysis → Synchronization Elimination

If an object is local to a thread, JIT removes synchronization.

```java
private final Object lock = new Object();

int foo() {
    synchronized (lock) {   // removed if lock is not shared
        return 42;
    }
}
```

---

## 19. Avoid Autoboxing

Autoboxing creates garbage and slows down arithmetic.

```java
Long sum = 0L;  // BAD: mixing primitives and wrappers
sum += i;       // creates new Long internally
```

Always use primitives.

---

## 20. Use `final` Where Possible

`final` helps the JIT prove invariants → more inlining & better scalar replacement.

---

## 21. Array Bounds Check Elimination

JIT removes bounds checks inside predictable loops:

```java
for (int i = 0; i < arr.length; i++) {
    sum += arr[i];  // bounds check hoisted or removed
}
```

---

## 22. CPU-Friendly Memory Layout

Prefer **arrays of primitives** over arrays of objects.

```java
int[] values;     // fast, contiguous
Integer[] values; // slow, pointer chasing
```

Why?
- fewer cache misses
- sequential prefetch
- less GC pressure

---

## 23. Branch Prediction Optimization

Avoid unpredictable branches inside hot loops.

Bad:

```java
if (random.nextBoolean()) doA(); else doB();
```

Good:

* move branches out of loops
* sort data to reduce randomness
* split data into buckets

---

## 24. Warmup Time (JIT needs warmup to optimize)

Benchmarking rule:
**first runs lie** — JVM must warm up before real performance happens.

Always benchmark with JMH.

---

## 25. Avoid Frequent Logging in Hot Paths

Logging = synchronization + I/O + string building.

```java
// DO NOT DO THIS IN HOT LOOPS
log.info("value={}", x);
```

Disable or move logs outside hot regions.

---

## 26. Use Bulk Operations

Good:

```java
System.arraycopy(a, 0, b, 0, len);
```

Bad:

```java
for (int i = 0; i < len; i++) {
    b[i] = a[i];
}
```

Native ops use SIMD (Single Instruction, Multiple Data) under the hood. This is *much* faster.

---

## 27. Prefer `EnumMap` / `EnumSet` Over `HashMap`

Both are array-backed → ultra fast, compact, predictable.

---

## 28. Avoid Using Reflection in Hot Paths

Reflection = slow, unpredictable, unoptimizable by JIT.

---

## 29. Avoid Frequent Resizing

For ArrayList, HashMap, etc. set initial capacity.

```java
new ArrayList<>(100_000);
new HashMap<>(initialCapacity, loadFactor);
```

---

## 30. Prefer Switch Over Cascading If/Else

Especially on primitives and enums.

