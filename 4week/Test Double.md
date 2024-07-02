# Test Double

테스트 목적으로 실제 객체 대신 사용되는 모든 종류의 가상 객체에 대해서 `Test Double`이라고 부른다. 그리고 다음 5가지 종류로 정의된다.

- Dummy: 아무 것도 하지 않는 깡통 객체, 일반적으로 파라미터를 채워넣는데 사용
- Fake: 단순한 형태로 동일한 기능은 수행하나, 프로덕션에서 쓰기에는 부족한 객체 (ex. FakeRepository, in-memory DB)
- Stub: 테스트에서 요청한 것에 대해 미리 준비한 결과를 제공하는 객체, 그외에는 응답하지 않는다.
- Spy: Stub이면서 호출된 내용을 기록하여 보여줄 수 있는 객체, 일부는 실제 객체처럼 동작시키고 일부만 Stubbing할 수 있다.
- Mock: 행위에 대한 기대를 명세하고, 그에 따라 동작하도록 만들어진 객체

## Stub vs Mock

Stub은 상태 검증(state verification)에 사용되는 반면, Mock은 행동 검증(behavior verification)에 사용한다. 그리고 `Meszaros`는 Mock 객체는 항상 행동 검증에 사용하기 때문에, 어떤 식으로든 사용할 수 있는 Stub을 선호한다고 한다.
### Stub

```java
public interface MailService {
  public void send (Message msg);
}

public class MailServiceStub implements MailService {
  private List<Message> messages = new ArrayList<Message>();
  public void send (Message msg) {
    messages.add(msg);
  }
  public int numberSent() {
    return messages.size();
  }
}
```
```java
class OrderStateTester...

  public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    MailServiceStub mailer = new MailServiceStub();
    order.setMailer(mailer);
    order.fill(warehouse);
    assertEquals(1, mailer.numberSent());
  }
```
### Mock

```java
class OrderInteractionTester...

  public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    Mock warehouse = mock(Warehouse.class);
    Mock mailer = mock(MailService.class);
    order.setMailer((MailService) mailer.proxy());

    mailer.expects(once()).method("send");
    warehouse.expects(once()).method("hasInventory")
      .withAnyArguments()
      .will(returnValue(false));

    order.fill((Warehouse) warehouse.proxy());
  }
}
```


[reference]
- https://martinfowler.com/articles/mocksArentStubs.html