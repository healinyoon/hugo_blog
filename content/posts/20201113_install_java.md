---
title: "java(openjdk) 설치 및 환경 변수 설정"
date: 2020-11-13T14:16:05+09:00
draft: false
categories: [  
  "linux",
]
tags: [
  "java",
  "openjdk",
]
---

# jdk8 설치

### 버전 확인
```
$ java -version
```

### repository 업데이트
```
$ sudo apt-get update
```

### openjdk 설치
```
$ sudo apt-get install openjdk-8-jdk
```

### 설치 확인
```
$ java -version
openjdk version "1.8.0_275"
OpenJDK Runtime Environment (build 1.8.0_275-8u275-b01-0ubuntu1~18.04-b01)
OpenJDK 64-Bit Server VM (build 25.275-b01, mixed mode)
```

# 환경 변수 설정

### javac 설치 경로 확인
```
# javac -version
javac 1.8.0_275

# which javac
/usr/bin/javac

# readlink -f /usr/bin/javac
/usr/lib/jvm/java-8-openjdk-amd64/bin/javac
```

### 환경 변수 설정
```
# vi /etc/profile

맨 아래에 추가
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/bin/javac
```

### 적용
```
# source /etc/profile
```

### 확인
```
# echo $JAVA_HOME
/usr/lib/jvm/java-8-openjdk-amd64/bin/javac
```
