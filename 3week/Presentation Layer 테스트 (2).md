## WebMvcTest

- MockMvc랑 같이 사용하는 `@WebMvcTest`가 있음
    - [[WebMvcTest가 뭐지?]] → 찾아서 정리. 강의에서는 컨트롤러 레이어만 집중해서 테스트하기 위한 어노테이션이라 설명함.
    - [\#11 이슈](https://github.com/Keep-Going-Web-Study/BE-Spring-Test-Guide/issues/11) 질문에 관한 답을 통해 대략적인 동작과 목적 파악


## @MockBean?

`@MockBean`가 뭔지 찾아가다 보면 `Mockito`를 확인할 수 있다. 해당 라이브러리에 대해서는 [site.mockito.org](site.mockito.org) 에서 자세하게 설명하고 있다. (강의의 다음 섹션에서 다룰 예정)

위의 `@WebMvcTest` 정리에서 다뤘던 것 처럼 서비스, 레포지토리를 주입받지 않기 때문에 `@MockBean`으로 서비스를 처리해주지 않으면 예외가 터지게 된다.


**테스트 실습 진행**
```java
...
// when // then
mockMvc.perform(MockMvcRequestBuilders.post(...)
                .content(...)
                .contentType(...)
        )
        .andDo(MockMvcResultHandler.print())
        .andExpect(MockMvcResultMatchers.status().isOk());
```

- <font color="#ff0000">[Err] BaseEntity에서 Auditing 설정을 해놓음 → WebMvcTest에서 관련 Bean을 만들 수 없음</font>
    - <font color="#00b050">[Sol] Config 분리를 통해 문제 해결 (`@EnableJpaAuditing` 분리)</font>

```java
@EnableJpaAuditing
@Configuration
public class JpaAuditingConfig {}
```

- <font color="#ff0000">[Err] 테스트에서 400 에러 발생</font>
    - [Sol] `ProductController.createProduct()`인자에 `@RequestBody` 추가
    - [Sol] `ProductCreateRequest` 클래스에 `@NoArgsConstructor` 추가
        - 사용했던 `ObjectMapper`가 역직렬화를 도와준다. 따라서 
          `JSON` → `ProductCreateRequest`로 변환하는 과정에서 기본 생성자를 필요로 함


## Validation

jakarta.validation 사용을 통해 데이터 유효성을 검증
- Controller에 `@Valid` 사용
- Entity에 `@NotNull` `@NotBlank` `@Positive` 등 사용

ApiResponse 라는 응답 공통 클래스 사용해보자.

**실습 진행**
```java
public class ApiResponse<T> {

  private int code;
  private HttpStatus status;
  ...
  
  // 생성자도 생성
  ...
  
  public static <T> ApiResponse<T> of(HttpStatus httpStatus, String message, T data) {
      return new ApiResponse<>(httpStatus, message, data);
  }
  
  public static <T> ApiResponse<T> of(HttpStatus httpstatus, T data) {
    return new ApiResponse<>(HttpStatus, httpStatus.name(), data);
  }

  public static <T> ApiResponse<T> ok(T data) {
    return of(HttpStatus.OK, data);
  }
}
```

```java
// ProductController class 변경
...
@PostMapping(...)
public ApiResponse<ProductReponse> createProduct(...) {
  ...
  return ApiResponse.of(HttpStatus.Ok,
                productService.createProduct(...));
}
```

**테스트 실습 진행**
- NotNull, NotBlank, Positive 해놓은 거 모두 테스트
- `@RestControllerAdvice`에 `@ResponseStatus(BAD_REQUEST)`를 주어야 그대로 잘 오게 된다.
    - <font color="#ff0000">`@ControllerAdvice`로 테스트할 경우 에러 발생</font>

```java
// ProductControllerTest.java
...
// when // then
mockMvc.perform(
        post("/api/v1/products/new")
            .content(objectMapper.writeValueAsString(request))
            .contentType(MediaType.APPLICATION_JSON)
    )
    .andDo(print())
    .andExpect(status().isBadRequest())
    .andExpect(jsonPath("$.code").value("400"))
    .andExpect(jsonPath("$.status").value("BAD_REQUEST"))
    .andExpect(jsonPath("$.message").value("상품 이름은 필수입니다."))
    .andExpect(jsonPath("$.data").isEmpty());
...
```

- Getter 기반으로 JSON 생성해줌


## @NotNull, @NotBlank, @NotEmpty

차이는 뭘까?
- NotEmpty: 공백은 통과 ("")
- NotNull: null만 아니면 됨 ("", "   ")
    - Enum 타입은 null 값만 오기 때문에 해당 어노테이션 사용
- NotBlank: 무조건 문자값이 있어야 함

Validation 책임분리는 어떻게?
- 예를 들어 상품 이름이 20자 제한이라면, 어디서 검증하는게 맞을까?
- 컨트롤러 레이어 앞단에서 튕겨낼 책임이 맞는가?
- 한 번에 한 레이어에서 검증할 필요는 없다.


**실습 진행**
- Get 요청 테스트 진행
- `OrderControllerTest` 진행


## 레이어간 의존?

하위 레이어는 상위 레이어를 몰라야 한다.
- 현재 패키지 구조는 `controller > order + product` 와 `service > order + product` 이다.
- 따라서 `service > order`에서 `controller > order`의 `OrderCreateRequest`를 사용하는건 레이어간 의존에 해당함
- 이를 해결하기 위해 `OrderCreateServiceRequest` DTO를 만들어서 `service > order`는 해당 DTO만 사용하게 한다.
    - `controller > order`에서 `OrderCreateRequest`를 `OrderCreateServiceRequest`로 변환해 service로 전달해준다.
    - 이렇게 되면 Service Layer에서 Bean Validation을 사용하지 않아도 됨 (모듈 분리시 불필요한 의존성 제거)

**예시**
- 상황
    - [ ] 다양한 채널에서 서비스를 사용한다. `app controller`, `pos controller` 추가 생성 요청
    - [ ] 하나의 order service 에서 다양한 채널의 요청을 처리해야한다.
- 이때, 서비스단계에서 `kiosk controller`의 DTO를 의존한다면 다른 컨트롤러에서도 kiosk를 알아야한다.
- 따라서 service용 DTO를 만들어 책임을 분리한다. 의존 문제를 해결하고 유지 보수성을 높일 수 있는 장점이 있다.
