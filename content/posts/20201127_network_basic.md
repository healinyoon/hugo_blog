---
title: "네트워크 용어, 인터페이스 및 프로토콜 기초 이해하기"
date: 2020-11-27T11:22:24+09:00
draft: true
categories: [  
  "networks",
]
---

# 소개
서버를 관리하다보면 네트워크에 대한 지식에 아쉬움을 느낄 때가 종종 있다. [An Introduction to Networking Terminology, Interfaces, and Protocols](https://www.digitalocean.com/community/tutorials/an-introduction-to-networking-terminology-interfaces-and-protocols)의 내용을 정리하면서 네트워크의 기본 개념을 살펴보고자 한다.

# Networking Glossary
네트워크 기본 지식을 살펴보기 앞서, 자주 사용되는 용어를 정의하면 다음과 같다.

* Connection: 데이터 블록을 전달하기 위한 통로
    * 일반적으로 데이터 전송 전 'connection' 준비 / 데이터 전송 후 'connection' 해제
* Packet: 네트워크를 통해 전송되는 데이터의 가장 기본적인 단위(블록)
    * Packet = Header + Data + Tail
    * Header = packet의 송수신 주소, 타임 스탬프, network hop 등의 정보 포함
    * Body(or Payload) = 전송되는 데이터를 포함
* Network Interface: 네트워크 하드웨어와 통신하기 위한 네트워크 고유 소프트웨어
* WAN(Wide Area Network): 광역 통신망
    * 일반적으로 인터넷 전체를 의미 => interface가 WAN에 연결 = 인터넷을 통해 연결 가능하다고 가정
    * LAN과 LAN 사이를 연결
        ![그림 출처: https://ledgku.tistory.com/17](/images/20201127_network_basic/WAN.png)
* LAN(Local Area Network): 근거리 통신망
    * 가정 또는 회사 네트워크 
    * 주로 이더넷 프로토콜 사용
* Protocol
* Port
* Firwall
* NAT
* VPN

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