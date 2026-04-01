# Spring Boot Annotations — Complete Master Guide

> A structured interview-ready reference for Java developers.  
> Every annotation: definition → why → where → how it works → example → common mistakes → interview tip.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Core Annotations](#part-1--core-annotations)
   - [@SpringBootApplication](#springbootapplication)
   - [@Component, @Service, @Repository, @Controller, @RestController](#component-service-repository-controller-restcontroller)
   - [@Autowired](#autowired)
   - [@Configuration and @Bean](#configuration-and-bean)
3. [Web Annotations](#part-2--web-annotations)
   - [@RequestMapping, @GetMapping, @PostMapping](#requestmapping-getmapping-postmapping)
   - [@PathVariable, @RequestParam, @RequestBody](#pathvariable-requestparam-requestbody)
4. [Exception Handling](#part-3--exception-handling)
   - [@ExceptionHandler](#exceptionhandler)
   - [@ControllerAdvice](#controlleradvice)
5. [JPA Annotations](#part-4--jpa-annotations)
   - [@Entity, @Table, @Id, @GeneratedValue](#entity-table-id-generatedvalue)
   - [@OneToMany and @ManyToOne](#onetomany-and-manytoone)
   - [@Transactional](#transactional)
6. [Advanced Annotations](#part-5--advanced-annotations)
   - [@Profile](#profile)
   - [@Conditional](#conditional)
   - [@Async](#async)
   - [@Scheduled](#scheduled)
   - [@Cacheable](#cacheable)
7. [Cheatsheet by Layer](#part-6--cheatsheet-by-layer)
8. [10 Interview Questions & Answers](#part-7--10-common-interview-questions-with-answers)
9. [Additional Tips](#additional-tips)

---

## Architecture Overview

```
Client
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  Controller Layer                                       │
│  @RestController  @Controller  @RequestMapping          │
│  @GetMapping  @PostMapping  @PathVariable               │
│  @RequestParam  @RequestBody                            │
│  @ExceptionHandler  @ControllerAdvice                   │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  Service Layer                                          │
│  @Service  @Transactional  @Async                       │
│  @Scheduled  @Cacheable  @Autowired  @Component         │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│  Repository / JPA Layer                                 │
│  @Repository  @Entity  @Table  @Id  @GeneratedValue     │
│  @OneToMany  @ManyToOne  @Column  @JoinColumn           │
└─────────────────────────────────────────────────────────┘

─────────────────────────────────────────────────────────
  Config / Cross-cutting
  @SpringBootApplication  @Configuration  @Bean
  @Profile  @Conditional
─────────────────────────────────────────────────────────
```

---

## Part 1 — Core Annotations

---

### `@SpringBootApplication`

#### 1. Definition (Interview-ready)
A convenience annotation that combines three annotations: `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`. It marks the main entry point of a Spring Boot application.

#### 2. Why it is used
- **Problem it solves:** Before Spring Boot, you had to manually configure the entire Spring context — data sources, MVC, security, etc. This was verbose and error-prone.
- **Why not alternatives:** You could use the three individual annotations separately, but `@SpringBootApplication` is the standard, idiomatic choice and reduces boilerplate without hiding any functionality.

#### 3. Where it is used
- Always on the **main class** — the one with `public static void main(String[] args)`.
- Used exactly **once** per application.

#### 4. How it works internally
- `@EnableAutoConfiguration` tells Spring Boot to scan the classpath and auto-configure beans based on what's present (e.g., if `spring-data-jpa` is on the classpath, it auto-configures a `DataSource`).
- `@ComponentScan` scans the current package and all sub-packages for `@Component`, `@Service`, `@Repository`, etc.
- `@Configuration` marks the class as a source of bean definitions.
- Spring Boot uses `spring.factories` / `AutoConfiguration.imports` files inside JAR dependencies to discover which auto-configurations to apply.

#### 5. Example
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

#### 6. Common Mistakes
- Placing the main class in the **default (root) package** — component scanning won't work correctly.
- Having **multiple `@SpringBootApplication` classes** — creates duplicate context issues.
- Forgetting that `basePackages` can be customized: `@SpringBootApplication(scanBasePackages = "com.example")`.

#### 7. Interview Tip
> *"@SpringBootApplication is a meta-annotation that combines @Configuration, @EnableAutoConfiguration, and @ComponentScan. The most important part is @EnableAutoConfiguration — it reads the spring.factories file from all JARs on the classpath and conditionally configures beans. So if I add the JPA starter, Spring Boot automatically sets up EntityManagerFactory, DataSource, and TransactionManager — I don't have to write a single line of config."*

---

### `@Component`, `@Service`, `@Repository`, `@Controller`, `@RestController`

#### 1. Definition (Interview-ready)
These are **stereotype annotations** that mark a class as a Spring-managed bean. `@Component` is the base annotation. The others are specialized versions that add semantic meaning and sometimes extra behaviour.

#### 2. Why it is used

| Annotation | Extra behaviour beyond @Component |
|---|---|
| `@Component` | None — generic bean |
| `@Service` | Semantic marker for business logic |
| `@Repository` | Translates database exceptions into Spring's `DataAccessException` hierarchy |
| `@Controller` | Marks as an MVC controller, works with view templates |
| `@RestController` | `@Controller` + `@ResponseBody` — every method returns data directly (JSON/XML), not a view name |

- **Problem:** Without these, Spring doesn't know which classes to manage and wire.
- **Why not `new` keyword:** Instantiating manually breaks dependency injection, lifecycle management, AOP proxying, and transaction management.

#### 3. Where they are used
- `@Controller` / `@RestController` → Web layer
- `@Service` → Business logic layer
- `@Repository` → Data access layer
- `@Component` → Utility classes, helper beans, anything that doesn't fit cleanly into the other layers

#### 4. How it works internally
- During application startup, `@ComponentScan` scans packages for classes annotated with these (they're all meta-annotated with `@Component`).
- Spring creates a `BeanDefinition` for each and registers it in the `ApplicationContext`.
- For `@Repository`, Spring wraps the bean in a proxy that catches database-specific exceptions and re-throws them as `DataAccessException` subclasses.
- For `@RestController`, the `@ResponseBody` triggers `HttpMessageConverters` (like Jackson) to serialize the return value directly to the HTTP response body.

#### 5. Example
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;
    public UserController(UserService userService) {
        this.userService = userService;
    }
}

@Service
public class UserService {
    private final UserRepository userRepository;
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {}
```

#### 6. Common Mistakes
- Using `@Service` on a repository class — it works but breaks semantic clarity and you lose exception translation.
- Forgetting `@Repository` on a custom DAO class and missing the exception translation benefit.
- Confusing `@Controller` and `@RestController` — returning a String from `@Controller` without `@ResponseBody` tries to resolve a view template, not return JSON.

#### 7. Interview Tip
> *"All four — @Service, @Repository, @Controller, and @RestController — are specializations of @Component. They all get detected by component scan, but each adds meaning or behaviour. The key one to highlight is @Repository: it activates Spring's exception translation, converting JPA or JDBC exceptions into Spring's unified DataAccessException hierarchy, which makes your service layer cleaner and database-agnostic."*

---

### `@Autowired`

#### 1. Definition (Interview-ready)
Tells Spring to automatically inject a dependency — a bean from the application context — into a field, constructor, or setter.

#### 2. Why it is used
- **Problem:** In traditional code you'd write `UserService service = new UserService(new UserRepository())` everywhere. This creates tight coupling and makes testing extremely hard.
- **Why not manual wiring:** Dependency injection lets you swap implementations (especially for testing with mocks), and Spring manages the lifecycle.

#### 3. Where it is used
- Any Spring-managed bean: controllers, services, repositories, config classes.
- Three injection types: **constructor (preferred)**, field, setter.

#### 4. How it works internally
- Spring's `AutowiredAnnotationBeanPostProcessor` processes beans after they are created.
- It finds fields/constructors/setters marked with `@Autowired` and looks in the `ApplicationContext` for a matching bean by type.
- If multiple beans of the same type exist, it uses the field name or `@Qualifier` to disambiguate.
- Constructor injection is handled at instantiation time; field injection uses reflection to set the field directly.

#### 5. Example
```java
// Preferred: constructor injection
// (Spring 4.3+ — @Autowired optional if single constructor)
@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}

// Field injection (works but avoid in production code)
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService;
}
```

#### 6. Common Mistakes
- Using **field injection everywhere** — it makes the class impossible to unit test without a Spring context.
- Not handling `NoUniqueBeanDefinitionException` — when two beans of the same type exist, add `@Qualifier("beanName")`.
- **Circular dependency:** A depends on B, B depends on A — Spring throws an error. Fix by restructuring or using `@Lazy`.
- `@Autowired` on a `static` field — it doesn't work; Spring doesn't inject into static fields.

#### 7. Interview Tip
> *"I always prefer constructor injection over field injection. With constructor injection, dependencies are explicit, the class is easily testable with plain JUnit (just call new MyService(mockDep)), and the compiler enforces that required dependencies are provided. Field injection hides dependencies and requires reflection or a Spring context to test. Since Spring 4.3, if there's a single constructor, @Autowired is optional."*

---

### `@Configuration` and `@Bean`

#### 1. Definition (Interview-ready)
- `@Configuration` marks a class as a source of bean definitions, equivalent to a Spring XML config file.
- `@Bean` marks a method inside a `@Configuration` class whose return value is registered as a Spring bean.

#### 2. Why it is used
- **Problem:** Some beans can't be auto-detected with `@Component` — e.g., third-party classes like `RestTemplate`, `ObjectMapper`, or `DataSource` that you don't own.
- **Why not XML:** Java-based config is type-safe, refactorable, and IDE-friendly.

#### 3. Where it is used
- Config layer — infrastructure beans, third-party integrations, custom bean definitions.
- Common examples: `DataSource`, `RestTemplate`, security config, cache config.

#### 4. How it works internally
- Spring uses **CGLIB to subclass** every `@Configuration` class. This is critical: method calls between `@Bean` methods return the cached singleton from the context, not a new instance.
- Without `@Configuration` (using just `@Component`), calling another `@Bean` method creates a new instance each time — this is called **lite mode** vs **full mode**.

#### 5. Example
```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return mapper;
    }

    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

#### 6. Common Mistakes
- Calling a `@Bean` method directly from another `@Bean` method in a `@Component` class (not `@Configuration`) — you'll get a new instance instead of the singleton.
- Forgetting `@Configuration` and wondering why singleton behaviour is broken.
- Overriding beans accidentally — Spring Boot 2.1+ disallows duplicate bean names by default.

#### 7. Interview Tip
> *"The key internal detail examiners love: @Configuration classes are CGLIB-proxied. This means that if beanA() calls beanB() inside the config class, Spring intercepts that call and returns the existing beanB singleton from the context — it doesn't create a new one. This is the difference between 'full' @Configuration mode and 'lite' mode (using @Component). I always use @Configuration for proper singleton semantics."*

---

## Part 2 — Web Annotations

---

### `@RequestMapping`, `@GetMapping`, `@PostMapping`

#### 1. Definition (Interview-ready)
- `@RequestMapping` maps HTTP requests to handler methods or classes. Supports all HTTP methods, URL patterns, headers, and media types.
- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping` are composed shortcuts.

#### 2. Why it is used
Maps incoming HTTP requests to the right Java method — the core of Spring MVC routing.

#### 3. Where it is used
- Controller layer.
- `@RequestMapping` at class level defines the base path; method-level annotations refine it.

#### 4. How it works internally
- `DispatcherServlet` receives every HTTP request.
- `RequestMappingHandlerMapping` scans all `@Controller` beans at startup and builds a routing table.
- On each request, it matches URL, HTTP method, headers, and `produces`/`consumes` media types.
- The matched method is invoked by `RequestMappingHandlerAdapter`.

#### 5. Example
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping                          // GET /api/products
    public List<Product> getAllProducts() { ... }

    @GetMapping("/{id}")                 // GET /api/products/5
    public Product getProduct(@PathVariable Long id) { ... }

    @PostMapping                         // POST /api/products
    public Product createProduct(@RequestBody Product product) { ... }

    @PutMapping("/{id}")                 // PUT /api/products/5
    public Product updateProduct(@PathVariable Long id,
                                  @RequestBody Product product) { ... }

    @DeleteMapping("/{id}")              // DELETE /api/products/5
    public void deleteProduct(@PathVariable Long id) { ... }
}
```

#### 6. Common Mistakes
- **Ambiguous mappings** — two methods matching the same URL causes a startup exception.
- Forgetting `produces = "application/json"` when content negotiation is needed.
- Using `@RequestMapping` without `method` at method level — it matches ALL HTTP methods (security risk).

#### 7. Interview Tip
> *"@GetMapping is just a composed annotation — a shortcut for @RequestMapping(method=RequestMethod.GET). The important internal piece is DispatcherServlet — it's the front controller that receives all requests, delegates to RequestMappingHandlerMapping to find the handler, and then to RequestMappingHandlerAdapter to invoke it. Understanding this pipeline helps you debug 405 Method Not Allowed or 404s."*

---

### `@PathVariable`, `@RequestParam`, `@RequestBody`

#### 1. Definition (Interview-ready)
- `@PathVariable` — extracts a value from the URI path (e.g., `/users/{id}`).
- `@RequestParam` — extracts a query parameter from the URL (e.g., `?page=1&size=10`).
- `@RequestBody` — deserializes the HTTP request body (JSON/XML) into a Java object.

#### 2. Why it is used
These handle the three main ways a client sends data: in the path, in query strings, or in the request body.

#### 3. How it works internally
- `HandlerMethodArgumentResolver` implementations handle each annotation.
- `@PathVariable` → `PathVariableMethodArgumentResolver` reads URI template variables.
- `@RequestParam` → `RequestParamMethodArgumentResolver` reads from `HttpServletRequest.getParameter()`.
- `@RequestBody` → `RequestResponseBodyMethodProcessor` uses Jackson's `HttpMessageConverter` to deserialize JSON.

#### 4. Example
```java
// GET /api/orders/42?status=PENDING
@GetMapping("/api/orders/{id}")
public Order getOrder(
    @PathVariable Long id,                              // from /42
    @RequestParam(defaultValue = "ALL") String status   // from ?status=PENDING
) { ... }

// POST /api/orders   body: {"item":"laptop","qty":2}
@PostMapping("/api/orders")
public Order createOrder(@RequestBody @Valid OrderRequest request) { ... }
```

#### 5. Common Mistakes
- Using `@RequestParam` to read a POST JSON body — use `@RequestBody` instead.
- Forgetting `required = false` on optional `@RequestParam` — Spring throws `MissingServletRequestParameterException`.
- Mismatched variable name: `@PathVariable("userId") Long id` when URI template uses `{userId}`.
- Not adding `@Valid` alongside `@RequestBody` — Bean Validation won't trigger without it.

#### 6. Interview Tip
> *"The interview classic: @PathVariable is for RESTful path segments like /users/42, @RequestParam is for query parameters like ?sort=name. For the request body, @RequestBody with Jackson does the deserialization. A follow-up they love: @RequestBody reads the request body once — you can't read it twice, so you can't combine @RequestBody with reading raw request streams."*

---

## Part 3 — Exception Handling

---

### `@ExceptionHandler`

#### 1. Definition (Interview-ready)
Marks a method that handles exceptions thrown by handler methods in the same controller (or globally with `@ControllerAdvice`).

#### 2. Why it is used
- **Problem:** Without it, uncaught exceptions result in Spring's default error page or a generic 500 response.
- **Why not try-catch everywhere:** Cross-cutting exception handling in every method creates massive duplication.

#### 3. Where it is used
- Inside a `@Controller` / `@RestController` for controller-local handling.
- Inside a `@ControllerAdvice` class for global handling.

#### 4. How it works internally
- `ExceptionHandlerExceptionResolver` intercepts exceptions during request processing.
- It scans the controller (and `@ControllerAdvice` beans) for methods annotated with `@ExceptionHandler`.
- Matches by exception type and invokes the handler method.

#### 5. Example
```java
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User " + id + " not found"));
    }

    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse("USER_NOT_FOUND", ex.getMessage());
    }
}
```

#### 6. Common Mistakes
- Defining `@ExceptionHandler` inside a controller — it only catches exceptions from **that controller**. Use `@ControllerAdvice` for global scope.
- Handling `Exception` (root) locally — accidentally swallows things you didn't intend.
- Forgetting `@ResponseStatus` — handler returns 200 OK even for errors.

---

### `@ControllerAdvice`

#### 1. Definition (Interview-ready)
A specialization of `@Component` that allows you to define global exception handlers applied across all controllers.

#### 2. Why it is used
Centralizes cross-cutting concerns for the entire web layer. Without it, every controller needs its own `@ExceptionHandler`.

#### 3. How it works internally
- Spring detects `@ControllerAdvice` beans during startup and registers them with `ExceptionHandlerExceptionResolver`.
- When an exception is thrown, the resolver checks the originating controller first, then falls back to `@ControllerAdvice` beans.
- Can be scoped with `basePackages`, `assignableTypes`, or `annotations` attributes.

#### 4. Example
```java
@RestControllerAdvice  // = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return new ErrorResponse("VALIDATION_FAILED", message);
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneric(Exception ex) {
        return new ErrorResponse("INTERNAL_ERROR", "Something went wrong");
    }
}
```

#### 5. Common Mistakes
- Using `@ControllerAdvice` instead of `@RestControllerAdvice` for REST APIs — handler objects won't be serialized to JSON without `@ResponseBody`.
- Generic `Exception.class` handler catching everything unintentionally.
- Not handling `ConstraintViolationException` (from `@Validated` on services) separately from `MethodArgumentNotValidException` (from `@RequestBody @Valid`).

#### 6. Interview Tip
> *"I always create a single @RestControllerAdvice class as the centralized error contract for the API. The key distinction: @ExceptionHandler inside a controller is local. @ControllerAdvice is global. In real projects, I also use a standard ErrorResponse POJO — error code, message, timestamp — so clients always get predictable JSON regardless of the exception."*

---

## Part 4 — JPA Annotations

---

### `@Entity`, `@Table`, `@Id`, `@GeneratedValue`

#### 1. Definition (Interview-ready)
- `@Entity` — marks a class as a JPA entity mapped to a database table.
- `@Table` — customizes the table name, schema, or unique constraints (optional).
- `@Id` — marks the primary key field.
- `@GeneratedValue` — configures how the primary key is auto-generated.

#### 2. Generation Strategies

| Strategy | Description |
|---|---|
| `IDENTITY` | Database auto-increment (MySQL, PostgreSQL serial) |
| `SEQUENCE` | Uses a database sequence — preferred for PostgreSQL |
| `UUID` (JPA 3.1+) | Generates a UUID |
| `AUTO` | Hibernate picks a strategy — avoid, can cause issues |

#### 3. Example
```java
@Entity
@Table(name = "users",
       uniqueConstraints = @UniqueConstraint(columnNames = "email"))
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "email", nullable = false, unique = true)
    private String email;

    @Column(name = "full_name", length = 100)
    private String fullName;

    // Getters, setters, no-arg constructor required by JPA
}
```

#### 4. Common Mistakes
- Forgetting the **no-argument constructor** — JPA requires it for instantiation (can be `protected`).
- Using `GenerationType.AUTO` in PostgreSQL — shares a single `hibernate_sequence` across all entities, causing ID conflicts.
- Missing `@Column` on fields with reserved SQL names (e.g., `order`).
- Using primitive `long` instead of `Long` for `@Id` — wrapper type works better with proxies.

---

### `@OneToMany` and `@ManyToOne`

#### 1. Definition (Interview-ready)
- `@ManyToOne` — on the **many** side. Many orders belong to one customer. Creates a foreign key column.
- `@OneToMany` — on the **one** side. One customer has many orders. The inverse (non-owning) side.

#### 2. How it works internally
- `@ManyToOne` creates a foreign key column (e.g., `customer_id`) in the owning entity's table.
- `@OneToMany(mappedBy = "customer")` is the non-owning side — no column created; just navigation.
- `FetchType.LAZY` (default for `@OneToMany`) — loads on access.
- `FetchType.EAGER` (default for `@ManyToOne`) — loads immediately with the parent.

#### 3. Example
```java
@Entity
public class Customer {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```

#### 4. Common Mistakes
- **N+1 query problem** — loading 100 customers and accessing `.getOrders()` fires 101 queries. Fix with `@EntityGraph` or `JOIN FETCH`.
- Using `FetchType.EAGER` on `@OneToMany` — massive data over-fetching on every query.
- Forgetting `mappedBy` on the `@OneToMany` side — JPA creates a join table instead of a foreign key.
- **Infinite JSON recursion** — bidirectional entities serialize each other. Fix with `@JsonManagedReference` / `@JsonBackReference` or `@JsonIgnore`.

#### 5. Interview Tip
> *"The N+1 problem is the most important thing to understand with @OneToMany. If I load 100 customers and iterate their orders, Hibernate fires 100 extra SELECT queries. The solution in Spring Data JPA is @EntityGraph on the repository method, or JOIN FETCH in JPQL. I also always use FetchType.LAZY on @OneToMany — EAGER is almost never the right default for collections."*

---

### `@Transactional`

#### 1. Definition (Interview-ready)
Manages database transactions declaratively. Spring automatically begins a transaction before the annotated method and commits (or rolls back on exception) when the method completes.

#### 2. Why it is used
- **Problem:** Manually writing `beginTransaction()`, `commit()`, `rollback()` in every service method is verbose, error-prone, and duplicated.
- **Why not try-catch-rollback manually:** Declarative transactions are cleaner and handle nested transactions and propagation automatically.

#### 3. Where it is used
- **Service layer** (strongly preferred). Avoid at repository or controller level.

#### 4. How it works internally
- Spring creates a JDK dynamic proxy (or CGLIB proxy) around the bean.
- `TransactionInterceptor` opens a transaction via `PlatformTransactionManager`.
- On successful return: `commit()`. On `RuntimeException`: `rollback()`. On **checked exception: commit by default** (unless `rollbackFor` is specified).
- Default propagation: `REQUIRED` — reuse existing transaction or create a new one.

#### 5. Propagation Levels

| Propagation | Behaviour |
|---|---|
| `REQUIRED` (default) | Join existing or create new |
| `REQUIRES_NEW` | Always create new; suspend existing |
| `NESTED` | Create savepoint within existing |
| `SUPPORTS` | Use existing if present; no-tx if not |
| `NEVER` | Throw exception if transaction exists |

#### 6. Example
```java
@Service
public class TransferService {

    @Transactional
    public void transferMoney(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepo.findById(fromId).orElseThrow();
        Account to = accountRepo.findById(toId).orElseThrow();
        from.debit(amount);
        to.credit(amount);
        accountRepo.save(from);
        accountRepo.save(to);
        // Any exception → full rollback
    }

    @Transactional(readOnly = true) // Optimization for read queries
    public Account getAccount(Long id) {
        return accountRepo.findById(id).orElseThrow();
    }
}
```

#### 7. Common Mistakes
- **Self-invocation problem** — calling a `@Transactional` method from another method in the same class bypasses the proxy; no transaction is started.
- Annotating `private` or `final` methods — CGLIB can't proxy these.
- Not knowing that **checked exceptions do NOT trigger rollback** by default. Use `@Transactional(rollbackFor = Exception.class)`.
- Overusing `@Transactional` at the controller layer — keeps the DB connection open for the entire request.

#### 8. Interview Tip
> *"@Transactional works through AOP proxy. The gotcha I always mention: calling a @Transactional method from within the same class bypasses the proxy — no transaction. Also: checked exceptions don't trigger rollback by default, only RuntimeExceptions do. And readOnly=true is an optimization hint to the database driver and Hibernate — it avoids dirty checking and can use read replicas."*

---

## Part 5 — Advanced Annotations

---

### `@Profile`

#### 1. Definition (Interview-ready)
Marks a bean or configuration class as active only for a specific named profile (e.g., `dev`, `prod`, `test`).

#### 2. Why it is used
Different environments need different configurations — in-memory H2 for tests, real PostgreSQL for production.

#### 3. How it works internally
- Spring checks the active profiles (`spring.profiles.active` property or environment variable).
- Uses `ProfileCondition` internally to determine if the bean should be registered.

#### 4. Example
```java
@Configuration
@Profile("dev")
public class DevDataSourceConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder().setType(H2).build();
    }
}

@Configuration
@Profile("prod")
public class ProdDataSourceConfig {
    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://prod-db:5432/myapp")
            .build();
    }
}
```

```properties
# application.properties
spring.profiles.active=dev
```

#### 5. Common Mistakes
- Not setting `spring.profiles.active` — no beans loaded.
- Forgetting `@Profile("!prod")` means "any profile except prod".

---

### `@Conditional`

#### 1. Definition (Interview-ready)
Registers a bean only when a specific condition is true at application startup. Foundation of Spring Boot's auto-configuration.

#### 2. Common Built-in Conditions

| Annotation | Condition |
|---|---|
| `@ConditionalOnProperty` | Property exists with a specified value |
| `@ConditionalOnClass` | A class is present on the classpath |
| `@ConditionalOnMissingBean` | No bean of that type exists yet |
| `@ConditionalOnExpression` | SpEL expression is true |
| `@ConditionalOnWebApplication` | Running as a web application |

#### 3. Example
```java
@Configuration
public class NotificationConfig {

    @Bean
    @ConditionalOnProperty(name = "feature.email.enabled", havingValue = "true")
    public EmailNotificationService emailService() {
        return new EmailNotificationService();
    }

    @Bean
    @ConditionalOnMissingBean(NotificationService.class)
    public SmsNotificationService smsService() {
        return new SmsNotificationService();
    }
}
```

#### 4. Interview Tip
> *"@Conditional is the engine behind all of Spring Boot's auto-configuration. JpaAutoConfiguration is guarded by @ConditionalOnClass(JpaRepository.class) and @ConditionalOnMissingBean(EntityManagerFactory.class). So if I define my own EntityManagerFactory, Spring Boot's auto-config backs off. This is the 'opinionated defaults with easy overrides' philosophy."*

---

### `@Async`

#### 1. Definition (Interview-ready)
Marks a method to be executed in a separate thread pool, asynchronously. The caller doesn't wait for the method to complete.

#### 2. Why it is used
For non-blocking operations: sending emails, pushing notifications, processing files — things that shouldn't block the HTTP response.

#### 3. How it works internally
- Requires `@EnableAsync` on a configuration class.
- Spring creates an AOP proxy; method calls are submitted to an `Executor` (thread pool) and return immediately.
- Return type can be `void`, `Future<T>`, or `CompletableFuture<T>`.

#### 4. Example
```java
@SpringBootApplication
@EnableAsync
public class MyApplication { ... }

@Service
public class EmailService {

    @Async
    public CompletableFuture<Boolean> sendWelcomeEmail(String to) {
        emailSender.send(to, "Welcome!");
        return CompletableFuture.completedFuture(true);
    }
}

// Caller returns immediately — email sent in background thread
emailService.sendWelcomeEmail(user.getEmail());
```

#### 5. Common Mistakes
- **Self-invocation** — `this.sendEmail()` won't be async.
- No `@EnableAsync` — annotation silently ignored.
- Using the default `SimpleAsyncTaskExecutor` in production — creates a new thread per call. Always configure a `ThreadPoolTaskExecutor`.
- Not handling exceptions from `CompletableFuture` — async exceptions are swallowed silently.

---

### `@Scheduled`

#### 1. Definition (Interview-ready)
Marks a method to be executed at a fixed rate, fixed delay, or on a cron schedule — automatically by Spring's scheduler.

#### 2. Why it is used
For recurring background tasks: cleanup jobs, data sync, report generation, cache warming.

#### 3. How it works internally
- Requires `@EnableScheduling`.
- Spring's `ScheduledAnnotationBeanPostProcessor` registers the method with a `TaskScheduler`.
- Default: single-threaded executor.

#### 4. Example
```java
@EnableScheduling
@SpringBootApplication
public class MyApplication { ... }

@Component
public class CleanupJob {

    @Scheduled(cron = "0 0 2 * * *")           // 2 AM every day
    public void cleanupExpiredSessions() { ... }

    @Scheduled(fixedRate = 60_000)               // every 60 seconds
    public void refreshCache() { ... }

    @Scheduled(fixedDelay = 30_000,              // 30s after last run completes
               initialDelay = 10_000)            // start 10s after app starts
    public void pollExternalApi() { ... }
}
```

#### 5. Cron Expression Format
```
┌──────── second (0-59)
│ ┌───────── minute (0-59)
│ │ ┌────────── hour (0-23)
│ │ │ ┌─────────── day of month (1-31)
│ │ │ │ ┌──────────── month (1-12)
│ │ │ │ │ ┌───────────── day of week (0-7, Sun=0 or 7)
│ │ │ │ │ │
0 0 2 * * *    → 2:00 AM every day
0 */15 * * * * → every 15 minutes
0 9 * * MON    → every Monday at 9 AM
```

#### 6. Common Mistakes
- Default scheduler is **single-threaded** — long tasks delay subsequent runs.
- Method must be `void` and take **no arguments**.
- No `@EnableScheduling` → tasks never run, silently.
- Running in all instances of a clustered app — use `ShedLock` or `@ConditionalOnProperty` for one-node execution.

---

### `@Cacheable`

#### 1. Definition (Interview-ready)
Caches the return value of a method. On subsequent calls with the same arguments, the cached value is returned without executing the method body.

#### 2. Why it is used
Avoids expensive repeated operations: database queries, external API calls, heavy computations.

#### 3. How it works internally
- Requires `@EnableCaching`.
- Spring wraps the bean in a proxy; on method call, constructs a cache key from arguments.
- **Cache hit** → return cached value. **Cache miss** → execute method, store result, return.

#### 4. The Full Cache Lifecycle

| Annotation | Behaviour |
|---|---|
| `@Cacheable` | Read — execute only on cache miss |
| `@CachePut` | Always execute and update the cache |
| `@CacheEvict` | Remove entry from the cache |

#### 5. Example
```java
@EnableCaching
@SpringBootApplication
public class MyApplication { ... }

@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) {
        return productRepository.findById(id).orElseThrow();
    }

    @CachePut(value = "products", key = "#product.id")
    public Product update(Product product) {
        return productRepository.save(product);
    }

    @CacheEvict(value = "products", key = "#id")
    public void delete(Long id) {
        productRepository.deleteById(id);
    }
}
```

```properties
# Use Redis as cache backend
spring.cache.type=redis
```

#### 6. Common Mistakes
- **Self-invocation** — same proxy bypass as `@Transactional` and `@Async`.
- Caching **mutable objects** — the cached object is shared; mutating it corrupts the cache.
- Using **default in-memory cache** in production — data lost on restart and not shared across instances. Use Redis.
- **Cache key collisions** — multiple methods on the same cache name without specifying `key`.
- No **TTL (time-to-live)** configured — stale data served indefinitely.

#### 7. Interview Tip
> *"I always pair @Cacheable with @CachePut and @CacheEvict — that's the complete cache lifecycle. In production I configure Redis as the cache provider so it's shared across app instances. A common pitfall: if your method throws an exception, the result is NOT cached — which is actually the desired behavior. And like @Transactional and @Async, self-invocation bypasses the proxy and the cache entirely."*

---

## Part 6 — Cheatsheet by Layer

### Config / Bootstrap Layer
| Annotation | Purpose |
|---|---|
| `@SpringBootApplication` | Main class — combines @Configuration + @EnableAutoConfiguration + @ComponentScan |
| `@Configuration` | Java-based config class. CGLIB-proxied for singleton semantics |
| `@Bean` | Register method's return value as a Spring bean |
| `@Profile` | Activate beans only for specific profiles (dev/prod/test) |
| `@Conditional` | Register bean only when a condition is met |
| `@ConditionalOnProperty` | Condition: property has a specific value |
| `@ConditionalOnClass` | Condition: class is on the classpath |
| `@ConditionalOnMissingBean` | Condition: no bean of that type exists yet |

### Controller Layer
| Annotation | Purpose |
|---|---|
| `@RestController` | @Controller + @ResponseBody — returns data, not view names |
| `@Controller` | MVC controller — returns view names |
| `@RequestMapping` | Base URL mapping for class or method |
| `@GetMapping` | Maps HTTP GET requests |
| `@PostMapping` | Maps HTTP POST requests |
| `@PutMapping` | Maps HTTP PUT requests |
| `@DeleteMapping` | Maps HTTP DELETE requests |
| `@PatchMapping` | Maps HTTP PATCH requests |
| `@PathVariable` | Extracts value from URI path `/users/{id}` |
| `@RequestParam` | Extracts query parameter `?page=1` |
| `@RequestBody` | Deserializes JSON body to Java object |
| `@ResponseStatus` | Sets HTTP status code on response |
| `@ExceptionHandler` | Handles specific exceptions in controller scope |
| `@ControllerAdvice` | Global exception handler across all controllers |
| `@RestControllerAdvice` | @ControllerAdvice + @ResponseBody |

### Service Layer
| Annotation | Purpose |
|---|---|
| `@Service` | Semantic marker for business logic layer |
| `@Autowired` | Injects a Spring bean (prefer constructor injection) |
| `@Qualifier` | Specifies which bean to inject when multiple exist |
| `@Primary` | Marks a bean as the default when multiple exist |
| `@Transactional` | Declarative transaction management |
| `@Async` | Runs method in background thread pool |
| `@Scheduled` | Cron / fixed-rate / fixed-delay recurring task |
| `@Cacheable` | Cache method return value on cache miss |
| `@CachePut` | Always execute and update cache |
| `@CacheEvict` | Remove entry from cache |

### Repository / JPA Layer
| Annotation | Purpose |
|---|---|
| `@Repository` | Marks DAO — translates DB exceptions to DataAccessException |
| `@Entity` | Maps class to a database table |
| `@Table` | Customizes table name and constraints |
| `@Id` | Marks the primary key field |
| `@GeneratedValue` | Auto-generation strategy for primary key |
| `@Column` | Customizes column name, length, nullable |
| `@OneToMany` | One-to-many relationship (inverse/non-owning side) |
| `@ManyToOne` | Many-to-one relationship (owning side — holds FK) |
| `@JoinColumn` | Customizes the foreign key column name |
| `@ManyToMany` | Many-to-many relationship |
| `@OneToOne` | One-to-one relationship |

### Cross-cutting / Utility
| Annotation | Purpose |
|---|---|
| `@Component` | Generic Spring-managed bean |
| `@Value` | Injects property value from application.properties |
| `@Profile` | Activates bean for specific environment profile |
| `@Lazy` | Delays bean initialization until first use |
| `@Scope` | Sets bean scope (singleton, prototype, request, session) |
| `@EnableAsync` | Enables @Async processing |
| `@EnableScheduling` | Enables @Scheduled task execution |
| `@EnableCaching` | Enables @Cacheable caching |
| `@EnableTransactionManagement` | Enables @Transactional (auto-enabled by Spring Boot) |

---

## Part 7 — 10 Common Interview Questions with Answers

---

**Q1. What is the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?**

All four are stereotypes discovered by `@ComponentScan` and registered as Spring beans. Functional differences:
- `@Repository` adds **exception translation** — converts DB-specific exceptions (like `HibernateJdbcException`) into `DataAccessException` subclasses.
- `@Controller` integrates with Spring MVC and participates in view resolution.
- `@Service` and `@Component` have no extra runtime behaviour — they're semantic markers.

---

**Q2. Why prefer constructor injection over field injection with `@Autowired`?**

Three reasons:
1. **Testability** — instantiate the class with `new MyService(mockDep)` without a Spring context.
2. **Immutability** — mark the field `final`, preventing accidental reassignment.
3. **Fail-fast** — if a required dependency is missing, the application fails to start at construction time, not at runtime.

Field injection hides dependencies, uses reflection, and makes the class harder to test and reason about.

---

**Q3. Explain the `@Transactional` proxy and the self-invocation problem.**

`@Transactional` works via an AOP proxy. When external code calls your method through the Spring bean reference, the proxy intercepts the call, starts a transaction, invokes the real method, and commits or rolls back. But if you call a `@Transactional` method from within the same class (`this.someMethod()`), you bypass the proxy entirely — no transaction is started. Fix: inject a self-reference via `@Autowired` or restructure into a separate service.

---

**Q4. What happens if you have two beans of the same type and use `@Autowired`?**

Spring throws `NoUniqueBeanDefinitionException`. Resolutions:
- `@Qualifier("beanName")` — specify which bean to inject.
- `@Primary` — make one bean the default.
- Inject `List<MyService>` — get all implementations (useful for strategy patterns).

---

**Q5. What is the difference between `@Controller` and `@RestController`?**

`@RestController` is `@Controller` + `@ResponseBody`. With `@Controller`, returning a `String` means Spring tries to resolve a view template (Thymeleaf, etc.). With `@RestController`, the return value is serialized to the HTTP response body using `HttpMessageConverter` (Jackson for JSON). For REST APIs, always use `@RestController`.

---

**Q6. What is the N+1 problem and how do you solve it in Spring Data JPA?**

Loading 100 `Customer` entities and accessing `customer.getOrders()` for each fires 1 + 100 = 101 queries total. Solutions:
1. `@EntityGraph(attributePaths = "orders")` on the repository method — generates a single JOIN query.
2. JPQL with `JOIN FETCH`: `SELECT c FROM Customer c JOIN FETCH c.orders`.
3. Hibernate's `@BatchSize` for batch loading.

---

**Q7. How does `@SpringBootApplication`'s auto-configuration work?**

`@EnableAutoConfiguration` reads `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` from every JAR on the classpath. Each entry is an auto-configuration class guarded by `@Conditional` annotations. For example, `DataSourceAutoConfiguration` is active only if a JDBC driver is on the classpath AND no `DataSource` bean is already defined. Run with `--debug` flag to see the auto-configuration report.

---

**Q8. What is the difference between `@Async` and `@Scheduled`?**

- `@Async` is **triggered by a caller** — runs a specific method call in a background thread pool. The calling thread returns immediately. Used for one-off async tasks.
- `@Scheduled` is **timer-driven** — Spring's scheduler automatically invokes the method at configured intervals (fixed rate, fixed delay, or cron). No external caller needed. Used for recurring background jobs.

Both require their respective `@Enable*` annotations and both suffer from the self-invocation proxy limitation.

---

**Q9. What is `@ControllerAdvice` and when would you use it over `@ExceptionHandler` in the controller?**

`@ExceptionHandler` inside a `@RestController` only handles exceptions from that specific controller. `@ControllerAdvice` (or `@RestControllerAdvice`) handles exceptions from any controller in the application. In real projects, always use a single `@RestControllerAdvice` class — this gives your API a consistent error response format and eliminates duplication.

---

**Q10. What is the difference between `@Bean` and `@Component`?**

- `@Component` is placed on **your own class** and detected automatically by classpath scanning.
- `@Bean` is placed on a **method** inside a `@Configuration` class — the method creates and returns a bean instance. Used for third-party classes you don't own (like `RestTemplate`, `ObjectMapper`) or beans that need programmatic configuration.

**Rule of thumb: if you own the class → `@Component`. If you don't → `@Bean`.**

---

## Additional Tips

### Bean Scopes
```java
@Scope("singleton")   // default — one instance per Spring context
@Scope("prototype")   // new instance on every injection/getBean call
@Scope("request")     // one instance per HTTP request (web only)
@Scope("session")     // one instance per HTTP session (web only)
```

### `@Value` — Property Injection
```java
@Value("${app.timeout:5000}")  // inject property with default value 5000
private int timeout;

@Value("${app.name}")           // inject required property (fails if missing)
private String appName;
```

### `@Primary` vs `@Qualifier`
- `@Primary` — sets a default bean when multiple of the same type exist.
- `@Qualifier` — explicitly names which bean to inject. **Wins over `@Primary`.**

### `@Lazy` — Delayed Initialization
```java
@Lazy
@Service
public class HeavyService { ... }  // initialized only on first use
```
Useful for breaking circular dependencies or speeding up startup.

### `@Valid` vs `@Validated`
- `@Valid` — triggers Bean Validation on `@RequestBody` in controllers (JSR-303/380 standard).
- `@Validated` — Spring variant; additionally supports **validation groups** and works on method parameters in services (not just controllers).

### Excluding Auto-configurations
```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
// Common when writing tests without a real database
```

### The AOP Proxy — Common Gotcha for 3 Annotations

`@Transactional`, `@Async`, and `@Cacheable` all use the **same AOP proxy mechanism**.  
All three share the same failure mode — self-invocation:

```java
@Service
public class MyService {

    @Transactional
    public void methodA() {
        methodB();  // ❌ NO transaction! Bypasses proxy (self-invocation)
    }

    @Transactional
    public void methodB() { ... }
}
```

**Fix:** Always call through the Spring bean reference, not `this`.

```java
@Service
public class MyService {

    @Autowired
    private MyService self;  // inject self-reference

    public void methodA() {
        self.methodB();  // ✅ Goes through the proxy
    }

    @Transactional
    public void methodB() { ... }
}
```

Or better: extract `methodB()` into a separate `@Service` class.

---

*Spring Boot 3.x / Java 17+ | For interview preparation*
