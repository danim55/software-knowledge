### Table of Contents

1. [Understanding Component Annotation](#component-in-spring-boot)
    1. [Key Concepts](#key-concepts)
        1. [Common Related Annotations](#common-related-annotations)
        1. [Example](#example)
        1. [Notes](#notes)
        1. [When to Use](#when-to-use)

# `@Component` in Spring Boot

In Spring Boot, the `@Component` annotation is used to define a **Spring-managed bean**. When a class is annotated with `@Component`, it becomes a **candidate for component scanning**, allowing Spring to automatically detect and register it in the **application context** at startup.

By default, all components in Spring are **singleton-scoped**, meaning only **one instance** of the class will be created and shared across the entire application.

### Key Concepts

* **Bean:** An object that is managed by the Spring IoC (Inversion of Control) container.
* **Application Context:** A container that holds and manages beans in a Spring application.
* **Singleton (default):** Only one shared instance of the bean exists within the Spring context.
* **Component Scanning:** Spring automatically finds and registers components based on package structure.

#### Common Related Annotations

* `@Component`: Generic stereotype for a Spring-managed component.
* `@Service`: Specialization of `@Component` used in the service layer.
* `@Repository`: Specialization of `@Component` used in the persistence layer.
* `@Controller`: Specialization of `@Component` used in the web layer.

All of the above are treated as `@Component` internally, but provide better semantic clarity.

#### Example

```java
import org.springframework.stereotype.Component;

@Component
public class MyComponent {

    public void sayHello() {
        System.out.println("Hello from MyComponent!");
    }
}
```

Spring will automatically instantiate `MyComponent` at application startup and make it available for injection:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final MyComponent myComponent;

    @Autowired
    public MyService(MyComponent myComponent) {
        this.myComponent = myComponent;
    }

    public void useComponent() {
        myComponent.sayHello();
    }
}
```

#### Notes

* `@Component` works only if **component scanning** is properly configured (automatically enabled with `@SpringBootApplication` if packages are structured correctly).
* If you want multiple instances (not singleton), you must explicitly define a different scope using `@Scope("prototype")`, or manage instantiation manually.

#### When to Use

Use `@Component` for any generic class that should be automatically detected and managed by Spring. If the component plays a specific role (e.g., service, repository, controller), prefer the more specific annotations for clarity.
