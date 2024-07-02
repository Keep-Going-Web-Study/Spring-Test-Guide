# 통합 테스트 전체 코드

```java
@SpringBootTest  
class OrderStatisticsServiceTest {  
  
    @Autowired  
    private OrderStatisticsService orderStatisticsService;  
  
    @Autowired  
    private OrderProductRepository orderProductRepository;  
  
    @Autowired  
    private OrderRepository orderRepository;  
  
    @Autowired  
    private ProductRepository productRepository;  
  
    @Autowired  
    private MailSendHistoryRepository mailSendHistoryRepository;  
  
    @MockBean  
    private MailSendClient mailSendClient;  
  
    @AfterEach  
    void tearDown() {  
        orderProductRepository.deleteAllInBatch();  
        orderRepository.deleteAllInBatch();  
        productRepository.deleteAllInBatch();  
        mailSendHistoryRepository.deleteAllInBatch();  
    }  
  
    @DisplayName("결제완료 주문들을 조회하여 매출 통계 메일을 전송한다.")  
    @Test  
    void sendOrderStatisticsMail() {  
        // given  
        LocalDateTime now = LocalDateTime.of(2023, 3, 5, 0, 0);  

		// 상품 3개 저장
        Product product1 = createProduct(HANDMADE, "001", 1000);  
        Product product2 = createProduct(HANDMADE, "002", 2000);  
        Product product3 = createProduct(HANDMADE, "003", 3000);  
        List<Product> products = List.of(product1, product2, product3);  
        productRepository.saveAll(products);  

		// 주문 4개 저장
        Order order1 = createPaymentCompletedOrder(LocalDateTime.of(2023, 3, 4, 23, 59, 59), products);  
        Order order2 = createPaymentCompletedOrder(now, products);  
        Order order3 = createPaymentCompletedOrder(LocalDateTime.of(2023, 3, 5, 23, 59, 59), products);  
        Order order4 = createPaymentCompletedOrder(LocalDateTime.of(2023, 3, 6, 0, 0), products);  
  
        // stubbing  
        when(mailSendClient.sendEmail(any(String.class), any(String.class), any(String.class), any(String.class)))  
            .thenReturn(true);  
  
        // when  
        boolean result = orderStatisticsService.sendOrderStatisticsMail(LocalDate.of(2023, 3, 5), "test@test.com");  
  
        // then  
        assertThat(result).isTrue();  
  
        List<MailSendHistory> histories = mailSendHistoryRepository.findAll();  
        assertThat(histories).hasSize(1)  
            .extracting("content")  
            .contains("총 매출 합계는 12000원입니다.");  
    }  
 
	  
	/**  
	 * Order 엔티티 생성 후 DB 저장   
	 */  
	private Order createPaymentCompletedOrder(LocalDateTime now, List<Product> products) {  
	    Order order = Order.builder()  
	        .products(products)  
	        .orderStatus(OrderStatus.PAYMENT_COMPLETED)  
	        .registeredDateTime(now)  
	        .build();  
	    return orderRepository.save(order);  
	}  
	  
	/**  
	 * Product 엔티티 생성   
	 */  
	private Product createProduct(ProductType type, String productNumber, int price) {  
	    return Product.builder()  
	        .type(type)  
	        .productNumber(productNumber)  
	        .price(price)  
	        .sellingStatus(SELLING)  
	        .name("메뉴 이름")  
	        .build();  
	}
  
}
```

## Stubbing

`Stub`은 테스트에서 요청한 것에 대해 미리 준비한 결과를 제공하는 객체이다.

```java
when(mailSendClient.sendEmail(any(String.class), any(String.class), any(String.class), any(String.class)))  
    .thenReturn(true);
```

`@MockBean`을 사용하여 `MailSendClient`를 Mocking하고 `Mockito`의 `when().thenReturn()` 체이닝 메소드로 `Stubbing`할 수 있다.

이렇게 외부에서 동작하는 부분을 테스트하기 어려울 경우 `Mocking`과 `Stubbing`을 통해 정상적으로 진행할 수 있다.

### any() 를 사용해야하는 경우

위 예시 코드에서 `anyString()` 또는 `any(String.class)`를 사용하여 인자값을 전달하고 있다. 하지만 `any()`를 사용하면 실제 집어넣은 인자값이 제대로 전달되었는지 확인이 안된다. 따라서 `Stubbing`을 하더라도 전달되는 인자값을 명시적으로 지정해주는 것이 올바르다고 판단할 수 있다.

any()를 사용해야하는 경우는 Mocking한 객체의 메소드 내에서 생성되어 전달되는 인자값에 사용할 수 있다. `sendEmail()` 메소드에서는 `String.format()`으로 String 객체를 생성하기 때문에 테스트에서 전달한 인자값(객체)와 상이하게된다.

코드를 수정해보면 다음과 같이 작성할 수 있다.

```java
String email = "test@test.com";  
LocalDate orderDate = LocalDate.of(2023, 3, 5);  
when(mailSendClient.sendEmail(eq("no-reply@cafekiosk.com"), eq(email), anyString(), anyString()))  
    .thenReturn(true);
```

