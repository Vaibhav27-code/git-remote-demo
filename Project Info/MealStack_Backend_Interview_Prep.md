# MealStack Backend - Complete Interview Preparation Guide

## 📋 TABLE OF CONTENTS
1. [Project Overview](#project-overview)
2. [Architecture & Design Patterns](#architecture--design-patterns)
3. [Backend Components - Line by Line](#backend-components)
4. [Security Implementation](#security-implementation)
5. [Database Design](#database-design)
6. [API Endpoints](#api-endpoints)
7. [Common Interview Questions](#common-interview-questions)

---

## 🎯 PROJECT OVERVIEW

### What is MealStack?
**Elevator Pitch (30 seconds):**
"MealStack is a full-stack canteen management system that digitizes food ordering in educational institutions. Students can browse daily menus, place orders using a digital wallet, and track their orders - all cashless. Admins manage inventory, set daily menus, process orders, and view analytics. Built with Spring Boot backend, React frontend, MySQL database, and secured with JWT authentication."

### Key Statistics to Mention:
- **Tech Stack:** Spring Boot 3.x, React 18, MySQL 8.0, JWT Security
- **Architecture:** Layered (Controller → Service → Repository)
- **Database:** 9 tables with proper relationships
- **Features:** User management, Inventory tracking, Order processing, Wallet system
- **Security:** JWT-based stateless authentication, BCrypt password hashing

---

## 🏗️ ARCHITECTURE & DESIGN PATTERNS

### 1. Layered Architecture (MVC Pattern)

```
┌─────────────────────────────────────┐
│      PRESENTATION LAYER             │
│    (Controllers - REST APIs)        │
├─────────────────────────────────────┤
│      BUSINESS LOGIC LAYER           │
│    (Services - Business Rules)      │
├─────────────────────────────────────┤
│      DATA ACCESS LAYER              │
│    (Repositories - DB Operations)   │
├─────────────────────────────────────┤
│      DATABASE LAYER                 │
│         (MySQL Tables)              │
└─────────────────────────────────────┘
```

**Why This Architecture?**
- **Separation of Concerns:** Each layer has single responsibility
- **Maintainability:** Easy to modify one layer without affecting others
- **Testability:** Can test each layer independently
- **Scalability:** Can scale layers independently

### 2. Design Patterns Used

#### a) **DTO Pattern (Data Transfer Object)**
**What:** Separate objects for transferring data between layers
**Why:** 
- Don't expose entity structure to client
- Customize data sent/received
- Security (hide sensitive fields)

**Example in Your Project:**
```java
// Entity (Database representation)
@Entity
public class Student {
    private Long id;
    private String password; // Don't send to client!
    private Double walletBalance;
    // ... other fields
}

// DTO (Client representation)
public class StudentDTO {
    private Long id;
    // NO password field!
    private Double walletBalance;
    // ... other fields
}
```

#### b) **Repository Pattern**
**What:** Abstract data access logic
**Why:**
- Centralize database queries
- Easy to switch database
- Cleaner code

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
    Optional<Student> findByEmail(String email);
}
```

#### c) **Dependency Injection (DI)**
**What:** Spring automatically provides required objects
**Why:**
- Loose coupling
- Easy testing (can inject mocks)
- No need for 'new' keyword

```java
@Service
public class StudentServiceImpl {
    @Autowired  // Spring injects this automatically
    private StudentRepository studentRepo;
}
```

#### d) **Builder Pattern (Lombok)**
**What:** Clean object creation
```java
Student student = Student.builder()
    .name("John")
    .email("john@example.com")
    .build();
```

---

## 💻 BACKEND COMPONENTS - LINE BY LINE EXPLANATION

### 📁 PROJECT STRUCTURE

```
backend/
├── src/main/java/com/app/
│   ├── Application.java           # Main entry point
│   ├── config/                    # Configuration classes
│   │   ├── CorsConfig.java        # Cross-Origin settings
│   │   ├── SecurityConfig.java    # Security settings
│   │   ├── SwaggerConfig.java     # API documentation
│   │   └── DataInitializer.java   # Initial data load
│   ├── controller/                # REST API endpoints
│   │   ├── StudentController.java
│   │   ├── AdminController.java
│   │   ├── OrderController.java
│   │   ├── ItemMasterController.java
│   │   ├── ItemDailyController.java
│   │   └── RechargeHistoryController.java
│   ├── service/                   # Business logic
│   │   ├── StudentService.java
│   │   ├── StudentServiceImpl.java
│   │   ├── OrderService.java
│   │   ├── OrderServiceImpl.java
│   │   └── ... (other services)
│   ├── repository/                # Database operations
│   │   ├── StudentRepository.java
│   │   ├── OrderRepository.java
│   │   └── ... (other repos)
│   ├── entities/                  # Database models
│   │   ├── Student.java
│   │   ├── Order.java
│   │   ├── ItemMaster.java
│   │   └── ... (other entities)
│   ├── dto/                       # Data Transfer Objects
│   │   ├── StudentDTO.java
│   │   ├── OrderDTO.java
│   │   └── ... (other DTOs)
│   ├── security/                  # Security components
│   │   ├── JwtUtils.java          # JWT generation/validation
│   │   ├── JwtAuthFilter.java     # Request filter
│   │   └── UserPrincipal.java     # User details
│   ├── exceptions/                # Custom exceptions
│   │   ├── ApiException.java
│   │   └── ResourceNotFoundException.java
│   ├── exc_handler/               # Exception handling
│   │   └── GlobalExceptionHandler.java
│   └── scheduler/                 # Scheduled tasks
│       └── MenuScheduler.java
└── src/main/resources/
    └── application.properties     # App configuration
```

---

## 📝 DETAILED COMPONENT BREAKDOWN

### 1️⃣ APPLICATION.JAVA (Entry Point)

```java
package com.app;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication  // Why: Combines @Configuration + @EnableAutoConfiguration + @ComponentScan
@EnableScheduling      // Why: Enable scheduled tasks (like daily menu reset)
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**Interview Questions:**
- **Q:** What does @SpringBootApplication do?
  **A:** It's a convenience annotation combining three annotations:
  - `@Configuration`: Makes class a configuration class
  - `@EnableAutoConfiguration`: Auto-configures Spring based on dependencies
  - `@ComponentScan`: Scans for Spring components in package and sub-packages

- **Q:** Why @EnableScheduling?
  **A:** To enable scheduled tasks. In our project, we have MenuScheduler that resets daily menu at midnight.

---

### 2️⃣ ENTITIES (Database Models)

#### A. USER.JAVA (Parent Entity)

```java
package com.app.entities;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "users")
@Inheritance(strategy = InheritanceType.JOINED) // Why: Single Table Inheritance
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    // Why: Auto-increment ID, DB generates value
    private Long id;
    
    @Column(nullable = false)
    // Why: Name cannot be null
    private String name;
    
    @Column(unique = true, nullable = false)
    // Why: Email must be unique for login
    private String email;
    
    @Column(nullable = false)
    // Why: Password required for authentication
    private String password;
    
    @Column(name = "mobile_no", length = 10)
    // Why: Phone number, max 10 digits
    private String mobileNo;
    
    @Enumerated(EnumType.STRING)
    // Why: Store enum as string (ADMIN/STUDENT), not ordinal (0/1)
    private Role role;
    
    @Column(name = "created_at")
    // Why: Track when user registered
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at")
    // Why: Track last profile update
    private LocalDateTime updatedAt;
    
    @PrePersist
    // Why: Called before saving new entity
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }
    
    @PreUpdate
    // Why: Called before updating entity
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

**Key Concepts:**

1. **@Entity:** Marks class as JPA entity (maps to DB table)
2. **@Table:** Specifies table name (default would be class name)
3. **@Inheritance:** Defines inheritance strategy
   - JOINED: Each class has its own table, joined by ID
   - SINGLE_TABLE: All in one table (faster, denormalized)
   - TABLE_PER_CLASS: Each concrete class has table

4. **Lombok Annotations:**
   - `@Getter/@Setter`: Auto-generates getters/setters
   - `@NoArgsConstructor`: Empty constructor
   - `@AllArgsConstructor`: Constructor with all fields
   - `@Builder`: Builder pattern for object creation

5. **@Column Attributes:**
   - `nullable = false`: NOT NULL constraint
   - `unique = true`: UNIQUE constraint
   - `length`: VARCHAR length
   - `name`: Custom column name

6. **@Enumerated:**
   - `EnumType.STRING`: Store "ADMIN" (better)
   - `EnumType.ORDINAL`: Store 0, 1 (risky if order changes)

7. **Lifecycle Callbacks:**
   - `@PrePersist`: Before INSERT
   - `@PreUpdate`: Before UPDATE
   - `@PostLoad`: After SELECT

#### B. STUDENT.JAVA (Child Entity)

```java
package com.app.entities;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDate;

@Entity
@Table(name = "students")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Student extends User {  // Why: Inherits from User
    
    @Column(name = "date_of_birth")
    private LocalDate dateOfBirth;
    
    @Enumerated(EnumType.STRING)
    private Course course;  // PGDAC, PGDBDA, etc.
    
    @Column(name = "wallet_balance", nullable = false)
    private Double walletBalance = 0.0;  // Default 0
    
    // Relationships
    @OneToMany(mappedBy = "student", cascade = CascadeType.ALL)
    // Why: One student can have many orders
    // mappedBy: Orders table has 'student' field as foreign key
    // cascade: If student deleted, delete all orders
    private List<Order> orders;
    
    @OneToMany(mappedBy = "student", cascade = CascadeType.ALL)
    private List<RechargeHistory> rechargeHistories;
}
```

**Key Concepts:**

1. **Inheritance:** Student extends User
   - Gets id, name, email, password from User
   - Adds student-specific fields

2. **@OneToMany Relationship:**
   - One student → Many orders
   - `mappedBy`: "student" field in Order entity owns the relationship
   - `cascade = ALL`: If student deleted, delete related records

3. **Default Values:** `walletBalance = 0.0` ensures new students start with 0 balance

#### C. ORDER.JAVA

```java
package com.app.entities;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "orders")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Order {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "student_id", nullable = false)
    // Why: Many orders → One student
    // LAZY: Don't load student unless accessed (performance)
    // JoinColumn: Foreign key column name
    private Student student;
    
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "item_daily_id", nullable = false)
    // Why: EAGER because we always need item details with order
    private ItemDaily itemDaily;
    
    @Column(nullable = false)
    private Integer quantity;
    
    @Column(name = "total_price", nullable = false)
    private Double totalPrice;
    
    @Column(name = "transaction_id", unique = true)
    // Why: Unique transaction ID for tracking
    private String transactionId;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status = OrderStatus.PENDING;
    
    @Column(name = "order_time")
    private LocalDateTime orderTime;
    
    @PrePersist
    protected void onCreate() {
        orderTime = LocalDateTime.now();
        // Generate unique transaction ID
        transactionId = "TXN" + System.currentTimeMillis();
    }
}
```

**Key Concepts:**

1. **@ManyToOne:** Many orders can belong to one student/item
2. **Fetch Types:**
   - `LAZY`: Load only when accessed (better performance)
   - `EAGER`: Load immediately with parent

3. **When to use LAZY vs EAGER?**
   - LAZY: When relationship not always needed
   - EAGER: When relationship always needed
   - Default: @ManyToOne = EAGER, @OneToMany = LAZY

4. **Transaction ID:** Unique identifier for each order

#### D. ITEM_MASTER.JAVA

```java
package com.app.entities;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "item_master")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ItemMaster {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    // Why: Each item name should be unique
    private String itemName;
    
    @Column(nullable = false)
    private Double price;
    
    @Enumerated(EnumType.STRING)
    private ItemCategory category;  // BREAKFAST, LUNCH, DINNER, SNACKS
    
    @Enumerated(EnumType.STRING)
    private ItemGenre genre;  // SOUTH_INDIAN, NORTH_INDIAN, ORIENTAL
    
    @Column(length = 500)
    private String imageUrl;
    
    @Column(columnDefinition = "TEXT")
    // Why: TEXT for long descriptions
    private String description;
}
```

**Key Concepts:**

1. **Master Data:** Template for items (like menu card)
2. **@Column(columnDefinition):** Custom SQL type
3. **Unique Constraint:** No duplicate item names

#### E. ITEM_DAILY.JAVA

```java
package com.app.entities;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDate;

@Entity
@Table(name = "item_daily",
    uniqueConstraints = @UniqueConstraint(columnNames = {"date", "item_master_id"})
)
// Why: Same item can appear only once per day
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ItemDaily {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "item_master_id", nullable = false)
    // Why: Daily item references master item
    private ItemMaster itemMaster;
    
    @Column(nullable = false)
    private LocalDate date;
    
    @Column(name = "initial_quantity", nullable = false)
    // Why: Starting quantity for the day
    private Integer initialQuantity;
    
    @Column(name = "available_quantity", nullable = false)
    // Why: Current available quantity (decreases with orders)
    private Integer availableQuantity;
    
    @Column(name = "sold_quantity", nullable = false)
    // Why: Track how much sold
    private Integer soldQuantity = 0;
    
    // Business logic method
    public boolean hasStock() {
        return availableQuantity > 0;
    }
    
    public void reduceStock(int quantity) {
        if (quantity > availableQuantity) {
            throw new IllegalStateException("Insufficient stock");
        }
        this.availableQuantity -= quantity;
        this.soldQuantity += quantity;
    }
}
```

**Key Concepts:**

1. **Composite Unique Constraint:** (date + item_master_id) must be unique
   - Same item can't be added twice on same day
   
2. **Inventory Tracking:**
   - `initialQuantity`: Set by admin at start of day
   - `availableQuantity`: Decreases with each order
   - `soldQuantity`: Increases with each order

3. **Business Methods in Entity:**
   - `hasStock()`: Check if item available
   - `reduceStock()`: Decrease quantity when ordered

---

### 3️⃣ REPOSITORIES (Data Access Layer)

```java
package com.app.repository;

import com.app.entities.Student;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.Optional;

@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    
    // Spring Data JPA auto-generates implementation!
    
    Optional<Student> findByEmail(String email);
    // Why Optional: Email might not exist, avoid null pointer
    // Generated SQL: SELECT * FROM students WHERE email = ?
    
    boolean existsByEmail(String email);
    // Generated SQL: SELECT COUNT(*) > 0 FROM students WHERE email = ?
    
    List<Student> findByCourse(Course course);
    // Generated SQL: SELECT * FROM students WHERE course = ?
    
    @Query("SELECT s FROM Student s WHERE s.walletBalance < :minBalance")
    List<Student> findLowBalanceStudents(@Param("minBalance") Double minBalance);
    // Why @Query: Custom JPQL query
}
```

**Interview Questions:**

**Q:** What is JpaRepository?
**A:** Interface providing CRUD operations:
- `save(entity)`: Insert/Update
- `findById(id)`: Select by ID
- `findAll()`: Select all
- `deleteById(id)`: Delete
- `count()`: Count records

**Q:** How does Spring Data JPA generate queries from method names?
**A:** It parses method name:
- `findBy` → SELECT
- `Email` → WHERE email = ?
- `And` → AND
- `Or` → OR
- Example: `findByEmailAndCourse` → WHERE email = ? AND course = ?

**Q:** Difference between JPQL and SQL?
**A:**
- **JPQL:** Query on entities (object-oriented)
  ```sql
  SELECT s FROM Student s WHERE s.email = :email
  ```
- **SQL:** Query on tables
  ```sql
  SELECT * FROM students WHERE email = ?
  ```

---

### 4️⃣ SERVICES (Business Logic Layer)

#### STUDENT_SERVICE_IMPL.JAVA

```java
package com.app.service;

