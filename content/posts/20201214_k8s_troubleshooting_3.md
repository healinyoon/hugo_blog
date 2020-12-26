---
title: "Kubernets(쿠버네티스) Troubleshooting (3) - Pod STATUS가 Running되지 않고 Error 또는 CrashLoopBackOff을 반복할 때"
date: 2020-12-14T11:50:39+09:00
draft: false
categories: [  
  "msa",
]
tags: [
  "docker",
  "kubernetes",
  "troubleshooting",
]
---

# 이슈

Jenkins를 Helm으로 설치하는 과정에서 발생한 이슈이다. 컨테이너 초기화가 계속 실패했는데, 아래와 같이 `STATUS`가 Running이 되지 않고 CrashLoopBackOff 또는 Error가 되는 것이다. 일단 `Init:0/1`은 '초기화 컨테이너라는 것'인데 자세한 내용은 아래의 공식 문서를 참고하자.

```
# kubectl get all -n jenkins
NAME                                         READY   STATUS     RESTARTS   AGE
pod/jenkins-0                                0/2     Init:0/1   0          4m31s
```

* [https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/ko/docs/concepts/workloads/pods/init-containers/)
* [https://kubernetes.io/ko/docs/tasks/debug-application-cluster/debug-init-containers/](https://kubernetes.io/ko/docs/tasks/debug-application-cluster/debug-init-containers/)

```
# kubectl get all -n jenkins
NAME             READY   STATUS                 RESTARTS   AGE
pod/jenkins-0   0/2     Init:CrashLoopBackOff   4          8m45s

```

여기서 중요한 포인트는 `Running`이 안되고 `Init:0/1` -> `CrashLoopBackOff` -> `Error`를 반복한다는 것이다.

`kubectl describe` 명령어로 Event를 확인하면 container init 과정에서 계속 에러가 발생한다.
```
# kubectl describe pod jenkins-0 -n jenkins
Name:         jenkins-0
Namespace:    jenkins
Priority:     0
Node:         ringu/x.x.x.x
Start Time:   Wed, 09 Dec 2020 20:19:26 +0900
Labels:       app.kubernetes.io/component=jenkins-controller
              app.kubernetes.io/instance=jenkins
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=jenkins
              controller-revision-hash=jenkins-584bd849f4
              statefulset.kubernetes.io/pod-name=jenkins-0

(중략)

Events:
  Type     Reason     Age                            From                Message
  ----     ------     ----                           ----                -------
  Normal   Scheduled  8m51s                          default-scheduler   Successfully assigned jenkins/jenkins-0 to ringu
  Normal   Started    <invalid> (x4 over <invalid>)  kubelet, ringu      Started container init
  Warning  BackOff    <invalid> (x7 over <invalid>)  kubelet, ringu      Back-off restarting failed container
  Normal   Pulling    <invalid> (x5 over <invalid>)  kubelet, ringu      Pulling image "jenkins/jenkins:lts"
  Normal   Pulled     <invalid> (x5 over <invalid>)  kubelet, ringu      Successfully pulled image "jenkins/jenkins:lts"
  Normal   Created    <invalid> (x5 over <invalid>)  kubelet, ringu      Created container init
```

이를 해결하기 위해서 node에 접근해서 container 로그를 확인해봤다.

```
# docker logs 0150155f58e9
Downloading plugin workflow-durable-task-step from url: https://updates.jenkins.io/download/plugins/workflow-durable-task-step/2.37/workflow-durable-task-step.hpi
Downloaded file is not a valid ZIP
Unable to download workflow-support
```

특정 플러그인을 설치하는데만 문제가 발생하나 싶어서 로그를 계속 확인해봤는데 '특정 플러그인에서만 발생하는 이슈는 아님'

플러그인 다운로드의 시간이 livenessProbe와 startupProbe에 설정한 시간을 초과하는 건가 싶어서 설정 조정 후 `helm upgrade`를 시도했으나, 여전히 안됐다.

> (참고) Helm Release Upgrade

```
# helm upgrade -f jenkins-values.yaml jenkins jenkins/jenkins --namespace=jenkins
```

# 해결 방법

결국 Plugin을 초기화할 때 설치하지 않도록 jenkins-values.yaml에서 주석 처리했다.

> jenkins-values.yaml

```
변경 전
installPlugins:
    - kubernetes:1.27.6
    - workflow-aggregator:2.6
    - git:4.4.5
    - configuration-as-code:1.46
변경 후
installPlugins:
   # - kubernetes:1.27.6
   # - workflow-aggregator:2.6
   # - git:4.4.5
   # - configuration-as-code:1.46
```

그리고 다시 helm release upgrade하면 정상 동작을 확인할 수 있다.

```
# helm upgrade -f jenkins-values.yaml jenkins jenkins/jenkins --namespace=jenkins
```

```
# kubectl get all -n jenkins
NAME            READY   STATUS    RESTARTS   AGE
pod/jenkins-0   2/2     Running   0          6m20s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/jenkins         NodePort    10.109.131.192   <none>        8080:8080/TCP   178m
service/jenkins-agent   ClusterIP   10.99.253.235    <none>        50000/TCP        178m

NAME                       READY   AGE
statefulset.apps/jenkins   1/1     178m
```

대신 필요한 플러그인을 설치하고 보안 설정도 해줘야 한다.

- kubernetes
- workflow-aggregator
- git
- configuration-as-code

# 참고
* https://stackoverflow.com/questions/57790463/cans-initiate-jenkins-using-stable-helm