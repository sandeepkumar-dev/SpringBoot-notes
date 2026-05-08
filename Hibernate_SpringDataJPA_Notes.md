# 🗄️ Hibernate & Spring Data JPA — Complete Notes

---

## 📌 Table of Contents

1. [What is Hibernate?](#1-what-is-hibernate)
2. [JPA vs Hibernate](#2-jpa-vs-hibernate)
3. [Spring Data JPA](#3-spring-data-jpa)
4. [Repository Layer](#4-repository-layer)
5. [Hibernate Annotations Reference](#5-hibernate-annotations-reference)
6. [Normalization](#6-normalization)
7. [Dependency Injection (DI)](#7-dependency-injection-di)
8. [Spring IoC Container](#8-spring-ioc-container)
9. [Bean Lifecycle](#9-bean-lifecycle)
10. [CRUD Operations with Hibernate](#10-crud-operations-with-hibernate)
11. [Finder Methods](#11-finder-methods)
12. [JPQL — Java Persistence Query Language](#12-jpql--java-persistence-query-language)
13. [Native SQL Queries](#13-native-sql-queries)
14. [Hibernate Mappings](#14-hibernate-mappings)
    - [OneToMany — Unidirectional](#onetomany--unidirectional)
    - [OneToMany — Bidirectional](#onetomany--bidirectional)
    - [ManyToMany Mapping](#manytomany-mapping)
    - [OneToOne — Bidirectional](#onetoone--bidirectional)
15. [FetchType — EAGER vs LAZY](#15-fetchtype--eager-vs-lazy)
16. [Default Fetch Types](#16-default-fetch-types)
17. [Hibernate Cache](#17-hibernate-cache)
18. [JUnit Testing in Spring Boot](#18-junit-testing-in-spring-boot)
19. [Optional Class](#19-optional-class)
20. [SessionFactory (Interview Question)](#20-sessionfactory-interview-question)

---

## 1. What is Hibernate?

> **Hibernate** is an **ORM (Object Relational Mapping)** tool that maps Java classes to database tables and automates CRUD operations — without writing complex SQL queries.

### Basic Entity Example

```java
@Entity
public class Employee {

    @Id
    private int id;
    private String name;
    private float salary;

    // Getters & Setters
    public void setId(int id) { this.id = id; }
    public void setName(String name) { this.name = name; }
    public void setSalary(float salary) { this.salary = salary; }

    public int getId() { return id; }
    public String getName() { return name; }
    public float getSalary() { return salary; }
}
```

| Annotation | Purpose |
|---|---|
| `@Entity` | Maps this Java class to a database table |
| `@Id` | Marks the field as the **primary key** (mandatory) |

---

## 2. JPA vs Hibernate

| | JPA | Hibernate |
|---|---|---|
| **Type** | Specification (interfaces only) | Implementation of JPA |
| **Role** | Defines *what* to do | Defines *how* to do it (+ extra features) |
| **Examples** | — | EclipseLink, OpenJPA, DataNucleus |

> 🔹 **JPA** = The rulebook  
> 🔹 **Hibernate** = The player that follows (and extends) the rules

---

## 3. Spring Data JPA

Spring Data JPA sits on top of JPA/Hibernate and **eliminates boilerplate code** for common DB operations.

- No need to manually write `save`, `findById`, `delete` logic
- By default uses **Hibernate** as the underlying ORM provider
- Enables finder methods, JPQL, and native SQL with minimal code

---

## 4. Repository Layer

The Repository layer exposes **utility methods** to interact with the database.

```java
public interface EmployeeRepository extends CrudRepository<Employee, Long> {
    // Inherits: save(), findById(), findAll(), delete(), count(), etc.
}
```

> 💡 `CrudRepository<Entity, IDType>` — extend this to get all basic CRUD operations for free.

---

## 5. Hibernate Annotations Reference

| Annotation | Description |
|---|---|
| `@Entity` | Marks class as a JPA entity (maps to a DB table) |
| `@Id` | Marks the primary key field |
| `@GeneratedValue` | Configures auto-generation of primary key (e.g., `IDENTITY` = auto-increment) |
| `@Table(name="...")` | Overrides the default table name |
| `@Column(...)` | Configures column properties: name, length, nullable, unique |
| `@OneToMany` | One record in this table maps to many in the other |
| `@ManyToOne` | Many records in this table map to one in the other |
| `@OneToOne` | One-to-one relationship between two tables |
| `@ManyToMany` | Many records on both sides relate to each other |
| `@JoinColumn` | Specifies the foreign key column |
| `@JoinTable` | Defines the join/bridge table for ManyToMany |
| `@Query` | Write custom JPQL or native SQL in repository |
| `@Transactional` | Manages DB transaction; auto-commits or rolls back |

---

## 6. Normalization

### 1NF — First Normal Form
- All values must be **atomic** (no lists/arrays in a cell)
- Each row must be **uniquely identifiable**
- No repeating column groups

### 2NF — Second Normal Form
- Must already satisfy 1NF
- Group columns by their **entity** and separate into different tables
- Eliminates **partial dependency**

### 3NF — Third Normal Form
- Must already satisfy 2NF
- Remove all **duplicate/redundant** data
- Each non-key column must depend only on the **primary key**
- Establish **foreign key** relationships between tables

---

## 7. Dependency Injection (DI)

> **DI** is a design pattern where the Spring framework **automatically creates and injects** the objects your class needs — you don't create them manually.

### ❌ Tightly Coupled (Bad)

```java
public class Notification {
    // Directly instantiating — tightly coupled to EmailService
    private EmailService emailService = new EmailService();

    public void send(String msg) {
        emailService.sendEmail(msg);
    }
}
```

### ✅ Loosely Coupled (Good — using DI)

```java
// Step 1: Interface
public interface MessageService {
    void sendMessage(String msg);
}

// Step 2: Implementations
public class EmailService implements MessageService {
    public void sendMessage(String msg) {
        System.out.println("Email sent: " + msg);
    }
}

public class WhatsAppService implements MessageService {
    public void sendMessage(String msg) {
        System.out.println("WhatsApp sent: " + msg);
    }
}

// Step 3: Inject via constructor
public class Notification {
    private MessageService messageService;

    public Notification(MessageService messageService) {
        this.messageService = messageService;
    }

    public void send(String msg) {
        messageService.sendMessage(msg);
    }
}
```

**Why use DI?**
- ✅ Loose coupling between components
- ✅ Automatic lifecycle management by Spring
- ✅ Easy to swap implementations (e.g., Email → WhatsApp)

---

## 8. Spring IoC Container

> **IoC (Inversion of Control)** Container is the core of Spring. It manages the **lifecycle and creation** of all Spring beans.

- Automatically creates, configures, and wires objects
- You declare what you need; Spring provides it
- Promotes loose coupling

---

## 9. Bean Lifecycle

```
Bean Created
    ↓
Dependencies Injected (@Autowired)
    ↓
Execution (Bean used by application)
    ↓
Destruction
```

---

## 10. CRUD Operations with Hibernate

### Step 1 — `application.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test_crud_db
spring.datasource.username=root
spring.datasource.password=test

# update = reuse existing table; create = drop & recreate on startup
spring.jpa.hibernate.ddl-auto=update
```

### Step 2 — Entity Class

```java
@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(name = "first_name", nullable = false, length = 45)
    private String firstName;

    @Column(name = "last_name", nullable = false, length = 45)
    private String lastName;

    @Column(name = "email_id", nullable = false, length = 256, unique = true)
    private String emailId;

    @Column(name = "mobile", nullable = false, unique = true)
    private String mobile;

    // Getters & Setters ...
}
```

### Step 3 — Repository

```java
public interface EmployeeRepository extends CrudRepository<Employee, Long> { }
```

### Step 4 — JUnit Tests

```java
@SpringBootTest
class DemocrudApplicationTests {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Test
    void saveRecord() {
        Employee emp = new Employee();
        emp.setFirstName("Adam");
        emp.setLastName("A");
        emp.setEmailId("adam@gmail.com");
        emp.setMobile("9632629455");
        employeeRepository.save(emp);
    }

    @Test
    void deleteRecord() {
        employeeRepository.deleteById(3L);
    }

    @Test
    void getRecordById() {
        Optional<Employee> opEmp = employeeRepository.findById(1L);
        opEmp.ifPresentOrElse(
            e -> System.out.println(e.getFirstName()),
            () -> System.out.println("No record found")
        );
    }

    @Test
    void getAllRecords() {
        employeeRepository.findAll().forEach(e -> {
            System.out.println(e.getId() + " - " + e.getFirstName());
        });
    }

    @Test
    void countRecords() {
        System.out.println("Total: " + employeeRepository.count());
    }
}
```

---

## 11. Finder Methods

> Finder methods let you query data by simply **declaring method names** in the repository interface. Spring Data JPA auto-generates the SQL.

### Repository with Finder Methods

```java
public interface EmployeeRepository extends CrudRepository<Employee, Long> {

    Optional<Employee> findByEmailId(String email);
    Optional<Employee> findByMobile(String mobile);
    Optional<Employee> findByEmailIdAndMobile(String email, String mobile);
    List<Employee>     findByEmailIdOrMobile(String email, String mobile);

    boolean existsByEmailId(String email);
    boolean existsByMobile(String mobile);

    long countByEmailId(String email);

    List<Employee> findByFirstNameContaining(String keyword);
    List<Employee> findByFirstNameStartingWith(String prefix);
    List<Employee> findByFirstNameEndingWith(String suffix);
}
```

### Method Naming Convention

| Keyword | Example | Generated SQL condition |
|---|---|---|
| `findBy` | `findByEmail` | `WHERE email = ?` |
| `And` | `findByEmailAndMobile` | `WHERE email = ? AND mobile = ?` |
| `Or` | `findByEmailOrMobile` | `WHERE email = ? OR mobile = ?` |
| `Containing` | `findByNameContaining` | `WHERE name LIKE '%?%'` |
| `StartingWith` | `findByNameStartingWith` | `WHERE name LIKE '?%'` |
| `EndingWith` | `findByNameEndingWith` | `WHERE name LIKE '%?'` |
| `existsBy` | `existsByEmail` | `EXISTS (WHERE email = ?)` |
| `countBy` | `countByEmail` | `COUNT WHERE email = ?` |

---

## 12. JPQL — Java Persistence Query Language

> JPQL is like SQL, but it uses **entity names and field names** instead of table/column names.

```java
public interface EmployeeRepository extends CrudRepository<Employee, Long> {

    // Single parameter
    @Query("SELECT e FROM Employee e WHERE e.emailId = :x")
    Optional<Employee> searchByEmail(@Param("x") String email);

    @Query("SELECT e FROM Employee e WHERE e.mobile = :m")
    Optional<Employee> searchByMobile(@Param("m") String mobile);

    // Multiple parameters
    @Query("SELECT e FROM Employee e WHERE e.emailId = :x AND e.mobile = :m")
    Optional<Employee> searchByEmailAndMobile(@Param("x") String email, @Param("m") String mobile);

    @Query("SELECT e FROM Employee e WHERE e.emailId = :x OR e.mobile = :m")
    List<Employee> searchByEmailOrMobile(@Param("x") String email, @Param("m") String mobile);
}
```

> ⚠️ Note: In JPQL, `Employee` refers to the **Java class name**, not the DB table name. `e.emailId` refers to the **Java field name**, not the column name.

---

## 13. Native SQL Queries

> Use `nativeQuery = true` when you want to write **raw SQL** against actual table/column names.

```java
@Query(value = "SELECT * FROM employees WHERE email_id = ?1", nativeQuery = true)
Optional<Employee> findByEmailUsingSql(String email);

@Query(value = "SELECT * FROM employees WHERE mobile = ?1", nativeQuery = true)
Optional<Employee> findByMobileUsingSql(String mobile);

@Query(value = "SELECT * FROM employees WHERE email_id = ?1 AND mobile = ?2", nativeQuery = true)
Optional<Employee> findByEmailAndMobileUsingSql(String email, String mobile);
```

| | JPQL | Native SQL |
|---|---|---|
| Uses | Entity class & field names | Table & column names |
| DB portable | ✅ Yes | ❌ Depends on DB dialect |
| `nativeQuery` flag | `false` (default) | `true` |

---

## 14. Hibernate Mappings

### Mapping Types Overview

| Mapping | Annotation | Example |
|---|---|---|
| One-to-Many | `@OneToMany` | One Post → Many Comments |
| Many-to-One | `@ManyToOne` | Many Comments → One Post |
| Many-to-Many | `@ManyToMany` | Students ↔ Courses |
| One-to-One | `@OneToOne` | One Person → One KYC |

---

### OneToMany — Unidirectional

> **Post** knows about **Comments**, but **Comment** does NOT know about **Post**.

```java
// Post.java
@Entity
public class Post {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;

    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinColumn(name = "post_id")   // FK added to comment table
    private List<Comment> comments = new ArrayList<>();

    // Getters & Setters ...
}
```

```java
// Comment.java
@Entity
public class Comment {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String content;

    // No reference back to Post (unidirectional)
    // Getters & Setters ...
}
```

**When to use unidirectional:**
- ✅ Small apps / POC / development/testing
- ✅ Less boilerplate, simpler to set up
- ❌ Not ideal for large datasets (EAGER can be expensive)

---

### OneToMany — Bidirectional

> Both **Post** and **Comment** know about each other.

```java
// Post.java
@Entity
public class Post {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;

    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL)
    // fetch defaults to LAZY for @OneToMany
    private List<Comment> comments = new ArrayList<>();

    // Getters & Setters ...
}
```

```java
// Comment.java
@Entity
public class Comment {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String content;

    @ManyToOne
    @JoinColumn(name = "post_id")   // FK lives in comment table
    private Post post;

    // Getters & Setters ...
}
```

**Saving a comment with bidirectional:**

```java
@Test
void addCommentToPost() {
    Post post = postRepository.findById(1L).orElseThrow();

    Comment comment = new Comment();
    comment.setContent("Great post!");
    comment.setPost(post);              // ✅ must set the owning side

    commentRepository.save(comment);
}
```

**Fetching comments (with LAZY loading):**

```java
@Transactional          // ✅ keeps Hibernate session open
@Test
void getPostWithComments() {
    Post post = postRepository.findById(1L).orElseThrow();
    post.getComments().forEach(c -> System.out.println(c.getContent()));
}
```

> ⚠️ Without `@Transactional`, accessing a lazily-loaded collection after the session closes throws:
> `LazyInitializationException: could not initialize proxy — no Session`

---

### ManyToMany Mapping

> Example: **Bus** ↔ **Stop** — a bus travels multiple stops, a stop has multiple buses.

```java
// Bus.java (owning side)
@Entity
public class Bus {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String busName;

    @ManyToMany
    @JoinTable(
        name = "bus_stop",
        joinColumns        = @JoinColumn(name = "bus_id"),
        inverseJoinColumns = @JoinColumn(name = "stop_id")
    )
    private List<Stop> stops = new ArrayList<>();

    // Getters & Setters ...
}
```

```java
// Stop.java (inverse side)
@Entity
public class Stop {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String stopName;

    @ManyToMany(mappedBy = "stops")
    private List<Bus> buses = new ArrayList<>();

    // Getters & Setters ...
}
```

**Saving buses with stops:**

```java
@Test
void createBusWithStops() {
    Stop s1 = new Stop(); s1.setStopName("Majestic");
    Stop s2 = new Stop(); s2.setStopName("BTM Layout");
    Stop s3 = new Stop(); s3.setStopName("Electronic City");

    stopRepo.saveAll(Arrays.asList(s1, s2, s3));

    Bus b1 = new Bus(); b1.setBusName("Bus 101"); b1.setStops(Arrays.asList(s1, s2));
    Bus b2 = new Bus(); b2.setBusName("Bus 202"); b2.setStops(Arrays.asList(s2, s3));

    busRepo.saveAll(Arrays.asList(b1, b2));
}
```

> 🗂️ Hibernate auto-creates the **bridge table** `bus_stop` with columns `bus_id` and `stop_id`.

---

### OneToOne — Bidirectional

> Example: **User** ↔ **Profile**

```java
// User.java (owning side — holds the FK)
@Entity
public class User {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")    // FK column in user table
    private Profile profile;

    // Getters & Setters ...
}
```

```java
// Profile.java (inverse side)
@Entity
public class Profile {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String bio;

    @OneToOne(mappedBy = "profile")     // no FK here
    private User user;

    // Getters & Setters ...
}
```

**Saving:**

```java
@Test
void saveUserWithProfile() {
    Profile profile = new Profile();
    profile.setBio("Full Stack Developer");

    User user = new User();
    user.setName("Sandeep");
    user.setProfile(profile);

    userRepo.save(user);    // CascadeType.ALL saves profile automatically
}
```

---

## 15. FetchType — EAGER vs LAZY

| | EAGER | LAZY |
|---|---|---|
| **When loaded** | Immediately with parent entity | Only when explicitly accessed |
| **SQL fired** | On parent fetch | On first access of collection |
| **Memory** | High (loads everything) | Low (loads on demand) |
| **Risk** | Performance issues on large data | `LazyInitializationException` if session closed |
| **Best for** | Small data / dev/testing | Production / large datasets |

### EAGER — What happens behind the scenes

```java
// When you run:
Post post = postRepository.findById(1L).orElseThrow();

// Hibernate fires:
// SELECT * FROM post WHERE id = 1;
// SELECT * FROM comment WHERE post_id = 1;   ← immediate
```

### LAZY — Problem without @Transactional

```java
Post post = postRepository.findById(1L).orElseThrow();
// Session closes here ↓
post.getComments();   // ❌ LazyInitializationException!
```

### LAZY — Fixed with @Transactional

```java
@Transactional
public void readComments() {
    Post post = postRepository.findById(1L).orElseThrow();
    post.getComments().forEach(c -> System.out.println(c.getContent())); // ✅ Works
}
```

---

## 16. Default Fetch Types

| Annotation | Default Fetch Type |
|---|---|
| `@OneToOne` | `EAGER` |
| `@ManyToOne` | `EAGER` |
| `@OneToMany` | `LAZY` |
| `@ManyToMany` | `LAZY` |

---

## 17. Hibernate Cache

> Hibernate Cache is an **in-memory (RAM) storage** mechanism that reduces repeated DB queries and improves performance.

**Benefits:**
- Avoids repetitive SQL queries
- Reduces DB load
- Lowers latency

### First-Level Cache

| Property | Value |
|---|---|
| Scope | Per Hibernate `Session` |
| Enabled by default | ✅ Always |
| Can be disabled | ❌ No |

```java
// Same session → second call hits cache, not DB
User user1 = session.get(User.class, 1L);   // DB hit
User user2 = session.get(User.class, 1L);   // ✅ From cache
```

### Second-Level Cache

| Property | Value |
|---|---|
| Scope | Shared across all sessions |
| Enabled by default | ❌ No — requires configuration |
| Providers | Ehcache, Caffeine, Infinispan, Redis |

```
User A logs in → loads User#1 → stored in 2nd-level cache
User B logs in → loads User#1 → ✅ served from cache (no DB hit!)
```

---

## 18. JUnit Testing in Spring Boot

> Unit testing in Spring Boot uses **JUnit 5** (included by default).  
> This is called **Unit / White-box Testing** — testing at the code level.

### JUnit Lifecycle Annotations

| Annotation | When it runs |
|---|---|
| `@Test` | Marks a method as a test case |
| `@BeforeEach` | Runs **before every** `@Test` method |
| `@AfterEach` | Runs **after every** `@Test` method |
| `@BeforeAll` | Runs **once before all** tests (must be `static`) |
| `@AfterAll` | Runs **once after all** tests (must be `static`) |

### Full Example

```java
@SpringBootTest
class Demo1ApplicationTests {

    @BeforeAll
    public static void beforeAll() {
        System.out.println("=== Before All Tests ===");
    }

    @BeforeEach
    public void beforeEach() {
        System.out.println("--- Before Each Test ---");
    }

    @Test
    void test1() {
        System.out.println("Running test1");
    }

    @Test
    void test2() {
        System.out.println("Running test2");
    }

    @AfterEach
    public void afterEach() {
        System.out.println("--- After Each Test ---");
    }

    @AfterAll
    public static void afterAll() {
        System.out.println("=== After All Tests ===");
    }
}
```

**Output:**
```
=== Before All Tests ===
--- Before Each Test ---
Running test1
--- After Each Test ---
--- Before Each Test ---
Running test2
--- After Each Test ---
=== After All Tests ===
```

---

## 19. Optional Class

> Introduced in **Java 8** as a safer alternative to handling `null` values — helps prevent `NullPointerException`.

```java
import java.util.Optional;

public class Example {
    int x = 10;

    public static void main(String[] args) {
        Example obj = new Example();

        Optional<Example> val = Optional.ofNullable(obj);

        if (val.isPresent()) {
            System.out.println(obj.x);      // 10
        } else {
            System.out.println("Null reference");
        }
    }
}
```

**Common Optional methods:**

| Method | Description |
|---|---|
| `Optional.of(val)` | Creates Optional; throws NPE if null |
| `Optional.ofNullable(val)` | Creates Optional; allows null |
| `Optional.empty()` | Creates an empty Optional |
| `.isPresent()` | Returns `true` if value exists |
| `.get()` | Returns the value (throws if empty) |
| `.orElse(default)` | Returns value or a default |
| `.orElseThrow()` | Returns value or throws exception |
| `.ifPresent(action)` | Runs action if value is present |

---

## 20. SessionFactory (Interview Question)

> **SessionFactory** is a core Hibernate interface responsible for:
> - **Connecting** to the database
> - **Creating `Session` objects** used to perform `INSERT`, `UPDATE`, `DELETE`, and queries

```
SessionFactory
     │
     ├── creates → Session (per request / unit of work)
     │                 │
     │                 ├── save()
     │                 ├── get() / load()
     │                 ├── update()
     │                 └── delete()
     │
     └── manages → First-Level Cache (per Session)
```

- `SessionFactory` is **thread-safe** and created once at startup
- `Session` is **not thread-safe** and should be used per transaction/request

---

*Notes compiled from Spring Boot + Hibernate coursework. Last updated: May 2026.*
