---
title: "Kuberenets 특정 버전으로 설치하기"
date: 2020-10-09T12:48:29+09:00
draft: false
categories: [
  "msa",
]
tags: [
  "kubernetes",
]
---

# 배경
회사에서 Kubernetes를 운영하다보면 클러스터에 새로운 worker node를 추가해야하는 일이 종종 생깁니다.  
문제는 `apt-get` 등 기본 패키지 관리 도구를 사용하여 생각 없이 설치하면 기존에 운영하던 k8s 클러스터 버전과 맞지 않은 최신 버전이 설치 된다는 것입니다...(그러면 저처럼 작업을 2번 하게 됩니다)

그런데 바이너리 파일을 사용해서 설치하기는 또 귀찮고..  
따라서 `apt`를 사용하되, 버전을 옵션으로 주는 방식으로 설치를 하기로 했습니다.  

나중에 또 2번 작업하지 않기 위해서, 그리고 중간에 발생한 이슈도 기록해둘겸 내용을 정리하였습니다.

# 설치하기
사실 기존 k8s 클러스터 설치 프로세스와 다른 점은 거의 없습니다.
기존의 프로세스는 [여기](https://healinyoon.github.io/2020/09/20200828_install_kubernetes_cluster_ubuntu/)를 참고 바랍니다.

특정 버전 설치 옵션을 주는 부분만 신경써서 진행하면 됩니다.

### 저장소 추가
```
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

# sudo apt-get update
```

### 원래는 아래와 같이 그냥 최신 버전을 설치했다면 이번엔 옵션으로 버전을 줘야 합니다.
```
sudo apt-get install -y kubelet kubeadm kubectl
```

### 운영 중인 클러스터의 버전 확인하기
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
이런 경우, kubectl을 downgrade 해주는 방법도 있지만..
```
# apt-get install kubectl=1.18.8-00
```

애초에 처음부터 모두 버전을 지정해주면 됩니다.
```
# sudo apt-get install -y kubelet=1.18.8-00 kubeadm=1.18.8-00 kubectl=1.18.8-00
```

### 설치된 버전 확인 하기
마지막으로 설치된 버전을 확인해보겠습니다.
```
#  dpkg -l kube*
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                      Version           Architecture      Description
+++-=========================-=================-=================-========================================================
ii  kubeadm                   1.18.8-00         amd64             Kubernetes Cluster Bootstrapping Tool
ii  kubectl                   1.18.8-00         amd64             Kubernetes Command Line Tool
ii  kubelet                   1.18.8-00         amd64             Kubernetes Node Agent
ii  kubernetes-cni            0.8.7-00          amd64             Kubernetes CNI
```

# Master에 join하기
[참고](https://stackoverflow.com/questions/51126164/how-do-i-find-the-join-command-for-kubeadm-on-the-master)와 동일하게 진행합니다.

### master node에서 join 명령어 발급받기
```
# kubeadm token create --print-join-command
```

### worker node에서 join 명령어 수행하기
신규로 join하려는 worker node에서 위에서 발급받은 명령어를 수행합니다.
```
# kubeadm join X.X.X.X:XX --token xxxx --discovery-token-ca-cert-hash sha256:xxxx
W1113 13:04:14.543859   83613 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

### master node에서 cluster로 join된 것 확인하기
`STATUS`가 NotReady -> Ready로 변경되는데 시간이 걸릴 수 있습니다.
```
# kubectl get nodes
```