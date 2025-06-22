**3 main ways to test a Spring Boot application** 

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

Sure! Below is a simple example of **ItemService interface**, its **implementation**, and **Mockito unit tests** for the service using Java 8 features like `Optional` and lambdas.

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

---
