# Ch 1. Computer Networks and the Internet
## 1.1 What is the Internet?
### 1.1.1 A Nuts-and-Bolts Description
인터넷은 세계의 computing device를 연결해주는 것이다.
세상이 발전하면서 computing device의 종류도 확장되었는데, 이를 host/end system이라고 부른다. end system 끼리는 communication link / packet switch로 연결됨.
sending end system이 데이터를 segment하고 거기에 header byte를 segmentwise하게 붙인다. 
**이러한 결과로 생긴 정보 package를 packet이라고 부른다**. computer network jargon!
destination에서 reassemble 한다.

packet switched network는 교차로, 고속도로 수송 체계와 비슷하다.

비유 
항구에서 수하물을 트럭 여러개에 옮겨 담는다. 트럭은 고속도로, 교차로를 지나 목적지에 도착한다.
**packet=트럭, communication link=고속도로,packet switch=교차로(intersection).**

Transmission Control Protocol; TCP
Internet Protocol; IP

### 1.1.2 A Services Description
이메일이나 웹 서핑 스트리밍 p2p 등등의 application은 distributed application이라 불리운다. 
내가 distributed application 하나 만든다면?
먼저 소프트웨어를 짜야된다. 자바나 파이썬 C 등..
그리고 그것들이 different end system 위에서 돌아가야 한다. 
인터넷을 application이 돌아가는 platform으로 보아야 하는 것.
그러면 어떻게 한 end sys가 다른 end system에게 데이터를 전달하도록 internet에 지시할 수 있는가?

**Answer: Application Programming Interface; API -> Ch2**

A가 B에게 편지를 쓴다. 바로 B의 창문에다가 던질 수는 없는 노릇노릇.
그 대신 편지를 봉투에 싸서 주소 수취인 우편번호를 적고 봉투를 sealing하고 우표 붙여서 우체통에 넣는다. 

우체국은 its own postal service API를 가지고 있다. 이것이 Internet API와 유사하다.

### 1.1.3 What Is a Protocol?
>_protocols_ _define_ _format_, _order_ _of_ _messages sent and received_ _among_
_network entities, and_ _actions taken_ _on message transmission, receipt._
### 1.2 the Network Edge
end system은 Internet의 edge에 있기 때문에 그렇게 불리운다.
host라고 불리는 이유는 그들이 application program(웹브라우저, 이메일 리더기,  서버 등)을 host=run 하기 때문이다.
host는 client와 server로 나뉜다.
### 1.2.1 Client and Server Programs
networking software의 맥락에서 정의해보자.
client program: host에서 돌아가는 프로그램인데 server program 에 요청하고 서비스를 받아오는 프로그램.
### 1.2.2 Access Networks
Access network는 엔드 시스템을 멀리 있는 엔드 시스템과 연결하기 위해 first router(=edge router)와 연결해주는 physical link이다.
#### Home Access: DSL, cable, FTTH, Dial-up, Satellite
#### Access in the Enterprise(and the home): Ethernet, WiFi

### 1.2.3 Physical Media
#### Twisted-Pair Copper Wire
#### Coaxial Cable
#### Fiber Optics
#### Terrestrial Radio Channels
#### Satellite Radio Channels

## 1.3 The Network Core
## 1.3.1 Packet Switching

#### Store-and-Forward Transmission
![[Pasted image 20240919124716.png]]
message를 보내기 위해 source에서 long message를 작은 chunk 단위로 쪼개는데, 이를 **packet**이라고 부른다. packet은 communication link와 packet switch를 타고 여행을 떠난다.
패킷이 라우터에 도달하자 마자 목적지에 전송하는 것이 아니라, 패킷을 먼저 저장한다. (buffer, **store**)
그리고 모든 bit가 수신되었을 때 패킷을 outbound link로 전송(**forward**)한다.
이를 **Store-and-forward packet switching** 이라고 한다.
소스-라우터에서 L/R, 라우터-목적지에서 L/R 
총 2L/R의 시간이 든다. 왜 라우터에서 저장되는지는 1.4에서 TBD..
라우터가 N-1개, 즉 N개의 link가 있다면 NL/R 만큼의 시간 경과.
위 시간들이 store and forward delay

