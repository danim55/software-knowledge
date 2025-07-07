## Understanding the Difference Between Instance and Class Variables

### Instance (Per-object)

* An **instance** is a single **object** created from a class.
* Each object has its **own copy** of instance variables.
* Analogy: Each person has their own name and birthdate.

```java
public class Person {
    private final String name; // Each person has their own name

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

**Usage:**

```java
Person a = new Person("Alice");
Person b = new Person("Bob");

System.out.println(a.getName()); // Alice
System.out.println(b.getName()); // Bob
```

---

### Class (Static, Shared)

* A **class variable** (marked `static`) belongs to the **class itself**, not to any individual object.
* It is shared across **all instances** of that class.
* Analogy: A common rule or version number that applies to everyone.

```java
public class Person {
    private static final String SPECIES = "Homo sapiens"; // Shared

    public static String getSpecies() {
        return SPECIES;
    }
}
```

**Usage:**

```java
System.out.println(Person.getSpecies()); // Homo sapiens
```

Even if you create 10 `Person` objects, there’s still only one copy of `SPECIES`.

---

## Revisiting the Original Question

### What’s the difference between:

```java
private final String something;
```

and

```java
private static final String something;
```

---

### `private final String something;` (Instance / Per-object)

* Each **object** gets its own `something`.
* It must be initialized once per object (e.g., via constructor).
* Analogy: A user's email address — unique to each user.

---

### `private static final String something;` (Class-level / Global constant)

* There is only **one shared copy**, no matter how many objects exist.
* It must be set once when the class is loaded.
* Analogy: The company name — same across all users.

---

## Why `var` Is Not Allowed for Fields

Java does **not** allow `var` for fields (class-level or instance-level). It can only be used for **local variables inside methods**.

This will not compile:

```java
private final var name = "Alice"; // Invalid
```

Instead, use explicit typing:

```java
private final String name = "Alice"; // Valid
```

---

## Summary Table

| Concept        | Keyword                | Scope        | Shared? | Analogy                     |
| -------------- | ---------------------- | ------------ | ------- | --------------------------- |
| Instance Field | `private final`        | Per-object   | No      | A user's email address      |
| Class Field    | `private static final` | Class-wide   | Yes     | The company name            |
| `var`          | Local variables only   | Method-local | No      | Temporary inferred variable |
