- 프레젠테이션 레이어란?
    - 외부세계 요청을 가장 먼저 받는 계층
    - 파라미터에 대한 최소한의 검증을 수행한다.

- 어떻게 테스트 할것인가?
    - 하위에 있는 레이어를 Mocking 처리 할 거임
        

## 1.1. Mock 이란?

- ex) MockTail: 알콜 없는 칵테일
- MockMvc?
    - 테스트를 위한 의존관계를 처리해주기 위함
    - 스프링 mvc 동작을 재현할 수 있는 테스트 프레임워크


### 실습 시작

**컨트롤러 코드 작성**

```java
@PostMapping("/api/v1/products/news")
public void createProduct(ProductCreateRequest request) {
  // DB에서 마지막 저장된 Product의 상품번호를 읽어와서 +1
  String latestProductNumber = productRepository.findLatestProduct();
}
```

- 위 코드 생성과 함께 `ProductCreateRequestDto` 만듦
- `findLatestProduct()`는 native query로 만듦
```mysql
SELECT COALESCE(MAX(), 0) FROM product;
```


**테스트 생성**

```java
@DisplayName("가장 마지막으로 저장한 상품의 상품번호를 읽어온다.")
@Test
class ProductRepositoryTest {
  ...
  // when
  String latestProductNumber = productRepository.findLatestProduct();
  
  // then
  assertThat(latestProductNumber).isEqualTo(targetProductNumber);
}
```

- builder pattern 가독성을 올리기 위해 새로 메서드 만듦
- 추가로 `@DisplayName("가장 마지막으로 저장한 상품의 상품번호를 읽어올 때, 상품이 하나도 없는 경우에는 null을 반환한다.")` 이것도 작성


**서비스 테스트 만들기**

```java
@SpringBootTest
class ProductServiceTest {
  ...
  @DisplayName("신규 상품을 등록한다. 상품번호는 가장 최근 상품번호에서 1 증가한 값이다.")
  @Test
  void test() {
    ...
    // when
    ProductResponse product = productService.create(request);
 
    // then
    assertThat(productResponse)
        .extracting(...)
        .contains(...);
  }
}
```

- 레포지토리와 얇은 서비스 테스트는 동일한 테스트라 하더라도 작성하는게 좋음
    - 서비스가 기능 추가하면서 발전하기 때문
- Red - Green - Refactor 순서에 따라 실제 코드 만들기
- 추가로 `@DisplayName("상품이 하나도 없는 경우, 신규 상품을 등록하면 상품 번호는 001이다.")` 테스트도 만듦
- 강의 중에 `@ActiveProfiles("test")` 추가, 이후에도 에러가 나서 `@AfterEach void tearDown() { productRepository.deleteAllInBatch(); }` 작성 해줌

- AfterEach 관련해서 나중에 또 언급할 예정
- 해당 `createProduct()` 메소드에는 동시성 이슈가 존재. 어떻게 하면 해결할 수 있을까?
    - 3회 이상 재시도?
    - 동시 접속자가 너무 많은 경우, ProductNumber 정책을 다르게 가져갈 수 있음 (UUID)
        
- productRepository.findAll() 호출할 경우 저장이 잘 되어있는지도 확인해야함


```java
// then (위 테스트와 동일한 부분에 추가 작성)
...
List<Product> products = productRepository.findAll();
assertThat(products).hasSize(1)
    .extracting()
    .contains( tuple(...) ... );
```

- Test가 Production 코드를 보장해주도록 TDD 진행


## Transactional과 롤백 처리

- Transactional (readOnly = true) 왜이렇게 되어있나?
- CRUD에서 C..UD 동작 X / only Read
    - JPA에서 이점이 생긴다.
    - 플러시하는 부분에서 변경감지를 함
    - 읽기전용시 CUD의 스냅샷 저장, 변경감지를 안해도 됨. -> 성능 향상
        
    - CQRS - Command / Query (책임분리 패턴)
        
    - Master DB - Slave DB 에서 @Transactional의 readOnly 옵션을 구분하여 설정이 가능함.
        - DB endpoint를 구분함으로써 장애 격리를 할 수 있는 좋은 포인트임
        - 따라서 서비스 용도의 클래스에 readOnly를 준다. 이후 메서드에 필요하다면 @Transactional (false) 을 주는걸 추천함
        - **이유**는 TDD를 통해 메서드를 구현할때 CUD 관련 메서드 테스트를 하면서 역할을 구분 및 검증할 수 있기 때문이다.
        - 이 개념을 확장하여 CQRS 즉, Command 서비스와 Query 서비스를 애플리케이션에서 구분하여 사용하는 방법을 고려해볼 법 함

> [!tip] 
> 그렇다면 CQRS는 어떤 상황에 고려해볼 수 있을까?


---


### + CQRS 사용해야 할 때[^1]

CQRS는 다음과 같은 경우에 사용을 고려해볼 수 있다.

- 많은 사용자가 동일한 데이터에 병렬로 액세스하는 도메인. CQRS는 도메인 레벨에서의 병합 충돌을 최소화할 수 있도록 충분히 세분화된 명령(Command)를 정의하는 것을 가능하게 해 준다.
- 복잡한 프로세스나 도메인 모델을 통해 가이드되는 작업 기반 사용자 인터페이스. 쓰기 모델은 비즈니스 로직, 유효성 검사 등을 모두 가진 완전한 명령(Command) 처리 기능을 가진다. 쓰기 모델은 관련된 객체들의 집합을 하나의 단위(DDD에서의 aggregate)로 다룰 수 있고, 이 객체들이 항상 일관된 상태를 가지도록 보장할 수 있다. 읽기 모델의 경우 비즈니스 로직이나 유효성 검사 같은 것 없이, 오직 DTO만 반환한다. 읽기 모델은 최종적으로 쓰기 모델과 일치하게 된다. (딜레이는 있을 수 있음)
- 데이터 읽기의 성능이 데이터 쓰기의 성능과 별도로 조정이 가능해야 할 때. 특히 읽기의 수가 쓰기의 수보다 훨씬 많을 때. 이 경우, 읽기 모델은 스케일 아웃을 하고, 쓰기 모델은 적은 수로 유지할 수 있다. 적은 수의 쓰기 모델은 병향 충돌의 가능성을 최소화해준다.
- 한 팀은 쓰기 모델에 대한 복잡한 도메인 모델에만 집중해야 하고, 다른 한 팀은 사용자 인터페이스에 대한 읽기 모델에만 집중해야 할 때
- 시스템이 시간이 지남에 따라 계속해서 진화하고, 여러 버전을 가질 수 있으며, 정기적으로 바뀔 수 있는 경우
- 다른 시스템과의 통합, 특히 이벤트 소싱과 결합할 때. 서브시스템의 일시적 장애가 다른 시스템에 영향을 줘선 안된다. 

다음과 같은 경우에는 CQRS 사용이 권장되지 않는다.

- 도메인과 비즈니스 로직이 간단할 때
- 단순한 CRUD일 때


**추가적으로 참고해보면 좋을 블로그**

[CQRS 패턴, 코드에 순식간에 적용해보기](https://bluayer.com/37), 2024년 06월 24일 (Mon) 11:00 확인
[실전에서 TDD하기, 카카오페이 테크 블로그](https://tech.kakaopay.com/post/implementing-tdd-in-practical-applications/), 2024년 06월 24일 (Mon) 11:06 확인

[^1]: [CQRS란 무엇인가?](https://mslim8803.tistory.com/73), 2024년 06월 24일 (Mon) 10:42 확인
