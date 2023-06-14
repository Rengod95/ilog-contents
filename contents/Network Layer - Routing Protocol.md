---
title : Network Layer - Routing Protocol
date: 2023/06/14
author: '이인'
tags: ['Network Layer','Link State', 'Distance Vector', 'Djikstra', 'AS', 'OSPF']
---

# 라우팅 프로토콜

목적 : 송신 호스트부터 수신 호스트 까지 '좋은' 경로를 결정하는 것

여기서 말하는 경로란 source to destination 호스트 까지 이동하면서 거치는 모든 라우터 시퀀스를 나타낸다. 또한 '좋은'경로의 의미는 최소의 비용, 최소의 혼잡함, 가장 빠른 경로 등을 의미한다.


## 라우팅 알고리즘의 분류

정보:
	글로벌 : 모든 라우터가 완벽한 토폴로지와 링크 비용에 대한 정보를 가진다. 이 때는 link state 알고리즘을 사용한다
	분산화된 정보 (Decentralized) : 각 라우터는 물리적으로 이웃한 라우터들에 대한 정보와 해당 라우터까지의 링크 비용에 대한 정보만을 가진다.
	이웃 라우터와 정보 교환 및 비용 계산을 iterative 하게 수행하는 방식으로 Distance Vector 알고리즘을 사용한다.

Static : 경로는 시간의 변화에 따라 천천이 업데이트 된다.
Dynamic : 주기적인 업데이트와 링크 비용 변화에 대한 반응으로 경로는 더 빠르게 변화한다.


## Link State Algorithm

링크 스테이트 알고리즘은 모든 라우터가 전체 네트워크의 토폴로지 정보와 링크 비용을 알 수 있도록 설계 됨을 바탕으로 알고리즘이 작동한다. 이를 위해 라우터 간에 "링크 상태 브로드캐스트" 매커니즘이 사용된다.

여기서 말하는 토폴로지란, 네트워크의 물리적 혹은 논리적 구조를 나타내는 것으로, 네트워크를 구성하는 각 노드와 링크에 대한 정보를 포함한다. 

네트워크를 구성하는 모든 개별 라우터가 전체 네트워크 토폴로지 정보를 가진다는 것은 개별 라우터가 전체 네트워크의 모든 링크와 노드에 대한 정보를 가진다는 말이며, 독립적으로 라우팅 결정을 내릴 수 있다는 것을 의미한다.

링크 스테이트 알고리즘의 일환으로 다익스트라 알고리즘을 사용하는데, 이는 포워딩 테이블을 통해 하나의 노드에서 모든 다른 노드까지의 최소 비용의 경로를 계산한다.


C(x,y) : x 에서 y 노드 까지의 링크 코스트를 나타낸다. x와 y가 직접 연결되어있지 않다면 무한으로 표기한다.

D(v) : 소스 노드에서 목적지 노드 v까지의 경로의 현재 비용

P(v) : 소스 노드에서 목적지 노드 V까지 경로에 있어 v 직전의 노드(predecessor node)를 나타낸다.

N : 최소 비용이 알려진 노드의 집합

