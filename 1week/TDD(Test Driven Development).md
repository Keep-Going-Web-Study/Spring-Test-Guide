# TDD(Test Driven Development)

프로덕션 코드보다 테스트 코드를 먼저 작성하여 테스트가 구현 과정을 주도하도록 하는 방법론

![](./imgs/Pasted%20image%2020240611002700.png)

## RED: 실패하는 테스트 작성

### 기능 정의

```java
public int calculateTotalPrice() {  
    return 0;
}
```

### 실패하는 테스트 작성

```java
@Test  
void calculateTotalPrice() {  
    CafeKiosk cafeKiosk = new CafeKiosk();  
    Americano americano = new Americano();  
    Latte latte = new Latte();  
  
    cafeKiosk.add(americano);  
    cafeKiosk.add(latte);  
  
    int totalPrice = cafeKiosk.calculateTotalPrice();  
  
    assertThat(totalPrice).isEqualTo(8500);  
}
```

## GREEN: 테스트를 통과하는 최소한의 코딩

### 기능 수정

```java
public int calculateTotalPrice() {  
    return 8500;
}
```

## REFACTOR: 구현 코드 개선 & 테스트 통과 유지

### 기능 구현 코드 개선

```java
public int calculateTotalPrice() {  
    int totalPrice = 0;  
    for (Beverage beverage : beverages) {  
        totalPrice += beverage.getPrice();  
    }  
    return totalPrice;  
}
```

---

## 선 기능 구현, 후 테스트 작성의 문제

1. 테스트 자체를 누락할 가능성이 있음
2. 특정 테스트 케이스(e.g. 해피케이스)만 검증할 가능성이 있음
3. 잘못된 구현을 다소 늦게 발견할 가능성이 있음


## 선 테스트 작성, 후 기능 구현의 이점

1. 복잡도가 낮은(유연하며 유지보수가 쉬운), 테스트 가능한 코드로 구현할 수 있게됨
2. 쉽게 발견하기 어려운 엣지(Edge) 케이스를 놓치지 않게 해줌
3. 구현에 대한 빠른 피드백을 받을 수 있음
4. 과감한 리팩토링이 가능
