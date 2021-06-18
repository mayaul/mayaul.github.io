---
layout: post
title: "IntelliJ(with gradle) 에서 JAVA 개발환경"
excerpt: "build 와 run 에 만난 시행착오와 설정들"
date: 2021-06-18 00:00:00 +0900
comments: true
published : true
tag:
- intelliJ
- gradle 

---

### gradle heap memory 
* gradle 의 heap memory 가 부족하면 만나는 에러는 아래와 같습니다.
`GC overhead limit exceeded` 라는 오류를 만나실 수 있습니다.
  
#### 해결방법
* 해결 방법 1
    - project 에 `gradle.properties` 파일을 만들고 heap memory 설정
    프로젝트 root 에 파일을 생성을 하고 아래 코드 추가 
        ```groovy
        org.gradle.jvmargs=-Xmx4096m
        ```
* 해결 방법 2(제가 선택한 방법)
    - project 마다 `gradle.properties` 파일을 만들기 귀찮아서 기본 설정을 변경
    보통 gradle home 디렉토리는 `/{USER_HOME}/.gradle` 입니다. 여기서 위에 말한 `gradle.properties` 파일을 추가하고 아래내용 추가
        ```groovy
        org.gradle.jvmargs=-Xmx4096m
        ```     

### java: Compilation failed: internal java compiler error
#### 해결방법
* `Preferences | Build, Execution, Deployment | Compiler` 로 이동해서 `Shared build process heap size` 변경
  - 저는 기본이 700으로 되어있었고, 2000으로 변경했더니 빌드 성공 했습니다.
    ![!JDK configuration](/assets/img/posts/develop_configuration/compiler_shared_heap_size.png)
  