import com.app.dto.*;
import com.app.entities.*;
import com.app.repository.*;
import com.app.exceptions.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service  // Why: Spring manages this as service bean
@Transactional  // Why: All methods run in transaction (rollback on error)
public class StudentServiceImpl implements StudentService {
    
    @Autowired
    private StudentRepository studentRepo;
    
    @Autowired
    private UserRepository userRepo;
    
    @Autowired
    private PasswordEncoder passwordEncoder;  // BCrypt encoder
    
    @Override
    public StudentDTO registerStudent(RegisterStudentDTO dto) {
        // 1. Validate email uniqueness
        if (userRepo.existsByEmail(dto.getEmail())) {
            throw new ApiException("Email already registered!");
        }
        
        // 2. Build student entity
        Student student = Student.builder()
            .name(dto.getName())
            .email(dto.getEmail())
            .password(passwordEncoder.encode(dto.getPassword()))  // Hash password!
            .mobileNo(dto.getMobileNo())
            .dateOfBirth(dto.getDateOfBirth())
            .course(dto.getCourse())
            .role(Role.STUDENT)
            .walletBalance(0.0)  // Start with 0 balance
            .build();
        
        // 3. Save to database
        Student saved = studentRepo.save(student);
        
        // 4. Convert to DTO (don't send password!)
        return convertToDTO(saved);
    }
    
