---
title: "쿠버네티스를 위한 JenkinsX 개념과 구축하기"
date: 2020-08-28T16:29:47+09:00
draft: false
---

# Kubernetes에서 CI/CD를 구축할 때 겪는 어려움
쿠버네티스 베이스에서 훌륭한 CI/CD 환경을 구축하기 위해서는 다음과 같은 어려움이 있습니다.
* Setup the cloud/kuberenetes & environments
* Move to containers
* Deploy containers into Kubernetes
  * Generate YAML or helm charts?
* Adopt CD(continuous Delivery) and promotion
* Automate everything and improve feedback
* Keep delivering actual business value to customers

자연히 개발자는 본래의 업무 외에도 추가적으로 수행해야하는 업무가 많아지게 됩니다.

# Jenkins X
JenkinsX는 이러한 문제를 해결하기 위해 등장했습니다. JenkinsX는 다음의 업무를 대신 수행해줍니다.
* Automates the installation/upgrade of tools
  * Helm, Skaffold, Kaniko, Jenkins, KSync, Monocular, Nexus etc
  * All configured + optimised for Kubernetes OOTB
* Automates CI/CD for your applications on Kubernetes
  * Docker images
  * Helm charts
  * Pipelines
* USES GitOps to manage promotion between environments
  * Test -> Staging -> Production
* Lots of feedback
  * E.g. commenting on issues as they hit Staging + Production


# Kubernetes + Jenkins X 시작하기

