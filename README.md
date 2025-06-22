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


 **complete unit test example** for a **Spring Boot controller** with **CRUD operations**, using **ResponseEntity**, **MockMvc**, **@WebMvcTest**, and **@MockBean**.

---

### ✅ Sample: `ItemController` with CRUD Operations

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
        Item item = itemService.getItemById(id);
        return ResponseEntity.ok(item);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Item> updateItem(@PathVariable Long id, @RequestBody Item item) {
        Item updated = itemService.updateItem(id, item);
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteItem(@PathVariable Long id) {
        itemService.deleteItem(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

### ✅ Sample `Item` Model

```java
public class Item {
    private Long id;
    private String name;

    // Constructors, Getters, Setters
}
```

---

### ✅ Unit Test for `ItemController`

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
    public void testGetItemById() throws Exception {
        Item item = new Item(1L, "Laptop");
        Mockito.when(itemService.getItemById(1L)).thenReturn(item);

        mockMvc.perform(get("/api/items/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("Laptop"));
    }

    @Test
    public void testUpdateItem() throws Exception {
        Item updatedItem = new Item(1L, "Tablet");
        Mockito.when(itemService.updateItem(Mockito.eq(1L), Mockito.any())).thenReturn(updatedItem);

        mockMvc.perform(put("/api/items/1")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(updatedItem)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1L))
                .andExpect(jsonPath("$.name").value("Tablet"));
    }

    @Test
    public void testDeleteItem() throws Exception {
        Mockito.doNothing().when(itemService).deleteItem(1L);

        mockMvc.perform(delete("/api/items/1"))
                .andExpect(status().isNoContent());
    }
}
```

---

### 🧪 Key Points:

| Part                  | Explanation                            |
| --------------------- | -------------------------------------- |
| `@WebMvcTest`         | Loads only controller layer            |
| `@MockBean`           | Mocks service layer dependency         |
| `MockMvc`             | Simulates HTTP requests                |
| `jsonPath("$.field")` | Checks JSON field values               |
| `ObjectMapper`        | Converts Java objects to JSON and back |

---



