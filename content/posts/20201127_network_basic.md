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
        *  인터넷 내에서 데이터를 보내기 위한 라우팅을 효율적으로 하기 위해 데이터를 여러 조각으로 나누어 전송: 조각 = packet
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
    * Protocol의 종류는 매우 다양
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

![](/images/20201127_network_basic/osi_vs_tcpip.jpeg)
출처: https://shlee0882.tistory.com/110

OSI Model과 TCP/IP Model의 차이를 자세히 알고 싶다면 [여기](https://medium.com/harrythegreat/osi%EA%B3%84%EC%B8%B5-tcp-ip-%EB%AA%A8%EB%8D%B8-%EC%89%BD%EA%B2%8C-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-f308b1115359)를 참고한다.

### OSI Model

네트워크 통신의 여러 계층을 설명하는 일반적인 방법으로, Open System Interconnect를 의미한다. OSI는 7계층을 정의한다.

* Application:(7 layer)
    * 최종 사용자에게 가장 가까운 계층으로 => 7계층에서 작동하는 응용프로그램은 사용자와 직접적으로 상호 작용
    * 예) Chrome, Firefox, Safari, Skype, Outlook, Office 등등
* Presentation(6 layer)
    * 응용프로그램이나 네트워크를 위해 낮은 수준의 데이터를 "표현(변환)" 하는 기능
    * 데이터를 어떻게 표현할 지 결정
    * 3가지 기능 수행
        * 송신자에서 온 데이터를 해석하기 위한 응용 계층 데이터 부호화, 변화
        * 수신자에서 데이터의 압축을 풀 수 있는 방식으로 된 데이터 압축
        * 데이터 암호화/복호화
* Session(5 layer)
    * 서로 다른 노드(device, 컴퓨터 or 서버) 간의 대화를 위한 session을 제공하는 역할
    * 지속적인 방식으로 노드 간의 연결을 생성, 유지(조율, 예: 시스템의 응답 대기 시간), 파괴
* Transport(4 layer)
    * 상위 계층의 안정적인 연결을 제공
    * 최종 시스템 및 호스트 간의 신뢰성있는 데이터 송수신을 위한 오류 검출, 복구, 흐름 제어, 중복 검사 등 수행
    * 프로토콜은 UDP와 TCP로 구분
    * port를 사용하여 다른 애플리케이션 처리
    * Data unit: Segment
* Network(3 layer)
    * 서로 다른 노드 간에 데이터(packet) 라우팅 담당
    * IP adress 정보를 가짐
    * Data unit: Packet
    * Device: 라우터, L3 스위치
* Data Link(2 layer)
    * Physical 계층에서 송수신되는 데이터의 오류 수정
    * 서로 다른 노드 또는 장치 간의 안정적인 링크를 설정하고 유지
    * MAC adress 정보를 가짐
    * Data unit: Frame
    * Device: 스위치, 브릿지
* Physical(1 layer)
    * 연결에 사용되는 실제 물리적 장치를 처리
    * 전기적, 기계적, 기능적인 특성을 이용하여 데이터를 전송
    * Data unit: Bit(on/off)
    * Device: 케이블, 허브, 리피터

### TCP/IP Model

TCP protocol과 IP protocol을 OSI 7계층에 맞추어 간략화 시킨 모델이다.

* Application
    * 사용자 응용 프로그램으로부터 요청을 받아서 사용자 데이터를 생성하고 하위 계층으로 전달하는 역할
    * OSI 7계층의 Application + Presentation + Session 
* Transport
    * 프로세스 간의 통신 담당
    * 프로토콜은 UDP와 TCP로 구분
    * port를 사용하여 다른 애플리케이션 처리
* Internet
    * Transport 계층에서 받은 packet을 목적지까지 효율적으로 전달하는 기능
    * 데이터의 주소를 판독하고 주소에 맞는 네트워크 탐색, 해당 host가 데이터를 수신할 수 있도록 송신
* Link
    * Internet 계층에 주소로 사용 가능한 인터페이스를 제공하기 위해 로컬 네트워크의 실제 topology 구현
    * 인접 node간의 connection을 구축하고 데이터 전송

