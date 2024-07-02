# 메일 전송 단위 테스트

메일을 전송하는 `MailSendClient`와 MailSendHistory를 저장하는 `MailSendHistoryRepository` 객체를 Mocking하기 위해서 다음과 같이 Mockito를 사용할 수 있다.

```java
MailSendClient mailSendClient = Mockito.mock(MailSendClient.class);  
MailSendHistoryRepository mailSendHistoryRepository = Mockito.mock(MailSendHistoryRepository.class);
```

이후 생성한 Mock 객체를 사용하여 메일 전송 테스트 코드를 작성할 수 있다.

```java
class MailServiceTest {  
  
    @DisplayName("메일 전송 테스트")  
    @Test  
    void sendMail() {  
        // given  
        MailSendClient mailSendClient = Mockito.mock(MailSendClient.class);  
        MailSendHistoryRepository mailSendHistoryRepository = Mockito.mock(MailSendHistoryRepository.class);  
        when(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString()))  
            .thenReturn(true);  
 
        MailService mailService = new MailService(mailSendClient, mailSendHistoryRepository);  
        
        // when  
        boolean result = mailService.sendMail("", "", "", "");  
  
        // then  
        assertThat(result).isTrue();  
        verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));  
    }  

}
```

## @Mock

위에서 `mock()`으로 Mocking하는 대신 `@Mock`을 사용하여 Mocking 할 수 있다. 이때 테스트 클래스에 대해 `@ExtendWith(MockitoExtension.class)`을 적용하여 Mock 객체를 사용한다는 것을 알려주어야한다.

```java
@ExtendWith(MockitoExtension.class)  
class MailServiceTest {  
  
    @Mock  
    private MailSendClient mailSendClient;  
  
    @Mock  
    private MailSendHistoryRepository mailSendHistoryRepository;  
  
    @DisplayName("메일 전송 테스트")  
    @Test  
    void sendMail() {  
        // given  
		when(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString()))  
		    .thenReturn(true); 
        MailService mailService = new MailService(mailSendClient, mailSendHistoryRepository);    
		  
        // when  
        boolean result = mailService.sendMail("", "", "", "");  
  
        // then  
        assertThat(result).isTrue();  
        verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));  
    }  
  
}
```

## @InjectMocks

그리고 Mock 객체를 주입받아 생성되는 `MailService`도 `@InjectMocks`를 적용하여 간단하게 표현할 수 있다.

```java
@ExtendWith(MockitoExtension.class)  
class MailServiceTest {  
  
    @Mock  
    private MailSendClient mailSendClient;  
  
    @Mock  
    private MailSendHistoryRepository mailSendHistoryRepository;  
  
    @InjectMocks  
    private MailService mailService;
	
	...
```

## @Spy

`MailSendClient`를 실제로 동작하는 것처럼 만들고, 일부만 Stubbing하고 싶을 경우 `@Spy`를 적용할 수 있다. 이때 `when().thenReturn()`이 아닌 `doReturn().when().sendEmail()` 처럼 사용해야한다.

```java
@ExtendWith(MockitoExtension.class)  
class MailServiceTest {  
  
    @Spy  
    private MailSendClient mailSendClient;  
  
    @Mock  
    private MailSendHistoryRepository mailSendHistoryRepository;  
  
    @InjectMocks  
    private MailService mailService;  
  
    @DisplayName("메일 전송 테스트")  
    @Test  
    void sendMail() {  
        // given  
        doReturn(true)  
            .when(mailSendClient)  
            .sendEmail(anyString(), anyString(), anyString(), anyString());  
  
        // when  
        boolean result = mailService.sendMail("", "", "", "");  
  
        // then  
        assertThat(result).isTrue();  
        verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));  
    }  
}
```


[reference]
- [https://github.com/wbluke/practical-testing/tree/lesson6-3](https://github.com/wbluke/practical-testing/tree/lesson6-3)
