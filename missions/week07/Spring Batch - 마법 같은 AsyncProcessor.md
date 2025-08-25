### 문제 상황

단기 날씨 예보 데이터를 처리하는 `WeatherJob`의 실행 시간이 30분 가량 되었다. 외부 서비스를 호출하는 부분이 동기적으로 단일 스레드에서 실행되고 있었기에 IO 블락 비용이 큰 상황이었다. 따라서 외부 서비스를 호출하는 부분을 개선하여 Job의 실행 속도를 개선하고자 하였다.

##### 기존 WeatherJob의 Step 구조

```java
public Step weatherStep(JobRepository jobRepository, PlatformTransactionManager transactionManager) {  
    return new StepBuilder("weatherStep", jobRepository)  
            .<VilageFcstResponseDto, ShortTermWeatherDto>chunk(100, transactionManager)  
            .reader(weatherReader)  
            .processor(weatherProcessor)  
            .writer(weatherWriter)  
            .build();  
}
```

- 단일 스레드에서 작업을 수행한다.
- 모든 연산이 동기적으로 이뤄지기 때문에 IO 대기 비용이 크다.

### 해결 방법

`WeatherJob`의 데이터를 가공하는 비즈니즈 로직은 복잡도가 높지 않은 작업이다. 외부 서비스를 계속 호출하기 때문에 IO-Bound 작업으로 볼 수 있다. 그래서 외부 서비스 호출을 멀티 스레드에서 동시적으로 실행되게 하면 IO 블록 비용을 줄이는 개선이 가능할 것이라 생각했다.

Reader는 상태(내부 커서)를 가지고 있다. 따라서 단일 스레드에서만 실행되도록 하는 것이 좋겠다고 판단했다. 외부 서비스 호출을 Processor에서 하도록 하고 이를 멀티 스레딩으로 처리할 수 있도록 개선하기로 했다.

Spring Batch에서 제공하는 AsyncItemProcessor와 AsyncItemWriter를 사용하여 문제를 해결하였다. Processor를 멀티 스레딩으로 처리하는 아주 간편한 방법을 제공해준다.

**AsyncItemProcessor**
- ItemProcessor 구현체를 감싸는 데코레이터.
- 위임자로 지정된 ItemProcessor의 호출은 새 스레드에서 실행된다.
- TaskExecutor을 지정해 스레드 정책을 설정할 수 있다.
- Future를 반환한다.

**AsyncItemWriter**
- ItemWriter 구현체를 감싸는 데코레이터.
- Future를 처리해 그 결과를 위임 ItemWriter에게 전달한다.

AsyncItemProcessor와 AsyncItemWriter는 함께 사용해야 한다.

##### AsyncProcessor를 사용한 WeatherJob의 Step 구조

![[The flow of a step.png]]

```java
public Step weatherStep(JobRepository jobRepository, PlatformTransactionManager transactionManager) {  
    return new StepBuilder("weatherStep", jobRepository)  
            .<VilageFcstResponseDto, ShortTermWeatherDto>chunk(100, transactionManager)  
            .reader(weatherReader)  
            .processor(weatherProcessor)  
            .writer(weatherWriter)  
            .build();  
}

public AsyncItemProcessor<UniqueNxNy, ShortTermWeatherDto> asyncItemProcessor() {  
    var asyncProcessor = new AsyncItemProcessor<UniqueNxNy, ShortTermWeatherDto>();  
    asyncProcessor.setDelegate(processor);  
    asyncProcessor.setTaskExecutor(asyncWeatherProcessorExecutor());  
    return asyncProcessor;  
}  
 
public AsyncItemWriter<ShortTermWeatherDto> asyncItemWriter() {  
    var asyncItemWriter = new AsyncItemWriter<ShortTermWeatherDto>();  
    asyncItemWriter.setDelegate(writer);  
    return asyncItemWriter;  
}
```

#### 결과
![[job_time.png]]

외부 서비스 호출부를 멀티 스레딩으로 처리하여 작업의 실행 시간을 크게 단축할 수 있었다.

#### 참고 자료

스프링 배치 완벽 가이드, 마이클 미넬라






