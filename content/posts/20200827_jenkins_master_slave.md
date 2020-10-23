---
title: "Jenkins Master-Slave 구성하기"
date: 2020-08-27T19:35:29+09:00
draft: false
categories: [
  "devops",
  "ci/cd",
]
tags: [
  "jenkins",
]
---

# Jenkins Master-Slave
Jenkins는 기본적으로 단일 서버로 동작합니다. 그러나 단일 서버는 다음과 같은 상황을 충족하기 충분하지 않습니다.
* 빌드 테스트를 위한 다양한 환경이 필요한 경우
* 크고 무거운 프로젝트의 작업 부하를 분산해야하는 경우

위의 요구사항을 충족하기 위해 Jenknis 분산 아키텍처인 Master-Slave 구성이 도입되었습니다.
![](/images/20200827_jenkins_master_slave/2-14.png)

# Jenkins Master와 Slave의 역할

| 구분 | 역할 |
| --- | --- |
| Master | * Build 작업 예약<br>* Build 실행을 위한 작업 분배<br>* Slave node 모니터링(필요에 따라 on/offline 전환 가능)<br>* Build 결과 기록 |
| Slave | * Jenkins Master의 요청 수신<br>* Build 실행|

# Jenkins Master-Slave 연동 방법

Jenkins Master-Slave는 다음의 요구사항을 만족시키면 매우 간단하게 구성할 수 있습니다.
* Master 서버에서 Slave 서버에 접근 가능하도록 설정
* Slave 서버의 [Jenkins Java 요구사항](https://www.jenkins.io/doc/administration/requirements/java/) 충족

### 1. Jenkins 사용자 생성(모든 Slave node에서 진행)

```
# adduser jenkins

사용자 생성 후 다음과 같이 홈디렉토리의 권한을 변경해준다
# chown ldccai:jenkins /home/jenkins
# chmod 775 /home/jenkins
```

### 2. Java 8 설치(모든 Slave node에서 진행)

```
# apt-get install openjdk-8-jdk
```

### 3. Jenkins에서 Slave node 등록

![](/images/20200827_jenkins_master_slave/2-9.png)  
![](/images/20200827_jenkins_master_slave/2-10.png)  
![](/images/20200827_jenkins_master_slave/2-11.png)  
![](/20200827_jenkins_master_slave/images/2-12.png)  

등록된 노드의 [로그]를 확인하면 다음과 같이 Master 서버와 잘 연동된 것을 확인할 수 있습니다.
![](/images/20200827_jenkins_master_slave/2-13.png)
