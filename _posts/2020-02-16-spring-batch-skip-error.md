---
layout: post
title: "Spring batch error skip"
excerpt: "Spring batch 에서 processor 에서 에러 발생시 skip 처리"
date: 2020-02-17 00:00:00 +0900
comments: true
published : true
tag:
- spring
- spring-batch
---
## 들어가며
* batch 를 개발을 하다 보면, Reader -> Processor -> Writer 구조로 개발을 할때 processor 에서 발생한 오류를 건너 뛰어야할 필요가 있습니다.
    - billing 과 같은 데이터는 모두 처리가 되어야 경우를 제외하면
    - 데이터가 없거나, 다른 도메인의 API 가 오류가 발생하는 경우에는 건너뛰어야 할 필요가 있습니다. 그렇지 않으면 batch 자체가 실패할 수도 있기 때문입니다.
* [예제](https://github.com/mayaul/spring-batch-fault-tolerant-example) 참고해주세요.

## Skip
* LIMIT 로 설정한 갯수를 초과한 에러가 발생이되면 step 은 실패하게 됩니다.
``` java
return stepBuilderFactory.get("sample step")
    // ...
    .faultTolerant()
    .skipLimit(10)
    .skip(Exception.class)
    .noRollback(NoRollbackException.class)
    .noSkip(NoSkipException.class)
    // ...
    .build();
```
* processor 중간에 몇 개 아이템에서 오류가 발생을 하면 해당 아이템은 skip 이 되므로 `chunkSize` 보다 작게 writer 에 전달이 될 수 있습니다.
* processor 가 여러개가 설정이 되어있는경우, 몇번째 processor 에서 오류가 발생했는지와 상관 없이 첫번째 processor 부터 다시 시작합니다.
    - 이 이유는 제 생각에는 특정 processor 의 오류가 이전의 processor 에 영향이 있을지도 모르기 때문에 처음부터 다시 검토하는것으로 생각이 됩니다.
    - 여러가지 케이스 중에 특정 케이스에서는, 모든 processor 가 통과를 해야 이 데이터가 검증된 데이터이고 이 경우만 writer 를 수행을 해야할 수도 있습니다.
* 이 설정과 무관하게 `ItemWriter` 에서 에러가 발생이 되면 transaction 은 rollback 됩니다.
    - rollback 을 하지 않기 위해서는 `noRollback` 으로 선언을 해야합니다.
* `skip` 에서 설정한 exception 의 하위 특정 exception 에 대해서는 skip 을 하면 안되는 exception 이 있을 수 있습니다. 그때는 `noSkip`에 설정을 해주면 됩니다.

## Retry
``` java
return stepBuilderFactory.get("sample step")
    // ...
    .faultTolerant()
    .retryLimit(10)
    .retry(Exception.class)
    // ...
    .build();
```
* `retryLimit` 의 default 는 0이다. 0 또는 1로 설정을 하면 1번만 수행이 되고 retry 하지 않습니다.
* reader 에서 오류가 발생시에는 retry 하지 않습니다
* processor 또는 writer 에서 오류가 발생하면 맨 처음 processor 부터 다시 시작 합니다.

## Skip / Retry Listener 등록하기

``` java
    @Bean
    @JobScope
    public Step sampleStep() {
        return stepBuilderFactory.get("sample step")
            // ...
            .listener(sampleBatchListener())
            // ...
            .build();
    }
    
    @Bean
    @StepScope
    public SampleBatchListener sampleBatchListener() {
        return new SampleBatchListener();
    }
```

``` java
@Slf4j
public class SampleBatchListener extends SkipListenerSupport<Long, Long> {

    @Override
    public void onSkipInRead(Throwable t) {
        log.error("reader error", t);
    }

    @Override
    public void onSkipInProcess(Long data,
                                Throwable t) {
        log.error("processor error data: {}", data, t);
    }

    @Override
    public void onSkipInWrite(Long data,
                              Throwable t) {
        log.error("writer error data: {}", data, t);
    }
}
```
* Listener 를 등록하여 각 skip 이 되었을때 로그를 기록하거나 그 이외 처리가 가능합니다.
* 위 예제는 skip 에 대한 처리이지만, retry 도 Listener 가 있습니다.
    - `org.springframework.retry.listener.RetryListenerSupport` 참고해주세요.
