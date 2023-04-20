---
title : Application Layer - DNS Overview
date: 2023/04/20
tags: ["DNS","Domain","HostName", "Record", "Hierarchial",'Application Layer']
author: 이인
---
# Domain Name System
우리가 특정 웹사이트에 접속하기 위해 해당 서버의 IP와 포트를 모두 외우고 다닐 수는 없다. 해당 웹 서비스의 특징을 잘 나타내는 alias가 사용자나 공급자나 모두 의미있는 방식일 것이다. 때문에 이런 IP 주소에 대한 Alias를 사용하기 위해서 DNS 가 필요하다.

## DNS 가 제공하는 Services

- translate hostname(도메인 이름)을 IP 주소로 변환
	- 여기서 말하는 hostname은 hostname(ex: www)와 도메인 example.com 이 합쳐진 정규화된 도메인 이름인 www.example.com 을 의미하는 듯 함
- host 별칭 : 하나의 호스트에 대해 다양한 이름을 할당할 수 있도록 함
- mail server 별칭 : 복잡한 메일 서버의 주소에 대한 별칭을 할당하여 이메일 전송을 단순화
- load balancing(distribution) : 복사본을 가지는 여러 개의 서버들을 배치하여 부하 분산
- 분산되고 계층적 구조의 데이터베이스

왜 단일 서버를 사용하지 않고 여러 서버를 사용할까? 

- single point failure : 하나의 서버가 죽어버리면 전체 시스템이 마비된다
- traffic volume : 하나의 서버에 과도한 트래픽이 집중된다.
- Distance Dleay : DNS 서버와 가까운 지역일수록 빠르고, 먼 지역일수록 딜레이가 크게 증가한다.
- 유지보수 및 관리 : 단일 서버는 확장이 매우 어렵다.

## Hierarchial Database

DNS 는 단일 서버가 아닌 계층적으로 분산화된 여러 서버들의 집합으로 이루어져 있다. 예로 www.amazon.com 라는 도메인에 대해 DNS가 IP로 변환하려고 한다면 다음과 같은 과정을 거친다.

1. client(browser)가 Root DNS servers 에 .com DNS server를 찾아달라는 쿼리를 전송한다.
2. Root DNS server는 .com DNS server의 주소를 찾아 다시 브라우저에 전달한다.
3. 브라우저는 전달받은 .com DNS server의 주소로 amazon.com 를 관리하는 Authoritative DNS server 주소를 요청한다.
4. .com DNS server가 amazon.com 을 관리하는 Authoritative DNS server 주소를 브라우저에게 반환한다.
5. 브라우저는 해당 Authoritative DNS server 에  www.amazon.com 과 매핑되는 IP 주소를 요청하고 반환받는다.

각 DNS Server에 대해 자세히 알아보자

### Root Name Servers

로컬 네임 서버가 해결할 수 없는 도메인에 대해 제일 먼저 Root name server로 전달된다.
요청받은 도메인 이름에 대한 정보가 없으면, 해당 도메인의 Top Level Domain을 관리하는 서버에 대한 주소를 반환한다.

### TLD

일반 TLD (.com, .org, .net 등)와 국가 코드 TLD (.us, .uk, .kr 등)에 대한 정보를 관리한다.
각 도메인에 대해 Authoritative DNS 서버 정보를 관리하며 요청받은 도메인에 대한 Authoritative DNS 서버 정보를 반환한다.

### Authoritative

일반적으로 웹 호스팅 회사등의 조직이 관리하며 특정 도메인의 최종 IP 주소 및 레코드 정보를 관리하는 서버이다.
요청된 도메인 이름에 대한 최종 IP 주소 및 관련 DNS 레코드를 반환한다.



## Local DNS Name Server

일반적으로 내부 네트워크 혹은 ISP 쪽에 위치한 DNS 서버이다. 다음과 같은 역할을 담당한다.

- 캐싱과 요청 중계 : 사용자가 요청한 도메인에 대한 DNS 쿼리를 받아 Root, TL, Authoritative 계층 구조에 순차적으로 접근하여 실제 주소를 반환해주며, 조회 결과는 일정기간 캐싱되어 동일한 도메인에 대한 DNS 쿼리가 도착했을 때 사전에 캐싱된 주소를 반환해주어 인터넷 전반적인 트래픽의 감소와 응답 시간의 감소 등의 이점을 가져다 준다.
- DNS Query의 Entry point : 사용자가 브라우저를 통해 웹 사이트에 접속할 때, 해당 도메인을 IP 주소로 변환하기 위한 DNS Query 요청의 첫 전달지점이 바로 Local DNS Name Server이다.

이때 캐시 데이터의 최신 상태 유지를 위해 TTL을 둔다. TTL은 유효기간과 같아 특정 캐시가 TTL 기간이 지나면 자동으로 만료되어 삭제된다. 때문에 전체 인터넷 입장에서 TTL이 만료되지 않은 상태에서 해당 도메인과 매핑되는 호스트의 주소가 변경된 경우, 이를 알 수 없다.


## DNS Resolution Example

DNS가 특정 호스트이름과 매핑되는 주소를 찾기 위해 resolution 하는 과정은 크게 iterated 방식과 recursive 방식으로 나뉜다.

