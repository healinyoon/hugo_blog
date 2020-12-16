---
title: "Kubernets(쿠버네티스) Troubleshooting (4) - Jenkins Agent로 Slave Pod가 뜨지 않고 계속 죽을 때"
date: 2020-12-16T16:40:24+09:00
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

Jenkins Kubernetes 플러그인을 사용하여 Jenkins Slave Pod를 배포하려고 하는 데 아래와 같은 에러가 계속 발생했다.

### {pod name} is offline 또는 Pod 재시작 무한 반복
![](/images/20201216_k8s_troubleshooting_4/error1.png)

### Still waiting to schedule task "Jenkins doesn't have label {pod name}
![](/images/20201216_k8s_troubleshooting_4/error2.png)


Pod가 배포된 노드에서 `docker logs {container name}` 명령어로 컨테이너 로그를 확인해보니 다음과 같은 내용이 출력되었다.

```
# docker logs 53e57b2202c7
Dec 16, 2020 8:32:45 AM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: k8s-pipeline-chatbot-pipeline-24-c8mmg-r47mn-mgpnd
Dec 16, 2020 8:32:45 AM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
Dec 16, 2020 8:32:45 AM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 4.3
Dec 16, 2020 8:32:45 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
Dec 16, 2020 8:32:45 AM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
Dec 16, 2020 8:32:46 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://x.x.x.x:31122/]
Dec 16, 2020 8:32:46 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
Dec 16, 2020 8:32:46 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver isPortVisible
WARNING: Connection refused (Connection refused)
Dec 16, 2020 8:32:46 AM hudson.remoting.jnlp.Main$CuiListener error
SEVERE: http://x.x.x.x:31122/ provided port:50000 is not reachable
java.io.IOException: http://x.x.x.x:31122/ provided port:50000 is not reachable
	at org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver.resolve(JnlpAgentEndpointResolver.java:314)
	at hudson.remoting.Engine.innerRun(Engine.java:693)
	at hudson.remoting.Engine.run(Engine.java:518)
```

`SEVERE: http://x.x.x.x:31122/ provided port:50000 is not reachable` = **즉 jenkins-agent에 연결이 안된다는 것..**

# 해결 방법

[Jenkins 관리] > [시스템 설정] > [Cloud(The cloud configuration has moves to a separate configuration page)]에 접근하여 아래와 같이 설정해준다.

![](/images/20201214_k8s_ci_cd/set-k8s-plugin1.png)
![](/images/20201214_k8s_ci_cd/set-k8s-plugin2.png)

중요한 점은 jenkins-agent pod는 아래와 같이 `ClusterIP` serviceType으로 배포되어 있기 때문에 **coreDNS**로 Jenkins tunnel을 입력해줘야 한다는 것이다!! coreDNS 형식은 `{svc name}.{namespace}.svc.cluster.local:50000`

```
# kubectl get svc -n jenkins -o wide
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE    SELECTOR
jenkins          NodePort    10.107.89.164    <none>        8080:31122/TCP   6d2h   app.kubernetes.io/component=jenkins-controller,app.kubernetes.io/instance=jenkins
jenkins-agent    ClusterIP   10.109.207.50    <none>        50000/TCP        6d2h   app.kubernetes.io/component=jenkins-controller,app.kubernetes.io/instance=jenkins
```

> Jenkins tunnel: Connect to the specified host and port, instead of connecting directly to Jenkins. Useful when connection to Jenkins needs to be tunneled. Can be also HOST: or :PORT, in which case the missing portion will be auto-configured like the default behavior

이제 아름답게 잘된다..

![](/images/20201216_k8s_troubleshooting_4/good1.png)
![](/images/20201216_k8s_troubleshooting_4/good2.png)



