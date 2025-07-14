## Table of Contents

- [Checked Exceptions](#checked-exceptions)  
- [Unchecked Exceptions](#unchecked-exceptions)  

## Checked Exceptions

Checked exceptions represent conditions a well‑written application should anticipate and recover from. The compiler enforces that you either catch them or declare them with `throws` on your method signature.

**Characteristics**  
- Checked at compile time  
- Typically I/O, networking, and database errors  
- Force explicit handling

**Example:**

```java
import java.io.*;

public void readFile(String path) throws IOException {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        System.out.println(reader.readLine());
    }
}
````

Here, `FileNotFoundException` and `IOException` are checked, so the method must declare `throws IOException` or catch it.

---

## Unchecked Exceptions

Unchecked exceptions (subclasses of `RuntimeException`) indicate programming errors that could have been prevented by proper code (null checks, valid indices, etc.). The compiler does not require you to catch or declare them.

**Characteristics**

* Not checked at compile time
* Often signify bugs (e.g., null dereference, invalid casts)
* Can propagate freely

**Examples:**

```java
// Null pointer
String s = null;
int len = s.length(); // throws NullPointerException

// Bad array access
int[] a = {1, 2, 3};
int x = a[3];        // throws ArrayIndexOutOfBoundsException
```

Unchecked exceptions can be caught if desired, but you are not forced to do so.