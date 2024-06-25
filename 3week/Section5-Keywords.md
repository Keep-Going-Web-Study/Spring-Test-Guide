**연결 문서**: [[Presentation Layer 테스트 (1)]], [[Presentation Layer 테스트 (2)]], [[WebMvcTest가 뭐지?]]


## Keywords
- Layered Architecture
- Hexagonal Architecture
- IoC, DI, AOP
- ORM, 패러다임의 불일치, Hibernate
- Spring Data JPA

- @SpringBootTest vs. @DataJpaTest
- @SpringBootTest vs. @WebMvcTest
- @Transactional (readOnly=true)
- Optimistic Lock, Pessimistic Lock
- CQRS

- @RestControllerAdvice, @ExceptionHandler
- Spring bean validation
- @WebMvcTEst
- ObjectMapper
- Mock, Mockito, @MockBean


## Layered Architecture

JPA를 사용하면서 관련 어노테이션을 많이 사용했음
- 도메인 객체에 @Entity를 사용하면서 DB와 강결합 되어있음 → 분리의 필요성
- 도메인 레이어와 인프라 레이어의 결합을 줄일 수 있을까?
- Hexagonal Architecture의 등장


## Hexagonal Architecture

도메인 모델을 중심으로 외부의 시스템이 접근하게 되는 설계 패턴.
- 도메인 모델은 외부를 아예 모르도록 설계함
- 멀티 모듈 추가, 서버 확장에 따라 장점이 뚜렷해짐

AI에게 물어본 결과

Hexagonal Architecture, 또는 Ports and Adapters Architecture는 Alistair Cockburn이 제안한 소프트웨어 아키텍처 패턴입니다. 이 패턴은 애플리케이션을 내부와 외부 두 부분으로 나눕니다.
- 내부: 이 부분은 어플리케이션의 핵심 비즈니스 로직을 담당합니다. 일반적으로 도메인 모델이 여기에 포함되며, 비즈니스 규칙을 적용하고 유지합니다. 내부는 외부의 변경에 영향을 받지 않아야하며, 순수하게 비즈니스 로직만을 다루어야 합니다.
- 외부: 이 부분은 그 외 모든 것을 다룹니다. 입출력, 데이터 저장, 네트워크 통신 등 입니다. 외부 구성요소들은 내부와 독립적으로 변경되고 확장할 수 있어야 합니다.
Hexagonal Architecture는 이 두 부분 사이에 "ports"와 "adapters"라는 추상화 레이어를 만들어서 정보를 교환합니다. "Ports"는 시스템 안으로 들어오거나 나가는 "interface"를 제공하며, "adapters"는 이 interfaces를 통해 구체적인 기술과 상호작용합니다.
이러한 방식은 애플리케이션의 핵심 비즈니스 로직을 클린하게 유지하면서, 서로 다른 외부 시스템이나 인프라와의 통신을 용이하게 하고, 유지보수와 테스트를 쉽게합니다

*참고 링크 1: [DDD Part 3: Domain-Driven Design and the Hexagonal Architecture](https://vaadin.com/blog/ddd-part-3-domain-driven-design-and-the-hexagonal-architecture), 2024년 06월 25일 (Tue) 19:37 확인*
*참고 링크 2: [지속 가능한 소프트웨어 설계 패턴: 포트와 어댑터 아키텍처 적용하기](https://engineering.linecorp.com/ko/blog/port-and-adapter-architecture), 2024년 06월 25일 (Tue) 19:37 확인*


## QueryDSL

타입 체크, 동적 쿼리 생성에 장점이 있는 프레임워크
JPA 쓸 때 거의 필수다.
- 동적 쿼리 생성시 null 같은 거 처리하기 좋음


## @RestController

ControllerAdvice + ResponseBody 



다음 세션에서 Mock에 관해 할거임