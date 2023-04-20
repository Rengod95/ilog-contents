---
title : Application Layer - FTP Overview
date: 2023/04/20
tags: ["FTP", "Network", '네트워크', 'Passive Mode', 'Active Mode','Application Layer']
---

# FTP Overview

## FTP 
FTP(File Transfer Protocol)는 애플리케이션 계층(Application Layer)에서 사용하는 프로토콜이다. FTP는 TCP를 기반으로 하여 클라이언트와 서버 간에 파일을 전송하는 데 사용된다. 클라이언트와 서버 사이의 연결을 설정하고, 파일 및 디렉토리 목록을 검색하며, 파일을 업로드하고 다운로드하는 기능을 제공한다.

FTP의 가장 큰 특징 중 하나는 파일 전송을 위한 커넥션과 제어를 위한 커넥션, 총 두 개의 커넥션을 연결하여 사용한다.

- Data Connection : 실질적인 파일 데이터를 전송하기 위한 커넥션으로 20번 포트를 사용한다. 
- TCP Control Connection : Logical Communication 을 위한 커넥션이다. 21번 포트를 사용한다.

다음은 FTP 프로토콜은 통해 클라이언트와 서버가 파일을 주고받는 기본적인 프로세스이다.

1. 먼저 클라이언트가 TCP 프로토콜을 사용해 서버측 21번 포트로 접촉을 시도한다.
2. 서버가 인증된 사용자임을 확인해주고 control connection 을 맺는다.
3. 이 후 서버가 파일 전송 명령을 받는다면 20번 포트에 대해 client와 data connection을 맺는다.
4. 하나의 파일이 data connection을 통해 전송되고 난 후, 서버는 해당 data connection을 닫는다.
5. 이후 추가 적인 파일 전송 명령을 받을 때 마다 새로운 data connection을 생성하고 닫는 과정을 반복한다.

여기서 파일 하나당 하나의 data connection을 사용하는 걸 확인할 수 있다. 가늠컨데 보안상의 이유가 가장 클 것이다. 이런 특징 때문에 client 입장에선 하나의 파일을 주고받을 때 마다 새로운 data connection 연결에 대한 port를 열어야 한다. 즉 data connection 에 대해 서버의 포트는 항상 20으로 고정되지만, 클라이언트는 매번 유동적으로 달라진다.

다만 data connection 만 매번 열고 닫을 뿐, 오히려 TCP Control connection은 한 번 맺고나면 계속 상태를 유지한다. 이말은 한 번 인증된 사용자(클라이언트)가 파일 전송과 관계없이 자신이 탐색하고 있던 디렉토리에 대한 정보를 계속 가지고 작업을 이어갈 수 있도록 한다.

### Sample Commands

![](https://i.imgur.com/8GXt7l1.png)


## Passive Mode vs Active Mode

FTP 프로토콜의 가장 큰 특징 중 또 다른 하나는 Passive 와 Active 이다. Passive와 Active 모드의 가장 큰 차이점은 data connection 을 맺는 주체가 서버이냐 클라이언트이냐 에 따라 달라진다.

### Active Mode
FTP Active mode 방식은 data connection 을 맺는 주체가 서버이다. 

1. TCP Control Connection 을 연결한 이후에, 클라이언트가 특정 파일을 다운 혹은 업로드하려고 control connection 을 통해 메세지를 보낸다. 이때 해당 메세지에는 **클라이언트 측의 IP와 Port 번호가 포함된다.**
2. 서버는 해당 메세지를 수신하여 OK 메세지를 전달한다.
3. 서버는 자신의 20 port와 전달받은 클라이언트의 IP와 Port 번호로 data connection을 맺고 파일 전송을 시작한다.

여기서 가장 큰 문제는 클라이언트 측 방화벽이다. 일반적으로 클라이언트 측의 임의의 포트에 아무나 연결할 수 있는 보안 취약을 방지하기 위해 방화벽을 둔다. 서버의 입장에서는 약속된 방식대로 파일 전송을 위해 클라이언트가 전달한 IP와 Port 에 대해 data connection을 연결하려고 시도하는 것이다. 그러나 방화벽의 입장에서 클라이언트가 현재 어떤 작업을 하고있는지 알고 있지 않고 알 필요도 없다. 따라서 그냥 외부에서 함부로 클라이언트에게 연결을 시도한다고 판단하여 막아버린다.

때문에 연결의 주체를 Client로 하는 Passive Mode가 등장한다.

### Passive Mode

FTP Passive Mode는 data connection을 맺는 주체가 클라이언트이다.

1. 클라이언트가 서버에게 PASV 메세지를 보낸다.
2. 서버는 패시브 모드를 사용하여 파일 전송을 할 것이라고 인지하고, 서버측 임시 포트를 연다.(ex:3267)
3. 이후 클라이언트에게 OK 메세지와 서버측 IP 및 해당 임시 포트번호(3267)을 전달한다.
4. 클라이언트는 전달받은 IP와 포트 번호를 활용해 data connection을 맺는다.

이렇게 data connection 을 맺는 주체를 server -> client 로 바꾸어 방화벽 문제를 어느정도 해결할 수 있다. 
해당 Passive Mode Process에서 사용하는 명령어를 Passive Command라고 하는데, 요즘은 보안을 더 강화한 Extended Passive Command를 더 사용한다.

- Passive Command : 클라이언트가 PASV 명령을 보내면, 서버는 data connection 에 사용할 IP와 Port Number 둘 다 클라이언트로 전송한다. IPv4까지 지원한다.
- Extended Passive Command : 클라이언트가 PASV 명령을 보내면, 서버는 data connection 에 사용할 Port 번호만 응답으로 전송하게 된다. IP 주소는 TCP Control Connection 을 맺을 때 사용한 IP와 동일한 IP로 사용한다.

![](https://i.imgur.com/a8Kp04d.png)

여기서 server측 응답의 Port 번호 형태가 Passive 와 EPassive가 조금 다른것을 알 수 있다.

기본적으로 IP는 총 32비트, 4바이트로 구성되며 포트번호는 상위 8비트, 하위 8비트 총 16비트인 2바이트로 구성된다.

IP= (h1,h2,h3,h4) => hx 는 8bit 범위의 수를 표현 => 0~255 => 256가지 경우의 수
2^8 * 4 =  약 21억가지

PORT number = (P1, P2) => Px 는 8bit 범위의 수를 표현, 중요한건 P1은 **상위 8Bit**

예로 1111 1111 0000 0000 가 16비트로 표현된 수 라면, P1이 상위 8Bit를 명시한다는 말은 
전체 1111 1111 0000 0000 중에서
1111 1111 부분을 나타낸다는 말이다.

그래서 실제 포트번호를 구하기 위해서는 하위 8bit을 나타내는 P2와 자릿수를 맞춰주어야 하는 것이다.

따라서 P1 * 256 + P2 = 포트번호(Digit) 이 나오게 된다.





