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
Docker container는 image의 실행 가능한 인스턴스이다. Docker API 또는 CLI를 사용하여 container를 생성, 시작, 중지, 이동 또는 삭제 할 수 있다. container는 image와 사용자가 생성하거나 시작할 때 제공하는 구성 옵션에 의해 정의된다. container가 제거되면 영구 저장소에 저장되지 않은 데이터의 변경 사항들은 함께 제거된다. 

> 예제 `docker run` 명령
다음 명령어는 `ubuntu` container를 실행하고 container 내부와 대화형 command 창을 연결한다.
```
$ docker run -i -t ubuntu /bin/bash
```
위의 명령어를 입력하면 다음 절차가 순차적으로 진행된다.
1. `ubuntu` image가 local에 없는 경우, Docker는 자동으로 `docker pull ubuntu` 명령을 수행하여 registry에서 image를 가져온다.
2. Docker는 `docker container create` 명령을 수행하여 새로운 container를 생성한다.
3. Docker는 container layer(Read/Write 가능)를 최종 layer로 container에 할당한다. 이를 통해 실행 중인 container는 container 내부의 로컬 파일 시스템에서 파일과 디렉토리를 읽고 쓰는 것이 가능하다.
4. 네트워크 옵션을 지정하지 않았으므로, Docker는 container를 기본 네트워크에 연결하는 인터페이스를 만든다. 여기에는 container에 IP 주소를 할당하는 것도 포함된다.
5. Docker는 container를 시작하고 `/bin/bash` 명령을 실행시킨다. container가 실행중이며 `-i`와 `-t` 옵션을 사용하였기 때문에 container 내부에서 명령 입력이 가능하다.
6. `exit` 명령을 사용하여 container를 빠져나오면 container가 중지되지만 제거되지는 않는다. 다시 시작하거나 제거할 수 있다.

# Container technology
Docker는 Go 프로그래밍 언어로 작성되었으며, Linux 커널의 여러 기능을 활용하도록 제공한다.

### Namespace
Linux는 격리된 작업 공간을 제공하기 위해 `namespace` 라는 기능을 커널에 내장하고 있다. 현재 Docker Engine은 Linux에서 다음과 같은 namespace를 사용한다.
1. mnt(MNT; 파일 시스템 마운트): host 파일 시스템에 구애받지 않고 독립적으로 파일 시스템을 mount or unmount 가능하다. 
2. pid(PID; 프로세스 ID): 독립적인 프로세스 공간 할당
3. net(NET; 네트워킹): namespace간의 네트워크 충돌 방지(중복 port 바인딩 등)
4. ipc(IPC; 프로세스간 통신): 프로세스간의 독립적인 통신 통로 할당
5. uts(UTS; Unix Timesharing System): 독립적인 hostname 할당

#### PID namespace 밖의 공간(regular namespace)에서도 독립적인 프로세스 확인 가능
즉 namespace 기능은 같은 공간을 공유하되, **좀 더 제한된 공간**을 할당하는 원리이다. namespace를 통해 독립적인 공간을 할당한 후에는 `nsenter`(namespace enter)라는 명령어를 통해 이미 실행 중인 프로세스의 namespace 공간에 접근할 수 있다.

`nsenter`는 docker의 `exec`와 비슷한 역할을 한다. 다만 `nsenter`는 `exce`와 다르게 cgroups에 들어가지 않기 때문에 리소스 제한의 영향을 받지 않는다.


### Cgroups
자원(resource)에 대한 제어를 가능하게 해주는 Linux 커널 기능이다. Croups은 다음 리소스를 제어할 수 있다.
* 메모리
* CPU
* I/O
* 네트워크
* device 노드(/dev/)

# 참고
* https://docs.docker.com/get-started/overview/
* https://velog.io/@labyu/docker-3
* https://tech.ssut.me/what-even-is-a-container/