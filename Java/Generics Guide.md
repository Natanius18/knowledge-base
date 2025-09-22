# Generics Guide

Generics were introduced in Java 5 to improve **type safety**, **readability**, and **maintainability**. They allow you to specify the type of objects a class, method, or collection can work with, avoiding runtime casting issues.

---

## Table of Contents
1. [Avoid Raw Types](#1-avoid-raw-types)  
2. [Eliminate Unchecked Warnings](#2-eliminate-unchecked-warnings)  
3. [Prefer Lists to Arrays](#3-prefer-lists-to-arrays)  
4. [Favor Generic Types](#4-favor-generic-types)  
5. [Favor Generic Methods](#5-favor-generic-methods)  
6. [Use Bounded Wildcards](#6-use-bounded-wildcards)  
7. [Minimize Scope of Type Parameters](#7-minimize-scope-of-type-parameters)  
8. [Use Bounded Type Parameters](#8-use-bounded-type-parameters)  
9. [Prefer Class Literals](#9-prefer-class-literals)  
10. [Special Cases and Pitfalls](#10-special-cases-and-pitfalls)  
11. [Generics and Varargs](#11-generics-and-varargs)  
12. [Best Practices](#12-best-practices)  
13. [Generics Restrictions Cheat Sheet](#13-generics-restrictions-cheat-sheet)

---

## 1. Avoid Raw Types
Raw types disable generics and lose type safety.  
Prefer `Set<?>` over `Set` **only** when you do not know the element type. If you know the type, always use `Set<T>`.

✅
```java
Set<?> safeSet = new HashSet<String>();
````

❌

```java
Set rawSet = new HashSet();
```

> **Note:** With `Set<?>`, you cannot add any elements except `null`.

---

## 2. Eliminate Unchecked Warnings

Unchecked warnings signal a possible `ClassCastException`. Fix or safely suppress them with a clear explanation.

Correct `Arrays.copyOf` usage:

```java
public static <T> T[] toArray(Collection<T> c, T[] a) {
    Object[] elements = c.toArray();
    @SuppressWarnings("unchecked") // Safe: runtime type matches T[]
    T[] result = (T[]) Arrays.copyOf(elements, elements.length, a.getClass());
    return result;
}
```

---

## 3. Prefer Lists to Arrays

Arrays are *covariant* and *reified* (arrays know their element type at runtime), whereas generics are *invariant* and *erased*.
This mismatch makes arrays and generics tricky.

❌ Illegal:

```java
T arr[] = new T[10];                // Error: generic array creation
Gen<Integer>[] arr = new Gen<Integer>[10]; // Error
new Gen<?>[10];                     // Error
```

✅ Workarounds:
Prefer collections:

```java
List<T> list = new ArrayList<>(); // preferred
```

If you must have an array:

```java
@SuppressWarnings("unchecked")
T[] arr = (T[]) java.lang.reflect.Array.newInstance(clazz, 10);

@SuppressWarnings("unchecked")
Gen<String>[] arr2 = (Gen<String>[]) new Gen[10]; // unchecked but allowed
```

---

## 4. Favor Generic Types

Instead of `Object`, prefer parameterized types:

```java
public class Gen<T> {
    T value;

    // ❌ Illegal: cannot do new T()
    // Gen() { value = new T(); }

    // ✅ Workaround: pass Class<T>
    Gen(Class<T> clazz) throws Exception {
        value = clazz.getDeclaredConstructor().newInstance();
    }
}
```

---

## 5. Favor Generic Methods

Parameterize methods to increase reusability:

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

---

## 6. Use Bounded Wildcards

Apply PECS: **Producer `extends`, Consumer `super`**.

Generic copy method:

```java
public static <T> void copy(List<? extends T> src, List<? super T> dest) {
    for (T t : src) dest.add(t);
}
```

---

## 7. Minimize Scope of Type Parameters

Don’t overuse `<T>` if a wildcard suffices.

❌

```java
public <T> boolean isEmpty(List<T> list) { return list.size() == 0; }
```

✅

```java
public static boolean isEmpty(List<?> list) { return list.isEmpty(); }
```

---

## 8. Use Bounded Type Parameters

Bounded type parameters restrict type arguments.


```java
public static <T extends Comparable<? super T>> T max(Collection<? extends T> coll) {
    Iterator<? extends T> it = coll.iterator();
    if (!it.hasNext()) throw new NoSuchElementException();
    T max = it.next();
    while (it.hasNext()) {
        T e = it.next();
        if (max.compareTo(e) < 0) max = e;
    }
    return max;
}
```

Multiple bounds:

```java
<T extends Number & Comparable<? super T>>
```

---

## 9. Prefer Class Literals

* `List.class` ✅
* `List<String>.class` ❌ (illegal)
* `List<?>.class` ❌ (illegal)

Use `Class<T>` tokens instead:

```java
public <T> T createInstance(Class<T> clazz) throws Exception {
    return clazz.getDeclaredConstructor().newInstance();
}
```

---

## 10. Special Cases

### Arrays of Generics

* `new T[10]` ❌
* `new Gen<Integer>[10]` ❌
* `new Gen<?>[10]` ❌

Instead, use:

```java
@SuppressWarnings("unchecked")
Gen<String>[] arr = (Gen<String>[]) new Gen[10];
```

or prefer `List<Gen<String>>`.

### Static Fields in Generics

Generic types cannot have type-parameter-dependent static fields:

```java
class Gen<T> {
    // ❌ Not allowed
    // static T someObj;
}
```

All instantiations of a generic class share the same static members.

### Set vs Set\<?>

* `Set` (raw) loses type safety.
* `Set<?>` is safe but read-only (except `null`).
* Use `Set<T>` whenever you know the element type.

### Map with Class as Key

Heterogeneous container pattern:

```java
private final Map<Class<?>, Object> registry = new HashMap<>();

public <T> void put(Class<T> type, T instance) {
    registry.put(Objects.requireNonNull(type), type.cast(instance));
}

public <T> T get(Class<T> type) {
    return type.cast(registry.get(type));
}
```

---

## 11. Generics and Varargs

Varargs with generics cause warnings due to heap pollution risk.
Safe if the array is not exposed or modified externally.

Correct usage:

```java
@SafeVarargs
public static <T> List<T> asListSafe(T... elements) {
    // Copy to avoid exposing the backing array
    return new ArrayList<>(Arrays.asList(elements));
}
```

> **Note:** `@SafeVarargs` is allowed only on **static**, **final**, or **private** methods/constructors.

---

## 12. Best Practices

* **Never use raw types** (except with `instanceof`).
* **Prefer `List` to arrays** in generic code.
* **Eliminate unchecked warnings** or clearly document why suppression is safe.
* **Use PECS**:

  * Producer → `? extends T`
  * Consumer → `? super T`
* **Keep type parameters minimal** – avoid `<T>` when `?` suffices.
* **Avoid static fields of type parameters**.
* **Document why `@SuppressWarnings` or `@SafeVarargs` is safe**.
* **Use `Class<T>` tokens** to safely instantiate generic types.

---

## 13. Generics Restrictions Cheat Sheet

Java Generics come with several important restrictions due to **type erasure**.
Here is a compact reference table:

| Restriction                           | Example (❌ illegal)                              | Workaround / Explanation                           |
| ------------------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| **No primitive types**                | `List<int> nums;`                                | Use wrapper types: `List<Integer>`                 |
| **No `new T()`**                      | `T obj = new T();`                               | Use `clazz.getDeclaredConstructor().newInstance()` |
| **No `new T[]`**                      | `T[] arr = new T[10];`                           | Use `Array.newInstance(clazz,10)` or raw cast      |
| **No generic array creation**         | `new List<String>[10]`                           | Use `List<List<String>>`                           |
| **No static fields of T**             | `static T value;`                                | Use static of raw type or move outside generic     |
| **No parameterized literals**         | `List<String>.class`                             | Only `List.class` is legal                         |
| **No generic exception types**        | `class MyEx<T> extends Exception {}`             | Use non-generic exception and cast payload         |
| **Type erasure removes runtime type** | `obj instanceof List<String>`                    | Use `obj instanceof List<?>`                       |
| **Cannot overload by type parameter** | `void f(List<String>)` / `void f(List<Integer>)` | Both erase to `f(List)`                            |
| **Cannot instantiate wildcard types** | `new ?()`                                        | Wildcards are only for references                  |
| **Varargs + generics warning**        | `List<String>... lists`                          | Use `@SafeVarargs` if safe                         |

---

### Sources

* [Oracle Generics Tutorial](https://docs.oracle.com/javase/tutorial/java/generics/)
* [Medium](https://medium.com/@engmohamedsayed20102010/effective-java-generics-d706f208ce85)
* [Effective Java Generics](https://ahdak.github.io/blog/effective-java-part-4/)
