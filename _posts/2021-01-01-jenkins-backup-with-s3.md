---
layout: post
title: "Jenkins 정보를 S3에 백업"
excerpt: "JCasC 와 EFS 를 포기하고 S3로 간 여정"
date: 2021-01-01 00:00:00 +0900
comments: true
published : true
tag:
- jenkins
- jenkins backup
- jenkins backup s3
---
### 들어가면서
* 전/현회사에서 배치서버로 Jenkins 를 사용을 하고 있습니다.
* Jenkins 배치서버가 갑작스럽게 죽거나, 복구가필요할 때 백업을 해놓은 것이 없으면 당황 할 수밖에 없습니다. 
* Jenkins 를 백업하고 복구하는 방법은 다양한 방법이 있습니다.
    - 첫번째로 가장 최근에 나온 JCasC(Jenkins Configuration as Code) 라는 방법을 이용을 하는 겁니다.
        + 이 방법을 처음으로 진행을 하다가 포기를 한 이유는 유지보수성이 너무나 떨어졌습니다. 
        + IntelliJ 로 개발하기에 code highlighting, 자동완성등의 기능이 지원이 되지 않아 한쪽화면에는 가이드 문서를 보고 작성을 해야했습니다. 
        + 또, Jenkins 에서 직접 배치를 수정을 하는일이 생기면, code 와 실제 적용된 버전의 차이가 생기는 것 또한 문제점 중 하나로 생각이 됩니다.
    - 두번째로 EFS 를 이용하여 진행을 하는 거였습니다. 
        + 이 방법의 경우는 기존과 다르게 배치의 성능이 너무나 떨어지는게 문제였습니다. 
        + 그래서 바로 포기를 했었고, 상세 성능을 비교한 수치는 당시에 별도로 기록을 남기지 않아없네요.
    - 세번째로 [thinBackup plugin 을 사용](https://blog.leocat.kr/notes/2018/04/25/jenins-backup-configuration) 을 하는 방법입니다.
        + 저는 S3를 이용해서 별도의 공간에 백업을 하고 싶어 이 방법 말고 다른 방법을 찾아봤습니다. 

### 백업과 복구 
#### 백업를 진행하는 방식
1. Jenkins 에 배치를 하나 만들어서 Jenkins 의 주요 폴더를 tar 로 압축을 합니다.
2. 백업된 tar 파일을 S3에 업로드를 합니다. 
3. S3 설정을 이용해서, 최근 7일에 대한 파일만 남기도록 설정을 합니다. 

#### 복구를 진행하는 방식
1. 저는 [Packer](https://woowabros.github.io/experience/2019/04/20/ami-packer-ansible.html) 를 이용해서 AMI 를 만들었습니다. 
    - 회사의 표준 AMI 에 Jenkins 를 설치를 하고 설정을 한다음에 AMI로 만들었습니다.   
    - 이렇게 하면 Jenkins 에 문제가 생겼을때 만든 AMI 를 이용해서 바로 EC2를 만들고, 백업된 tar 파일을 이용해서 압축만 풀면 바로 복구가 가능합니다.
2. Jenkins EC2 까지 없어졌다면, 1번에서 만든 AMI 를 이용해서 EC2를 생성을 합니다.
3. S3 에 있는 tar 파일을 다운을 받고 Jenkins 폴더에 압축을 풉니다.
4. Jenkins 를 가동을 합니다. 

### 백업에 대한 상세 내용
#### Jenkins 에 tar 압축하는 배치 만들기
* 운영의 Jenkins 는 보통 master, slave 구조로 설정을 진행을 합니다.
    - master 는 jenkins 를 관리만 진행을 하고, 실제적인 배치는 slave 에서 수행을 하게 됩니다.
* Jenkins 의 여러 배치와 설정파일의 정보는 master 에 있으니, 백업 배치는 master 에서 수행이 되어야 합니다. 
![jenkins label expression](/assets/img/posts/jenkins/jenkins_1.png)

* 주기적으로 수행을 진행을 하기 위해 `Build periodiclly` 설정을 해서 주기적으로 수행을 하게 합니다. 
    - 새벅새간대 중에서 배치잡 수행이 거의 없는 시간으로 하기위해 저는 새벽4시에 진행을 하게 했습니다. 
![build periodically](/assets/img/posts/jenkins/jenkins_2.png)
      
* shell 명령어를 이용을 해서 실제 백업을 진행을 하게 합니다. 
    - 전체를 다 압축하면 너무나 압축파일의 용량이 커서, 숨김파일, 로그, 워크스페이스는 제외를 하고 백업을 진행했습니다.
![execute shell](/assets/img/posts/jenkins/jenkins_3.png)
``` shell
echo 'tar $JENKINS_HOME directory'
set +e 
tar -cvf jenkins_backup$(date +%Y%m%d).tar $JENKINS_HOME --exclude=".*" --exclude="logs" --exclude="workspace" .
exitcode=$?
if [ "$exitcode" != "1" ] && [ "$exitcode" != "0" ]; then
exit $exitcode
fi
set -e
echo 'Upload jenkins_backup.tar to S3 bucket'
aws s3 cp jenkins_backup$(date +%Y%m%d).tar s3://batch-backup/
echo 'Remove files after succesful upload to S3'
rm -rf *
```

#### s3 bucket 만들고, 주기적으로 삭제하게 설정하기
* s3 에 bucket 를 생성을 한 후 주기적으로 삭제를 진행을 해야합니다.
* bucket 을 어떻게 생성을 하는지는 건너띄고, 주기적 삭제 설정하는 부분만 설명 드리겠습니다.
* bucket 에 들어가면 `Management` 탭을 선택하면 아래와 같은 스샷을 볼 수 있습니다.
![bucket configuration](/assets/img/posts/jenkins/jenkins_4.png)
* bucket 에 새로은 lifecycle rule 를 생성을 진행을 해주면 됩니다. 
    - 저는 아래 스샷처럼 설정을 진행을 했습니다.
![bucket configuration](/assets/img/posts/jenkins/jenkins_5.png)
      
### 복구에 대한 상세 내용
#### EC2 생성
* Packer 로 만든 AMI 를 이용해서 EC2 생성
* EC2 에 적용을 해야하는 `IAM Role`과 `Security Group` 의 경우에는 별도의 문서로 기록을 해놔야 합니다.
    - 해당 기록된 내역으로 설정해서 EC2를 생성합니다.

#### 압축파일 받기
* aws cli 명령으로 s3 압축 파일 리스트 찾기

``` shell
$ aws s3 ls batch-backup
2020-10-29 05:51:26 3483832320 jenkins_backup20201029.tar
2020-10-29 16:06:11 3489792000 jenkins_backup20201030.tar
2020-10-30 16:06:10 3493355520 jenkins_backup20201031.tar
2020-10-31 16:06:10 3496591360 jenkins_backup20201101.tar
```

* tar 압축파일 다운로드
``` shell
$ aws s3 cp s3://productsystem-batch/jenkins_backup20201101.tar .
download: s3://productsystem-batch/jenkins_backup20201101.tar to ./jenkins_backup20201101.tar
```

#### 서비스 복구
``` shell
# jenkins home 폴더로 이동
$ cd /var/lib/jenkins
# jenkins home 폴더 내용 지우기
$ sudo rm -rf ./*
# 압축파일 옮기기
$ sudo cp ~/jenkins_backup20201101.tar ./
# 압축파일 풀기
$ sudo tar -xvf jenkins_backup20201101.tar
# 풀린 압축파일 내용 jenkins home 폴더로 이동
$ sudo mv -f ./var/lib/jenkins/* ./
# 압축 파일 지우기
$ sudo rm jenkins_backup20201101.tar
$ sudo rm -rf ./var*
# 파일/폴더 소유자 변경
$ sudo chown -R jenkins:jenkins /var/lib/jenkins
# jenkins 재시작
$ sudo service jenkins restart
```

### 오류 대응
#### JDK 를 찾을 수 없어 오류 발생
* 오류 Stacktrace
``` java
Exception in thread "main" java.lang.UnsupportedClassVersionError: BatchApplication has been compiled by a more recent version of the Java Runtime (class file version 55.0), this version of the Java Runtime only recognizes class file versions up to 52.0
    at java.lang.ClassLoader.defineClass1(Native Method)
    at java.lang.ClassLoader.defineClass(ClassLoader.java:756)
    at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
    at java.net.URLClassLoader.defineClass(URLClassLoader.java:468)
    at java.net.URLClassLoader.access$100(URLClassLoader.java:74)
    at java.net.URLClassLoader$1.run(URLClassLoader.java:369)
    at java.net.URLClassLoader$1.run(URLClassLoader.java:363)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.net.URLClassLoader.findClass(URLClassLoader.java:362)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:418)
    at org.springframework.boot.loader.LaunchedURLClassLoader.loadClass(LaunchedURLClassLoader.java:92)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
    at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:46)
    at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
    at org.springframework.boot.loader.Launcher.launch(Launcher.java:51)
    at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:52)
Build step 'Execute shell' marked build as failure
```
    - 백업된 Jenkins 와 복원된 Jenkins EC2 의 JDK 경로가 다를 수 있습니다.
    - 해결방법은 `Jenkins 관리 -> Global Tool Configuration` 으로 이동하면 JDK 경로가 설정이 되어 있습니다. 
    - 이 경로를 JDK 가 설치된 경로로 설정하면 됩니다.
![!JDK configuration](/assets/img/posts/jenkins/jenkins_6.png)