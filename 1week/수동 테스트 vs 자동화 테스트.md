# 수동 테스트 vs 자동화 테스트

## 수동 테스트

```java
@Test  
void add_manual_test() {  
    CafeKiosk cafeKiosk = new CafeKiosk();  
    cafeKiosk.add(new Americano()); // 아메리카노 추가
  
    System.out.println(">>> 담긴 음료 수: " + cafeKiosk.getBeverages().size()); // 1
    System.out.println(">>> 담긴 음료: " + cafeKiosk.getBeverages().get(0).getName());  // 아메리카노
}
```

## 자동화 테스트

### 자동화 테스트 예시 코드

```java
@Test  
void add() {  
    CafeKiosk cafeKiosk = new CafeKiosk();  
    cafeKiosk.add(new Americano());  // 아메리카노 추가
  
    assertThat(cafeKiosk.getBeverages()).hasSize(1);  
    assertThat(cafeKiosk.getBeverages().get(0).getName()).isEqualTo("아메리카노");  
}
```

### AssertJ

`spring-boot-starter-test`에는 `assertj-core`와 `junit-jupiter-api`가 있고, 각각 Assertions가 존재한다.

![](./imgs/Pasted%20image%2020240610214242.png)

#### 테스트 코드 예시

- `org.assertj.core.api.Assertions.assertThat`
	- 메소드 체이닝을 지원
	- assertThat(actual).isEqualTo(expected) 순서대로 값을 입력
	- `isEqualTo`처럼 조금 더 명시적으로 값을 비교할 수 있음
- `org.junit.jupiter.api.Assertions.assertEquals`
	- assertEquals(expected, actual) 순서대로 값을 입력

```java
import org.junit.jupiter.api.Test;  
  
import static org.assertj.core.api.Assertions.assertThat;  
import static org.junit.jupiter.api.Assertions.assertEquals;  
  
class AmericanoTest {  
  
    @Test  
    void getName() {  
        Americano americano = new Americano();  
  
        assertEquals("아메리카노", americano.getName());  
        assertThat(americano.getName()).isEqualTo("아메리카노");  
    }  
}
```



