# 3 main ways to test a Spring Boot application

---

## 🧪 JUnit, Mockito & Test-Driven Development (TDD) – A Developer’s Guide

This repository showcases how to test various layers of a Spring Boot application using **JUnit 5**, **Mockito**, and **TDD principles**.

---

### 🔍 Ways to Test a Spring Boot Application

There are **three main testing strategies** in Spring Boot:

| Type                       | Description                                                                                                 |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- |
| ✅ **Unit Testing**         | Tests **one class in isolation** (e.g., service, controller) using mocks.                                   |
| 🔄 **Integration Testing** | Tests **multiple layers together** (e.g., controller + service + DB).                                       |
| 🌐 **End-to-End (E2E)**    | Tests **full application flow** (from API to DB) using tools like Postman, `TestRestTemplate`, or Selenium. |

---

### ✅ Test-Driven Development (TDD)

**TDD is a development process where tests guide the code:**

1. **Write the test first** → it fails.
2. **Write just enough code** to make the test pass.
3. **Refactor** the code for quality.

TDD ensures better design, fewer bugs, and more confidence in refactoring.

---

### 🧱 How to Test Each Layer in Spring Boot

| Layer           | Testing Approach                                                  |
| --------------- | ----------------------------------------------------------------- |
| **Controller**  | Use `@WebMvcTest` and `MockMvc` to test REST APIs                 |
| **Service**     | Use `@Mock` and `@InjectMocks` to isolate and test business logic |
| **ServiceImpl** | Same as service; mock DAO/repo dependencies                       |
| **DTO**         | Unit test mapping/conversion logic (if present)                   |
| **DAO**         | Tested via integration tests or mocking database responses        |
| **Repository**  | Use `@DataJpaTest` with H2 or other in-memory DBs                 |
| **Entity**      | Only test if custom logic exists (e.g., validation methods)       |


These are general testing practices in a **Spring Boot enterprise app** using **JUnit 5 and Mockito**, especially in a **Test-Driven Development (TDD)** setup.

---

### 🔹 1. **ServiceImpl (Service Implementation Class)**

* ✅ **What to test?** Business logic inside the service methods.
* 🧪 **How to test?** Unit tests with **Mockito** by mocking dependencies (e.g., DAO or repository).
* 💡 Example:

  ```java
  @Mock
  private UserRepository userRepository;

  @InjectMocks
  private UserServiceImpl userService;
  ```

---

### 🔹 2. **DTO (Data Transfer Object)**

* ✅ **What to test?** Only if DTOs contain **conversion logic** (like mapping between entity ↔ DTO).
* 🧪 **How to test?** Unit test the conversion functions or utility classes.
* ❌ If DTOs are just POJOs with getters/setters, no test needed.

---

### 🔹 3. **DAO (Data Access Object)**

* ✅ **What to test?** Normally tested via **integration tests**, not unit tests.
* 🧪 **Why?** DAOs are too close to the database; better to test them by connecting to an in-memory DB (like H2) or mocking DB behavior if needed.

---

### 🔹 4. **Repository (Spring Data JPA)**

* ✅ **What to test?** Custom queries or any custom logic.
* 🧪 **How to test?** Use `@DataJpaTest` with **H2 in-memory database** for fast, isolated testing of JPA behavior.
* 💡 Spring automatically configures an embedded DB during `@DataJpaTest`.

---

### 🔹 5. **Entity**

* ✅ **What to test?** Only if the entity has custom methods or validation logic.
* ❌ No need to test simple `@Entity` classes that only have fields and annotations.

---

### Summary Table:

| Layer           | Do You Test? | How?                     |
| --------------- | ------------ | ------------------------ |
| **ServiceImpl** | ✅ Yes        | Unit test with mocks     |
| **DTO**         | ❌/✅ Maybe    | Test if it has logic     |
| **DAO**         | ✅ Yes        | Integration test/mock DB |
| **Repository**  | ✅ Yes        | `@DataJpaTest` + H2      |
| **Entity**      | ❌/✅ Maybe    | Test if logic is present |

---

Great! You're testing a **Spring Boot application** with a **PostgreSQL database**, and your focus is on how to **test the DAO and Repository layers**.

Let me break this down clearly:

---

## ✅ What You’re Testing:

* **DAO Layer** (Custom interfaces or classes between service and repository)
* **Repository Layer** (Spring Data JPA interfaces like `UserRepository extends JpaRepository<User, Long>`)

---

## 🧪 Goal:

Test these layers in a way that **interacts with the actual database** — or a close simulation — to make sure queries work correctly.

---

## ✅ 3 Options to Test DAO/Repository in Spring Boot with PostgreSQL

---

### 🔹 **Option 1: `@DataJpaTest` + H2 In-Memory DB** (Quick but not PostgreSQL-specific)

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.ANY) // Replaces PostgreSQL with H2
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void testFindByEmail() {
        User user = new User("john", "john@example.com");
        userRepository.save(user);

        Optional<User> found = userRepository.findByEmail("john@example.com");
        assertTrue(found.isPresent());
    }
}
```

🔸 Use this if you’re not relying on PostgreSQL-specific SQL features.

---

### 🔹 **Option 2: `@DataJpaTest` + Real PostgreSQL DB** (Best for production-like behavior)

#### ✅ Steps:

1. Create a **test PostgreSQL DB** (`myapp_test`).
2. Add a **`src/test/resources/application-test.yml`**:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/myapp_test
    username: testuser
    password: testpass
  jpa:
    hibernate:
      ddl-auto: update
  sql:
    init:
      mode: always
```

3. Annotate your test class:

```java
@DataJpaTest
@ActiveProfiles("test")
@AutoConfigureTestDatabase(replace = Replace.NONE) // Use real PostgreSQL
public class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;

    @Test
    void testSaveAndFindUser() {
        User user = new User("john", "john@example.com");
        userRepository.save(user);

        Optional<User> found = userRepository.findByEmail("john@example.com");
        assertTrue(found.isPresent());
    }
}
```

