---
title: "Jenkins Node Heap Memory 사이즈 설정하기(-Xmx/-Xms 옵션)"
date: 2020-08-31T13:26:55+09:00
draft: false
categories: [
    "jenkins",
]
tags: [
    "ci/cd",
    "jvm",
]
---

# 목적
Jenkins node heap memory 사이즈 변경 방법 정리

# 참고 자료
* [Java Memory Management for Java Virtual Machine (JVM)](https://betsol.com/java-memory-management-for-java-virtual-machine-jvm/)
* [Java Platform, Standard Edition HotSpot Virtual Machine Garbage Collection Tuning Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/ergonomics.html)
* [CloudBees Jenkins JVM troubleshooting](https://docs.cloudbees.com/docs/admin-resources/latest/jvm-troubleshooting/#recommended-options)
* [Jenkins 권장 사양](https://docs.cloudbees.com/docs/admin-resources/latest/jvm-troubleshooting/#suggested-specifications)


# 내용
Jenkins에서는 다양한 JAVA 옵션을 인수로 받을 수 있음.
Jenkins node heap memory 사이즈를 변경하기 위한 인수는 `-Xmx`와 `-Xms`가 있음
```
-Xms<size>        set initial Java heap size
-Xmx<size>        set maximum Java heap size
```

### 예시
#### master의 /etc/default/jenkins에서 설정
```
AVA_ARGS="-Xmx256m"     # default value
JAVA_ARGS="-Xmx2048m"   # 2048MB size
```

#### Jenkins UI에서 slave 설정
[Jenkins 관리] > [노드 관리] > '노드 선택 후 ' [설정] > [고급]
![heap](/images/20200831_jenkins_node_heap_size/args.png)

### [Jenkins 권장 사양](https://docs.cloudbees.com/docs/admin-resources/latest/jvm-troubleshooting/#suggested-specifications)
![heap](/images/20200831_jenkins_node_heap_size/heap.png)

### 추가
1) It is recommended to define the same value for both -Xms and -Xmx so that the memory is allocated on startup rather than runtime.
2) 대/소문자는 상관 없음(예: `-Xmx10G`는 `-Xmx10g`와 동일함)
3) Java processes 전역에 적용하고 싶으면 `JAVA_TOOL_OPTIONS` 환경 변수 사용(예: `export JAVA_TOOL_OPTIONS="-Xmx6g"`)