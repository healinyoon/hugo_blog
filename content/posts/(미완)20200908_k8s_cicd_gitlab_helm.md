---
title: "Kubernetes CI/CD 구축하기 - Tool (1) Gitlab + Helm"
date: 2020-09-08T13:02:19+09:00
draft: true
categories: [
  "msa",
  "ci/cd",
]
tags: [
  "docker",
  "kubernetes",
  "gitlab",
  "helm",
]
---

# Intro

> There is no single, best set of tools for continuous integration / continuous delivery (CI/CD) with Kubernetes — each organization will use the tools that are best suited for its specifc use case. - [CI/CD WITH KUBERNETES, THE NEW STACK](https://thenewstack.io/ebooks/kubernetes/ci-cd-with-kubernetes/)

Kubernetes 기반의 CI/CD 구축을 위해 다양한 Best Practice를 찾아봤지만
* 완벽하게 지원하는 단일 tool은 없었고
* 다양한 사례에서 살펴본 tool set은 구축 배경에 따라 tool 구성이 다양했다.

따라서 다양한 tool을 사용하여 Kubernetes CI/CD를 구축해보고, 우리 팀에 가장 적합한 tool 조합을 찾아보고자 한다.

## 사전 요구 사항
1) Gitlab 설치
2) Kubernetes Cluster 구성
3) Helm 설치




