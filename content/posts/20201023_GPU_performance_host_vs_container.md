---
title: "GPU 성능 비교하기: Host vs Container"
date: 2020-10-23T10:50:18+09:00
draft: false
categories: [
    "docker",
]
tags: [
    "msa",
]
---

# Intro
회사에서 쿠버네티스 업무를 하다가 "컨테이너 위에서도 GPU 성능이 보장되는가?"라는 질문이 나왔다.
쿠버네티스 운영 중에 자주 마주하는 질문이므로, 이번 기회에 정리를 해보려고 한다.

### [ 목차 ]
1.  Infra workstation 정보
2.  T4 정보
3.  T4 -  Host 설정
4.  T4 -  Container 설정
5.  T4 -  Host vs Container 성능 비교
6.  상세 - Host
7.  상세 - Container

### [ 요약 ]

#### 결론
GPU on Host VS Container를 비교하였을 때 속도의 성능 차이가 없었음

#### 상세 내용
* GPU Performance on Host VS Container 비교시 정확도에는 차이가(모델이 같으므로) 없으므로, 속도만 비교하였다.
* GPU를 1개 사용하였을 경우와 2개 사용하였을 경우를 나누어 비교하였다.
* 동일한 조건에 대해 테스트 케이스(횟수)를 총 2회씩 수행하였다.

# 1. 정보 확인

### 2.1. GPU 현황
```
GPU 0: Tesla T4 (UUID: GPU-f881729c-c452-b42b-fab6-ab0647359a66)
GPU 1: Tesla T4 (UUID: GPU-9e53cd27-5256-0904-95f3-0e6b7b9cce1d)
GPU 2: Tesla T4 (UUID: GPU-5491ddfe-3d4f-283c-800c-fbd7f99fd87f)
GPU 3: Tesla T4 (UUID: GPU-c2184c80-a215-c6bb-7db1-5a9d37b66961)
```

### 2.2. 도커 버전
```
Version:          19.03.6
```


# 3. Host 설정

### 3.1. 가상환경 생성 - /home/ldccai/PerformTestEnv/performTestEnv
```
# pip3 install virtualenv
# virtualenv /home/ldccai/PerformTestEnv/performTestEnv
```

### 3.2. Tensorflow 설치
```
# source /home/ldccai/PerformanceTest/performTestEnv/bin/activate
# python -m pip install --upgrade pip
# sudo -H pip install --upgrade tf-nightly-gpu
```

### 3.3. 성능 테스트 코드 실행
```
# source /home/ldccai/PerformanceTest/performTestEnv/bin/activate
# cd /home/ldccai/PerformanceTest/
# git clone https://github.com/tensorflow/benchmarks.git
# cd benchmarks
```

# 4. Container 설정

### 4.1. 컨테이너 이미지 다운로드
```
root@ubuntu:~# docker pull tensorflow/tensorflow:nightly-gpu
```

### 4.2. 컨테이너 run with GPU
```
root@ubuntu:~# docker container run -d --gpus all -it tensorflow/tensorflow:nightly-gpu
94515f052adbddf25bb9c66e5d5d7ee6a6010cfc74c6936f3b01bdf764202488
```

### 4.3. 컨테이너 접속
```
root@ubuntu:~# docker exec -it 94515f052adb /bin/bash
"docker exec" requires at least 2 arguments.
See 'docker exec --help'.

Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container
root@ubuntu:~# docker container exec -it 94515f052adb /bin/bash

________                               _______________
___  __/__________________________________  ____/__  /________      __
__  /  _  _ \_  __ \_  ___/  __ \_  ___/_  /_   __  /_  __ \_ | /| / /
_  /   /  __/  / / /(__  )/ /_/ /  /   _  __/   _  / / /_/ /_ |/ |/ /
/_/    \___//_/ /_//____/ \____//_/    /_/      /_/  \____/____/|__/


WARNING: You are running this container as root, which can cause new files in
mounted volumes to be created as the root user on your host machine.

To avoid this, run the container by specifying your user's userid:

$ docker run -u $(id -u):$(id -g) args...
```

### 4.4. 파이썬 버전 확인
```
root@282efcfdc3c8:/# python
Python 3.6.9 (default, Apr 18 2020, 01:56:04)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
2020-06-22 11:16:19.882107: I tensorflow/stream_executor/platform/default/dso_loader.cc:48] Successfully opened dynamic library libcudart.so.10.1
>>> tf.__version__
'2.3.0-dev20200621'
```

### 4.5.  git 설치 / 리포지토리 다운로드
```
root@94515f052adb:/# apt-get install git
root@94515f052adb:/# git clone https://github.com/tensorflow/benchmarks.git
```