# Interfaces

* 컴퓨터의 네트워크 통신 point로, 각 interface는 물리 또는 가상 네트워크 장치와 연결
* 일반적으로 서버는 Ethernet 또는 무선 인터넷 카드에 대한 구성 가능한 network interface를 하나씩 가지고 있음
* loopback 또는 local host interface 라고 하는 가상 network interface를 정의
    * 단일 컴퓨터의 응용 프로그램 및 프로세스를 다른 응용 프로그램 및 프로세스에 연결하는 interface로 사용
    * 많은 tool에서 "IO" interface로 참조되는 것을 확인 가능
* 일반적으로 관리자는 인터넷 트래픽을 처리하기 위한 interface와 LAN 또는 prive network를 위한 interface로 각각 구성
* DigitalOcean(사설 네트워트를 사용하는 데이터 센터)에서 VPS는 local interface 외에도 2개의 interface를 가짐
    * "eth0": 인터넷 트래픽 처리
    * "eth1": private network와 통신

# Protocols

네트워크에서 데이터 교환 방법 및 순서에 대해 정의한 의사소통 약속, 규약 혹은 규칙 체계를 의미한다. 낮은 네트워크 계층에서 높은 네트워크 계층까지 사용되는 다양한 프로토콜을 살펴보자.

### MAC(Media Access Control)

* Device를 구별하기 위한 protocol로 모든 device는 제조 과정에서 고유한 MAC adress 부여
* Link 계층

### IP(Internet Protocol)

* Internet Protocol Suite
* IP 주소는 각 네트워크에서 고유하며 이를 통해 컴퓨터 네트워크를 통해 서로 주소를 지정
* TCP/IP 모델의 Internet 계층 또는 OSI의 Network 계층

### ICMP(Internet Control Message Protocol)

* Internet Protocol Suite
* 가용성 또는 오류 조건을 표시하기 위해 장치간에 메시지를 보내는 데 사용
    * `ping`, `tracerroute`와 같은 네트워크 진단 도구에 사용
* OSI의 Network 계층

### TCP(Transmission Control Protoco)

* Internet Protocol Suite
* 데이터를 메세지의 형태로 보내기 위해 IP와 함께 사용하는 protocol
    * TCP의 역할: 전송하는 데이터를 추적 및 관리
    * IP의 역할: 데이터의 전송을 담당
* 연결형 protocol
    * 신뢰성 있는 데이터 전송
* Transport 계층

### UDP(User Datagram Protocol)

* Internet Protocol Suite
* 데이터를 메세지의 형태로 보내기 위해 사용되는 protocol
* 비연결형 protocol
    * 낮은 신뢰성
* Transport 계층

### HTTP

* W3(World Wide Web, WWW) 상에서 정보를 주고 받기 위한 protocol
* Application 계층

### FTP(File Transfer Protocol)

* 서버와 클라이언트 사이의 파일 전송을 위한 protocol
* Application 계층

### DNS(Domain Name System)

* 도메인 이름 <-> IP 주소로 변환하는 protocol
* TCP/IP 모델의 Application 계층

### SSH(Secure SHell)

* 네트워크 상의 컨퓨터 또는 device와 통신하기 위한 암호화된 protocol
* Application 계층


# 참고
* https://www.ibm.com/support/knowledgecenter/ko/ssw_aix_71/network/tcpip_interfaces.html
* https://ledgku.tistory.com/17
* https://m.blog.naver.com/PostView.nhn?blogId=on21life&logNo=221509574568&proxyReferer=https:%2F%2Fwww.google.com%2F
* https://bmind305.tistory.com/25
* https://m.blog.naver.com/PostView.nhn?blogId=ssdyka&logNo=221376674886&proxyReferer=https:%2F%2Fwww.google.com%2F
* https://shlee0882.tistory.com/110
* https://medium.com/harrythegreat/osi%EA%B3%84%EC%B8%B5-tcp-ip-%EB%AA%A8%EB%8D%B8-%EC%89%BD%EA%B2%8C-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-f308b1115359