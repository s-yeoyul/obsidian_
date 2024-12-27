# Ch2. Application Layer

## 2.1 Principles of Network Applications

Network Application 개발의 핵심은 다른 end system이 네트워크 위에서 서로 communicate하는 것이다.
예를 들면
1. 웹브라우저(호스트에서 돌아가는)
2. 넷플릭스 같은 Video on Demand Application(~2.6)
같은.
따라서 application software를 짤때는 multiple end system에서 돌아갈 수 있도록 구현 해야한다.

![](https://i.imgur.com/H1lO6Ys.png)
Application software는 end system에만 국한된다.
network-core device들은 network layer 밑 그 아래 영역에만 접근 가능하다.

### 2.1.1 Network Application Architectures

The application architecture is designed by the application developer and dictates **how the application is structured over the various end systems.**

1. Client-Server Architecture
2. P2P Architecture
#### 1. 
항상 켜져있는 host인 `server`가 있고, `client`로부터 들어오는 request에 서비스를 제공한다.
이 구조에서 클라이언트 끼리는 절대 direct하게 communicate하지 않는다.
또한 이 구조에서 서버는 고정되고 well-known한 주소값을 갖는데 이를 IP address라고 한다.
이러한 특징 때문에 클라이언트는 항상 서버로 패킷을 보내 통신할 수 있다.

single server만으로는 클라이언트의 request를 처리하기 어렵기 때문에 data center를 짓고 거기서 강력한 virtual server를 만들어서 host를 housing해서 처리한다. 
데이터 센터가 여러개 있는 경우도 흔하다.

#### 2.
P2P 구조에서는 서버에 대한 의존도가 매우 낮다.
peers: pairs of intermittently connected hosts.
파일을 공유해주는 BitTorrent 같은게 그 예시가 된다.

또 하나의 특징은 self-scalability이다.

![](https://i.imgur.com/X7bIsOZ.png)

싸고, 거대 구조(significant server infrastructure)가 없는 장점이 있지만 
보안, 신뢰성 이슈가 있고 퍼포먼스도 별로.

### 2.1.2 Processes Communicating
운영체제의 용어로, communicate하는 것은 program이 아니라 process이다.
process는 end system에서 돌아가는 프로그램으로 생각하면 된다.

다른 운영체제에서 돌아가는 프로그램이 어떻게 communicate할 수 있을까??

서비스를 제공하는 자를 server, 요청하는 자를 client라고 하자.
이는 P2P에서도 사용되는 개념이다.

>[!note] Definition
>In the context of a communication session between a pair of processes, the process that **initiates the communication is labeled as the client.** The process that **waits to be contacted to begin the session is the server.**

client가 먼저 시작한다. server는 가만히 있다가 봉변.

먼저 한 process에서 다른 process로 가기 위해서는 깔려있는 네트워크를 통해 가야한다.
process는 네트워크로 메시지를 보내고, 또 네트워크로 메시지를 받는데, **socket**이라는 interface를 통해 받는다.
비유하자면 process는 집이고 socket은 문이다.
API(Application Programming Interface)로도 불린다.
Application 개발자는 application layer에만 컨트롤 권한이 있지 transport layer에는 권한이 없다.
오직 transport protocol을 고르고 몇개의 parameter를 변경하는 수준(...챕터3)

호스트에게서 다른 호스트로 패킷을 보내려면 1. host의 주소와 2. identifier가 필요하다.
**호스트는 IP address로 identify된다**(...챕터4). 32비트 숫자.(65536)
그리고 어떤 process에 접근하려 하는지도 알아야 한다. 즉 어떤 문(socket)으로 들어오는지 알아야 한다!
**이건 port number를 통해 알 수 있다.**

### 2.1.3 Transport Services Available to Applications

대부분의 네트워크들은 하나 이상의 Transport-layer protocol을 제공한다. 어떻게 선택할까?
이는 마치 도시를 오갈 때 기차를 탈 지 비행기를 탈 지 고르는 정도의 고민과 가깝다.
각자 상황에 맞게 일장일단이 있는 것!

>[!note] Possible services of Transport layer
>1. Reliable Data Transfer
>2. Throughput
>3. Timing
>4. Security

#### Reliable Data Transfer(Data Integrity)
앞서 보았듯 packet loss가 발생할 수 있는데, 대부분의 경우 절망적인 결과를 야기한다. (특히 은행 거래를 생각해보라.. 끔찍)
그렇기 때문에 **데이터가 올바르고 완전하게 전달되는 것이 보장되어야 한다**.
만약 어떤 protocol이 이를 보장한다면 그 protocol이 Reliable Data Transfer를 provide한다고 말한다.
만약 위와 같은 성질이 보장되는 게 필요없는 상황이라면(예를 들어 스트리밍에서 조금 오디오/비디오 손실, 위와 같은 application을 **loss-tolerant application**이라고 부른다) 까짓거 좀 견딜수 있다.

#### Throughput
네트워크의 특성 때문에 Throughput은 시간에 따라 변동될 수 있다. 어떤 서비스에서는 특정 rate로 throughput을 보장받을 필요가 있다. 즉 최소 throughput of bits/sec를 보장해준다.
위와 같은 제약이 걸린 application을 **bandwidth-sensitive applications**라고 부른다.
반대로 있는대로 쓰는 application을 **elastic applications**라고 부른다.

#### Timing
패킷이 보내진 후 특정 시간 내에 목적지에 도달해야 하는 요구 사항이 있는 경우.
실시간 application 등에서 요구된다.

#### Security
데이터가 송신자에게서 encrypt되고 수신자에게서 decrypt된다.
송신 과정에서 데이터를 접근 할 수 있어도 기밀이 유지되어야 할 때. 
챕터 8에서 더 자세히.

![](https://i.imgur.com/v3m7HV6.png)
### 2.1.4 Transport Services Provided by the Internet

우리가 새로운 network application을 만들때 처음 결정해야 하는것은 UDP를 쓸건지 TCP를 쓸건지이다.

#### TCP
- **Connection-oriented** service
  본격적으로 application-level에서 동작하기 전에 control information을 transport-layer단에서 교환한다. 이러한 **handshaking procedure**를 거쳐서 패킷 무차별 공격을 준비한다. 그 후 **TCP Connection**단계는 두 process의 socket 사이에 존재하고, 서로서로 동시에 메시지를 주고받을 수 있게 된다. 끝나면 연결을 끊어야 한다. Ch3 에서 더 자세히...
- **Reliable data transfer** service
  한 소켓으로부터 데이터가 나가면 받는 소켓에서 데이터가 잘 왔는지 확인한다.

#### UDP
담백하고 **경량화된 protocol**, 최소한의 서비스만 제공한다.
UDP는 connectionless이기 때문에 별도의 handshaking process도 없다.
즉 unreliable data transfer service이다.

위를 보면 Timing과 Throughput에 대한 얘기는 빠져있는 걸 알 수 있다.
그런데 현재의 Application들은 그것들에 잘 대처할 수 있도록 이미 디자인 되어있어서 그렇다.
즉 time sensitive application에 대해 만족할만한 서비스를 제공하기는 하지만 어떠한 timing / throughput도 보장하지는 않는다.

#### Securing TCP
TCP나 UDP 모두 encryption을 제공하지 않는다. 즉 socket에 들어가는 데이터와 네트워크 위에서 전달되는 데이터가 같다. 그래서 만약 password가 unencrypted 된 채로 전달되면 중간에 유출될 우려가 있다. 그래서 TCP를 **SSL(Secure Sockets Layer)** 로 enhance하는 방법이 개발되었다. TCP enhancement는 TCP와 같은 transport level에서 수행되는 것이 아니라 application layer에서 구현된 것이다.

### 2.1.5 Application Layer Protocols
의문점: 메시지는 어떻게 만들어지냐? 메시지의 의미가 뭐냐? 메시지는 언제 보내지냐?
이러한 질문에 답은 application layer protocol에 존재한다.
Application layer protocol은 application이 어떻게 동작하고 다른 end system에서 실행되고 메시지를 서로 전달하게 되는지를 정의한다. 구체적으로는 다음과 같다
![](https://i.imgur.com/BbkNuYA.png)

---

## 2.2 The Web and HTTP

### 2.2.1 Overview of HTTP

HTTP: HyperText Transfer Protocol = Web의 application layer protocol.
HTTP는 서버/클라이언트 프로그램으로 구현된다.

먼저 Web의 용어에 대해 알아보자.
Web Page는 object로 구성되는데, object는 간단하게 file을 의미한다.
**Web Page는 Base HTML file과 다른 object들로 구성된다.**

base HTML은 다른 object들을 URL을 통해 referencing한다.
`http://www.someSchool.edu/someDepartment/picture.gif`
에서
`www.someSchool.edu`가 hostname이고
`/someDepartment/picture.gif`가 path name이다.

대략적인 설명은 다음과 같다.

![](https://i.imgur.com/8mHTGUG.png)

HTTP는 **TCP를 transport protocol로 사용한다.** 
client는 TCP connection을 시작하여 서버와 port 80으로 연결한다. 

### 2.2.2 Non-Persistent and Persistent Connections

>[!note] 
>**Persist**는 네트워크적 관점에서 **“지속적인 연결”**이나 **“연결 유지”**와 관련된 개념으로 사용됩니다. 일반적으로 **세션이나 연결을 끊지 않고 지속적으로 유지하는 방식**을 의미합니다. 이는 클라이언트와 서버 간의 연결이 끊어지지 않고 계속해서 **연결 상태를 유지**하거나, **재시도**를 통해 데이터를 다시 보내는 방식 등을 포함할 수 있습니다.


서버-클라이언트 interaction이 TCP에서 발생할 때, application developer는 중요한 결정을 해야 한다;
**request/response pair가 separate한 TCP connection으로 보내져야 하는가 아니면 same한 TCP connection으로 보내져야 하는가?**
전자가 non-persistent, 후자가 persistent connection이다.

#### Non Persistent
TCP connection이 server가 매 object를 보낼 때 마다 종료된다. 다른 object에 대해서는 persist 되지 않는다.

>[!abstract] Procedure
>1. HTTP 클라이언트가 서버의 포트 80으로 TCP 연결을 시작함.
>2. 클라이언트가 서버에 HTTP 요청 메시지를 보냄.
>3. 서버가 요청을 받아 파일을 찾아 응답 메시지로 클라이언트에 보냄.
>4. 서버가 TCP 연결을 닫도록 지시함 (클라이언트가 응답을 완전히 받을 때까지 기다림).
>5. 클라이언트가 응답을 받고 연결이 종료됨. HTML 파일을 확인하고 10개의 JPEG 파일을 참조함.
>6. 위 과정을 각 JPEG 파일에 대해 반복함.

client가 base HTML 파일을 요청할 때 부터 전체 파일을 받을 때 까지 걸리는 시간을 계산해보자.
이를 위해서 우리는 **RTT(Round-Trip Time)** 을 정의하는데, 패킷이 클라이언트에서 서버로, 그리고 다시 클라이언트로 돌아오는데 걸리는 시간을 의미한다. 챕터 1에서 다룬 딜레이를 포함한 시간이다.

![](https://i.imgur.com/3GwfJpK.png)

3단계의 hand-shake process를 거치게 된다. (가는 파란색 선)
1. 클라이언트가 small TCP segment를 보내고
2. 서버가 그걸 acknowledge하고 small TCP segment로 respond하고
3. 클라이언트가 acknowledge한다. 이때 HTTP request를 같이 보낸다.

결과적으로 총 소요 시간은 두 번의 RTT와 HTML을 보내는 transmisson time의 합이다.

#### Persistent

1. 새로운 연결을 만들고 각각의 requested object에 대해 유지된다. TCP buffer가 할당되고 TCP variable은 서버와 클라이언트에 각각 유지되어 있어야 한다. 부담될 수 있음
2. 두 번의 RTT 시간을 통해 communicate

### 2.2.3 HTTP Message Format

>[!important] Request Format
>GET /somedir/page.html HTTP/1.1 
>Host: www.someschool.edu 
>Connection: close  
  User-agent: Mozilla/5.0 
  Accept-language: fr

첫 줄을 request line이라고 부르고 나머지 줄을 header line이라고 부른다.
request line은 세 개의 필드로 나뉜다

1. Method: `GET, POST, DELETE, ...`
2. URL:
3. HTTP version

`Connection: close`라고 되어있는 부분은 브라우저가 서버한테 persistent connection을 원치 않는다고 전달하는 것이다. 내가 원하는거 주고 꺼져(ㅠ)
`User-agent`는 browser의 타입을 의미한다.
`Accept-language`는 클라이언트가 선호하는 버전의 객체를 명시해놓은 것이다. 예시에서는 French.

![](https://i.imgur.com/QoYfA4o.png)

`sp`: SPACE `cr`: CARRIAGE RETURN `lf`: LINE FEED
`cr` + `lf` = `\n`
Entity Body는 POST method에서 사용된다.

>[!important] Response Format
>HTTP/1.1 200 OK  
Connection: close  
Date: Tue, 18 Aug 2015 15:44:04 GMT  
Server: Apache/2.2.3 (CentOS)  
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT 
Content-Length: 6821  
Content-Type: text/html
(data data data data data ...)

첫 번째 줄은 status line, 마지막의 데이터는 entity body, 사이에는 header lines.
status line은 protocol version, status code, status message 총 3개의 field로 구성된다.
![](https://i.imgur.com/pBJ0Eq1.png)

### 2.2.4 User-Server Interaction: Cookies
~~내가 만든 쿠키\~~~
HTTP는 `stateless`하다. 이는 **HTTP가 클라이언트에 대한 어떠한 정보도 유지하지 않는 성질**을 의미한다.
그러나 가끔 서버가 유저를 확인해야 되는 경우가 생길 수도 있다. 그러한 연유로 HTTP는 cookies를 사용한다.
cookies는 site가 user를 keep track하게 해준다.
![](https://i.imgur.com/MsdyeCU.png)
수연이가 아마존을 처음 방문했다.
서버에 요청이 처음 들어오면, 서버는 unique ID를 만들고 백엔드 데이터베이스에 유지시킨다.
서버가 response와 함께 `Set-cookie` 값을 보내는데, 수연이의 브라우저는 이걸 받고 쿠키 파일을 관리하는 line을 하나 추가한다. 즉 이제 수연이는 request를 보낼 때 마다 header line에 `Cookie: 1678` line이 추가로 붙게 되는 것이다. 
수연이가 회원가입을 하지 않아도 좋아요 누른걸 유지해놓다가 회원가입을 했을 때 그 정보를 그냥 붙이기만 하면 된다.

### 2.2.5 Web Caching
`Web cache` = `proxy server` is a network entity that satifies HTTP requests on the behalf of an origin web server.
![](https://i.imgur.com/k1AbcOP.png)

1. 브라우저는 웹 캐시와 TCP 연결을 설정하고, 해당 객체에 대한 HTTP 요청을 웹 캐시에 전송한다.
2. 웹 캐시는 로컬에 객체가 저장되어 있는지 확인한 후, 존재할 경우 HTTP 응답 메시지로 클라이언트 브라우저에 반환한다.
3. 웹 캐시에 객체가 없을 경우, 웹 캐시는 원 서버(www.someschool.edu)로 TCP 연결을 설정하고, 해당 객체에 대한 HTTP 요청을 원 서버로 전송한다. 원 서버는 이 요청을 받은 후, HTTP 응답으로 객체를 웹 캐시에 전송한다.
4. 웹 캐시는 받은 객체를 로컬 저장소에 저장한 후, 기존의 TCP 연결을 통해 HTTP 응답 메시지로 클라이언트 브라우저에 전달한다.

**특정 시점에 cache는 서버 역할과 클라이언트의 역할을 동시에 수행한다!**
Web cache는 ISP에 의해 주로 구매되고 설치된다. 예컨대 대학에서는 cache를 캠퍼스 네트워크에 깔아서 모든 캠퍼스 내의 브라우저가 cache를 가리키게 설정한다.
이름에서 알 수 있듯이 웹 캐시는 마치 CPU~메모리 사이의 속도 차이를 맞춰주기 위한 memory hiararchy로써의 cache와 같은 역할을 한다. CPU와 메모리가 서버와 클라이언트에 비유된다.
`Hit rate`는 들어온 request를 캐시 선에서 처리할 수 있는 확률을 의미한다. 보통 .2 ~ .7

근데 서버의 정보가 업데이트 되어서 캐시에 있는 정보가 서버랑 다르면 어떡하지? Conditional GET.
캐시가 up-to-date check를 한다.

>[!important] Conditional GET, cache to server
>GET /fruit/kiwi.gif HTTP/1.1  
Host: www.exotiquecuisine.com 
If-modified-since: Wed, 9 Sep 2015 09:23:24

>[!important] Conditional GET, server to cache
>HTTP/1.1 304 Not Modified  
Date: Sat, 10 Oct 2015 15:39:29 
Server: Apache/1.3.0 (Unix)

이렇게 되면 캐시는 자기가 가지고 있는거를 브라우저에 쏴주면 되겠다는 판단을 한다.

### 2.2.6 HTTP/2

> The primary goals for HTTP/2 are to reduce perceived latency by enabling request and response multiplexing over a single TCP connection, provide request prioritization and server push, and provide efficient compression of HTTP header fields.

HOL Blocking: 용량이 큰 영상 데이터 밑에 작은 object들이 있는 html page를 생각해보자.
Single TCP connection을 사용하면 비디오를 로딩하느라 밑에 object들이 block되는 사고가 발생한다.
HTTP/1.1 은 multiple parallel TCP connection으로 이를 해결한다.

HTTP/2는 각각의 메시지를 작은 프레임 단위로 쪼개서 request와 response를 동일 TCP connection 상에서 번갈아 가며 송수신한다. 만약 큰 비디오 하나랑 8개의 작은 object가 있으면 HTTP/1.1은 TCP를 9개 쓴다면 HTTP/2는 비디오의 첫 프레임을 보내고 각각의 작은 object의 첫 프레임을 보내고 비디오의 두번째 프레임을 보내고 object의 다음 프레임을 보내고... 뭐 이런 식이다.
따라서 HTTP/2는 user perceived delay를 줄여준다.

HTTP/2에는 메시지에 가중치를 붙여서 가중치가 높은 메시지부터 먼저 처리하는 **Response message prioritization** 기법이 있다.
또한 서버가 컴파일러 마냥 HTML base page를 분석해서 필요한걸 알아서 먼저 보내주는 **Server push** 테크닉도 존재한다.

## 2.3 Electronic Mail in the Internet
![](https://i.imgur.com/BMtakxl.png)

일반적인 메일의 송수신 과정은 다음과 같다.
송신자가 user agent에서 메일을 보내고 송신자의 메일 서버로 간다. 그게 수신자의 메일 서버로 간 뒤 수신자의 mailbox로 들어오게 된다. 수신자가 메일을 확인하고자 하면 수신자의 권한을 확인 후 접근을 허용한다. 만약 수신자의 서버에 메일이 보내지지 않는다면 message queue에 메시지를 두고 계속 시도한다.

### 2.3.1 SMTP; Simple Mail Transfer Protocol
SMTP는 HTTP보다 오래된 기술로 옛날에는 7비트 char 까지만 보낼 수 있었다...
![](https://i.imgur.com/rhkrPsr.png)

사람과 사람이 만나서 인사하는 과정과 매우 유사하다.
client side의 SMTP가 message queue의 메시지를 보고 TCP connection을 연다. handshaking을 진행하는데, 이때 client가 수신자 및 송신자의 email address를 알려준다. handshaking이 끝나면 서버에 메시지를 보내고, server side의 SMTP가 수신한다. TCP의 Reliable Data Transfer 때문에 안정적으로 메시지 수신이 가능하다.

>[!quote] Example Dialogue
S: 220 hamburger.edu 
C: HELO crepes.fr
S: 250 Hello crepes.fr, pleased to meet you 
C: MAIL FROM: <alice@crepes.fr>  
S: 250 alice@crepes.fr ... Sender ok  
C: RCPT TO: <bob@hamburger.edu>
S: 250 bob@hamburger.edu ... Recipient ok  
C: DATA  
S: 354 Enter mail, end with ”.” on a line by itself 
C: Do you like ketchup?  
C: How about pickles?  
C: .  
S: 250 Message accepted for delivery  
C: QUIT  
S: 221 hamburger.edu closing connection

SMTP는 persistent connection을 사용한다.

그리고 Alice가 Alice의 메일 서버에 먼저 보내고 그 다음에 Bob의 서버로 보내는데, 두 단계를 거치는 이유는 Alice 서버로 먼저 보내면서 Bob의 서버로 계속 요청을 보낼 수 있기 때문이다. 클라이언트가 그렇게 하는건 매우 비효율적이므로...
근데 또다른 문제는 Bob이 메일을 어떻게 받느냐 하는 것이다. 메시지를 받는건 `PULL` 인데 SMTP는 `PUSH` protocol이다. 그래서 HTTP나 IMAP(Internet MAil Access Protocol)을 사용한다.

## 2.4 DNS-The Internet's Directory Service
사람들이 다양한 방법으로 identification 되듯이 internet host도 마찬가지이다. host의 하나의 identifier는 **hostname**이다. `www.google.com, www.facebook.com` 과 같은 mnemonic을 말한다.
사람이 이해하기는 쉽지만 router가 이해하기는 어렵고 담겨있는 정보량도 적다.
따라서 **IP Address**와 같은 identifier도 사용한다. (Ch 4)
### 2.4.1 Services Provided by DNS
사람의 hostname 선호와 router의 ip 선호를 reconcile하기 위해 Directory Service가 필요하다.
**Here Domain Name System(DNS) kicks in**

>The DNS is (1) **a distributed database** implemented in a hierarchy of DNS servers, and (2) **an application-layer protocol** that allows hosts to query the distributed database.

DNS는 다른 application layer protocol에 사용된다. 예시를 보자.

1. 동일한 사용자 기기가 DNS application의 client side를 실행한다.
2. 브라우저는 URL에서 hostname, www.someschool.edu를 추출하고, 이를 DNS application의 client side로 전달한다.
3. DNS client는 hostname이 포함된 query를 DNS server로 보낸다.
4. DNS client는 hostname에 해당하는 IP address가 포함된 reply를 받는다.
5. 브라우저가 DNS로부터 IP address를 받으면, 해당 IP address의 port 80에서 HTTP server process와 TCP connection을 시작할 수 있다.

DNS가 끼어들면서 추가적인 딜레이가 생기지만 대부분 caching 되어 있기 때문에 DNS delay는 매우 적다.
또한 다음과 같은 추가적인 역할이 있다.
1. Host aliasing
   canonical hostname 외에 추가적인 hostname을 가질 수 있게 해준다.
   `www.naver.com(canonical)` 이랑 `naver.com`
2. Mail Server aliasing
   메일의 @ 뒤에 붙는 주소가 canonical 이어야 한다면 복잡할 수 있는데 간단하게 할 수 있게 해준다.
3. Load distribution
   트래픽이 많은 사이트에 여러개의 IP 주소를 할당해줘서 Load를 분배해준다.

### 2.4.2 Overview of How DNS works

수연이가 브라우저를 돌린다. hostname을 IP로 바꿔야 하는 상황이 생겨서 DNS의 client side를 호출한다.
DNS가 네트워크에 query message를 보낸다. (DNS는 UDP의 포트 53번에서 Transport)
DNS delay가 지나면 원하던 mapping을 수신할 수 있다. 즉 유저의 application 관점에서 DNS는 translation service를 제공하는 simple하고 straightforward한 black box와 같다.
간단하게 생각해서 모든 DNS mapping을 다 때려박는 구조를 생각할 수 있겠지만 host가 너무 많은 현대에는 비효율적이다.

1. **Single point of failure**: DNS 서버가 다운되면 인터넷도 멈춘다.
2. **Traffic volume**: 모든 DNS 요청을 한 서버가 처리하기엔 과부하가 크다.
3. **Distant database**: DNS 서버가 멀리 있으면 지연이 발생할 수 있다.
4. **Maintenance**: 모든 호스트 기록을 관리하는 중앙 서버는 유지보수가 어렵다.

그래서 우리는
#### A Distributed, Hierarchical Database
로 DNS를 구현하고자 한다.
`www.amazon.com` 에 대한 IP 주소를 받아오는 과정을 통해 알아보자.

![](https://i.imgur.com/8DzqkRK.png)

• **Root DNS servers**: 전 세계에 1000개 이상의 root server 인스턴스가 있으며, 13개의 서로 다른 root 서버의 복사본으로, IANA에 의해 관리된다. 이 서버들은 TLD 서버의 IP 주소를 제공한다.
• **Top-level domain (TLD) servers**: com, org, net 등의 일반 TLD와 국가별 TLD(uk, fr, jp 등)를 위한 서버들이다. Verisign은 com TLD 서버를, Educause는 edu TLD 서버를 관리한다. TLD 서버는 authoritative DNS 서버의 IP 주소를 제공한다.
• **Authoritative DNS servers**: 공용 웹 서버나 메일 서버를 운영하는 조직은 DNS 레코드를 관리하는 authoritative DNS 서버를 제공해야 한다. 조직은 자체 DNS 서버를 운영하거나 서비스 제공자의 DNS 서버를 사용할 수 있다. 대부분의 대학과 대기업은 자체 primary와 secondary authoritative DNS 서버를 운영한다.

DNS 서버에는 root, top-level domain(TLD), authoritative 3개의 계층이 존재한다.
먼저 client가 root server 중 하나랑 접촉해서 Top Level Domain = `.com` 서버의 ip 주소를 받아온다.
그러면 client가 TLD server에 접촉해서 authoritative server = `amazon.com`의 ip 주소를 받아온다.
이제 client가 Authoritative server에 접촉하면 `www.amazon.com` 의 ip 주소를 받을 수 있다.

![](https://i.imgur.com/br5hAsJ.png)
local DNS server라는 개념이 있다.
ISP에는 local DNS server가 존재해서 같은 LAN에서 빠르게 처리 가능하다.
우리가 `gaia.cs.umass.edu` 의 IP Address를 요청한다고 하자.
3번에서 root는 `.edu` 에 해당하는 TLD DNS Server의 주소 리스트를 제공한다.
그걸 4번을 통해 묻고 5번으로 받는다.
6번에서 Local DNS는 `.umass.edu` 에 해당하는 Authoritative DNS Server의 주소를 query한다.
7번으로 받고 8번으로 local에 던져준다.
recursive query는 물음에 대해 물음으로 답하는 것이다. 1 -> 2번.
나머지는 다 iterative query이다. 물음에 대해 답을 즉각적으로 주는것. 예컨대 2-> 3번.

DNS Caching은 local DNS server에 IP 주소값을 캐싱해놓는 것이다. 그럼 좀 더 빠르겠지.
![](https://i.imgur.com/2Aom4Rl.png)

### 2.4.3 DNS Records and Messages
RR: Resource Record
`(name, value, type, ttl)`
`ttl`은 time to live의 약자로 resource의 생존 시간을 의미한다.

>1. **Type=A**: Name은 호스트 이름이고, Value는 해당 호스트 이름의 IP 주소이다. Type A 레코드는 **호스트 이름을 IP 주소로 매핑**하는 기본 DNS 레코드이다. 예: (relay1.bar.foo.com, 145.37.93.126, A).
>2. **Type=NS**: Name은 도메인 이름이고, Value는 해당 도메인에 대한 **Authoritative DNS server의 host name**이다. 이 레코드는 **DNS 쿼리를 더 진행**하기 위해 사용된다. 예: (foo.com, dns.foo.com, NS).
>3. **Type=CNAME**: Value는 Name의 **정식 호스트 이름(canonical hostname)** 이다. 이 레코드는 별칭 호스트 이름에 대한 정식 이름을 제공한다. 예: (foo.com, relay1.bar.foo.com, CNAME).
>4. **Type=MX**: Name은 메일 서버의 별칭이고, Value는 해당 메일 서버의 **정식 호스트 이름**이다. MX 레코드는 **메일 서버에 대한 별칭**을 제공하며, DNS 클라이언트는 MX 레코드를 조회하여 메일 서버의 정식 이름을 얻을 수 있다. 예: (foo.com, mail.bar.foo.com, MX).

즉 Autoritative DNS Serversms Type=A를 사용하고 이외의 서버는 Type=NS를 사용할 것이다.

DNS Message의 Format은 아래와 같다.

![](https://i.imgur.com/hzkUXsG.png)
이제 Record를 DNS DB에 기록해보도록 하자.

>A **registrar** is a commercial entity that verifies the uniqueness of the domain name, enters the domain name into the DNS database (as discussed below), and collects a small fee from you for its services. (도메인 이름 등록 기관)

domain name `carrotshop.shop` 을 등록하면 primary/secondary authoritative DNS Server의 name과 IP 주소 또한 등록해야 한다. 이러한 과정을 마치면 이용 가능한 도메인이 된다.

## 2.5 P2P File Distribution

always on server 대신에 간헐적으로 연결된 Peer의 쌍이 직접적으로 연결되며 communicate 하는 방식이다.
클라이언트-서버 구조에서는 서버가 파일의 복사본을 각각의 클라이언트에게 보내야 하기 때문에 서버에게 부담이 되고 large amount of bandwidth를 소모하는데 반해 P2P File Distribution에서는 각각의 peer가 수신한 파일을 다른 peer에게 공유할 수 있다. 즉 서버를 도와주는 것!
대표적인 예시가 BitTorrent. 

**\<Scalability of P2P Architectures>**
![](https://i.imgur.com/WQ4Ji5g.png)
![](https://i.imgur.com/nxwAEyt.png)

u, d는 각각 upload time과 download time을 의미하고, 파일 크기는 F이며 peer의 수는 N으로 주어졌다.
이때 파일을 모든 peer에게 보내는 데 걸리는 시간을 **distribution time**이라고 한다.
먼저 클라이언트-서버 구조에서 Distribution time을 계산해보자. 
- 서버 혼자서 N명에게 F bits 크기의 파일을 보내야 한다. 즉 NF 크기의 파일을 보내야 한다. 서버의 upload rate가 u 이면 distribution time은 최소 NF/u 만큼 걸린다.
- 제일 느리게 다운로드 받는 peer의 download rate를 d라고 하면 파일 ![XG4qSgx.png](https://i.imgur.com/XG4qSgx.png)다운로드는 F/d 보다 빨리 완료될 수 없다.
- 위 둘을 결합하면 다음과 같은 부등식, 즉 Lower bound를 얻는다.
![](https://i.imgur.com/XG4qSgx.png)
(homework problem에 lower bound가 achieve되는 걸 보이는 게 있다)
만약 N이 커지면 distribution time이 N에 영향을 받을 것임을 알 수 있다. 

이제 P2P의 경우를 볼 텐데, 훨씬 복잡한 것이 각 peer가 file을 얼마씩 나눠가지고 있는지에 따라 distribution time이 결정되기 때문이다. 한 번 보자.
- 처음엔 서버만 파일을 갖고 있다. 그래서 어찌됐든 한 번은 파일을 access link로 보내긴 해야 한다. 이때 걸리는 시간은 F/u이다.
- 가장 느린 다운로드 속도를 갖는 peer에 의해, F/d 보다 distribution이 빠를 수 없다.
- 전체 system의 upload capacity는 서버와 각 peer의 upload rate의 합이다. 즉 distribution time은 최소 NF/(sum of u)가 된다. 
- 위 셋을 결합하면 다음과 같은 lower bound를 얻는다.
![](https://i.imgur.com/xrATsgc.png)

둘의 distribution time을 비교한 plot을 살펴보자.
![](https://i.imgur.com/5L90gX5.png)

P2P는 N의 크기에 관계없이 distribution time을 유지하기 때문에 self-scalability를 갖는다고 말할 수 있다!

**\<BitTorrent>**
> In BitTorrent lingo, the collection of all peers participating in the distribution of a particular file is called a *torrent*.

torrent에서의 peer는 equal-size의 *chunks*를 서로서로 download한다. 처음 들어오는 Peer는 chunks가 없다. 시간이 지나며 그 가난한 친구는 chunks를 축적하게 된다. 그 친구가 download 하는 동안 다른 peer에게 chunk를 upload한다. 만약 어떤 peer가 download가 끝났으면 이기적으로 torrent를 뜨거나 이타적으로 다른 peer에게 chunks를 공급하기 위해 남아있을 수도 있다. 아니면 적당량만 다운 받고 나중에 다시 들어와서 완료하기 위해 나가는 방법도 존재한다.

이제 어떻게 torrent가 작동하는지 개괄적으로 살펴보자. 먼저 각 torrent는 *tracker*라는 노드 구조가 존재한다. peer가 torrent에 참여하면 tracker에 register함으로써 지속적으로 '나 토렌트 안에 있어' 라는 것을 알린다. 즉 tracker는 말 그대로 각 Peer의 상태를 keep tracking 하는 것이다. 
![](https://i.imgur.com/ToxebPV.png)

만약 수연이가 torrent에 처음 방문하면 tracker가 랜덤으로 peer subset의 ip address를 제공하고 그 아이들과(걔네를 neighboring peers라고 부른다) TCP connection을 만들어서 communicate를 시작한다.
만약 걔네가 가지고 있지 않는 chunk가 있을경우 request를 날리게 되는 것이다. 
수연이는 두가지 중요한 질문을 떠올렸다.
1. 무슨 Chunk를 걔네한테 먼저 달라고 하지?
2. 그리고 requested chunk를 누구한테 보내야 되지?
첫번째 질문에 대해 대답하자면 **가장 요청률이 저조한 chunk를 먼저 받아와야 한다(rarest first)**. 그래야지 그 chunk가 잘 재분배 될테니까!
두번째 질문을 생각해보자. BitTorrent에서는 수연이의 이웃에 대한 priority 정보를 가지고 있다. chunk를 높은 rate로 보내 주면 priority가 높은 것이다. **많이 주는 사람에게 많이 돌려준다.**
수연이는 10초마다 나한테 젤 많이 주는 four peer set을 슉슉 수정하는데 이때 four peers가 *unchoked* 되었다고 말한다. 또 매 30초마다 아무 Neighbor나 골라서 chunk를 보내는데 이때 random하게 걸린 이웃은 ***optimistically unchoked*** 되었다고 한다.
이렇게 함으로써 Best 4를 유지하면서 다른 괜찮은 후보를 탐색하는 greedy한 구조가 탄생했다. 위에 설명한 5명을 제외한 다른 Peer는 *choked* 되었다고 말한다. 수연이한테 chunk를 받지 않는다는 뜻이다.

즉 많이 기여하는 사람에게 많이 돌아오는 incentive mechanism이고, 그래서 tit-for-tat이라고도 불린다.
![](https://i.imgur.com/xNzHJx8.png)

## 2.6 Video Streaming and Content Distribution Networks
현대의 스트리밍 서비스가 cache와 비슷한 방법으로 application-level protocol과 server에 의해 돌아간다는 것을 자세히 알아보도록 해보자.

### 2.6.1 Internet Video
비디오 스트리밍에서 medium은 prerecorded video이다. 서버에 prerecorded video가 존재하고 유저가 요청을 보내는 방식이다.
일단 비디오는 일정한 rate로 재생되는 image sequence이다. 이미지는 pixel 단위로 정보가 인코딩 되어 있다. 비디오는 압축 가능하기 때문에 video quality ~ bit rate 사이의 trade off가 발생한다.
비디오 스트리밍에서 가장 중요한 것은 바로 높은 bit rate이다. 그로 인해 비디오 스트리밍의 performance를 잴 때 average end-to-end throughput을 기준으로 사용하게 된다.

### 2.6.2 HTTP Streaming and DASH
유저가 비디오를 보고 싶으면 TCP connection이 생겨서 HTTP GET으로 요청을 보내고 서버가 비디오 파일을 보내게 된다. 그 HTTP Response 내에 비디오 파일이 들어 있다.
**클라이언트 관점에서 살펴보자.**  그 비디오 데이터를 buffer에 모으고, 모인 양이 threshold를 넘기면, application이 재생을 시작한다. Application은 buffer에서 비디오 프레임을 가져와서 압축을 풀고, 화면에 비디오를 display한다. 즉 비디오를 수신하고 버퍼에 쌓으면서 동시에 재생하는 방식으로 이루어진다.

>[!info] GPT
>이러한 관점에서 **buffer**는 **데이터를 임시로 저장하는 공간**을 의미합니다. 즉, 스트리밍 중에는 비디오 파일이 서버로부터 클라이언트로 **연속적으로 전송**되는데, 클라이언트는 이 데이터를 한꺼번에 처리하는 것이 아니라 **순차적으로 처리**할 수 있도록 먼저 **버퍼(buffer)**에 저장해둡니다.
**역할:**
• **데이터 일시 저장**: 네트워크를 통해 받은 비디오 데이터를 일정량 모아 둡니다.
• **끊김 없는 재생**: 비디오 재생 중간에 데이터가 부족해 화면이 끊기지 않도록 **데이터를 미리 확보**하는 역할을 합니다. 즉, 네트워크 상황에 따라 전송 속도가 느려져도 **미리 저장된 데이터**를 사용해 재생을 이어갈 수 있습니다.
• **동기화**: 데이터를 받은 즉시 재생하지 않고, **일정량의 데이터를 쌓은 후 재생**함으로써 재생 속도와 전송 속도의 차이를 완충합니다.
결론적으로, **buffer**는 **비디오 데이터를 수신하는 동안 재생이 끊기지 않도록 일정량의 데이터를 미리 저장해두는 임시 저장 공간**입니다.

그러나 이런 방식에는 치명적인 단점이 있는데, 바로 서버에서 클라이언트로 가는 bandwidth는 천차만별인데 반해 같은 방식으로 인코딩된 데이터를 전송한다는 것이다. 
-> **Dynamic Adaptive Streaming over HTTP(DASH)**, 비디오를 다른 방식으로 인코딩 한다.
즉 bandwidth가 높으면 청크를 크게, 낮으면 청크를 작게 선택하는 것이다.
### 2.6.3 Content Distribution Networks
비디오 스트리밍을 하는 기업에게 가장 직관적이고 단순한 방법은 큰 데이터 센터를 짓는 것이다.
그러나 다음과 같은 문제가 있다.
1. 데이터 센터에서 멀리 있는 클라이언트는? 만약 멀리 떨어진 ISP가 throughput이 낮으면 딜레이 발생
2. 인기가 많은 비디오는 여러번 전송된다. 
3. 데이터 센터가 하나기 때문에 이 서버가 다운되면 전체가 다운된다.
그래서 대부분의 기업에서 **Content-Distribution Networks(CDN)** 을 사용한다.
지리적으로 떨어진 곳에 비디오 복사본을 두고 유저에게 최적의 경험을 주는 CDN을 연결시킨다.
CDN을 어디에 배치할 지에 대한 두 가지 철학이 존재한다.

1. Enter Deep: 수천 개의 작은 서버 클러스터를 배치하여 사용자와 가까운 위치에서 콘텐츠를 제공
   지연 시간이 줄고 속도가 빠르지만 관리 어려움
2. Bring Home: 적은 수의 대규모 클러스터를 배치한다. Enter Deep과 장단점 당연히 반대!

#### CDN Operation
브라우저가 어떤 비디오를 가져오라고 요청했을 때, CDN은 request에 개입하여 (1) 적당한 CDN server cluster를 결정해야 하고 (2) 클라이언트의 request를 cluster로 redirect 해줘야 한다. 
그리고 interception과 redirection에는 DNS가 사용된다.
1. 사용자가 NetCinema 웹 페이지에 접속한다.
2. 사용자가 http://video.netcinema.com/6Y7B23V 링크를 클릭하면, 사용자의 호스트가 video.netcinema.com에 대한 DNS query를 보낸다.
3. 사용자의 Local DNS Server(LDNS)는 NetCinema의 **authoritative DNS server**로 query를 전달한다. NetCinema의 DNS server는 hostname에 있는 “video” 문자열을 인식하고, IP 주소 대신 **KingCDN**의 도메인 이름(**a1105.kingcdn.com**)을 반환하여 DNS query를 KingCDN으로 넘긴다.
4. 이후 DNS query는 KingCDN의 private DNS system으로 전달되고, LDNS는 **a1105.kingcdn.com**에 대한 두 번째 query를 보낸다. KingCDN의 DNS system은 최종적으로 LDNS에 KingCDN content server의 IP 주소를 반환하며, 해당 server에서 사용자는 콘텐츠를 받게 된다.
5. LDNS는 content-serving CDN node의 IP 주소를 사용자의 호스트로 전달한다.
6. 클라이언트가 KingCDN content server의 IP 주소를 받으면, 해당 IP 주소로 **직접 TCP connection**을 설정하고 **HTTP GET request**를 보내 비디오를 요청한다. **DASH**가 사용될 경우, 서버는 먼저 클라이언트에게 각 비디오 버전마다 하나씩 포함된 URL 목록이 있는 **manifest file**을 보내고, 클라이언트는 다양한 버전 중에서 동적으로 chunk를 선택한다.
![](https://i.imgur.com/qzFUP1O.png)

#### Cluster Selection Strategies
이제 어떻게 cluster를 선택하는지 알아보자.
간단한 전략은 지리적으로 가까운 cluster를 클라이언트에게 할당해주는 것이다.
다른 방법은 clusters와 클라이언트의 real-time measurements를 수행하는 것이다.

### 2.6.4 Case Studies
TBD...
