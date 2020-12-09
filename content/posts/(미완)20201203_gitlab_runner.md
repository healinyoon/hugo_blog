---
title: "Gitlab Runner 사용해보기"
date: 2020-12-03T13:11:04+09:00
draft: true
categories: [  
  "cicd",
]
tags: [
  "gitlab",
]
---


Gitlab Runner
Gitlab Runner는 Gitlab CI/CD와 함께 동작하여 파이프라인 작업을 실행하는 애플리케이션이다. Gitlab Runner가 배포 서버에서 동작하기 위해서는 다음과 같이 각 배포 서버에 Gitlab Runner가 설치되어 있어야 하며, Gitlab 서버에 runner를 register 해두면, Gitlab repository에 이벤트가 발생했을 때 `.gitlab-ci.yml`에 작성된 스크립트에 따라 파이프라인 작업이 배포 서버에서 실행된다.

Gitlab Runner 설치하기
※ 주의 사항: Gitlab 서버와 동일한 버전이어야 한다. 

1. (배포 서버) Gitlab repository 추가
```
# curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
```

2. (Gitlab 서버) Gitlab 버전 확인
```
# yum list installed | grep gitlab
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
gitlab-ce.x86_64                        12.2.5-ce.0.el7                @gitlab_gitlab-ce
```

3. (배포 서버) 동일한 버전의 Gitlab Runner를 설치하기 위해, 설치 가능한 버전 확인
```
# apt-cache madison gitlab-runner
```

4. (배포 서버) Gitlab Runner 설치 및 확인
```
# apt-get install gitlab-runner=12.2.0

# # dpkg -l | grep gitlab
ii  gitlab-runner                              12.2.0                                           amd64        GitLab Runner
```

# 참고
* https://docs.gitlab.com/runner/
* https://namioto.ip.or.kr/2018/07/16/gitlab-ci%EB%A1%9C-%EC%9E%90%EB%8F%99%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0/

미완 이유: 프로젝트마다 runner를 매번 register 해줘야 함 -> 관리 포인트 너무 많음

그 외 추가로 **gitlab <-> k8s cluster**는 버전 이슈로 불가 -> [참고](https://gitlab.com/agepoly/it/infra/kubernetes/-/issues/44)