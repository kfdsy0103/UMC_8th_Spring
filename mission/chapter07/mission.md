## 실습 미션
- ![실습1.png](%EC%8B%A4%EC%8A%B51.png)
- ![실습2.png](%EC%8B%A4%EC%8A%B52.png)


## 시니어 미션
1. 에러 템플릿 작성
```java
@Getter
@RequiredArgsConstructor
public enum DiscordErrorLogTemplate {

	INTERNAL_SERVER_ERROR(
		"""
		❗ **[500 Internal Server Error]**
		📅 발생 시각: ${timestamp}
		🌐 요청 URL: ${requestUrl}
		📄 예외 메시지:
		```
		${exceptionMessage}
		```
		"""
	);

	private final String template;
}
```

2. 디스코드 알람 발행 클래스 작성
```java
@Component
public class DiscordNotifier {

	@Value("${discord.webhook.url}")
	private String discordWebhookUrl;
	private final RestTemplate restTemplate = new RestTemplate();

	public void sendNotification(String timeStamp, String exceptionMessage, String requestUrl) {
		// 1. 템플릿 바인딩 파라미터 설정
		Map<String, String> parameters = Map.of("timestamp", timeStamp, "requestUrl", requestUrl, "exceptionMessage", exceptionMessage);
		// 2. 템플릿 양식과 파라미터로 디스코드 전송
		sendDiscordMessage(DiscordErrorLogTemplate.INTERNAL_SERVER_ERROR.getTemplate(), parameters);
	}

	private void sendDiscordMessage(String template, Map<String, String> parameters) {

		// 1. 템플릿에 파라미터 바인딩
		for (Map.Entry<String, String> entry : parameters.entrySet()) {
			template = template.replace("${" + entry.getKey() + "}", entry.getValue());
		}
		Map<String, String> body = Map.of("content", template);

		// 2. 디스코드로 post 요청
		restTemplate.postForEntity(discordWebhookUrl, body, String.class);
	}
}
```

3. Advice에서 웹훅으로 post 전송
```java
	@ExceptionHandler
	public ResponseEntity<Object> exception(Exception e, WebRequest request) {
		e.printStackTrace();

		// 발생 시각 및 URL 파싱
		String timeStamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
		String requestUrl = request.getDescription(false).replace("uri=", "");

		// Discord 알림 전송
		discordNotifier.sendNotification(
			timeStamp,
			e.getMessage() == null ? e.toString() : e.getMessage(),
			requestUrl
		);

		return handleExceptionInternalFalse(
			e,
			ErrorStatus._INTERNAL_SERVER_ERROR,
			HttpHeaders.EMPTY,
			ErrorStatus._INTERNAL_SERVER_ERROR.getHttpStatus(),
			request,
			e.getMessage()
		);
	}
```


- 코드 디자인 설명
  - 에러 로그 템플릿은 ENUM 타입으로 관리하였다.
  - 다른 종류의 알람을 발행하기 용이하도록 위처럼 코드를 작성하였다.
  - 만약 새로운 종류의 알람을 발행한다면 아래의 과정만 거치면 된다.
    - DiscordErrorLogTemplate에 새로운  종류의 템플릿 생성
    - DiscordNotifier에 메서드 생성 후, 원하는 템플릿을 sendDiscordMessage() 파라미터로 전달


- 실행 사진
  - ![시니어과정1.png](%EC%8B%9C%EB%8B%88%EC%96%B4%EA%B3%BC%EC%A0%951.png)
  - ![시니어과정2.png](%EC%8B%9C%EB%8B%88%EC%96%B4%EA%B3%BC%EC%A0%952.png)