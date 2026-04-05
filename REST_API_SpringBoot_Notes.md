# 🚀 REST API with Java Spring Boot — Complete Notes
### From Beginner to Experience Level | Interview Preparation Included

---

## 📑 Table of Contents

1. [What is REST API?](#1-what-is-rest-api)
2. [Core REST Principles (Constraints)](#2-core-rest-principles-constraints)
3. [HTTP Methods](#3-http-methods)
4. [HTTP Status Codes](#4-http-status-codes)
5. [Spring Boot Setup](#5-spring-boot-setup)
6. [Project Structure](#6-project-structure)
7. [Core Annotations](#7-core-annotations)
8. [Building Your First REST Controller](#8-building-your-first-rest-controller)
9. [Request Mapping & URL Patterns](#9-request-mapping--url-patterns)
10. [Request Parameters, Path Variables & Request Body](#10-request-parameters-path-variables--request-body)
11. [Response Entity & Status Codes](#11-responseentity--status-codes)
12. [Service Layer & Dependency Injection](#12-service-layer--dependency-injection)
13. [Repository Layer (JPA)](#13-repository-layer-jpa)
14. [DTOs (Data Transfer Objects)](#14-dtos-data-transfer-objects)
15. [Exception Handling](#15-exception-handling)
16. [Validation](#16-validation)
17. [Pagination & Sorting](#17-pagination--sorting)
18. [Spring Security & JWT Basics](#18-spring-security--jwt-basics)
19. [CORS Configuration](#19-cors-configuration)
20. [API Versioning](#20-api-versioning)
21. [Swagger / OpenAPI Documentation](#21-swagger--openapi-documentation)
22. [Testing REST APIs](#22-testing-rest-apis)
23. [Best Practices](#23-best-practices)
24. [Top Interview Questions & Answers](#24-top-interview-questions--answers)

---

## 1. What is REST API?

**REST** stands for **Representational State Transfer**. It is an architectural style for designing networked applications. A REST API (also called RESTful API) uses HTTP requests to perform CRUD operations.

| Term | Meaning |
|------|---------|
| **Resource** | Any data entity (e.g., User, Product, Order) |
| **Endpoint** | URL that represents a resource (e.g., `/api/users`) |
| **Client** | The one making the request (browser, mobile app, Postman) |
| **Server** | The one responding (your Spring Boot app) |
| **Stateless** | Each request is independent; no session stored on server |

### Why REST?
- Language-independent (client can be in any language)
- Uses standard HTTP protocol
- Easy to scale and maintain
- Widely adopted industry standard

---

## 2. Core REST Principles (Constraints)

REST has **6 guiding constraints** or **6 Rules or principles**:

| # | Constraint | Explanation |
|---|-----------|-------------|
| 1 | **Client-Server** | UI and backend are separated. Client handles UI, server handles data. |
| 2 | **Stateless** | Each HTTP request must contain ALL information needed. Server stores NO session. |
| 3 | **Cacheable** | Responses should indicate if they can be cached to improve performance. |
| 4 | **Uniform Interface** | Standard way to interact with resources (HTTP verbs + URIs). |
| 5 | **Layered System** | Client doesn't know if it's talking to the actual server or a proxy/load balancer. |
| 6 | **Code on Demand** *(optional)* | Server can send executable code (like JavaScript) to the client. |

> 💡 **Interview Tip:** Stateless is the most important constraint. The server does NOT remember previous requests.

---

## 3. HTTP Methods

These are the verbs used in REST APIs:

| Method | CRUD Operation | Description | Idempotent? | Safe? |
|--------|---------------|-------------|-------------|-------|
| **GET** | Read | Retrieve resource | ✅ Yes | ✅ Yes |
| **POST** | Create | Create a new resource | ❌ No | ❌ No |
| **PUT** | Update (Full) | Replace entire resource | ✅ Yes | ❌ No |
| **PATCH** | Update (Partial) | Update specific fields | ❌ No | ❌ No |
| **DELETE** | Delete | Remove a resource | ✅ Yes | ❌ No |

> **Idempotent** = Making the same request multiple times gives the same result.
> **Safe** = Request does NOT modify data.

### Real-world URL Design Examples:

```
GET    /api/users          → Get all users
GET    /api/users/1        → Get user with ID 1
POST   /api/users          → Create a new user
PUT    /api/users/1        → Replace user with ID 1 (full update)
PATCH  /api/users/1        → Partially update user with ID 1
DELETE /api/users/1        → Delete user with ID 1
```

---

## 4. HTTP Status Codes

Every HTTP response includes a **status code** that tells the client what happened.

### 2xx — Success

| Code | Name | When to Use |
|------|------|-------------|
| `200 OK` | OK | Successful GET, PUT, PATCH, DELETE |
| `201 Created` | Created | Successful POST (resource created) |
| `204 No Content` | No Content | Successful DELETE (nothing to return) |

### 4xx — Client Errors

| Code | Name | When to Use |
|------|------|-------------|
| `400 Bad Request` | Bad Request | Invalid input / validation failure |
| `401 Unauthorized` | Unauthorized | Not authenticated (no/invalid token) |
| `403 Forbidden` | Forbidden | Authenticated but not authorized |
| `404 Not Found` | Not Found | Resource doesn't exist |
| `409 Conflict` | Conflict | Duplicate resource (e.g., email already exists) |
| `422 Unprocessable Entity` | Validation Error | Semantically invalid data |

### 5xx — Server Errors

| Code | Name | When to Use |
|------|------|-------------|
| `500 Internal Server Error` | Server Error | Unexpected server-side error |
| `503 Service Unavailable` | Unavailable | Server down / overloaded |

---

## 5. Spring Boot Setup

### Prerequisites
- Java 17+
- Maven or Gradle
- IDE (IntelliJ IDEA recommended)

### Dependencies (pom.xml)

```xml
<dependencies>

    <!-- Core Web (REST API) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- JPA + Database -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- H2 In-Memory Database (for testing/dev) -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- MySQL (for production) -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Lombok (reduces boilerplate) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

### application.properties

```properties
# Server
server.port=8080

# MySQL Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# H2 Console (dev only)
spring.h2.console.enabled=true
```

---

## 6. Project Structure

```
src/
├── main/
│   ├── java/
│   │   └── com/example/myapp/
│   │       ├── MyAppApplication.java        ← Main class
│   │       ├── controller/
│   │       │   └── UserController.java      ← REST endpoints
│   │       ├── service/
│   │       │   ├── UserService.java         ← Interface
│   │       │   └── UserServiceImpl.java     ← Business logic
│   │       ├── repository/
│   │       │   └── UserRepository.java      ← DB operations (JPA)
│   │       ├── model/
│   │       │   └── User.java                ← Entity / Domain class
│   │       ├── dto/
│   │       │   ├── UserRequestDTO.java      ← Input DTO
│   │       │   └── UserResponseDTO.java     ← Output DTO
│   │       └── exception/
│   │           ├── ResourceNotFoundException.java
│   │           └── GlobalExceptionHandler.java
│   └── resources/
│       └── application.properties
└── test/
    └── java/
        └── com/example/myapp/
            └── UserControllerTest.java
```

> 💡 **This is the standard layered architecture:** Controller → Service → Repository → Database

---

## 7. Core Annotations

### Controller Annotations

| Annotation | Purpose |
|-----------|---------|
| `@RestController` | Marks class as REST controller. Combines `@Controller` + `@ResponseBody` |
| `@Controller` | Traditional MVC controller (returns view names) |
| `@RequestMapping` | Maps URL path to class or method |
| `@GetMapping` | Shortcut for `@RequestMapping(method = GET)` |
| `@PostMapping` | Shortcut for `@RequestMapping(method = POST)` |
| `@PutMapping` | Shortcut for `@RequestMapping(method = PUT)` |
| `@PatchMapping` | Shortcut for `@RequestMapping(method = PATCH)` |
| `@DeleteMapping` | Shortcut for `@RequestMapping(method = DELETE)` |
| `@ResponseBody` | Serializes return value to JSON/XML |
| `@ResponseStatus` | Sets the HTTP status code on the response |

### Parameter Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@PathVariable` | Extracts value from URL path | `/users/{id}` |
| `@RequestParam` | Extracts query parameter from URL | `/users?name=John` |
| `@RequestBody` | Deserializes JSON body into Java object | POST body |
| `@RequestHeader` | Extracts value from HTTP header | `Authorization` header |

### Component/Bean Annotations

| Annotation | Purpose |
|-----------|---------|
| `@Component` | Generic Spring-managed bean |
| `@Service` | Business logic layer bean |
| `@Repository` | Data access layer bean (also translates DB exceptions) |
| `@Configuration` | Java-based configuration class |
| `@Bean` | Declares a Spring bean inside `@Configuration` |
| `@Autowired` | Injects a dependency (prefer constructor injection) |

### JPA / Database Annotations

| Annotation | Purpose |
|-----------|---------|
| `@Entity` | Marks class as a JPA entity (database table) |
| `@Table(name="...")` | Specifies table name |
| `@Id` | Marks primary key field |
| `@GeneratedValue` | Auto-generates primary key value |
| `@Column(name="...")` | Maps field to a column |
| `@OneToMany` | One-to-many relationship |
| `@ManyToOne` | Many-to-one relationship |
| `@ManyToMany` | Many-to-many relationship |
| `@JoinColumn` | Specifies FK column in a relationship |

---

## 8. Building Your First REST Controller

### Step 1: Create the Entity (Model)

```java
package com.example.myapp.model;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "users")
@Data                    // Lombok: generates getters, setters, toString, equals, hashCode
@NoArgsConstructor       // Lombok: generates no-arg constructor
@AllArgsConstructor      // Lombok: generates all-arg constructor
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "first_name", nullable = false)
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(unique = true, nullable = false)
    private String email;
}
```

### Step 2: Create the Repository

```java
package com.example.myapp.repository;

import com.example.myapp.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // JpaRepository provides: save(), findById(), findAll(), deleteById(), etc.

    // Custom query methods (Spring Data JPA auto-generates SQL from method names):
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

### Step 3: Create the Service

```java
package com.example.myapp.service;

import com.example.myapp.model.User;
import java.util.List;

public interface UserService {
    List<User> getAllUsers();
    User getUserById(Long id);
    User createUser(User user);
    User updateUser(Long id, User user);
    void deleteUser(Long id);
}
```

```java
package com.example.myapp.service;

import com.example.myapp.exception.ResourceNotFoundException;
import com.example.myapp.model.User;
import com.example.myapp.repository.UserRepository;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository; // Constructor injection (preferred over @Autowired)

    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    @Override
    public User getUserById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
    }

    @Override
    public User createUser(User user) {
        return userRepository.save(user);
    }

    @Override
    public User updateUser(Long id, User userDetails) {
        User user = getUserById(id); // Reuse method (also throws 404 if not found)
        user.setFirstName(userDetails.getFirstName());
        user.setLastName(userDetails.getLastName());
        user.setEmail(userDetails.getEmail());
        return userRepository.save(user);
    }

    @Override
    public void deleteUser(Long id) {
        User user = getUserById(id); // Ensure user exists before deleting
        userRepository.delete(user);
    }
}
```

### Step 4: Create the Controller

```java
package com.example.myapp.controller;

import com.example.myapp.model.User;
import com.example.myapp.service.UserService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    // GET /api/v1/users
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.getAllUsers();
        return ResponseEntity.ok(users); // 200 OK
    }

    // GET /api/v1/users/1
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.getUserById(id);
        return ResponseEntity.ok(user); // 200 OK
    }

    // POST /api/v1/users
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.createUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created); // 201 Created
    }

    // PUT /api/v1/users/1
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
        User updated = userService.updateUser(id, user);
        return ResponseEntity.ok(updated); // 200 OK
    }

    // DELETE /api/v1/users/1
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build(); // 204 No Content
    }
}
```

---

## 9. Request Mapping & URL Patterns

```java
// Class-level mapping (base path for all methods in this class)
@RequestMapping("/api/v1/products")
public class ProductController {

    // GET /api/v1/products
    @GetMapping
    public List<Product> getAll() { ... }

    // GET /api/v1/products/5
    @GetMapping("/{id}")
    public Product getById(@PathVariable Long id) { ... }

    // GET /api/v1/products/category/electronics
    @GetMapping("/category/{categoryName}")
    public List<Product> getByCategory(@PathVariable String categoryName) { ... }

    // GET /api/v1/products/search?name=laptop&minPrice=500
    @GetMapping("/search")
    public List<Product> search(
        @RequestParam String name,
        @RequestParam(required = false, defaultValue = "0") Double minPrice
    ) { ... }
}
```

---

## 10. Request Parameters, Path Variables & Request Body

### @PathVariable — Extract from URL path

```java
// URL: GET /users/42
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) { ... }

// URL: GET /users/42/orders/7
@GetMapping("/users/{userId}/orders/{orderId}")
public Order getOrder(
    @PathVariable Long userId,
    @PathVariable Long orderId
) { ... }
```

### @RequestParam — Extract from query string

```java
// URL: GET /users?page=0&size=10&sort=name
@GetMapping("/users")
public List<User> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(required = false) String sort
) { ... }
```

### @RequestBody — Extract from request body (JSON)

```java
// POST /users with body: { "firstName": "John", "email": "john@example.com" }
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    // Spring auto-deserializes JSON → User object
    return userService.create(user);
}
```

### @RequestHeader — Extract from HTTP header

```java
@GetMapping("/profile")
public String getProfile(@RequestHeader("Authorization") String authToken) {
    // authToken = "Bearer eyJhbGciO..."
    return authToken;
}
```

---

## 11. ResponseEntity & Status Codes

`ResponseEntity<T>` is the most powerful way to return HTTP responses. It lets you control:
- **Status Code**
- **Headers**
- **Body**

```java
// Return 200 OK with body
return ResponseEntity.ok(user);

// Return 201 Created with body
return ResponseEntity.status(HttpStatus.CREATED).body(createdUser);

// Return 204 No Content (no body)
return ResponseEntity.noContent().build();

// Return 404 Not Found with custom message
return ResponseEntity.status(HttpStatus.NOT_FOUND).body("User not found");

// Return with custom headers
HttpHeaders headers = new HttpHeaders();
headers.add("X-Custom-Header", "value");
return ResponseEntity.ok().headers(headers).body(user);

// Return 400 Bad Request
return ResponseEntity.badRequest().body("Invalid input");
```

---

## 12. Service Layer & Dependency Injection

### Why a Service Layer?
- Separates business logic from HTTP handling (controller)
- Makes code testable and reusable
- Single Responsibility Principle

### Dependency Injection (DI)

Spring manages object creation. You just declare what you need:

```java
// ❌ BAD — Manual object creation (tightly coupled)
public class UserController {
    private UserService userService = new UserServiceImpl(); // Don't do this!
}

// ✅ GOOD — Constructor Injection (recommended, immutable, testable)
@RestController
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService; // Spring injects automatically
    }
}

// ✅ ALSO OK — @Autowired field injection (less preferred)
@RestController
public class UserController {
    @Autowired
    private UserService userService;
}
```

> 💡 **Interview Tip:** Always prefer **constructor injection** over field injection. Reasons:
> - Supports immutability (`final` fields)
> - Makes dependencies explicit
> - Easier to unit test (no Spring context needed)

---

## 13. Repository Layer (JPA)

### JpaRepository Built-in Methods

```java
// Extends JpaRepository<EntityType, PrimaryKeyType>
public interface UserRepository extends JpaRepository<User, Long> { }

// Available methods out of the box:
userRepository.save(user)                   // Insert or Update
userRepository.findById(1L)                 // Returns Optional<User>
userRepository.findAll()                    // Returns List<User>
userRepository.findAll(pageable)            // Returns Page<User>
userRepository.deleteById(1L)               // Delete by ID
userRepository.delete(user)                 // Delete entity
userRepository.existsById(1L)               // Returns boolean
userRepository.count()                      // Count all records
```

### Derived Query Methods (Spring generates SQL automatically)

```java
// Spring Data JPA reads the method name and auto-generates the SQL!
List<User> findByFirstName(String firstName);
// SQL: SELECT * FROM users WHERE first_name = ?

List<User> findByFirstNameAndLastName(String firstName, String lastName);
// SQL: SELECT * FROM users WHERE first_name = ? AND last_name = ?

List<User> findByAgeGreaterThan(int age);
// SQL: SELECT * FROM users WHERE age > ?

List<User> findByEmailContaining(String keyword);
// SQL: SELECT * FROM users WHERE email LIKE '%keyword%'

List<User> findByOrderByLastNameAsc();
// SQL: SELECT * FROM users ORDER BY last_name ASC

boolean existsByEmail(String email);
// SQL: SELECT COUNT(*) > 0 FROM users WHERE email = ?
```

### Custom JPQL Query

```java
@Query("SELECT u FROM User u WHERE u.email = :email AND u.active = true")
Optional<User> findActiveUserByEmail(@Param("email") String email);

// Native SQL query
@Query(value = "SELECT * FROM users WHERE created_at > :date", nativeQuery = true)
List<User> findUsersCreatedAfter(@Param("date") LocalDate date);
```

---

## 14. DTOs (Data Transfer Objects)

### Why Use DTOs?
- **Security:** Don't expose internal entity fields (e.g., password)
- **Flexibility:** API response shape can differ from DB schema
- **Validation:** Validate input separately from entity

```java
// Request DTO — what client sends
public class UserRequestDTO {
    @NotBlank(message = "First name is required")
    private String firstName;

    @NotBlank(message = "Last name is required")
    private String lastName;

    @Email(message = "Invalid email format")
    @NotBlank(message = "Email is required")
    private String email;

    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;

    // Getters & Setters (or use Lombok @Data)
}

// Response DTO — what server returns (NO password!)
public class UserResponseDTO {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String createdAt;

    // Getters & Setters
}
```

### Mapping Entity ↔ DTO

```java
// Manual mapping
public UserResponseDTO toResponseDTO(User user) {
    UserResponseDTO dto = new UserResponseDTO();
    dto.setId(user.getId());
    dto.setFirstName(user.getFirstName());
    dto.setEmail(user.getEmail());
    return dto;
}

// Or use ModelMapper library (add to pom.xml)
// <dependency>
//   <groupId>org.modelmapper</groupId>
//   <artifactId>modelmapper</artifactId>
//   <version>3.1.1</version>
// </dependency>

@Bean
public ModelMapper modelMapper() {
    return new ModelMapper();
}

// Usage:
UserResponseDTO dto = modelMapper.map(user, UserResponseDTO.class);
```

---

## 15. Exception Handling

### Custom Exception Class

```java
package com.example.myapp.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

### Global Exception Handler

```java
package com.example.myapp.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice  // Applies to ALL controllers
public class GlobalExceptionHandler {

    // Handles 404 Not Found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    // Handles Validation Errors (400 Bad Request)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        return ResponseEntity.badRequest().body(errors);
    }

    // Handles any unexpected exception (500 Internal Server Error)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return ResponseEntity.internalServerError().body(error);
    }
}
```

### Error Response Model

```java
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;

    // Constructor, Getters & Setters
}
```

**JSON error response example:**
```json
{
  "status": 404,
  "message": "User not found with id: 99",
  "timestamp": "2024-01-15T10:30:00"
}
```

---

## 16. Validation

### Validation Annotations (jakarta.validation)

```java
public class UserRequestDTO {
    @NotNull               // Field cannot be null
    @NotBlank              // String cannot be null, empty, or whitespace
    @NotEmpty              // String/collection cannot be null or empty
    private String name;

    @Size(min = 2, max = 50)         // String length constraint
    private String username;

    @Min(18)                          // Minimum numeric value
    @Max(120)                         // Maximum numeric value
    private Integer age;

    @Email                            // Must be valid email format
    private String email;

    @Pattern(regexp = "^[0-9]{10}$")  // Must match regex
    private String phone;

    @Positive                         // Must be > 0
    private Double price;

    @Future                           // Date must be in the future
    private LocalDate dueDate;

    @Past                             // Date must be in the past
    private LocalDate birthDate;
}
```

### Trigger Validation in Controller

```java
@PostMapping
public ResponseEntity<User> create(@Valid @RequestBody UserRequestDTO dto) {
    // @Valid triggers validation; if fails, MethodArgumentNotValidException is thrown
    // GlobalExceptionHandler catches it and returns 400 with field errors
    ...
}
```

---

## 17. Pagination & Sorting

### API Request

```
GET /api/users?page=0&size=10&sort=firstName,asc
```

### Controller

```java
@GetMapping
public ResponseEntity<Page<User>> getAllUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "id") String sortBy,
    @RequestParam(defaultValue = "asc") String sortDir
) {
    Sort sort = sortDir.equalsIgnoreCase("asc")
        ? Sort.by(sortBy).ascending()
        : Sort.by(sortBy).descending();

    Pageable pageable = PageRequest.of(page, size, sort);
    Page<User> users = userService.getAllUsers(pageable);
    return ResponseEntity.ok(users);
}
```

### Service & Repository

```java
// Service
public Page<User> getAllUsers(Pageable pageable) {
    return userRepository.findAll(pageable);
}

// Repository — JpaRepository already supports Pageable!
// No extra code needed for simple pagination
```

### Page Response Structure

```json
{
  "content": [
    { "id": 1, "firstName": "John" },
    { "id": 2, "firstName": "Jane" }
  ],
  "pageNumber": 0,
  "pageSize": 10,
  "totalElements": 100,
  "totalPages": 10,
  "last": false,
  "first": true
}
```

---

## 18. Spring Security & JWT Basics

### What is JWT?

**JSON Web Token (JWT)** is a compact, URL-safe token used for authentication.

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJqb2huQGV4YW1wbGUuY29tIn0.SIGNATURE
     ↑ Header (Base64)       ↑ Payload (Base64)              ↑ Signature
```

Structure: `Header.Payload.Signature`

### Authentication Flow

```
1. Client: POST /api/auth/login  { email, password }
2. Server: Validate credentials
3. Server: Generate JWT token
4. Server: Return token to client
5. Client: Store token (localStorage / cookie)
6. Client: Send token in every request: Authorization: Bearer <token>
7. Server: Validate token on each request
8. Server: Allow or deny access
```

### Security Configuration (Spring Boot 3.x)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // Disable CSRF for stateless REST API
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // No sessions
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()       // Public endpoints
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // Admin only
                .anyRequest().authenticated()                       // All others require auth
            );

        // Add JWT filter before Spring's default UsernamePasswordAuthenticationFilter
        http.addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // Always hash passwords!
    }
}
```

---

## 19. CORS Configuration

**CORS (Cross-Origin Resource Sharing)** — browsers block requests from different origins by default.

### Method 1: `@CrossOrigin` on controller

```java
@CrossOrigin(origins = "http://localhost:3000") // Allow React frontend
@RestController
public class UserController { ... }
```

### Method 2: Global CORS configuration (recommended)

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("http://localhost:3000", "https://myapp.com")
                .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

---

## 20. API Versioning

### Why Version APIs?
When you change your API (new fields, different structure), old clients shouldn't break.

### Strategy 1: URI Versioning (Most Common)

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserV1Controller { ... }

@RestController
@RequestMapping("/api/v2/users")
public class UserV2Controller { ... }
```

### Strategy 2: Header Versioning

```java
@GetMapping(value = "/users", headers = "X-API-VERSION=1")
public UserV1DTO getUserV1() { ... }

@GetMapping(value = "/users", headers = "X-API-VERSION=2")
public UserV2DTO getUserV2() { ... }
```

### Strategy 3: Request Param Versioning

```java
@GetMapping(value = "/users", params = "version=1")
public UserV1DTO getUserV1() { ... }
```

---

## 21. Swagger / OpenAPI Documentation

Auto-generates interactive API documentation.

### Dependency

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

### Access URL after running app:
```
http://localhost:8080/swagger-ui/index.html
```

### Annotating Your API

```java
@Tag(name = "User Management", description = "APIs for managing users")
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @Operation(summary = "Get user by ID", description = "Fetches a single user by their ID")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(
        @Parameter(description = "ID of the user to fetch") @PathVariable Long id
    ) {
        ...
    }
}
```

---

## 22. Testing REST APIs

### Unit Test (Service Layer)

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository; // Mock DB calls

    @InjectMocks
    private UserServiceImpl userService; // Real service with mocked repo

    @Test
    void shouldReturnUserWhenValidIdProvided() {
        // Given (Arrange)
        User mockUser = new User(1L, "John", "Doe", "john@example.com");
        when(userRepository.findById(1L)).thenReturn(Optional.of(mockUser));

        // When (Act)
        User result = userService.getUserById(1L);

        // Then (Assert)
        assertNotNull(result);
        assertEquals("John", result.getFirstName());
        verify(userRepository, times(1)).findById(1L);
    }

    @Test
    void shouldThrowExceptionWhenUserNotFound() {
        when(userRepository.findById(99L)).thenReturn(Optional.empty());

        assertThrows(ResourceNotFoundException.class, () -> userService.getUserById(99L));
    }
}
```

### Integration Test (Controller Layer)

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc; // Simulates HTTP requests

    @Autowired
    private ObjectMapper objectMapper; // For JSON conversion

    @MockBean
    private UserService userService;

    @Test
    void shouldReturn200WhenGetAllUsers() throws Exception {
        List<User> users = List.of(new User(1L, "John", "Doe", "john@test.com"));
        when(userService.getAllUsers()).thenReturn(users);

        mockMvc.perform(get("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$[0].firstName").value("John"))
                .andDo(print());
    }

    @Test
    void shouldReturn201WhenUserCreated() throws Exception {
        User user = new User(null, "Jane", "Smith", "jane@test.com");
        User saved = new User(1L, "Jane", "Smith", "jane@test.com");

        when(userService.createUser(any(User.class))).thenReturn(saved);

        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(user)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1));
    }
}
```

---

## 23. Best Practices

### ✅ URL Design

```
✅ Good                             ❌ Bad
/api/v1/users                      /api/v1/getUsers
/api/v1/users/5                    /api/v1/getUserById?id=5
/api/v1/users/5/orders             /api/v1/getOrdersForUser/5
DELETE /api/v1/users/5             /api/v1/deleteUser/5
```

- Use **nouns** for resources, not verbs
- Use **plural nouns** (`/users` not `/user`)
- Use **lowercase** and **hyphens** for readability (`/user-profiles`)
- Use **nesting** for relationships (`/users/1/orders`)
- Always **version** your API (`/api/v1/`)

### ✅ Response Consistency

Always return a consistent structure:

```json
{
  "success": true,
  "data": { ... },
  "message": "User created successfully",
  "timestamp": "2024-01-15T10:30:00"
}
```

### ✅ Security Checklist

- [ ] Always validate and sanitize input
- [ ] Never expose internal entity IDs directly (use UUIDs or obfuscation)
- [ ] Always hash passwords with BCrypt
- [ ] Use HTTPS in production
- [ ] Return appropriate error messages (don't expose stack traces)
- [ ] Implement rate limiting
- [ ] Use JWT expiry times

### ✅ Performance Tips

- Use pagination for large datasets
- Use `@Transactional(readOnly = true)` for read operations
- Use DTOs to avoid over-fetching (only return needed fields)
- Enable caching with `@Cacheable` for frequent reads
- Use database indexes on commonly queried fields

---

## 24. Top Interview Questions & Answers

---

### 🔵 BEGINNER LEVEL

**Q1. What is REST API?**
> REST (Representational State Transfer) is an architectural style for building web services using HTTP. It defines a set of constraints like statelessness, client-server separation, and uniform interface. A REST API exposes resources (like users, products) through URLs and uses HTTP methods (GET, POST, PUT, DELETE) to perform operations.

**Q2. What is the difference between @Controller and @RestController?**
> `@Controller` is a traditional Spring MVC annotation that returns a view name (for web pages). `@RestController` = `@Controller` + `@ResponseBody`, which means it serializes return values directly to JSON/XML and sends them in the response body. For REST APIs, always use `@RestController`.

**Q3. What is the difference between PUT and PATCH?**
> `PUT` replaces the entire resource. If you send PUT with only 2 of 5 fields, the other 3 fields become null. `PATCH` partially updates a resource — only the fields you send are updated. PUT is idempotent; PATCH may or may not be idempotent.

**Q4. What does idempotent mean in HTTP?**
> An operation is idempotent if making the same request multiple times produces the same result. GET, PUT, DELETE are idempotent. POST is NOT idempotent (calling POST multiple times creates multiple resources).

**Q5. What is @RequestBody vs @RequestParam?**
> `@RequestBody` reads data from the HTTP request body (usually JSON). Used for POST/PUT requests with complex data. `@RequestParam` reads data from the URL query string (e.g., `/users?name=John`). Used for filtering/searching.

**Q6. What HTTP status code should POST return on success?**
> `201 Created` — This indicates a new resource was successfully created. The response should also include a `Location` header pointing to the new resource URL.

---

### 🟡 INTERMEDIATE LEVEL

**Q7. What is Spring Boot auto-configuration?**
> Spring Boot automatically configures your application based on the dependencies in the classpath. If you add `spring-boot-starter-web`, Spring Boot automatically configures DispatcherServlet, Jackson (JSON), and an embedded Tomcat server. You can override any configuration using `application.properties`.

**Q8. What is @RestControllerAdvice? How does global exception handling work?**
> `@RestControllerAdvice` is a specialization of `@ControllerAdvice` that applies to all controllers. Combined with `@ExceptionHandler`, it intercepts specific exceptions thrown anywhere in the application and returns structured error responses. This centralizes error handling and avoids try-catch blocks in every controller method.

**Q9. What is the difference between @Component, @Service, @Repository?**
> All three are specializations of `@Component` and tell Spring to create a bean. The difference is semantic and tooling support: `@Service` marks business logic layer, `@Repository` marks data access layer (also enables exception translation — converts DB exceptions to Spring's DataAccessException). `@Component` is a generic stereotype for any other bean.

**Q10. Why is constructor injection preferred over @Autowired field injection?**
> Constructor injection: (1) supports immutability (`final` fields), (2) makes dependencies explicit — you can't instantiate the class without providing them, (3) easier to unit test — no Spring context needed, just pass mock objects in the constructor, (4) avoids circular dependency issues (Spring fails fast at startup instead of at runtime).

**Q11. What is a DTO and why should we use it?**
> A Data Transfer Object (DTO) is a simple object that carries data between layers. We use DTOs to: (1) control what data is exposed (security — never expose password or sensitive fields), (2) decouple API contract from internal domain model, (3) apply input validation separately, (4) create API-specific response shapes without changing entities.

**Q12. Explain Spring Data JPA and derived query methods.**
> Spring Data JPA is an abstraction over JPA that reduces boilerplate for database operations. `JpaRepository` provides ready-made CRUD methods. Derived query methods let you define methods whose names are parsed by Spring to auto-generate JPQL queries. E.g., `findByEmailAndActive(String email, boolean active)` → `SELECT u FROM User u WHERE u.email = :email AND u.active = :active`.

**Q13. What is @Transactional and when should you use it?**
> `@Transactional` wraps a method in a database transaction. If the method completes successfully, the transaction commits. If an exception is thrown, it rolls back. Use it on service methods that perform multiple database operations that must succeed or fail together. Use `@Transactional(readOnly = true)` on read-only methods for performance optimization.

---

### 🔴 ADVANCED LEVEL

**Q14. How does JWT authentication work in Spring Boot?**
> Flow: (1) Client sends credentials to `/auth/login`. (2) Server validates credentials, generates a signed JWT token containing claims (user ID, roles, expiry). (3) Client stores the token and sends it in the `Authorization: Bearer <token>` header with every subsequent request. (4) A custom `OncePerRequestFilter` (JWT filter) intercepts each request, extracts and validates the token, and sets the Authentication in Spring's SecurityContext. (5) Spring Security then applies authorization rules.

**Q15. What is the difference between Authentication and Authorization?**
> Authentication = verifying WHO you are (login with username/password → you get a token). Authorization = verifying WHAT you can do (does this user have the ADMIN role to access `/admin/users`?). Authentication comes first, then authorization.

**Q16. What is CORS and how do you configure it?**
> CORS (Cross-Origin Resource Sharing) is a browser security mechanism that blocks requests from a different origin (domain/port/protocol). For example, a React app on `localhost:3000` can't call a Spring Boot API on `localhost:8080` by default. In Spring Boot, you configure CORS using `WebMvcConfigurer.addCorsMappings()` to specify which origins, methods, and headers are allowed. For REST APIs, you can also use `@CrossOrigin` on individual controllers.

**Q17. How do you handle N+1 query problem in Spring Data JPA?**
> N+1 occurs when fetching a list of entities triggers N additional queries for each entity's related data. Solutions: (1) Use `JOIN FETCH` in JPQL: `SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id`. (2) Use `@EntityGraph` annotation to specify which relationships to fetch eagerly. (3) Use `FetchType.LAZY` by default and fetch only what you need. (4) Use `@BatchSize` for batch fetching.

**Q18. Explain the difference between optimistic and pessimistic locking.**
> Both prevent concurrent update conflicts. **Pessimistic locking** locks the row in the database when you read it (no one else can update until you release the lock). Use `@Lock(LockModeType.PESSIMISTIC_WRITE)`. **Optimistic locking** doesn't lock at write time; instead, a `@Version` field is checked on update — if someone else already updated the row, an `OptimisticLockException` is thrown. Optimistic locking is preferred for high-concurrency read-heavy scenarios.

**Q19. What is the difference between `@OneToMany` with `FetchType.LAZY` vs `EAGER`?**
> `EAGER` fetches the related collection immediately when the parent entity is loaded (always an extra query or JOIN). `LAZY` defers loading until the collection is actually accessed in code. LAZY is always preferred as default — only load what you need. Warning: accessing a LAZY collection outside a transaction causes `LazyInitializationException`. Use `@Transactional` on your service method or use DTOs to transfer the data before the session closes.

**Q20. How would you design a REST API for a social media platform?**
> Key design decisions: (1) Resource-based URLs: `/users`, `/posts`, `/comments`, `/likes`. (2) Nested resources for relationships: `GET /posts/1/comments`. (3) Use pagination for feed endpoints: `GET /feed?page=0&size=20`. (4) Use HATEOAS for discoverability. (5) Version the API: `/api/v1/`. (6) Use JWT for authentication with refresh token pattern. (7) Implement rate limiting per user. (8) Use soft deletes (mark as deleted, don't physically delete). (9) Use event sourcing or messaging (Kafka) for notifications.

---

## 📌 Quick Reference Summary

```
Architecture Layer:    Controller → Service → Repository → Database
HTTP Methods:          GET(Read) POST(Create) PUT(Full Update) PATCH(Partial) DELETE
Status Codes:          200 OK | 201 Created | 204 No Content | 400 Bad Request
                       401 Unauthorized | 403 Forbidden | 404 Not Found | 500 Server Error
Key Annotations:       @RestController @RequestMapping @GetMapping @PostMapping
                       @PathVariable @RequestParam @RequestBody @Valid
                       @Service @Repository @Entity @Id @GeneratedValue
                       @RestControllerAdvice @ExceptionHandler @Transactional
DI Preference:         Constructor Injection > Field Injection (@Autowired)
Best Practice:         Use DTOs, validate input, handle exceptions globally, version your API
Security:              BCrypt passwords, JWT tokens, HTTPS, never expose stack traces
```

---

*Happy Coding & All the Best for Your Interviews! 🎯*
