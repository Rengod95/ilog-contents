---
title : Application Layer - Web and HTTP
date: 2023/04/19
tags: ['네트워크', '컴퓨터 네트워크', 'Application Layer', 'HTTP', 'TCP', 'Web']
---

# HTTP와 Web

## Overview

우리가 흔히 말하는 웹사이트는 모두 객체들의 합으로 구성된다. 여기서 말하는 객체는 이미지 혹은 HTML 등을 말하는 것이다. 그러나 기본적으로는 HTML File을 통해 다른 형태의 오브젝트들을 참조할 수 있는 것이다.

User -> HTML -> Image, Audio 등

그리고 이런 웹을 표현하는 모든 객체들은 주소를 가질 수 있다. 바로 URL을 통해서.
예로 www.example.com/exam/?post=1 과 같은 URL이 존재할 때,
www.example.com 은 Host name 이라고 하며, 뒤에 따라오는 exam/?post=1 은 pathname이라고 한다.

 **URL = host name + pathname**

HTTP는 이런 웹을 위한 Application 계층의 프로토콜이다. 즉 일반적으로 우리가 웹 브라우저를 통해 접속하는 대부분의 웹 사이트는 알게모르게 HTTP 프로토콜 기반의 통신을 통해 오브젝트를 서버로 부터 전달받아 화면에서 볼 수 있는 것이다.

## HTTP Protocol 의 특징

- Stateless : 서버는 이전의 클라이언트 요청에 대한 정보를 유지하지 않는다. 각 HTTP Request는 독립적이다.
- Connection 방식 :
	- non-persistent : 하나의 connection 당 하나의 오브젝트만 전달할 수 있다. 여러 개의 오브젝트가 필요하다면 오브젝트의 갯수에 맞는 커넥션을 맺어야 한다.
		=> 하나의 오브젝트만을 주고 받기 위해 커넥션을 생성하는 건 오버헤드가 크다. 효율성을 위해 하나의 커넥션을 재사용하려고 한다. => persistent connection
	- persistent : 하나의 커넥션으로 여러 개의 오브젝트를 전달할 수 있다.

## Non-Persistent Connection
TCP Transport Protocol(Non-persistent) 을 사용한다는 가정 하에, 일반적인 HTTP 통신의 과정을 요약해서 살펴보면 다음과 같다.

1-a. Client가 HTTP server와 TCP connection을 맺는다. 포트는 80
1-b. HTTP server는 80 포트의 TCP 커넥션 요청을 수용하고 클라이언트에게 알린다. (3-way- hanbshakes)
2. client는 알림을 받고 HTTP Request Message를 Tcp connecntion socket으로 전송한다.
3. server는 요청 메세지를 받아 클라이언트가 원하는 오브젝트를 담은 response message를 자신의 socket에 전달한다.
4. server는 TCP 커넥션을 닫는다.
5. 클라이언트가 서버로부터 오브젝트를 전달받는다.