    @Override
    public StudentDTO getStudentProfile(Long studentId) {
        Student student = studentRepo.findById(studentId)
            .orElseThrow(() -> new ResourceNotFoundException("Student not found"));
        
        return convertToDTO(student);
    }
    
    @Override
    public StudentDTO updateProfile(Long studentId, StudentDTO dto) {
        Student student = studentRepo.findById(studentId)
            .orElseThrow(() -> new ResourceNotFoundException("Student not found"));
        
        // Update allowed fields only
        student.setName(dto.getName());
        student.setMobileNo(dto.getMobileNo());
        student.setDateOfBirth(dto.getDateOfBirth());
        
        Student updated = studentRepo.save(student);
        return convertToDTO(updated);
    }
    
    @Override
    public void updatePassword(Long studentId, UpdatePasswordDTO dto) {
        Student student = studentRepo.findById(studentId)
            .orElseThrow(() -> new ResourceNotFoundException("Student not found"));
        
        // Verify old password
        if (!passwordEncoder.matches(dto.getOldPassword(), student.getPassword())) {
            throw new ApiException("Old password is incorrect");
        }
        
        // Set new password (hashed)
        student.setPassword(passwordEncoder.encode(dto.getNewPassword()));
        studentRepo.save(student);
    }
    
    @Override
    @Transactional  // Important: Wallet update must be atomic
    public void rechargeWallet(Long studentId, Double amount) {
        if (amount <= 0) {
            throw new ApiException("Amount must be positive");
        }
        
        Student student = studentRepo.findById(studentId)
            .orElseThrow(() -> new ResourceNotFoundException("Student not found"));
        
        // Update balance
        student.setWalletBalance(student.getWalletBalance() + amount);
        studentRepo.save(student);
        
        // Create recharge history
        RechargeHistory history = RechargeHistory.builder()
            .student(student)
            .amount(amount)
            .transactionId("RECH" + System.currentTimeMillis())
            .build();
        
        rechargeHistoryRepo.save(history);
    }
    
    // Helper method: Entity to DTO conversion
    private StudentDTO convertToDTO(Student student) {
        return StudentDTO.builder()
            .id(student.getId())
            .name(student.getName())
            .email(student.getEmail())
            // NO password field!
            .mobileNo(student.getMobileNo())
            .dateOfBirth(student.getDateOfBirth())
            .course(student.getCourse())
            .walletBalance(student.getWalletBalance())
            .build();
    }
}
```

**Key Concepts:**

1. **@Transactional:**
   - Wraps method in database transaction
   - If exception occurs, rollback all changes
   - Example: rechargeWallet updates balance + creates history (both or none)

2. **Password Encoding:**
   ```java
   // Encoding (registration)
   String encoded = passwordEncoder.encode("password123");
   // Result: $2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM...
   
   // Matching (login)
   boolean matches = passwordEncoder.matches("password123", encoded);
   ```

3. **Exception Handling:**
   - `throw new ApiException()`: Custom business logic errors
   - `throw new ResourceNotFoundException()`: Entity not found
   - Caught by GlobalExceptionHandler

4. **DTO Conversion:**
   - Never expose entity directly to client
   - Remove sensitive fields (password)
   - Custom field names/formats

---

#### ORDER_SERVICE_IMPL.JAVA

```java
package com.app.service;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class OrderServiceImpl implements OrderService {
    
    @Autowired
    private OrderRepository orderRepo;
    
    @Autowired
    private StudentRepository studentRepo;
    
    @Autowired
    private ItemDailyRepository itemDailyRepo;
    
    @Override
    @Transactional(isolation = Isolation.SERIALIZABLE)
    // Why SERIALIZABLE: Prevent concurrent order issues
    public OrderDTO placeOrder(PlaceOrderRequest request) {
        
        // 1. Fetch student
        Student student = studentRepo.findById(request.getStudentId())
            .orElseThrow(() -> new ResourceNotFoundException("Student not found"));
        
        // 2. Fetch item
        ItemDaily itemDaily = itemDailyRepo.findById(request.getItemDailyId())
            .orElseThrow(() -> new ResourceNotFoundException("Item not found"));
        
        // 3. Check stock availability
        if (!itemDaily.hasStock() || itemDaily.getAvailableQuantity() < request.getQuantity()) {
            throw new ApiException("Item out of stock");
        }
        
        // 4. Calculate total price
        Double totalPrice = itemDaily.getItemMaster().getPrice() * request.getQuantity();
        
        // 5. Check wallet balance
        if (student.getWalletBalance() < totalPrice) {
            throw new ApiException("Insufficient wallet balance");
        }
        
        // 6. Deduct balance (atomic operation)
        student.setWalletBalance(student.getWalletBalance() - totalPrice);
        studentRepo.save(student);
        
        // 7. Reduce stock (atomic operation)
        itemDaily.reduceStock(request.getQuantity());
        itemDailyRepo.save(itemDaily);
        
        // 8. Create order
        Order order = Order.builder()
            .student(student)
            .itemDaily(itemDaily)
            .quantity(request.getQuantity())
            .totalPrice(totalPrice)
            .status(OrderStatus.PENDING)
            .build();
        
        Order savedOrder = orderRepo.save(order);
        
        // 9. Return DTO
        return convertToDTO(savedOrder);
    }
    
    @Override
    public List<OrderDTO> getStudentOrders(Long studentId) {
        List<Order> orders = orderRepo.findByStudentIdOrderByOrderTimeDesc(studentId);
        return orders.stream()
            .map(this::convertToDTO)
            .collect(Collectors.toList());
    }
    
    @Override
    public List<OrderDTO> getPendingOrders() {
        List<Order> orders = orderRepo.findByStatus(OrderStatus.PENDING);
        return orders.stream()
            .map(this::convertToDTO)
            .collect(Collectors.toList());
    }
    
    @Override
    public OrderDTO markAsServed(Long orderId) {
        Order order = orderRepo.findById(orderId)
            .orElseThrow(() -> new ResourceNotFoundException("Order not found"));
        
        order.setStatus(OrderStatus.SERVED);
        Order updated = orderRepo.save(order);
        
        return convertToDTO(updated);
    }
    
    private OrderDTO convertToDTO(Order order) {
        return OrderDTO.builder()
            .id(order.getId())
            .studentName(order.getStudent().getName())
            .itemName(order.getItemDaily().getItemMaster().getItemName())
            .quantity(order.getQuantity())
            .totalPrice(order.getTotalPrice())
            .transactionId(order.getTransactionId())
            .status(order.getStatus())
            .orderTime(order.getOrderTime())
            .build();
    }
}
```

**Key Concepts:**

1. **Transaction Isolation Levels:**
   ```
   READ_UNCOMMITTED → Can read uncommitted changes (dirty read)
   READ_COMMITTED   → Can read committed changes only
   REPEATABLE_READ  → Same read in transaction gives same result
   SERIALIZABLE     → Complete isolation (slowest, safest)
   ```
   
   **Why SERIALIZABLE for placeOrder?**
   - Scenario: Two students ordering last item simultaneously
   - Without SERIALIZABLE: Both might succeed (overselling)
   - With SERIALIZABLE: Second student gets "out of stock" error

2. **Atomic Operations:**
   ```
   Step 1: Check balance ✓
   Step 2: Deduct balance ✓
   Step 3: Reduce stock ✓
   Step 4: Create order ✓
   
   If ANY step fails → ROLLBACK all steps
   ```

3. **Stream API:**
   ```java
   orders.stream()
       .map(this::convertToDTO)      // Transform each order to DTO
       .collect(Collectors.toList()); // Collect to list
   ```

---

### 5️⃣ CONTROLLERS (REST API Layer)

#### STUDENT_CONTROLLER.JAVA

```java
package com.app.controller;

