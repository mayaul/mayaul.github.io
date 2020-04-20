---
layout: post
title: "spring batch context enum "
excerpt: "spring batch context 에서 enum 사용시 문제 점"
date: 2020-04-12 00:00:00 +0900
comments: true
published : true
tag:
- spring batch context enum
---
### 들어 가면서
* spring batch 에서 database 를 지정을 해서 사용을 하면 배치의 수행 이력의 다양한 정보들이 database 에 남아 있게 됩니다.
* 이때 job context, step context 에 있던 정보들도 table 에 저장이 됩니다.
* 간단히 [예제](https://github.com/mayaul/spring-batch-context-enum) 를 만들어 보겠습니다.

### 예제 구성

* application yml
<hr/>
``` yaml
mayaul:
  batch:
    datasource:
      type: com.zaxxer.hikari.HikariDataSource
      driverClassName: org.mariadb.jdbc.Driver
      jdbcUrl: jdbc:mysql://localhost:3306/spring_batch?useSSL=false
      username: root
      password: root

spring:
  batch:
    initialize-schema: always
```
<hr/>
    
* spring.batch.initialize-schema: always 로 설정하면 spring batch 에서 사용하는 table 을 자동으로 생성을 하게 됩니다.
    - 기본값은 `EMBEDDED` 이며, `ALWAYS, EMBEDDED, NEVER` 옵션도 있습니다.

<hr/>
``` java
    public Job sampleJob() {
        return jobBuilderFactory.get("sample job")
                .listener(myJobListener())
                .start(sampleStep())
                .build();
    }

    @Bean
    @JobScope
    public JobExecutionListener myJobListener() {
        return new MyJobListener();
    }

    @Bean
    @JobScope
    public Step sampleStep() {
        return stepBuilderFactory.get("sample step")
                .listener(myStepListener())
                .tasklet((contribution, chunkContext) -> RepeatStatus.FINISHED)
                .transactionManager(new ResourcelessTransactionManager())
                .build();

    }

    @Bean
    @StepScope
    public StepExecutionListener myStepListener() {
        return new MyStepListener();
    }
```
<hr/>
* 이렇게 배치잡을 설정을 하고 JobExecutionListener 를 통해서 jobExecutionContext 에 enum 을 넣었습니다.
* 그리고 stepExecutionContext 도 아래와 비슷한 방법으로 StepExecutionListener 을 이용해서 넣었습니다.
<hr/>
```java
    public class MyJobListener implements JobExecutionListener {
    
        @Override
        public void beforeJob(JobExecution jobExecution) {
            ArrayList<TestEnum> hellos = new ArrayList<>();
            hellos.add(TestEnum.A);
            hellos.add(TestEnum.B);
            hellos.add(TestEnum.C);
    
            jobExecution.getExecutionContext().put(SampleBatchConfig.JOB_EXECUTION_CONTEXT_KEY, hellos);
        }
    
        @Override
        public void afterJob(JobExecution jobExecution) {
    
        }
    }
```
<hr/>
* 이렇게 실행 후 `BATCH_JOB_EXECUTION_CONTEXT`, `BATCH_STEP_EXECUTION_CONTEXT` 에 내용을 보면 각 context 에 있는 내용이 table 에 저장이 됩니다.
    - `BATCH_JOB_EXECUTION_CONTEXT` Table 의 `SHORT_CONTEXT` 의 내용
        ``` json
{"jobExecutionContextKey":["java.util.ArrayList",[["io.github.mayaul.batch.job.TestEnum","A"],["io.github.mayaul.batch.job.TestEnum","B"],["io.github.mayaul.batch.job.TestEnum","C"]]]}
        ```
    - `BATCH_STEP_EXECUTION_CONTEXT` Table 의 `SHORT_CONTEXT` 의 내용
        ``` json
{"batch.taskletType":"io.github.mayaul.batch.job.SampleBatchConfig$$Lambda$316/0x0000000800371840","stepExecutionContextKey":["java.util.ArrayList",[["io.github.mayaul.batch.job.TestEnum","A"],["io.github.mayaul.batch.job.TestEnum","B"],["io.github.mayaul.batch.job.TestEnum","C"]]],"batch.stepType":"org.springframework.batch.core.step.tasklet.TaskletStep"}
        ```
* 이제 `TestEnum` enum 에서 C를 없애고 다시 돌려보겠습니다.
    - 당연히 Listener 에서도 `TestEnum.C` 를 사용하는 곳을 모두 지워도 아래와 같은 오류를 만나게 됩니다.<br/>
<hr/>
```java
Caused by: com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize value of type `io.github.mayaul.batch.job.TestEnum` from String "C": value not one of declared Enum instance names: [A, B]
 at [Source: (ByteArrayInputStream); line: 1, column: 178] (through reference chain: java.util.HashMap["jobExecutionContextKey"]->java.util.ArrayList[2])
    at com.fasterxml.jackson.databind.exc.InvalidFormatException.from(InvalidFormatException.java:67) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.DeserializationContext.weirdStringException(DeserializationContext.java:1549) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.DeserializationContext.handleWeirdStringValue(DeserializationContext.java:911) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.EnumDeserializer._deserializeAltString(EnumDeserializer.java:255) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.EnumDeserializer.deserialize(EnumDeserializer.java:179) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.jsontype.impl.AsArrayTypeDeserializer._deserialize(AsArrayTypeDeserializer.java:116) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.jsontype.impl.AsArrayTypeDeserializer.deserializeTypedFromAny(AsArrayTypeDeserializer.java:71) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.UntypedObjectDeserializer$Vanilla.deserializeWithType(UntypedObjectDeserializer.java:712) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:288) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:245) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:27) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.jsontype.impl.AsArrayTypeDeserializer._deserialize(AsArrayTypeDeserializer.java:116) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.jsontype.impl.AsArrayTypeDeserializer.deserializeTypedFromAny(AsArrayTypeDeserializer.java:71) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.UntypedObjectDeserializer$Vanilla.deserializeWithType(UntypedObjectDeserializer.java:712) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.MapDeserializer._readAndBindStringKeyMap(MapDeserializer.java:529) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.MapDeserializer.deserialize(MapDeserializer.java:364) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.deser.std.MapDeserializer.deserialize(MapDeserializer.java:29) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:4014) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:3078) ~[jackson-databind-2.9.9.3.jar:2.9.9.3]
    at org.springframework.batch.core.repository.dao.Jackson2ExecutionContextStringSerializer.deserialize(Jackson2ExecutionContextStringSerializer.java:70) ~[spring-batch-core-4.1.2.RELEASE.jar:4.1.2.RELEASE]
    at org.springframework.batch.core.repository.dao.Jackson2ExecutionContextStringSerializer.deserialize(Jackson2ExecutionContextStringSerializer.java:50) ~[spring-batch-core-4.1.2.RELEASE.jar:4.1.2.RELEASE]
    at org.springframework.batch.core.repository.dao.JdbcExecutionContextDao$ExecutionContextRowMapper.mapRow(JdbcExecutionContextDao.java:325) ~[spring-batch-core-4.1.2.RELEASE.jar:4.1.2.RELEASE]
    ... 35 common frames omitted
```
<hr/>
* 이런 에러가 발생을 하는 이유는 
    - `JobParametersBuilder#getNextJobParameters` 이전 job 의 context 를 읽어와서 deserialize 를 하게 됩니다.
    - 이전 수행내역에서는 `TestEnum.C` 가 있었는데, 지금은 없으니 deserialize 를 하지 못하게 되어 에러가 발생하게 됩니다.

### 결론
* job 또는 step 에서 데이터를 공유하기 위해서 context 를 쓰는것보다는 @JobScope 나 @StepScope 를 사용해서 bean 만들어서 사용
* 그렇지 않다면, Context 에 데이터를 넣을 때는 Enum 이 아는 String 과 같은거를 쓰는걸 추천 드립니다.
* 감사합니다.  
