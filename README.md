# JUnit-Mockito

Step 01: Creating a Hello World Controller
Create a simple controller class:

java
Copy
Edit
@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello, World!";
    }
}
Run the application and access:
http://localhost:8080/api/hello → You should see "Hello, World!".

Add a basic unit test using JUnit:

java
Copy
Edit
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


Explanation:

@WebMvcTest: Loads only the web layer (controller).

MockMvc: Lets you send fake HTTP requests and verify responses.

.perform(get(...)): Simulates a GET request.

.andExpect(...): Checks that the status is 200 OK and response body matches "Hello, World!".
