---
title: "CI/CD와 Jenkins"
date: 2020-08-27T19:25:40+09:00
draft: false
tags: [
    "jenkins",
]
categories: [
    "ci/cd",
]
---

# CI/CD란

CI/CD는 애플리케이션 개발 단계를 자동화하여 보다 **작은 코드 단위**와 **짧은 주기**로 Test와 Build를 수행하고 고객에게 제공하는 방법을 의미합니다.  
![](/images/20200827_cicd_and_jenkins/jenkins_cicd01-cicd01.png)

### CI(Continuous Integration): 지속적 통합

개발자의 변경 사항이 정기적으로(최상의 경우 하루 여러번) **빌드** 및 **테스트** 되고 **공유 리포지토리에 병합**되는 프로세스입니다.  

### CD(Continous Deploy or ontinuous Delivery): 지속적 배포

[Jez Humble의 정의](https://continuousdelivery.com/2010/08/continuous-delivery-vs-continuous-deployment/)

**Continuous Deployment** is about **automating the release of a good build to the production environment**. In fact, Humble thinks it might be more accurate to call it **“continuous release.”**

**Continuous Delivery** is about **1) ensuring that every good build is potentially ready for production release.** At the very least, **2) it’s sent to the user acceptance test (UAT) environment.** **3) Your business team can then decide when a successful build in UAT can be deployed to production** —and they can do so at the push of a button.

상황에 따라 이미 운영되고 있는 프로덕션 환경에 바로 release 하는 것은 문제가 될 수 있습니다. 이러한 경우로 임베디드 소프트웨어 등이 해당됩니다.  
따라서 잠재적으로는 release 가능하지만, 프로덕션 환경에 자동으로 release 되지 않는 것이 **Continuous Delivery** 입니다. 
![](/images/20200827_cicd_and_jenkins/jenkins_cicd01-cicd03.png)

### 정리

* CI: 지속적인 빌드 / 테스트 / 통합
* CD: CI의 연장선 ~ Release 준비 완료(Delivery) or 제품 출시(Deploy)

---

# Jenkins for CI/CD

### 다양한 CI/CD tool

CI/CD 구현을 위한 다양한 tool이 있습니다.
![](/images/20200827_cicd_and_jenkins/jenkins_cicd01-cicd02.png)
[tool에 대한 자세한 정보 참고](https://landscape.cncf.io/category=continuous-integration-delivery&format=card-mode&grouping=category)

### 왜 Jenkins인가

Jenkins는 아래와 같은 강점을 가지고 있습니다.
* It is an open-source tool with great community support.
* It is easy to install.
* It has 1000+ plugins to ease your work. If a plugin does not exist, you can code it and share it with the community.
* It is free of cost.
* It is built with Java and hence, it is portable to all the major platforms.

하지만 단점 역시 존재합니다. CI/CD tool 중에서 자주 사용되는 [Jenkins vs Gitlab vs Travis 비교 글](https://owin2828.github.io/devlog/2020/01/09/cicd-1.html)을 참고하시면 Jenkins의 장단점을 이해하는데 도움이 됩니다.
