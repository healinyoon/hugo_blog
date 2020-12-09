---
title: "Kubernets(쿠버네티스) Troubleshooting (1) - PV Terminating에서 멈출 때 "
date: 2020-12-09T10:56:19+09:00
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

# 에러

PV를 delete 하는 과정에서 계속 **"Terminating"** 상태에서 멈춰 있다.
```
# kubectl delete pv jenkins2-pv
persistentvolume "jenkins2-pv" deleted
```

```
# kubectl describe pv jenkins2-pv
Name:            jenkins2-pv
Labels:          <none>
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:
Status:          Terminating (lasts 27m)
Claim:           jenkins/jenkins
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        10Gi
Node Affinity:   <none>
Message:
Source:
    Type:      NFS (an NFS mount that lasts the lifetime of a pod)
    Server:    10.231.238.239
    Path:      /mnt/nfs_server/pv_jenkins2
    ReadOnly:  false
Events:        <none>
```

# 해결 방법
제거하려는 pv를 사용 중인 k8s resource를 먼저 제거해야 한다.
```
 # kubectl delete namespace jenkins
namespace "jenkins" deleted
```

그 후 pv도 바로 제거 된다.
```
# kubectl delete pv jenkins2-pv
persistentvolume "jenkins2-pv" deleted
```
