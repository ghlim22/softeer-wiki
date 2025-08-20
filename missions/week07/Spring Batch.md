#### Job
Job: 전체 batch 작업의 캡슐화된 표현

#### Job Instance
A `JobInstance` refers to the concept of a logical job run.
Job Instance: Job의 하나의 논리적 실행. 하나의 Job instance는 여러 executions를 가질 수 있다.

However, using the same `JobInstance` determines whether or not the “state” (that is, the `ExecutionContext`, which is discussed later in this chapter) from previous executions is used. Using a new `JobInstance` means “start from the beginning,” and using an existing instance generally means “start from where you left off”.
#### [](https://docs.spring.io/spring-batch/docs/5.0.4/reference/html/domain.html#jobParameters)

같은 JobInstance를 사용하면 이전 상태에서 이어서 시작하는 것이고, 새로운 JobInstance를 실행하면 새로운 상태에서 작업을 시작하는 것이다.


#### Job Parameters
JobParameters: JobInstance들은 JobParameter를 통해 다른 JobInstane들과 구분된다.
`JobInstance` = `Job` + identifying `JobParameters`

#### Job Execution
A `JobExecution` refers to the technical concept of a single attempt to run a Job.

A `Job` defines what a job is and how it is to be executed, and a `JobInstance` is a purely organizational object to group executions together, primarily to enable correct restart semantics. A `JobExecution`, however, is the primary storage mechanism for what actually happened during a run and contains many more properties that must be controlled and persisted, as the following table shows:

#### Step
A `Step` is a domain object that encapsulates an independent, sequential phase of a batch job.

execution context: 
executions 간에 보존되어야 하는 데이터.
The “property bag” containing any user data that needs to be persisted between executions.


#### Test


```java
@SpringBatchTest
@SpringBootTest
@ExtendWith(OutputCaptureExtension.class)
class BillingJobApplicationTests {
  @Autowired
  private Job job;

  @Autowired
 .private JobLauncher jobLauncher;
 
    @Test
    void testJobExecution(CapturedOutput output) throws Exception {
        // given
        JobParameters jobParameters = new JobParametersBuilder()
            .addString("input.file", "/some/input/file")
            .toJobParameters();
    
        // when
        // ** Update the following line:
        JobExecution jobExecution = this.jobLauncherTestUtils.launchJob(jobParameters);
    
        // then
        Assertions.assertTrue(output.getOut().contains("processing billing information from file /some/input/file"));
        Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus());
    }
}
```

```java
@test 
void testjobexecution() throws exception {
// given 
JobParameters jobparameters = new jobparametersbuilder() .addstring("input.file", "src/main/resources/billing-2023-01.csv") .tojobparameters(); 
// when 
JobExecution jobexecution = this.joblaunchertestutils.launchjob(jobparameters);
// then 
assertions.assertequals(exitstatus.completed, jobexecution.getexitstatus());
assertions.asserttrue(files.exists(paths.get("staging", "billing-2023-01.csv"))); 
}

```

```
[~/exercises] $ ./mvnw clean test -Dspring.batch.job.enabled=false
```
Job이 배치에서 자동 실행되는 것을 막기 위한 커맨드 (이게 없으면 테스트 때 job이 한 번 실행되고, 배치에서 job을 또 실행하여 job이 두 번 실행된다.)

#### Job
```java
@Bean
public Job myJob(JobRepository jobRepository, Step step1, Step step2) {
  return new JobBuilder("job", jobRepository)
    .start(step1)
    .next(step2)
    .build();
}
```

#### Step
```java
@Bean
public Step taskletStep(JobRepository jobRepository, Tasklet tasklet, PlatformTransactionManager transactionManager) {
  return new StepBuilder("step1", jobRepository)
    .tasklet(tasklet, transactionManager)
    .build();
}
```

```java
@Bean
public Step partitionedtStep(JobRepository jobRepository, Partitioner partitioner) {
  return new StepBuilder("step1", jobRepository)
    .partitioner("worker", partitioner)
    .build();
}
```

```java
package example.billingjob;

  

import java.nio.file.Files;

import java.nio.file.Path;

import java.nio.file.Paths;

import java.nio.file.StandardCopyOption;

  

import org.springframework.batch.core.JobParameters;

import org.springframework.batch.core.StepContribution;

import org.springframework.batch.core.scope.context.ChunkContext;

import org.springframework.batch.core.step.tasklet.Tasklet;

import org.springframework.batch.repeat.RepeatStatus;

  

public class FilePreparationTasklet implements Tasklet {

  

@Override

public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {

JobParameters jobParameters = contribution.getStepExecution().getJobParameters();

String inputFile = jobParameters.getString("input.file");

Path source = Paths.get(inputFile);

Path target = Paths.get("staging", source.toFile().getName());

Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);

return RepeatStatus.FINISHED;

}

}
```

```java
@Bean public Step step1(JobRepository jobRepository, JdbcTransactionManager transactionManager) { 
return new StepBuilder("filePreparation", jobRepository) .tasklet(new FilePreparationTasklet(), transactionManager) .build(); 
}
```

```java
@Bean public Job job(JobRepository jobRepository, Step step1) { 
return new JobBuilder("BillingJob", jobRepository) .start(step1) .build(); 
}
```

```java
@Test void testJobExecution() throws Exception { 
// given 
JobParameters jobParameters = new JobParametersBuilder() .addString("input.file", "src/main/resources/billing-2023-01.csv") .toJobParameters(); 
// when JobExecution jobExecution = this.jobLauncherTestUtils.launchJob(jobParameters); 
// then 
Assertions.assertEquals(ExitStatus.COMPLETED, jobExecution.getExitStatus()); Assertions.assertTrue(Files.exists(Paths.get("staging", "billing-2023-01.csv"))); 
}
```

run the tests
```java
[~/exercises] $ ./mvnw clean test -Dspring.batch.job.enabled=false
```

#### 청크-단위 처리

```java
@Bean
public Step ingestFile(
              JobRepository jobRepository,
              PlatformTransactionManager transactionManager,
              FlatFileItemReader<BillingData> billingDataFileReader,
              JdbcBatchItemWriter<BillingData> billingDataTableWriter) {

    return new StepBuilder("fileIngestion", jobRepository)
            .<BillingData, BillingData>chunk(100, transactionManager)
            .reader(billingDataFileReader)
            .writer(billingDataTableWriter)
            .build();
}
```


```java
public class FilePreparationTasklet implements Tasklet {

  

@Override

public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {

JobParameters jobParameters = contribution.getStepExecution().getJobParameters();

String inputFile = jobParameters.getString("input.file");

Path source = Paths.get(inputFile);

Path target = Paths.get("staging", source.toFile().getName());

Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);

return RepeatStatus.FINISHED;

}
}

```

```java
@Bean

public Step step2(

JobRepository jobRepository, JdbcTransactionManager transactionManager,

ItemReader<BillingData> billingDataFileReader, ItemWriter<BillingData> billingDataTableWriter) {

return new StepBuilder("fileIngestion", jobRepository)

.<BillingData, BillingData>chunk(100, transactionManager)

.reader(billingDataFileReader)

.writer(billingDataTableWriter)

.build();

}

```

```java
@Bean public Job job(JobRepository jobRepository, Step step1, Step step2) 
{ 
return new JobBuilder("BillingJob", jobRepository) .start(step1) .next(step2) .build(); 
}
```


#### 병렬 처리

https://medium.com/@sunkyung/spring-batch-%EB%B3%91%EB%A0%AC%EC%B2%98%EB%A6%AC-with-partitioning-step-cc0f94ee0f66

