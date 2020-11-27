---
title: "네트워크 용어, 인터페이스 및 프로토콜 기초 이해하기"
date: 2020-11-27T11:22:24+09:00
draft: false
categories: [  
  "networks",
]
---

# 소개
서버를 관리하다보면 네트워크에 대한 지식에 아쉬움을 느낄 때가 종종 있다. [An Introduction to Networking Terminology, Interfaces, and Protocols](https://www.digitalocean.com/community/tutorials/an-introduction-to-networking-terminology-interfaces-and-protocols)의 내용을 정리하면서 네트워크의 기본 개념을 살펴보고자 한다.

# Networking Glossary
네트워크 기본 지식을 살펴보기 앞서, 자주 사용되는 용어를 정의하면 다음과 같다.

* Connection
    * 데이터 블록을 전달하기 위한 통로
    * 일반적으로 데이터 전송 전 'connection' 준비 / 데이터 전송 후 'connection' 해제
* Packet
    * 네트워크를 통해 전송되는 데이터의 가장 기본적인 단위(블록)
    * Packet = Header + Data + Tail
    * Header = packet의 송수신 주소, 타임 스탬프, network hop 등의 정보 포함
    * Body(or Payload) = 전송되는 데이터를 포함
* Network Interface
    * 네트워크 하드웨어와 통신하기 위한 네트워크 고유 소프트웨어
* WAN(Wide Area Network)
    * 광역 통신망
    * 일반적으로 인터넷 전체를 의미 => interface가 WAN에 연결 = 인터넷을 통해 연결 가능하다고 가정
    * LAN과 LAN 사이를 연결  
        ![](/images/20201127_network_basic/WAN.png)  
        그림 출처: https://ledgku.tistory.com/17
* LAN(Local Area Network)
    * 근거리 통신망
    * 가정 또는 회사 네트워크 
    * 주로 이더넷 프로토콜 사용
* Protocol
    * 데이터 교환 방법 및 순서에 대해 정의한 의사소통 약속, 규약 혹은 규칙 체계를 의미
    * Protocol의 종류는 매우 다양하며, 
* Port
    * 네트워크를 통해 데이터를 주고 받는 프로세스를 식별하기 위해 호스트 내부적으로 프로세스가 할당 받는 고유한 주소 값(number)
* Firwall
    * 서버의 in/out 트래픽의 허용 여부를 결정하는 프로그램
* NAT(Network Address Translation)
    * 네트워크 주소를 변환해주는 기능
    * public outside address와 private inside address의 사이에서 border router로서의 역할
    * 내부 망에서는 사설 IP 주소를 사용하여 통신을 하고, 외부망과의 통신시에는 NAT를 거쳐 공인 IP 주소로 자동 변환
* VPN(Virtual Private Network)
    * 가상 사설망
    * 보안을 유지하면서 인터넷을 통해 LAN에 연결
    
# Network Layers

### OSI Model

* Application
* Presentation
* Session
* Transport
* Network
* Data Link
* Physical

### TCP/IP Model

* Application
* Transport
* Internet
* Link

# Interfaces

# Protocols

### Media Access Control

### IP

### ICMP

### TCP

### UDP

### HTTP

### FTP

### DNS

### SSH

# Conclusion



# 참고
* https://www.ibm.com/support/knowledgecenter/ko/ssw_aix_71/network/tcpip_interfaces.html
* https://ledgku.tistory.com/17
* https://m.blog.naver.com/PostView.nhn?blogId=on21life&logNo=221509574568&proxyReferer=https:%2F%2Fwww.google.com%2F
* https://bmind305.tistory.com/25
* https://m.blog.naver.com/PostView.nhn?blogId=ssdyka&logNo=221376674886&proxyReferer=https:%2F%2Fwww.google.com%2F