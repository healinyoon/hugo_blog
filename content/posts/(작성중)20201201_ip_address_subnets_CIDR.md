---
title: "네트워킹을 위한 IP address, Subnet, CIDR 표기법 이해하기"
date: 2020-12-01T13:13:54+09:00
draft: false
categories: [  
  "networks",
]
---

# 소개

서버를 관리하다보면 네트워크에 대한 지식에 아쉬움을 느낄 때가 종종 있다. [Understanding IP Addresses, Subnets, and CIDR Notation for Networking](https://www.digitalocean.com/community/tutorials/understanding-ip-addresses-subnets-and-cidr-notation-for-networking)의 내용을 정리하면서 네트워크의 기본 개념을 살펴보고자 한다. 이번 글은 IP 주소 그룹화를 위한 네트워크 클래스, 서브넷 및 CIDR 표기법을 줄 살펴본다.

# Understanding IP addresses

네트워크 상의 모든 장소 또는 장치는 "주소로 지정 가능해야 한다". 즉, 주소를 통해(by referencing under a predefined system of addresses) 네트워크 상에서 접근 가능해야한다. 일반적으로 네트워크에서 언급되는 'address'는 IP 주소를 의미한다.

IP 주소를 사용하면 네트워크 인터페이스를 통해서 네트워크 리소스에 접근 가능해진다. 한 컴퓨터가 다른 컴퓨터와 통신하려면 원격 컴퓨터의 IP 주소를 사용한다. 두 컴퓨터가 동일한 네트워크에 있거나, 혹은 두 컴퓨터 사이의 다른 장치가 네크워크 요청을 변환하여 전달하면 두 컴퓨터는 서로 통신 가능하다.

네트워크 내에서의 각 IP 주소는 고유한 값이여야 한다. 네트워크는 다른 네트워크와 격리될 수 있으며, 서로 다른 네트워크 간의 연결(bridged)과 변환(translation)이 가능하다. **네트워크 주소 변환(Network Address Translation)**은 packet(= 네트워크를 통해 전송되는 데이터의 가장 기본적인 단위)이 다른 네트워크에 전달되기 위해 주소를 변환하여 원하는 목적지로 도달할 수 있도록 한다. 물론 격리된 네트워크는 동일한 IP를 가질 수 있다.


### The difference between IPv4 and IPv6

오늘날 가장 널리 사용되는 IP Protocol은 Ipv4와 IPv6이다. IP Protocol의 4번째 버전인 IPv4는 현재 대부분의 시스템에서 지원된다. 새로운 6번째 버전인 IPv6는 IPv4 주소 공간의 부족과 protocol의 개선으로 더 자주 배포되고 있다. 인터넷에 연결된 장치가 IPv4로 표현할 수 있는 주소 공간의 한계보다 많기 때문이다.

IPv4는 32bit으로 표현되는 주소이다. 주소의 각 8bit segment는 .(dot)으로 나뉘고 0~255 사이의 값으로 표현 가능하다. 각 segment는 사람에게 읽기 쉽게 십진수로 표현되지만, 8bit의 배열 표기이므로 일반적으로 **octet**이라고 한다.

일반적인 IPv4 주소는 다음과 같다.

```
192.168.0.5
```

각 octet의 가장 낮은 값은 0이고, 가장 높은 값은 255이다. 위의 표현을 바이너라로 표현하면 다음과 같다(가독성을 위해 4bit을 공백으로 구분하고 '.'은 '-'로 대체한다).

```
1100 0000 - 1010 1000 - 0000 0000 - 0000 0101
```

IPv4와 IPv6은 protocol 및 background 기능에 차이가 있지만 가장 큰 차이점은 **주소 공간**이다. IPv6는 128bit으로 주소를 표현한다. 즉 IPv4에 비해 7.9*10^28배 더 많은 주소 공간 표현이 가능하다.

IPv6는 일반적으로 4개의 16진수로 이루어진 8개의 segment로 표현된다. 일반적인 IPv6의 주소는 다음과 같다.

```
1203:8fe0:fe80:b897:8990:8a7c:99bf:323d
```

# IPv4 Addresses Classes and Reserved Ranges

### Reserved Private Ranges

# Netmasks and Subnets

# CIDR Notation


# 참고
* https://www.digitalocean.com/community/tutorials/understanding-ip-addresses-subnets-and-cidr-notation-for-networking
