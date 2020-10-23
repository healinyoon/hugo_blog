---
title: "Kuberenets 특정 버전으로 설치하기"
date: 2020-10-09T12:48:29+09:00
draft: false
categories: [
    "kubernetes",
]
tags: [
    "msa",
    "k8s",
]
---

# 배경
회사에서 Kubernetes를 운영하다보면 클러스터에 새로운 worker node를 추가해야하는 일이 종종 생깁니다.  
문제는 `apt-get` 등 기본 패키지 관리 도구를 사용하여 생각 없이 설치하면 기존에 운영하던 k8s 클러스터 버전과 맞지 않은 최신 버전이 설치 된다는 것입니다...(~그러면 저처럼 작업을 2번 하게 됩니다 하하~)

그런데 바이너리 파일을 사용해서 설치하기는 또 귀찮고..  
따라서 `apt`를 사용하되, 버전을 옵션으로 주는 방식으로 설치를 하기로 했습니다.  

나중에 또 2번 작업하지 않기 위해서, 그리고 중간에 발생한 이슈도 기록해둘겸 내용을 정리하였습니다.

# 설치하기
사실 기존 k8s 클러스터 설치 프로세스와 다른 점은 거의 없습니다.
기존의 프로세스는 [여기]()를 참고 바랍니다.

특정 버전 설치 옵션을 주는 부분만 신경써서 진행하면 됩니다.
```
# cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

// 패키지가 자동으로 업그레이드 되지 않도록 설정
sudo apt-mark hold kubelet kubeadm kubectl
```

### 원래는 아래와 같이 그냥 최신 버전을 설치했다면 이번엔 옵션으로 버전을 줘야 합니다.
```
sudo apt-get install -y kubelet kubeadm kubectl
```

### 운영 중인 클러스터의 버전 확인하자
먼저 기존에 운영 중인 k8s 클러스터의 버전을 확인합니다.

```
root@hci-k8s-master-01:~# kubectl get nodes
NAME                STATUS     ROLES    AGE   VERSION
k8s-worker-01       Ready      <none>   74d   v1.18.8
k8s-worker-02       Ready      <none>   74d   v1.18.8
k8s-master-01       Ready      master   76d   v1.18.6
```

클러스터의 node들은 1.18.x 버전을 사용하는 것을 알 수 있습니다.  
버전을 확인했으니 이제 설치를 진행합니다.

### 설치 가능한 버전 확인하기
저는 `sudo apt-get install -y kubelet=1.18.8` 이렇게 옵션을 주고 설치하려고 했는데, 애석하게도 실패했습니다.  
아래와 같은 로그가 발생합니다.

```
# sudo apt-get install kubelet=1.18.8
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Version '1.18.8' for 'kubelet' was not found
```

정확한 버전 옵션을 확인해보도록 합니다.
```
# apt-cache madison kubeadm
```

출력 결과에 제가 사용하려던 1.18.8 버전은 다음과 같이 나와있습니다.
```
kubeadm |  1.18.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
```

### 1.18.8-00 버전으로 설치하기
이제 다시 설치를 진행해봅니다.

* kubeadm 설치
원래 kubeadm만 설치해도 kubectl과 kubelet이 의존적으로 설치됩니다.  
그런데 말입니다... 특이점이 발생합니다.

```
# sudo apt-get install kubeadm=1.18.8-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  kubectl
The following NEW packages will be installed:
  kubeadm kubectl
0 upgraded, 2 newly installed, 0 to remove and 5 not upgraded.
Need to get 0 B/16.5 MB of archives.
After this operation, 82.8 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Selecting previously unselected package kubectl.
(Reading database ... 128167 files and directories currently installed.)
Preparing to unpack .../kubectl_1.19.2-00_amd64.deb ...
Unpacking kubectl (1.19.2-00) ...
Selecting previously unselected package kubeadm.
Preparing to unpack .../kubeadm_1.18.8-00_amd64.deb ...
Unpacking kubeadm (1.18.8-00) ...
Setting up kubectl (1.19.2-00) ...
Setting up kubeadm (1.18.8-00) ...
```

위와 같이 kubectl이 1.19.2-00 버전으로 설치가 됩니다.
찜찜하니까 downgrade 해줍니다.
```
# apt-get install kubectl=1.18.8-00
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be DOWNGRADED:
  kubectl
0 upgraded, 0 newly installed, 1 downgraded, 0 to remove and 6 not upgraded.
Need to get 8,827 kB of archives.
After this operation, 1,036 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubectl amd64 1.18.8-00 [8,827 kB]
Fetched 8,827 kB in 9s (1,024 kB/s)
dpkg: warning: downgrading kubectl from 1.19.2-00 to 1.18.8-00
(Reading database ... 128169 files and directories currently installed.)
Preparing to unpack .../kubectl_1.18.8-00_amd64.deb ...
Unpacking kubectl (1.18.8-00) over (1.19.2-00) ...
Setting up kubectl (1.18.8-00) ...
```

# Master에 join하기
[참고](https://stackoverflow.com/questions/51126164/how-do-i-find-the-join-command-for-kubeadm-on-the-master)와 동일하게 진행한다.

-끝-