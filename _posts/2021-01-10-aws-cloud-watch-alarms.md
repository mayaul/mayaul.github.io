---
layout: post
title: "AWS CloudWatch 를 이용한 알람 설정"
excerpt: "시스템 이상 징후 알람을 누락하지 않기"
date: 2021-01-10 00:00:00 +0900
comments: true
published : false
tag:
- aws
- aws cloud watch 
- aws cloud watch alarm
- aws cloud watch metric
---
### 들어가면서
* AWS 를 사용을 하면서 시스템 지표를 확인하고 알람 설정을 하면서 경험을 적어보자 합니다.
* 저희팀은 여러가지 솔루션을 통해서 알람 설정을 했는데요.
  - [CloudWatch](https://aws.amazon.com/ko/cloudwatch/)
  - [ELK](https://www.elastic.co/kr/)
  - [Grafana](https://grafana.com/)
* 위 3가지를 사용을 하고 있습니다.
  - DB CPU, SQS, SNS 등의 알람은 CloudWatch 를 사용을 하였고, 
  - JVM 모니터링은 Prometheus 를 이용해서 ELK 모니터링 하면서 알람 설정을 했습니다.
  - 그 이외 Grafana 로 만든 대시보드로 알람 설정한 부분도 있습니다.
* 오늘은 CloudWatch 로 알람을 설정을 했는데, 알람을 설정한 허용수치보다 높아졌는데 알람이 오지 않았던 내용에 대해서 알아보고자 합니다. 


### 참고 사이트
> https://aws.amazon.com/ko/premiumsupport/knowledge-center/cloudwatch-trigger-metric/    
> https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html