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
* Java code 를 읽기 쉽게 변환해 준다.

#### [DbUnit Extractor](https://plugins.jetbrains.com/plugin/7958-dbunit-extractor)
* DbUnit 테스트를 진행을 하면 DB 의 데이터를 쉽게 xml 로 변환을 하고 싶을 요구가 생길때가 있다.
이때 사용하면 괜찮은 plugin
* DBUnit 과 이 plugin 에 대해서 자세히 알고싶다면 링크 참고 ([DBUnit](http://woowabros.github.io/experience/2019/11/06/db-unit.html))

#### [Flyway Migration Creation](https://plugins.jetbrains.com/plugin/8597-flyway-migration-creation)
* Flayway 를 사용을 하면 자동으로 버전된 sql 파일을 만들고 싶을때가 있다. 이때 사용하면 좋은 plugin

#### [GitToolBox](https://plugins.jetbrains.com/plugin/7499-gittoolbox)
* 가끔 내가 어떤 branch 에서 작업을 하고 있는지 모르고 develop, master 등에 작업을 하고 습관적으로 push 하는것을 예방을 하고 현재 branch 를 알려주는 plugin
* branch 별로 pull / push 할 것이 있는지도 알려준다.
* 그 이외에도 추가적으로 많은 기능이 있지만.. 저는 주로 이 기능만 써요

#### [Grep Console](https://plugins.jetbrains.com/plugin/7125-grep-console)
* 무조건 필수.. 이거 없으면 intellij console log 볼때 너무 힘들어요

#### [Lombok](https://plugins.jetbrains.com/plugin/6317-lombok)
* lombok 을 사용한다면 필수

#### [Material Theme UI](https://plugins.jetbrains.com/plugin/8006-material-theme-ui)
* 뭔가 화려한 intellij theme 를 원한다면 좋은 plugin

#### [Rainbow Brackets](https://plugins.jetbrains.com/plugin/10080-rainbow-brackets)
* `{ ... }`, `(((( ... ))))` 가 많다면 무지개색상으로 구분을 해준다.

#### [Request mapper](https://plugins.jetbrains.com/plugin/9567-request-mapper)
* Rest API 를 찾기 쉽게 해준다.

#### [RestfulToolkit](https://plugins.jetbrains.com/plugin/10292-restfultoolkit)
* RestfulToolkit 이라는 view 가 생기며, 거기서 검색을 하면 full url 을 복사할 수 있다.
* Controller class 와 method 에서 rest api 분리해놓고 사용한다면 full url 복사시 유용하다.

#### [Sexy Editor](https://plugins.jetbrains.com/plugin/1833-sexy-editor)
* editor 배경화면을 언어별로 설정을 할 수 있다.

#### [SonarLint](https://plugins.jetbrains.com/plugin/7973-sonarlint)
* sonarqube 를 사용을 한다면, intellij 에서 분석을 돌려 볼 수 있다.
#### [String Manipulation](https://plugins.jetbrains.com/plugin/2162-string-manipulation)
* string 관련 반복성 작업에 대한 변경작업이 필요 할때 유용한 plugin

#### [Upsource Integration](https://plugins.jetbrains.com/plugin/7431-upsource-integration)
* upsource 를 이용하여, code review 를 한다면 필수 plugin

## Setting
#### Preferences new projects
* 새로운 프로젝트를 열거나 만들때, 기본적으로 설정해야하는 것들이 있다.
* 대표적으로 `Preferences | Build, Execution, Deployment | Compiler | Annotation Processors` 에 `Enable annotation processing` 이다.
* `File -> Other Settings -> Preferences new projects...` 라는 메뉴가 있다.
* ![Preferences new projects](/assets/img/posts/intellij_setting_plugin/preferences-new-project.png) 에서 설정을 해주면 신규 프로젝를 생성을 할때 기본값으로 설정 할 수있다.

#### Font
* [D2coding]('https://github.com/naver/d2codingfont') 몇가지 서체를 써봤지만, 한글도 잘 나오고 개발에 필요한 `i,I,l` 와 `0, O`구분이 명확한다

#### Gradle
* path: `Build, Execution, Deployment | Build Tools | Gradle`
* IntelliJ IDEA 특정버전에 gradle 로 변경되었다.
* Intellij 에서 TestCode 실행시, 기대결과가 틀려도 Read, 에러가 나도 Red 로 표시된다.
* 기대결과가 틀렸을 때, Yellow 로 표시하고 싶다면 `Intellij IDEA` 로 변경한다.<br/>
<img src="/assets/img/posts/intellij_setting_plugin/intellij-gradle.png" />

#### Annotation Processors
* path: `Preferences | Build, Execution, Deployment | Compiler | Annotation Processors` 에 `Enable annotation processing`
![Enable annotation processing](/assets/img/posts/intellij_setting_plugin/enable-annotation-processing.png)

#### Task
* path: `Preferences | Tools | Tasks`
* 회사에서는 jira 를 사용하고 있다.
    - jira 의 이슈번호를 이용해서 git feature branch 를 생성을 하고, jira ticket 번호를 commit message 를 넣어야 한다면 설정하면 좋다.
* Changelist name format: `${id} ${summary}`
    - `Version control` 에 있는 `Local Changes` 에 changelist 를 자동으로 만들어 준다.
* Feature branch name format: `feature/${id}`
    - 자동으로 지정된 format 으로 branch 이름을 만들어 준다.
* `Preferences | Tools | Tasks | Servers` 여기로 가서 jira 서버 설정을 해주면 된다.

#### Issue Navigation
`Preferences | Version Control | Issue Navigation`
* 이전회사와 현 회사에서 git commit 을 할 때, jira 번호를 필수로 넣고 있다. 이 기능을 활성화 하면, git log 를 볼 때 바로 jira 내용으로 이동을 할 수 있다.
    - Issue ID: [A-Z]+\-\d+
    - Issue Link: https://jira.woowa.in/browse/$0
        + git commit 시에 `[`,`]` 으로 jira 번호를 감싸고 있다.

#### Spock(groovy) 에서 자동으로 변수를 만들 때 def 가 아닌 정확한 타입으로 명시하기
* groovy 에서 변수를 자동으로 생성을 하면 `def` 라는 type 으로 생성이 된다.
** 이때 `shift+tap` 을 눌러 def 로 포커스를 변경을 한다음에 해당 변수의 정확한 타입으로 변경을 하면 그 다음부터는 기억을 해서 모든 변수 생성시에 정확한 타입으로 생성이 된다.

