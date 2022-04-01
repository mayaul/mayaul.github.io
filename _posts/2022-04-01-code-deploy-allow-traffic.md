---
layout: post
title: "Code Deploy 에서 allow traffic"
excerpt: "allow traffic 에서 멈춰있을때 확인해보면 좋은 것"
date: 2022-04-01 00:00:00 +0900
comments: true
published : true
tag:
- aws
- aws code deploy 
- aws code deploy allow traffic
---
### 들어가면서
* 제가 현재 재직중인 회사에서는 AWS 의 code deploy 를 활용을해서 배포를 하고 있습니다.
* 오늘 배포를 하면서 allow traffic 에서 멈춰서 더이상 진행이 되고 있지 않았던 현상을 경험을 했고, 해당 원인과 해결한 방법에 대해서 말씀드리겠습니다. 

### 문제의 상황
* Code deploy 에서 `ValidateService` 까지는 성공을 했으나, `AllowTraffic` 에서 멈춰있는 것을 볼 수 있습니다.
![allow traffic 문제](/assets/img/posts/220401/allow traffic.png)

### 문제의 원인
* target group 에서도 EC2 로 health check 를 하는데 그것이 실패를 했습니다. 
* code deploy 설정에서 `ValidateService` 의 health check 의 경우는 성공을 하였고, target group 에서 하는 health check 가 실패했습니다.
![target group 확인](/assets/img/posts/220401/target group.png)
* 이번에 갑자기 실패를 한 원인의 경우는 기존에는 worker service 에서 nginx 가 설치되어 있었고, nginx -> spring boot 되었습니다.
    - 굳이 nginx 가 필요하지 않아, nginx 를 없애고 code deploy 에서 health check 의 경우 spring boot 의 8080으로 health check 를 할 수 있도록 수정을 했습니다.
    - 그러나, target group 의 health check 설정은 변경은 하지 않았고 그래서 `AllowTraffic` 에서 멈춰 있었습니다.

### 해결 방법
1. target group 에 있는 설정처럼 80 port 로도 health check 가 성공하게 한다.
    - nginx 를 다시 설치를 하던가, spring boot 서비스를 80 port 로 해야합니다.
2. target group 도 동일하게 8080 port 로도 health check 를 성공을 하게 한다.
    - target group 의 health check 하는 default port 는 변경을 할 수 없어, 새로 만들어서 load balancer 에 연결을 해야합니다.

### 저의 선택은?
* 저는 굳이 worker 서비스에는 nginx 가 필요 없어서 2번 방법으로 선택을 했습니다.