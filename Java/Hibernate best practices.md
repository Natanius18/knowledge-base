# Hibernate Best Practices

## 📋 Contents

1. [Entity Design](#1-entity-design)
2. [Primary Keys and Identifiers](#2-primary-keys-and-identifiers)
3. [Relationships and Associations](#3-relationships-and-associations)
4. [Fetching Strategies](#4-fetching-strategies)
5. [Caching](#5-caching)
6. [Transaction Management](#6-transaction-management)
7. [Query Optimization](#7-query-optimization)
8. [Session Management](#8-session-management)
9. [Batch Operations](#9-batch-operations)
10. [Common Pitfalls](#10-common-pitfalls)
11. [Best Practices Summary](#11-best-practices-summary)
12. [Interview Q&A](#12-interview-qa)

***

## 1. Entity Design

### Use `@Entity` correctly

Entities must have:

- No-argument constructor (can be private)
- Non-final class
- Non-final fields for lazy loading to work

✅ **Correct:**

```java

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    protected User() {
    } // for Hibernate

    public User(String name) {
        this.name = name;
    }
}
```

❌ **Wrong:**

```java

@Entity
public final class User { // final class breaks proxying
    private final String name; // final fields break lazy loading
}
```

***

### Override `equals()` and `hashCode()`

**Rule:** Base them on **business keys**, not `@Id`.

✅ **Correct:**

```java

@Entity
public class User {
    @Id
    @GeneratedValue
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return email != null && email.equals(user.email);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

**Why:**

- `id` is `null` before persistence
- Collections (`Set`, `Map`) rely on stable hash codes
- Using `id` breaks when adding transient entities to collections

***

### Use `@Column` for constraints

```java

@Column(name = "user_name", nullable = false, length = 100)
private String name;

@Column(name = "email", unique = true, updatable = false)
private String email;
```

***

### Avoid Lombok `@Data` on entities

❌ **Avoid:**

```java

@Data
@Entity
public class User { ...
}
```

**Problems:**

- Generates `equals()` based on all fields (including `id`)
- Includes lazy collections in `toString()` → N+1
- Can trigger lazy initialization exceptions

✅ **Use:**

```java

@Getter
@Setter
@ToString(exclude = {"orders"}) // exclude lazy collections
@EqualsAndHashCode(of = "email") // use business key
@Entity
public class User { ...
}
```

***

## 2. Primary Keys and Identifiers

### Use `Long` or `UUID` for IDs

✅ **Long (Auto-increment):**

```java

@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

✅ **UUID (Distributed systems):**

```java

@Id
@GeneratedValue(generator = "UUID")
@GenericGenerator(name = "UUID", strategy = "org.hibernate.id.UUIDGenerator")
@Column(updatable = false, nullable = false)
private UUID id;
```

***

### Avoid composite keys unless necessary

Composite keys add complexity and performance overhead.

❌ **Avoid:**

```java

@EmbeddedId
private OrderItemId id;

@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
}
```

✅ **Prefer:**

```java

@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

@ManyToOne
private Order order;

@ManyToOne
private Product product;
```

***

### Choose the right `GenerationType`

| Strategy   | Use Case                         | Performance | Database Support |
|------------|----------------------------------|-------------|------------------|
| `IDENTITY` | Single DB, simple apps           | Medium      | MySQL, Postgres  |
| `SEQUENCE` | High performance, batch inserts  | ✅ Best      | Postgres, Oracle |
| `TABLE`    | DB-agnostic (rarely recommended) | ❌ Slow      | All              |
| `AUTO`     | Let Hibernate decide             | Varies      | All              |

**Recommendation:** Use `SEQUENCE` for production apps.

```java

@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 50)
private Long id;
```

***

## 3. Relationships and Associations

### Always use `@ManyToOne` with `@JoinColumn`

```java

@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id", nullable = false)
private User user;
```

***

### Be careful with `@OneToMany`

**Default is `LAZY`**, which is good, but always use `mappedBy`:

✅ **Correct:**

```java

@Entity
public class User {
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;
}
```

**Why `mappedBy`?**

- Avoids extra join table
- Makes `Order` the owner of the relationship

***

### Avoid `@OneToOne` on the non-owning side

`@OneToOne` breaks lazy loading on the inverse side.

❌ **Problem:**

```java

@Entity
public class User {
    @OneToOne(mappedBy = "user", fetch = FetchType.LAZY)
    private Profile profile; // still eagerly loaded!
}
```

✅ **Workaround:**
Use `@MapsId` or redesign as `@ManyToOne`.

***

### Use `CascadeType` carefully

```java

@OneToMany(mappedBy = "user", cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private List<Order> orders;
```

**Avoid `CascadeType.ALL`** unless you're sure.

**Never cascade `@ManyToOne`** — it can delete parent entities unexpectedly.

***

### Use `orphanRemoval` for strict parent-child relationships

```java

@OneToMany(mappedBy = "user", orphanRemoval = true)
private List<Order> orders;
```

Deletes `Order` when removed from the collection.

***

## 4. Fetching Strategies

### Default to `LAZY` loading

✅ **Correct:**

```java

@ManyToOne(fetch = FetchType.LAZY)
private User user;

@OneToMany(mappedBy = "user", fetch = FetchType.LAZY) // default
private List<Order> orders;
```

**Why:**

- Eager loading causes N+1 problems
- Load only what you need

***

### Use `JOIN FETCH` for specific queries

```java

@Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") Long id);
```

***

### Use `@EntityGraph` for flexible fetching

```java

@EntityGraph(attributePaths = {"orders", "profile"})
@Query("SELECT u FROM User u WHERE u.id = :id")
Optional<User> findByIdWithGraph(@Param("id") Long id);
```

***

### Avoid `FetchType.EAGER`

❌ **Bad:**

```java

@OneToMany(fetch = FetchType.EAGER)
private List<Order> orders; // always loads all orders
```

**Problem:** Cannot be overridden, always loads related data.

✅ **Good:** Use `LAZY` + `@EntityGraph` or `JOIN FETCH` per query.

***

## 5. Caching

### Use second-level cache for read-heavy data

Enable in `application.properties`:

```properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```

Mark entities:

```java

@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Category {
    @Id
    private Long id;
    private String name;
}
```

***

### Use query cache for repeated queries

```java

@QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
@Query("SELECT c FROM Category c WHERE c.active = true")
List<Category> findActiveCategories();
```

***

### Understand cache strategies

| Strategy               | Use Case                              |
|------------------------|---------------------------------------|
| `READ_ONLY`            | Immutable data (reference tables)     |
| `READ_WRITE`           | Frequently read, occasionally updated |
| `NONSTRICT_READ_WRITE` | Tolerate stale data                   |
| `TRANSACTIONAL`        | Strict consistency (JTA only)         |

---

## 6. Transaction Management

### Always use transactions

❌ **Wrong:**

```java
public void saveUser(User user) {
    userRepository.save(user); // no transaction boundary
}
```

✅ **Correct:**

```java

@Transactional
public void saveUser(User user) {
    userRepository.save(user);
}
```

***

### Use `@Transactional` at the service layer

```java

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void createUser(String name) {
        User user = new User(name);
        userRepository.save(user);
    }
}
```

***

### Set appropriate isolation levels

```java

@Transactional(isolation = Isolation.READ_COMMITTED)
public void processOrder(Long orderId) { ...}
```

#### Transaction Isolation Levels — Anomalies Overview

Different isolation levels protect against specific concurrency problems. Higher isolation = fewer anomalies but lower performance.

| **Isolation Level**        | **Dirty Read** | **Lost Updates** | **Nonrepeatable Read** | **Phantom Reads** |
|----------------------------|----------------|------------------|------------------------|-------------------|
| `READ_UNCOMMITTED`         | ✅ Yes          | ✅ Yes            | ✅ Yes                  | ✅ Yes             |
| `READ_COMMITTED` (default) | ❌ No           | ✅ Yes            | ✅ Yes                  | ✅ Yes             |
| `REPEATABLE_READ`          | ❌ No           | ❌ No             | ❌ No                   | ✅ Yes             |
| `SERIALIZABLE`             | ❌ No           | ❌ No             | ❌ No                   | ❌ No              |
| `SNAPSHOT`*                | ❌ No           | ❌ No             | ❌ No                   | ❌ No              |

\* Snapshot isolation is supported by some databases (SQL Server, PostgreSQL with SSI).

***

#### What each anomaly means:

| Anomaly                | Problem                                                     | Example                                                                             |
|------------------------|-------------------------------------------------------------|-------------------------------------------------------------------------------------|
| **Dirty Read**         | Reading uncommitted changes from another transaction        | T1 writes X=10 (not committed), T2 reads X=10, T1 rolls back → T2 read invalid data |
| **Lost Updates**       | Concurrent writes overwrite each other                      | T1 reads X=5, T2 reads X=5, T1 writes X=10, T2 writes X=15 → T1's update is lost    |
| **Nonrepeatable Read** | Same query returns different results within one transaction | T1 reads X=5, T2 updates X=10 and commits, T1 reads X again → X=10 (changed)        |
| **Phantom Reads**      | New rows appear/disappear in range queries                  | T1 selects COUNT(*) → 10, T2 inserts new row, T1 selects COUNT(*) again → 11        |

***

#### When to use each level:

| Level              | Use Case                                              | Performance |
|--------------------|-------------------------------------------------------|-------------|
| `READ_UNCOMMITTED` | Read-heavy analytics, logs (dirty reads acceptable)   | Fastest     |
| `READ_COMMITTED`   | Most applications (default for Postgres, Oracle)      | Fast        |
| `REPEATABLE_READ`  | Financial calculations, reports requiring consistency | Moderate    |
| `SERIALIZABLE`     | Critical operations (banking, inventory)              | Slowest     |

***

### Mark read-only transactions

```java

@Transactional(readOnly = true)
public List<User> findAll() {
    return userRepository.findAll();
}
```

**Benefits:**

- Hibernate skips dirty checking
- Some DBs optimize read-only transactions

***

## 7. Query Optimization

### Avoid [N+1 problem](https://github.com/Natanius18/knowledge-base/blob/main/Java/Issues%20in%20code.md#n1-problem)

❌ **Bad:**

```java
List<User> users = userRepository.findAll();
for(
User user :users){
    System.out.

println(user.getOrders().

size()); // N queries
    }
```

✅ **Solution 1: JOIN FETCH**

```java

@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

✅ **Solution 2: @EntityGraph**

```java

@EntityGraph(attributePaths = "orders")
List<User> findAll();
```

✅ **Solution 3: @BatchSize**

```java

@OneToMany(mappedBy = "user")
@BatchSize(size = 10)
private List<Order> orders;
```

***

### Use pagination for large datasets

```java
Pageable pageable = PageRequest.of(0, 20);
Page<User> users = userRepository.findAll(pageable);
```

***

### Use projections for read-only data

**DTO projection:**

```java
public interface UserSummary {
    String getName();

    String getEmail();
}

List<UserSummary> findAllProjectedBy();
```

**Query-based DTO:**

```java

@Query("SELECT new com.example.UserDTO(u.name, u.email) FROM User u")
List<UserDTO> findAllDTOs();
```

***

### Index foreign keys and search columns

```java

@Entity
@Table(name = "orders", indexes = {
    @Index(name = "idx_user_id", columnList = "user_id"),
    @Index(name = "idx_order_date", columnList = "order_date")
})
public class Order { ...
}
```

***

## 8. Session Management

### Use `getCurrentSession()` in Hibernate-only apps

```java
Session session = sessionFactory.getCurrentSession();
```

***

### Close sessions in plain Hibernate

```java
Session session = sessionFactory.openSession();
Transaction tx = null;
try{
tx =session.

beginTransaction();
// work
    tx.

commit();
}catch(
Exception e){
    if(tx !=null)tx.

rollback();
    throw e;
}finally{
    session.

close();
}
```

***

### Spring manages sessions automatically

With Spring Data JPA, you don't manage sessions manually:

```java

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void updateUser(Long id, String newName) {
        User user = userRepository.findById(id).orElseThrow();
        user.setName(newName);
        // no need to call save() — managed by persistence context
    }
}
```

***

## 9. Batch Operations

### Enable batch inserts

```properties
spring.jpa.properties.hibernate.jdbc.batch_size=20
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
```

***

### Use `SEQUENCE` generator for batching

```java

@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", allocationSize = 50)
private Long id;
```

**Why:** `IDENTITY` disables batch inserts.

***

### Flush and clear the session periodically

```java
for(int i = 0;
i< 10000;i++){
User user = new User("User" + i);
    entityManager.

persist(user);
    
    if(i %20==0){
    entityManager.

flush();
        entityManager.

clear();
    }
        }
```

***

## 10. Common Pitfalls

### LazyInitializationException

**Cause:** Accessing lazy-loaded data outside a transaction.

❌ **Wrong:**

```java

@Transactional
public User getUser(Long id) {
    return userRepository.findById(id).orElseThrow();
}

// Controller
User user = userService.getUser(1L);
user.

getOrders().

size(); // LazyInitializationException
```

✅ **Fix 1: Fetch eagerly in service**

```java

@Transactional
public User getUserWithOrders(Long id) {
    return userRepository.findByIdWithOrders(id);
}
```

✅ **Fix 2: Use DTO**

```java

@Transactional
public UserDTO getUser(Long id) {
    User user = userRepository.findById(id).orElseThrow();
    return new UserDTO(user.getName(), user.getOrders().size());
}
```

***

### MultipleBagFetchException

**Cause:** Using multiple `@OneToMany` with `JOIN FETCH`.

❌ **Wrong:**

```java
@Query("SELECT u FROM User u JOIN FETCH u.orders JOIN FETCH u.addresses")
```

✅ **Fix 1: Use `@EntityGraph`**

```java
@EntityGraph(attributePaths = {"orders", "addresses"})
```

✅ **Fix 2: Use separate queries**

```java
User user = userRepository.findById(id).orElseThrow();
Hibernate.

initialize(user.getOrders());
    Hibernate.

initialize(user.getAddresses());
```

***

### Detached entities

**Problem:** Modifying entities outside transaction context.

✅ **Fix: Use `merge()`**

```java

@Transactional
public void update(User detachedUser) {
    entityManager.merge(detachedUser);
}
```

***

### Open Session In View (OSIV)

**Default:** `spring.jpa.open-in-view=true`

**Problem:**

- Keeps database connections open during request processing
- Can cause N+1 in views/controllers

✅ **Recommendation:** Disable it.

```properties
spring.jpa.open-in-view=false
```

Load all data in the service layer.

***

## 11. Best Practices Summary

- ✅ **Always use `LAZY` loading** by default
- ✅ **Override `equals()` and `hashCode()`** based on business keys
- ✅ **Use `@Transactional`** at the service layer
- ✅ **Use `JOIN FETCH` or `@EntityGraph`** to avoid N+1
- ✅ **Prefer `SEQUENCE` generator** for better performance
- ✅ **Index foreign keys** and frequently queried columns
- ✅ **Use DTOs** for read-only queries
- ✅ **Enable second-level cache** for reference data
- ✅ **Flush and clear** sessions in batch operations
- ✅ **Disable OSIV** in production
- ✅ **Use projections** to fetch only needed fields
- ❌ **Avoid `FetchType.EAGER`**
- ❌ **Avoid Lombok `@Data`** on entities
- ❌ **Avoid composite keys** unless necessary
- ❌ **Don't use `IDENTITY` with batch inserts**

***

## 12. Interview Q&A

| Question                                                      | Short Answer                                                          |
|---------------------------------------------------------------|-----------------------------------------------------------------------|
| What is `LazyInitializationException`?                        | Accessing lazy data outside a transaction                             |
| How to fix it?                                                | Fetch data in transaction or use DTOs                                 |
| What's the difference between `persist()` and `merge()`?      | `persist()` for new, `merge()` for detached entities                  |
| What is the best `GenerationType` for production?             | `SEQUENCE` (allows batch inserts)                                     |
| Why avoid `FetchType.EAGER`?                                  | Causes unnecessary data loading and N+1                               |
| What is `orphanRemoval`?                                      | Deletes child entities when removed from parent collection            |
| What is the difference between `save()` and `saveAndFlush()`? | `saveAndFlush()` immediately syncs with DB                            |
| Why avoid `@Data` on entities?                                | Breaks `equals()`/`hashCode()`, triggers lazy loading in `toString()` |
| What is `@EntityGraph`?                                       | Defines fetch plan per query without changing entity mappings         |
| What is OSIV?                                                 | Open Session In View — keeps session open during request              |
| Should you enable OSIV in production?                         | ❌ No — causes connection leaks and N+1                                |

***

### Sources

- [Hibernate Documentation](https://hibernate.org/orm/documentation/)
- [Vlad Mihalcea's Blog](https://vladmihalcea.com/)
- [Geeks for geeks](https://www.geeksforgeeks.org/dbms/transaction-isolation-levels-dbms/)
- [Snapshot Isolation vs Serializable](https://www.geeksforgeeks.org/dbms/snapshot-isolation-vs-serializable/)
- [Habr](https://habr.com/ru/articles/845522/)
