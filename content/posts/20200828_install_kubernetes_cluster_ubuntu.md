---
title: "Kubernetes Cluster 설치하기(ubuntu18.04)"
date: 2020-09-02T17:11:38+09:00
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

이 페이지에서는 ubuntu18.04에 `kubeadm` tool을 설치하고 이를 사용하여 kubernetes cluster를 구축하는 방법을 정리했습니다.

# 구성

### H/W

구성하려는 kubernetes cluster는 다음과 같습니다.

| 노드 | vCPU | RAM | Disk |
| --- | --- | --- | --- |
| master01 | 2 | 8GiB |  |
| worker01 | 2 | 8GiB |  |
| worker02 | 2 | 8GiB |  |

### Required ports

**1) Control-plane node(s)**  

![](/images/20200828_install_kubernetes_cluster/1.png)

**2) Worker node(s)**  

![](/images/20200828_install_kubernetes_cluster/2.png)


# Docker 설치(모든 node)

```
# curl -fsSL https://get.docker.com/ | sudo sh
# systemctl start docker
# systemctl enable docker
```

# Kubernetes  클러스터 구성

[쿠버네티스 공식 사이트 kubeam 설치](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

모든 노드에 아래의 패키지를 설치한다.
* kubeadm: 클러스터를 부트스트랩하는 명령(쿠버네티스 관리)
* kubelet: 클러스터의 모든 시스템에서 실행되는 구성 요소로, 포트 및 컨테이너 시작과 같은 작업을 수행(쿠버네티스 서비스)
* kubectl: 클러스터와 통신하기 위한 command line util(쿠버네티스 클라이언트 프로그램, 클러스터 구성과는 전혀 상관 없음)

### 1) Kubernetes 리포지토리 구성(모든 node)

```
# sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

### 2) Kubeadm, Kubelet, Kubectl 설치(모든 node)

```
# cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

// 패키지가 자동으로 업그레이드 되지 않도록 설정
sudo apt-mark hold kubelet kubeadm kubectl

// 데몬 재시작
systemctl daemon-reload
systemctl restart kubelet
```

### 3) hostname 등록

```
# sudo hostnamectl set-hostname master01

또는

# sudo hostnamectl set-hostname worker01
```

### 4) /etc/hosts 파일 수정

```
# vi /etc/hosts

아래에 추가
{IP} master01
{IP} worker01
{IP} worker02
```

### 5) Iptables 설정
브릿지 되어있는 IPv4 트래픽을 iptables 체인으로 전달될 수 있도록 한다.
```
# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# sudo sysctl --system
```

### 6) 스왑 기능 비활성화

```
swap 끄기
# sudo swapoff -a

재부팅 후에도 swap 설정 유지 
# sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### 7) 마스터 노드 초기화

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
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
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

### 8) 네트워크 애플리케이션 설치

pod 네트워크 애플리케이션을 설치해야 클러스터 내의 node간 통신이 가능하다.  
사용 가능한 옵션은 [여기](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)에서 확인할 수 있다.  
다음 명령을 master node에서 수행하여 weave pod 네트워크 애플리케이션을 설치한다.
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

이런 오류 발생시
The connection to the server localhost:8080 was refused - did you specify the right host or port?

export KUBECONFIG=/etc/kubernetes/admin.conf
을 적용해보자
```

master node에서 `kubectl get nodes` 명령어를 잠시 후 다시 입력하면 다음과 같이 STATUS가 **NotReady** => **Ready**로 변경된 것을 확인할 수 있다.
```
# kubectl get nodes
NAME                  STATUS   ROLES    AGE     VERSION
healin-k8s-master01   Ready    master   17m     v1.19.0
healin-k8s-worker01   Ready    <none>   7m42s   v1.19.0
healin-k8s-worker02   Ready    <none>   7m41s   v1.19.0
```

### 9) master node를 worker node로도 사용하고 싶다면,
쿠버네티스 클러스터의  control-plane 노드는 보안상의 이유로 격리되어 있다(기본값). 
master node에서는 pod 가 스케줄링 되지 않으므로, 1대의 머신으로만 쿠버네티스 클러스터를 구축할 경우 격리 해제해야 한다.

```
$ kubectl taint nodes –all node-role.kubernetes.io/master-
```

# 참고
[쿠버네티스(kubernetes) 설치 및 환경 구성하기](https://medium.com/finda-tech/overview-8d169b2a54ff)