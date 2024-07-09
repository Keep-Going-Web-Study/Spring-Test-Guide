# @DynamicTest

- `@TestFactory` 적용
- List 안에 DynamicTest를 순서대로 지정해주면 공유 변수를 단계별로 사용하며 테스트할 수 있다.

```java
@DisplayName("재고 차감 시나리오")
@TestFactory
Collection<DynamicTest> stockDeductionDynamicTest() {
	// given
	Stock stock = Stock.create("001", 1);

	return List.of(
		DynamicTest.dynamicTest("재고를 주어진 개수만큼 차감할 수 있다.", () -> {
			// given
			int quantity = 1;

			// when
			stock.deductQuantity(quantity);

			// then
			assertThat(stock.getQuantity()).isZero();
		}),
		DynamicTest.dynamicTest("재고보다 많은 수의 수량으로 차감 시도하는 경우 예외가 발생한다.", () -> {
			// given
			int quantity = 1;

			// when // then
			assertThatThrownBy(() -> stock.deductQuantity(quantity))
				.isInstanceOf(IllegalArgumentException.class)
				.hasMessage("차감할 재고 수량이 없습니다.");
		})
	);
}
```

1. 첫 번째 DynamicTest, 1개 차감 후 Stock의 수량은 0개로 변경됨
2. 두 번째 DynamicTest, 1개 차감하는 과정에서 이전 수행 결과로 Stock 수량이 0개가 되어 예외가 발생됨

