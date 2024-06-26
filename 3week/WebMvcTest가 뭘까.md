**연결 문서**: [[Presentation Layer 테스트 (2)]]
**참고 문서 1**: https://spring.io/guides/gs/testing-web, 2024년 06월 23일 (Sun) 15:45 확인
**참고 문서 2**: https://ksh-coding.tistory.com/53, 2024년 06월 23일 (Sun) 15:45 확인
**참고 문서 3**: https://we1cometomeanings.tistory.com/65, 2024년 06월 23일 (Sun) 16:07 확인


## @SpringBootTest?

The `@SpringBootTest` annotation tells Spring Boot to look for a main configuration class (one with `@SpringBootApplication`, for instance) and use that to start a Spring application context. ...


## @MockMvc? @WebMvc?

Another useful approach is to not start the server at all but to test only the layer below that, where Spring handles the incoming HTTP request and hands it off to your controller. That way, almost all of the full stack is used, and your code will be called in exactly the same way as if it were processing a real HTTP request but without the cost of starting the server. To do that, use Spring’s `MockMvc` and ask for that to be injected for you by using the `@AutoConfigureMockMvc` annotation on the test case. The following listing (from `src/test/java/com/example/testingweb/TestingWebApplicationTest.java`) shows how to do so:

```java
package com.example.testingweb;
import static org.hamcrest.Matchers.containsString; 
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print; 
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content; 
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status; 
import org.junit.jupiter.api.Test; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc; 
import org.springframework.boot.test.context.SpringBootTest; 
import org.springframework.test.web.servlet.MockMvc; 

@SpringBootTest
@AutoConfigureMockMvc 
class TestingWebApplicationTest { 
  
  @Autowired 
  private MockMvc mockMvc; 
  
  @Test void shouldReturnDefaultMessage() throws Exception {
    this.mockMvc.perform(get("/"))
          .andDo(print())
          .andExpect(status().isOk())
          .andExpect(content()
              .string(containsString("Hello, World"))); 
    }
}
```

In this test, the full Spring application context is started but without the server. We can narrow the tests to only the web layer by using `@WebMvcTest`, as the following listing (from `src/test/java/com/example/testingweb/WebLayerTest.java`) shows:

```java
@WebMvcTest
include::complete/src/test/java/com/example/testingweb/WebLayerTest.java
```

- 여기서 알 수 있듯 @WebMvcTest는 테스트를 웹 계층으로 좁힐 수 있음

The test assertion is the same as in the previous case. However, in this test, Spring Boot instantiates only the web layer rather than the whole context. In an application with multiple controllers, you can even ask for only one to be instantiated by using, for example, `@WebMvcTest(HomeController.class)`.


## Conclusion

일반적으로 `@SpringBootTest` 어노테이션을 사용할 경우 서버를 시작하는 자원을 필요로 하기 떄문에 `@MockMvc`를 `@AuroWired`로 주입받아 빈으로 등록하는 과정(`@AutoConfigureMockMvc`)이 필요하다. 이때 `@AutoConfigureMockMvc`는 테스트 대상이 아닌 `@Service`, `@Repository` 도 모두 올린다.

따라서 `@WebMvcTest`를 사용하여 테스트를 웹 계층으로만 좁힐 수 있다.