✅ PostgreSQL-specific behavior (e.g., case sensitivity, JSONB fields) will now be properly tested.

---

### 🔹 **Option 3: Testcontainers (Spin up PostgreSQL in Docker for tests)**

* **Most accurate & clean** for CI/CD pipelines
* Isolated database for each test run

#### Example:

```java
@Testcontainers
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
public class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
        .withDatabaseName("testdb")
        .withUsername("user")
        .withPassword("pass");

    @DynamicPropertySource
    static void setDatasourceProps(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void testPostgresQuery() {
        User user = new User("john", "john@example.com");
        userRepository.save(user);
        assertEquals(1, userRepository.count());
    }
}
```

---

## ✅ Summary Table

| Approach                    | Accuracy | Speed     | Needs Docker? | Good for CI? |
| --------------------------- | -------- | --------- | ------------- | ------------ |
| `@DataJpaTest` + H2         | ❌ Medium | ✅ Fast    | ❌ No          | ✅ Yes        |
| `@DataJpaTest` + PostgreSQL | ✅ High   | ❌ Slower  | ❌ No          | ✅ Yes        |
| Testcontainers              | ✅ Best   | ❌ Slowest | ✅ Yes         | ✅ Best       |

---

## 🟢 My Suggestion for You:

Since you're already using **PostgreSQL**, use this:

* `@DataJpaTest` + PostgreSQL + `application-test.yml` + `@ActiveProfiles("test")`
* OR Testcontainers if you want CI/CD-ready isolation.

---



---

### 📘 JUnit 5 – Essentials

* Test lifecycle: `@Test`, `@BeforeEach`, `@AfterEach`
* Descriptions: `@DisplayName`
* Assertions: `assertEquals()`, `assertTrue()`, etc.
* Structure: `@Nested` for grouping, `@ParameterizedTest` for dynamic inputs

---

### 🎭 Mockito – Mocking Framework

* Annotations: `@Mock`, `@InjectMocks`
* Behavior: `when(...).thenReturn(...)`, `verify(...)`, `doThrow(...)`
* Concepts:

  * **Mock** – fake object for isolation
  * **Stub** – fake data or behavior
  * **Spy** – partial mock of real object

---

### ✅ Best Practices

* Focus on **one behavior per test**
* Name tests descriptively (`shouldReturnData_whenConditionMet()`)
* Test **positive, negative**, and **edge cases**
* Only test **public methods**
* Keep tests **fast**, **independent**, and **repeatable**

---

### 🧰 Useful Tools

* **JUnit 5** – Main testing framework
* **Mockito** – For mocking dependencies
* **JaCoCo** – Code coverage reports
* **AssertJ / Hamcrest** – Optional fluent assertion libraries

---

This guide gives you the **fundamentals, strategies, and tools** to effectively write tests and apply **Test-Driven Development** in any Spring Boot project.
Browse this repo for practical examples and real-world testing layers.

---

---

## 🧪 Step-by-Step: Testing Spring Boot with JUnit + Mockito

### ✅ Step 01: Create a Basic Hello World Controller

#### 🧩 1. Create a Controller

```java
@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello, World!";
    }
}
```

#### 🔗 2. Run the App

Visit: `http://localhost:8080/api/hello`
Output:

```
Hello, World!
```

---

### ✅ Step 02: Unit Test with `MockMvc`

#### 🧪 1. Test HelloController

```java
@WebMvcTest(HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testSayHello() throws Exception {
        mockMvc.perform(get("/api/hello"))
               .andExpect(status().isOk())
               .andExpect(content().string("Hello, World!"));
    }
}
```

#### 🧠 Explanation:

| Annotation / Method  | Purpose                              |
| -------------------- | ------------------------------------ |
| `@WebMvcTest`        | Loads only the controller layer      |
| `MockMvc`            | Simulates HTTP requests              |
| `.perform(get(...))` | Sends a GET request                  |
| `.andExpect(...)`    | Verifies status and response content |

---

## ✅ Step 03: Create Service and Controller with Dependency

### 🧩 1. Service Class

```java
@Service
public class ItemService {

    public String getItemName() {
        return "Laptop";
    }
}
```

### 🧩 2. Controller Class

```java
@RestController
@RequestMapping("/api/items")
public class ItemController {

    private final ItemService itemService;

    public ItemController(ItemService itemService) {
        this.itemService = itemService;
    }

    @GetMapping("/name")
    public String getItemName() {
        return itemService.getItemName();
    }
}
```

### 🔗 3. Run & Test Endpoint

Visit: `http://localhost:8080/api/items/name`
Output:

```
Laptop
```

---

## ✅ Step 04: Unit Test with Mocked Service (Mockito)

### 🧪 1. Test ItemController

```java
@WebMvcTest(ItemController.class)
public class ItemControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ItemService itemService;

    @Test
    public void testGetItemName() throws Exception {
        Mockito.when(itemService.getItemName()).thenReturn("Laptop");

        mockMvc.perform(get("/api/items/name"))
               .andExpect(status().isOk())
               .andExpect(content().string("Laptop"));
    }
}
```

#### 🧠 Key Concepts:

| Annotation / Method                 | Purpose                             |
| ----------------------------------- | ----------------------------------- |
| `@MockBean`                         | Injects mock of `ItemService`       |
| `Mockito.when(...).thenReturn(...)` | Defines mock behavior               |
| `MockMvc`                           | Sends request to controller         |
| `jsonPath(...)` (see below)         | Used for JSON response verification |

---

### ✅ Bonus: JSON Response & Test

#### 🧩 Controller Example Returning JSON

