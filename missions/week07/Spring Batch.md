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