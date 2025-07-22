## Table of Contents

- [Overview](#overview)  
- [Core Stereotype Annotations](#core-stereotype-annotations)  
  - [@Component](#component)  
  - [@Repository](#repository)  
  - [@Service](#service)  
  - [@Controller](#controller)  
- [@Configuration & @Bean](#configuration--bean)  

## Overview

Spring allows for automatic bean detection using class-level stereotype annotations in the `org.springframework.stereotype` package :contentReference[oaicite:0]{index=0}. These annotations simplify wiring and promote clean, layered application structure.

---

## Core Stereotype Annotations

### @Component

Marks any class to be auto-detected and registered as a bean during component scanning :contentReference[oaicite:1]{index=1}.

```java
@Component
public class CarUtility { … }
```

### @Repository

Specialized for data access objects (DAOs). Adds persistence exception translation automatically ([Baeldung][1]).

```java
@Repository
public class VehicleRepository { … }
```

### @Service

Designates a service-layer component, encapsulating business logic ([Baeldung][1]).

```java
@Service
public class VehicleService { … }
```

### @Controller

Defines a Spring MVC web controller to handle HTTP requests ([Baeldung][1]).

```java
@Controller
public class VehicleController { … }
```

---

## @Configuration & @Bean

Use `@Configuration` classes to define beans via factory methods labeled with `@Bean`.

```java
@Configuration
public class AppConfig {
  @Bean
  public Engine engine() {
    return new Engine();
  }
}
```

This creates a Spring-managed `Engine` bean named "engine" ([Baeldung][2]).

---

## Summary

These annotation types enable modular design and clean dependency injection:

* **@Component** – generic bean
* **@Repository** – database layer
* **@Service** – business logic
* **@Controller** – web endpoints
* **@Configuration/@Bean** – manual bean definitions

Use them to structure your application clearly, leveraging Spring’s powerful auto-detection and DI functionality.

[1]: https://www.baeldung.com/spring-bean-annotations?utm_source=chatgpt.com "Spring Bean Annotations"
[2]: https://www.baeldung.com/spring-core-annotations?utm_source=chatgpt.com "Spring Core Annotations"


### Table of Contents

1. [Component in Spring Boot](#component-in-spring-boot)

## `@Component` in Spring Boot

In Spring Boot, the `@Component` annotation is used to define a **Spring-managed bean**. When a class is annotated with `@Component`, it becomes a **candidate for component scanning**, allowing Spring to automatically detect and register it in the **application context** at startup.

By default, all components in Spring are **singleton-scoped**, meaning only **one instance** of the class will be created and shared across the entire application.

### Key Concepts

* **Bean:** An object that is managed by the Spring IoC (Inversion of Control) container.
* **Application Context:** A container that holds and manages beans in a Spring application.
* **Singleton (default):** Only one shared instance of the bean exists within the Spring context.
* **Component Scanning:** Spring automatically finds and registers components based on package structure.

### Common Related Annotations

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