```java
@GetMapping("/info")
public ResponseEntity<Map<String, String>> getItemInfo() {
    Map<String, String> response = Map.of("name", "Laptop", "type", "Electronics");
    return ResponseEntity.ok(response);
}
```

#### 🧪 Test JSON Response

```java
@Test
public void testGetItemInfo() throws Exception {
    Map<String, String> mockResponse = Map.of("name", "Laptop", "type", "Electronics");
    Mockito.when(itemService.getItemInfo()).thenReturn(mockResponse);

    mockMvc.perform(get("/api/items/info"))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$.name").value("Laptop"))
           .andExpect(jsonPath("$.type").value("Electronics"));
}
```

#### 🧠 Notes:

* `jsonPath("$.name")` → access key inside JSON.
* Works well for validating nested JSON responses too.

---


 Below is a full example including:

* `ItemService` + `ItemServiceImpl` (from before)
* `ItemController` (with CRUD using ResponseEntity)
* **Mockito unit tests for the controller**, mocking the service layer

---

## 1. `ItemController` with CRUD endpoints

```java
@RestController
@RequestMapping("/api/items")
public class ItemController {

    private final ItemService itemService;

    public ItemController(ItemService itemService) {
        this.itemService = itemService;
    }

    @PostMapping
    public ResponseEntity<Item> createItem(@RequestBody Item item) {
        Item created = itemService.createItem(item);
        return new ResponseEntity<>(created, HttpStatus.CREATED);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Item> getItemById(@PathVariable Long id) {
        return itemService.getItemById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Item> updateItem(@PathVariable Long id, @RequestBody Item item) {
        return itemService.updateItem(id, item)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteItem(@PathVariable Long id) {
        boolean deleted = itemService.deleteItem(id);
        if (deleted) {
            return ResponseEntity.noContent().build();
        } else {
            return ResponseEntity.notFound().build();
        }
    }
}
```

---

## 2. Mockito Unit Tests for `ItemController`

```java
@WebMvcTest(ItemController.class)
public class ItemControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ItemService itemService;

    private final ObjectMapper objectMapper = new ObjectMapper();

    @Test
    public void testCreateItem() throws Exception {
        Item item = new Item(1L, "Phone");
        Mockito.when(itemService.createItem(Mockito.any())).thenReturn(item);

        mockMvc.perform(post("/api/items")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(item)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("Phone"));
    }

    @Test
    public void testGetItemById_found() throws Exception {
        Item item = new Item(1L, "Laptop");
        Mockito.when(itemService.getItemById(1L)).thenReturn(Optional.of(item));

        mockMvc.perform(get("/api/items/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("Laptop"));
    }

    @Test
    public void testGetItemById_notFound() throws Exception {
        Mockito.when(itemService.getItemById(1L)).thenReturn(Optional.empty());

        mockMvc.perform(get("/api/items/1"))
                .andExpect(status().isNotFound());
    }

    @Test
    public void testUpdateItem_found() throws Exception {
        Item updatedItem = new Item(1L, "Tablet");
        Mockito.when(itemService.updateItem(Mockito.eq(1L), Mockito.any())).thenReturn(Optional.of(updatedItem));

        mockMvc.perform(put("/api/items/1")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updatedItem)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("Tablet"));
    }

    @Test
    public void testUpdateItem_notFound() throws Exception {
        Item updatedItem = new Item(1L, "Tablet");
        Mockito.when(itemService.updateItem(Mockito.eq(1L), Mockito.any())).thenReturn(Optional.empty());

        mockMvc.perform(put("/api/items/1")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updatedItem)))
                .andExpect(status().isNotFound());
    }

    @Test
    public void testDeleteItem_found() throws Exception {
        Mockito.when(itemService.deleteItem(1L)).thenReturn(true);

        mockMvc.perform(delete("/api/items/1"))
                .andExpect(status().isNoContent());
    }

    @Test
    public void testDeleteItem_notFound() throws Exception {
        Mockito.when(itemService.deleteItem(1L)).thenReturn(false);

        mockMvc.perform(delete("/api/items/1"))
                .andExpect(status().isNotFound());
    }
}
```

---
 If you're using annotations like **Lombok** to reduce boilerplate in your `Item` class, here’s how you can rewrite it with Lombok annotations:

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data               // Generates getters, setters, toString, equals, and hashCode
@NoArgsConstructor  // Generates no-arg constructor
@AllArgsConstructor // Generates all-args constructor
public class Item {
    private Long id;
    private String name;
}
```

---

### What you get with Lombok annotations:

* `@Data` bundles:

  * Getters/setters for all fields
  * `toString()`
  * `equals()` and `hashCode()`
* `@NoArgsConstructor` adds a default constructor
* `@AllArgsConstructor` adds a constructor with all fields

---

**Make sure** you have Lombok added as a dependency in your `pom.xml` or `build.gradle` and your IDE supports it.

---

Below is a simple example of **ItemService interface**, its **implementation**, and **Mockito unit tests** for the service using Java 8 features like `Optional` and lambdas.

---

### 1. **ItemService Interface**

```java
import java.util.List;
import java.util.Optional;

public interface ItemService {

    Item createItem(Item item);

    Optional<Item> getItemById(Long id);

    List<Item> getAllItems();

    Optional<Item> updateItem(Long id, Item item);

    boolean deleteItem(Long id);
}
```

---

### 2. **ItemServiceImpl Class**

```java
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Service
public class ItemServiceImpl implements ItemService {

    private final List<Item> items = new ArrayList<>();

    @Override
    public Item createItem(Item item) {
        items.add(item);
        return item;
    }

    @Override
    public Optional<Item> getItemById(Long id) {
        return items.stream()
                .filter(i -> i.getId().equals(id))
                .findFirst();
    }

    @Override
    public List<Item> getAllItems() {
        return new ArrayList<>(items);
    }