#### Queuing Delays and Packet Loss

그리고 패킷 스위치에는 패킷을 전달하기 전 패킷을 저장해놓는 output **buffer**(=queue)가 있는데, link가 busy할 경우 자신의 차례가 올 때 까지 기다려야 한다.
이를 output buffer **queuing delay**라고 한다.
만약 buffer가 가득 차 있다면 **packet loss**가 발생하여, 굴러 들어온 돌이 빠지거나 박힌 돌이 빠진다.

#### Forwarding Tables and Routing Protocols

근데 어떻게 라우터가 패킷을 어디로 보낼지 알까?
인터넷에서 모든 host는 IP 주소가 있고 목적지의 IP 주소를 패킷의 header에 담아 보낸다.
각 라우터들은 forwarding table이 있어서 목적지의 주소(일부만 취사 선택도 가능)와 라우터의 outbound link를 mapping할 수 있다.

End2End routing process를 더 잘 알기 위해서, **길길이 물어서 목적지를 찾아가는 여행자를 상상해보라.**

근데 이 테이블 누가 만드는데? CH4... 잠깐 흘리자면 routing protocol에 의해 자동적으로 table이 set 된다.

### 1.3.2 Circuit Switching

써킷 스위칭은 전화선 교환 같은거다.
식당에 방문하는걸로 비유하자면 패킷 스위칭은 그냥 무지성 방문(웨이팅이 존재할 수 있음)
써킷 스위칭은 예약제 운영 식당이다.
서킷 스위칭 발식에서 네트워크는 송신자와 수신자 사이의 연결을 gurantee해주며 그 연결을 telephony jargon으로 circuit이라고 부른다.
써킷은 constant transmission rate를 보장한다.
#### Multiplexing in Circuit-Switched Networks
Circuit in a link는 frequency-division multiplexing(FDM) or time-division multiplexing(TDM)으로 구현된다.
FDM의 경우 link가 connection별로 frequency band를 특정 기간 동안 dedicate해준다.
**놀랍게도 이 band의 width를 bandwidth라고 부른다....!!!!!**
TDM의 경우 time이 고정된 간격의 프레임으로 나뉘어져 있다. 네트워크가 connection을 establish 하면 네트워크가 time slot을 dedicate해준다.
예를 들어 640000bit를 A에서 B로 전달한다고 해보자. TDM을 사용하고 24개 slot, 1.536Mbps의 bit rate를 가진다고 하자. 또한 end2end circuit을 estabilish 하는데 500ms가 걸린다고 하자. 
A가 파일 전송을 시작했다. 그렇다면 각 circuit은 1.536/24 = 64kbps의 transmission rate를 갖고, 따라서 640000/64 + 500 = 10500ms가 소요된다.
#### Packet Switching Versus Circuit Switching
packet switching의 지지자들은 circuit switching이 쉬는 동안, 즉 silent period 동안 dedicated circuit이 낭비된다는 점을 지적했다. 또한 end2end circuit을 구현하는 것이 복잡하고 cost가 많이 든다는 점도 지적한다.
아무튼 패킷 스위칭은 서킷 스위칭보다 낫다
1. better sharing of transmission capacity 
2. simpler, more efficient, less costly to implement

### 1.3.3 A Network of Networks
access ISP끼리 연결된다 = network of networks
이러한 변화는 경제 및 정책에 의해 바뀐 거지 퍼포먼스 측면에서 주도된 것이 아님.
##### Structure 1
mesh마냥 ISP끼리 다 연결
그러면 global transit ISP가 생김
근데 이러면 global을 만든 회사에서 수익을 독식하기 때문에...
##### Structure 2
다른 기업들도 global ISP를 만듬
이 여러 개의 global끼리 연결되어 있음
access ISP들은 상황에 맞게 global을 취사 선택하게 되는데, 이게 Regional ISP 개념.
regional ISP가 access ISP를 tier-1(=global, e.g. AT&T)에 연결 시켜줌.

##### Structure 3
regional ISP에도 크고 작은 우열이 생김.
중국은 각 도시에 access ISP가 있고 그것이 provincial ISP에, 그것이 national ISP에 연결됨.

