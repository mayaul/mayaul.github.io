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
* elastic
    - [https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
        + docker image: `docker pull docker.elastic.co/elasticsearch/elasticsearch:7.7.0'
        + docker run command
        ``` bash
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.7.0
        ```
* kibana
    - [https://www.elastic.co/guide/en/kibana/current/docker.html](https://www.elastic.co/guide/en/kibana/current/docker.html)
        + docker image: `docker pull docker.elastic.co/kibana/kibana:7.7.0'
        + docker run command
        ``` bash
docker run --link YOUR_ELASTICSEARCH_CONTAINER_NAME_OR_ID:elasticsearch -p 5601:5601 {docker-repo}:{version}
        ```        
* logstash 
    - [https://www.elastic.co/guide/en/logstash/current/docker.html](https://www.elastic.co/guide/en/logstash/current/docker.html)
        + docker image: `docker pull docker.elastic.co/logstash/logstash:7.7.0`
        + docker run command
        ``` bash
       
        ```
* filebeat
    - [https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html](https://www.elastic.co/guide/en/beats/filebeat/current/running-on-docker.html)
        + docker image: `docker pull docker.elastic.co/beats/filebeat:7.7.0`
        + docker run command
        ``` bash
docker run \
docker.elastic.co/beats/filebeat:7.7.0 \
setup -E setup.kibana.host=kibana:5601 \
-E output.elasticsearch.hosts=["elasticsearch:9200"]
        ```
 
### 샘플 파일
* elastic search sample 파일 다운로드 받기 
    - [https://www.elastic.co/guide/en/kibana/7.6/tutorial-build-dashboard.html#load-dataset](https://www.elastic.co/guide/en/kibana/7.6/tutorial-build-dashboard.html#load-dataset)
