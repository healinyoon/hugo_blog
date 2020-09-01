---
title: "Kubernetes Cluster 설치하기"
date: 2020-08-28T17:11:38+09:00
draft: false
categories: [
    "docker",
    "kubernetes",
]
tags: [
    "msa",
    "k8s",
]
---

# Kubernetes Cluster 설치하기

이 페이지에서는 `kubeadm` tool을 설치하고 이를 사용하여 kubernetes cluster를 구축하는 방법을 정리했습니다.

# 구성

### H/W

구성하려는 kubernetes cluster는 다음과 같습니다.

| 노드 | vCPU | RAM | Disk |
| --- | --- | --- | --- |
| Master | 2 | 8GiB |  |
| Worker01 | 2 | 8GiB |  |
| Worker02 | 2 | 8GiB |  |

### Required ports

**1) Control-plane node(s)**
![](/images/20200828_install_kubernetes_cluster/1.png)

**2) Worker node(s)**
![](/images/20200828_install_kubernetes_cluster/2.png)


# Docker 설치

아래의 링크를 참고하여 Docker를 설치합니다.
[Docker 설치하기](https://healinyoon.github.io/2019/06/20190611_docker_install/)

# Kubernetes  클러스터 구성

* [쿠버네티스 공식 사이트 kubeam 설치](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

모든 노드에 아래의 패키지를 설치한다.
* kubeadm: 클러스터를 부트스트랩하는 명령(쿠버네티스 관리)
* kubelet: 클러스터의 모든 시스템에서 실행되는 구성 요소로, 포트 및 컨테이너 시작과 같은 작업을 수행(쿠버네티스 서비스)
* kubectl: 클러스터와 통신하기 위한 command line util(쿠버네티스 클라이언트 프로그램, 클러스터 구성과는 전혀 상관 없음)

### 1) Kubernetes 리포지토리 구성

```
# cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

### 2) Kubeadm, Kubelet, Kubectl 설치

```
# sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
# sudo systemctl enable --now kubelet
# systemctl start kubelet
```

패키지가 자동으로 업그레이드 되지 않도록 설정
```
# yum install yum-plugin-versionlock
# sudo yum versionlock kubelet kubeadm kubectl
Adding versionlock on: 0:kubelet-1.19.0-0
Adding versionlock on: 0:kubeadm-1.19.0-0
Adding versionlock on: 0:kubectl-1.19.0-0
versionlock added: 3
```

### 3) hostname 등록

```
sudo hostnamectl set-hostname master01

또는

sudo hostnamectl set-hostname worker01
```

### 4) /etc/hosts 파일 수정

```
아래에 추가
{IP} master01
{IP} worker01
{IP} worker02
```

### 5) SELinux 끄기

```
# sudo setenforce 0
# sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### 6) Iptables 설정

```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 7) 스왑 기능 비활성화

```
swap 끄기
# sudo swapoff -a

재부팅 후에도 swap 설정 유지 
# sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### 8) 마스터 노드 초기화

```
# kubeadm init
```

`kubeadm init` 명령어 실행시 아래와 같이 출력된다.
```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.1.11.4:6443 --token xxxxxxxxxxxxxxxxxxxxxxx \
    --discovery-token-ca-cert-hash sha256:e184c470296359bc4a35bc57624b03d8c4b3eb2bd46f413f3a68a86f182c9844
```

master node에서 아래 명령어를 수행하고
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

worker node에서 아래의 명령어를 수행하여 master node와 join한다.
```
kubeadm join 10.1.11.4:6443 --token xxxxxxxxxxxxxxxxxxxxxxx \
    --discovery-token-ca-cert-hash sha256:e184c470296359bc4a35bc57624b03d8c4b3eb2bd46f413f3a68a86f182c9844
```

master node에서 `kubectl get nodes` 명령어를 입력하면 다음과 같이 출력된다.
```
# kubectl get nodes
NAME                  STATUS     ROLES    AGE   VERSION
healin-k8s-master01   NotReady   master   10m   v1.19.0
healin-k8s-worker01   NotReady   <none>   30s   v1.19.0
healin-k8s-worker02   NotReady   <none>   29s   v1.19.0
```

### 9) 네트워크 애플리케이션 설치

pod 네트워크 애플리케이션을 설치해야 클러스터 내의 node간 통신이 가능하다.  
사용 가능한 옵션은 [여기](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)에서 확인할 수 있다.  
다음 명령을 master node에서 수행하여 flannel pod 네트워크 애플리케이션을 설치한다.
```
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

master node에서 `kubectl get nodes` 명령어를 잠시 후 다시 입력하면 다음과 같이 STATUS가 **NotReady** => **Ready**로 변경된 것을 확인할 수 있다.
```
# kubectl get nodes
NAME                  STATUS   ROLES    AGE     VERSION
healin-k8s-master01   Ready    master   17m     v1.19.0
healin-k8s-worker01   Ready    <none>   7m42s   v1.19.0
healin-k8s-worker02   Ready    <none>   7m41s   v1.19.0
```

# 참고
[쿠버네티스(kubernetes) 설치 및 환경 구성하기](https://medium.com/finda-tech/overview-8d169b2a54ff)