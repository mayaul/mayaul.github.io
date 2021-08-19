---
layout: post
title: "elastic stack 처음 사용 해볼 떄"
excerpt: "elastic stack 을 처음 사용시 참고 자료"
date: 2020-05-26 00:00:00 +0900
comments: true
published : true
tag:
- elastic stack
- elastic
- kibana
- filebeat
- logstash 
---
### 설치하기
* brew 설치
```bash
brew tap elastic/tap
brew install elastic/tap/elasticsearch-full
brew install elastic/tap/kibana-full
brew install elastic/tap/filebeat-full
brew install elastic/tap/logstash-full
```

### 서비스 시작
#### ElasticSearch
```bash
vi ~/.profile
#.profile 파일 수정
... 
export $ES_HOME="/usr/local/var/homebrew/linked/elasticsearch-full"
...
#.profile 파일 수정
source ~/.profile
$ES_HOME/bin/elasticsearch -d -p pid
#-d 데몬모드, -p는 pid 를 저장하기 위해서 
```
`http://localhost:9200` 으로 확인

#### Kibana
`brew services start elastic/tap/kibana-full`

#### Logstash
`brew services start elastic/tap/logstash-full`
 
### 샘플 파일
* elastic search sample 파일 다운로드 받기 
    - [https://www.elastic.co/guide/en/kibana/7.6/tutorial-build-dashboard.html#load-dataset](https://www.elastic.co/guide/en/kibana/7.6/tutorial-build-dashboard.html#load-dataset)
### 참고 링크
* https://www.elastic.co/guide/en/elasticsearch/reference/7.13/brew.html
