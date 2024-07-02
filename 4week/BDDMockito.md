# BDDMockito

## given()

아래에서 `//given` 주석에 `when().thenReturn()`을 사용하는 것은 가독성면에서 안좋은 모습이다. 따라서 이를 `BDDMockito.given()`으로 수정해주면서 해결할 수 있다.

```java
@DisplayName("메일 전송 테스트")  
@Test  
void sendMail() {  
	// given  
	given(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString())).willReturn(true);
	
	// when        
	boolean result = mailService.sendMail("", "", "", "");  

	// then  
	assertThat(result).isTrue();  
	verify(mailSendHistoryRepository, times(1)).save(any(MailSendHistory.class));  
}
```

## then()

`verify()`도 마찬가지로 `BDDMockto.then()`으로 수정해줄 수 있다.

```java
@DisplayName("메일 전송 테스트")  
@Test  
void sendMail() {  
	// given  
	given(mailSendClient.sendEmail(anyString(), anyString(), anyString(), anyString())).willReturn(true);
	
	// when        
	boolean result = mailService.sendMail("", "", "", "");  

	// then  
	assertThat(result).isTrue();  
	then(mailSendHistoryRepository).should(times(1)).save(any(MailSendHistory.class));  
}
```

