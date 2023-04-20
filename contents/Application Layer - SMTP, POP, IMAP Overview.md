---
title : Application Layer - SMTP, POP, IMAP Overview
date: 2023/04/20
tags: ["Email","SMTP","POP", "IMAP", "Network", '네트워크','Application Layer']
---

# Email 의 전송 과정 알아보기
![](https://i.imgur.com/Jj5ZcGZ.png)

1. 유저가 UserAgent를 통해 이메일 URL을 포함한 메세지를 작성한다.
2. 해당 메세지를 해당 유저의 Mail Server로 전송하면 메세지가 Mail Server 의 message queue에 삽입된다.
3. Mail Server는 Receiver 측 Mail server와 TCP Connection을 맺고 SMTP 프로토콜 방식으로 message queue의 message 들을 순차적으로 전송한다.
4. Receiver측 Mail server는 해당 메세지를 Receiver의 MailBox에 저장한다.
5. Receiver는 UserAgent로 해당 메세지를 확인한다.


## SMTP 특징

- 메세지는 내용의 손실이 발생해선 안된다. 따라서 data integrity를 보장하는 TCP 를 사용한다
- 포트는 25번이다.
- 명령어는 역사적인 이유로 7Bit 기반의 ASCII 문자를 사용한다.
- 헤더, 바디 형태의 메세지를 전달한다.
- SMTP는 persistent connection을 사용하는데, Body에 어느정도의 데이터가 담길 지 알 수 없기 때문이다.
- CRLF . CRLF 를 통해 메세지가 끝났음을 표현한다.

## SMTP Sample Message
![](https://i.imgur.com/86SJwJ6.png)

- Header :
	- To: reciever mail server url
	- From : sender mail server url
- Blank line : HTTP와 동일하게 Header, Body를 명확히 나누기 위한 CRLF
- Body : 'message' ASCII to MIME

## MIME
![](https://i.imgur.com/pUsnCBP.png)

## Mail Access Protocols - POP, IMAP

SMTP 는 mail server 간 메세지를 전송 및 저장하기 위해 사용되는 프로토콜이다. 실질적으로 사용자가 해당 메세지를 확인하려면 mail server에서 해당 메세지를 불러와야 한다. 즉 mail server와 클라이언트 간 통신을 통해 mail server에 있는 메세지를 불러와야 한다. 그때 사용되는 프로토콜이 바로 POP 과 IMAP 이다.


- POP : 다른 클라이언트간에 메세지를 재사용 할 수 없다. 즉 하나의 클라이언트에 메세지가 다운로드 되면, 해당 메세지는 mail server에서 사라진다. 예로 동일한 메일 서버를 사용하는 PC, 스마트폰 둘 다 있다고 가정했을 때, PC에서 해당 메세지를 읽고 나면, 스마트폰에서는 해당 메세지를 읽을 수 없다.
- IMAP : 메일 서버에서 메시지를 관리하고, 클라이언트가 서버와 실시간으로 동기화하여 이메일을 확인할 수 있도록 하는 프로토콜이다. 즉 특정 클라이언트에 메세지가 다운로드 되더라도, 메일 서버에 복사본이 남아 있어, 다른 클라이언트에서 해당 메세지를 재사용할 수 있다.
- HTTP : GMAIL 과 같이 브라우저에서 메일을 확인할 수 있도록 Web Mail에서 사용한다.