import com.app.dto.*;
import com.app.service.StudentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

import jakarta.validation.Valid;

@RestController  // Why: Combines @Controller + @ResponseBody
@RequestMapping("/api/students")  // Base path for all endpoints
@CrossOrigin(origins = "http://localhost:3000")  // Allow React frontend
public class StudentController {
    
    @Autowired
    private StudentService studentService;
    
    // POST /api/students/register
    @PostMapping("/register")
    public ResponseEntity<ApiResponse> registerStudent(
        @Valid @RequestBody RegisterStudentDTO dto
    ) {
        // @Valid: Trigger validation annotations
        // @RequestBody: Parse JSON to DTO
        
        StudentDTO student = studentService.registerStudent(dto);
        
        return ResponseEntity.ok(
            new ApiResponse("Registration successful", student)
        );
    }
    
    // POST /api/students/login
    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(
        @Valid @RequestBody SignInDTO dto
    ) {
        AuthResponse response = studentService.authenticate(dto);
        return ResponseEntity.ok(response);
    }
    
    // GET /api/students/profile/{id}
    @GetMapping("/profile/{id}")
    @PreAuthorize("hasRole('STUDENT')")  // Only students can access
    public ResponseEntity<StudentDTO> getProfile(@PathVariable Long id) {
        // @PathVariable: Extract {id} from URL
        
        StudentDTO student = studentService.getStudentProfile(id);
        return ResponseEntity.ok(student);
    }
    
    // PUT /api/students/profile/{id}
    @PutMapping("/profile/{id}")
    @PreAuthorize("hasRole('STUDENT')")
    public ResponseEntity<ApiResponse> updateProfile(
        @PathVariable Long id,
        @Valid @RequestBody StudentDTO dto
    ) {
        StudentDTO updated = studentService.updateProfile(id, dto);
        return ResponseEntity.ok(
            new ApiResponse("Profile updated", updated)
        );
    }
    
    // POST /api/students/wallet/recharge
    @PostMapping("/wallet/recharge")
    @PreAuthorize("hasRole('STUDENT')")
    public ResponseEntity<ApiResponse> rechargeWallet(
        @RequestParam Long studentId,
        @RequestParam Double amount
    ) {
        // @RequestParam: Extract query parameters
        // URL: /wallet/recharge?studentId=1&amount=500
        
        studentService.rechargeWallet(studentId, amount);
        return ResponseEntity.ok(
            new ApiResponse("Wallet recharged successfully", null)
        );
    }
    
    // GET /api/students/wallet/balance/{id}
    @GetMapping("/wallet/balance/{id}")
    @PreAuthorize("hasRole('STUDENT')")
    public ResponseEntity<Double> getWalletBalance(@PathVariable Long id) {
        Double balance = studentService.getWalletBalance(id);
        return ResponseEntity.ok(balance);
    }
}
```

**Key Annotations:**

1. **@RestController:**
   - Combines `@Controller` + `@ResponseBody`
   - Automatically converts return value to JSON

2. **@RequestMapping:**
   - Base URL for all endpoints in controller
   - Example: `/api/students`

3. **HTTP Method Annotations:**
   ```
   @GetMapping    → HTTP GET (read)
   @PostMapping   → HTTP POST (create)
   @PutMapping    → HTTP PUT (update)
   @DeleteMapping → HTTP DELETE (delete)
   @PatchMapping  → HTTP PATCH (partial update)
   ```

4. **@PathVariable vs @RequestParam:**
   ```
   @PathVariable:  /students/{id}    → /students/5
   @RequestParam:  /students?id=5    → /students?id=5
   ```

5. **@Valid:**
   - Triggers validation on DTO fields
   - Example:
   ```java
   public class RegisterStudentDTO {
       @NotBlank(message = "Name required")
       private String name;
       
       @Email(message = "Invalid email")
       private String email;
       
       @Size(min = 8, message = "Password min 8 chars")
       private String password;
   }
   ```

6. **@PreAuthorize:**
   - Security check before method execution
   - Examples:
   ```java
   @PreAuthorize("hasRole('ADMIN')")
   @PreAuthorize("hasRole('STUDENT')")
   @PreAuthorize("hasAnyRole('ADMIN', 'STUDENT')")
   @PreAuthorize("#id == authentication.principal.id")  // Own profile only
   ```

7. **ResponseEntity:**
   ```java
   ResponseEntity.ok(data)              // 200 OK
   ResponseEntity.status(201).body(data) // 201 Created
   ResponseEntity.badRequest().build()   // 400 Bad Request
   ResponseEntity.notFound().build()     // 404 Not Found
   ```

---

#### ORDER_CONTROLLER.JAVA

```java
package com.app.controller;

import com.app.dto.*;
import com.app.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

import jakarta.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/orders")
@CrossOrigin(origins = "http://localhost:3000")
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    // POST /api/orders/place
    @PostMapping("/place")
    @PreAuthorize("hasRole('STUDENT')")
    public ResponseEntity<ApiResponse> placeOrder(
        @Valid @RequestBody PlaceOrderRequest request
    ) {
        try {
            OrderDTO order = orderService.placeOrder(request);
            return ResponseEntity
                .status(HttpStatus.CREATED)  // 201 Created
                .body(new ApiResponse("Order placed successfully", order));
        } catch (ApiException e) {
            return ResponseEntity
                .badRequest()
                .body(new ApiResponse(e.getMessage(), null));
        }
    }
    
    // GET /api/orders/student/{studentId}
    @GetMapping("/student/{studentId}")
    @PreAuthorize("hasRole('STUDENT')")
    public ResponseEntity<List<OrderDTO>> getStudentOrders(
        @PathVariable Long studentId
    ) {
        List<OrderDTO> orders = orderService.getStudentOrders(studentId);
        return ResponseEntity.ok(orders);
    }
    
    // GET /api/orders/pending (Admin only)
    @GetMapping("/pending")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<List<OrderDTO>> getPendingOrders() {
        List<OrderDTO> orders = orderService.getPendingOrders();
        return ResponseEntity.ok(orders);
    }
    
    // PUT /api/orders/{orderId}/serve (Admin only)
    @PutMapping("/{orderId}/serve")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<ApiResponse> markAsServed(@PathVariable Long orderId) {
        OrderDTO order = orderService.markAsServed(orderId);
        return ResponseEntity.ok(
            new ApiResponse("Order marked as served", order)
        );
    }
    
    // GET /api/orders/history/{studentId}
    @GetMapping("/history/{studentId}")
    @PreAuthorize("hasRole('STUDENT')")
    public ResponseEntity<List<OrderDTO>> getOrderHistory(
        @PathVariable Long studentId,
        @RequestParam(required = false) String status
    ) {
        // Optional query param: /history/5?status=SERVED
        
        List<OrderDTO> orders;
        if (status != null) {
            orders = orderService.getOrdersByStatus(studentId, status);
        } else {
            orders = orderService.getStudentOrders(studentId);
        }
        return ResponseEntity.ok(orders);
    }
}
```

---

### 6️⃣ SECURITY CONFIGURATION

#### SECURITY_CONFIG.JAVA

```java
package com.app.config;

import com.app.security.JwtAuthFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableMethodSecurity(prePostEnabled = true)
// Why: Enable @PreAuthorize annotations
public class SecurityConfig {
    
    @Autowired
    private JwtAuthFilter jwtAuthFilter;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF (not needed for stateless JWT)
            .csrf(csrf -> csrf.disable())
            
