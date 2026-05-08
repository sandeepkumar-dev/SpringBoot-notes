# 🚀 Real-World Spring Boot Project — Complete Workflow Notes
> A beginner-friendly, detailed guide to understanding how a Spring Boot project works from scratch.

---

## 📌 Table of Contents

1. [What is Spring Boot?](#1-what-is-spring-boot)
2. [How Spring Boot Starts](#2-how-spring-boot-starts)
3. [Project Structure](#3-project-structure)
4. [Layer by Layer Explanation](#4-layer-by-layer-explanation)
   - [Entity Layer](#-layer-1--entity)
   - [Repository Layer](#-layer-2--repository)
   - [DTO Layer](#-layer-3--dto-data-transfer-object)
   - [Mapper Layer](#-layer-4--mapper)
   - [Service Layer](#-layer-5--service)
   - [Controller Layer](#-layer-6--controller)
   - [Config Layer](#-layer-7--config)
   - [Exception Layer](#-layer-8--exception-handling)
5. [application.properties](#5-applicationproperties)
6. [Full Request Lifecycle](#6-full-request-lifecycle)
7. [REST API Methods](#7-rest-api-methods)
8. [Complete Working Example — Employee API](#8-complete-working-example--employee-api)
9. [How Data Flows — Visual Summary](#9-how-data-flows--visual-summary)
10. [Important Annotations Cheat Sheet](#10-important-annotations-cheat-sheet)
11. [Common Mistakes Beginners Make](#11-common-mistakes-beginners-make)
12. [Summary Table](#12-summary-table)

---

## 1. What is Spring Boot?

> **Spring Boot** is a framework built on top of Spring that helps you build **production-ready Java web applications quickly** — with minimal configuration.

### What it gives you out of the box:
- Embedded **Tomcat server** (no need to deploy WAR files)
- **Auto-configuration** (no XML config needed)
- Easy **database connection** via `application.properties`
- Built-in **dependency injection** (IoC Container)
- REST API support out of the box

### Simple Analogy
> 🏗️ Think of Spring Boot as a **ready-made building frame**.
> You don't build the foundation from scratch — you just add your rooms (Controller, Service, Repository) and it's ready to live in.

---

## 2. How Spring Boot Starts

When you run `DemoApplication.java`, this is what happens step by step:

```
You run main() method
        ↓
@SpringBootApplication triggers 3 things:
  1. @ComponentScan      → Scans all classes with @Component, @Service, @Repository, @Controller
  2. @Configuration      → Reads all config/bean definitions
  3. @EnableAutoConfiguration → Auto-configures DB, Security, etc. based on dependencies
        ↓
Reads application.properties
  → DB URL, username, password
  → Server port, app name etc.
        ↓
Connects to Database (MySQL)
        ↓
Hibernate reads all @Entity classes
  → Creates / updates tables in DB
        ↓
IoC Container creates all Beans
  → @Service, @Repository, @Controller objects created
        ↓
@Autowired dependencies injected into each class
        ↓
Embedded Tomcat Server starts on port 8080
        ↓
✅ App is READY — waiting for HTTP requests
```

### Entry Point

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

| Annotation inside `@SpringBootApplication` | What it does |
|---|---|
| `@ComponentScan` | Finds and registers all Spring components |
| `@Configuration` | Marks as a source of bean definitions |
| `@EnableAutoConfiguration` | Auto-configures everything based on classpath |

---

## 3. Project Structure

```
src/
└── main/
    ├── java/com/example/employeeapp/
    │   │
    │   ├── 📁 controller/
    │   │   └── EmployeeController.java       → Handles HTTP requests/responses
    │   │
    │   ├── 📁 service/
    │   │   ├── EmployeeService.java          → Interface (contract)
    │   │   └── EmployeeServiceImpl.java      → Implementation (business logic)
    │   │
    │   ├── 📁 repository/
    │   │   └── EmployeeRepository.java       → DB operations
    │   │
    │   ├── 📁 entity/
    │   │   └── Employee.java                 → DB table mapping
    │   │
    │   ├── 📁 dto/
    │   │   ├── EmployeeRequestDTO.java       → Data received FROM client
    │   │   └── EmployeeResponseDTO.java      → Data sent TO client
    │   │
    │   ├── 📁 mapper/
    │   │   └── EmployeeMapper.java           → Converts Entity ↔ DTO
    │   │
    │   ├── 📁 config/
    │   │   ├── AppConfig.java                → Bean definitions
    │   │   └── SecurityConfig.java           → Security rules
    │   │
    │   ├── 📁 exception/
    │   │   ├── ResourceNotFoundException.java → Custom exception
    │   │   └── GlobalExceptionHandler.java   → Handles all errors globally
    │   │
    │   └── DemoApplication.java              → Main class (entry point)
    │
    └── resources/
        └── application.properties            → DB config, server config
```

---

## 4. Layer by Layer Explanation

---

### 🟫 Layer 1 — Entity

> **Entity is the Java class that represents a table in your database.**
> Each field in the entity = one column in the DB table.

Hibernate reads the Entity class and automatically creates the table for you.

```java
package com.example.employeeapp.entity;

import jakarta.persistence.*;

@Entity                          // tells Hibernate: map this class to a DB table
@Table(name = "employees")       // name of the table in DB
public class Employee {

    @Id                                                    // primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)    // auto increment
    private Long id;

    @Column(name = "first_name", nullable = false, length = 50)
    private String firstName;

    @Column(name = "last_name", nullable = false, length = 50)
    private String lastName;

    @Column(name = "email", nullable = false, unique = true)
    private String email;

    @Column(name = "salary")
    private Double salary;

    @Column(name = "department")
    private String department;

    // ✅ Getters
    public Long getId() { return id; }
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String getEmail() { return email; }
    public Double getSalary() { return salary; }
    public String getDepartment() { return department; }

    // ✅ Setters
    public void setId(Long id) { this.id = id; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    public void setEmail(String email) { this.email = email; }
    public void setSalary(Double salary) { this.salary = salary; }
    public void setDepartment(String department) { this.department = department; }
}
```

**DB Table created by Hibernate:**

```
+-----------+-------------+------------+-------------------------+--------+------------+
|    id     | first_name  | last_name  |         email           | salary | department |
+-----------+-------------+------------+-------------------------+--------+------------+
|     1     |   Sandeep   |   Kumar    |  sandeep@gmail.com      | 50000  |    IT      |
|     2     |   Rahul     |   Sharma   |  rahul@gmail.com        | 60000  |    HR      |
+-----------+-------------+------------+-------------------------+--------+------------+
```

---

### 🟦 Layer 2 — Repository

> **Repository layer talks directly to the database.**
> You just extend `JpaRepository` and get all CRUD methods for free — no SQL needed.

```java
package com.example.employeeapp.repository;

import com.example.employeeapp.entity.Employee;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.List;
import java.util.Optional;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {

    // ✅ All these methods are available automatically (no code needed):
    // save(entity)           → INSERT or UPDATE
    // findById(id)           → SELECT WHERE id = ?
    // findAll()              → SELECT * FROM employees
    // deleteById(id)         → DELETE WHERE id = ?
    // count()                → SELECT COUNT(*)
    // existsById(id)         → returns true/false

    // ✅ Custom finder methods (Spring auto-generates SQL from method name)
    Optional<Employee> findByEmail(String email);
    List<Employee> findByDepartment(String department);
    List<Employee> findByFirstNameContaining(String keyword);
    boolean existsByEmail(String email);
}
```

> 💡 `JpaRepository<Employee, Long>` means:
> - `Employee` → the Entity class
> - `Long` → the data type of the primary key (`id`)

---

### 🟨 Layer 3 — DTO (Data Transfer Object)

> **DTO is a simple Java class used to carry data between the client and your app.**

#### Why not use Entity directly?

| Problem with using Entity directly | Solution with DTO |
|---|---|
| Exposes DB structure to client | DTO hides internal details |
| May expose sensitive fields (password) | DTO includes only safe fields |
| Client data shape ≠ DB shape | DTO shapes data exactly as needed |
| Hard to validate incoming data | DTO can have validation annotations |

```java
// DTO for data COMING FROM client (Create / Update request)
package com.example.employeeapp.dto;

public class EmployeeRequestDTO {

    private String firstName;   // client sends this
    private String lastName;
    private String email;
    private Double salary;
    private String department;

    // Getters & Setters
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String getEmail() { return email; }
    public Double getSalary() { return salary; }
    public String getDepartment() { return department; }

    public void setFirstName(String firstName) { this.firstName = firstName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    public void setEmail(String email) { this.email = email; }
    public void setSalary(Double salary) { this.salary = salary; }
    public void setDepartment(String department) { this.department = department; }
}
```

```java
// DTO for data GOING TO client (Response)
package com.example.employeeapp.dto;

public class EmployeeResponseDTO {

    private Long id;            // include id in response
    private String firstName;
    private String lastName;
    private String email;
    private Double salary;
    private String department;
    // ✅ No password, no internal fields exposed

    // Getters & Setters
    public Long getId() { return id; }
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String getEmail() { return email; }
    public Double getSalary() { return salary; }
    public String getDepartment() { return department; }

    public void setId(Long id) { this.id = id; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    public void setEmail(String email) { this.email = email; }
    public void setSalary(Double salary) { this.salary = salary; }
    public void setDepartment(String department) { this.department = department; }
}
```

---

### 🟧 Layer 4 — Mapper

> **Mapper converts Entity → DTO and DTO → Entity.**
> It is the bridge between the DB world (Entity) and the Client world (DTO).

```java
package com.example.employeeapp.mapper;

import com.example.employeeapp.dto.EmployeeRequestDTO;
import com.example.employeeapp.dto.EmployeeResponseDTO;
import com.example.employeeapp.entity.Employee;
import org.springframework.stereotype.Component;

@Component   // register as a Spring bean so it can be @Autowired
public class EmployeeMapper {

    // ✅ Convert RequestDTO → Entity  (used before saving to DB)
    public Employee toEntity(EmployeeRequestDTO dto) {
        Employee employee = new Employee();
        employee.setFirstName(dto.getFirstName());
        employee.setLastName(dto.getLastName());
        employee.setEmail(dto.getEmail());
        employee.setSalary(dto.getSalary());
        employee.setDepartment(dto.getDepartment());
        return employee;
    }

    // ✅ Convert Entity → ResponseDTO  (used before sending to client)
    public EmployeeResponseDTO toDTO(Employee employee) {
        EmployeeResponseDTO dto = new EmployeeResponseDTO();
        dto.setId(employee.getId());
        dto.setFirstName(employee.getFirstName());
        dto.setLastName(employee.getLastName());
        dto.setEmail(employee.getEmail());
        dto.setSalary(employee.getSalary());
        dto.setDepartment(employee.getDepartment());
        return dto;
    }
}
```

> 💡 **Shortcut:** In real projects, use **ModelMapper** library to avoid writing this manually:
> ```java
> // In AppConfig.java
> @Bean
> public ModelMapper modelMapper() {
>     return new ModelMapper();
> }
>
> // Usage — auto maps fields with matching names
> EmployeeResponseDTO dto = modelMapper.map(employee, EmployeeResponseDTO.class);
> Employee entity = modelMapper.map(requestDTO, Employee.class);
> ```

---

### 🟩 Layer 5 — Service

> **Service layer contains all the business logic of your application.**
> Controller calls Service. Service calls Repository. Service never talks to the client directly.

#### Step 1 — Create Interface (contract)

```java
package com.example.employeeapp.service;

import com.example.employeeapp.dto.EmployeeRequestDTO;
import com.example.employeeapp.dto.EmployeeResponseDTO;
import java.util.List;

public interface EmployeeService {
    EmployeeResponseDTO createEmployee(EmployeeRequestDTO requestDTO);
    EmployeeResponseDTO getEmployeeById(Long id);
    List<EmployeeResponseDTO> getAllEmployees();
    EmployeeResponseDTO updateEmployee(Long id, EmployeeRequestDTO requestDTO);
    void deleteEmployee(Long id);
}
```

#### Step 2 — Create Implementation

```java
package com.example.employeeapp.service;

import com.example.employeeapp.dto.EmployeeRequestDTO;
import com.example.employeeapp.dto.EmployeeResponseDTO;
import com.example.employeeapp.entity.Employee;
import com.example.employeeapp.exception.ResourceNotFoundException;
import com.example.employeeapp.mapper.EmployeeMapper;
import com.example.employeeapp.repository.EmployeeRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class EmployeeServiceImpl implements EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;  // Spring injects this

    @Autowired
    private EmployeeMapper employeeMapper;          // Spring injects this

    // ✅ CREATE
    @Override
    public EmployeeResponseDTO createEmployee(EmployeeRequestDTO requestDTO) {
        // Step 1: Check if email already exists
        if (employeeRepository.existsByEmail(requestDTO.getEmail())) {
            throw new RuntimeException("Email already exists: " + requestDTO.getEmail());
        }

        // Step 2: Convert DTO → Entity
        Employee employee = employeeMapper.toEntity(requestDTO);

        // Step 3: Save to DB
        Employee savedEmployee = employeeRepository.save(employee);

        // Step 4: Convert saved Entity → ResponseDTO and return
        return employeeMapper.toDTO(savedEmployee);
    }

    // ✅ READ BY ID
    @Override
    public EmployeeResponseDTO getEmployeeById(Long id) {
        // Find employee or throw exception if not found
        Employee employee = employeeRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Employee not found with id: " + id
            ));

        return employeeMapper.toDTO(employee);
    }

    // ✅ READ ALL
    @Override
    public List<EmployeeResponseDTO> getAllEmployees() {
        List<Employee> employees = employeeRepository.findAll();

        // Convert each Entity to DTO using stream
        return employees.stream()
            .map(employeeMapper::toDTO)
            .collect(Collectors.toList());
    }

    // ✅ UPDATE
    @Override
    public EmployeeResponseDTO updateEmployee(Long id, EmployeeRequestDTO requestDTO) {
        // Step 1: Find existing employee
        Employee existingEmployee = employeeRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Employee not found with id: " + id
            ));

        // Step 2: Update the fields
        existingEmployee.setFirstName(requestDTO.getFirstName());
        existingEmployee.setLastName(requestDTO.getLastName());
        existingEmployee.setEmail(requestDTO.getEmail());
        existingEmployee.setSalary(requestDTO.getSalary());
        existingEmployee.setDepartment(requestDTO.getDepartment());

        // Step 3: Save updated entity back to DB
        Employee updatedEmployee = employeeRepository.save(existingEmployee);

        // Step 4: Return response DTO
        return employeeMapper.toDTO(updatedEmployee);
    }

    // ✅ DELETE
    @Override
    public void deleteEmployee(Long id) {
        // Check if employee exists before deleting
        if (!employeeRepository.existsById(id)) {
            throw new ResourceNotFoundException("Employee not found with id: " + id);
        }
        employeeRepository.deleteById(id);
    }
}
```

> 💡 **Why use Interface + Impl?**
> - Follows the principle of **programming to an interface**
> - Makes it easy to **swap implementations** (e.g. different logic for different environments)
> - Makes **unit testing** easier using mocks

---

### 🟥 Layer 6 — Controller

> **Controller is the entry point of every HTTP request.**
> It receives requests from the client, calls the Service, and returns the response.
> Controller should have **NO business logic** — just receive and respond.

```java
package com.example.employeeapp.controller;

import com.example.employeeapp.dto.EmployeeRequestDTO;
import com.example.employeeapp.dto.EmployeeResponseDTO;
import com.example.employeeapp.service.EmployeeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController                        // marks as REST controller — returns JSON
@RequestMapping("/api/employees")      // base URL for all endpoints in this class
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;   // Spring injects this

    // ✅ CREATE — POST /api/employees
    @PostMapping
    public ResponseEntity<EmployeeResponseDTO> createEmployee(
            @RequestBody EmployeeRequestDTO requestDTO) {

        EmployeeResponseDTO response = employeeService.createEmployee(requestDTO);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);  // 201
    }

    // ✅ GET BY ID — GET /api/employees/1
    @GetMapping("/{id}")
    public ResponseEntity<EmployeeResponseDTO> getEmployee(
            @PathVariable Long id) {

        EmployeeResponseDTO response = employeeService.getEmployeeById(id);
        return ResponseEntity.ok(response);   // 200
    }

    // ✅ GET ALL — GET /api/employees
    @GetMapping
    public ResponseEntity<List<EmployeeResponseDTO>> getAllEmployees() {
        List<EmployeeResponseDTO> response = employeeService.getAllEmployees();
        return ResponseEntity.ok(response);   // 200
    }

    // ✅ UPDATE — PUT /api/employees/1
    @PutMapping("/{id}")
    public ResponseEntity<EmployeeResponseDTO> updateEmployee(
            @PathVariable Long id,
            @RequestBody EmployeeRequestDTO requestDTO) {

        EmployeeResponseDTO response = employeeService.updateEmployee(id, requestDTO);
        return ResponseEntity.ok(response);   // 200
    }

    // ✅ DELETE — DELETE /api/employees/1
    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteEmployee(@PathVariable Long id) {
        employeeService.deleteEmployee(id);
        return ResponseEntity.ok("Employee deleted successfully");   // 200
    }
}
```

---

### ⚙️ Layer 7 — Config

> **Config classes define beans and application-wide settings** that Spring needs to manage.

```java
package com.example.employeeapp.config;

import org.modelmapper.ModelMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration    // tells Spring: scan this class for bean definitions
public class AppConfig {

    // ✅ Register ModelMapper as a bean so it can be @Autowired anywhere
    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

```java
package com.example.employeeapp.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()    // public endpoints
                .anyRequest().authenticated()                   // all others need login
            );
        return http.build();
    }
}
```

---

### 🔴 Layer 8 — Exception Handling

> **Exception handling ensures your app returns clean, meaningful error messages** instead of ugly stack traces.

#### Step 1 — Create Custom Exception

```java
package com.example.employeeapp.exception;

public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

#### Step 2 — Create Error Response DTO

```java
package com.example.employeeapp.exception;

import java.time.LocalDateTime;

public class ErrorResponse {

    private int statusCode;
    private String message;
    private LocalDateTime timestamp;

    public ErrorResponse(int statusCode, String message) {
        this.statusCode = statusCode;
        this.message = message;
        this.timestamp = LocalDateTime.now();
    }

    // Getters
    public int getStatusCode() { return statusCode; }
    public String getMessage() { return message; }
    public LocalDateTime getTimestamp() { return timestamp; }
}
```

#### Step 3 — Create Global Exception Handler

```java
package com.example.employeeapp.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice    // intercepts exceptions from ALL controllers globally
public class GlobalExceptionHandler {

    // ✅ Handle ResourceNotFoundException (404)
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(404, ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    // ✅ Handle general RuntimeException (400)
    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<ErrorResponse> handleRuntime(RuntimeException ex) {
        ErrorResponse error = new ErrorResponse(400, ex.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    // ✅ Handle any other unexpected exception (500)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(500, "Something went wrong. Please try again.");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

**What the client gets when employee is not found:**

```json
{
  "statusCode": 404,
  "message": "Employee not found with id: 99",
  "timestamp": "2026-05-09T10:30:00"
}
```

---

## 5. application.properties

```properties
# ── App name ──────────────────────────────────────────────
spring.application.name=employee-app

# ── Server ────────────────────────────────────────────────
server.port=8080

# ── Database ──────────────────────────────────────────────
spring.datasource.url=jdbc:mysql://localhost:3306/employee_db
spring.datasource.username=root
spring.datasource.password=yourpassword

# ── Hibernate ─────────────────────────────────────────────
# update  → reuse existing table, add new columns if needed
# create  → drop & recreate table every time app starts
# none    → do nothing (use in production)
spring.jpa.hibernate.ddl-auto=update

# ── Show SQL in console (useful for learning/debugging) ───
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

---

## 6. Full Request Lifecycle

### Example: `POST /api/employees` — Create a new employee

**Client sends this JSON:**
```json
{
  "firstName": "Sandeep",
  "lastName": "Kumar",
  "email": "sandeep@gmail.com",
  "salary": 50000,
  "department": "IT"
}
```

**Step-by-step journey through the app:**

```
Step 1: Client sends POST request to /api/employees
        ↓
Step 2: Embedded Tomcat server receives the request
        ↓
Step 3: DispatcherServlet (Spring's front controller) routes
        the request to the correct Controller method
        ↓
Step 4: EmployeeController.createEmployee() is called
        @RequestBody converts JSON → EmployeeRequestDTO object
        ↓
Step 5: Controller calls employeeService.createEmployee(requestDTO)
        ↓
Step 6: Service checks: does email already exist?
        employeeRepository.existsByEmail("sandeep@gmail.com")
        → Hibernate fires: SELECT COUNT(*) FROM employees WHERE email = 'sandeep@gmail.com'
        → Returns false (doesn't exist) → proceed
        ↓
Step 7: Service calls employeeMapper.toEntity(requestDTO)
        → EmployeeRequestDTO converted to Employee entity object
        ↓
Step 8: Service calls employeeRepository.save(employee)
        → Hibernate fires:
           INSERT INTO employees (first_name, last_name, email, salary, department)
           VALUES ('Sandeep', 'Kumar', 'sandeep@gmail.com', 50000, 'IT')
        → Returns saved Employee with generated id = 1
        ↓
Step 9: Service calls employeeMapper.toDTO(savedEmployee)
        → Employee entity converted to EmployeeResponseDTO
        ↓
Step 10: Service returns EmployeeResponseDTO to Controller
        ↓
Step 11: Controller wraps it in ResponseEntity with status 201 CREATED
        ↓
Step 12: Spring converts EmployeeResponseDTO → JSON automatically
        ↓
Step 13: Response sent back to Client
```

**Client receives this JSON response:**
```json
{
  "id": 1,
  "firstName": "Sandeep",
  "lastName": "Kumar",
  "email": "sandeep@gmail.com",
  "salary": 50000,
  "department": "IT"
}
```

---

## 7. REST API Methods

| HTTP Method | URL | Action | Success Status |
|---|---|---|---|
| `POST` | `/api/employees` | Create new employee | `201 Created` |
| `GET` | `/api/employees` | Get all employees | `200 OK` |
| `GET` | `/api/employees/1` | Get employee by ID | `200 OK` |
| `PUT` | `/api/employees/1` | Update employee by ID | `200 OK` |
| `DELETE` | `/api/employees/1` | Delete employee by ID | `200 OK` |

### HTTP Status Codes

| Code | Meaning | When to use |
|---|---|---|
| `200 OK` | Success | GET, PUT, DELETE success |
| `201 Created` | Resource created | POST success |
| `400 Bad Request` | Invalid input | Validation failure |
| `404 Not Found` | Resource missing | Employee with ID not found |
| `500 Internal Server Error` | Server crashed | Unexpected error |

---

## 8. Complete Working Example — Employee API

### Dependencies in `pom.xml`

```xml
<dependencies>

    <!-- Spring Boot Web — REST APIs -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data JPA — DB operations -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- MySQL Driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- ModelMapper — auto Entity ↔ DTO conversion -->
    <dependency>
        <groupId>org.modelmapper</groupId>
        <artifactId>modelmapper</artifactId>
        <version>3.1.1</version>
    </dependency>

    <!-- Spring Boot Test — JUnit -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

---

## 9. How Data Flows — Visual Summary

```
CLIENT (Postman / Browser / React Frontend)
            │
            │  JSON Request
            ▼
┌─────────────────────────────┐
│      CONTROLLER LAYER       │  @RestController
│   EmployeeController.java   │  → Receives HTTP request
│                             │  → @RequestBody JSON → DTO
│                             │  → Calls Service
│                             │  → Returns ResponseEntity<DTO>
└────────────┬────────────────┘
             │  EmployeeRequestDTO
             ▼
┌─────────────────────────────┐
│       SERVICE LAYER         │  @Service
│   EmployeeServiceImpl.java  │  → Business logic
│                             │  → Validation (email exists?)
│                             │  → Calls Mapper: DTO → Entity
│                             │  → Calls Repository
│                             │  → Calls Mapper: Entity → DTO
└────────────┬────────────────┘
             │  Employee (Entity)
             ▼
┌─────────────────────────────┐
│      REPOSITORY LAYER       │  @Repository
│   EmployeeRepository.java   │  → save(), findById(), findAll()
│                             │  → Passes to Hibernate
└────────────┬────────────────┘
             │  SQL Query
             ▼
┌─────────────────────────────┐
│          HIBERNATE          │  ORM
│                             │  → Converts Entity → SQL
│                             │  → Executes on DB
└────────────┬────────────────┘
             │
             ▼
┌─────────────────────────────┐
│         DATABASE            │
│         (MySQL)             │  employees table
└─────────────────────────────┘
```

---

## 10. Important Annotations Cheat Sheet

### Class-Level Annotations

| Annotation | Layer | Purpose |
|---|---|---|
| `@SpringBootApplication` | Main class | Starts the Spring Boot app |
| `@RestController` | Controller | Marks as REST controller, returns JSON |
| `@RequestMapping("/path")` | Controller | Sets base URL for the controller |
| `@Service` | Service | Marks as service bean (business logic) |
| `@Repository` | Repository | Marks as repository bean (DB layer) |
| `@Entity` | Entity | Maps class to DB table |
| `@Table(name="...")` | Entity | Sets custom table name |
| `@Configuration` | Config | Marks as configuration class |
| `@RestControllerAdvice` | Exception | Handles exceptions globally |
| `@Component` | Any | Generic Spring bean |

### Method-Level Annotations

| Annotation | Purpose |
|---|---|
| `@GetMapping` | Handles HTTP GET |
| `@PostMapping` | Handles HTTP POST |
| `@PutMapping` | Handles HTTP PUT |
| `@DeleteMapping` | Handles HTTP DELETE |
| `@Bean` | Registers method return value as a Spring bean |
| `@ExceptionHandler` | Handles specific exception type |
| `@Transactional` | Wraps method in a DB transaction |
| `@Query("...")` | Custom JPQL or SQL query |

### Field/Parameter-Level Annotations

| Annotation | Purpose |
|---|---|
| `@Autowired` | Injects a Spring bean automatically |
| `@Id` | Marks primary key |
| `@GeneratedValue` | Auto-generates primary key value |
| `@Column(...)` | Configures DB column properties |
| `@RequestBody` | Converts JSON request body → Java object |
| `@PathVariable` | Reads value from URL path (`/employees/{id}`) |
| `@RequestParam` | Reads query param from URL (`?name=sandeep`) |
| `@Param("x")` | Names a JPQL query parameter |

---

## 11. Common Mistakes Beginners Make

### ❌ Mistake 1 — Writing business logic in Controller

```java
// ❌ Wrong — business logic in controller
@PostMapping
public ResponseEntity<?> createEmployee(@RequestBody Employee employee) {
    if (employeeRepository.existsByEmail(employee.getEmail())) {
        return ResponseEntity.badRequest().body("Email exists");
    }
    return ResponseEntity.ok(employeeRepository.save(employee));
}

// ✅ Correct — controller only calls service
@PostMapping
public ResponseEntity<EmployeeResponseDTO> createEmployee(@RequestBody EmployeeRequestDTO dto) {
    return ResponseEntity.status(HttpStatus.CREATED)
                         .body(employeeService.createEmployee(dto));
}
```

---

### ❌ Mistake 2 — Returning Entity directly instead of DTO

```java
// ❌ Wrong — exposes DB structure and sensitive fields
public Employee getEmployee(Long id) {
    return employeeRepository.findById(id).orElseThrow();
}

// ✅ Correct — return DTO
public EmployeeResponseDTO getEmployee(Long id) {
    Employee emp = employeeRepository.findById(id).orElseThrow();
    return employeeMapper.toDTO(emp);
}
```

---

### ❌ Mistake 3 — Not handling the case when record is not found

```java
// ❌ Wrong — will throw NoSuchElementException with ugly message
Employee emp = employeeRepository.findById(id).get();

// ✅ Correct — throw a meaningful custom exception
Employee emp = employeeRepository.findById(id)
    .orElseThrow(() -> new ResourceNotFoundException("Employee not found with id: " + id));
```

---

### ❌ Mistake 4 — Forgetting `@Transactional` with LAZY loading

```java
// ❌ Wrong — LazyInitializationException
public void getPostWithComments() {
    Post post = postRepository.findById(1L).orElseThrow();
    post.getComments();  // 💥 session already closed
}

// ✅ Correct
@Transactional
public void getPostWithComments() {
    Post post = postRepository.findById(1L).orElseThrow();
    post.getComments();  // ✅ session still open
}
```

---

### ❌ Mistake 5 — Using `spring.jpa.hibernate.ddl-auto=create` in production

```properties
# ❌ NEVER use this in production — drops and recreates all tables on startup!
spring.jpa.hibernate.ddl-auto=create

# ✅ Use this in production
spring.jpa.hibernate.ddl-auto=none

# ✅ Use this during development
spring.jpa.hibernate.ddl-auto=update
```

---

## 12. Summary Table

| Layer | File | Annotation | Job |
|---|---|---|---|
| **Entity** | `Employee.java` | `@Entity` | Represents DB table |
| **Repository** | `EmployeeRepository.java` | `@Repository` | DB CRUD operations |
| **DTO** | `EmployeeRequestDTO.java` / `EmployeeResponseDTO.java` | Plain class | Data shape for client |
| **Mapper** | `EmployeeMapper.java` | `@Component` | Entity ↔ DTO conversion |
| **Service** | `EmployeeServiceImpl.java` | `@Service` | Business logic |
| **Controller** | `EmployeeController.java` | `@RestController` | HTTP request/response |
| **Config** | `AppConfig.java` | `@Configuration` | Bean definitions |
| **Exception** | `GlobalExceptionHandler.java` | `@RestControllerAdvice` | Global error handling |

---

> ✅ **Golden Rules to Remember:**
> - **Controller** → only handles HTTP, no logic
> - **Service** → all business rules live here
> - **Repository** → only DB operations
> - **Entity** → DB shape (never send to client directly)
> - **DTO** → client shape (what client sends and receives)
> - **Mapper** → bridge between Entity and DTO
> - **Config** → beans and settings
> - **Exception** → clean error responses

---

*Notes compiled for Spring Boot beginners. Last updated: May 2026.*
