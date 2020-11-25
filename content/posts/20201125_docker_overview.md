---
title: "Docker 개요"
date: 2020-11-25T13:06:23+09:00
draft: false
categories: [  
  "msa",
]
tags: [
  "docker",
]
---

# The Docker platform
Docker는 어플리케이션을 패키징하고 실행할 수 있는 **Container라는 격리된 환경**을 제공한다. Container는 hypervisor의 추가 로드가 필요하지 않기 때문에 경량이지만 호스트 머신의 커널 내에서 직접 실행된다. 

# Docker architecture
Docker는 client-server 아키텍처를 사용한다. Docker client는 Docker daemon과 통신한다. Docker daemon은 cotainer를 빌드, 실행, 배포하는 작업을 수행한다. Docker client와 daemon은 동일한 시스템에서 실행되거나, 원격으로 연결할 수도 있습니다. Docker client와 daemon은 UNIX 소켓 또는 네트워크 인터페이스를 통해 REST API로 통신한다.

![](/images/20201125_docker_overview/architecture.png)

### Docker daemon
Docker daemon(=`dockerd`)은 Docker API 요청을 수신하고 image, container, network, volume과 같은 Docker 객체를 관리한다. Docker daemon의 상세 내용은 [여기](https://velog.io/@labyu/docker-3)를 참고하자.

### Docker client
Docker client(=`docker`)는 사용자가 입력한 명령어를 Docker daemon에 전달한다. 이때의 동작 흐름은 다음과 같다.
1. 사용자가 docker 명령어 입력
2. Docker client는 Docker deamon에게 명령어 전달
3. Docker daemon은 명령어를 파싱하고 해당하는 작업 수행
4. Docker daemon은 수행 결과를 Docker client에게 반환
5. Docker client는 사용자에게 결과를 출력

### Docker registries
Docker registry는 Docker image를 저장한다. Docker Hub는 누구나 사용 가능한 public registry로, Docker는 기본적으로 Docker Hub에서 image를 찾도록 구성된다. private registry 구축도 가능하다.

`docker pull` 또는 `docker run` 명령어를 사용하면 registry에서 image를 가져온다. `docker push` 명령을 사용하면 image가 구성된 registry로 push 된다.

### Docker objects
Docker를 사용하면 image, container, network, volume, plugin 및 기타 object를 생성하고 사용하게 된다.

#### Images
Docker container를 만들기 위한 템플릿이다. registry에 존재하는 image를 가져와 사용할 수도 있고, image를 만들고 실행하는데 필요한 단계를 정의하는 `Dockerfile`를 작성하여 고유한 image를 빌드할 수도 있다.

#### Containers
Docker container는 image의 실행 가능한 인스턴스이다.

# Container technology

# 참고
* https://docs.docker.com/get-started/overview/
* https://velog.io/@labyu/docker-3