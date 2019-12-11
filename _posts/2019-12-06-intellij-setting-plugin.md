---
layout: post
title: "Intellij setting 과 plugin"
excerpt: "개발하면서 유용한 Intellij plugin 과 하면 좋은 setting"
date: 2019-12-06 00:00:00 +0900
comments: true
changefreq : daily
tag:
- intellij
- setting
- plugin
---
## 들어가며
 Intellij IDEA 를 이용해서 Java(Spring) 개발을 하면서 설정을 하면 좋은 것과 유용한 plugin 모음
 
## Plugin
#### [Advanced Java Folding](https://plugins.jetbrains.com/plugin/9320-advanced-java-folding)
#### [DbUnit Extractor](https://plugins.jetbrains.com/plugin/7958-dbunit-extractor)
#### [Flyway Migration Creation](https://plugins.jetbrains.com/plugin/8597-flyway-migration-creation)
#### [GitToolBox](https://plugins.jetbrains.com/plugin/7499-gittoolbox)
#### [Grep Console](https://plugins.jetbrains.com/plugin/7125-grep-console)
#### [Lombok](https://plugins.jetbrains.com/plugin/6317-lombok)
#### [Material Theme UI](https://plugins.jetbrains.com/plugin/8006-material-theme-ui)
#### [Rainbow Brackets](https://plugins.jetbrains.com/plugin/10080-rainbow-brackets)
#### [Request mapper](https://plugins.jetbrains.com/plugin/9567-request-mapper)
#### [RestfulToolkit](https://plugins.jetbrains.com/plugin/10292-restfultoolkit)
#### [Sexy Editor](https://plugins.jetbrains.com/plugin/1833-sexy-editor)
#### [Show REST Services](https://plugins.jetbrains.com/plugin/9503-show-rest-services)
#### [SonarLint](https://plugins.jetbrains.com/plugin/7973-sonarlint)
#### [String Manipulation](https://plugins.jetbrains.com/plugin/2162-string-manipulation)
#### [Upsource Integration](https://plugins.jetbrains.com/plugin/7431-upsource-integration)

## Setting
#### Font
[D2coding]('https://github.com/naver/d2codingfont') 몇가지 세처를 써봤지만, 한글도 잘 나오고 개발에 필요한 `i,I,l` 와 `0, O`구분이 명확한다
#### Gradle 
`Build, Execution, Deployment | Build Tools | Gradle` 에 설정이 IntelliJ IDEA 특정버전에 gradle 로 변경되었다.<br/>
Intellij 에서 TestCode 실행시, 기대결과가 틀려도 Read, 에러가 나도 Red 로 표시된다. <br/>
기대결과가 틀렸을 때, Yellow 로 표시하고 싶다면 `Intellij IDEA` 로 변경한다.<br/>
<img src="/assets/img/posts/intellij_setting_plugin/intellij-gradle.png" />
#### Preferences new projects
새로운 프로젝트를 열거나 만들때, 기본적으로 설정해야하는 것들이 있다.  
대표적으로 `Preferences | Build, Execution, Deployment | Compiler | Annotation Processors` 에 `Enable annotation processing` 이다.
![Enable annotation processing](/assets/img/posts/intellij_setting_plugin/enable-annotation-processing.png)
`File -> Other Settings -> Preferences new projects...` 라는 메뉴가 있다.
![Preferences new projects](/assets/img/posts/intellij_setting_plugin/preferences-new-project.png) 에서 설정을 해주면 신규 프로젝를 생성을 할때 기본값으로 설정 할 수있다.
