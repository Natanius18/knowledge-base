# Common Code Issues

## N+1 Problem

### Definition
The **N+1 query problem** occurs in ORMs (like JPA/Hibernate) when fetching related entities.  
The ORM executes **1 query for the main entity** and **N additional queries** for each related record — instead of a single optimized join.

### Example
```java
List<User> users = userRepository.findAll(); // 1 query
for (User user : users) {
    System.out.println(user.getOrders().size()); // N extra queries
}
````

**SQL under the hood:**

```sql
SELECT * FROM users;                      -- 1
SELECT * FROM orders WHERE user_id = 1;   -- +N
```

### Why it happens

* Default fetch type for collections (`@OneToMany`, `@ManyToMany`) is `LAZY`.
* Data is loaded only when accessed.

### Impact

* Works fine with small datasets.
* Causes major performance drops when records increase.

### ✅ Solutions

#### 1. `JOIN FETCH` in JPQL

```java
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

#### 2. `@EntityGraph`

```java
@EntityGraph(attributePaths = {"orders"})
@Query("SELECT u FROM User u")
List<User> findAllWithOrders();
```

#### 3. `@BatchSize`

```java
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
@BatchSize(size = 20)
private List<Order> orders;
```

#### 4. DTO Projection

```java
@Query("SELECT new com.example.UserOrderDTO(u.name, o.product) " +
       "FROM User u JOIN u.orders o")
List<UserOrderDTO> findUserOrders();
```

### Detecting N+1

Enable SQL logging:

```properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

If you see dozens of repeated `select ... where id=?`, you have N+1.

---

## EntityGraph

### Purpose

`EntityGraph` defines which relations should be fetched eagerly **per query**, without changing entity mappings.

### Example entity:

```java
@Entity
public class User {
    @Id
    private Long id;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders;

    @OneToOne(fetch = FetchType.LAZY)
    private Profile profile;
}
```

### ✅ Usage (Spring Data)

```java
@EntityGraph(attributePaths = {"orders", "profile"})
@Query("SELECT u FROM User u")
List<User> findAllWithOrdersAndProfile();
```

### Named Entity Graph

```java
@NamedEntityGraph(
    name = "user-with-orders",
    attributeNodes = @NamedAttributeNode("orders")
)
@Entity
public class User { ... }

@EntityGraph(value = "user-with-orders")
@Query("SELECT u FROM User u")
List<User> findAllWithOrders();
```

### Nested Subgraphs

```java
@NamedEntityGraph(
    name = "user-orders-products",
    attributeNodes = @NamedAttributeNode(value = "orders", subgraph = "ordersWithProducts"),
    subgraphs = {
        @NamedSubgraph(
            name = "ordersWithProducts",
            attributeNodes = @NamedAttributeNode("products")
        )
    }
)
```

### `JOIN FETCH` vs `EntityGraph`

| Feature       | JOIN FETCH     | EntityGraph               |
| ------------- | -------------- | ------------------------- |
| Type          | JPQL syntax    | JPA API                   |
| Flexibility   | Fixed in query | Configurable dynamically  |
| Pagination    | Issues         | Works                     |
| Count queries | Breaks         | Works                     |
| Readability   | Simple cases   | Better for complex graphs |

### Graph types

| Type         | Behavior                            |
| ------------ | ----------------------------------- |
| `fetchgraph` | Loads **only** specified relations  |
| `loadgraph`  | Loads specified + all default EAGER |

---

## Exception Suppression

### Problem

When a secondary exception hides the original cause.

```java
try {
    resource.close();
} catch (Exception e) {
    throw new RuntimeException("Error"); // original lost
}
```

### ✅ Solution — `try-with-resources`

```java
try (Resource r = new Resource()) {
    r.doSomething();
} // automatically handles suppressed exceptions
```

---

## Swallowed Exceptions

Ignoring exceptions hides bugs.

```java
try {
    riskyOperation();
} catch (Exception e) {
    // silently ignored 😬
}
```

✅ **Fix:** log or rethrow.

```java
catch (Exception e) {
    log.error("Error while processing", e);
}
```

## String Literals

### Definition

Strings written directly in code (`"hello"`) are stored in the **String pool** — a special area where identical strings share one object.

```java
String a = "Hello";
String b = "Hello";
System.out.println(a == b); // true
```

But:

```java
String c = new String("Hello");
System.out.println(a == c); // false
```

### How it works

* String pool ensures memory efficiency: identical literals are reused.
* The pool lives in the Java heap.
* Created at class load time.

### Problems

1. **Duplication:** hardcoded literals scattered across code.
2. **Memory waste:** using `new String("...")` creates extra objects.
3. **Wrong comparison:** using `==` instead of `.equals()`.

### ✅ Best Practices

* Use constants:

  ```java
  public static final String USER_ROLE = "user";
  ```
* Avoid `"magic strings"`.
* Always compare using `.equals()`.
* Don’t use `new String()` unnecessarily.

---

## String Concatenation

### Problem

Using `+` in loops creates many temporary `String` objects (immutable).

```java
String s = "";
for (int i = 0; i < 100; i++) {
    s += i; // inefficient
}
```

### ✅ Solution — Use `StringBuilder` or `StringBuffer`

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100; i++) {
    sb.append(i);
}
String result = sb.toString();
```

