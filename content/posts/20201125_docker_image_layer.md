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

Docker image는 container를 만들기 위한 템플릿이다. registry에 존재하는 image를 가져와 사용할 수도 있고, image를 만들고 실행하는데 필요한 단계를 정의하는 `Dockerfile`를 작성하여 고유한 image를 빌드할 수도 있다. Dockerfile의 각 단계에 정의된 명령어는 image에 layer를 만든다. Docker image를 이해하기 위해서는 이 **Layer**에 대한 이해가 필요하다.

# Image Layer

앞서 설명한 바와 같이 image는 Dockerfile로 빌드된다. Dockerfile을 작성하고 빌드하는 과정을 통해 image layer에 대해 이해해보자. Dockerfile은 다음과 같이 단계별로 구성된다.
```
// step1
FROM alpine:3.10

// step2
ENTRYPOINT ["echo", "hello"]
```

Dockerfile을 빌드하면 아래와 같이 각 단계별로 image가 생성되는 것을 확인할 수 있다. 즉 Dockerfile을 빌드하면 Dockerfile의 단계별로 image layer가 생성되며, 이들이 계층적으로 하나씩 쌓이며 image를 이루게 된다.
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

# Container Layer
Image를 빌드한 후 `docker container run` 명령을 수행하면 아래와 같이 가장 마지막에 **container layer**를 생성한다(이 container layer는 container를 삭제하면 같이 삭제 된다).

![](/images/20201125_docker_image_layer/container-layer.png)

그런데 실제로 container를 사용할 때는 하나의 파일 시스템으로 보이는데, 이렇게 계층적으로 나눠진 image들이 어떻게 하나의 파일 시스템으로 보이는 것일까? 바로 Union FS 덕분이다.

# Union FS

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

# Image Layer 디렉토리 파악하기


### Image 정보 확인
nginx image를 다운받아보자.
* Image pull
```
# sudo docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
852e50cd189d: Already exists
571d7e852307: Pull complete
addb10abd9cb: Pull complete
d20aa7ccdb77: Pull complete
8b03f1e11359: Pull complete
Digest: sha256:6b1daa9462046581ac15be20277a7c75476283f969cb3a61c8725ec38d3b01c3
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

* Image 정보 확인
Image의 정보는 `docker inspect {image}` 명령어로 확인할 수 있다. 
```
# docker inspect nginx
[
    {
        "Id": "sha256:bc9a0695f5712dcaaa09a5adc415a3936ccba13fc2587dfd76b1b8aeea3f221c",
        "RepoTags": [
            "nginx:latest"
        ],

(중략)
```

### Image 저장소 위치 확인
docker image 저장소 위치는 `docker info` 명령어로 확인할 수 있다.
```
# docker info
Client:
 Debug Mode: false

Server:
 Containers: 57
  Running: 16
  Paused: 0
  Stopped: 41
 Images: 149
 Server Version: 19.03.13
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 8fba4e9a7d01810a393d5d25a3621dc101981175
 runc version: dc9208a3303feef5b3839f4323d9beb36df0a9dd
 init version: fec3683
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 5.4.0-1031-azure
 Operating System: Ubuntu 18.04.5 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 7.749GiB
 Name: master
 ID: 5HUW:6SVP:6Q4Q:LDGN:YMF2:RBDM:VEXF:3UKJ:XVTV:5SRK:SS7R:TPJ2
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

WARNING: No swap limit support
```

### Layer 디렉토리
* 디렉토리 구조
Image layer 정보를 확인하기 위한 디렉토리만 살펴보면 다음과 같다.
```
/var/lib/docker# tree docker
.
docker
├── containers: docker container 정보를 저장한다.
├── image: docker image 정보를 저장한다.
│   └── overlay2
│       ├── imagedb: imagedb에 대한 정보는 layerdb에 저장된다.
│       └── layerdb: layerdb에 대한 정보는 overlay2에 저장된다.
├── overlay2: docker image의 파일 시스템이 저장된다. 실질적으로 image layer 데이터가 저장되는 경로이다.
│   ├── 0090fbeed32cba3aed09c2459d4a5f59144be127dfed23cbe8c7f47982dd3c12
│   ├── 014997dffd51a7e9a1418e7f888097c360c592ddb791c0973b35b9935a1eea9d
``` 

* 각 디렉토리별 용량
위의 각 경로의 데이터 용량을 조회해보면 다음과 같다. 실제로 docker 데이터가 저장되는 root 경로인 `/var/lib/docker` 디렉토리 데이터 용량과 `/var/lib/docker/overlay2` 디렉토리 데이터 용량이 가장 근접한 것을 볼 수 있다(즉 실질적인 image layer 데이터가 여기에 저장된다는 것).
```
# du -sh /var/lib/docker
9.1G    /var/lib/docker

# du -sh /var/lib/docker/containers/
39M     /var/lib/docker/containers/

# du -sh /var/lib/docker/image/
11M     /var/lib/docker/image/

# du -sh /var/lib/docker/overlay2/
6.0G    /var/lib/docker/overlay2/
```

# 참고
* https://docs.docker.com/get-started/overview/
* https://nirsa.tistory.com/63
* https://devaom.tistory.com/5