    @Override
    public Optional<Item> updateItem(Long id, Item item) {
        return getItemById(id).map(existingItem -> {
            existingItem.setName(item.getName());
            return existingItem;
        });
    }

    @Override
    public boolean deleteItem(Long id) {
        return items.removeIf(i -> i.getId().equals(id));
    }
}
```

---

### 3. **Mockito Unit Tests for ItemServiceImpl**

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

public class ItemServiceImplTest {

    private ItemServiceImpl itemService;

    @BeforeEach
    public void setUp() {
        itemService = new ItemServiceImpl();
    }

    @Test
    public void testCreateItem() {
        Item item = new Item(1L, "Phone");
        Item created = itemService.createItem(item);
        assertEquals(item, created);
        assertTrue(itemService.getAllItems().contains(item));
    }

    @Test
    public void testGetItemById_found() {
        Item item = new Item(2L, "Laptop");
        itemService.createItem(item);

        Optional<Item> found = itemService.getItemById(2L);
        assertTrue(found.isPresent());
        assertEquals("Laptop", found.get().getName());
    }

    @Test
    public void testGetItemById_notFound() {
        Optional<Item> found = itemService.getItemById(999L);
        assertFalse(found.isPresent());
    }

    @Test
    public void testUpdateItem_found() {
        Item item = new Item(3L, "Tablet");
        itemService.createItem(item);

        Item updateInfo = new Item(null, "Updated Tablet");
        Optional<Item> updated = itemService.updateItem(3L, updateInfo);

        assertTrue(updated.isPresent());
        assertEquals("Updated Tablet", updated.get().getName());
    }

    @Test
    public void testUpdateItem_notFound() {
        Item updateInfo = new Item(null, "Updated Tablet");
        Optional<Item> updated = itemService.updateItem(999L, updateInfo);
        assertFalse(updated.isPresent());
    }

    @Test
    public void testDeleteItem_found() {
        Item item = new Item(4L, "Monitor");
        itemService.createItem(item);

        boolean deleted = itemService.deleteItem(4L);
        assertTrue(deleted);
        assertFalse(itemService.getItemById(4L).isPresent());
    }

    @Test
    public void testDeleteItem_notFound() {
        boolean deleted = itemService.deleteItem(999L);
        assertFalse(deleted);
    }
}
```

---

---

### Summary:

* Service methods return `Optional<Item>` for safe null handling.
* Java 8 Streams used to find and filter items.
* Mockito is not needed here since this is testing a simple concrete class without dependencies.
* You can easily mock this service when testing controllers.

---

**level up your testing** by using **JUnit 5 advanced features** like:

* `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll`
* Assertions: `assertEquals`, `assertTrue`, `assertNull`, `assertThrows`
* `@Nested` test classes
* `@ParameterizedTest` with different input values

Let’s go step-by-step using your `ItemServiceImpl` example so you see it all in action.

---

## ✅ Full Example: Advanced JUnit 5 Testing for `ItemServiceImpl`

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

import java.util.Optional;
import java.util.stream.Stream;

public class ItemServiceImplTest {

    private ItemServiceImpl itemService;

    @BeforeAll
    static void setupAll() {
        System.out.println("🔧 Setup before all tests");
    }

    @AfterAll
    static void tearDownAll() {
        System.out.println("🧹 Cleanup after all tests");
    }

    @BeforeEach
    void setup() {
        itemService = new ItemServiceImpl();
        System.out.println("✅ Setup before each test");
    }

    @AfterEach
    void tearDown() {
        System.out.println("❌ Tear down after each test");
    }

    @Test
    void testCreateItem() {
        Item item = new Item(1L, "Phone");
        Item created = itemService.createItem(item);

        assertEquals(item, created);
        assertTrue(itemService.getAllItems().contains(item));
    }

    @Test
    void testGetItemById_returnsItem() {
        itemService.createItem(new Item(2L, "Tablet"));
        Optional<Item> result = itemService.getItemById(2L);

        assertTrue(result.isPresent());
        assertEquals("Tablet", result.get().getName());
    }

    @Test
    void testGetItemById_notFound() {
        Optional<Item> result = itemService.getItemById(999L);
        assertTrue(result.isEmpty());
        assertNull(result.orElse(null));
    }

    @Test
    void testUpdateItem_throwsExceptionIfNull() {
        assertThrows(NullPointerException.class, () -> {
            itemService.updateItem(null, null);
        });
    }

    @Nested
    class DeleteTests {

        @Test
        void testDeleteItem_exists() {
            itemService.createItem(new Item(3L, "Laptop"));
            boolean deleted = itemService.deleteItem(3L);

            assertTrue(deleted);
        }

        @Test
        void testDeleteItem_notExists() {
            boolean deleted = itemService.deleteItem(999L);
            assertFalse(deleted);
        }
    }

    @ParameterizedTest
    @MethodSource("provideItems")
    void testParameterizedCreateItem(Item item) {
        Item created = itemService.createItem(item);
        assertNotNull(created);
        assertEquals(item.getName(), created.getName());
    }

    static Stream<Item> provideItems() {
        return Stream.of(
                new Item(101L, "Mouse"),
                new Item(102L, "Keyboard"),
                new Item(103L, "Monitor")
        );
    }
}
```

---

## 🔍 Feature Breakdown:

| Feature                                                                                    | Purpose                                            |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------- |
| `@BeforeAll` / `@AfterAll`                                                                 | Run once before/after all tests (must be `static`) |
| `@BeforeEach` / `@AfterEach`                                                               | Run before/after each test                         |
| `@Test`                                                                                    | Standard test case                                 |
| `@Nested`                                                                                  | Group related tests for better readability         |
| `@ParameterizedTest`                                                                       | Run same test logic with different input data      |
| `@MethodSource`                                                                            | Feeds parameters from a method                     |
| `assertEquals`, `assertTrue`, `assertFalse`, `assertNull`, `assertNotNull`, `assertThrows` | Assertions                                         |

---

### ✅ Requirements:

* **JUnit 5 (Jupiter)** in `pom.xml` or `build.gradle`
* For `@MethodSource`: method must be `static` or in the same test class

---
Here's a short explanation:

> 🔹 **`@MethodSource`** needs the data-provider method to be:

* `static` ✅ **if it's in a different class**
* Not static ❌ \*\*only if it's in the ***same test class***

---

### ✅ Example in **same class** (non-static works):

```java
@ParameterizedTest
@MethodSource("itemProvider")
void testItem(Item item) {
    // test logic
}

