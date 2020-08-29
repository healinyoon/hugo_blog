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
이 페이지에서는 `kubeadm` tool을 설치하고 이를 사용하여 kubernetes cluster를 


# 구성

### 하드웨어
구성하려는 kubernetes cluster는 다음과 같다.

| 노드 | vCPU | RAM | Disk |
| --- | --- | --- | --- |
| Master | 2 | 8GiB |  |
| Worker01 | 2 | 8GiB |  |
| Worker02 | 2 | 8GiB |  |

# Docker 설치
아래의 링크를 참고하여 Docker를 설치합니다.
[Docker 설치하기](https://healinyoon.github.io/2019/06/20190611_docker_install/)

# Kubernetes 