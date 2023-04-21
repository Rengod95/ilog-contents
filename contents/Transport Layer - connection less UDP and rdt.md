---
title : Transport Layer - connection less UDP and rdt
date: 2023/04/21
tags: ["UDP",'Transport Layer', 'connection less', 'checksum',"rdt", ' reliable data transfer', 'selective repeat', 'go-back-n']
author: 이인
---

# UDP

## User Datagram Protocol

- No frills, Bare bones : 최소한의 기능 제공을 목표로 한다
- Lost and out-of-order : 데이터가 송신 과정 중 손실되거나 순서를 보장하지 않는다
- connection less : no handshaking, 딜레이 최소화
- small header size : 패킷 자체의 용량 감소 및 속도 상승
- DNS, Stremaing, SNMP, RIP 등에서 사용됨
- Reliable transfer over UDP : Appliacation Layer의 책임이다.

## UDP Header

![](https://i.imgur.com/6vEVw9W.png)

- Length : 헤더 부터 페이로드의 모든 데이터를 다 더한 것의 길이를 byte 형태로 나타냄
- Checksum : 패킷의 송신 과정 중 특정 비트의 손실이 있는지를 판단하는 매개체
	- 헤더를 포함한 세그먼트 내용을 연속된 16비트 정수로 다 더한다.
	- 해당 값에 1의 보수인 값을 checksum에 넣는다.
	- receiver 는 패킷 데이터의 합 + checksum 비트가 모두 1이 되는지 검사하여 해당 패킷이 손상되었는지 파악할 수 있다.
	![](https://i.imgur.com/PNB6voM.png)
	![](https://i.imgur.com/9DWWgFy.png)


# Reliable Data Transfer (rdt)

## Why need it?

Application Layer의 프로세스 입장에선 reliable 한 데이터 전송을 보장받기 위해 Transport Layer 의 프로토콜을 사용한다. 이론상 transport layer는 송신 호스트랑 수신 호스트간 reliable channel 을 보장해 주어야 하는데 현실은 그렇지 않다. 많은 요소로 인해 패킷이 전송되는 과정에서 손실이 발생할 수 있다는 사실은 필연적이다.

따라서 우리는 완벽한 데이터 송수신을 목적으로 하는 것이 아니라, 송수신 과정에서 발생한 유실에 대해 어떻게 대처할 것인지 고민해야 한다.

data transfer 과정에서 발생할 수 있는 에러와 솔루션을 간략하게 나타내면 다음과 같다.

- Corruption(Bit-error)
	- Reciever는 segment header의 checksum 을 사용해 비트 에러가 발생했는지 판단한다.
	- 정상이라면 ACK를, 비정상이라면 NAK를 송신측에 전송하여 송신자가 제대로 전달 되었는지 확인할 수 있다.
- Packet Loss(패킷 자체의 유실)
	- 송신측에서 Timer를 사용하여 Time out 이벤트에 대한 처리를 한다.
		- ex : 재전송
	- Seq # 사용 : 순서 보장과 중복을 방지할 수 있다.


다음은 실질적으로 transport layer가 제공해야 하는 서비스와, 실제 서비스의 구현을 비교하는 사진이다.
![](https://i.imgur.com/Fns3VY2.png)


## rdt 1.0

![](https://i.imgur.com/SpOgmvv.png)


rdt 1.0 은 데이터의 실질적 송수신을 담당하는 underlying channel 이 완벽하게 reliable 하다는 가정에서 설계되었다. 즉 외부 환경으로 인한 패킷 유실이나 비트 에러등이 전혀 발생하지 않고, 항상 패킷이 순서대로 도착한다고 가정하에 데이터 송수신을 처리한다.

![](https://i.imgur.com/Uj2U6x1.png)


## rdt 2.0

rdt 2.0은 underlying channel에서 비트 에러가 발생할 수 있다는 가정이 추가되었다. 유실은 안되지만 error가 발생 할 수 있다고 생각한다.

때문에 체크섬을 통해 sender가 전송한 패킷에 에러가 있는지를 판단하여, 에러의 유무에 따라 reciever는 ACK 혹은 NAK 패킷을 응답으로 전송하여 sender에게 알린다.

sender는 NAK에 대해 이전 패킷에 대한 재 전송을 실시하게 된다.

![](https://i.imgur.com/ULIYUU0.png)

Stop-and-Wait 방식 때문에 sender가 받는 receiver의 NAK 혹은 ACK 패킷은 항상 직전에 자신이 전송한 패킷에 대한 피드백이라는 걸 보장받을 수 있다.

그러나 반대로 생각해보면 receiver가 보낸 NAK 혹은 ACK가 sender 측으로 전달되는 과정에서 비트 에러가 발생했을 때 송신측은 무조건 NAK이라는 가정하에 다시 패킷을 전송한다. 이 때 receiver는 받은 패킷이 NAK 에 대한 응답으로 재전송한 패킷 인지, 다음 순서의 패킷을 보낸건지 알 수 없다.

## rdt 2.1

그래서 이런 중복 패킷 문제를 해결하기 위해 sender는 각 패킷에 seq # 를 추가하여 개선할 수 있다. seq # 를 통해 reciever는 중복된 패킷을 폐기한다.

![](https://i.imgur.com/8A7quTP.png)
![](https://i.imgur.com/DCDSwTG.png)
![](https://i.imgur.com/Omym9Ld.png)
![](https://i.imgur.com/1wVdihb.png)
![](https://i.imgur.com/hqkQdQr.png)


### 두 개의 seq # 만으로 2.1 방식이 가능한 이유

바로 stop-and-wait 방식으로 동작하기 때문이다. stop-and-wait 방식이 아닐 경우, sender 가 0번째 패킷을 보낸 이후 해당 패킷에 대한 ACK 혹은 NAK 이 올 때 까지 기다리지 않고 계속 패킷을 보낸다. 그러면 ACK 를 받아야하는 sender 입장에서나, ACK를 보내야 하는 receiver 입장에서나 현재 자기가 어떤 패킷에 대한 ACK를 송수신하는 지 알 수 없다.


## rdt 2.2

rdt 2.2는 2.1과 동일하지만 NAK 대신 오로지 ACK만 사용하는 방식이다. sender는 중복된 ACK 에 대해 현재 패킷을 재전송한다.

![](https://i.imgur.com/phOGzX3.png)


## rdt 3.0

패킷의 송수신 과정에서 비트에러 뿐만 아니라 패킷 자체가 유실될 수 있는 상황도 고려한다. 이를 위해 sender는 패킷 전송 시 매 패킷에 대한 timer를 사용한다.

그러나 여전이 stop and wait 방식을 기반으로 작동한다.
![](https://i.imgur.com/APx3ZVU.png)
![](https://i.imgur.com/iYUk7SO.png)
![](https://i.imgur.com/VzqPknN.png)
![](https://i.imgur.com/3KBpdLu.png)

stop and wait 방식의 가장 큰 단점은 송신자가 통신 시간에서 패킷 전송에 실제로 사용하는 시간의 비율 즉 이용률이 낮다는 것이다.

1. 전송률 (R) = 1Gbps
2. 전파 지연 (prop delay) = 15ms
3. 패킷 크기 (L) = 8000bits 
4. RTT = prop delay * 2 = 15ms * 2 = 30ms
5. Dtrans = 전송 지연 시간 = L / R = 8000/10^9 bit = 8 microsec
6. Usender = 송신자의 이용률 = (L / R) / (RTT + L / R) =  8msecs / 30 + 0.08 msec = 0.08 / 30.008 ms = 약 0.00027
7. 1kb 패킷이 대략 30msec 마다 하나 씩 처리될 때,
8. 초당 처리량 : 8000 bits / 30.008 ms = 266 kbps (킬로비트/초)

![](https://i.imgur.com/8i43l0O.png)


링크 대역폭을 최대한 활용하지 못하며, 이로 인해 전체 성능이 저하된다는 것을 보여준다. 따라서 한 번에 하나의 패킷을 보내고 기다리는 stop and wait 방식 대신 pipelining 방식의 프로토콜을 사용하여 사용률을 높여볼 수 있다.
![](https://i.imgur.com/sJVNyj1.png)


pipelining 을 사용하는 대표적인 프로토콜은 go-Back-N, selective repeat 이 있다.


## go-Back-N

Sliding window 기반의 프로토콜이다. sender는 윈도우 내의 패킷을 순차적으로 전송할 수 있으며, 그 중 가장 오래된 unacked 패킷에 대해 타이머를 설정한다.
또 Cumulative ACK 방식으로 수신자가 받은 패킷들 중 가장 최근 연속적인 패킷의 번호를 송신자에게 알린다.

예로 송신자가 순차적으로 0,1,2,3 패킷을 보냈을 때 2번 패킷이 유실되고 0,1,3 번 패킷만 수신자에게 전달되었다면, 3번 패킷을 전달받은 시점에 수신자는 해당 패킷을 폐기하고, 가장 마지막으로 전달받은 패킷인 1번에 대한 ACK를 재전송한다.

패킷은 다음 4가지 상태로 나뉜다.
1. alreadt acked : 송신에 대해 유실이나 에러 없이 정상적으로 ack를 전달받은 패킷이다.
2. sent, not yet acked : 송신자가 순차적으로 보낸 패킷이지만, 아직 수신자로부터 ACK를 전달받지 못한 패킷을 의미한다
3. usable not yet sent :  윈도우의 N개의 공간 중 sent not yet acked 패킷을 제외한 나머지 패킷을 의미한다.
	윈도우 사이즈를 N이라고 가정할 때,  Usable not yet sent size = base + N -1 - (nextseqeuncenum -1) = base + N - nextSequenceNum
4. not usable : 아직 윈도우가 도달하지 못한 부분의 패킷이다.

![](https://i.imgur.com/DIVmD2B.png)
![](https://i.imgur.com/LQcg2kq.png)
![](https://i.imgur.com/AmT4AOJ.png)

## Selecive repeat

패킷 유실이 발생했을 때 오직 손실된 패킷만 재전송한다.

1.  개별 패킷에 대한 ACK: 수신자는 패킷을 개별적으로 확인하며 순서에 관계없이 각 패킷에 대한 ACK를 보냅니다. 이를 통해 송신자는 각 패킷의 전송 상태를 정확하게 알 수 있다.
2. 개별 타이머 설정: 송신자는 각 패킷에 대해 개별 타이머를 설정합니다. 이를 통해 각 패킷의 전송 상태를 독립적으로 추적할 수 있습니다. 각 패킷에 대한 ACK를 받으면 해당 패킷의 타이머를 중지합니다.
3. 손실된 패킷만 재전송: 패킷 유실이 발생한 경우, 송신자는 해당 패킷만 재전송한다. 이와 달리 GBN에서는 손실된 패킷 이후에 전송된 모든 패킷을 재전송한다.
4. 수신 버퍼 사용: 수신자는 패킷을 순서대로 처리하기 위한 버퍼를 사용한다. 패킷이 도착한 순서와 상관없이 버퍼에 저장한 다음, 순서에 맞는 연속적인 패킷을 상위 계층으로 전달하게 된다. GBN과 달리 순차적으로 패킷이 도착하지 않았을 때 폐기하는 것이 아니라 버퍼에 저장하게 된다.

![](https://i.imgur.com/ATfb4ke.png)
![](https://i.imgur.com/WmPl762.png)
![](https://i.imgur.com/pip8RVH.png)
![](https://i.imgur.com/E7G1bwk.png)
![](https://i.imgur.com/Lr0xk8C.png)




