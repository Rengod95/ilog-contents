---
title : Link Layer - 기본
date: 2023/06/14
author: '이인'
tags: ['Link Layer', 'MAC Protocols']
---
# Link Layer 

인접한 노드간 연결되는 통신 채널을 링크라고 한다. host 및 라우터를 모두 노드라고 하며 DTU는 Frame이다.

데이터 링크 레이어는 물리적으로 인접한 노드에게 링크를 통해 데이터그램을 전송하는 책임을 가진다.

Ethernet, 802.11 등의 링크 프로토콜을 통해 전송하며 각 링크 프로토콜은 서로 다른 서비스를 제공한다. Reliable Data Transfer를 제공하지 않을 수도 있다.

## Link Layer services

1. Framing : datagram을 encapsulation 해서 Frame으로 만든다. 이 때 header와 더불어 trailer도 붙는다. 출발지 목적지를 식별하기 위해 MAC 주소를 사용한다.
2. Reliable Delivery between adjacent nodes 
3. Flow Control
4. Error Detection
5. Error Correction
6. Half-duplex, Full Duplex : half는 메세지를 보낼 때와 받을 때 동시에 불가능, full은 동시에 수신과 발신이 가능하다

## Link Layer Implementation

링크 레이어는 모든 호스트 각각에 구현되며 정확히는 Network Interface Card 일명 NIC 에 구현된다.(어댑터) 혹은 링크 프로토콜에 따라 이더넷 카드, 802.11 카드 등에 구현되기도 한다.

