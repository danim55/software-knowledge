## Table of Contents

- [Synchronized keyword](#synchronized-keyword)
    - [Using `synchronized` on Instance Methods](#using-synchronized-on-instance-methods)  
    - [Using `synchronized` on Static Methods](#using-synchronized-on-static-methods)  
    - [Using `synchronized` Blocks](#using-synchronized-blocks)  
    - [Reentrant Locks](#reentrant-locks)  


## Synchronized keyword

Simply put, in a multi-threaded environment, a race condition occurs when two or more threads attempt to update mutable shared data at the same time. Java offers a mechanism to avoid race conditions by synchronizing thread access to shared data.

A piece of logic marked with synchronized becomes a synchronized block, allowing only one thread to execute at any given time.

We can use the synchronized keyword on different levels:

- Instance methods
- Static methods
- Code blocks

When we use a synchronized block, Java internally uses a monitor, also known as a monitor lock or intrinsic lock, to provide synchronization. These monitors are bound to an object; therefore, all synchronized blocks of the same object can have only one thread executing them at the same time.

---

### Why `synchronized` Matters

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
```

If two threads call `increment()` simultaneously, `count++` may run non-atomically, leading to lost updates and incorrect counts.

---

### Using `synchronized` on Instance Methods

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

Now, concurrent calls to `increment()` are queued, and threads never interfere.

---

### Using `synchronized` on Static Methods

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

Multiple instances respect the same lock, preventing race conditions on static fields.

---

### Using `synchronized` Blocks

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

This approach scopes the lock explicitly and avoids locking the entire method.

For static contexts:

```java
synchronized (MyClass.class) {
  // operate on static data
}
```

---

### Reentrant Locks

`synchronized` is reentrant: the same thread can re-acquire the same lock without blocking:

```java
synchronized (lock) {
  // first acquisition
  synchronized (lock) {
    // re-entry
  }
}
```

This is safe and supported by the JVM's monitor mechanism.

---

### Summary

| Usage                 | Locking Target     | Behavior                              |
| --------------------- | ------------------ | ------------------------------------- |
| `synchronized` method | `this` or `.class` | Entire method is atomic               |
| `synchronized (obj)`  | Specific object    | Only block is atomic                  |
| Reentrancy            | N/A                | Lock can be re-entered by same thread |

Use the simplest form that ensures thread safety while minimizing unnecessary blocking.
