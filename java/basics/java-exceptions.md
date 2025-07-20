## Table of Contents

- [Checked Exceptions](#checked-exceptions)  
- [Unchecked Exceptions](#unchecked-exceptions)  

## Checked Exceptions

Checked exceptions represent conditions a well‑written application should anticipate and recover from. The compiler enforces that you either catch them or declare them with `throws` on your method signature.

**Characteristics**  
- Checked at compile time  
- Typically I/O, networking, and database errors  
- Force explicit handling

**Example with `throws`:**

```java
import java.io.*;

public void readFile(String path) throws IOException {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        System.out.println(reader.readLine());
    }
}
```

**Example with `try-catch`:**

```java
import java.io.*;

public void readFileSafely(String path) {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        System.out.println(reader.readLine());
    } catch (FileNotFoundException e) {
        System.err.println("File not found: " + path);
    } catch (IOException e) {
        System.err.println("I/O error reading file: " + e.getMessage());
    }
}
```

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


## How to Create a Custom Exception in Java

### Table of Contents
- [Why Create Custom Exceptions](#why-create-custom-exceptions)  
- [Custom Checked Exception](#custom-checked-exception)  
- [Custom Unchecked Exception](#custom-unchecked-exception)  

---

## Why Create Custom Exceptions

Standard Java exception types sometimes don't convey enough detail about domain-specific errors. Custom exceptions allow you to:

- Surface **business logic issues** more clearly  
- Catch and handle them separately from generic Java errors :contentReference[oaicite:0]{index=0}  

---

## Custom Checked Exception

Checked exceptions are enforced at compile-time and must be handled or declared.

### 1. Define the exception

```java
public class IncorrectFileNameException extends Exception {
    public IncorrectFileNameException(String message) {
        super(message);
    }

    public IncorrectFileNameException(String message, Throwable cause) {
        super(message, cause);
    }
}
````

### 2. Use it in code

```java
import java.util.Scanner;
import java.io.File;
import java.io.FileNotFoundException;

public String readFirstLine(String fileName) throws IncorrectFileNameException {
    try (Scanner file = new Scanner(new File(fileName))) {
        if (file.hasNextLine()) {
            return file.nextLine();
        }
        return null;
    } catch (FileNotFoundException e) {
        if (!isCorrectFileName(fileName)) {
            throw new IncorrectFileNameException(
              "Incorrect filename: " + fileName, e);
        }
        throw new IncorrectFileNameException("Unable to open file: " + fileName, e);
    }
}
```

---

## Custom Unchecked Exception

Unchecked exceptions extend `RuntimeException`; no need to declare or handle them explicitly.

### 1. Define the exception

```java
public class IncorrectFileExtensionException extends RuntimeException {
    public IncorrectFileExtensionException(String message) {
        super(message);
    }

    public IncorrectFileExtensionException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 2. Throw it at runtime

```java
import java.util.Scanner;
import java.io.FileNotFoundException;

public String readFirstLineWithExtensionCheck(String fileName) {
    try (Scanner file = new Scanner(new File(fileName))) {
        if (file.hasNextLine()) {
            return file.nextLine();
        }
        return null;
    } catch (FileNotFoundException e) {
        // assume name is correct here
        if (!fileName.contains(".")) {
            throw new IncorrectFileExtensionException(
              "Filename missing extension: " + fileName, e);
        }
        throw new RuntimeException("Unexpected error reading file", e);
    }
}
```

---

## Summary

Use **checked exceptions** (`extends Exception`) when the caller can *reasonably recover* from the error.
Use **unchecked exceptions** (`extends RuntimeException`) for programming errors or invalid input the caller can’t fix. ([baeldung.com][1])

This pattern helps you maintain precise, well-structured error handling in your Java applications.

```
::contentReference[oaicite:5]{index=5}
```