Stream<Item> itemProvider() {
    return Stream.of(new Item(1L, "A"), new Item(2L, "B"));
}
```

---

### ✅ Example from **another class** (must be static):

```java
@ParameterizedTest
@MethodSource("com.example.TestData#itemProvider")
void testItem(Item item) {
    // test logic
}
```

```java
// In TestData.java
public class TestData {
    public static Stream<Item> itemProvider() {
        return Stream.of(new Item(1L, "A"), new Item(2L, "B"));
    }
}
```

In short:

* 🔹 **Same class?** → static or non-static method is fine
* 🔹 **Other class?** → method **must be static**


---

Here's how to apply **advanced JUnit 5 features** like `@BeforeEach`, `@AfterEach`, `@Nested`, `@ParameterizedTest`, `@MethodSource`, and all key **assertions** to **controller unit testing** using **MockMvc** and **Mockito**.

We’ll base it on your `ItemController` example.

---

## ✅ Advanced Controller Test with MockMvc & JUnit 5 Features

```java
@WebMvcTest(ItemController.class)
class ItemControllerAdvancedTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ItemService itemService;

    private final ObjectMapper objectMapper = new ObjectMapper();

    @BeforeAll
    static void beforeAllTests() {
        System.out.println("🌐 Starting ItemController tests...");
    }

    @AfterAll
    static void afterAllTests() {
        System.out.println("🧹 Finished all controller tests.");
    }

    @BeforeEach
    void setupEach() {
        System.out.println("🚀 Before each test");
    }

    @AfterEach
    void tearDownEach() {
        System.out.println("🛑 After each test");
    }

    @Test
    void testGetItemById_found() throws Exception {
        Item item = new Item(1L, "Monitor");
        Mockito.when(itemService.getItemById(1L)).thenReturn(Optional.of(item));

        mockMvc.perform(get("/api/items/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("Monitor"));
    }

    @Test
    void testGetItemById_notFound() throws Exception {
        Mockito.when(itemService.getItemById(99L)).thenReturn(Optional.empty());

        mockMvc.perform(get("/api/items/99"))
                .andExpect(status().isNotFound());
    }

    @Test
    void testCreateItem_asserts() throws Exception {
        Item item = new Item(10L, "Phone");
        Mockito.when(itemService.createItem(Mockito.any())).thenReturn(item);

        mockMvc.perform(post("/api/items")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(item)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.name").value("Phone"));

        assertNotNull(item.getId());
        assertEquals("Phone", item.getName());
        assertTrue(item.getId() > 0);
    }

    @Nested
    class UpdateItemTests {

        @Test
        void testUpdateItem_success() throws Exception {
            Item updated = new Item(2L, "Updated Laptop");
            Mockito.when(itemService.updateItem(Mockito.eq(2L), Mockito.any()))
                   .thenReturn(Optional.of(updated));

            mockMvc.perform(put("/api/items/2")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(updated)))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.name").value("Updated Laptop"));
        }

        @Test
        void testUpdateItem_notFound() throws Exception {
            Item updated = new Item(999L, "Ghost");
            Mockito.when(itemService.updateItem(Mockito.eq(999L), Mockito.any()))
                   .thenReturn(Optional.empty());

            mockMvc.perform(put("/api/items/999")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(updated)))
                    .andExpect(status().isNotFound());
        }
    }

    @ParameterizedTest(name = "Delete item with ID = {0}")
    @MethodSource("provideIdsForDeletion")
    void testDeleteItem(Long id, boolean expectedResult) throws Exception {
        Mockito.when(itemService.deleteItem(id)).thenReturn(expectedResult);

        mockMvc.perform(delete("/api/items/" + id))
                .andExpect(status().is(expectedResult ? 204 : 404));
    }

    static Stream<Arguments> provideIdsForDeletion() {
        return Stream.of(
                Arguments.of(1L, true),
                Arguments.of(2L, true),
                Arguments.of(999L, false)
        );
    }

    @Test
    void testCreateItem_throwException() throws Exception {
        Mockito.when(itemService.createItem(Mockito.any()))
               .thenThrow(new RuntimeException("Internal error"));

        mockMvc.perform(post("/api/items")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(new Item(null, "Error"))))
                .andExpect(status().isInternalServerError()); // Assuming global exception handler
    }
}
```

---

## 🔍 What This Test Class Demonstrates:

| Feature                                       | Usage Description                  |
| --------------------------------------------- | ---------------------------------- |
| `@WebMvcTest`                                 | Loads controller layer only        |
| `@BeforeAll`, `@AfterAll`                     | Logs before/after all tests        |
| `@BeforeEach`, `@AfterEach`                   | Logs before/after each test        |
| `@Test`                                       | Regular unit test                  |
| `@Nested`                                     | Organizes related tests            |
| `@ParameterizedTest`                          | Repeats test with different inputs |
| `@MethodSource`                               | Provides custom test data          |
| `assertEquals`, `assertTrue`, `assertNotNull` | Classic assertions                 |
| `assertThrows` (indirectly tested)            | Exception test case                |
| `jsonPath()`                                  | Validates JSON response            |
| `Mockito.when(...)`                           | Mocks service methods              |
| `.andExpect(...)`                             | Verifies response status/body      |

---


# Integration Testing

Let's now **convert everything into a full integration test** using `@SpringBootTest`, covering the **controller, service, and global exception handler**, with **real HTTP request/response** using `TestRestTemplate`.

---

## ✅ Step-by-Step: Spring Boot Integration Test

---
Here's a **reframed version** of your explanation **before and after integration testing** to clearly show the transition and improvements introduced with a full integration test setup using `@SpringBootTest`, `MockMvc`, and global exception handling.

---

### 🔄 **Before Integration Testing (Unit & Web Layer Tests Only)**

* ✅ Tests were limited to **individual layers** like the controller or service.
* ✅ Used `@WebMvcTest` with `MockMvc` for controller tests only.
* ❌ **Service** and **repository layers** were **mocked**, not tested together.
* ❌ Global exception handler was not triggered fully in real request flow.
* ❌ No real application context or full HTTP request lifecycle.

---

### ✅ **After: Full Spring Boot Integration Testing**

We now use `@SpringBootTest` to launch the **entire Spring Boot context**, testing real beans and actual HTTP flow from controller to service to exception handler.

---

### ✅ **Step-by-Step Integration Test Setup**

#### 1. Enable Integration Testing with `@SpringBootTest`

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class ItemControllerIntegrationTest {
    @Autowired private MockMvc mockMvc;
    @Autowired private ItemService itemService;
    @Autowired private ObjectMapper objectMapper;

    @BeforeEach
    void setup() {
        itemService.createItem(new Item(100L, "Keyboard"));
    }
```

