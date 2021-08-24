---
layout: post
title: "AWS CloudWatch 를 이용한 알람 설정"
excerpt: "시스템 이상 징후 알람을 누락하지 않기"
date: 2021-08-19 00:00:00 +0900
comments: true
published : true
tag:
- aws
- aws cloud watch 
- aws cloud watch alarm
- aws cloud watch metric
---
### 들어가면서
* AWS 를 사용을 하면서 시스템 지표를 확인하고 알람 설정을 하면서 경험을 적어보자 합니다.
* 여러가지 솔루션을 통해서 알람 설정을 했는데요.
  - [CloudWatch](https://aws.amazon.com/ko/cloudwatch/)
  - [ELK](https://www.elastic.co/kr/)
  - [Grafana](https://grafana.com/)
* 위 3가지를 사용을 하고 있습니다.
  - DB CPU, SQS, SNS 등의 알람은 CloudWatch 를 사용을 하였고, 
  - JVM 모니터링은 Prometheus 를 이용해서 ELK 모니터링 하면서 알람 설정을 했습니다.
  - 그 이외 Grafana 로 만든 대시보드로 알람 설정한 부분도 있습니다.
* 오늘은 CloudWatch 로 알람을 설정을 했는데, 알람을 설정한 허용수치보다 높아졌는데 알람이 오지 않았던 내용에 대해서 알아보고자 합니다. 

### 실제 설정
* 기본적으로 CloudWatch 를 통해서 알람을 설정을 해보겠습니다.
  ![npm run start](/assets/img/posts/cloudwatch/cloudwatch_aws_1.png)
  - 이렇게 설정을 하면
  ![npm run start](/assets/img/posts/cloudwatch/cloudwatch_1.png)
  - 1분동안 수집된 지표로 판단을 하게 됩니다. 

### 문제의 상황
* 이제 문제의 상황을 마주칠 수 있습니다. 
* `backfill` 현상이라고 있습니다. 이 현상은 실제 데이터가 늦게 쌓여지는 현상인데요.
  ![npm run start](/assets/img/posts/cloudwatch/cloudwatch_2.png)
  - 이렇게 `평기가간` 에는 데이터가 적재가 되어있지 않다가, 시간이 지난다음에 데이터가 적재가 되는 것 입니다.
* 이 문제는 적재가 완료된 기준으로 보면 알람이 발생할 조건이 되어있지만, 실제로 평가하는 기간에는 데이터 적재가 되어 있지 않아 알람이 오지 않게 됩니다.

### 해결 방법
* AWS에서 제안을 해준 평가방법은 2가지가 있습니다.

1. period 를 1분이 아닌 5분으로 설정한다. 
   - 이 방법의 경우 5분정도로 설정을 하면 `backfill` 현상이 있더라도 5분내에는 다 수집이 되기 때문에 알람이 보내지는 조건이 되면 알람이 발송 됩니다.
   ![npm run start](/assets/img/posts/cloudwatch/cloudwatch_3.png)
2. Datapoints to alarm 설정을 `1 out of 1` 이 아니라 `2 out of 4` 정도로 수정
   - 이 방법의 경우 `backfill` 현상이 있어도, 4개 구간을 평가하기 때문에 알람이 발송이 되게 됩니다. 
   ![npm run start](/assets/img/posts/cloudwatch/cloudwatch_4.png)

### 마치며
* 이 게시글을 쓸려고 마음을 먹은건 올해 초인데 이제서야 정리를 하게 되네요.
* 추가 질문이 있으면 댓글로 알려주세요. 감사합니다.

### 참고 사이트
> https://aws.amazon.com/ko/premiumsupport/knowledge-center/cloudwatch-trigger-metric/    
> https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html