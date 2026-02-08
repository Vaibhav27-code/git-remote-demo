# MealStack - Complete Interview Questions & Answers

## 📋 TABLE OF CONTENTS
1. [Project Overview Questions](#project-overview-questions)
2. [Backend (Spring Boot) Questions](#backend-spring-boot-questions)
3. [Frontend (React) Questions](#frontend-react-questions)
4. [Database Questions](#database-questions)
5. [Security Questions](#security-questions)
6. [Scenario-Based Questions](#scenario-based-questions)
7. [Behavioral Questions](#behavioral-questions)

---

## 🎯 PROJECT OVERVIEW QUESTIONS

### Q1: Tell me about your project.
**Answer:**
"MealStack is a full-stack canteen management system I developed during my PG-DAC course. The project digitizes the entire food ordering process in educational institutions.

**Problem Statement:**
Traditional canteens face issues like long queues, cash handling errors, manual inventory tracking, and lack of order history.

**Solution:**
MealStack provides a cashless, digital platform where students can browse daily menus, check real-time availability, place orders using their digital wallet, and track order history. Admins can manage inventory, set daily menus, process orders, and view analytics.

**Tech Stack:**
- Backend: Spring Boot 3.x with Spring Security, JPA/Hibernate
- Frontend: React 18 with Redux Toolkit, Material-UI
- Database: MySQL 8.0
- Security: JWT-based authentication, BCrypt password hashing

**Key Features:**
- Real-time inventory tracking
- Atomic order transactions
- Role-based access control (Student/Admin)
- Automated daily menu reset via scheduler
- Comprehensive order history and analytics

**Impact:**
- Reduced order processing time from minutes to seconds
- Eliminated cash handling
- Real-time stock visibility
- Data-driven decision making for admins"

---

### Q2: Why did you choose this tech stack?

**Answer:**
"I chose this tech stack based on industry standards and project requirements:

**Spring Boot:**
- Auto-configuration reduces boilerplate
- Built-in embedded server (Tomcat)
- Production-ready features (actuator, metrics)
- Large ecosystem and community support
- Easy to create RESTful APIs

**React:**
- Component-based architecture promotes reusability
- Virtual DOM for performance
- Large ecosystem (Redux, Router, Material-UI)
- Active community and job market demand

**MySQL:**
- Reliable and mature RDBMS
- ACID compliance for transaction safety
- Good performance for our scale
- Easy integration with Spring Boot via JPA

**Redux:**
- Centralized state management
- Predictable state updates
- Time-travel debugging with DevTools
- Prevents prop drilling

**JWT:**
- Stateless authentication (scalable)
- Works well with REST APIs
- No server-side session storage
- Mobile-friendly"

---

### Q3: What challenges did you face and how did you solve them?

**Answer:**
"**Challenge 1: Concurrent Order Handling**
- Problem: Two students ordering the last item simultaneously could both succeed, causing negative stock.
- Solution: Used `@Transactional(isolation = Isolation.SERIALIZABLE)` for complete transaction isolation. This ensures only one transaction proceeds at a time.

**Challenge 2: Real-time Stock Updates**
- Problem: Stock shown on menu page could become outdated.
- Solution: Implemented atomic inventory updates. Every order placement immediately reduces stock in the database, ensuring accuracy.

**Challenge 3: State Management Complexity**
- Problem: Multiple components needed access to cart, menu, and user data.
- Solution: Implemented Redux for centralized state management, eliminating prop drilling and ensuring single source of truth.

**Challenge 4: Security**
- Problem: Protecting sensitive endpoints and data.
- Solution: 
  - JWT-based authentication for stateless security
  - BCrypt for password hashing
  - Role-based access control with @PreAuthorize
  - DTO pattern to hide sensitive entity fields

**Challenge 5: Daily Menu Reset**
- Problem: Manual menu reset was error-prone.
- Solution: Implemented scheduled task using Spring's @Scheduled annotation to automatically reset menu at midnight."

---

## 🔧 BACKEND (SPRING BOOT) QUESTIONS

### Q4: Explain the architecture of your Spring Boot application.

**Answer:**
"I used a layered architecture with clear separation of concerns:

**1. Controller Layer (@RestController):**
- Handles HTTP requests and responses
- Maps URLs to methods using @GetMapping, @PostMapping, etc.
- Validates input using @Valid
- Returns JSON responses
- Example: StudentController handles student registration, login, profile

**2. Service Layer (@Service):**
- Contains business logic
- Orchestrates operations between repositories
- Handles transactions with @Transactional
- Example: OrderService validates balance, checks stock, creates order

**3. Repository Layer (@Repository):**
- Data access layer
- Extends JpaRepository for CRUD operations
- Custom queries using method names or @Query
- Example: StudentRepository.findByEmail()

**4. Entity Layer (@Entity):**
- JPA entities mapping to database tables
- Define relationships (@OneToMany, @ManyToOne)
- Example: Student, Order, ItemMaster

**5. DTO Layer:**
- Data Transfer Objects
- Used for request/response to avoid exposing entities
- Example: StudentDTO (no password field)

**6. Security Layer:**
- JwtAuthFilter for JWT validation
- SecurityConfig for security rules
- UserDetailsService for loading user details

**Flow Example (Place Order):**
```
Client → OrderController → OrderService → OrderRepository → Database
                              ↓
                        StudentService
                              ↓
                        StudentRepository
                              ↓
                        ItemDailyRepository
```"

---

### Q5: What is Dependency Injection? How does Spring implement it?

**Answer:**
"Dependency Injection is a design pattern where objects receive their dependencies from external sources rather than creating them.

**Without DI:**
```java
public class OrderService {
    private OrderRepository orderRepo = new OrderRepository();  // Tight coupling!
}
```

**With DI:**
```java
@Service
public class OrderService {
    @Autowired  // Spring injects this
    private OrderRepository orderRepo;
}
```

**Spring's DI Container (IoC Container):**
1. Scans for @Component, @Service, @Repository annotations
2. Creates beans (objects) and stores them
3. Injects dependencies where needed using @Autowired

**Benefits:**
- **Loose Coupling:** OrderService doesn't depend on concrete implementation
- **Testability:** Can inject mock repositories for testing
- **Flexibility:** Easy to swap implementations

**Types of DI in Spring:**
```java
// 1. Constructor Injection (Recommended)
@Service
public class OrderService {
    private final OrderRepository orderRepo;
    
    @Autowired  // Optional in Spring 4.3+
    public OrderService(OrderRepository orderRepo) {
        this.orderRepo = orderRepo;
    }
}

// 2. Setter Injection
@Service
public class OrderService {
    private OrderRepository orderRepo;
    
    @Autowired
    public void setOrderRepo(OrderRepository orderRepo) {
        this.orderRepo = orderRepo;
    }
}

// 3. Field Injection (I used this)
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepo;
}
```"

---

### Q6: Explain JPA/Hibernate and how it works.

**Answer:**
"**JPA (Java Persistence API):**
- Specification for ORM (Object-Relational Mapping)
- Standard way to map Java objects to database tables

**Hibernate:**
- Implementation of JPA
- Most popular ORM framework

**How it works:**

1. **Entity Mapping:**
```java
@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String email;
}
```

2. **Persistence Context (First-Level Cache):**
```java
Student s1 = studentRepo.findById(1);  // DB query
Student s2 = studentRepo.findById(1);  // Cache hit, no query!
s1 == s2  // true (same object)
```

3. **Automatic Query Generation:**
```java
// Method name
Optional<Student> findByEmail(String email);

// Generated SQL
SELECT * FROM students WHERE email = ?
```

4. **Dirty Checking:**
```java
Student student = studentRepo.findById(1);  // MANAGED state
student.setName("New Name");                // Change tracked
// No save() needed!
// At transaction commit, Hibernate auto-generates UPDATE
```

5. **Lazy Loading:**
```java
@ManyToOne(fetch = FetchType.LAZY)
private Student student;

Order order = orderRepo.findById(1);  // Student NOT loaded
String name = order.getStudent().getName();  // NOW loaded (proxy)
```

**In my project:**
- Used JPA annotations for entity mapping
- JpaRepository for CRUD operations
- Custom queries for complex operations
- @Transactional for transaction management"

---

### Q7: What is @Transactional and when do you use it?

**Answer:**
"@Transactional ensures ACID properties for database operations:

**ACID:**
- **Atomicity:** All operations succeed or all fail
- **Consistency:** Database remains in valid state
- **Isolation:** Concurrent transactions don't interfere
- **Durability:** Committed changes persist

**Example from my project:**
```java
@Service
@Transactional
public class OrderServiceImpl {
    
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public OrderDTO placeOrder(PlaceOrderRequest request) {
        // Step 1: Check balance
        if (student.getWalletBalance() < totalPrice) {
            throw new ApiException("Insufficient balance");
        }
        
        // Step 2: Deduct balance
        student.setWalletBalance(student.getWalletBalance() - totalPrice);
        studentRepo.save(student);
        
        // Step 3: Reduce stock
        itemDaily.reduceStock(request.getQuantity());
        itemDailyRepo.save(itemDaily);
        
        // Step 4: Create order
        Order order = orderRepo.save(newOrder);
        
        // If ANY step fails → ALL steps rollback!
        return convertToDTO(order);
    }
}
```

**When to use:**
- Multiple database operations that must succeed/fail together
- Methods that modify data
- Operations requiring specific isolation levels

**Propagation:**
```java
REQUIRED (default) → Use existing transaction or create new
REQUIRES_NEW       → Always create new transaction
NESTED            → Create nested transaction
MANDATORY         → Must be called within transaction
```

**Isolation Levels:**
```java
READ_UNCOMMITTED  → Dirty reads possible
READ_COMMITTED    → No dirty reads
REPEATABLE_READ   → No non-repeatable reads
SERIALIZABLE      → Complete isolation (I used for order placement)
```"

---

### Q8: How does Spring Security work in your project?

**Answer:**
"Spring Security provides authentication and authorization:

**Authentication Flow:**

1. **Login Request:**
```
POST /api/students/login
Body: { email, password }
```

2. **StudentService validates:**
```java
// Load user from database
UserDetails user = userDetailsService.loadUserByUsername(email);

// Verify password (BCrypt)
if (!passwordEncoder.matches(password, user.getPassword())) {
    throw new BadCredentialsException();
}

// Generate JWT token
String token = jwtUtils.generateToken(user);

return new AuthResponse(user, token);
```

3. **Client stores JWT in localStorage**

**Authorization Flow:**

1. **Client sends request with JWT:**
```
GET /api/students/profile/5
Headers: { Authorization: "Bearer eyJhbGc..." }
```

2. **JwtAuthFilter intercepts:**
```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
    protected void doFilterInternal(...) {
        // 1. Extract JWT from header
        String token = extractToken(request);
        
        // 2. Validate JWT
        if (jwtUtils.validateToken(token)) {
            // 3. Load user details
            UserDetails user = userDetailsService.loadByUsername(username);
            
            // 4. Set authentication in SecurityContext
            SecurityContextHolder.getContext()
                .setAuthentication(authToken);
        }
    }
}
```

3. **Controller checks authorization:**
```java
@PreAuthorize("hasRole('STUDENT')")
public ResponseEntity<StudentDTO> getProfile(@PathVariable Long id) {
    // Only executes if user has STUDENT role
}
```

**Security Configuration:**
```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        http
            .csrf().disable()  // Stateless JWT
            .authorizeHttpRequests()
                .requestMatchers("/api/students/login", "/api/students/register")
                .permitAll()
                .anyRequest().authenticated()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        
        return http.build();
    }
}
```"

---

### Q9: Explain the difference between @Component, @Service, and @Repository.

**Answer:**
"All three are Spring stereotypes for component scanning, but have different purposes:

**@Component:**
- Generic stereotype
- Used for any Spring-managed component
- Other annotations are specializations of @Component

**@Service:**
- Used for service layer classes
- Contains business logic
- Example: OrderService, StudentService

**@Repository:**
- Used for data access layer
- Adds automatic exception translation (SQLException → DataAccessException)
- Example: StudentRepository, OrderRepository

**@Controller / @RestController:**
- Used for presentation layer
- Handles HTTP requests
- @RestController = @Controller + @ResponseBody

**Why different annotations?**
1. **Code Readability:** Clear indication of layer
2. **AOP:** Can apply aspects to specific layers
3. **Exception Translation:** @Repository adds DB exception translation
4. **Future Features:** Spring may add layer-specific features

**In my project:**
```java
@RestController  // Controller layer
public class StudentController { }

@Service  // Service layer
public class StudentServiceImpl { }

@Repository  // Data access layer
public interface StudentRepository extends JpaRepository { }
```"

---

### Q10: What are HTTP methods and when do you use each?

**Answer:**
"HTTP methods define the operation to perform:

**GET:**
- Retrieve data
- Should not modify server state
- Idempotent (same result on multiple calls)
- Example: Get student profile, Get menu items
```java
@GetMapping("/students/{id}")
public StudentDTO getStudent(@PathVariable Long id) { }
```

**POST:**
- Create new resource
- Modifies server state
- Not idempotent
- Example: Register student, Place order
```java
@PostMapping("/students/register")
public ApiResponse register(@RequestBody RegisterDTO dto) { }
```

**PUT:**
- Update entire resource
- Idempotent
- Example: Update student profile
```java
@PutMapping("/students/{id}")
public StudentDTO update(@PathVariable Long id, @RequestBody StudentDTO dto) { }
```

**PATCH:**
- Update partial resource
- Example: Update only name
```java
@PatchMapping("/students/{id}/name")
public void updateName(@PathVariable Long id, @RequestParam String name) { }
```

**DELETE:**
- Delete resource
- Idempotent
- Example: Delete item from cart
```java
@DeleteMapping("/cart/{itemId}")
public void removeItem(@PathVariable Long itemId) { }
```

**In my project:**
- GET → Fetch menu, orders, profile
- POST → Login, register, place order, recharge wallet
- PUT → Update profile, mark order as served
- DELETE → Remove cart items"

---

## ⚛️ FRONTEND (REACT) QUESTIONS

### Q11: What is React and why did you use it?

**Answer:**
"React is a JavaScript library for building user interfaces, created by Facebook.

**Why I chose React:**

1. **Component-Based:**
   - Reusable components (MenuCard, OrderCard)
   - Each component manages its own state
   - Easy to maintain and test

2. **Virtual DOM:**
   - Efficient updates (only changed parts re-render)
   - Better performance than direct DOM manipulation

3. **Declarative:**
   - Describe UI for each state
   - React handles DOM updates
   ```jsx
   {isLoggedIn ? <Dashboard /> : <LoginPage />}
   ```

4. **Large Ecosystem:**
   - Redux for state management
   - React Router for navigation
   - Material-UI for components
   - Active community support

5. **Job Market:**
   - Most demanded frontend skill
   - Used by Facebook, Netflix, Airbnb

**In my project:**
- MenuPage component displays daily menu
- CartPage manages shopping cart
- OrdersPage shows order history
- Navbar component reused across pages"

---

### Q12: Explain Virtual DOM.

**Answer:**
"Virtual DOM is a lightweight JavaScript copy of the actual DOM.

**How it works:**

1. **Initial Render:**
```jsx
<div>
  <h1>Count: {0}</h1>
  <button>Click</button>
</div>
```
React creates:
- Virtual DOM tree
- Actual DOM tree

2. **State Change:**
```jsx
// count changes from 0 to 1
<div>
  <h1>Count: {1}</h1>
  <button>Click</button>
</div>
```

3. **Diffing:**
React compares:
- Old Virtual DOM
- New Virtual DOM
- Identifies: "Only text in <h1> changed"

4. **Update:**
React updates ONLY the text node:
```javascript
// Instead of recreating entire div:
document.querySelector('h1').textContent = 'Count: 1';
```

**Benefits:**
- **Performance:** Batch updates, minimize DOM operations
- **Efficiency:** Only update what changed
- **Declarative:** Developer describes end state, React handles updates

**Real DOM vs Virtual DOM:**
```
Real DOM                    Virtual DOM
- Slow to update            - Fast in-memory operation
- Expensive operations      - Cheap operations
- Directly updates browser  - Updates minimal changes
```

**Example from my project:**
When cart total updates, React only changes the price text, not the entire cart component."

---

### Q13: What is the difference between state and props?

**Answer:**
"**State:**
- Data owned by component
- Mutable (can be changed)
- Private to component
- Triggers re-render when changed
- Created using useState

```jsx
function Counter() {
  const [count, setCount] = useState(0);  // State
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

**Props:**
- Data passed from parent to child
- Immutable (read-only)
- Public
- Component re-renders when props change
- Passed as attributes

```jsx
// Parent
function MenuPage() {
  const item = { name: 'Dosa', price: 50 };
  return <MenuCard item={item} />;  // Props
}

// Child
function MenuCard({ item }) {  // Receiving props
  // item.price = 100;  // ERROR: Can't modify props!
  return <h3>{item.name}: ₹{item.price}</h3>;
}
```

**Comparison:**
```
State                      Props
- Internal to component    - External (from parent)
- Can change (setState)    - Cannot change
- Asynchronous updates     - Synchronous
- Causes re-render         - New props cause re-render
```

**In my project:**
- State: Cart items, order form data, loading status
- Props: Menu item details, order details, user info passed to child components"

---

### Q14: What are React Hooks? Explain useState and useEffect.

**Answer:**
"Hooks are functions that let you use React features in functional components.

**useState:**
Adds state to functional component.

```jsx
import { useState } from 'react';

function OrderForm() {
  // [stateVariable, setterFunction] = useState(initialValue)
  const [quantity, setQuantity] = useState(1);
  const [totalPrice, setTotalPrice] = useState(0);
  
  const increase = () => setQuantity(quantity + 1);
  const decrease = () => setQuantity(q => q - 1);  // Functional update
  
  return (
    <div>
      <button onClick={decrease}>-</button>
      <span>{quantity}</span>
      <button onClick={increase}>+</button>
    </div>
  );
}
```

**useEffect:**
Performs side effects (API calls, subscriptions, timers).

```jsx
import { useEffect } from 'react';

function MenuPage() {
  const [menu, setMenu] = useState([]);
  
  // Runs after every render
  useEffect(() => {
    console.log('Rendered');
  });
  
  // Runs once on mount (componentDidMount)
  useEffect(() => {
    fetchMenu().then(data => setMenu(data));
  }, []);  // Empty dependency array
  
  // Runs when date changes (componentDidUpdate)
  useEffect(() => {
    fetchMenu(date).then(data => setMenu(data));
  }, [date]);  // Dependency array
  
  // Cleanup (componentWillUnmount)
  useEffect(() => {
    const interval = setInterval(() => {
      console.log('Tick');
    }, 1000);
    
    return () => clearInterval(interval);  // Cleanup
  }, []);
}
```

**Other Hooks I used:**
- **useSelector:** Access Redux state
- **useDispatch:** Dispatch Redux actions
- **useNavigate:** Navigate programmatically
- **useContext:** Access Context API

**In my project:**
- useState: Form inputs, cart items, loading states
- useEffect: Fetch menu on mount, refetch when date changes"

---

### Q15: Explain Redux and why you used it.

**Answer:**
"Redux is a state management library for predictable state updates.

**Why I used Redux:**

1. **Centralized State:**
   - Single store for entire application
   - Cart, user, menu data accessible everywhere

2. **Predictable:**
   - State changes only through actions + reducers
   - Easy to debug with Redux DevTools

3. **Avoid Prop Drilling:**
```jsx
// Without Redux (Prop drilling)
<App user={user}>
  <Dashboard user={user}>
    <Sidebar user={user}>
      <Profile user={user} />
    </Sidebar>
  </Dashboard>
</App>

// With Redux
<App>
  <Dashboard>
    <Sidebar>
      <Profile />  {/* Access user via useSelector */}
    </Sidebar>
  </Dashboard>
</App>
```

4. **DevTools:**
   - See every action and state change
   - Time-travel debugging

**Redux Flow:**
```
┌──────────────┐
│  Component   │
└──────┬───────┘
       │ dispatch(action)
       ▼
┌──────────────┐
│    Action    │
│ { type, payload }
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Reducer    │
│ (state, action) → newState
└──────┬───────┘
       │
       ▼
┌──────────────┐
│    Store     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Component   │
│  (re-render) │
└──────────────┘
```

**Example from my project:**
```jsx
// Action
dispatch(addToCart(item));

// Reducer
addToCart: (state, action) => {
  state.items.push(action.payload);
  state.totalPrice += action.payload.price;
}

// Access in component
const { items, totalPrice } = useSelector(state => state.cart);
```"

---

## 💾 DATABASE QUESTIONS

### Q16: Explain your database schema.

**Answer:**
"My database has 9 tables with proper relationships:

**1. users (Parent table):**
- id, name, email, password, mobile_no, role, created_at
- Parent for students and admin (inheritance)

**2. students:**
- Inherits from users
- Additional: date_of_birth, course, wallet_balance
- One student → Many orders, recharge_histories

**3. admin:**
- Inherits from users
- Admin-specific fields

**4. item_master (Menu catalog):**
- id, item_name, price, category, genre, image_url
- Template for daily items
- One item_master → Many item_daily

**5. item_daily (Today's menu):**
- id, item_master_id, date, initial_quantity, available_quantity, sold_quantity
- Unique constraint: (date, item_master_id) - same item can't appear twice per day
- Tracks daily stock

**6. orders:**
- id, student_id, item_daily_id, quantity, total_price, transaction_id, status, order_time
- Links student to item_daily
- Many orders → One student
- Many orders → One item_daily

**7. carts:**
- id, student_id, item_daily_id, quantity
- Temporary storage for cart items
- Cleared after order placement

**8. recharge_history:**
- id, student_id, amount, transaction_id, recharge_time
- Audit trail for wallet recharges

**Relationships:**
```
User
 ├── Student (1:1 inheritance)
 │   ├── Orders (1:N)
 │   ├── Carts (1:N)
 │   └── RechargeHistory (1:N)
 └── Admin (1:1 inheritance)

ItemMaster
 └── ItemDaily (1:N)
     └── Orders (1:N)
```"

---

### Q17: What is normalization? What normal form is your database in?

**Answer:**
"Normalization is organizing data to reduce redundancy and improve integrity.

**Normal Forms:**

**1NF (First Normal Form):**
- Atomic values (no multi-valued attributes)
- Each column has unique name
- ✓ My database: All columns have single values

**2NF (Second Normal Form):**
- Must be in 1NF
- No partial dependencies (non-key attributes depend on entire primary key)
- ✓ My database: All tables have single-column primary keys

**3NF (Third Normal Form):**
- Must be in 2NF
- No transitive dependencies (non-key attributes don't depend on other non-key attributes)
- ✓ My database: In 3NF

**Example from my project:**

**Not in 3NF:**
```sql
orders (
  id,
  student_id,
  student_name,  -- Depends on student_id, not order id!
  student_email, -- Transitive dependency
  ...
)
```

**In 3NF:**
```sql
orders (
  id,
  student_id,  -- Foreign key to students table
  ...
)

students (
  id,
  name,
  email,
  ...
)
```

**Benefits:**
- No data redundancy (student name stored once)
- Update anomalies avoided
- Referential integrity maintained"

---

### Q18: What are indexes and did you use them?

**Answer:**
"Indexes are data structures that improve query performance.

**How indexes work:**
```sql
-- Without index: Full table scan O(n)
SELECT * FROM students WHERE email = 'john@example.com';
-- Scans all rows

-- With index: Tree search O(log n)
CREATE INDEX idx_email ON students(email);
-- Uses B-Tree for fast lookup
```

**In my project:**
```sql
-- Automatic indexes (Primary keys)
CREATE INDEX ON students(id);
CREATE INDEX ON orders(id);

-- Manual indexes for frequent queries
CREATE INDEX idx_student_email ON students(email);
-- Used by: SELECT * FROM students WHERE email = ?

CREATE INDEX idx_order_student ON orders(student_id);
-- Used by: SELECT * FROM orders WHERE student_id = ?

CREATE INDEX idx_item_daily_date ON item_daily(date);
-- Used by: SELECT * FROM item_daily WHERE date = ?

-- Composite index
CREATE INDEX idx_item_daily_date_item ON item_daily(date, item_master_id);
-- Used by: SELECT * FROM item_daily WHERE date = ? AND item_master_id = ?
```

**When NOT to index:**
- Small tables (< 1000 rows)
- Columns with low cardinality (few unique values)
- Frequently updated columns (indexes slow down INSERT/UPDATE)

**Trade-offs:**
- **Pros:** Faster SELECT queries
- **Cons:** Slower INSERT/UPDATE/DELETE, extra storage"

---

## 🔐 SECURITY QUESTIONS

### Q19: How did you implement authentication and authorization?

**Answer:**
"I implemented JWT-based authentication and role-based authorization.

**Authentication (Who are you?):**

1. **Login:**
```
POST /api/students/login
Body: { email, password }
```

2. **Verify credentials:**
```java
// Load user from database
UserDetails user = userDetailsService.loadUserByUsername(email);

// Verify password (BCrypt)
boolean matches = passwordEncoder.matches(inputPassword, storedHash);

if (!matches) {
    throw new BadCredentialsException();
}
```

3. **Generate JWT:**
```java
String token = Jwts.builder()
    .setSubject(user.getEmail())
    .claim("role", user.getRole())
    .setIssuedAt(new Date())
    .setExpiration(new Date(System.currentTimeMillis() + 86400000))  // 24 hours
    .signWith(secretKey, SignatureAlgorithm.HS256)
    .compact();
```

4. **Return token:**
```json
{
  "token": "eyJhbGc...",
  "user": { "id": 1, "name": "John", "role": "STUDENT" }
}
```

**Authorization (What can you do?):**

1. **Client sends token:**
```
GET /api/students/profile/5
Headers: { Authorization: "Bearer eyJhbGc..." }
```

2. **JwtAuthFilter validates:**
```java
// Extract token
String token = extractTokenFromHeader(request);

// Validate signature and expiration
if (jwtUtils.validateToken(token)) {
    // Extract username and load user
    UserDetails user = userDetailsService.loadUserByUsername(username);
    
    // Set authentication
    SecurityContextHolder.getContext().setAuthentication(authToken);
}
```

3. **Controller checks role:**
```java
@PreAuthorize("hasRole('STUDENT')")
public ResponseEntity<StudentDTO> getProfile(@PathVariable Long id) {
    // Only executes if user has STUDENT role
}
```

**Why JWT?**
- Stateless (no server-side sessions)
- Scalable (works across multiple servers)
- Self-contained (token has all info)
- Mobile-friendly"

---

### Q20: Why BCrypt over MD5 or SHA?

**Answer:**
"BCrypt is specifically designed for password hashing:

**Problems with MD5/SHA:**
```java
// MD5 (DON'T use!)
String hash = MD5("password123");
// Result: 482c811da5d5b4bc6d497ffa98491e38
// Problems:
// 1. Fast algorithm → Brute force easy
// 2. Same password → Same hash (no salt)
// 3. Rainbow tables exist
```

**BCrypt Advantages:**

1. **Slow by design:**
```java
// Cost factor 10 = 2^10 iterations
String hash = passwordEncoder.encode("password123");
// Takes ~100ms
// Brute force would take years!
```

2. **Automatic salt:**
```
$2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM36bIWGCJQbiVB1IW.6
─┬─ ─┬─ ──────────────┬────────────── ──────────────┬─────────────
 │   │               │                              │
Algorithm Cost       Salt                        Hash
```

3. **Adaptive:**
```java
// Can increase cost factor as computers get faster
BCryptPasswordEncoder(12);  // More secure, slower
```

4. **One-way:**
- Cannot reverse hash to get password
- Must use `matches()` to verify

**Example from my project:**
```java
// Registration
String hashed = passwordEncoder.encode("password123");
student.setPassword(hashed);
// Stored: $2a$10$N.zmdr9k7uOCQb...

// Login
boolean valid = passwordEncoder.matches("password123", storedHash);
```"

---

## 🎭 SCENARIO-BASED QUESTIONS

### Q21: Two students order the last item simultaneously. How do you handle this?

**Answer:**
"This is a classic race condition. I solved it using transaction isolation:

**Problem:**
```
Time    Student A              Student B
T1      Check stock: 1          -
T2      -                       Check stock: 1
T3      Place order ✓           -
T4      -                       Place order ✓
Result: Stock = -1 (WRONG!)
```

**Solution 1: SERIALIZABLE Isolation**
```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public OrderDTO placeOrder(PlaceOrderRequest request) {
    // Complete isolation
    // Student B waits until Student A completes
}
```

Result:
```
Student A: Order succeeds, stock = 0
Student B: Waits, then gets "Out of stock" error
```

**Solution 2: Optimistic Locking**
```java
@Entity
public class ItemDaily {
    @Version
    private Integer version;  // Increments on each update
}
```

Flow:
```
1. Student A loads item (version=1)
2. Student B loads item (version=1)
3. Student A updates → version=2 ✓
4. Student B tries to update with version=1 → Fails!
```

**I used SERIALIZABLE because:**
- Simpler implementation
- Guaranteed correctness
- Small performance trade-off acceptable for order placement"

---

### Q22: How would you handle if the database goes down?

**Answer:**
"I would implement multiple layers of resilience:

**1. Connection Pooling (Already implemented):**
```properties
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.connection-timeout=30000
```
- Reuses connections
- Handles temporary network issues

**2. Retry Logic:**
```java
@Retryable(
    value = {DataAccessException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000)
)
public Student findByEmail(String email) {
    return studentRepo.findByEmail(email);
}
```

**3. Circuit Breaker:**
```java
@CircuitBreaker(name = "database", fallbackMethod = "getCachedMenu")
public List<ItemDaily> getTodaysMenu() {
    return itemDailyRepo.findByDate(LocalDate.now());
}

public List<ItemDaily> getCachedMenu(Exception e) {
    // Return cached data or empty list
    return cacheService.getMenu();
}
```

**4. Graceful Degradation:**
```java
try {
    return orderService.placeOrder(request);
} catch (DataAccessException e) {
    log.error("Database down", e);
    return ResponseEntity.status(503)
        .body("Service temporarily unavailable. Please try again.");
}
```

**5. Database Replication:**
- Master-Slave setup
- Write to master, read from slaves
- If master down, promote slave

**6. Monitoring & Alerts:**
- Health check endpoint
- Alerting when DB connections fail
- Automated recovery scripts"

---

### Q23: How would you optimize if 10,000 students access the menu at the same time?

**Answer:**
"I would implement caching and optimization:

**1. Application-Level Caching:**
```java
@Service
public class MenuService {
    
    @Cacheable(value = "dailyMenu", key = "#date")
    public List<ItemDaily> getMenuByDate(LocalDate date) {
        return itemDailyRepo.findByDate(date);
    }
    
    @CacheEvict(value = "dailyMenu", allEntries = true)
    public void updateMenu() {
        // Cache cleared when menu updated
    }
}
```

**2. Redis Caching:**
```java
// Store in Redis (in-memory)
redisTemplate.opsForValue().set("menu:" + date, menu, 1, TimeUnit.HOURS);

// Retrieve from Redis
List<ItemDaily> menu = redisTemplate.opsForValue().get("menu:" + date);
```

**3. Database Query Optimization:**
```java
// Before: N+1 query problem
List<ItemDaily> items = itemDailyRepo.findAll();  // 1 query
for (ItemDaily item : items) {
    item.getItemMaster().getName();  // N queries!
}

// After: Single query with JOIN FETCH
@Query("SELECT id FROM ItemDaily id JOIN FETCH id.itemMaster WHERE id.date = :date")
List<ItemDaily> findByDateWithItem(@Param("date") LocalDate date);
```

**4. Connection Pooling:**
```properties
spring.datasource.hikari.maximum-pool-size=50
spring.datasource.hikari.minimum-idle=10
```

**5. CDN for Images:**
- Store item images on CDN (CloudFront, Cloudinary)
- Reduce server load

**6. Load Balancing:**
```
                    Load Balancer
                         |
         ┌───────────────┼───────────────┐
         |               |               |
    Instance 1      Instance 2      Instance 3
         |               |               |
         └───────────────┴───────────────┘
                         |
                   MySQL Database
                         |
                    Redis Cache
```

**7. Pagination:**
```java
@GetMapping("/menu")
public Page<ItemDaily> getMenu(
    @RequestParam LocalDate date,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size
) {
    Pageable pageable = PageRequest.of(page, size);
    return menuService.getMenu(date, pageable);
}
```

**Expected Performance:**
- Without optimization: ~5000ms for 10,000 requests
- With caching: ~50ms for cached data
- 100x improvement!"

---

## 🤝 BEHAVIORAL QUESTIONS

### Q24: What was the most challenging part of this project?

**Answer:**
"The most challenging part was implementing atomic order transactions with concurrent access handling.

**Challenge:**
Multiple students could order the last item simultaneously, potentially causing:
1. Overselling (negative stock)
2. Incorrect wallet deductions
3. Data inconsistency

**Approach:**
1. **Research:** Studied transaction isolation levels and locking mechanisms

2. **Initial Solution:** Used `@Transactional` with default isolation
   - Problem: Still had race conditions with 2 concurrent orders

3. **Revised Solution:** Implemented `SERIALIZABLE` isolation
   - Completely isolated transactions
   - Performance trade-off acceptable

4. **Testing:** Simulated concurrent requests using JMeter
   - Verified no overselling occurs
   - Checked database consistency

**Learning:**
- Importance of transaction management in multi-user systems
- Balance between performance and correctness
- Testing concurrent scenarios is crucial

**Result:**
Successfully handled concurrent orders without data corruption, ensuring business logic integrity."

---

### Q25: How did you manage your time during the project?

**Answer:**
"I followed an iterative development approach with clear milestones:

**Week 1: Planning & Design**
- Created ER diagram, use cases
- Designed database schema
- Finalized tech stack
- Set up Git repository

**Week 2-3: Backend Development**
- Day 1-2: Entity classes, database setup
- Day 3-4: Repository layer, basic CRUD
- Day 5-7: Service layer, business logic
- Day 8-10: Security (JWT, BCrypt)
- Day 11-12: Controllers, API testing with Postman
- Day 13-14: Exception handling, validation

**Week 4-5: Frontend Development**
- Day 1-2: React setup, routing
- Day 3-5: Redux setup, state management
- Day 6-8: UI components (Material-UI)
- Day 9-10: API integration with Axios
- Day 11-12: Form validation, error handling
- Day 13-14: Styling, responsive design

**Week 6: Integration & Testing**
- Full-stack integration
- End-to-end testing
- Bug fixes
- Performance optimization

**Week 7: Documentation & Deployment**
- Code documentation
- User manual
- Presentation preparation

**Challenges:**
- Initially underestimated security implementation
- Adjusted timeline, worked extra hours
- Used weekends for catching up

**Tools:**
- GitHub for version control
- Trello for task tracking
- Daily progress notes"

---

### Q26: If you had more time, what would you add to this project?

**Answer:**
"Several enhancements I would implement:

**1. Real-time Features:**
- WebSocket integration for live order status updates
- Real-time stock updates (no page refresh needed)
- Push notifications when order ready

**2. Advanced Analytics:**
- Dashboard with charts (Chart.js, Recharts)
- Popular items analysis
- Revenue trends
- Student spending patterns
- Peak hour identification

**3. AI/ML Features:**
- Meal recommendations based on order history
- Predictive inventory (forecast demand)
- Price optimization
- Dietary preference suggestions

**4. Mobile Application:**
- React Native app for iOS/Android
- QR code scanning for order pickup
- Mobile payment integration

**5. Enhanced Security:**
- Refresh token mechanism
- Two-factor authentication for admin
- Rate limiting to prevent API abuse
- CAPTCHA for login

**6. Better UX:**
- Progressive Web App (PWA)
- Offline support
- Dark mode
- Multi-language support

**7. Integration:**
- Payment gateway (Razorpay/Stripe)
- Email notifications (order confirmation)
- SMS alerts
- Export reports (PDF/Excel)

**8. Performance:**
- Server-side caching (Redis)
- CDN for static assets
- Database indexing optimization
- Lazy loading for images

**9. Testing:**
- Unit tests (80%+ coverage)
- Integration tests
- E2E tests (Selenium)
- Load testing (JMeter)

**10. DevOps:**
- Docker containerization
- CI/CD pipeline (GitHub Actions)
- Kubernetes deployment
- Monitoring (Prometheus, Grafana)"

---

## 🎯 QUICK REFERENCE

### Most Important Topics to Revise

**Backend:**
1. Spring Boot architecture (Controller → Service → Repository)
2. JPA/Hibernate (entities, relationships, lazy/eager loading)
3. @Transactional (isolation levels, propagation)
4. Spring Security (JWT, filters, authentication flow)
5. REST API design (HTTP methods, status codes)
6. Exception handling (@RestControllerAdvice)

**Frontend:**
1. React fundamentals (components, JSX, props, state)
2. Hooks (useState, useEffect, useSelector, useDispatch)
3. Redux flow (actions, reducers, store)
4. Virtual DOM and reconciliation
5. React Router (protected routes)

**Database:**
1. Schema design (normalization, relationships)
2. Indexes and query optimization
3. Transaction management

**Security:**
1. JWT authentication flow
2. BCrypt password hashing
3. CORS configuration

**Project-Specific:**
1. Concurrent order handling
2. Inventory management
3. Wallet system
4. Daily menu reset scheduler

### Common Mistakes to Avoid

1. ❌ "I used Spring Boot because it's popular"
   ✅ "I used Spring Boot for auto-configuration, embedded server, and production-ready features"

2. ❌ "Redux stores data"
   ✅ "Redux manages application state in a predictable way using actions and reducers"

3. ❌ "JWT is more secure than sessions"
   ✅ "JWT is stateless and scalable, better for REST APIs and mobile apps"

4. ❌ "Normalization removes all redundancy"
   ✅ "Normalization reduces redundancy while maintaining data integrity and query performance"

5. ❌ "Indexes always improve performance"
   ✅ "Indexes improve SELECT performance but slow down INSERT/UPDATE operations"

---

## 💡 TIPS FOR INTERVIEW

1. **Start with the problem:**
   - Always explain why before what
   - "We had X problem, so I used Y solution"

2. **Use examples from your project:**
   - Don't give generic answers
   - "In my project, when placing an order..."

3. **Show trade-offs:**
   - "I used SERIALIZABLE isolation for correctness, accepting slight performance cost"
   - Demonstrates critical thinking

4. **Admit what you don't know:**
   - "I haven't used microservices yet, but I understand the concept"
   - Better than making up answers

5. **Be ready to code:**
   - Practice writing common algorithms
   - Know how to write basic SQL queries

6. **Ask clarifying questions:**
   - "Do you want me to explain the entire flow or focus on a specific part?"
   - Shows communication skills

7. **Practice the 2-minute project summary:**
   - Have a concise elevator pitch ready
   - Practice it until it flows naturally

---

Good luck with your interview preparation! 🚀