#### 2. Integration Test Cases

```java
    @Test
    void testGetItemById_success() throws Exception {
        mockMvc.perform(get("/api/items/100"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("Keyboard"));
    }

    @Test
    void testGetItemById_notFound_shouldInvokeGlobalExceptionHandler() throws Exception {
        mockMvc.perform(get("/api/items/999"))
               .andExpect(status().isNotFound())
               .andExpect(jsonPath("$.message").value("Item with ID 999 not found"))
               .andExpect(jsonPath("$.status").value(404))
               .andExpect(jsonPath("$.error").value("Not Found"));
    }

    @Test
    void testCreateItem_success() throws Exception {
        Item newItem = new Item(200L, "Monitor");

        mockMvc.perform(post("/api/items")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(newItem)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(200L))
                .andExpect(jsonPath("$.name").value("Monitor"));
    }

    @Test
    void testDeleteItem_success() throws Exception {
        itemService.createItem(new Item(300L, "Mouse"));

        mockMvc.perform(delete("/api/items/300"))
               .andExpect(status().isNoContent());
    }

    @Test
    void testDeleteItem_notFound_shouldReturn404() throws Exception {
        mockMvc.perform(delete("/api/items/9876"))
               .andExpect(status().isNotFound())
               .andExpect(jsonPath("$.message").value("Item with ID 9876 not found"));
    }
}
```

---

### ✅ 3. Update Service Layer to Throw Exceptions

To trigger the `@ControllerAdvice`, ensure exceptions are thrown:

```java
@Override
public Optional<Item> getItemById(Long id) {
    return items.stream()
                .filter(i -> i.getId().equals(id))
                .findFirst();
}

@Override
public Optional<Item> updateItem(Long id, Item item) {
    return getItemById(id).map(existing -> {
        existing.setName(item.getName());
        return existing;
    });
}

@Override
public boolean deleteItem(Long id) {
    boolean removed = items.removeIf(i -> i.getId().equals(id));
    if (!removed) {
        throw new ItemNotFoundException("Item with ID " + id + " not found");
    }
    return true;
}
```

---

### ✅ 4. Global Exception Handler

Your existing `@RestControllerAdvice` automatically handles exceptions during full integration tests—no changes needed:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ItemNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ItemNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(404, "Not Found", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    // other exception handlers...
}
```

---

### ✅ Final Outcome

* ✅ **Real HTTP requests** tested via `MockMvc` within Spring context.
* ✅ Covers **controller + service + exception handler** together.
* ✅ Ensures **end-to-end behavior** is validated.
* ✅ Automatically verifies **error responses** from global exception handler.

Let me know if you want the **same style** for other layers or test strategies like `TestRestTemplate` or database-backed tests (`@DataJpaTest`).

---

### ✅ Final Tips

| Annotation               | Purpose                                   |
| ------------------------ | ----------------------------------------- |
| `@SpringBootTest`        | Boots full application context            |
| `@AutoConfigureMockMvc`  | Auto-wires `MockMvc` for full-stack tests |
| `@BeforeEach`            | Setup preconditions for tests             |
| `MockMvc` + `jsonPath()` | Assert HTTP status, body, structure       |

---

Perfect! Let’s now upgrade your project with:

---

## ✅ **1. H2 + Spring Data JPA Integration**

### ✅ 2. `@Valid` Validation with Global Exception Handling

### ✅ 3. Integration Tests with Validation and H2

---

## ✅ STEP 1: Add H2 + Spring Data JPA

### ✅ `pom.xml` dependencies:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

---

## ✅ STEP 2: Add Repository

```java
public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

---

## ✅ STEP 3: Update Entity with JPA + Validation

```java
import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.*;

@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Name is required")
    private String name;
}
```

---

## ✅ STEP 4: Update Service Layer

```java
@Service
public class ItemServiceImpl implements ItemService {

    private final ItemRepository repo;

    public ItemServiceImpl(ItemRepository repo) {
        this.repo = repo;
    }

    @Override
    public Item createItem(Item item) {
        return repo.save(item);
    }

    @Override
    public Optional<Item> getItemById(Long id) {
        return repo.findById(id);
    }

    @Override
    public List<Item> getAllItems() {
        return repo.findAll();
    }

    @Override
    public Optional<Item> updateItem(Long id, Item item) {
        return repo.findById(id).map(existing -> {
            existing.setName(item.getName());
            return repo.save(existing);
        });
    }

    @Override
    public boolean deleteItem(Long id) {
        if (!repo.existsById(id)) {
            throw new ItemNotFoundException("Item with ID " + id + " not found");
        }
        repo.deleteById(id);
        return true;
    }
}
```

