---
title : Socket Programming
date: 2023/04/20
tags: ["UDP","TCP",'Transport Layer', 'Multiflexing', "Demultiflexing"]
author: 이인
---


# Transport Layer Overview

## Transport Layer vs Network layer

- Transport Layer : 프로세스간의 논리적 통신을 위한 계층이다.
- Network Layer : 호스트간의 논리적 통신을 위한 계층이다.

예로 집에 12명의 사람이 살 때, '호스트 들 = 집 들'로 볼 수 있고, '프로세스 들 = 해당 집에 있는 사람 들'로 볼 수 있다. 
A라는 집에서 B라는 집으로 우편을 보낸다고 할 때 해당 우편이 어떤 집으로 가야하는지를 알 기 위해 Network Layer간 논리적 통신을 통해 집의 위치를 확인하고, Transport Layer간 논리적 통신을 통해 해당 우편이 해당 집의 어떤 사람에게 전달되어야 하는지를 확인한다.

이때 TCP 와 UDP는 보장해야할 서비스가 다르다.

- TCP
	- congestion control
	- flow control
	- connection setup : for reliability
		해당 서비스를 통해 최종적으로 data integrity를 보장해야 한다.
- UDP 
	- 최소한의 서비스만을 제공해야 한다 : no frills extension of best effort IP
	- 최소한의 서비스 제공 대신 빠른 데이터그램의 전송에 초점을 맞추어 낮은 지연시간과 높은 처리량을 제공하려고 함
	- 대신 unreliable, unordered delivery 라는 특징을 가짐

전반적으로 딜레이와 대역폭에 대한 보장은 Transport Layer가 담당하는 부분이 아니다.

![](https://i.imgur.com/FCOosMT.png)


## Data Encapsulation / Decapsulation

![](https://i.imgur.com/sqQgEnx.png)
각 계층의 역할을 보면 Encaplsulation 단계에서 추가되는 데이터들의 의미를 파악할 수 있다.
Transport Layer는 프로세스간 논리적 통신을 담당한다. 같은 호스트에서 프로세스를 구별하기 위해서는 당연히 포트 번호가 필요하다. 따라서 Application Layer에서 넘어온 데이터 청크에 포트번호를 붙인다.

마찬가지로 Network Layer는 호스트간 논리적 통신을 담당해야 한다. 따라서 패킷의 목적지가 되는 호스트의 위치정보가 담긴 IP 정보를 붙인다. 

Data Link Layer는 실질적인 하드웨어의 주소를 붙인다. 

Decapsulation 과정에서 수신자는 capsulation의 역과정을 통해 해당 패킷이 정상적인 패킷인지를 판단할 수 있다.


## Multiflexing, Demultiflexing

![](https://i.imgur.com/mq4tCV6.png)

p3 와 p4 가 개별적인 데이터 스트림으로 전송된 데이터가 서버측의 동일한 물리적 인터페이스(파이프)를 공유하여 서버측의 Transport Layer에 전달되고, 서버측 Transport Layer는 Demultiflexing 과정을 통해 패킷의 헤더 부분의 destination port 를 분석하여 적절한 소켓으로 전달해 서버 호스트의 알맞은 프로세스로 해당 패킷을 전달하게 된다.

반대로 서버측 개별 프로세스 P1, P2 각각에서 Transport Layer로부터 데이터 스트림을 전송받으며, Transport Layer 및 Network Layer 에서 Mulitflexing 과정을 통해 데이터 스트림의 각 패킷에 destination 및 source 각각의 IP 와 Port number를 붙여 **하나의 물리적 네트워크 인터페이스**를 통해 동시에 전송한다. 이때 각 데이터는 별개의 데이터 스트림으로 유지되지만, 물리적 네트워크 구성에 따라 일시적으로 하나의 통신 경로를 공유한다.

여기서 데이터 스트림이란 일련의 연속적인 데이터 요소들의 순차적으로 전송되는 데이터의 흐름을 의미한다. 이때 패킷이라는 작은 데이터 조각들로 나뉘어 전송되게 되며 각 패킷의 헤더는 transport layer와 Network layer를 거치며 source Ip, Port number 및 destination IP, Port number가 붙어 전송 과정에서 출처와 목적지를 식별 가능하다.

Demultiflexing 과 Multiflexing 은 Transport Layer 에서만 이루어지는 것이 아니라 각 계층별 여러 요소를 통해 이루어질 수 있는 것이다.

## Connectionless Demultiflexing - UDP

![](https://i.imgur.com/plHEsvf.png)

송신 호스트에서 datagram 전송을 요청하는 프로세스의 포트 번호가 포함된 소켓을 생성하여 해당 소켓을 통해 destination IP, Port number 가 담긴 datagram을 생성해 전송한다. 이후 해당 패킷의 헤더에 담긴 destination IP를 통해 서버 측 즉 수신 호스트의 Network Layer에서 datagram을 받고 Transport Layer에서 해당 datagram의 destination Port number를 확인하여 해당하는 소켓으로 전달하면 소켓을 통해 해당 애플리케이션 프로세스로 datagram이 전달된다.

여기서 알 수 있는건 어떤 datagram 이던 destination Port 만 같으면 전부 같은 socket으로 해당 datagram이 전달된다는 것이다.


## Connection-oriented Demultiflexing - TCP

![](https://i.imgur.com/56FAPlu.png)

connectionless demux 와 가장 큰 차이점은 connectionless 방식은 해당 데이터를 적절한 소켓으로 분배하는데 사용되는 요소가 destination port number 만 있었다면 connection oriented 방식은 source, destination 측 IP, Port number 모두를 사용해 결정한다는 점이다.

즉 데이터 세그먼트를 전송한 source client가 누구냐에 따라 서로 다른 socket을 사용한다. Non-persistent 방식의 HTTP 요청에서는 매번 socket이 달라질 수 있다.

먼저 3-way-handshakes 과정을 통해 송수신자간 연결을 설정하고 클라이언트(송신 호스트)는 소켓을 통해 데이터 스트림을 전송한다. 이 때 각 세그먼트는 순서가 할당된다. 이후 수신 호스트의 Transport Layer에서 데이터 스트림의 각 데이터 세그먼트의 헤더를 확인해 Port Number를 체크하고, 해당 포트번호와 연결을 맺은 프로세스가 누군지를 확인하여 적절한 socket에 해당 세그먼트를 순서에 맞게 재조립하여 전달한다.
![](https://i.imgur.com/DvIMe4V.png)