            // Configure authorization
            .authorizeHttpRequests(auth -> auth
                // Public endpoints (no authentication)
                .requestMatchers(
                    "/api/students/register",
                    "/api/students/login",
                    "/api/admin/login",
                    "/swagger-ui/**",
                    "/v3/api-docs/**"
                ).permitAll()
                
                // All other requests need authentication
                .anyRequest().authenticated()
            )
            
            // Stateless session (no session, JWT only)
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            
            // Add JWT filter before UsernamePasswordAuthenticationFilter
            .addFilterBefore(
                jwtAuthFilter, 
                UsernamePasswordAuthenticationFilter.class
            );
        
        return http.build();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
        // Why BCrypt:
        // - Slow algorithm (prevents brute force)
        // - Automatic salt generation
        // - Industry standard
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
        AuthenticationConfiguration config
    ) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

**Key Concepts:**

1. **CSRF (Cross-Site Request Forgery):**
   - Attack: Trick user into submitting malicious request
   - Solution: CSRF token
   - Why disabled: Stateless JWT (token in header, not cookie)

2. **Session Management:**
   ```
   STATEFUL (Session):
   1. Login → Server creates session
   2. Server returns session ID (cookie)
   3. Client sends session ID with each request
   4. Server looks up session
   
   STATELESS (JWT):
   1. Login → Server creates JWT
   2. Client stores JWT (localStorage)
   3. Client sends JWT with each request
   4. Server validates JWT (no database lookup)
   ```

3. **Filter Chain:**
   ```
   Request → JwtAuthFilter → UsernamePasswordAuthFilter → Controller
   ```

4. **BCrypt Password Encoding:**
   ```java
   String password = "myPassword123";
   String encoded = encoder.encode(password);
   // Result: $2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy
   //         ─┬─ ─┬─ ──────────────┬──────────── ──────────────┬─────────
   //          │   │               │                            │
   //       Algorithm │            Salt                        Hash
   //            Cost Factor
   ```

---

#### JWT_AUTH_FILTER.JAVA

```java
package com.app.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
public class JwtAuthFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtUtils jwtUtils;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {
        
        // 1. Extract JWT from header
        String authHeader = request.getHeader("Authorization");
        // Format: "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
        
        String token = null;
        String username = null;
        
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            token = authHeader.substring(7);  // Remove "Bearer "
            username = jwtUtils.extractUsername(token);
        }
        
        // 2. Validate token and set authentication
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            
            // Load user details from database
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            
            // Validate token
            if (jwtUtils.validateToken(token, userDetails)) {
                
                // Create authentication object
                UsernamePasswordAuthenticationToken authToken = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,  // credentials (password) not needed
                        userDetails.getAuthorities()  // roles
                    );
                
                authToken.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request)
                );
                
                // Set authentication in SecurityContext
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        
        // 3. Continue filter chain
        filterChain.doFilter(request, response);
    }
}
```

**Flow:**
```
1. Client Request:
   GET /api/students/profile/5
   Headers: {
     "Authorization": "Bearer eyJhbGc..."
   }

2. JwtAuthFilter:
   - Extract token
   - Validate token
   - Load user details
   - Set authentication

3. Controller:
   - @PreAuthorize checks authentication
   - Execute method

4. Response:
   - Return data
```

---

#### JWT_UTILS.JAVA

```java
package com.app.security;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@Component
public class JwtUtils {
    
    @Value("${jwt.secret}")
    private String secret;  // From application.properties
    
    @Value("${jwt.expiration}")
    private Long expiration;  // 24 hours = 86400000 ms
    
    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(secret.getBytes());
    }
    
    // Generate JWT token
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("role", userDetails.getAuthorities());
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())  // email
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }
    
    // Extract username from token
    public String extractUsername(String token) {
        return extractClaims(token).getSubject();
    }
    
    // Extract expiration date
    public Date extractExpiration(String token) {
        return extractClaims(token).getExpiration();
    }
    
    // Extract all claims
    private Claims extractClaims(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
    
    // Check if token expired
    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }
    
    // Validate token
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (
            username.equals(userDetails.getUsername()) && 
            !isTokenExpired(token)
        );
    }
}
```

**JWT Structure:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2huQGV4YW1wbGUuY29tIiwicm9sZSI6IlNUVURFTlQiLCJpYXQiOjE3MDcyMDAwMDAsImV4cCI6MTcwNzI4NjQwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
────────────────────────────┬─────────────────────  ───────────────────────────────────────────────────────────┬───────────────────────────────  ──────────────────────────┬─────────────────────────
         Header (Base64)                                            Payload (Base64)                                              Signature (HMAC-SHA256)

Header:                      Payload:                              Signature:
{                            {                                     HMACSHA256(
  "alg": "HS256",              "sub": "john@example.com",            base64UrlEncode(header) + "." +
  "typ": "JWT"                 "role": "STUDENT",                    base64UrlEncode(payload),
}                              "iat": 1707200000,                    secret
                               "exp": 1707286400                   )
                             }
```

---

### 7️⃣ EXCEPTION HANDLING

#### GLOBAL_EXCEPTION_HANDLER.JAVA

```java
package com.app.exc_handler;

import com.app.dto.ApiResponse;
import com.app.exceptions.ApiException;
import com.app.exceptions.ResourceNotFoundException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice  // Global exception handler for all controllers
public class GlobalExceptionHandler {
    
    // Handle custom API exceptions
    @ExceptionHandler(ApiException.class)
    public ResponseEntity<ApiResponse> handleApiException(ApiException ex) {
        return ResponseEntity
            .badRequest()
            .body(new ApiResponse(ex.getMessage(), null));
    }
    
    // Handle resource not found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiResponse> handleResourceNotFoundException(
        ResourceNotFoundException ex
    ) {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(new ApiResponse(ex.getMessage(), null));
    }
    
    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();
        
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        return ResponseEntity
            .badRequest()
            .body(errors);
    }
    
    // Handle all other exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse> handleGlobalException(Exception ex) {
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(new ApiResponse("An error occurred: " + ex.getMessage(), null));
    }
}
```

**Example Error Responses:**

1. **Validation Error:**
```json
{
  "name": "Name is required",
  "email": "Invalid email format",
  "password": "Password must be at least 8 characters"
}
```

2. **Custom Exception:**
```json
{
  "message": "Insufficient wallet balance",
  "data": null
}
```

3. **Resource Not Found:**
```json
{
  "message": "Student with ID 999 not found",
  "data": null
}
```

---

### 8️⃣ CONFIGURATION FILES

#### APPLICATION.PROPERTIES

```properties
# Server Configuration
server.port=8080
server.servlet.context-path=/

# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/mealstack_db
# Why: JDBC URL format - jdbc:mysql://host:port/database
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate Configuration
spring.jpa.hibernate.ddl-auto=update
# Why: Auto-create/update tables from entities
# Options:
# - create: Drop and recreate tables (lose data)
# - update: Update schema without losing data
# - validate: Only validate schema
# - none: No automatic schema management

spring.jpa.show-sql=true
# Why: Print SQL queries in console (debugging)

spring.jpa.properties.hibernate.format_sql=true
# Why: Format SQL for readability

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
# Why: SQL dialect for MySQL 8

# JWT Configuration
jwt.secret=MySecretKeyForJWTTokenGenerationAndValidation12345678
# Why: Secret key for signing JWT (keep this secure!)
# In production: Use environment variable

jwt.expiration=86400000
# Why: Token expiry time in milliseconds (24 hours)

# File Upload Configuration
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
# Why: Limit file upload size

# Logging Configuration
logging.level.root=INFO
logging.level.com.app=DEBUG
# Why: Set log levels for debugging

# CORS Configuration
cors.allowed-origins=http://localhost:3000
# Why: Allow React frontend to call APIs
```

---

### 9️⃣ SCHEDULER

#### MENU_SCHEDULER.JAVA

```java
package com.app.scheduler;

import com.app.repository.ItemDailyRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;

@Component
public class MenuScheduler {
    
    @Autowired
    private ItemDailyRepository itemDailyRepo;
    