##### Structure 4
PoPs, multi-homing, peering, IXP를 추가하면 Structure 4

##### Structure 5
구글같은 Content-provider network가 추가되면 현대의 network structure 5가 된다!


![](https://i.imgur.com/3WEvixi.png)
# 1.4 Delay, Loss, And Throughput in Packet Switched Networks
## 1.4.1 Overview of Delay in Packet-Switched Networks 
딜레이의 종류에는 
1. processing delay
2. queuing delay
3. transmission delay
4. propagation delay
가 있다. 이를 합쳐서 total nodal delay라고 한다.
![](https://i.imgur.com/ihGFJpb.png)


![](https://i.imgur.com/Z2OAf55.png)
#### Processing delay
패킷의 헤더를 검사하고 어디로 패킷을 보내야 하는지 결정하는데 걸리는 시간을 Processing delay라고 한다.
bit-level error를 체크하는 다른 요인들도 포함된다. 대부분은 microsecond의 order를 갖는다.

#### Queuing delay
패킷이 링크로 전송되기 위해 대기하는데 걸리는 시간을 queuing delay라고 한다.
queuing delay의 길이는 일찍 도착한 packet의 수에 영향을 받는다.
대부분 microsecond ~ milisecond의 order를 갖는다.

#### Transmission delay
패킷의 모든 bits를 링크로 push(=transmit)하는데 걸리는 시간의 양을 transmission delay라고 한다.
만약 Transmission rate R이 주어지면 transmission delay는 L/R 이다.

#### Propagation delay
링크의 시작점에서 목적 라우터까지 propagate하는데 걸리는 시간을 propagation delay라고 한다.
propagation speed는 physical medium의 영향을 받는다. 광속의 66%~99% 정도이다. 만약 라우터 사이의 거리가 d이고 propagation speed가 s면 d/s가 propagation delay이다.

**톨게이트 비유**
![](https://i.imgur.com/1oFOdbm.png)
car = bit, caravan = packet, toll booth = router, highway segment = link.

톨게이트를 통과하는데 걸리는 시간이 transmission delay이고
톨게이트에서 다른 톨게이트로 가는데 걸리는 시간이 propagation delay이다.

만약 톨게이트가 일을 못해서, 카라반의 일부가 이전 톨게이트에 있는데 카라반 머리가 이미 목적 라우터에 도착했다면?
그런 상황도 충분히 존재 가능하다.

### 1.4.2 Queuing Delay and Packet Loss
nodal delay의 가장 복잡한 부분은 queuing delay이다. 그래서 관련 논문도 가장 많다.
queuing delay는 패킷마다 다를 수 있다. 예를 들면 빈 큐에 10개의 패킷이 한번에 도착하면, 첫 번째는 queuing delay가 없고 마지막은 9번 기다려야 하는 상황이 해당된다.
그렇기 때문에 queuing delay를 분석할 때는 statistical measure를 사용한다(average, variance, or the probability of queuing delay)

queuing delay는 traffic이 주기적으로 딱딱 도착하는지, 아니면 한번에 와다다 오는지에 따라 다르다.
Transmission delay를 R이라 하고 모든 패킷이 L bits 로 구성된다고 하자. queue에 packet이 도착하는 average rate를 a라고 하고, queue가 무한하다고 가정해보자. 
(사실 큐는 무한할 수 없기 때문에 패킷이 손실, drop 된다.)
이때 La/R을 **Traffic Intensity** 라고 한다.
- 1보다 크면 큐에 도착하는 속도가 큐를 빠져나가는 속도를 초과해서, queuing delay가 끊임없이 늘어난다.

>[!important] 
>Design your system so that the traffic intensit is no greater than 1.

- 1보다 작거나 같은 경우
  1. packet arrives periodically
     모든 패킷이 빈 큐에 재깍재깍 오기 때문에 queuing delay가 없다
2. packet arrives in bursts but periodically
   상당히 큰 average queuing delay가 발생.
   첫 번째는 딜레이 없고, 두번째는 L/R, 세번째는 2L/R, ...

위에서 분석한 경우도 사실 이상적인 경우이지 현실에서는 packet이 random order로 도착하기 때문에 traffic intensity만으로는 충분히 분석하기 어렵다.

![](https://i.imgur.com/5LPVbsN.png)

다만 위와 같은 직관적인 관점은 제공해준다.
### 1.4.3 End-to-End delay

지금까지 single router에서 발생하는 delay인 nodal delay만 봤었는데 이제 src to dest total delay를 보자.
source host와 destination host 사이에 N - 1 개의 router가 있다고 해보자.
각 router에서 발생하는 processing delay는 d_proc 이라고 하고 transmission rate = R, packet size = L, d_prop은 propagation delay를 말한다.
![](https://i.imgur.com/acEPL0R.png)
d_trans = L/R 이다.


### 1.4.4 Throughput in Computer Networks
Host A에서 Host B로 큰 파일을 전달하는 상황을 생각해보자.
이때 B가 파일을 받는 순간 속도를 instantaneous throughput이라고 한다. 파일을 다운로드 받을 때 interface에 나오는 속도가 바로 이것이다.
만약 파일이 F bits이고 T초가 걸린다면 F/T가 average throughput이다.
![](https://i.imgur.com/07fFAzx.png)

위와 같은 상황을 살펴보자.
Rs는 서버와 라우터, Rc는 라우터와 클라이언트 사이의 communication link를 의미한다.
여기서 throughput 계산을 어떻게 할까?
대답하기 전에 먼저 bit를 fluid, communication link를 pipe로 생각해보자.
1. Rs < Rc
서버에서 보내는 패킷이 client 쪽으로 흘러서 Rs의 속도로 도착할 것이다. 
2. Rs > Rc
라우터에서 들어오는 것 만큼 내보내지를 못하기 때문에 라우터에서 at most Rc의 속도로 송신한다. 클라이언트의 waiting이 길어지면서 backlog를 계속 보내는 악순환이 발생할 수 있다. 
어쨌든 두 경우 모두 throughput = min(Rs, Rc)이다. 이 throughput은 **bottleneck link**의 transmission rate이다.
![](https://i.imgur.com/2ZPDxKq.png)
이제 이런 경우를 보자.
마찬가지로 throughput은 min{R1, R2, ..., RN} 일 것이다.
![](https://i.imgur.com/pQ4oKK2.png)

위 그림의 a와 같은 경우에서 음영진 부분, 즉 computer network에서의 속도는 access link의 rate보다 훨씬 빠르기 때문에 access link rate가 throughput을 결정하는데 중요한 factor로 작용한다.
**즉 throughput = min(Rc, Rs)이다.**

>[!note] GPT
>1. **Access Link**: 사용자가 네트워크에 연결하는 경로(예: Wi-Fi, 이더넷)로, 데이터를 송수신할 수 있게 해줌.
>2. **라우터 (Router)**: 네트워크 간 데이터를 전달하고, IP 주소를 기반으로 최적의 경로를 찾아 패킷을 전송하는 중계자 역할을 함.
>3. **네트워크 (Network)**: 장치들이 연결되어 데이터를 주고받는 시스템으로, LAN, WAN, 인터넷과 같은 종류가 있음.

b 같은 경우에 link의 transmission link rate를 R이라고 하자.
만약 R이 충분히 크다면 throughput은 이번에도 min(Rc, Rs)일 것이다.
근데 만약 R이 Rs, Rc와 같은 order에 있다면??

구체적 예시를 통해 알아보자.
Rs = 2Mbps, Rc = 1MBps, R = 5Mbps 라고 하자. 10개의 download가 존재한다.
그럼 이제 bottleneck은 access network가 아니라 shared communication link에 존재한다. 10개로 나뉘므로 500Kbps의 throughput을 갖는다. 따라서 end2end throughput은 최솟값인 500Kbps가 된다.

## 1.5 Protocol Layers and Their Service Models
## 1.5.1 Layered Architecture
![](https://i.imgur.com/DEtgwti.png)

인터넷의 Architecture는 매우 복잡하다. 마치 우리가 공항에 가서 비행기를 타는 과정과 같이 말이다. 
우리가 비행기 표를 사서 수하물을 보내고 게이트를 통과하고 비행기에 탑승해서 이륙하고 목적지에 routing 된 후 과정을 거꾸로 한 번 반복한다.
중요한 것은 위 과정이 horizontal manner로 수행된다는 것이다. 
![](https://i.imgur.com/pbv9PCu.png)

좀 더 과감하게 말하면 각 단계에서의 functionality(service)가 layer로 나뉘어져 있다고 할 수 있다.
**각 layer에서는 어떤 역할이 존재하고, 바로 아래 단계에서의 서비스를 사용한다.**
이러한 방식의 장점은 특정 layer를 수정하는 일이 발생해도 전체 system을 수정할 필요 없이 그 layer의 implementation만 손봐주면 된다는 것이다.
#### Protocol Layering
각 protocol 또한 layer 수준에서 구현된다.

이러한 방식의 예상되는 단점은
1. 수행하는 기능이 겹칠 수 있고
2. 한 layer에서 요구하는 정보가 다른 layer에 있을 때 separation 철학에 위배된다
는 것이다. 
![](https://i.imgur.com/7Q5i6VY.png)
>When taken together, the protocols of the various layers are called the **protocol stack**. The Internet protocol stack consists of five layers: the physical, link, network, transport, and application layers, as shown in Figure 1.23. If you examine the Table of Contents, you will see that we have roughly organized this book using the layers of the Internet protocol stack. We take a **top-down approach**, first covering the application layer and then proceeding downward.

#### Application Layer
>The application layer is where network applications and their application-layer protocols reside.

HTTP, SMTP, FTP, DNS 등이 application layer protocol에 해당한다.
end system의 application은 protocol을 이용하여 packets of information을 다른 end system의 application과 교환하는데, 이때 **정보를 담은 패킷을 message**라고 부를 것이다.
#### Transport Layer
>The Internet’s transport layer transports application-layer messages between application endpoints.

인터넷에는 TCP와 UDP가 존재한다. 
**transport layer의 packet을 segment**라고 부를 것이다.
#### Network Layer
>The Internet’s network layer is responsible for moving network-layer packets known as **datagrams** from one host to another.

맥락상 **Network layer에서 packet을 datagram**으로 부르는 것 같다.

마치 우리가 우체국에 목적지가 적힌 편지를 주듯이, transport layer protocol은 segment와 destination address를 network layer에 전달한다.
Network layer의 protocol은 오직 하나, IP protocol만이 존재한다. IP layer라고도 부른다.
#### Link Layer
>The Internet’s network layer routes a datagram through a series of routers between the source and destination.

Network element 사이에서 data transfer를 담당한다.
packet을 한 노드에서 다른 노드(라우터나 host)로 옮길 때 network layer는 link layer에 의존한다.
Ethernet, WiFi 등이 해당된다.
**link layer packet을 frame**이라고 부를 것이다.
#### Physical Layer
>While the job of the link layer is to move entire frames from one network element to an adjacent network element, the job of the physical layer is to move the individual bits within the frame from one node to the next.

### 1.5.2 Encapsulation
![](https://i.imgur.com/871KR3l.png)
link layer의 switch와 router는 모두 packet switch 방식이다.

Source host에서 application layer의 message가 transport layer로 전달 되고 앞에 transport layer의 header를 붙인다. 이는 수신자의 transport layer에서 사용될 것이다. 헤더가 붙은게 segment가 되고, 따라서 **segment는 message를 encapsulate 하고 있다.** 여기에 network layer header를 붙인다. end system address 같은게 저장되어 있고, 이것은 datagram이 된다. datagram은 link layer header가 붙어서 frame이 된다.
즉 각 layer에서 packet은 header와 payload로 구성된다.
비유를 들어보자.
Alice가 Bob에게 메모(Application layer의 message)를 보내려고 한다. Alice는 메모를 부서 이름과 함께 봉투에 넣어 보내는데, 이 봉투는 Transport layer의 segment에 해당한다. 그 후 회사 우편실에서 이 봉투를 또 다른 봉투에 넣고, 송수신 부서의 주소를 적어 공공 우편 서비스로 보낸다. 이 두 번째 봉투는 network layer의 datagram을 의미한다. Bob이 속한 부서 우편실에서 이 봉투를 열고, Bob에게 메모를 전달해주는 과정이 이어진다.

## 1.6 Networks Under Attack

malware라는 고의적으로 나쁘게 만든 소프트웨어가 device에 침투해서 나쁜짓을 할 수 있다. 대부분 self-replicating이어서 한 host가 감염되면 다른 host에게 전파를 한다.
DoS(Denial of Service)는 서비스를 unusable하게 만드는 공격이다. 세가지 카테고리가 있다.
1. Vulnerability attack
   시프에서 배운 그거..
2. Bandwidth flooding
   attacker가 packet을 많이 보내서 access link를 마비시키고 유저들이 서버에 접근하지 못하게 한다.
3. Connection flooding
   attacker가 TCP connection을 target host에 많이 만든다. 그래서 새로운 connection이 형성되지 못하게 하는 방법.

bandwidth flooding에서, access rate가 R인데 이게 충분히 크면 single source attack으로는 마비시키기 어려울 뿐더러 single source를 router가 감지하고 block해버리면 실패한다.
그래서 DDoS(Distributed DoS)에서는 여러개의 source에서 DoS 공격을 해서 방어하기 어렵게 만든다.

중간에 packet을 훔쳐서 정보를 유출시킬 수도 있다!
취약점이 생기는 곳에 passive receiver를 두면 걔가 packet을 복사해둘 수 있다. 그걸 **packet sniffer**라고 부른다.

사실 임의의 주소, content를 갖는 packet을 만들어서 아무데나 보내는 것은 매우 쉽다.
false source address를 가진 packet을 inject하는 것을 **IP spoofing** 이라고 부른다.
이게 성공하려면 end-point authentication, 즉 수신한 packet이 예상한 지점에서 온 packet인 척 하는 방법이 필요하다.

## 1.7 History of Computer Networking and the Internet
### 1.7.1 The Development of Packet Switching, 1961-1972
1960년으로 돌아가보자. telephone network가 지배적이였던 시절.
이맘때 컴퓨터의 중요성이 대두되면서, 자연스럽게 컴퓨터끼리 연결할 수 있는 지에 대한 논의가 형성되었을 것이다. packet switching이 발명되기 시작했다. 첫번째로 ARPAnet이 나왔다. 처음으로 end2end protocol이 사용 가능해지며 application을 작성할 수 있는 토대가 되었다.

### 1.7.2 Proprietary Networks and Internetworking, 1972-1980
초기 ARPAnet은 single, closed network 였다. 여기에 시간이 지나며 다른 stand-alone packet switching network가 추가되기 시작했다. 네트워크가 자라고 있는 것이다. 그 네트워크들을 엮은 네트워크를 만들려는 시도가 Cerf & Kahn에 의해 수행되었고, **이 work를 묘사하기 위해 *internetting*이라는 용어가 만들어지게 된 것이다.**
이 아키텍쳐는 TCP로 구현되었다. 초기 TCP는 지금과는 다른게, IP의 forwarding function도 가지고 있었다.  그러나 reliablility, flow control 등의 이유로 현재의 TCP와 IP로 분리되었고 UDP의 개발로 이어졌다.
ALOHAnet이라는 packet-based radio network가 만들어졌다.

### 1.7.3 A Proliferation of Networks, 1980-1990
1970년 말까지 200여개의 host가 연결되었다. 1980년 말에는 백만명에 이른다. 1980년은 인터넷의 급성장기라고 볼만하다. 1980년대에는 무슨 일이 있었던 걸까?
그러한 대부분의 성장은 university를 link하려는 노력의 결과이다.

### 1.7.4 The Internet Explosion, 1990s
1990년대의 메인 이벤트는 World Wide Web의 출현이다. HTML, HTTP, Web server, browser의 개념이 등장하였다. 그 위에서 4개의 killer application도 출현했다.
- 이메일
- 웹
- Instant message
- p2p sharing of MP3

### 1.7.5 The New Millennium
