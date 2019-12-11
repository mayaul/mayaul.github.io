---
layout: post
title: "Spring batch error skip"
excerpt: "Spring batch 에서 processor 에서 에러 발생시 skip 처리"
date: 2019-12-06 00:00:00 +0900
comments: true
published : false
tag:
- spring
- spring-batch
---
## 들어가며
Spring batch 를 Reader -> Processor -> Writer 구조로 개발을 할때..  
Processor 에서 에러가 발생하면 해당 Item 은 무시하고 다른 Item 에 대해서 처리를 진행을 하고 싶을때가 있다.  
이때 사용하면 괜찮을 것 같다.

