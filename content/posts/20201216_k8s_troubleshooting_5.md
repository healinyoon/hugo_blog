---
title: "Kubernets(쿠버네티스) Troubleshooting (5) - docker login 안될 때 Cannot autolaunch D-Bus without X11 $DISPLAY"
date: 2020-12-22T16:40:24+09:00
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

```
# docker login 10.231.238.220:31000
Username: admin
Password:
Error saving credentials: error storing credentials - err: exit status 1, out: `Cannot autolaunch D-Bus without X11 $DISPLAY`
```

# 해결 방법

```
# sudo apt remove golang-docker-credential-helpers
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libpython-stdlib python python-asn1crypto python-backports.ssl-match-hostname python-cached-property python-certifi
  python-cffi-backend python-chardet python-cryptography python-dockerpty python-docopt python-enum34 python-funcsigs
  python-functools32 python-idna python-ipaddress python-jsonschema python-minimal python-mock python-openssl python-pbr
  python-pkg-resources python-requests python-six python-texttable python-urllib3 python-websocket python-yaml python2.7
  python2.7-minimal
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  docker-compose golang-docker-credential-helpers python-docker python-dockerpycreds
0 upgraded, 0 newly installed, 4 to remove and 37 not upgraded.
After this operation, 2,438 kB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 207323 files and directories currently installed.)
Removing docker-compose (1.17.1-2) ...
Removing python-docker (2.5.1-1) ...
Removing python-dockerpycreds (0.2.1-1) ...
Removing golang-docker-credential-helpers (0.5.0-2) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

# 정상 로그인 확인

```
# docker login {docker private registry}
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

# 참고
* https://sukill.tistory.com/100
