해당 이슈 보러가기 : https://github.com/bmm522/quiz-studio/issues/40

![](https://velog.velcdn.com/images/leon/post/83150ddb-9b7d-41ce-997e-419794fe28ac/image.png)


기존의 카테고리 별로 데이터를 쌓는 job에는 중복 코드가 있습니다. 
다음 릴리즈 버전인 v1.3.0에 새로운 카테고리 Spring, Network, Interview가 추가될 예정이었는데, 똑같은 중복코드로 추가해도 되겠지만, 추후에 확장가능성과 유지보수성을 위해 리팩토링 작업을 했습니다.

리팩토링을 시작하기 전에 로직을 먼저 파악해야 했습니다.

```java

@Component
@RequiredArgsConstructor
public class ApiDataStructureRequestTasklet implements Tasklet {

	@Qualifier("openaiRestTemplate")
	private final RestTemplate restTemplate;

	@Value("${openai.model}")
	private String model;

	@Value("${openai.api.url}")
	private String apiUrl;

	/**
	 * API를 호출하여 자료구조 관련 퀴즈를 요청하고 응답을 처리하는 메서드입니다.
	 */
	@Override
	public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext)
		throws Exception {
		StepContext stepContext = chunkContext.getStepContext();  --- 1
		final ExecutionContext jobExecutionContext = stepContext.getStepExecution()
			.getJobExecution().getExecutionContext();  --- 2

		ChatRequest request = new ChatRequest(model, CategoryTitle.DATA_STRUCTURE); --- 3
		ChatResponse response = restTemplate.postForObject(apiUrl, request, ChatResponse.class); --- 4
		String responseJson = response.getChoices().get(0).getMessage().getContent(); --- 5
		jobExecutionContext.put("response", responseJson); --- 6
		jobExecutionContext.put("categoryTitle", CategoryTitle.DATA_STRUCTURE); --- 7
		return RepeatStatus.FINISHED; 
	}


}

```

> 1~2. 다음 스텝에 데이터를 넘기기 위해 ExecutionContext 객체를 불러오는 작업입니다.
 3. chat gpt의 ai 모델과, 요청해야 하는 퀴즈의 카테고리를 인자로 넘겨주어 OPENAI API에 필요한 reuqest를 만드는 작업을 합니다.
 4. OPENAI API서버에 요청을 보낸 후 Response 받아오는 작업을 합니다.
 5. 받아온 Response에서 퀴즈 데이터만 가져오는 작업을 합니다.
 6~7. 다음 스텝에 필요한 response 데이터와 CategoryTitle을 담아주는 작업을 합니다.

모든 로직의 flow가 같고, 카테고리에 따른 request만 다르다는 점에서 저는 추상 템플릿 메서드 패턴을 적용했습니다.

```java
해당 flow를 로직으로 구성한 추상 템플릿 메서드 패턴 객체
@RequiredArgsConstructor
public abstract class ApiRequestAbstractTemplate implements Tasklet {

	@Qualifier("openaiRestTemplate")
	private final RestTemplate restTemplate;

	@Value("${openai.model}")
	private String model;

	@Value("${openai.api.url}")
	private String apiUrl;

	@Override
	public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext)
		throws Exception {

		StepContext stepContext = chunkContext.getStepContext();
		final ExecutionContext jobExecutionContext = stepContext.getStepExecution()
			.getJobExecution().getExecutionContext();
		CategoryTitle categoryTitle = getCategoryTitle(); <--- 수정된 부분
		ChatRequest request = new ChatRequest(model, categoryTitle);
		ChatResponse response = restTemplate.postForObject(apiUrl, request, ChatResponse.class);
		String responseJson = response.getChoices().get(0).getMessage().getContent();
		jobExecutionContext.put("response", responseJson);
		jobExecutionContext.put("categoryTitle", categoryTitle);
		return RepeatStatus.FINISHED;
	}

	public abstract CategoryTitle getCategoryTitle(); <--- 추가된 부분


}

```

기존 로직 flow에서 바뀐점은 CategoryTitle을 로직 내부에서 직접 선언하는 것이 아닌, 추상 메서드로 각 자식객체에서 설정하도록 한 후, 자식객체에서 직접 선언하는 형식으로 바뀌었습니다.

```java
@Component
public class ApiJavaRequestTasklet extends ApiRequestAbstractTemplate {


	public ApiJavaRequestTasklet(RestTemplate restTemplate) {
		super(restTemplate);
	}

	@Override
	public CategoryTitle getCategoryTitle() {
		return CategoryTitle.JAVA;
	}


}

@Component
public class ApiDatabaseRequestTasklet extends ApiRequestAbstractTemplate {


	public ApiDatabaseRequestTasklet(RestTemplate restTemplate) {
		super(restTemplate);
	}

	@Override
	public CategoryTitle getCategoryTitle() {
		return CategoryTitle.DATABASE;
	}
}

@Component
public class ApiDataStructureRequestTasklet extends ApiRequestAbstractTemplate {


	public ApiDataStructureRequestTasklet(RestTemplate restTemplate) {
		super(restTemplate);
	}

	@Override
	public CategoryTitle getCategoryTitle() {
		return CategoryTitle.DATA_STRUCTURE;
	}
}
```

api를 요청하는 step에서 category만 직접적으로 명시해주고 나머지의 flow를 부모 객체에 넘김으로써, 추후에 다른 카테고리를 추가하더라도 용이하게 확장하기 쉽게 되었습니다.