    // Run every day at midnight (00:00:00)
    @Scheduled(cron = "0 0 0 * * *")
    @Transactional
    public void resetDailyMenu() {
        // Delete yesterday's menu items
        LocalDate yesterday = LocalDate.now().minusDays(1);
        itemDailyRepo.deleteByDate(yesterday);
        
        System.out.println("Daily menu reset completed for: " + yesterday);
    }
    
    // Alternative: Run every 24 hours
    // @Scheduled(fixedRate = 86400000)  // 24 hours in milliseconds
    
    // Alternative: Run 5 minutes after app startup, then every 24 hours
    // @Scheduled(initialDelay = 300000, fixedRate = 86400000)
}
```

**Cron Expression Explained:**
```
@Scheduled(cron = "0 0 0 * * *")
                  │ │ │ │ │ │
                  │ │ │ │ │ └─── Day of week (0-7, 0=Sunday)
                  │ │ │ │ └───── Month (1-12)
                  │ │ │ └─────── Day of month (1-31)
                  │ │ └───────── Hour (0-23)
                  │ └─────────── Minute (0-59)
                  └───────────── Second (0-59)

Examples:
"0 0 0 * * *"     → Every day at midnight
"0 0 12 * * *"    → Every day at noon
"0 0 9-17 * * *"  → Every hour from 9 AM to 5 PM
"0 */15 * * * *"  → Every 15 minutes
"0 0 0 1 * *"     → First day of every month
```

---

## 🔒 SECURITY IMPLEMENTATION - DEEP DIVE

### 1. Authentication Flow

```
┌─────────────┐
│   Client    │
│   (React)   │
└──────┬──────┘
       │
       │ POST /api/students/login
       │ { email, password }
       ▼
┌─────────────────────────────────┐
│   StudentController             │
│   - Receives login request      │
└──────┬──────────────────────────┘
       │
       │ Call authenticate()
       ▼
┌─────────────────────────────────┐
│   StudentService                │
│   1. Find user by email         │
│   2. Verify password (BCrypt)   │
│   3. Generate JWT token         │
└──────┬──────────────────────────┘
       │
       │ Return AuthResponse
       │ { token, user info }
       ▼
┌─────────────┐
│   Client    │
│   Store JWT │
│   in localStorage │
└─────────────┘
```

### 2. Authorization Flow

```
┌─────────────┐
│   Client    │
│   Send JWT  │
└──────┬──────┘
       │
       │ GET /api/students/profile/5
       │ Headers: { Authorization: "Bearer <JWT>" }
       ▼
┌─────────────────────────────────┐
│   JwtAuthFilter                 │
│   1. Extract JWT from header    │
│   2. Validate JWT               │
│   3. Load user details          │
│   4. Set SecurityContext        │
└──────┬──────────────────────────┘
       │
       │ Authentication set
       ▼
┌─────────────────────────────────┐
│   StudentController             │
│   @PreAuthorize("hasRole...")   │
│   - Check if user has role      │
│   - Execute method              │
└──────┬──────────────────────────┘
       │
       │ Return response
       ▼
┌─────────────┐
│   Client    │
│   Display   │
│   data      │
└─────────────┘
```

### 3. Password Security

**BCrypt Algorithm:**
```java
// During Registration
String plainPassword = "myPassword123";
String hashedPassword = passwordEncoder.encode(plainPassword);
// Result: $2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM...

// During Login
boolean matches = passwordEncoder.matches(
    "myPassword123",          // User input
    hashedPassword            // From database
);
```

**Why BCrypt?**
1. **Slow by design:** Prevents brute force attacks
2. **Automatic salt:** Each password has unique salt
3. **Adaptive:** Cost factor can be increased over time
4. **One-way:** Cannot decrypt hash to get original password

---

## 📊 DATABASE DESIGN - DEEP DIVE

### Entity Relationships

```
┌──────────┐
│   User   │  (Parent)
└────┬─────┘
     │ Inheritance (JOINED)
     │
     ├─────────────┬─────────────┐
     │             │             │
┌────▼─────┐  ┌───▼────┐    ┌───▼────┐
│ Student  │  │ Admin  │    │ (Other)│
└────┬─────┘  └────────┘    └────────┘
     │
     │ 1:N
     │
┌────▼──────────────┐
│    Order          │
│ - student_id (FK) │
│ - item_daily_id   │
└────┬──────────────┘
     │
     │ N:1
     │
┌────▼──────────────┐
│   ItemDaily       │
│ - item_master_id  │
│ - date            │
└────┬──────────────┘
     │
     │ N:1
     │
┌────▼──────────────┐
│   ItemMaster      │
│ - itemName        │
│ - price           │
└───────────────────┘
```

### SQL Queries Generated

1. **Find student by email:**
```sql
SELECT s.*, u.* 
FROM students s 
INNER JOIN users u ON s.id = u.id 
WHERE u.email = ?
```

2. **Place order (atomic transaction):**
```sql
BEGIN TRANSACTION;

-- Check wallet balance
SELECT wallet_balance FROM students WHERE id = ?;

-- Check stock
SELECT available_quantity FROM item_daily WHERE id = ?;

-- Deduct balance
UPDATE students 
SET wallet_balance = wallet_balance - ? 
WHERE id = ?;

-- Reduce stock
UPDATE item_daily 
SET available_quantity = available_quantity - ?,
    sold_quantity = sold_quantity + ?
WHERE id = ?;

-- Create order
INSERT INTO orders (student_id, item_daily_id, quantity, total_price, status)
VALUES (?, ?, ?, ?, 'PENDING');

COMMIT;
```

3. **Get student orders with item details:**
```sql
SELECT o.*, 
       s.name as student_name,
       im.item_name,
       im.price