---

## StringBuilder vs StringBuffer

| Feature         | StringBuilder        | StringBuffer                 |
| --------------- | -------------------- | ---------------------------- |
| Thread safety   | Not thread-safe      | ✅ Thread-safe (synchronized) |
| Performance     | Faster               | Slower                       |
| Introduced      | Java 5               | Java 1.0                     |
| Recommended for | Single-threaded code | Multi-threaded code          |

### Example

```java
StringBuilder builder = new StringBuilder();
builder.append("Hello ").append("World!");
System.out.println(builder.toString());
```

---


## Memory Leaks

**Cause:** objects remain referenced and never garbage-collected.
**Examples:**

```java
private static List<User> cache = new ArrayList<>(); // never cleared
```

**Fixes:**

* Use `WeakReference`, `WeakHashMap`, or bounded caches (e.g. Guava).
* Remove listeners and close resources.
* Use profilers (`VisualVM`, `YourKit`).

---

## Resource Leaks

Forgetting to close connections, streams, files.

```java
try (Connection conn = dataSource.getConnection();
     Statement st = conn.createStatement()) {
    ...
} // auto-closed
```

✅ Always use **try-with-resources**.

---

## Mutable Keys in HashMap

If a key changes after insertion, the entry becomes inaccessible.

```java
Map<User, String> map = new HashMap<>();
User user = new User("Natan");
map.put(user, "dev");
user.setName("Eugene");
System.out.println(map.get(user)); // null
```

✅ **Fix:** use immutable keys (`String`, `UUID`).

---

## `equals()` and `hashCode()` inconsistency

**Symptom:** elements "disappear" from collections like `HashSet`.

**Rule:**
If `a.equals(b)` → their `hashCode()` must be identical.

✅ **Fix:**
Use IDE generation or Lombok:

```java
@EqualsAndHashCode
public class User { ... }
```

---

## Improper Synchronization

Shared mutable data accessed by multiple threads without locks.

✅ Use:

* `synchronized`
* `ReentrantLock`
* `ConcurrentHashMap`
* `AtomicInteger`

---

## Date/Time Pitfalls

Old classes (`Date`, `Calendar`, `SimpleDateFormat`) are mutable and not thread-safe.

✅ Use Java 8+ API:

```java
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter.ofPattern("yyyy-MM-dd");
```

---

## Quick Interview Q&A

| Question                                           | Short Answer                                                    |
| -------------------------------------------------- | --------------------------------------------------------------- |
| What is the N+1 problem?                           | ORM executes 1 + N SQL queries for related entities             |
| How to fix it?                                     | `JOIN FETCH`, `EntityGraph`, `@BatchSize`, DTOs                 |
| What is Exception Suppression?                     | Losing original exceptions when another overwrites it           |
| How to avoid it?                                   | Use try-with-resources                                          |
| What is a String literal?                          | String stored in the String pool                                |
| How to compare strings correctly?                  | Use `.equals()`, not `==`                                       |
| Why use StringBuilder?                             | Efficient concatenation in loops                                |
| Difference between StringBuilder and StringBuffer? | Builder = faster, not thread-safe; Buffer = slower, thread-safe |
| What causes memory leaks?                          | Unreleased references, static caches, listeners                 |
| How to fix equals/hashCode issues?                 | Keep both consistent; use Lombok or IDE                         |
| What’s wrong with swallowing exceptions?           | Hides real problems, makes debugging hard                       |
| Why prefer `java.time` API?                        | Immutable, thread-safe, and easier to use                       |

