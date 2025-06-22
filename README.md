Here's a clean and well-formatted version of your Spring Boot + JUnit/Mockito tutorial notes for your GitHub README or repo documentation:

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