FROM orders o
INNER JOIN students s ON o.student_id = s.id
INNER JOIN item_daily id ON o.item_daily_id = id.id
INNER JOIN item_master im ON id.item_master_id = im.id
WHERE o.student_id = ?
ORDER BY o.order_time DESC;
```

---

## 🎤 COMMON INTERVIEW QUESTIONS & ANSWERS

### Basic Questions

**Q1: Explain your project in 2 minutes.**

**A:** "MealStack is a canteen management system I developed during my PG-DAC course. It addresses the problem of manual food ordering and cash handling in educational institutions.

The system has two main users: Students and Admins. Students can browse the daily menu, check item availability in real-time, place orders using their digital wallet, and track order history. Admins manage the item catalog, set daily menus with quantities, process orders, and view analytics.

On the technical side, I used Spring Boot for the backend with a layered architecture - Controller, Service, and Repository layers. The frontend is React with Redux for state management. For security, I implemented JWT-based authentication with BCrypt password hashing.

The database is MySQL with 9 tables handling users, students, orders, items, and transactions. I used JPA and Hibernate for ORM, which simplified database operations.

Key features include real-time inventory tracking, atomic order transactions to prevent overselling, and a scheduled task that resets the daily menu at midnight. The system is fully cashless, reducing manual errors and improving operational efficiency."

---

**Q2: Why did you use layered architecture?**

**A:** "I used layered architecture for several reasons:

1. **Separation of Concerns:** Each layer has a single responsibility:
   - Controllers handle HTTP requests/responses
   - Services contain business logic
   - Repositories handle database operations

2. **Maintainability:** If I need to change database logic, I only modify the repository layer without touching controllers or services.

3. **Testability:** I can test each layer independently. For example, I can test service logic by mocking the repository layer.

4. **Scalability:** Different layers can be scaled independently. If database operations become slow, I can optimize just the repository layer.

5. **Team Collaboration:** In a team, different developers can work on different layers without conflicts.

For example, in my project, when placing an order, the OrderController receives the request, OrderService performs validation and business logic (check balance, check stock), and OrderRepository saves the data. Each layer is focused and independent."

---

**Q3: Why did you use DTO pattern?**

**A:** "I used the DTO pattern for three main reasons:

1. **Security:** Entities contain sensitive fields like password. DTOs let me exclude these fields when sending data to the client.

2. **Flexibility:** Sometimes the client needs data in a different format than how it's stored. For example, OrderDTO combines data from Order, Student, and ItemMaster entities.

3. **Decoupling:** Changes to database structure don't affect the API contract with the frontend.

For example, my Student entity has a password field, but StudentDTO doesn't. When a client requests profile information, I convert the entity to DTO, ensuring the password is never exposed in the response."

---

**Q4: Explain JWT authentication in your project.**

**A:** "In my project, JWT authentication works in two phases:

**Phase 1: Login (Token Generation)**
1. Student sends email and password to `/api/students/login`
2. Service verifies password using BCrypt
3. If valid, JwtUtils generates a JWT token containing:
   - Subject: User's email
   - Claims: User's role (STUDENT/ADMIN)
   - Expiration: 24 hours
4. Token is signed with a secret key using HMAC-SHA256
5. Token is returned to client, who stores it in localStorage

**Phase 2: Authorization (Token Validation)**
1. Client sends JWT in Authorization header: `Bearer <token>`
2. JwtAuthFilter intercepts every request
3. Filter extracts and validates token:
   - Verifies signature
   - Checks expiration
   - Extracts username
4. If valid, filter loads user details and sets authentication in SecurityContext
5. @PreAuthorize annotation on controller methods checks if user has required role
6. If authorized, method executes; otherwise, 403 Forbidden

This is stateless - no session stored on server, making it scalable and suitable for REST APIs."

---

### Intermediate Questions

**Q5: How did you handle concurrent order scenarios?**

**A:** "I handled concurrent orders using transaction isolation and optimistic locking:

1. **Transaction Isolation:** I used `@Transactional(isolation = Isolation.SERIALIZABLE)` for the placeOrder method. This is the highest isolation level, ensuring complete isolation between concurrent transactions.

2. **Atomic Operations:** The order placement involves multiple steps:
   - Check wallet balance
   - Deduct balance
   - Check stock
   - Reduce stock
   - Create order

   All these steps run in a single transaction. If any step fails, all changes rollback.

3. **Example Scenario:**
   - Last item in stock
   - Student A and Student B order simultaneously
   - SERIALIZABLE isolation ensures only one transaction proceeds
   - First transaction: Order succeeds, stock becomes 0
   - Second transaction: Fails with 'Insufficient stock' error

Without this, both orders might succeed, causing negative stock."

---

**Q6: Explain your exception handling strategy.**

**A:** "I implemented centralized exception handling using @RestControllerAdvice:

1. **Custom Exceptions:**
   - `ApiException`: For business logic errors (insufficient balance, out of stock)
   - `ResourceNotFoundException`: For entity not found errors

2. **Global Handler:** GlobalExceptionHandler catches all exceptions across controllers and returns consistent error responses.

3. **Validation Exceptions:** For @Valid annotations, I catch MethodArgumentNotValidException and return field-specific errors.

4. **Benefits:**
   - Consistent error format across all APIs
   - Controllers remain clean (no try-catch blocks)
   - Easy to add logging or monitoring

Example: When a student tries to order with insufficient balance, the service throws ApiException, which is caught by the global handler and returned as a 400 Bad Request with a meaningful message."

---

**Q7: How does your password security work?**

**A:** "I used BCrypt for password security:

**During Registration:**
```java
String plainPassword = "password123";
String hashedPassword = passwordEncoder.encode(plainPassword);
// Stored in DB: $2a$10$N.zmdr9k7uOCQb...
```

**During Login:**
```java
boolean matches = passwordEncoder.matches(userInput, storedHash);
```

**Why BCrypt:**
1. **One-way hashing:** Cannot reverse hash to get password
2. **Automatic salt:** Each password gets unique salt, preventing rainbow table attacks
3. **Slow algorithm:** Intentionally slow (cost factor 10), making brute force impractical
4. **Adaptive:** Can increase cost factor as computers get faster

Even if the database is compromised, attackers can't get actual passwords. They would need to brute force each password individually, which is computationally infeasible with BCrypt."

---

**Q8: Explain the order placement transaction.**

**A:** "Order placement is a complex atomic transaction with multiple validations:

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public OrderDTO placeOrder(PlaceOrderRequest request) {
    // Step 1: Fetch and validate student
    Student student = studentRepo.findById(request.getStudentId())
        .orElseThrow(() -> new ResourceNotFoundException("Student not found"));
    
    // Step 2: Fetch and validate item
    ItemDaily itemDaily = itemDailyRepo.findById(request.getItemDailyId())
        .orElseThrow(() -> new ResourceNotFoundException("Item not found"));
    
    // Step 3: Check stock availability
    if (itemDaily.getAvailableQuantity() < request.getQuantity()) {
        throw new ApiException("Insufficient stock");
    }
    
    // Step 4: Calculate total
    Double totalPrice = itemDaily.getItemMaster().getPrice() * request.getQuantity();
    
    // Step 5: Check wallet balance
    if (student.getWalletBalance() < totalPrice) {
        throw new ApiException("Insufficient balance");
    }
    
    // Step 6: Deduct balance
    student.setWalletBalance(student.getWalletBalance() - totalPrice);
    studentRepo.save(student);
    
    // Step 7: Reduce stock
    itemDaily.setAvailableQuantity(itemDaily.getAvailableQuantity() - request.getQuantity());
    itemDaily.setSoldQuantity(itemDaily.getSoldQuantity() + request.getQuantity());
    itemDailyRepo.save(itemDaily);
    
    // Step 8: Create order
    Order order = Order.builder()
        .student(student)
        .itemDaily(itemDaily)
        .quantity(request.getQuantity())
        .totalPrice(totalPrice)
        .status(OrderStatus.PENDING)
        .build();
    
    Order saved = orderRepo.save(order);
    return convertToDTO(saved);
}
```

If any step fails (exception thrown), all changes rollback automatically due to @Transactional."

---

### Advanced Questions

**Q9: How would you scale this application?**

**A:** "To scale MealStack, I would:

1. **Database Optimization:**
   - Add indexes on frequently queried columns (email, date, student_id)
   - Implement database connection pooling
   - Consider read replicas for reporting queries

2. **Caching:**
   - Cache daily menu items using Redis (updated once per day)
   - Cache student profile data (invalidate on update)
   - Reduce database load for frequent reads

3. **Horizontal Scaling:**
   - Deploy multiple instances of Spring Boot app behind load balancer
   - Stateless JWT authentication enables this
   - Session not stored on server, so any instance can handle any request

4. **Async Processing:**
   - Use message queue (RabbitMQ/Kafka) for non-critical operations
   - Example: Send order confirmation emails asynchronously

5. **Database Sharding:**
   - If institution grows very large, shard students table by course
   - PGDAC students on DB1, PGDBDA on DB2, etc.

6. **CDN for Images:**
   - Store item images on CDN (CloudFront, Cloudinary)
   - Reduce server load and improve load times

7. **Monitoring:**
   - Add application monitoring (Prometheus, Grafana)
   - Track API response times, error rates
   - Alert on anomalies"

---

**Q10: If you had more time, what would you improve?**

**A:** "Several enhancements I would add:

1. **Real-time Updates:**
   - WebSocket integration for live order status
   - Push notifications when order is ready
   - Real-time stock updates for all users

2. **Advanced Features:**
   - QR code generation for order pickup
   - AI-based meal recommendations based on order history
   - Predictive analytics for inventory planning
   - Nutrition information for items

3. **Better Security:**
   - Implement refresh tokens (short-lived access token + long-lived refresh token)
   - Add rate limiting to prevent API abuse
   - Implement CAPTCHA for login
   - Two-factor authentication for admin

4. **Testing:**
   - Increase unit test coverage to 80%+
   - Add integration tests
   - Load testing with JMeter

5. **DevOps:**
   - Dockerize application
   - CI/CD pipeline with GitHub Actions
   - Kubernetes deployment for auto-scaling

6. **Mobile App:**
   - React Native mobile app for students
   - Push notifications for order updates

7. **Analytics Dashboard:**
   - Advanced reporting for admins
   - Revenue tracking
   - Popular items analysis
   - Peak hour identification"

---

**Q11: Explain @Transactional in detail.**

**A:** "@Transactional provides ACID properties for database operations:

**ACID Properties:**
1. **Atomicity:** All or nothing - all operations succeed or all rollback
2. **Consistency:** Database remains in consistent state
3. **Isolation:** Concurrent transactions don't interfere
4. **Durability:** Committed data persists even after system failure