![](https://i.imgur.com/OSNe8aQ.png)
![](https://i.imgur.com/qAl9L8r.png)

해당 알고리즘의 복잡도는 n개의 노드에 대해, 각 노드마다 최소 비용이 알려진 노드를 제외한 모든 노드에 대해 비용을 계산하기 때문에 계산의 횟수는 N + (N-1) + (n-2) + ... + 1 즉 등차가 1인 등차수열의 합이다. 따라서 n(n+1)/2 번 만큼의 비용 계산을 진행하므로 시간복잡도는 O(n(n+1)/) == O(n^2) 이다.


## Distance Vector Algorithm

**Distance Vector 알고리즘의 핵심 아이디어**는 다음과 같다.

- 각 노드는 주기적으로 그의 DV 평가값을 이웃 노드들에게 전송합니다
- 특정 노드 x는 이웃 노드로부터 새로운 DV를 수신할 때마다, Bellman-Ford 방정식을 사용하여 자신의 거리 벡터를 업데이트합니다. Dx(y) ← minv{c(x,v) + Dv(y)} for each node y ∊ N

즉 각 라우터는 자신의 이웃 라우터에게만 DV 정보를 보내고, 받은 정보를 통해 라우팅 테이블을 업데이트한다.

Dx(y): 이는 노드 x로부터 노드 y까지의 최소 비용을 추정한 값. 

여기서 네트워크를 구성하는 개별 노드 x 는 이웃한 노드에 대한 링크 코스트 c(x,v) 정보를 가지며, 해당 이웃한 노드의 DV 값을 알고 있어야 한다.

결과적으로 노드 x 는 전체 네트워크 N을 구성하는 특정 노드 y 에 대한 Distance Vector 값을 유지한다. node x maintains Dx = [Dx(y): y є N ]



## Distance Vector 특징

1. Iterative, Asynchronous : 이웃한 링크 코스트 값의 변화 혹은 이웃 노드의 DV 값이 업데이트 되었다는 메세지를 받음에 따라 비동기적으로, 그리고 iterative 하게 알고리즘이 수행된다.
2. Distributed : 개별 노드는 오직 이웃한 노드에게만 자신의 DV 평가값이 변화함을 알린다.

![](https://i.imgur.com/p3HcvKp.png)
![](https://i.imgur.com/EDARWaK.png)

## Count to Infinity 문제

노드들이 서로의 비용 정보를 교환하면서, 실제 비용보다 낮은 비용을 계산하게 되는 현상을 "Count to Infinity" 문제라고 한다.


![](https://i.imgur.com/E70ePNJ.png)

x-y 링크 코스트가 60으로 변경되기 이전 Dz(x) = C(z,y) + Dy(x) = 1 + 4 = 5 이다.

x-y 링크 코스트가 60으로 변경된 후 y는 x에 대한 DV를 다시 평가한다. 이 시점에서 Y는 Dy(x) = min(C(y,x), C(y,z)+Dz(x)) 로 비교하는데, 해당 시점에서 Dz(x) 값은 5이다. 즉 Dy(x) = min(60, 1 + 5) = 6 으로 평가된다. 이는 변경된 코스트 값인 60이 제대로 반영되지 않은 더 낮은 비용을 계산하게 된다.

Dy(x) 값이 재 평가 되었으므로 이웃 라우터에게도 재평가 메세지를 전파한다. 이 때 라우터 Z가 구성하는 포워딩 테이블의 값도 재평가 되는데, 해당 시점에서 Dz(x) 또한 재평가 된다.

Dz(x) = min(C(z,x), C(z,y) + Dy(x)) 이고, 해당 시점에서 Dy(x) 값은 6 이므로 
Dz(x) = min(50, 1 + 6) = 7

즉 C(z,x)(50) < C(z,y)+Dy(x) 인 시점에서 Dz(x)가 제대로 된 값으로 평가되고,
Dz(x)가 제대로 평가된 시점에 Dy(x) 도 제대로 된 값으로 평가되게 된다.

즉 에 대하여 C(z,x)(50) < C(z,y)+Dy(x) === 50 < n+6(Dy(x)) 인 시점까지 n번의 불필요한 평가값 반복 계산이 이루어진다.

"poisoned reverse" 를 통해 해당 문제를 해결할 수 있다. Poisoned Reverse는 노드가 자신으로부터 정보를 받은 이웃노드로의 링크 코스트를 무한대로 설정하는 것이다.


https://code-lab1.tistory.com/35

## LS vs DV

1. **메시지 복잡성 (Message Complexity):**
    
    - LS: n개의 노드와 E개의 링크가 있을 때, O(nE) 개의 메시지가 전송됩니다. 이는 각 노드가 네트워크의 전체 상태에 대한 정보를 교환하기 때문입니다.
    - DV: 이웃 노드 간에만 정보를 교환합니다. 이로 인해 메시지의 수가 줄어들지만, 네트워크의 전체 상태에 대한 정보가 더 느리게 전파됩니다.
2. **탄력성 (Robustness):**
    
    - LS: 잘못된 링크 비용을 광고할 수 있다.
    
    - DV : 잘못된 경로 비용을 광고할 수 있다
3. **수렴 속도 (Speed of Convergence):**
    
    - LS: O(n^2) 알고리즘이 필요하며, O(nE) 개의 메시지가 필요합니다. 이로 인해 수렴하는 데 시간이 오래 걸릴 수 있습니다.
    - DV: 수렴 시간이 다양합니다. 라우팅 루프가 발생할 수 있으며, "Count-to-Infinity" 문제로 인해 수렴하는 데 시간이 더 오래 걸릴 수 있습니다.



# Intra-AS, Inter-AS Routing

여태 배웠던 라우팅 프로토콜의 문제점은 모든 라우터가 이상적으로 동작하고 네트워크가 flat하게 이루어짐을 가정하고 학습했던 것이다. 실질적으로 네트워크는 라우팅에 대해 다음과 같은 문제점이 존재한다.

1. Scale : 목적지가 수십억 개인 대규모 네트워크에서 단일 라우팅 테이블이 모든 destination에 대한 정보를 저장할 수 없다. 또 라우팅 테이블 교환만으로도 링크가 포화상태가 될 수 있다.
2. administrative autonomy : 네트워크들의 네트워크인 인터넷에서,각 네트워크 관리자는 자신의 네트워크 내에서 라우팅을 제어하고 싶을 수 있습니다.

때문에 이런 실질적인 문제들을 해결하기 위해 라우터들을 "자율 시스템" 혹은 '도메인' 단위의 영역으로 묶어버린다.

AS 를 구성하는 각 라우터들은 Inter-AS Routing 및 Intra-AS Routing 알고리즘 모두를 포함하는 포워딩 테이블을 가진다.

## Inter-AS 라우팅

AS들 간의 라우팅을 의미한다.

## Intra-AS 라우팅

동일한 AS (네트워크) 내의 호스트 및 라우터 간의 라우팅을 말한다. 동일한 AS 내의 모든 라우터들은 동일한 내부 도메인 프로토콜을 실행해야 한다. 다른 AS에 속한 라우터는 다른 내부 도메인 프로토콜을 실행할 수 있다.

AS내에서 다른 AS들과의 Link를 가지는 엣지 라우터를 Gateway Router (= Border Router)라고 한다.

Intra-AS 내에서 실행되는 라우팅 프로토콜을 IGP(Interior Gateway Protocol)이라고 한다.

주요한 IGP는 다음과 같다.

RIP : Routing Information Protocol
OSPF : Open Shortest Path First
IGRP : Interior Gateway Routing Protocol

## OSPF (Intra-As Routing)
![](https://i.imgur.com/DzIc7Pt.png)
OSPF 는 기본적으로 Link State 알고리즘을 사용한다. 전체 AS의 다른 모든 라우터에게 링크 상태를 광고한다. 이는 Network Layer 프로토콜이기에 IP를 통해 직접 OSPF 메세지가 전송된다. 그리고 모든 OSPF 메세지는 Authenticated된다.

OSPF 는 Area 개념을 사용하여 계층적으로 영역을 나누게 된다. 크게 Backbone과 Local Area로 계층이 구성된다. Link-state Advertisement는 area 내부에서 진행되며 개별 노드는 Area에 대한 토폴로지 정보를 가진다. 다른 영역에 대해서는 전체 AS에 대한 방향만 알고 있다.

Area Border Router : Area와 Backbone의 이음새가 되는 라우터
Backbone Router : 백본 범위에서 OSPF 라우팅을 진행
Boundary Router : AS 에서 다른 AS로 연결되는 라우터

![](https://i.imgur.com/xAqWlOC.png)

## BGP - Border Gateway Protocol (Inter As Routing)

BGP란 AS 간에 경로를 전파하고 결정하는 데 사용되는 Path vector기반 프로토콜이다.

IBGP : 동일한 AS 내의 라우터간 Reachability 정보를 전파하는 것이다.
EBGP : 이웃하는 AS들로부터  Subnet Reachability 정보를 주고받으며 유지하는 것이다.
![](https://i.imgur.com/mEecKSq.png)


BGP에서 'good' path 를 결정하는 기준은 Reachability 와 Policy 에 기반을 둔다.

BGP Session :  두 BGP 라우터 간 메세지를 주고받을 때 TCP connection을 사용한다. 즉 Application Protocol임을 알 수 있다. 메세지에는 Path에 대한 정보를 전송하기 때문에 Path Vector Protocol이라고도 부른다.

BGP에 들어있는 attribute는 Network Prefix + Attributes 로 이루어진다.
여기서 Attributes 는 다시 Next Hop + As-Path로 이루어진다.

AS-PATH : path vector 정보
NEXT-HOP : indicates specific internal-AS router to next-hop AS

Policy-Based Routing : AS 정책은 특정 path vector 정보를 다른 이웃하는 AS들에게 Advertise 할 지 말지 결정하는 기준이 된다. 또한 게이트웨이 라우터들은 들어오는 route advertisement에 대해 정책을 적용하여 해당 advertisement를 거절할지 수락할지 결정한다.


### BGP Message
BGP 메세지는 피어들간 TCP connection을 통해 메세지를 주고받는다.

OPEN : TCP 커넥션을 맺기 위한 메세지
UPDATE : 새로운 경로에 대해 Advertise 하기 위한 메세지
KEEPALIVE : 오랫동안 UPDATE 메세지가 부재하더라도, TCP 커넥션을 계속 유지하기 위한 메세지
NOTIFICATION : 이전 메세지에 오류가 있음을 알리는 메세지, 혹은 커넥션을 종료하기 위한 메세지


### Hot Potato Routin
목적지로 가기 위한 local gateway router가 여러가지라면, 다른 건 신경쓰지 않고 최소한의 intra-domain cost을 가지는 gateway router를 선택하는 라우팅 기법이다.


### BGP Route Selection

1. Policy 기반
2. shortest AS-PATH : 이왕이면 가장 짧은 경로로
3. closest NEXT-HOP Router : 가장 가까운 next hop router라는 말은 가장 최소의 비용이 드는 next-hop으로 라우팅 한다는 의미와 같다. - hot potato routing
4. additional criteria

## Why different Intra, Inter AS Routing?

Inter-AS즉 AS간 라우팅에서는 해당 AS 네트워크 관리자가 트래픽의 흐름과 라우팅을 제어하고 싶은 경우가 많기에 정책기반의 라우팅이 우선시 된다.

반면 Intra-AS는 정책을 필요로 하지 않고 오직 퍼포먼스를 중시한다.

Intra, Inter AS를 구분하는 라우팅 방식은 계층적 라우팅을 가능하게 하여 라우팅 테이블의 사이즈를 줄이고 업데이트를 위한 트래픽을 감소한다.





