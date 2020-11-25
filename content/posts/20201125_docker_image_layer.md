---
title: "Docker Image의 핵심 기술: Layer"
date: 2020-11-25T15:12:04+09:00
draft: false
categories: [  
  "msa",
]
tags: [
  "docker",
]
---

# Docker Image의 핵심 기술: Layer

Docker container를 만들기 위한 템플릿이다. registry에 존재하는 image를 가져와 사용할 수도 있고, image를 만들고 실행하는데 필요한 단계를 정의하는 `Dockerfile`를 작성하여 고유한 image를 빌드할 수도 있다. Dockerfile의 각 단계에 정의된 명령어는 image에 layer를 만든다.

### Image Layer
앞서 설명한 바와 같이 image는 Dockerfile로 빌드된다. Dockerfile은 다음과 같이 단계별로 구성된다.
```
// step1
FROM alpine:3.10

// step2
ENTRYPOINT ["echo", "hello"]
```

Dockerfile을 빌드하면 각 단계별로 image가 생성되는 것을 확인할 수 있다. 즉 Dockerfile을 빌드하면 Dockerfile의 단계별로 image layer가 생성되며, 이들이 계층적으로 하나씩 쌓이며 image를 이루게 된다.
```
$ sudo docker build --tag echo:1.0 .
Sending build context to Docker daemon  7.168kB
FROM alpine:3.10
Step 1/2 : FROM alpine:3.10
3.10: Pulling from library/alpine
21c83c524219: Already exists
Digest: sha256:f0e9534a598e501320957059cb2a23774b4d4072e37c7b2cf7e95b241f019e35
Status: Downloaded newer image for alpine:3.10
 ---> be4e4bea2c2e
Step 2/2 : ENTRYPOINT ["echo", "hello"]`
 ---> Running in 2dbb42b167e8
FROM ubuntu:18:04
Removing intermediate container 2dbb42b167e8
 ---> 75b05f96c44d
Successfully built 75b05f96c44d
Successfully tagged echo:1.0
```
![](/images/20201125_docker_image_layer/img-layer.png)

### Container Layer
Image를 빌드한 후 `docker container run` 명령을 수행하면 아래와 같이 가장 마지막에 **container layer**를 생성한다(이 container layer는 container를 삭제하면 같이 삭제 된다).

![](/images/20201125_docker_image_layer/container-layer.png)

그런데 실제로 container를 사용할 때는 하나의 파일 시스템으로 보이는데, 이렇게 계층적으로 나눠진 image들이 어떻게 하나의 파일 시스템으로 보이는 것일까? 바로 Union FS 덕분이다.

### Union FS

### Union Mount
Union Mount는 복수의 파일 시스템을 하나의 파일 시스템으로 마운트하는 기능으로 두 파일 시스템에서 동일한 파일이 있다면 나중에 마운트 된 파일 시스템의 파일을 Overlay한다. 하위 파일에 대한 쓰기 작업은 CoW(Copy on Write) 전략에  따라 복사본을 생성하여 수행하므로 원본 파일 시스템은 변하지 않는 것이 특징이다.

![](/images/20201125_docker_image_layer/unionfs.png)


### Docker Image: Union File System 
Docker image는 Union File System 기반으로 동작한다. Union File Systme의 특성에 따라 하위 layer는 읽기 전용이며, CoW 전략에 의해 쓰기 작업은 상위 레이어로 복사해서 이루어지기 때문에 하나의 image로 부터 복수의 container가 실행 가능한 것이다.

* Container Layer
    - Write layer
    - 각 container마다 최 상단 layer에 생성되어, container마다 자신만의 상태를 가질 수 있게 해준다.
    - container가 생성된 후 모든 변경 작업은 여기서 이루어진다.
    - R/W 속도가 느리다.

* Image Layer
    - Read only layer
    - 다른 container와 공유 가능하다.

![](/images/20201125_docker_image_layer/layers.jpeg)

# 참고
* https://docs.docker.com/get-started/overview/
* https://nirsa.tistory.com/63
* https://devaom.tistory.com/5