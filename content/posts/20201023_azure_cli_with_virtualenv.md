---
title: "Azure CLI 설치하기 with virtualenv"
date: 2020-09-15T14:53:35+09:00
draft: false
categories: [
  "cloud",
]
tags: [
  "azure",
]
---

### Azure CLI를 virtualenv 환경에서 설치하기

Host 서버에 그냥 설치하면 파이썬 버전 문제로 실행이 안될 수 있다.
`virtualenv` 를 설치하고 `az cli` 를 설치하는 방법이다.

```
$ pip install virtualenv
$ mkdir azure-cli-with-python3
$ which python3
$ cd azure-cli-with-python3
$ virtualenv -p /usr/bin/python3.5.
$ source ./bin/activate
$ sudo apt install python3-dev
$ pip install azure-cli
$ python --version
```

### ref)
* [The way to configure Azure CLI to use Python 3.5 on system where the default Python version is 2.x · Issue #2529 · Azure/azure-cli](https://github.com/Azure/azure-cli/issues/2529)