# 5. T4 - Host vs Container 성능 비교

### 5.1. GPU 1개 사용
```
(performTestEnv) # CUDA_VISIBLE_DEVICES={GPU 디바이스} python tf_cnn_benchmarks.py --num_gpus=1 --batch_size={배치사이즈} --model={모델명} --variable_update=parameter_server
```

| 테스트 케이스(횟수) |	Host images/sec | Container images/sec | 비고 | 
| --- | --- | --- | --- |
| 1 | 124.77 | 126.95 | model: Resnet50 |
| 2 | 125.76 | 125.07 | model: Resnet50 |

### 5.2. GPU 2개 사용
```
(performTestEnv) # CUDA_VISIBLE_DEVICES={GPU 디바이스1, GPU 디바이스2} python tf_cnn_benchmarks.py --num_gpus=2 --batch_size={배치사이즈} --model={모델명} --variable_update=parameter_server
```

| 테스트 케이스(횟수) | Host images/sec | Container images/sec | 비고 |
| --- | --- | --- | --- |
| 1 | 242.60 | 240.64 | model: Resnet50 |
| 2 | 241.52 | 236.72 | model: Resnet50 |

# 6. 상세 - Host

### 6.1. 테스트 1

#### 6.1.1. 조건
```
(performTestEnv) # CUDA_VISIBLE_DEVICES=3 python tf_cnn_benchmarks.py --num_gpus=1 --batch_size=64 --model=resnet50 --variable_update=parameter_server

(중략)

Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 13968 MB memory) -> physical GPU (device: 0, name: Tesla T4, pci bus id: 0000:d8:00.0, compute capability: 7.5)
TensorFlow:  2.3
Model:       resnet50
Dataset:     imagenet (synthetic)
Mode:        training
SingleSess:  False
Batch size:  64 global
             64 per device
Num batches: 100
Num epochs:  0.00
Devices:     ['/gpu:0'] // GPU 1개 사용
NUMA bind:   False
Data format: NCHW
Optimizer:   sgd
Variables:   parameter_server
```

#### 6.1.2. 결과
```
Done warm up
Step	Img/sec	total_loss
1	images/sec: 127.9 +/- 0.0 (jitter = 0.0)	7.608
10	images/sec: 126.4 +/- 0.2 (jitter = 0.4)	7.849
20	images/sec: 126.3 +/- 0.1 (jitter = 0.5)	8.013
30	images/sec: 126.2 +/- 0.1 (jitter = 0.7)	7.940
40	images/sec: 126.0 +/- 0.1 (jitter = 0.7)	8.137
50	images/sec: 125.8 +/- 0.1 (jitter = 0.6)	8.052
60	images/sec: 125.6 +/- 0.1 (jitter = 0.7)	7.782
70	images/sec: 125.4 +/- 0.1 (jitter = 0.9)	7.856
80	images/sec: 125.3 +/- 0.1 (jitter = 1.0)	8.011
90	images/sec: 125.1 +/- 0.1 (jitter = 1.2)	7.843
100	images/sec: 124.8 +/- 0.1 (jitter = 1.4)	8.090
----------------------------------------------------------------
total images/sec: 124.77
----------------------------------------------------------------
```

### 6.2. 테스트 2
	
#### 6.2.1. 조건
```
(performTestEnv) # CUDA_VISIBLE_DEVICES=2,3 python tf_cnn_benchmarks.py --num_gpus=2 --batch_size=64 --model=resnet50 --variable_update=parameter_server

(중량)

Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:1 with 13968 MB memory) -> physical GPU (device: 1, name: Tesla T4, pci bus id: 0000:d8:00.0, compute capability: 7.5)
TensorFlow:  2.3
Model:       resnet50
Dataset:     imagenet (synthetic)
Mode:        training
SingleSess:  False
Batch size:  128 global
             64 per device
Num batches: 100
Num epochs:  0.01
Devices:     ['/gpu:0', '/gpu:1'] // GPU 2개 사용
NUMA bind:   False
Data format: NCHW
Optimizer:   sgd
Variables:   parameter_server
```

