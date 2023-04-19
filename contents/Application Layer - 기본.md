---
title : Application Layer - 기본
date: 2023/04/19
tags: ["OSI 7 Layer", "Network", '네트워크', '컴퓨터 네트워크', 'Application Layer']
---


# Appliaction Layer 알아보기

## 인터넷 프로토콜 스택

기본적으로 Application, Transport, Network, Link, physical 영역으로 나뉘며, 5계층의 각 계층별로 프로토콜 스택이 존재한다.

각 영역의 역할을 간략하게 소개하면 다음과 같다.

- Application : 네트워크 응용 프로그램을 도와주는 프로토콜 영역이다.
	- ex) FTP(파일 전송), SMTP(이메일 프로토콜), HTTP 등
- Transport : process to process 간 데이터 전송과 관련된 프로토콜이다.
	- ex) UDP, TCP
- Network : 출발지(source)로 부터 목적지(destination) 간 datagrams를 라우팅 한다.
	- ex) IP, routing protocols
- Link : 이웃하는 네트워크 영역간 데이터 전송을 담당한다
	- Ethernet, 802.111(wifi) 등
- physicl : 물리적 케이블

OSI 7 Layer의 presentation layer와 session layer가 application layer로 병합되어 보통 5계층을 통해 데이터 송수신이 이루어진다.

## Encapsulation 

client의 application 에서 통신을 요청할 때 어플리케이션에서 전송한 메세지는 Application Layer -> Physical Layer를 모두 거치며 각 Layer마다 추가적인 헤더가 붙게 된다.

이 헤더를 통해 각 영역 계층에서 필요한 정보를 주고받을 수 있게 되고, 데이터의 명명 단위가 달라진다.

 - application : message
 - transport : segment ( message + app header)
 - network : datagram ( message + transport header + app header)
 - link : frame ( message + network header + transport header + app header)

반대로 해당 데이터의 destination 호스트에서는 Physical Layer -> Application Layer 순서를 거치며 데이터가 전달된다. 즉 수신쪽의 각 계층이, 해당 계층에 맞는 헤더를 분석하며 decapculation 하고 최종적으로 application layer에는 메세지만 전달되게 된다.

=> 이런 계층 분리를 통해 개발자들이 프로그램을 작성할 때 비즈니스 로직에만 집중할 수 있어 빠른 어플리케이션 개발이 가능해 진다. 즉 네트워크 코어 시스템의 동작을 알 필요가 없다.


## Application Architecture

- client-server : 
	- client : 
		- 요청의 근원지이자 최초 요청 발생자
		- 유동 아이피 주소를 가진다.
		- 서로 직접적인 커뮤니케이션이 불가능하다.
	- server:
		- Always-on : 항상 요청을 받아드릴 준비가 되어있어야 한다. 
		- 영구적인 아이피 주소를 가진다.
- Peer to Peer :
	- 모든 호스트가 client 이자 server가 될 수 있다. 즉 하나의 어플리케이션이 client,server process 모두를 가질 수 있다.
	- 각각의 end system이 서로 직접적으로 상호작용 할 수 있다 : clien-server 모델에서 클라이언트가 다른 클라이언트와 통신하기 위해서는 server를 경유해야 하지만, P2P 모델은 모든 호스트가 클라이언트이자 서버이기 때문에 서로 직접 통신할 수 있다.
	- 자체 확장성 


## Addressing Process
하나의 호스트( ex: PC)에서 여러 프로세스가 가동될 수 있다. 즉 어떤 어플리케이션간의 통신인지를 식별하기 위해서는 호스트의 IP 주소 뿐만 아니라, 프로세스를 identify 할 수 있는 정보가 필요하다. 이게 바로 Port 이다.

즉 HTTP 메세지 등을 전송하여 통신할 때 destination 및 source 주소를 전달할 때 IP주소와 Port 번호 모두를 전달해야 한다.



## Transport Service 가 어플리케이션에게 제공해야 하는 요소

1. Data Integrity : sender 가 보낸 데이터와 receiver 가 받은 데이터가 동일한 데이터여야 하는 것
	- 어플리케이션의 특성에 따라 약간의 손실을 허용하기도 한다.
2. Timing : 데이터의 전송 시간과 관련된 요소, 딜레이, 지연 등을 의미
	- 가능한 한 적은 딜레이와 지연을 제공해야 하는 것
3. Throughput : 네트워크에서 데이터의 전송 속도이자 전송의 효율성을 나타내는 척도
	- Netflix 와 같은 스트리밍 어플리케이션 서비스는 이런 throughput을 중요하게 여김
4. security

## Transport Protocol services

### TCP Service

- Reliable Transport : 전송과 수신 측 간 믿을 수 있어야 한다. -> 데이터의 손실을 방지해야 한다.
- flow control : 전송자는 수신자를 놀라게 해선 안된다. -> 수신자가 감당할 수 없는 많은 양의 패킷을 한번에 보내지 않도록 컨트롤 해야한다.
- congestion control : 네트워크의 혼잡도에 맞게 패킷 손실이 발생하지 않도록 송신량을 제어해야 한다.

Reliable Transport + flow control + congestion control = Data Intergirty guarantee

- 제공하지 않는 것 : Timing, minimum throughput guarantee, security

즉 데이터 하나만큼은 책임지고 보장해줄 수는 있지만, TCP가 전송 속도나 지연시간 및 보안과 관련된 측면의 서비스는 제공하지 않는다는 것이다.

### UDP Service

- Unreliable Data Transfer
데이터 무결성, 전송 속도, 지연시간 및 보안과 관련된 측면의 서비스는 제공하지 않는다는 것이다. 왜 사용할까??



## Securing TCP
기본적으로 TCP는 보안과 관련된 서비스를 제공하지 않는다. 때문에 보안 서비스의 책임을 Application 영역에서 담당하도록 하여 TCP의 부족한 보안을 보완한다. 이를 위해 SSL(Secure Socket Layer)를 사용한다.



