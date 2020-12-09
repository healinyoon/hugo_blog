---
title: "Kubernets(쿠버네티스) Troubleshooting (2) - Node가 NotReady됐는데 docker가 죽어있을 때 "
date: 2020-12-09T16:11:19+09:00
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

# 요약
* k8s cluster의 특정 Node가 NotReady됐는데 docker가 stop 상태였다.
* 2개의 원인이 만나서 발생: apt 자동 업데이트 + docker.io 패키지 버그
* 해결 방법: systemd에 패키지를 추가해줘야 하는데 -> 문제는 이게 버그를 무조건 한번은 다시 트리거 한다. -> k8s cluster의 모든 도커가 발생 가능한 이슈이므로 -> 한번씩은 다 죽다 살아나야 한다는 이야기..(근데 어자피 설정 안하면 도커가 언제 죽어도 안 이상한 상태가 된다.)


# 이슈

docker가 혼자서 죽었다. 죽은 원인을 확인해보자.

```
# journalctl -u docker --since "2020-11-01 00:00:00"
-- Logs begin at Tue 2020-10-13 18:15:48 KST, end at Wed 2020-12-09 14:31:19 KST. --
12월 02 06:56:02 test-server systemd[1]: Stopping Docker Application Container Engine...
12월 02 06:56:02 test-server dockerd[16376]: time="2020-12-02T06:56:02.357507251+09:00" level=info msg="Processing signal 'terminated'"

(중략)

```

`12월 02 06:56:02 test-server systemd[1]: Stopping Docker Application Container Engine...` systemd에 의해서 docker가 stop되고 다시 start 되지 않는다.


# 원인([ubuntu package bug 페이지](https://bugs.launchpad.net/ubuntu/+source/containerd/+bug/1870514))

2개의 이슈가 만나서 발생했다. apt 자동 업데이트 + docker.io 패키지 버그

## 1. apt 자동 업데이트

apt는 unattended updates 라는 것이 있다. 확인해보자.
```
# journalctl -u apt* --since "2020-12-01 00:00:00"

(중략)

12월 02 06:55:52 test-server systemd[1]: Starting Daily apt upgrade and clean activities...
12월 02 06:56:25 test-server systemd[1]: Started Daily apt upgrade and clean activities.
```

이게 자동으로 패키지를 업데이트 하는데.. 이것이 아래의 docker.io 패키지 버그와 만나서 docker를 죽여놓고 살려놓지 않는 참사가 발생한다.

## 2. docker.io 패키지 버그

"bug in the docker.io package causing the service to stop on package upgrade"의 소스코드는 `/var/lib/dpkg/info/docker.io.prerm` 여기에 있다.
이렇게 생겼다.
```
# because we had to use "dh_installinit --no-start", we get to make sure the daemon is stopped on uninstall
# (this is adapted from debhelper's "autoscripts/prerm-init-norestart")
if [ "$1" = remove ]; then
        invoke-rc.d docker stop
fi
```

결론적으로는 stop만 하고 start는 안한다는 것...


# 해결 방법
* https://launchpad.net/~bryce/+archive/ubuntu/containerd-sru-lp1870514-docker-dh/

```
sudo add-apt-repository ppa:bryce/containerd-sru-lp1870514-docker-dh
sudo apt-get update
```

근데 문제가 있다. 이거 업데이트 할 때 위의 `prerm` 스크립트를 한번은 더 실행해야 하고 이건 **버그를 한번 더 발생 시킨다 = 도커가 한 번 더 죽는다**(unfortunately any change we make to docker.io requires the running of the prerm script (the version of the script already present on your system, not the one we'd be installing), and thus triggers the bug).
다른 서버의 도커에서도 발생할 수 있다 -> k8s cluster의 모든 노드가 설정을 위해서 한번씩은 죽었다가 살아나야 한다..ㅠㅠ