# @ParameterizedTest

## @ValueSource

- 지원하는 타입: short, byte, int, long, float, double, char, boolean, String, Class

```java
@ParameterizedTest 
@ValueSource(ints = { 1, 2, 3 }) 
void testWithValueSource(int argument) { 
	assertTrue(argument > 0 && argument < 4); 
}
```

## Null and Empty Sources

- `@NullSource`: 하나의 null 인자를 넘겨줌
- `@EmptySource`: 하나의 empty 인자(String, Collection 등)를 넘겨줌
- `@NullAndEmptySource`: `@NullSource`와 `@EmptySource`를 결합한 어노테이션

## @EnumSource

value를 지정하지 않을 경우 첫 번째 파라미터를 사용한다.

```java
@ParameterizedTest 
@EnumSource(ChronoUnit.class) 
void testWithEnumSource(TemporalUnit unit) { 
	assertNotNull(unit);
}
```

ChronoUnit은 TempralUnit의 구체 클래스이다.

names로 constants를 지정해줄 수 있고, 안한다면 모든 constants가 사용된다.

```java
@ParameterizedTest 
@EnumSource(names = { "DAYS", "HOURS" }) 
void testWithEnumSourceInclude(ChronoUnit unit) { 
	assertTrue(EnumSet.of(ChronoUnit.DAYS, ChronoUnit.HOURS).contains(unit)); 
}
```

## @CsvSource

```java
@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@CsvSource({"HANDMADE, false", "BOTTLE, true", "BAKERY, true"})
@ParameterizedTest
void containsStockType4(ProductType productType, boolean expected) {
	// when
	boolean result = ProductType.containsStockType(productType);

	// then
	assertThat(result).isEqualTo(expected);
}
```

## @MethodSource

```java
private static Stream<Arguments> provideProductTypesForCheckingStockType() {
	return Stream.of(
		Arguments.of(ProductType.HANDMADE, false),
		Arguments.of(ProductType.BOTTLE, true),
		Arguments.of(ProductType.BAKERY, true)
	);
}

@DisplayName("상품 타입이 재고 관련 타입인지를 체크한다.")
@MethodSource("provideProductTypesForCheckingStockType")
@ParameterizedTest
void containsStockType5(ProductType productType, boolean expected) {
	// when
	boolean result = ProductType.containsStockType(productType);

	// then
	assertThat(result).isEqualTo(expected);
}
```

[reference]
- [Junit5 Parameterized Tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)