---

## ✅ STEP 5: Update Controller to Use `@Valid`

```java
@PostMapping
public ResponseEntity<Item> createItem(@Valid @RequestBody Item item) {
    Item created = itemService.createItem(item);
    return new ResponseEntity<>(created, HttpStatus.CREATED);
}
```

---

## ✅ STEP 6: Update `GlobalExceptionHandler` for Validation

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<Object> handleValidationErrors(MethodArgumentNotValidException ex) {
    Map<String, String> errors = new HashMap<>();
    ex.getBindingResult().getFieldErrors().forEach(err ->
        errors.put(err.getField(), err.getDefaultMessage())
    );

    Map<String, Object> body = new HashMap<>();
    body.put("timestamp", LocalDateTime.now());
    body.put("status", 400);
    body.put("error", "Bad Request");
    body.put("message", "Validation failed");
    body.put("errors", errors);

    return new ResponseEntity<>(body, HttpStatus.BAD_REQUEST);
}
```

---

## ✅ STEP 7: Integration Test (H2 + Validation + Controller)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class ItemControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @Order(1)
    void testCreateItem_success() throws Exception {
        Item item = new Item(null, "Smart Watch");

        mockMvc.perform(post("/api/items")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(item)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").exists());
    }

    @Test
    @Order(2)
    void testCreateItem_validationFailure() throws Exception {
        Item item = new Item(null, ""); // empty name

        mockMvc.perform(post("/api/items")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(item)))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.errors.name").value("Name is required"));
    }

    @Test
    @Order(3)
    void testGetItemById_notFound() throws Exception {
        mockMvc.perform(get("/api/items/999"))
                .andExpect(status().isNotFound())
                .andExpect(jsonPath("$.message").value("Item with ID 999 not found"));
    }
}
```



Great! You're testing a **Spring Boot application** with a **PostgreSQL database**, and your focus is on how to **test the DAO and Repository layers**.

Let me break this down clearly:

---

## ✅ What You’re Testing:

* **DAO Layer** (Custom interfaces or classes between service and repository)
* **Repository Layer** (Spring Data JPA interfaces like `UserRepository extends JpaRepository<User, Long>`)

---

## 🧪 Goal:

Test these layers in a way that **interacts with the actual database** — or a close simulation — to make sure queries work correctly.

---

## ✅ 3 Options to Test DAO/Repository in Spring Boot with PostgreSQL

---

### 🔹 **Option 1: `@DataJpaTest` + H2 In-Memory DB** (Quick but not PostgreSQL-specific)

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.ANY) // Replaces PostgreSQL with H2
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void testFindByEmail() {
        User user = new User("john", "john@example.com");
        userRepository.save(user);

        Optional<User> found = userRepository.findByEmail("john@example.com");
        assertTrue(found.isPresent());
    }
}
```

🔸 Use this if you’re not relying on PostgreSQL-specific SQL features.

---

### 🔹 **Option 2: `@DataJpaTest` + Real PostgreSQL DB** (Best for production-like behavior)

#### ✅ Steps:

1. Create a **test PostgreSQL DB** (`myapp_test`).
2. Add a **`src/test/resources/application-test.yml`**:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/myapp_test
    username: testuser
    password: testpass
  jpa:
    hibernate:
      ddl-auto: update
  sql:
    init:
      mode: always
```

3. Annotate your test class:

```java
@DataJpaTest
@ActiveProfiles("test")
@AutoConfigureTestDatabase(replace = Replace.NONE) // Use real PostgreSQL
public class UserRepositoryTest {
    @Autowired
    private UserRepository userRepository;

    @Test
    void testSaveAndFindUser() {
        User user = new User("john", "john@example.com");
        userRepository.save(user);

        Optional<User> found = userRepository.findByEmail("john@example.com");
        assertTrue(found.isPresent());
    }
}
```

✅ PostgreSQL-specific behavior (e.g., case sensitivity, JSONB fields) will now be properly tested.

---

### 🔹 **Option 3: Testcontainers (Spin up PostgreSQL in Docker for tests)**

* **Most accurate & clean** for CI/CD pipelines
* Isolated database for each test run

#### Example:

```java
@Testcontainers
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
public class UserRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
        .withDatabaseName("testdb")
        .withUsername("user")
        .withPassword("pass");

    @DynamicPropertySource
    static void setDatasourceProps(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void testPostgresQuery() {
        User user = new User("john", "john@example.com");
        userRepository.save(user);
        assertEquals(1, userRepository.count());
    }
}
```

---

## ✅ Summary Table

| Approach                    | Accuracy | Speed     | Needs Docker? | Good for CI? |
| --------------------------- | -------- | --------- | ------------- | ------------ |
| `@DataJpaTest` + H2         | ❌ Medium | ✅ Fast    | ❌ No          | ✅ Yes        |
| `@DataJpaTest` + PostgreSQL | ✅ High   | ❌ Slower  | ❌ No          | ✅ Yes        |
| Testcontainers              | ✅ Best   | ❌ Slowest | ✅ Yes         | ✅ Best       |

---

## 🟢 My Suggestion for You:

Since you're already using **PostgreSQL**, use this:

* `@DataJpaTest` + PostgreSQL + `application-test.yml` + `@ActiveProfiles("test")`
* OR Testcontainers if you want CI/CD-ready isolation.

---

Excellent question! You're asking:

> "How do I test **DAO and Repository layers** in Spring Boot when using **SQL (like PostgreSQL)** vs **NoSQL (like MongoDB)**, especially when data is **unstructured**?"