**Isolation Levels:**
```
READ_UNCOMMITTED (Level 0):
- Can read uncommitted changes from other transactions
- Problem: Dirty reads
- Example: Transaction A updates price to 50 but not committed,
            Transaction B reads 50, A rollbacks, B has wrong data

READ_COMMITTED (Level 1):
- Can only read committed changes
- Problem: Non-repeatable reads
- Example: Transaction A reads price as 50,
            Transaction B updates to 60 and commits,
            Transaction A reads again, gets 60 (different value!)

REPEATABLE_READ (Level 2):
- Same data read multiple times gives same result
- Problem: Phantom reads
- Example: Transaction A counts 10 rows,
            Transaction B inserts 1 row and commits,
            Transaction A counts again, gets 11 (phantom row!)

SERIALIZABLE (Level 3):
- Complete isolation
- No dirty reads, non-repeatable reads, or phantom reads
- Slowest but safest
- Used in my placeOrder method
```

**Propagation:**
```
REQUIRED (default): Use existing transaction or create new
REQUIRES_NEW: Always create new transaction
NESTED: Create nested transaction (savepoint)
MANDATORY: Must be called within transaction
NEVER: Must not be called within transaction
```

Example in my project:
```java
@Transactional(
    isolation = Isolation.SERIALIZABLE,  // Highest isolation
    propagation = Propagation.REQUIRED,  // Use existing transaction
    rollbackFor = Exception.class        // Rollback on any exception
)
public OrderDTO placeOrder(PlaceOrderRequest request) {
    // Multiple DB operations
    // All succeed or all rollback
}
```"

---

**Q12: How does JPA/Hibernate work internally?**

**A:** "JPA is a specification, Hibernate is the implementation:

**Key Concepts:**

1. **Entity Manager:**
   - Manages entity lifecycle
   - Tracks changes to entities
   - Synchronizes with database

2. **Persistence Context (First-Level Cache):**
   - In-memory cache of entities
   - Same entity retrieved multiple times in a transaction = same object
   - Example:
   ```java
   Student s1 = studentRepo.findById(1);  // Database query
   Student s2 = studentRepo.findById(1);  // From cache, no query!
   // s1 == s2 (same object reference)
   ```

3. **Entity States:**
   ```
   NEW/TRANSIENT → Entity created, not in persistence context
   MANAGED       → In persistence context, changes tracked
   DETACHED      → Was managed, now removed from context
   REMOVED       → Marked for deletion
   ```

4. **Dirty Checking:**
   - Hibernate tracks changes to managed entities
   - At transaction commit, automatically generates UPDATE queries
   ```java
   Student student = studentRepo.findById(1);  // MANAGED
   student.setName("New Name");                // Change tracked
   // No save() needed!
   // Commit → Hibernate generates UPDATE automatically
   ```

5. **Lazy vs Eager Loading:**
   ```java
   @ManyToOne(fetch = FetchType.LAZY)
   private Student student;
   
   Order order = orderRepo.findById(1);
   // Student NOT loaded yet
   String name = order.getStudent().getName();
   // NOW student is loaded (second query)
   ```

6. **N+1 Problem:**
   ```java
   // BAD: N+1 queries
   List<Order> orders = orderRepo.findAll();  // 1 query
   for (Order order : orders) {
       String name = order.getStudent().getName();  // N queries!
   }
   
   // GOOD: 1 query with JOIN
   @Query("SELECT o FROM Order o JOIN FETCH o.student")
   List<Order> findAllWithStudent();
   ```

7. **Query Generation:**
   ```java
   // Method name
   List<Student> findByEmailAndCourse(String email, Course course);
   
   // Generated SQL
   SELECT * FROM students WHERE email = ? AND course = ?
   ```"

---

## 📝 QUICK REVISION NOTES

### Spring Boot Annotations Cheat Sheet

```java
// Component Stereotypes
@Component       // Generic component
@Service         // Business logic layer
@Repository      // Data access layer
@Controller      // Web controller (returns views)
@RestController  // REST controller (returns JSON)

// Dependency Injection
@Autowired       // Auto-inject dependency
@Qualifier       // Specify which bean to inject
@Primary         // Preferred bean when multiple candidates

// Configuration
@Configuration   // Configuration class
@Bean            // Define bean manually
@Value           // Inject property value

// JPA/Entity
@Entity          // JPA entity
@Table           // Specify table name
@Id              // Primary key
@GeneratedValue  // Auto-generate value
@Column          // Column properties
@OneToMany       // One-to-many relationship
@ManyToOne       // Many-to-one relationship
@JoinColumn      // Foreign key column

// REST Endpoints
@GetMapping      // HTTP GET
@PostMapping     // HTTP POST
@PutMapping      // HTTP PUT
@DeleteMapping   // HTTP DELETE
@RequestMapping  // Base mapping
@PathVariable    // URL parameter
@RequestParam    // Query parameter
@RequestBody     // Request body (JSON)
@Valid           // Trigger validation

// Security
@PreAuthorize    // Method-level security
@EnableWebSecurity        // Enable security
@EnableMethodSecurity     // Enable method security

// Exception Handling
@RestControllerAdvice     // Global exception handler
@ExceptionHandler         // Handle specific exception

// Transaction
@Transactional            // Transaction management

// Lombok
@Getter / @Setter         // Generate getters/setters
@NoArgsConstructor        // Empty constructor
@AllArgsConstructor       // Constructor with all fields
@Builder                  // Builder pattern
@Data                     // All of above + toString, equals, hashCode

// Scheduling
@EnableScheduling         // Enable scheduling
@Scheduled                // Schedule method execution
```

---

### HTTP Status Codes

```
2xx Success:
200 OK                    // Success
201 Created               // Resource created
204 No Content            // Success, no response body

4xx Client Errors:
400 Bad Request           // Invalid request
401 Unauthorized          // Not authenticated
403 Forbidden             // Not authorized
404 Not Found             // Resource not found
409 Conflict              // Duplicate resource

5xx Server Errors:
500 Internal Server Error // Server error
503 Service Unavailable   // Server overloaded
```

---

### SQL vs JPQL

```sql
-- SQL (table-based)
SELECT s.* FROM students s 
WHERE s.email = 'john@example.com'

-- JPQL (entity-based)
SELECT s FROM Student s 
WHERE s.email = 'john@example.com'
```

---

### Key Design Decisions

1. **Why JWT over Session?**
   - Stateless (scalable)
   - Works across multiple servers
   - Mobile-friendly

2. **Why BCrypt over MD5/SHA?**
   - Slow by design (prevents brute force)
   - Auto-generates salt
   - Adaptive cost factor

3. **Why DTO over Entity?**
   - Security (hide sensitive fields)
   - Flexibility (custom response format)
   - Decoupling (API independent of DB schema)

4. **Why @Transactional?**
   - Ensures atomicity
   - Auto-rollback on error
   - Maintains data consistency

5. **Why Layered Architecture?**
   - Separation of concerns
   - Easy testing
   - Better maintainability

---

### Common Pitfalls & Solutions

1. **N+1 Query Problem:**
   ```java
   // PROBLEM
   @ManyToOne
   private Student student;
   
   // SOLUTION
   @ManyToOne(fetch = FetchType.EAGER)
   // OR use @Query with JOIN FETCH
   ```

2. **LazyInitializationException:**
   ```java
   // PROBLEM: Access lazy field outside transaction
   Order order = orderRepo.findById(1);
   order.getStudent().getName();  // Error!
   
   // SOLUTION: Use @Transactional or EAGER loading
   ```

3. **Circular Dependency:**
   ```java
   // PROBLEM
   @Entity
   public class Student {
       @OneToMany
       private List<Order> orders;  // Includes order details
   }
   
   @Entity
   public class Order {
       @ManyToOne
       private Student student;  // Includes student details
   }
   // Infinite loop when converting to JSON!
   
   // SOLUTION: Use @JsonIgnore or DTO
   ```

---

This completes the backend explanation. Next, I'll create the Frontend preparation guide.

Would you like me to proceed with the Frontend (React) interview preparation notes?