### Iterated Resolution
![](https://i.imgur.com/ey8LCmo.png)

DNS Query가 일단 Local DNS Name server에 전달되면, LDNS가 DNS의 각 계층에 있는 서버에 반복적으로 접근하여 최종적인 IP 주소를 얻어서 Query를 요청한 호스트에게 전달하는 방식이다.

### Recursive Resolution
![](https://i.imgur.com/mKOYeWQ.png)

DNS Query의 최초 진입점이 LDNS 인건 동일하지만, 매 단계마다 쿼리의 주체가 이전 주체가 요청했던 서버로 넘어간다. 즉 LDNS -> ROOT DNS -> TLD -> AUTH 순으로 주체가 넘어간다. 각 주체는 응답을 받기 전까지는 Pending 상태로 대기해야 한다.

두 방식을 비교하여 알 수 있는건 Recursive 방식이 Iterated 방식보다 조금 더 친절하다. LDNS가 모든 제어를 담당하는게 아니기 때문이다. 그러나 가장 큰 문제점은 각 요청 주체는 응답이 오기 전까지 Pending 상태로 대기해야 하기 때문에 단일 Request host 입장에서는 이점이 많지만, 전체 네트워크 입장에서는 비효율적이다.

* 그리고 이러한 방식의 차이는 LDNS의 관점에서 차이가 나게 되는 것이다. DNS 쿼리를 발생시킨 주체, 즉 클라이언트는 Recursive 나 Iterated 나 똑같이 LDNS 에게 응답을 받아오는 것이기에 무조건 Recursive 하다고 볼 수 있다.

## DNS Record 살펴보기

- Type A :
	- Name : Hostname( hostname + domain, ex: www.example.com)
	- value : IP Address
- Type NS :
	- Name : domain(ex: example.com)
	- value : Authoritative Name server의 hostname(hostname + domain. ex: dns1.example.com)
- Type CNAME :
	- Name : 실제 이름의 별칭
		- www.ibm.com
	- Value : 실제 이름
		- servereast.backup2.ibm.com
- Type MX :
	- value : mailserver name


## 새로운 도메인 등록 예시

example.com 이라는 도메인을 새로 등록한다고 가정했을 때 다음의 과정을 거친다.

1. DNS Registrar(ex: 가비아, Route 53 등) 에게 example.com 도메인 이름을 등록요청한다
2. 이 때, Registrar 에게 해당 도메인의  primary 및 secondary Authoritative Name server의 이름과 주소를 제공한다.
3. Registrar 는 레지스트리가 관리하는 .com TLD 서버에 example.com 과 관련된 DNS Record를 추가한다. 이 때 추가하는 레코드는 다음과 같다.
```
Type NS = {
	name : example.com (hostname)
	value : auth1.example.com (도메인을 관리하는 Authoritative 서버의 hostname)
}

Type A = {
	name : auth1.example.com (Authoritative server hostname)
	value : 123.456.789.12 (Authoritative server IP)
}
```
단순히 생각해 봤을 때 TLD가  example.com의 실제 주소를 가진 레코드를 가질 이유가 없다. TLD는 해당 도메인에 대한 Authoritative Server 정보를 가지고 있어야 하기 때문에 해당 도메인을 관하는 Authoritative 서버의 hostname 정보를 담은 NS 레코드와, Authoritative 서버의 hostname에 매핑되는 실제 IP가 제공되어야 한다.

4. example.com 도메인의 등록을 요청한 요청자는 Authoritative 서버에 www.example.com(hostname)에 대한 IP 주소가 담긴 A 레코드를 등록하고 필요한 경우 MX레코드도 생성한다.

5. 이제 해당 도메인을 통해 실제 IP를 찾을 수 있다.

## 레지스트리(Registry), 레지스트라(Registrar), 리셀러(Reseller)

1.  레지스트리 (Registry): 레지스트리는 도메인 이름 확장자(TLDs, Top-Level Domains)에 대한 중앙 데이터베이스를 관리하고 운영하는 조직 예를 들어, `.com`, `.org`, `.net` 등의 TLD를 관리하는 ICANN(Internet Corporation for Assigned Names and Numbers)에서 인증받은 레지스트리가 있다.

2.  레지스트라 (Registrar): 레지스트라는 도메인 이름을 최종 사용자에게 판매하고 등록하는 회사(가비아, 아마존 Route 53 등) 레지스트라는 레지스트리와 계약을 체결하여 도메인 이름을 판매할 권한을 얻는다. 사용자는 레지스트라를 통해 원하는 도메인 이름을 등록하고 관리할 수 있다. 

3.  리셀러 (Reseller): 리셀러는 레지스트라와 제휴하여 도메인 이름을 판매하는 중개 업체 리셀러는 도메인 이름 등록 서비스를 제공하고, 이에 대한 고객 지원을 제공하지만, 도메인 이름의 등록과 관리는 실제로 레지스트라를 통해 이루어진다. 리셀러는 일반적으로 도메인 이름 뿐만 아니라 웹 호스팅, 이메일 호스팅, SSL 인증서 등 다양한 서비스를 함께 제공하는 경우가 많다. (ex: Cafe24)