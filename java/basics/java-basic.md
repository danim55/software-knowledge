## Table of Contents

1. [Understanding the Difference Between Instance and Class Variables](#understanding-the-difference-between-instance-and-class-variables)  
   1. [Instance (Per‑object)](#instance-per-object)  
   1. [Class (Static, Shared)](#class-static-shared)  
   1. [Revisiting the Original Question](#revisiting-the-original-question)  
      1. [`private final String something;`](#private-final-string-something)  
      1. [`private static final String something;`](#private-static-final-string-something)  
1. [Why `var` Is Not Allowed for Fields](#why-var-is-not-allowed-for-fields)  
1. [Summary Table](#summary-table)  

---

## Understanding the Difference Between Instance and Class Variables

### Instance (Per‑object)

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
````

**Usage:**

```java
Person a = new Person("Alice");
Person b = new Person("Bob");

System.out.println(a.getName()); // Alice
System.out.println(b.getName()); // Bob
```

---

### Class (Static, Shared)

* A **class variable** (`static`) belongs to the **class itself**, not to any individual object.
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

---

### Revisiting the Original Question

#### `private final String something;`

* Each **object** gets its own `something`.
* It must be initialized once per object (e.g., via constructor).
* Analogy: A user's email address—unique to each user.

---

#### `private static final String something;`

* Only **one shared copy** exists, regardless of how many objects you create.
* It’s initialized once when the class is loaded.
* Analogy: The company name—same across all users.

---

## Why `var` Is Not Allowed for Fields

Java does **not** allow `var` for fields (class-level or instance-level). It can only be used for **local variables inside methods**.

```java
// This will not compile:
private final var name = "Alice"; // Invalid

// Instead, use explicit typing:
private final String name = "Alice"; // Valid
```

---

## Summary Table

| Concept        | Keyword                | Scope        | Shared? | Analogy                     |
| -------------- | ---------------------- | ------------ | ------- | --------------------------- |
| Instance Field | `private final`        | Per‑object   | No      | A user's email address      |
| Class Field    | `private static final` | Class‑wide   | Yes     | The company name            |
| `var`          | Local variables only   | Method‑local | No      | Temporary inferred variable |