필요한 오브젝트의 수(N)에 따라 해당 과정을 N번 반복한다.
![](https://i.imgur.com/QJmibgQ.png)
![](https://i.imgur.com/M6TVcbZ.png)

## Round Trip Time(=RTT) 을 통해 Response Time 구해보기
성능을 측정하는 지표 중 하나는 HTTP Response Time이다. 클라이언트가 특정 오브젝트를 요청했을 때, 서버로부터 원하는 오브젝트를 전달받기 까지 얼마의 시간이 걸렸는가를 의미한다.

이를 구하기 위해서 먼저 RTT의 개념을 알아야 한다. RTT는 클라이언트가 보낸 패킷이 서버로 갔다가 다시 클라이언트에게 되돌아오기까지 걸린 시간을 말한다.

Non-persistent connection 방식에서는 Response Time을 계산할 때 크게 두가지 요소를 고려하면 된다.

앞서 살펴본 통신 과정을 통해 connection을 맺기위한 RTT, 그리고 실제 오브젝트 요청에 대한 RTT 총 2RTT와 File Transmission Time을 합하면 된다.

식으로 정리하면 다음과 같다.

**Non-persistent TCP Response Time = 2RTT + File Transmission Time**

여기서 File Transmission Time은 왜 고려해야 하는 것일까?

RTT의 뜻을 다시 생각해보자. RTT는 클라이언트의 요청에 대한 응답을 받기 까지 걸린 시간이다. 즉 클라이언트의 입장에서 자신이 보낸 HTTP Request가 서버를 거쳐 response를 받기 까지 걸린 시간을 의미한다.

서버의 입장에서 보았을 때, 클라이언트가 요청한 오브젝트를 전달하기 위해서 파일 전송 시간 자체도 고려해야 한다. RTT와는 별개로 클라이언트가 요청한 오브젝트의 크기와 네트워크 대역폭에 따라 클라이언트에게 응답을 보내기 위해 파일 데이터를 전송하는 시간 또한 염두해야 한다.

## Persistent Connection

Persistent Connection 방식은 맨 처음 client-server 간 커넥션을 한번 생성하고 나면, N개의 오브젝트 요청에 대해 이미 생성해 놓은 커넥션을 재사용할 수 있다는 것이다.

Non-persistent Connection 방식의 response time은 N개의 오브젝트 요청에 대해 N * (2RTT + File Transfer Time) 인 반면에, Persistent Connection의 response time은 N개의 오브젝트 요청에 대해서 **RTT + N * (RTT + File transferTime)** 으로 계산할 수 있다.

## Persistent vs Non-Persistent 요약
![](https://i.imgur.com/jlakYHS.png)
### Persistent Connection
- **Response Time = RTT(for connection) + N * (RTT + File Transfer time)**
- 매 request마다 최초 생성된 커넥션을 재사용

### Non-Persistent Connection
- **Response Time = N * (2 * RTT + File Transfer Time)**
- 매 request마다 새로운 커넥션을 생성
- 비효율성 때문에 병렬 커넥션 생성을 허용하기도 하지만, 네트워크 전체 입장에서 다른 클라이언트에게 피해를 준다.



## HTTP Request Message 살펴보기
![](https://i.imgur.com/7KXHyhQ.png)
메세지의 줄넘김을 할 때, /r/n 사용한다.
- /r (CR): carriage return
	- 타자기 시대의 잔재, 타자기의 캐리지를 원래 위치로 돌리기 위한 제어 문자
	- 커서를 현재 줄의 시작위치로 이동 시킴
- /n (LF): line feed
	- 다음 줄로 개행하는 동작을 의미, 새로운 줄로 이동하기 위해 종이를 한칸 위로 올리기 위한 제어 문자
	- 커서를 다음 줄로 이동 시킴
### Request Line
- method : GET,POST,DELETE,PUT 등의 어떤 행동의 요청인지를 명시한다.
- sp : space ,빈칸이다
- URL : 요청하고자 하는 오브젝트의 주소를 명시한다.(ex : /index.html)
- Version : HTTP 프로토콜의 버전
### Hedaer Lines
HTTP 요청에 부가적으로 필요한 메타 정보들을 넣는 공간이다.
name : value 형태로 삽입한다.

- header field name : Host, user-agent 등의 헤더 필드 이름이 들어간다
- value : 각 필드 이름에 맞는 값을 넣는다.
- CR;IF : HTTP Request Message의 헤더 부분이 끝났음을 의미한다. 아무 글자 없이 오로지 CRIF만 입력되었을 때 

## HTTP Response Message 살펴보기
![](https://i.imgur.com/8LoLrD1.png)


## Cookies overview

기본적으로 HTTP Protocol은 stateless 한 특성을 가진다. 이전의 요청이 현재의 요청에 영향을 줄 수 없다. 그러나 이러한 상태정보를 유지해야하는 경우들이 있다. 쇼핑몰의 장바구니, 사용자 인증 정보, 개인화된 콘텐츠 등

이런 상충되는 상황을 해결하기 위해 쿠키가 도입되었다. 클라이언트 측 호스트에 저장되는 텍스트 파일로 사용자의 상태 정보를 저장하고 HTTP Request 및 Response 에서 사용할 수 있다.

쿠키가 없는 최초의 HTTP Request를 클라이언트가 발송하면 그에 대한 response로 서버에서 response header에 쿠키 정보를 넣어 전달하면, 클라이언트가 해당 쿠키정보를 저장한 후, 이후의 요청에 대해 request header에 쿠키 정보를 포함하여 이런 상태정보를 활용한 HTTP Request & Response를 수행할 수 있다.

보통 이런 쿠키정보에 대한 관리는 클라이언트 측의 브라우저가 진행한다. 해당 진행과정을 메세지 차트로 만들면 다음과 같다.

![](https://i.imgur.com/Ct4bumw.png)

## Web Cache Overview
클라이언트와 오리진 서버 사이에 프록시 서버를 배치하여 해당 프록시 서버가 클라이언트의 요청을 받아 라우팅한다.

- 절차 : 
	1. 동일한 오브젝트 요청에 대해 매번 origin server에서 받아오는 것이 아니라, 캐시에 해당 오브젝트를 저장해 두고, 이후 동일한 오브젝트 요청에 대해 해당 캐시 오브젝트를 반환한다.
	2. 만약 캐시에 존재하지 않는 오브젝트를 클라이언트가 요청했을 경우, 프록시 서버(캐시)가 origin server에 해당 오브젝트를 직접 요청하여 받아오고, 클라이언트에게 반환한다.
- 목적 :
	1. 클라이언트의 입장에서 response time이 매우 줄어든다. 복잡한 쿼리를 수행하는 요청일수록 효과가 극대화 된다.
	2. origin server로 향하는 트래픽을 줄일 수 있다.


## No Cache vs Cache 비교해보기

- 가정
	- 평균 오브젝트 크기 : 1Mb
	- 평균 요청율(browser to server) : 15/sec, 1초에 대략 15개의 요청이 server로 전달된다는 의미
	- Router to Origin server RTT : 2sec,
	- access link rate : 15.4Mb/s
		access link 란 두 개의 서로 다른 네트워크 구성요소 (ex: router, hub, switch) 사이에 연결되어있는 통신 경로이다. 일반적으로 ISP 와 개인 사용자의 네트워크 사이의 연결을 나타낸다. 즉 access link를 통해 Public Internet에 연결 된다.
- 구해야 하는 것
	- LAN Utilization (LAN의 활성정도)
	- access link utilization (Access Link의 활성정도)
	- Total Delay

LAN Utilization 은 LAN의 평균 대역폭과 평균 요청율을 통해 구할 수 있다. 평균 대역폭은 1초 단위에서의 속도이며, 마찬가지로 평균 요청율도 1초 단위에서의 요청량이다.

즉 1초에 15개의 1Mb 오브젝트를 요청하기 때문에, **LAN Utilization = 15/100 = 15%**

Access Link Utilization도 마찬가지 이다. 이는 Access Link rate와 초당 요청 데이터의 평균을 통해 구할 수 있다.

**초당 평균 데이터 요청량 = 1Mb * 15 = 15Mb
Access Link Rate = 15.4Mb/s**
**Access Link Utilization = 15/15.4 = 97% (over 80%)**

Total Delay = RTT + Access Delay + LAN Delay
RTT = 2sec
Access Delay 는 access link utilization 이 80%를 넘어가기 때매 무수히 크다. minutes로 가정
	LAN Delay 는 매우 작기 때문에 usecs 로 가정

**Total Delay = 2 sec + minutes + usecs**

특히 Access Link는 가격이 매우 비싸기 때문에 Access Link Utilization을 낮추어야 한다.
이때 local cache server를 통해 이를 해결해볼 수 있다.

local web cache를 사용한다고 가정했을 때 앞선 과정을 다시 수행해보자.

조건은 고정이다. 다만 hit rate라는 조건을 추가해야 한다. hit rate란 클라이언트가 요청한 오브젝트가 캐시에 존재할 확률이다.
- 가정
	- 평균 오브젝트 크기 : 1Mb
	- 평균 요청율(browser to server) : 15/sec, 1초에 대략 15개의 요청이 server로 전달된다는 의미
	- Router to Origin server RTT : 2sec,
	- access link rate : 15.4Mb/s
	- hit rate : 0.4
- 구해야 하는 것
	- LAN Utilization (LAN의 활성정도)
	- access link utilization (Access Link의 활성정도)
	- Total Delay

hit rate에 따라 총 요청의 일부만 origin server로 가고, 나머지는 local cache로 전달된다. 때문에 요청을 분리하여 각 요청마다의 가중치를 부여해야 한다.

**Total delay = 0.6 * Origin server delay+ 0.4 * delay with hit cache**
origin server delay를 구하기 위해 RTT + Access delay + LAN delay 를 계산하면

RTT = 2sec

15개의 요청중 약 60%만 Access Link를 사용하므로 초당 9MB의 데이터가 Access Link를 사용한다. 즉
**Access Link Utilization = 9/15.4 = 0.58 (under 80%)** 이다. 80%를 넘지 않기 때문에 매우 작은 수치인 10ms 로 가정한다.

Access Delay = 10ms
LAN Delay = usecs 

따라서 **0.6 * (Origin Server Delay) = 0.6 * (2 + 0.01)**

나머지 Delay with hit cache는 고려하지 않아도 될 정도로 작으므로 
**Total delay = 0.6 * (2 + 0.01)** 이다.


## Conditional GET

트래픽 감소와 access link 비용 절감, response time 감소 등의 목적으로 캐시를 사용한다고 해도 고려해야 할 문제가 있다. 캐시된 데이터는 항상 최신 상태의 데이터가 아니라는 점이다. 클라이언트는 항상 최신 상태의 오브젝트를 전달 받아야 하는 입장이다.

그렇다면 해당 캐시 데이터가 최신상태가 아닌지를 판단하려면 어떻게 해야 할까? 먼저 origin server는 알 수 없다. 단지 요청이 오면 요청한 오브젝트를 전달할 뿐이다. 그렇다면 프록시 서버가 해야 하는가? 그것도 아니다. 프록시 서버의 역할은 단지 동일한 오브젝트를 여러번 요청하는 상황에서 더욱 효율적인 리소스 사용을 위해 캐싱 역할을 담당하는 것이다. 오브젝트의 최신 상태를 유지하는 책임은 없다.

결국 최신 상태의 오브젝트를 원하는 클라이언트가 요구해야 한다. 캐시 시스템에 따라 클라이언트가 따로 최신 상태의 오브젝트를 요청한다는 말이 없으면, 마찬가지로 캐시에 있는지 확인하고 있으면 반환, 아니면 origin server로 요청이 넘어갈 것이다.

여기서 중요한 건 캐시된 데이터가 클라이언트가 말하는 최신 상태의 데이터라는 걸 판단하는 기준이 필요하다는 것이다.

그래서 HTTP Reqeust Message의 헤더 부분에 레코드 하나를 추가하여 클라이언트가 원하는 데이터가 최신 상태인지 아닌지를 판별하는 기준을 제공할 수 있다.

**If-modified-since : <date>**

클라이언트가 요청한 오브젝트가 <date> 날짜 이후로 수정되었는가? 를 판단한다.

- <date>이후로 오브젝트가 수정되지 않았다 : 프록시 서버로부터 오브젝트를 받아온다.
	- 이 때 304 Not Modified 라는 status code를 전달받는다.
- <date>이후로 오브젝트가 수정되었다 : origin server로 요청을 전달하여 오브젝트를 받아온다.
	- 200 OK!