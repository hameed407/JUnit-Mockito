Here’s **Step 01: Creating a Hello World Controller** for your Spring Boot + JUnit/Mockito repo:

---


1. **Create a simple controller class:**

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

2. **Run the application and access:**
   `http://localhost:8080/api/hello` → You should see `"Hello, World!"`.

3. **Add a basic unit test using JUnit:**

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

---

---

### ✅  Using MockMvc to Test Hello World Controller

Now that we have a basic controller, let’s write a test using **MockMvc** to simulate an HTTP request.

#### 1. **Add the Test Class**

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

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

#### 2. **Explanation:**

* `@WebMvcTest`: Loads only the web layer (controller).
* `MockMvc`: Lets you send fake HTTP requests and verify responses.
* `.perform(get(...))`: Simulates a GET request.
* `.andExpect(...)`: Checks that the status is 200 OK and response body matches `"Hello, World!"`.

---




#### 1. **Create a Service Class**

```java
@Service
public class ItemService {

    public String getItemName() {
        return "Laptop";
    }
}
```

---

#### 2. **Create a Controller Class**

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

---

#### 3. **Run & Test the Endpoint**

Go to:
`http://localhost:8080/api/items/name`
You should see:

```
Laptop
```

---

This sets the foundation for **mocking the service layer** in controller tests, which we’ll do in the next step.

---


---

###  Unit Testing Item Controller and Basic JSON Assertions

Now that we have an `ItemController` and `ItemService`, let’s write a **unit test** for the controller by **mocking the service** using **Mockito** and verifying the response.

---

#### 1. **Add Unit Test for Controller**

```java
@WebMvcTest(ItemController.class)
public class ItemControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ItemService itemService;

    @Test
    public void testGetItemName() throws Exception {
        // Mock the service response
        Mockito.when(itemService.getItemName()).thenReturn("Laptop");

        mockMvc.perform(get("/api/items/name"))
               .andExpect(status().isOk())
               .andExpect(content().string("Laptop"));
    }
}
```

---

#### ✅ What’s Happening:

| Annotation       | Purpose                                         |
| ---------------- | ----------------------------------------------- |
| `@WebMvcTest`    | Loads only web layer (controllers, filters)     |
| `@MockBean`      | Creates a mock version of `ItemService`         |
| `Mockito.when()` | Defines return value when mock method is called |
| `MockMvc`        | Sends a fake GET request to the controller      |
| `.andExpect()`   | Verifies response status and content            |

---

#### ✅ JSON Response (if returning a JSON object)

If your controller returns a JSON:

```java
@GetMapping("/info")
public ResponseEntity<Map<String, String>> getItemInfo() {
    Map<String, String> response = Map.of("name", "Laptop", "type", "Electronics");
    return ResponseEntity.ok(response);
}
```

**Then test it like this:**

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

---

🧪 `jsonPath("$.key")` is used to check values in JSON responses easily.

---