NIC는 호스트의 시스템 버스와 연결되어있다.
![](https://i.imgur.com/EdajoEC.png)

![](https://i.imgur.com/h1qz9Gq.png)

CC000 에서 정보를 읽어 AA55라는 시그니쳐가 있으면 롬이 있다고 인식해 CC002로 이동해 초기화를 진행

![](https://i.imgur.com/qFGbjeT.png)


## Error Detection

EDC =  오직 에러 디텍션 및 커렉션을 위한 비트 (redundancy)
D =  헤더를 포함한 에러 체킹의 범위 안에 있는 데이터
![](https://i.imgur.com/b51obTL.png)
에러 디텍션은 100% 믿을 수 없다. 극히 드물게 프로토콜이 감지하지 못하는 에러가 발생할 수 있다. EDC 필드가 클 수록 에러 디텍션과 커렉션에 더 도움을 준다.

### Parity Checking

Single Bit Parity : D bit의 1의 개수가 짝수면 0 홀수면 1(변동가능) => 에러 디텍션 범위가 너무 적음

two-dimensional bit parity : 
![](https://i.imgur.com/3Z5igHy.png)

### Cyclic redundancy check
burst error에도 더 강력하게 에러 디텍션 하기 위한 방법

![](https://i.imgur.com/zQ8lSaN.png)
![](https://i.imgur.com/t2fbS6I.png)

# Multiple access links, protocols

## Link type

1. Point to Point : 1 대 1, Ethernet Switch 등
2. Broadcast : 공유된 와이어를 사용, 802.11 wireless LAN 등

## Multiple Access Protocols

하나의 공유되는 브로드캐스트 채널을 사용하며, 노드간 둘 이상의 동시 다발적인 전송으로 인해 Interference가 발생한다.

각 노드가 채널을 공유하기 위한 분산화된 알고리즘을 사용한다. 채널 자체만을 사용하여 통신하고 채널 외부의 어떤 요소와 협력하지 않는다.

이상적인 상태: 해당 브로드 캐스트 채널의 속도가 R bps일 떄, 하나의 노드만 전송한다면 R 속도로 전송할 수 있으며, M개의 노드에 대해서는 R/M 만큼의 평균 속도로 전송하고, 완전히 분산화된 구조

## MAC protocol : taxonomy

MAC 프로토콜을 다음과 같은 기준으로 분류할 수 있다.

1. Channel Partioning : 채널을 여러개의 작은 조각으로 나눈다. 각 조각은 개별 노드에게만 할당되고 해당 노드 이외에는 사용하지 못하도록 하는 방식
2. Random Access : 모두가 채널에 무작위 접근이 가능, 충돌을 허용
3. Taking Turns : 각 노드가 채널을 사용하는 턴을 가지는 방식, 현재 턴을 가진 노드가 해당 채널을 사용하고 턴이 끝나면 다음 노드에게 턴이 넘어가는 방식

## Channel Partioning : TDMA - Time Division Multiple Access

하나의 채널을 여러개의 'round'로 나누어 엑세스하며, 개별 라운드는 fixed length slot 을 가진다. 여기서 length 는 패킷 전송 시간을 의미한다. 사용하지 않는 슬랏은 idle 상태에 돌입한다.

![](https://i.imgur.com/4HVnhxT.png)

## Channel Partioning : FDMA : Frequency division multiple access

채널 스펙트럼을 주파수를 기준으로 나눈다. 개별 스테이션은 고정된 고정 주파수 대역폭을 가진다.

![](https://i.imgur.com/mIAIC97.png)



## Random Access Protocol - Slotted ALOHA

가정 :

1. 모든 프레임 사이즈는 동일하다
2. 시간은 동일한 크기의 슬랏들로 나뉘어지고 노드는 슬랏이 시작될 때 전송을 시작할 수 있다.
3. 노드들은 모두 동기화된 시간을 사용한다고 가정
4. 둘 이상의 노드가 동일 슬랏에서 전송한다면 모든 노드는 충돌을 감지하게 된다.

동작 :
1. 충돌 미발생 :  다음 슬랏으로 노드는 새로운 프레임을 전송할 수 있다.
2. 충돌 발생 : 노드는 확률에 의거해 성공할 때 까지, 이어지는 슬랏에서 프레임을 재전송한다.

![](https://i.imgur.com/1QzjWNG.png)
C : Collision, E : Error, S : Success

장점 : 
1. 하나의 노드가 최대의 속도로 전송할 수 있다.
2. 매우 분산화되어 있어서 단순하다.

단점 :
1. 충돌 발생시 낭비하는 슬랏이 생기고, idle 슬랏도 생긴다.
2. 시간 동기화가 필요하다.

###  Random Access Protocol - PURE ALOHA (unslotted)

Slotted Aloha 방식은 슬릇을 활용하므로 슬랏 마다의 컬리전을 체크하면 되었지만, Pure Aloha는 T0 시점에서 크기 1 만큼의 프레임을 전송할 때, T0-1 <= X <= T0+1 사이 시점 모두에서 충돌이 발생할 수 있으므로 충돌 발생 확률이 두배이다.
![](https://i.imgur.com/cA1QTs1.png)
![](https://i.imgur.com/DGltZXa.png)
![](https://i.imgur.com/C4cNlr8.png)

## Random Access Protocol - CSMA - Carrier sense multiple Access (Listen Before Transmit)

전송이전 채널의 상태를 감지하는 것을 기반으로 채널이 대기 상태이면 전체 프레임을 전송하고 만약 채널이 busy 상태라면 전송을 연기하는 것이 기본 동작이다.

그러나 채널의 상태를 감지한다고 에러가 발생하지 않는 것은 아니다. 캐리어가 전파되는 딜레이가 있기에 동일 채널을 사용하는 서로 다른 두 노드가 서로의 전송 상태를 파악하기 까지의 딜레이로 채널이 대기상태인 것으로 착각하여 에러가 발생할 수 있다.
![](https://i.imgur.com/THqHf2a.png)
이를 개선하기 위한 프로토콜이 CSMA/CD(Collision Detection)이다.

## CSMA/CD
충돌이 발생하면 충돌이 발생한 전송을 바로 중단한다. 이를 통해 채널이 낭비되는 것을 줄인다.

Wired LAN 에서는 신호강도를 측정하여 쉽게 충돌을 감지할 수 있지만 Wireless LAN은 충돌 감지가 힘들다.

![](https://i.imgur.com/R34TdkG.png)

알고리즘

1. NIC 가 데이터그램을 네트워크 계층으로부터 받아와 프레임을 만든다.
2. NIC가 채널의 상태가 대기 상태임을 감지하면 해당 프레임을 전송한다. 만약 busy 상태라면 대기상태가 될 때 까지 기다리다가 전송한다.
3. NIC가 전체 프레임을 보내는 과정에서 다른 전송을 발견하지 않았다면 NIC는 프레임 전송을 정상적으로 끝냈다.
4. 만약 전체 프레임을 보내는 과정에서 다른 전송이 발견되었다면 해당 전송을 취소하고 jam signal을 전송한다.
5. 전송을 취소한 뒤 NIC는 binary exponential backoff 작업을 수행한다. binary backoff는 m번째 충돌에 대해서 0~2^m-1 까지의 수의 집합 중 임의의 수 k를 선택하여 K * 512 bit 시간만큼 기다리다가 스탭 2로 넘어가는 작업이다.

## Taking Turns MAC protocols

Channel Partitioning MAC protocol 들은 채널을 효율적이고 공평하게 분배하므로 부하가 높은 경우에 유리하다.

random access MAC protocol의 경우 하나의 노드가 전체 채널을 사용할 수 있기에 부하가 높은 경우 충돌에 대한 오버헤드가 크므로 부하가 낮은 경우 유리하다.

Taking turns 프로토콜은 두 경우 모두에서 베스트이다.

### Polling 

마스터 노드가 노예 노드들에게 토큰을 주어 전송 턴을 결정한다. 폴링 오버헤드나 지연, single point of failure 문제등을 고려해야 한다.
![](https://i.imgur.com/h9bNRdt.png)

### Token Passing (Token ring)

마스터 노드가 존재하지 않고 토큰이 노드에서 다음 노드로 순차적으로 넘어간다. 토큰 오버헤드, 지연, single point failure 문제를 고려해야 한다.

