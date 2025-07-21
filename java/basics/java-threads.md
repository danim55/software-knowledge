````markdown
## Table of Contents

- [Why `synchronized` Matters](#why-synchronized-matters)  
- [Using `synchronized` on Instance Methods](#using-synchronized-on-instance-methods)  
- [Using `synchronized` on Static Methods](#using-synchronized-on-static-methods)  
- [Using `synchronized` Blocks](#using-synchronized-blocks)  
- [Reentrant Locks](#reentrant-locks)  

---

## Why `synchronized` Matters

Without synchronization, concurrent threads can corrupt shared state. For example:

```java
class Counter {
  private int count = 0;

  public void increment() {
    count++;
  }

  public int get() {
    return count;
  }
}
````

If two threads call `increment()` simultaneously, `count++` may run non-atomically, leading to lost updates and incorrect counts ([Baeldung][1]).

---

## Using `synchronized` on Instance Methods

Prefixing a method with `synchronized` ensures only one thread can execute it per object instance:

```java
public class SafeCounter {
  private int count = 0;

  public synchronized void increment() {
    count++;
  }

  public synchronized int get() {
    return count;
  }
}
```

Now, concurrent calls to `increment()` are queued, and threads never interfere ([Baeldung][1]).

---

## Using `synchronized` on Static Methods

Declaring a static method as `synchronized` locks on the class, not the instance, ensuring only one thread per class:

```java
public class StaticCounter {
  private static int count = 0;

  public static synchronized void increment() {
    count++;
  }

  public static synchronized int get() {
    return count;
  }
}
```

Multiple instances respect the same lock, preventing race conditions on static fields ([Baeldung][1]).

---

## Using `synchronized` Blocks

To limit the synchronized region and reduce contention:

```java
public class BlockCounter {
  private int count = 0;
  private final Object lock = new Object();

  public void increment() {
    synchronized (lock) {
      count++;
    }
  }

  public int get() {
    synchronized (lock) {
      return count;
    }
  }
}
```

This approach scopes the lock explicitly and avoids locking the entire method ([Baeldung][2]).

For static contexts:

```java
synchronized (MyClass.class) {
  // operate on static data
}
```

---

## Reentrant Locks

`synchronized` is reentrant: the same thread can re-acquire the same lock without blocking:

```java
synchronized (lock) {
  // first acquisition
  synchronized (lock) {
    // re-entry
  }
}
```

This is safe and supported by the JVM’s monitor mechanism ([Baeldung][1]).

---

## Summary

| Usage                 | Locking Target     | Behavior                              |
| --------------------- | ------------------ | ------------------------------------- |
| `synchronized` method | `this` or `.class` | Entire method is atomic               |
| `synchronized (obj)`  | Specific object    | Only block is atomic                  |
| Reentrancy            | N/A                | Lock can be re-entered by same thread |

Use the simplest form that ensures thread safety while minimizing unnecessary blocking.