Let’s compare them and explain how testing works **for both types of databases** in Spring Boot applications:

---

## ✅ 1. **SQL (Structured Data – PostgreSQL, MySQL, etc.)**

### 🔹 Tools You Use:

* `@DataJpaTest`
* H2 or PostgreSQL
* Spring Data JPA
* Testcontainers (optional)

### 🔹 Testing Layers:

| Layer      | How You Test                                                        |
| ---------- | ------------------------------------------------------------------- |
| DAO        | Integration test using `@DataJpaTest` with real DB                  |
| Repository | Use `@DataJpaTest` + PostgreSQL OR in-memory DB (H2/Testcontainers) |

---

### 🔹 Example:

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE)
@ActiveProfiles("test")  // Use test DB config
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void testFindByEmail() {
        userRepository.save(new User("Ali", "ali@mail.com"));
        assertTrue(userRepository.findByEmail("ali@mail.com").isPresent());
    }
}
```

---

## ✅ 2. **NoSQL – MongoDB (Unstructured/Semi-structured Data)**

### 🔹 Tools You Use:

* `@DataMongoTest`
* Embedded MongoDB (via **Flapdoodle**) or Testcontainers
* Spring Data MongoDB

---

### 🔹 Testing Layers:

| Layer      | How You Test                                              |
| ---------- | --------------------------------------------------------- |
| DAO        | Use `@DataMongoTest` or mock if logic-heavy               |
| Repository | Use `@DataMongoTest` + Embedded MongoDB or Testcontainers |

---

### 🔹 Option A: Embedded MongoDB using Flapdoodle (Common)

```xml
<!-- Dependency in pom.xml -->
<dependency>
  <groupId>de.flapdoodle.embed</groupId>
  <artifactId>de.flapdoodle.embed.mongo</artifactId>
  <scope>test</scope>
</dependency>
```

```java
@DataMongoTest
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void testInsertAndFind() {
        User user = new User("Sara", "sara@mail.com");
        userRepository.save(user);
        Optional<User> found = userRepository.findByEmail("sara@mail.com");
        assertTrue(found.isPresent());
    }
}
```

---

### 🔹 Option B: Testcontainers for MongoDB

```java
@Testcontainers
@DataMongoTest
public class MongoRepoTest {

    @Container
    static MongoDBContainer mongoDBContainer = new MongoDBContainer("mongo:6.0");

    @DynamicPropertySource
    static void setProps(DynamicPropertyRegistry registry) {
        registry.add("spring.data.mongodb.uri", mongoDBContainer::getReplicaSetUrl);
    }

    @Autowired
    private UserRepository userRepository;

    @Test
    void testMongoInsert() {
        User user = new User("Fatima", "fatima@mail.com");
        userRepository.save(user);
        assertEquals(1, userRepository.count());
    }
}
```

---

## ✅ Summary: SQL vs NoSQL Testing in Spring Boot

| Feature/Layer   | SQL (PostgreSQL)                   | NoSQL (MongoDB)                              |
| --------------- | ---------------------------------- | -------------------------------------------- |
| Test annotation | `@DataJpaTest`                     | `@DataMongoTest`                             |
| Test DB options | H2 / PostgreSQL / Testcontainers   | Embedded Mongo (Flapdoodle) / Testcontainers |
| Repo testing    | JPA repositories                   | Mongo repositories                           |
| Structured Data | Yes                                | No (flexible/unstructured)                   |
| Extra setup     | Test DB profile, schema (optional) | Embedded Mongo or Docker Mongo setup         |

---

Perfect! Here's a complete, **working example** of a **Spring Boot JUnit 5 test** for a **repository using PostgreSQL** — great for realistic **integration testing** in enterprise apps.

---

## 🔧 Technologies:

* Spring Boot
* Spring Data JPA
* PostgreSQL
* JUnit 5
* `@DataJpaTest`

---

## 📁 Structure Example:

```
com.example.demo
├── entity
│   └── User.java
├── repository
│   └── UserRepository.java
├── DemoApplication.java
└── UserRepositoryTest.java
```

---

## 1️⃣ `User` Entity

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import java.util.Objects;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Constructors
    public User() {}
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // Getters & Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    // equals and hashCode
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return Objects.equals(id, user.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

---

## 2️⃣ `UserRepository`

```java
package com.example.demo.repository;

import com.example.demo.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```

---

## 3️⃣ `application-test.yml` (in `src/test/resources`)

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/testdb
    username: testuser
    password: testpass
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
  test:
    database:
      replace: none
```

> ✅ Make sure a test DB `testdb` exists and the user/password matches.

---

## 4️⃣ `UserRepositoryTest`

```java
package com.example.demo;

import com.example.demo.entity.User;
import com.example.demo.repository.UserRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.ActiveProfiles;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@ActiveProfiles("test")
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void testSaveAndFindByEmail() {
        User user = new User("Hamraz", "hamraz@example.com");
        userRepository.save(user);

        Optional<User> found = userRepository.findByEmail("hamraz@example.com");

        assertTrue(found.isPresent());
        assertEquals("Hamraz", found.get().getName());
    }
}
```

---

## ✅ Result:

* This test **uses your real PostgreSQL DB** (not H2).
* It verifies `save()` and `findByEmail()` methods in your repository.
* Uses `@DataJpaTest` to load only JPA components (fast and clean).

---

## 🔍 Final Notes:

* You can create more tests for **custom JPQL or native queries** too.
* If you want to **use Testcontainers**, I can give you that version as well — it's perfect for CI/CD pipelines.

---

## 🎉 What You Now Have:

✅ Real DB (H2)
✅ JPA Repository
✅ Entity-level validation (`@NotBlank`)
✅ `@Valid` request validation in controller
✅ Global handler for:

* Generic exceptions
* Custom exceptions
* Validation errors
  ✅ Full-stack integration tests with real persistence + error checks

---






