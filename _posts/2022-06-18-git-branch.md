---
layout: post
title: "지금 사용하고 있는 git branch 전략"
excerpt: "전팀, 그리고 현재팀에서 선택하고 사용하고 있는 git branch 전략"
date: 2022-06-18 00:00:00 +0900
comments: true
published : true
tag:
- git
- git branch
- git branch 전략
---
### 들어가면서
* 현재 팀에서 사용하고 있고, 이전 팀에서도 사용하고 있는 git branch 전략에 대해서 소개하고자 합니다. 
* 처음에는 git flow 를 사용을 했으나, 팀의 상황과 맞지 않아 새로운 전략이 필요했습니다. 
* 조금씩 변경을 하였고, 지금은 이 방식으로 계속 진행하고 있습니다. 

### 우리의 GIT BRANCH 전략
![git branch 전략](/assets/img/posts/220618/git branch.drawio.png)
* master: 메인 브랜치
* release: 정기/비정기 배포 브랜치
  - 배포시 사용
  - feature branch 를 merge
    + 머지할때는 하나의 commit 으로 합친다.(squash merge)
  - 배포하는 날짜를 넣어 만든다. 
    + `release/220618` 등과 같이 한다.
  - 배포가 되면 master 로 merge
* hotfix: 핫픽스 배포 브랜치
  - 운영에 핫픽스로 배포를 위해서 생성하는 브랜치
  - 배포가 되면 master 로 merge
* develop, beta
  - develop, beta 환경(profile) 에 배포하기 위한 브랜치
  - 정기(비정기) 배포, 핫픽스 배포 후에 기존의 develop, beta branch 는 삭제되고 새로 생성
* feature branch
  - 각 개발 feature 를 개발하는 branch
  - 개발이 완료되거나, develop, beta 배포가 필요하면 언제든 develop, beta 로 머지해서 배포
* review branch
  - merge request 로 코드리뷰를 진행을 하면서, target branch 가 필요해서 생성
  - master 에서 checkout 해서 생성
  - 코드리뷰가 종료가 되면 review 브랜치는 feature 브랜치를 개발 담당자가 삭제 

### 자동화 할 수 있는건 자동화
* master branch merge 시점에 tag 생성
* release, hotfix branch 가 생성이 되면 master 로 머지리퀘스트 생성
* develop, beta 가 머지되면 각 환경(profile) 로 자동 배포
* feature branch 가 push 되면, review 브랜치 생성 후 머지리퀘스트 생성
  - 이때는 assignee, reviewer 자동 지정
  - assignee 는 팀내 개발자중 한명
  - reviewer 는 팀내 모든 개발자

감사합니다.