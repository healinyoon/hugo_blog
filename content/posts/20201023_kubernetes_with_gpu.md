---
title: "Kubernets(쿠버네티스) with GPU 구축하기"
date: 2020-10-23T10:56:19+09:00
draft: false
---

# Intro
Kubernetes에서 GPU를 사용하도록 환경을 구축하고 사용해보자.
여기서는 Kubernest 클러스터가 구축되어 있다고 전제하고 진행한다. 아직 설치가 되지 않았다면 [Kubernetes Cluster 설치하기(ubuntu18.04)](https://healinyoon.github.io/2020/09/20200828_install_kubernetes_cluster_ubuntu/)을 참고해서 설치 후 진행해야 한다.

# 1. Nvidia Plugin Pod 생성

### ref)

* [Nvidia k8s-device-plugin 공식 사이트](https://github.com/NVIDIA/k8s-device-plugin/tree/v1.12)
* [Nvidia docker 공식 사이트](https://github.com/NVIDIA/nvidia-docker)

## 1.1 가장 정보가 많았던 YAML으로 설치, 그러나 실패

처음에 [여기](https://likefree.tistory.com/15) 링크를 참고하여 진행했다.

다만 아래와 같이 버전만 변경하여 실행했다.
```
$ kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.12/nvidia-device-plugin.yml
daemonset.extensions/nvidia-device-plugin-
daemonset-1.12 created
```

하지만 아래 이슈 발생해서 실패했다 ↓ ↓ ↓

## 1.2. 이슈

### 1.2.1. 이슈 내용
쿠버네티스 `1.15` 버전 이하를 설치했을 경우 문제 없겠지만, `1.16` 버전 이상을 설치했을 경우 다음과 같은 에러가 발생한다.
```
error: unable to recognize "https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v1.12/nvidia-device-plugin.yml": no matches for kind "DaemonSet" in version "extensions/v1beta1"
```
이는 쿠버네티스 버전이 업그레이드 되면서, `Daemonset`의 `extensions/v1beta1` 버전을 더이상 지원하지 않기 때문이다.  
→ 1) 따라서 버전을 `apps/v1` 으로 변경하고 `selector` object를 추가한 후  
→ 2) k8s-device-plugin을 다시 설치하고  
→ 3) 매니패스트 파일도 적절하게 수정해주었다.  

### ref)

* [Kubectl convert 참고 자료](https://medium.com/star-systems-labs/kubectl-convert-update-api-versions-automatically-e669add17e3d)
* [No matches 이슈 해결 자료](ttps://www.kangwoo.kr/2020/02/17/pc%EC%97%90-kubeflow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-2%EB%B6%80-kubernetes-nvidia-device-plugin-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/)  

> (참고) Pod 생성 실패시 에러 정보 확인
```
$ kubectl describe pod {pod 명}
```

### 1.2.2. 변경한 YAML 파일을 사용하여 DaemonSet Pod 생성
이제 커스터 마이징한 YAML 파일로 Pod를 생성해보자.
> gpu-plugin.yaml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nvidia-device-plugin-daemonset-1.12
  namespace: kube-system
spec:
  updateStrategy:
    type: RollingUpdate
  selector:
    matchLabels:
      name: nvidia-device-plugin-ds
  template:
    metadata:
      # Mark this pod as a critical add-on; when enabled, the critical add-on scheduler
      # reserves resources for critical add-on pods so that they can be rescheduled after
      # a failure.  This annotation works in tandem with the toleration below.
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
      labels:
        name: nvidia-device-plugin-ds
    spec:
      tolerations:
      # Allow this pod to be rescheduled while the node is in "critical add-ons only" mode.
      # This, along with the annotation above marks this pod as a critical add-on.
      - key: CriticalAddonsOnly
        operator: Exists
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      containers:
      - image: nvidia/k8s-device-plugin:1.11
        name: nvidia-device-plugin-ctr
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: device-plugin
            mountPath: /var/lib/kubelet/device-plugins
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
      nodeSelector:
        gpus: "true"
```
위의 매니패스트 주요 사항은 다음과 같다.

##### ① 리소스 유형 = DaemonSet
```
kind: DaemonSet
```
    따라서 기본적으로는 모든 worker 노드 하나씩 동작하게 한다.

##### ② RollingUpdate
`spec.selector.matchLabels.name`

```
name: nvidia-device-plugin-ds
```

`spec.template.matadata.labels.name` 

```
labels:
  name: nvidia-device-plugin-ds
```

name object가 `nvidia-device-plugin-ds` 인 리소스에 대하여 RollingUpdate를 설정한다.

##### ③ node Label 지정

`spec.template.spec.nodeSelector` 로 어느 노드의 DaemonSet으로 띄워줄 것인지 레이블링해준다.

```
nodeSelector:
  gpus: "true"
```

## 1.3. gpu-plugin DeamonSet Pod 정상 동작 확인
```
$ kubectl -n kube-system get pod -l name=nvidia-device-plugin-ds
NAME                                        READY   STATUS    RESTARTS   AGE
nvidia-device-plugin-daemonset-1.12-d7t2g   1/1     Running   0          48d
nvidia-device-plugin-daemonset-1.12-jmcg9   1/1     Running   0          48d
nvidia-device-plugin-daemonset-1.12-zhsqm   1/1     Running   0          4h8mubectl -n kube-system get pod -l name=nvidia-device-plugin-ds
NAME                                        READY   STATUS    RESTARTS   AGE
nvidia-device-plugin-daemonset-1.12-7kh27   1/1     Running   0          27m

$ kubectl -n kube-system logs  -l name=nvidia-device-plugin-ds
2020/07/01 05:02:30 Loading NVML
2020/07/01 05:02:30 Fetching devices.
2020/07/01 05:02:30 Starting FS watcher.
2020/07/01 05:02:30 Starting OS watcher.
2020/07/01 05:02:30 Starting to serve on /var/lib/kubelet/device-plugins/nvidia.sock
2020/07/01 05:02:30 Registered device plugin with Kubelet
```

### 🌟🌟 여기서 잠깐! 🌟🌟
중요한 사항은 gpu를 사용하려는 Worker node가 `gpus: "true"` 레이블링이 되어 있어야 한다는 것이다. 

* 만약 GPU가 있는 node인데 해당 DeamonSet이 올라가있지 않거나
* 신규 Worker node를 추가하려는 경우

⇒ `kubectl label nodes {Worker node 명} gpus=true` 로 레이블링을 해주자.

# 2. GPU 개수 확인
이제 쿠버네티스가 사용 가능한 GPU 개수를 확인해보자.  
master node에서 아래의 명령어를 실행하면 각 worker node에서 사용 가능한 GPU 개수가 출력된다.
```
$ kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
NAME            GPU
gpu-1080ti-XX   9
gpu-1080ti-XX   9
```

# 3. Pod에서 그래픽 카드 명령어 테스트

## 3.1. GPU를 사용하는 Pod 생성하기

### 3.1.1. YAML 파일
> gpu-k8s.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: gpu-k8s
spec:
  containers:
  - name: gpu-container
    image: nvidia/cuda:9.0-runtime
    command:
      - "/bin/sh"
      - "-c"
    args:
      - nvidia-smi && tail -f /dev/null
    resources:
      requests:
        nvidia.com/gpu: 2
      limits:
        nvidia.com/gpu: 2
```

### ref)
nvidia/cuda 도커 이미지 버전이 맞지 않은 이슈 발생 시 ⇒ 도커 허브에서 맞는 이미지 버전을 찾아서 사용해주면 된다.

* [Docker Hub](https://hub.docker.com/r/nvidia/cuda/)
* [Kubernetes Resource Request와 Limit의 이해](https://itchain.wordpress.com/2018/05/16/kubernetes-resource-request-limit/)
* [Schedule GPUs](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/)

### 3.1.2. Pod 생성 및 확인
```
$ kubectl apply -f gpu-k8s.yaml
pod/gpu-k8s created

$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
gpu-k8s                             1/1     Running   0          12s
```

### 3.1.3. nvidia-smi 확인
```
$ kubectl logs gpu-k8s
Thu Jul  2 06:00:24 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.95.01    Driver Version: 440.95.01    CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  Off  | 00000000:86:00.0 Off |                  N/A |
| 29%   30C    P8     8W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce GTX 108...  Off  | 00000000:AF:00.0 Off |                  N/A |
| 29%   31C    P8     8W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

## 3.2. 2개의 Pod를 띄워서 gpu 4개를 모두 사용하기
gpu-k8s2.yaml 매니패스트 파일을 하나 더 만들어서 위와 동일하게 실행해보자.

### 3.2.1. 결과 확인

```
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
gpu-k8s                             1/1     Running   0          2m44s
gpu-k8s2                            1/1     Running   0          2
```

### 3.2.2. nvidia-smi 확인

```
$ kubectl logs gpu-k8s2
Thu Jul  2 06:03:06 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.95.01    Driver Version: 440.95.01    CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  Off  | 00000000:18:00.0 Off |                  N/A |
| 29%   31C    P8     7W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce GTX 108...  Off  | 00000000:3B:00.0 Off |                  N/A |
| 29%   33C    P8     8W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

## 3.3. 컨테이너가 노드의 모든 GPU를 사용 가능하게 하고 싶다면

* request와 limit 설정 부분을 없애주면 된다.
* 특이한 점은 이미 다른 파드에 GPU를 모두 할당 해준 상태에서도 파드 생성 가능하다.

### 3.3.1. YAML 파일
```
apiVersion: v1
kind: Pod
metadata:
  name: gpu-all
spec:
  containers:
  - name: gpu-container
    image: nvidia/cuda:9.0-runtime
    env:
    - name: DP_DISABLE_HEALTHCHECKS
      value: "xids"
    command:
      - "/bin/sh"
      - "-c"
    args:
      - nvidia-smi && tail -f /dev/null
```

### 3.3.2. Pod 실행
```
$ kubectl apply -f gpu-all.yaml
pod/gpu-all created
```

### 3.3.3. 확인
```
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
gpu-all                             1/1     Running   0          50s
gpu-k8s                             1/1     Running   0          42m
gpu-k8s2                            1/1     Running   0          40m

$ kubectl logs gpu-all
Thu Jul  2 06:42:25 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.95.01    Driver Version: 440.95.01    CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  Off  | 00000000:18:00.0 Off |                  N/A |
| 29%   32C    P8     7W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GeForce GTX 108...  Off  | 00000000:3B:00.0 Off |                  N/A |
| 29%   33C    P8     8W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  GeForce GTX 108...  Off  | 00000000:86:00.0 Off |                  N/A |
| 29%   31C    P8     8W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   3  GeForce GTX 108...  Off  | 00000000:AF:00.0 Off |                  N/A |
| 29%   31C    P8     8W / 250W |      0MiB / 11178MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```