#### 6.2.2. 결과
```
Done warm up
Step	Img/sec	total_loss
1	images/sec: 246.2 +/- 0.0 (jitter = 0.0)	7.749
10	images/sec: 244.9 +/- 0.6 (jitter = 1.7)	7.892
20	images/sec: 244.3 +/- 0.4 (jitter = 2.6)	7.968
30	images/sec: 244.5 +/- 0.4 (jitter = 2.3)	7.934
40	images/sec: 244.2 +/- 0.3 (jitter = 2.6)	8.016
50	images/sec: 243.8 +/- 0.3 (jitter = 2.6)	7.922
60	images/sec: 243.6 +/- 0.3 (jitter = 2.0)	7.872
70	images/sec: 243.5 +/- 0.2 (jitter = 1.9)	7.837
80	images/sec: 243.3 +/- 0.2 (jitter = 1.8)	7.850
90	images/sec: 243.0 +/- 0.2 (jitter = 2.1)	7.859
100	images/sec: 242.7 +/- 0.2 (jitter = 2.2)	7.946
----------------------------------------------------------------
total images/sec: 242.60
----------------------------------------------------------------
```

# 7. 상세 - Container

### 7.1. 테스트 1

#### 7.1.1. 조건
```
root@94515f052adb:# CUDA_VISIBLE_DEVICES=3 python tf_cnn_benchmarks.py --num_gpus=1 --batch_size=64 --model=resnet50 --variable_update=parameter_server

(중략)

Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 13968 MB memory) -> physical GPU (device: 0, name: Tesla T4, pci bus id: 0000:d8:00.0, compute capability: 7.5)
TensorFlow:  2.3
Model:       resnet50
Dataset:     imagenet (synthetic)
Mode:        training
SingleSess:  False
Batch size:  64 global
             64 per device
Num batches: 100
Num epochs:  0.00
Devices:     ['/gpu:0'] // GPU 1개 사용
NUMA bind:   False
Data format: NCHW
Optimizer:   sgd
Variables:   parameter_server
```

#### 7.1.2. 결과
```
Done warm up
Step	Img/sec	total_loss
1	images/sec: 129.2 +/- 0.0 (jitter = 0.0)	7.608
10	images/sec: 128.7 +/- 0.4 (jitter = 0.9)	7.849
20	images/sec: 128.4 +/- 0.2 (jitter = 1.1)	8.013
30	images/sec: 128.4 +/- 0.2 (jitter = 0.8)	7.940
40	images/sec: 128.3 +/- 0.1 (jitter = 0.9)	8.136
50	images/sec: 128.1 +/- 0.1 (jitter = 0.9)	8.053
60	images/sec: 127.9 +/- 0.1 (jitter = 1.1)	7.784
70	images/sec: 127.7 +/- 0.1 (jitter = 1.4)	7.859
80	images/sec: 127.5 +/- 0.1 (jitter = 1.4)	8.014
90	images/sec: 127.3 +/- 0.1 (jitter = 1.4)	7.842
100	images/sec: 127.1 +/- 0.1 (jitter = 1.4)	8.090
----------------------------------------------------------------
total images/sec: 126.95
----------------------------------------------------------------
```

### 7.2. 테스트 2

#### 7.2.1. 조건
```
root@94515f052adb:# CUDA_VISIBLE_DEVICES=2,3 python tf_cnn_benchmarks.py --num_gpus=2 --batch_size=64 --model=resnet50 --variable_update=parameter_server

(중략)

Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:1 with 13968 MB memory) -> physical GPU (device: 1, name: Tesla T4, pci bus id: 0000:d8:00.0, compute capability: 7.5)
TensorFlow:  2.3
Model:       resnet50
Dataset:     imagenet (synthetic)
Mode:        training
SingleSess:  False
Batch size:  128 global
             64 per device
Num batches: 100
Num epochs:  0.01
Devices:     ['/gpu:0', '/gpu:1'] // GPU 2개 사용
NUMA bind:   False
Data format: NCHW
Optimizer:   sgd
Variables:   parameter_server
```

#### 7.2.2. 결과
```
Done warm up
Step	Img/sec	total_loss
1	images/sec: 240.6 +/- 0.0 (jitter = 0.0)	7.749
10	images/sec: 243.4 +/- 0.8 (jitter = 3.5)	7.892
20	images/sec: 243.3 +/- 0.5 (jitter = 2.7)	7.968
30	images/sec: 243.1 +/- 0.4 (jitter = 2.8)	7.934
40	images/sec: 242.6 +/- 0.3 (jitter = 2.3)	8.019
50	images/sec: 242.4 +/- 0.3 (jitter = 2.0)	7.923
60	images/sec: 242.1 +/- 0.3 (jitter = 1.8)	7.878
70	images/sec: 241.8 +/- 0.2 (jitter = 1.6)	7.834
80	images/sec: 241.5 +/- 0.2 (jitter = 1.6)	7.861
90	images/sec: 241.1 +/- 0.2 (jitter = 2.0)	7.852
100	images/sec: 240.8 +/- 0.3 (jitter = 2.1)	7.948
----------------------------------------------------------------
total images/sec: 240.64
----------------------------------------------------------------
```
