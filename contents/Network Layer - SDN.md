---
title : Network Layer - SDN
date: 2023/06/14
author: '이인'
tags: ['Network Layer','SDN', 'OpenFlow']
---
# SDN - Software defined networking

기존의 라우터들은 Monolithic Router로 단일 라우터가 Controle Plane과 Data Plane을 모두 포함하여 하나의 닫힌 시스템으로 동작한다. 이런 모놀리식 구조의 문제는 전체 네트워크를 관리하거나 변경하려면 네트워크를 구성하는 각 장비를 개별적으로 관리해야 한다. 또 특정 벤더에 대한 의존성이 높아질 수 있다는 단점을 가진다.

SDN은 Monolithic 방식에서 Control Plane 과 Data Plane이 하나의 장비에서 닫힌 시스템으로 동작하던 방식을 소프트웨어로 제어 영역을 분리하는 방식을 통해 기존의 단점을 개선하는 기술이다.

## Pre-router  Control plane

SDN을 구현하기 위한 방법 중 하나로, 라우터의 Control Plane과 Data Plane을 명확히 분리해 라우터의 Control plane에서 라우팅 알고리즘을 수행하는 컴포넌트들이 작동하도록 하고, Data Plane은 단지 Control Plane이 만들어낸 포워딩 테이블을 바탕으로 패킷에 대한 포워딩만 수행하도록 한다.
![](https://i.imgur.com/juPJJdZ.png)

## Logically Centralized Control Plane - SDN

Pre-Router 방식과 더불어 SDN을 구현하기 위한 방법 중 하나이다. 네트워크를 구성하는 모든 라우터의 제어를 담당하는 중앙화된 제어 영역이 존재하는 방식이다. 중앙화된 제어 영역을 구현하기 위해 네트워크를 구성하는 모든 라우터와 연결된 Remote Controller가 존재하고, Remote Controller는 전체 네트워크의 전반적인 정보를 가지고 라우팅 정보를 계산하며, 각 라우터에게 포워딩 테이블 정보를 전달한다. 각 라우터는 Control Agent 인 CA가 존재해, CA 가 중앙 Remote Controller와 통신하여 포워딩 등에 대한 네트워크 정보를 받아온다. 
![](https://i.imgur.com/JspQEaG.png)


## logically centralized Control Plane 방식을 사용하는 이유

1. 더 쉬운 네트워크 관리 : 잘못된 라우터 구성을 피하고, 트래픽 흐름에 대한 유연성을 확보할 수 있다.
2. 테이블 기반 포워딩을 통해 라우터를 '프로그래밍'할 수 있다 : 개별 라우터를 프로그래밍하는 것은 각 라우터에서 구현된 라우팅 알고리즘의 결과를 의존하므로 더욱 어렵다. 반면 중앙화된 프로그래밍 방식은 테이블 계산과 정보의 분배가 더욱 쉽다. (ex : OpenFlow API)
3.  Control Plane Open Implementation : 열린 표준에 기반하여 Control Plane을 구현하므로 특정 장비에 종속되지 않는다.

### SDN 의 구성요소

1. 일반화된 플로우 기반 포워딩
2. Control, Data Plane 분리
3. Control Plane Functions
4. Programmable Control Applications


![](https://i.imgur.com/jjZmwMp.png)


## Control Applications

SDN 컨트롤러가 제공하는 하위 수준의 서비스와 API를 사용하여 실질적인 제어 기능을 구현한다. 중앙화된 제어 영역위에서 동작하므로 특정 SDN Controller나 벤더 등에 종속되지 않고 별개로 제공될 수 있다. 이를 Unbundled 되었다고 표현한다.
![](https://i.imgur.com/OevDoGp.png)

## SDN Controller
Control Plane 에서 실질적으로 Network OS 를 담당한다.
SDN Controller는 네트워크 상태정보를 유지한다. Control Application 과는 NorthBound API 를 통해 상호작용하며, Data Plane의 스위치들과는 SouthBound API를 통해 상호작용 한다.

SDN Controller는 Interface Layer, Network-wide State Management Layer, Communication Layer로 구성된다.
![](https://i.imgur.com/Qp3wn49.png)

logically centralized 되어있지만 실질적으로는 분산 시스템으로 구현된다.

![](https://i.imgur.com/jbnjqen.png)

## Data Plane Switches

1. 빠르고 단순하며 genralized forwarding이 구현되어있다.
2. 플로우 테이블은 controller에 의해 계산되고 설치된다.
3. 컨트롤러와의 통신을 위한 인터페이스로 OpenFlow API 등을 사용한다.



## OpenFlow Control

TCP를 사용하며 controller와 switch 사이의 통신을 위한 프로토콜이다. 보안을 위해 TLS를 사용할 수 있다.

### Controller To switch Messages

- Features : 컨트롤러가 스위치에 대한 정보를 요청하는 것
- Configure : 컨트롤러가 스위치의 configuration을 세팅하는 메시지
- Modify-state : 네트워크 상태에 변화가 있을 때 플로우 테이블의 엔트리를 수정, 추가, 삭제 하기 위한 메세지
- Packet-out : 컨트롤러가 스위치 포트 정보 없이 직접 패킷을 보낼 수 있는 메세지

### Switch To Controller Message

- packet-in : 패킷 전송, packet-out 메세지를 볼 수 있음
- flow-removed : 플로우 테이블 엔트리가 삭제된 경우 알리는 메세지
- port status : 포트의 변화에 대한 정보를 컨트롤러에게 알리는 것

- 네트워크 관리자가 직접적 OpenFlow 메세지를 통해 스위치의 플로우 테이블을 수정할 수 없다. OpenFlow Controller를 거치지 않고 직접 수정하게 될 경우, controller와 switch 간의 데이터 싱크가 맞지 않는 상황이 생길 수 있다. 따라서 더 높은 추상화 레벨의 Openflow Controller를 통해야만 가능하다.

## Data Plane Abstraction(스위치)

Match and Action : packet header 필드의 값을 통해 매칭되는 패턴을 파악하고 매칭에 대한 특정한 액션을 수행한다.

![](https://i.imgur.com/Czpi1nZ.png)


Router :
	Match : IP Longest prefix match
	action : forward
Switch : 
	match : destination Mac address
	action : forward
Firewall : 
	match: IP address, TCP/UDP Port numbers
	action : permit, deny
 NAT :
	 match : IP address, Port
	 action : rewrite address and port
