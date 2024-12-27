# Ch 3. Transport Layer
## 3.1 Introduction and Transport Layer Services
logical communication: application 관점에서 host가 연결된 것 처럼 보이는 것.
message -> segment -> datagram의 encapsulation을 거친다.
중요한 것은 router는 network layer에서 datagram에 대해서만 작동한다는 것이다!
### 3.1.1 Relationship Between Transport and Network Layers
먼저 network protocol과 transport layer protocol에 대한 차이를 이해할 필요가 있다.
예시를 통해 알아보자. 2개의 집에 각각 12명의 아이들이 산다고 해보자. 
12명의 아이들이 서로 편지를 보내서 총 144개의 연결이 생긴다. 가계마다 한 명씩 편지를 모아서 나눠주는 사람이 있다. 수연과 상면.
수연은 12개의 편지를 걷어서 우체부에게 전달한다. 또 편지가 오면 아이들에게 나눠주는 역할도 한다.
여기서 우체국은 두개의 house에 대한 logical communication을 제공한다.
수연과 상면은 12명의 아이들 사이에 logical communication을 제공한다.

- 편지 = application messages
- 아이들 = process
- 집 = hosts, end systems
- **수연, 상면 = Transport Layer Protocol**
- **우체국/우체부 = Network Layer Protocol**

위 예시에서 알 수 있듯이 수연과 상면은 우체국 소속이 아니다.
**즉 Transport layer protocol은 end system 내부에 존재한다.**
만약 수연과 상면 대신에 연수와 면상이가 와서 같은 일을 할 수도 있는 것이다.
수연과 상면이랑 똑같이 일을 할 필요는 없지만 유사한 방식으로!

수연이랑 상면이가 우체국이 편지를 언제까지 보내겠다는 얘기를 안해주는 이상 수연이랑 상면이 입장에서 편지가 얼마나 지연될 지 알 방법은 없다.
**마찬가지로 Transport layer protocol은 network layer protocol에 의해 constrained 된다.**

### 3.1.2 Overview of the Transport Layer in the Internet
User Datagram Protocol, Transmission Control Protocol. 2가지가 존재한다.
요것들에 대해 알아보기 전에 먼저 network layer에서의 용어를 몇 개 아라보자.
Network layer protocol은 IP, Internet Protocol을 가지는데, 얘는 host 사이에 logical communication을 제공한다. 
IP는 **best-effort delivery service**라고 불린다. 왜냐면 segment를 host에 전달하기 위해 최선을 다하지만 data integrity나 order 등을 보장하지는 않기 때문이다, 그래서 **unreliable service**라고도 불린다.

이제 TCP/UDP에 대해 알아보자. 얘네가 하는 가장 대표적인 일은 바로 **host간 delivery service인 IP protocol을 end system 내의 process간의 delivery service로 연장해주는 것이다.**
이를 **transport layer multiplexing/demultiplexing** 이라고 부른다.
그리고 추가적으로 integrity check도 해준다.

위 두가지 최소 기능만을 제공하는 protocol이 바로 UDP이다.
그래서 IP처럼 UDP 또한 unreliable service이다.

그러나 TCP는 reliable data transfer과 congestion control을 제공한다.
전자는 앞 챕터에서 이미 설명했고, 후자를 대충 설명하자면 host 사이의 링크와 라우터를 과부하 상태로 만들지 않도록 방지하는 것이다. 반면 UDP는 application이 원하는 속도와 양으로 자유롭게 트래픽을 보낼 수 있다.

앞으로 우리는 일반적인 이야기를 먼저 하고, TCP가 그걸 어떻게 구현했는지 알아볼 것이다.

## 3.2 Multiplexing and Demultiplexing
Host에서 Transport layer는 network layer에서 segment를 수신한다. 그리고 socket으로부터 network가 process에게 데이터를 전달하며 역으로 데이터를 전달한다. Transport layer는 실제로 데이터를 직접 프로세스로 전달하는 것이 아니라 중간 매개체인 socket에게 전달한다. 그리고 socket은 unique identifier를 갖는다.
![](https://i.imgur.com/hv6KO7V.png)

Receiving end에서 Transport layer는 수신하는 segment의 header field를 identify하고 해당 segment를 목적지 socket으로 전달하는데, 이를 **demultiflexing**이라고 부른다.
source host의 socket으로부터 데이터를 모아서 각 데이터를 encapsulation하여 segment를 만들고, network layer로 보내는 과정을 **multiplexing**이라고 한다.

위에서 설명한 예시를 다시 가져와서 설명해보자. 수연이 우체부에게 편지 묶음을 받으면 수연이는 우편물이 누구에게 온건지 보고 직접 나눠줌으로써 demultiplexing을 수행한다. 반대로 상면이 우편물을 모으고 우체부에게 전달하는 것은 multiplexing에 해당된다.

Transport layer의 multiplexing에는 다음과 같은 두 가지 요구 사항이 있다.
1. Socket have unique identifiers
2. Each segment have special fields indicate the socket to which the segment is to be delivered.
이러한 Field들은 Source/Destination port number field이다. (UDP와 TCP에서 다르다.)
![](https://i.imgur.com/l2YFRZ6.png)

각 port number는 2^16, 즉 0~65535까지 16비트 정수이며 0~1023까지는 well-known port number라고 해서 사용을 엄격하게 제한하고 있다. 새 application을 개발할 때 port number를 반드시 명시해야 한다.
이제 Transport layer에서, host의 각 socket은 port number를 할당받고, segment가 host에 도착했을 때 Header field를 검사하고 대응되는 socket으로 segment를 보내게 된다. 그러면 데이터가 process로 성공적으로 전달된다. 이는 UDP의 기본 방식이고 TCP는 조금 더 미묘하다.

#### Connectionless Muxing and Demuxing
`clientSocket = socket(AF_INET, SOCK_DGRAM)` 의 명령어를 통해 port number가 자동으로 할당된다. 그러나 
`clientSocket.bind(('', 56123))` 의 명령어를 통해 특정 port number를 할당할 수 있다.

일반적으로 application의 서버 측이 특정 port number를 할당한다면 client측에서는 Transport layer에서 자동으로 port number를 할당한다.

![](https://i.imgur.com/LJpLFlO.png)

예를 들어서 UDP socket 19157을 가진 host A에서 socket 46428을 가진 host B로 데이터 전송을 하려 한다. A의 Transport layer는 헤더에 source port number, destination port number 및 다른 2가지 정보를 담고 network layer에 전달되어 datagram으로 encapsulation된다. host B에 도착하면 segment의 field를 identify하여 demuxing해줌으로써 원하는 Process로 데이터가 전달된다. source의 port number는 B에서 A로 다시 데이터를 보낼 때 사용된다.
**UDP에서는 Source가 달라도 Destination port number이 같으면 같은 socket으로 전달된다.**
#### Connection-Oriented Muxing and Demuxing
TCP socket은 Source IP address, Source port number, Destination IP / port 4가지 요소들에 의해서 식별된다. 그래서 Network layer에서 segment가 도착하면 host는 socket으로 demuxing 하기 위해 4개의 값을 모두 사용한다.
**UDP와 다르게 다른 src에서 출발한 segment가 같은 destination port number로 보내져도 다른 socket으로 direct된다!**

TCP Server는 welcoming socket을 갖는다. connection establishment를 기다린다. port num 12000
TCP client는 socket을 만들고 서버에 welcoming socket과 connection을 만든다.
`clientSocket = socket(AF_INET, SOCK_STREAM)`
`clientSocket.connect((serverName,12000))`

Connection Establishment request는 dest port num 12000과 특별한 연결 설정 비트(3.5에서 후술)를 헤더에 가지는 TCP segment를 말하는 것 뿐이다.

Server에서 dest port num이 12000인 segment를 수신하면 그 segment를 connection-estabilshment 를 waiting하고 있는 서버 프로세스로 보낸다.
`connectionSocket, addr = serverSocket.accept()`

TCP에서 각 socket은 process에 접속되어 있으며 4개 요소들로 이루어진 tuple로 식별된다.
![](https://i.imgur.com/Mh1X2Yp.png)

host C가 HTTP session 2개, host A가 HTTP session 1개를 여는 과정이 위 그림에 나타나있다.
C와 A에서 같은 port로부터 segment를 보낼 수도 있는데, 어차피 source의 IP가 저장되어 있기 때문에 demuxing을 정상적으로 수행할 수 있다.

#### Web Servers and TCP
Apache web server가 80번에서 돌아가고 있는 host를 상상해보자. initial conneciton-estabilishment segment와 HTTP request를 전달하는 segment는 destination 80을 가질 것이다. 그러면 수신하는 server측에서는 IP address로 구분할 것이다.
사실! process와 socket이 one-to-one corresponding 하는 것은 아니다. Threading이 가능하다...

## 3.3 Connectionless Transport: UDP
아주 no-frillgkrh bare-bone한 Transport protocol을 설계한다고 가정해보자. 아무것도 안하는 걸 먼저 떠올릴 것이다. 예컨대 application이 바로 network layer에 쏴주는 것과 같이 말이다. 
근데 예전에도 배웠지만 그거보다 뭔가 더 하긴 해야한다. **최소한 mux/demux service를 제공해서 network layer와 application process 사이에 데이터를 잘 주고받아야 할 것이다.**
UDP는 mux/demuxing과 약간의 error checking을 제외하면 IP에 아무것도 하지 않는다.
UDP로 application을 짠다는 것은 바로 IP와 상호작용하겠다는 의미이다.
UDP는 목적지의 port number로 segment를 보내는데 handshaking process가 없다. 즉 connectionless.

DNS는 UDP를 사용하는 application의 전형적인 예시이다. 

사람들이 왜 UDP를 사용할까?
- application level에서 data control을 정교하게 할 수 있다.
  TCP는 congestion control mechanism 때문에 데이터 송수신을 조절하는데 UDP는 즉시 network layer로 전달할 수 있다.
- No connection estabilishment
  TCP는 3-way handshaking process가 있다...
- No connection state
  TCP는 connection state가 존재하는데 이는 buffer, congestion-control parameter, acknowledgement number params 등을 포함한다.
- Small packet header overhead
  TCP는 20바이트의 header overhead, UDP는 8바이트.

![](https://i.imgur.com/ySJAzIq.png)

HTTP/3처럼 자체적인 congestion control과 error control(application level에서의)을 가지고 있으면서 UDP를 이용하는 경우도 있다.
Reliable Data Transfer가 중요한 application - email, remote terminal access, file transfer 등의 경우 TCP를 사용한다.
Real-Time application의 경우 TCP의 congestion control에서 성능이 매우 구려서 UDP를 사용한다.

대부분의 Multimedia application의 경우 UDP를 사용하고 congestion control이 없기 때문에 누군가 high bit rate의 작업을 수행한다면 엄청난 packet overflow가 발생할 수 있다.

UDP도 Reliable data transfer가 가능하다, application level에서 북 치고 장구 친다면.
TCP의 congestion control에 의한 transmission-rate constraint 없이도...!! reliable하게 가능!!

### 3.3.1 UDP Segment Structure

![](https://i.imgur.com/UmdYDC7.png)

### 3.3.2 UDP Checksum
error detection을 수행하는 역할을 한다. 즉 UDP segment가 전송되는 과정에서 bit가 바뀌지 않았는지 검사할 때 참고가 되는 기준점이다.

segment내에 있는 모든 16비트 word를 다 더해서 1's complement를 적용하고 그 값을 checksum으로 사용한다.
덧셈 과정에서 발생하는 overflow는 올려버리고, MSB에서 발생하면 wraparound하여 LSB로 내려서 더해버린다.
이제 수신자가 데이터를 제대로 받았다면, sum과 checksum의 합은 1111111111111111이 될 것이다. 하나라도 0이 된다면 에러가 발생했다는 사실을 알 수 있다.

![](https://i.imgur.com/qbgz45B.png)


link layer protocol(Ethernet)에서도 error checking을 하는데 왜 UDP도 할까? 
어떤 link는 error check를 안하는 경우도 있고, 더 중요한건 error가 router의 메모리에 저장되어 있을 때 발생할 수 있다는 점이다!
위 2개의 error 발생 가능성이 없다고 guarentee되지 않으면 우리는 end2end로 error detection을 수행해야만 한다. 그걸 **end-end principle**이라고 부른단다.

## 3.4 Principles of Reliable Data Transfer
General한 Reliable Data Transfer 개념에 대해 다룰 것이다. Reliable Data Transfer는 Transport layer 뿐만 아니라, link layer와 application layer에서도 일어나는 일이기 때문에 여기서 한 번 짚고 넘어가는 것이 좋다. networking에서 매우 중요한 문제 top10에 들법한 문제이다.

![](https://i.imgur.com/hJPlKuM.png)

위 그림에 나와있는 abstraction을 보면 upper layer로 전달되는 데이터는 reliable하게 transfer되는 것을 알 수 있다.
그렇다면 Reliable Data Transfer Protocol을 가진 layer 하단에서는 unreliable하다는 문제가 발생하기 때문에 매우 구현이 어려울 것이다. 실제로 TCP도 unreliable한 IP end2end network 위에 구현되어 있다.
우리는 lower layer에 대해 그냥 unreliable end2end underlying channel이라고 생각하기로 한다.
우리는 packet이 송신된 순서로 수신되고, 몇개는 유실될수도 있다고 가정한다.
**즉 underlying channel은 packet을 reordering하지는 못한다.**
3.8(b)는 우리가 구현할 protocol의 interface이다.
`rdt_send` (rdt는 Reliable Data Transfer)를 통해 data transfer protocol이 먼저 호출되고,
`rdt_rcv` 를 통해 packet을 receiving side에서 수신한다. `rdt` protocol이 데이터를 upper layer로 보내고 싶으면 `deliver_data` 를 호출한다.
`udt` 는 unreliable data transfer를 의미한다..!

### 3.4.1 Building a Reliable Data Transfer Protocol
#### Reliable Data Transfer over a Perfectly Reliable Channel: rdt1.0
underlying channel도 reliable한 경우가 가장 간단할 것이다. 이 경우에 FSM으로 나타낸 protocol 구현 방식을 `rdt1.0` 이라고 정의하자.

![](https://i.imgur.com/WyEIwo6.png)

**sending side와 receiving side에서의 FSM이 separate 되어 있다는 점을 주의하도록 하자.** 위 그림을 보면 알겠지만 state가 하나밖에 없다. 그래서 transition은 필연적으로 자신의 상태로 되돌아오게 되는 방향으로 일어나게 된다. 
Transition을 유발하는 event는 실선 위에 적혀있고, event가 발생했을 때 일어나는 action은 실선 아래에 적혀있다. action이나 event가 정의되어 있지 않은 경우에는 Λ로 표기하겠다. 그리고 점선이 가리키고 있는 state가 initial state가 된다.
##### Sending side
`rdt_send(data)` event가 발생하면 upper layer에서 data를 받아오고, data를 포함하는 packet을 만들고 packet을 channel로 보낸다. 보통 `rdt_send()` 는 application layer의 procedure로부터 호출된다.
##### Receiving side
`rdt_rcv(packet)` 을 통해 underlying channel에서 packet을 수신하고, packet에서 data만 추출한 뒤 upper layer로 data를 전달한다. 보통 `rdt_rcv()` 는 lower layer protocol의 procedure로부터 발생한다.

우리가 perfect reliable을 가정했기 때문에 receiver가 sender에게 역으로 feedback을 전달할 필요가 없고, 따라서 완벽한 unidirection(단방향) communication이 이루어진다. 그리고 우리는 receiver가 sender가 at most 송신하는 속도로 수신할 수 있다고 가정한다. receiver가 sender에게 속도 좀 낮추라고 할 이유가 없다는 뜻!

#### Reliable Data Transfer over a Channel with Bit Errors: rdt2.0
현실적으로 생각했을때 underlying channel에서 packet에서 error가 발생할 수 밖에 없다.. 왜냐면 network의 physical component 때문.
먼저 전화로 길게 말하는 상황을 한 번 상상해보자. 만약 듣는 사람이 맞게 들었다면 OK라고 할거고, 잘 못 들었으면 응? 이라고 할 것이다. **전자를 positive acknowledgement(ACK), 후자를 negative acknowledgement(NAK)라고 부른다.** 이렇게 재전송 기반으로 Reliablility를 구현하는 protocol을 **ARQ(Automatic Repeat reQuest) protocol**이라고 부른다. bit error를 다루기 위해 아래의 3가지 조건을 만족해야 한다.
- Error Detection
  근본적으로 에러가 발생했는지 파악하는 메커니즘이 필요하다. UDP에서는 checksum이 해당. ch6에서 자세히 다룰 예정이지만 여기서는 추가적으로 bit를 송신하면 된다는 사실만 알고 가자.
- Receiver Feedback
  Sender 입장에서 Receiver의 시선을 알기 위해서는 Receiver의 응답을 explicit하게 받는 방법 말고는 존재하지 않는다. ACK, NAK 답장이 그러한 feedback의 역할을 수행한다. 원칙적으로 NAK는 0, ACK는 1로 구분하면 되기 때문에 1비트만 할당하면 된다.
- Retransmission
  Receiver에서 Sender로 재전송되어야 한다.
  
![](https://i.imgur.com/dvwFDAe.png)

##### Sending side
2개의 state가 존재한다. 왼쪽 state는 data를 upper layer로부터 수신받기를 기다리고 있는 상태를 의미한다. `rdt_send` event가 발생하면 sender는 **checksum을 포함한** packet을 만들어서 `udt_send` 를 통해 lower layer로 packet을 보낸다. 보내고 나면 오른쪽 state로 넘어가고 ACK나 NAK를 기다리게 된다. 
`rdt_rcv` 를 통해 packet을 받았는데 ACK가 적혀있다면 receiver가 잘 받은 것이기 때문에 initial state로 되돌아가게 된다. 그러나 NAK가 적혀있다면 protocol은 sender에게 다시 보내라고 하면서 state를 유지한다.
중요한 것은 sender가 ACK/NAK를 기다리는 state에 있다면 upper layer로부터 어떤 데이터도 받지 못한다는 점이다. **즉 Receiver가 잘 받았다고 확인하기 전까지 새로운 data를 보내지 않게 되는데, 때문에 rdt2.0은 stop-and-wait protocol이라 불린다.**
##### Receiving side
여전히 1개의 state만 존재한다. packet이 도착하면 receiver는 ACK나 NAK로 답장해야 하는데, 이때 packet이 corrupt되었는지 확인해야 한다. corrupt가 발생하면 NAK를 포함한 패킷을 만들어 보내고, 아니면 ACK를 포함한 패킷을 보낸 뒤 data를 끄집어 와서 upper layer에 보내주면 된다.

rdt2.0에는 치명적인 결함이 존재한다. **바로 ACK와 NAK 또한 corrupt될 수 있다는 것!!** 
해결 방안은 다음과 같다.
1. 인간이 어떻게 하는지 상상해보자. OK와 응?을 못알아듣는다면 sender가 '뭐라고? 잘 안들려' 라고 할 것이다. 새로운 type의 packet을 만드는 것. 근데 그것도 못 알아들으면? 대체 어디서부터 못 알아 들은 건지 모르게 되면서 미궁 속으로 빠지게 된다...
2. Checksum을 여러개를 만들어서 에러가 발생했다는 사실을 detect할 뿐만 아니라 recover도 해내게 한다. packet corrupt이 발생하긴 하지만 그럼 어쩔건데? 난 많이 보냈어. 라는 마인드
3. **sender가 손상된 ACK/NAK를 받으면 data packet을 다시 보내게 하면 된다. 근데 받는사람 입장에서는 packet이 쌓여 있게 되고, 뭐가 맞는 data인지 알 길이 없다.**

3을 해결하기 위한 방법으로 **sequence number field**를 추가하면 되는데, 이걸 보면 receiver가 수신한 packet이 retransmission인지 아닌지 파악할 수 있다. 1비트 sequence number를 추가하고 retransmission 된거는 이전에 보낸거랑 같은 sequence num으로 설정하고 새로운거는 다르게 설정하면 된다. 이거 그냥 bit shift 연산을 적용하면 된다. 이것때문에 FSM이 약간 더 복잡해진다. 아래는 위 수정을 반영한 rdt2.1이다.

##### Sending side

![](https://i.imgur.com/1tpT4SJ.png)

rdt2.0이랑 같은데 sequence number가 flipping 되기 때문에 state가 2배 늘어났다.
0 보내고 1 보내고 0 보내고 1 보내고 ... sender는 그냥 flip-flop만 하면 된다. 0과 1을 만들어내는 주체. 검사는 receiver에서 한다. 그렇기 때문에 FSM에서 seq num이 0, 1인 경우가 거울대칭이다.
##### Receiving side

![](https://i.imgur.com/U7ZzpqA.png)

0 받고 1 받고 0 받고 1 받고 ...
0 받고 또 0 받거나 1 받고 또 1 받으면 그거는 재전송된 packet이 뒤에 있다는 의미니까 데이터를 받지는 않고 ACK만 sender에 전달해서 sequence number를 동기화 해주고 뒤에 있든 packet만 정상적으로 수신하게 된다.
#### rdt2.2
receiver가 NAK를 보내는 대신에 같은 packet에 대해 또 ACK를 보내게 되면 sender가 두 개의 duplicate ACK를 받게 되고 이로써 receiver가 뭔가 문제가 있구나 라는걸 간접적으로 알 수도 있다. NAK 없이 rdt를 구현한 것이 바로 rdt2.2가 된다.
##### Sending side
![](https://i.imgur.com/zY1lcNU.png)

sequence number가 함수의 인자로 들어가야 한다. 그리고 ACK가 2번 오면 데이터를 다시 보내준다.
##### receiving side
![](https://i.imgur.com/rBLvCYJ.png)


#### Reliable Data Transfer over a Lossy Channel with Bit Errors: rdt3.0
bit가 corrupt 되는 걸 넘어서 이제는 underlying channel에서 조차 packet loss가 발생할 수 있다고 가정해보자. 충분히 일어날 수 있는 일이다. 
1. 어떻게 detect하고
2. 어떻게 대처할 것인가?
어떻게 대처할 것인지는 rdt2.2에서 checksum, ACK packet, sequence num, retransmission 등으로 이미 구현되어 있다. 어떻게 detect할지만 고려하면 된다.

먼저 packet loss를 감지하고 recovery하는 역할을 sender에게 부여하자.
sender가 보낸 packet이 누락되거나 receiver가 보낸 packet이 누락된다면 sender는 reply를 계속 기다리는 입장이 된다. 너무 오랫동안 **기대하고 있는 packet response가 오지 않으면 '얘 분명히 중간에 누락된거야' 라고 확신하고 retransmission을 하면 될 것이다.**
이제 문제는 내가 기다릴 수 있는 시간을 어떻게 정해야 하는지로 옮겨간다. 당연히 RTT + receiver에서 process time보다는 더 기다려야 한다. maximum delay를 명확히 알기란 매우 어려운 문제이다 ...
그래서 실제로 sender가 신중하게 packet loss가 발생했을 것 같은 시간을 예측하는 방식을 사용한다.
ACK가 그 시간내에 도착하지 않는다면 packet은 retransmission 된다. 근데 만약 packet이 누락된 게 아니라 단순히 delay를 오래 겪는 거라면 문제가 발생하지 않았는데도 retransmit을 할 수 있다는 점이 마음에 걸린다.
**Duplicate data packet** 문제가 발생할 수 있는 것이다. 다행히 rdt2.2에서 이 문제를 handling 해두었다.
Sender 입장에서 retransmission은 만병통치제 같은 거다. **무슨 상황이 발생했는지(예컨대 내가 보낸 packet이 누락됐는지, ACK가 누락됐는지, 아니면 그냥 delay되고 있는지) 잘 모르겠으면 일단 재전송하고 보면 되기 때문이다.** 우리는 countdown timer 매커니즘을 도입하여 time expire를 관찰하고 재전송할지 말지 정하면 된다.
packet이 보내지면 timer를 돌리고, timer interrupt에 대응하고 적당히 stop시키는 방법을 구현해보자.

##### Sending side
![](https://i.imgur.com/O1mZ1KJ.png)

##### Receiving side
![](https://i.imgur.com/HYGmSjJ.png)

rdt3.0의 operation을 시간순으로 여러 경우에 대해 살펴보자.

![](https://i.imgur.com/dZHvWOs.png)

위에서 아래로 시간이 흐른다. b-d에서 send side의 대괄호는 timer가 켜고 꺼지는 시간을 나타낸다.
packet sequence number가 0과 1을 와리가리 하기 때문에 rdt3.0을 **alternating-bit protocol**이라고도 부른다.

이제 rdt의 핵심적인 요소들을 모두 공부했으니 실제로 돌려보자.

### 3.4.2 Pipelined Reliable Data Transfer Protocols
![](https://i.imgur.com/SxYFMTL.png)
rdt3.0은 기능적으로 잘 작동하는 좋은 protocol이지만 근본적으로 stop-and-wait 방식이기 때문에 속도 면에서 저평가당할 수 밖에 없다. 그 문제를 자세히 한 번 들여다 보자. 서울, 부산에 살고 있는 각 host가 존재하고 transmission rate가 R인 channel로 연결되어 있다고 하자. packet size는 L이다.

이 때 packet을 transmit하는데 걸리는 시간은 L/R이다. R을 1Gbps, L을 1000바이트라고 하면 8us가 된다. RTT는 30ms라고 가정해보자.

![](https://i.imgur.com/zQsesnH.png)

(a)에서 packet을 보내는 데에 RTT/2 + L/R 시간이 걸리기 때문에 15.008ms가 소요된다. ACK는 작아서 receiver가 받자마자 보낸다고 가정하면 sender에게 ACK가 도착하는데 걸리는 시간은 30.008ms이다. 그렇다면 전체 30.008ms동안 sender는 0.008ms만 보내는 셈이 되고, 이를 utilization 개념으로 정의한다면 sender의 utilization은 L/R(RTT + L/R) = 0.00027이 되어서 sender가 꿀빨고 있다는 사실을 수치로써 알려주는 것이다.. 다르게 본다면 1Gbps channel을 두고 267kbps만 보냈다는 말이다. 닭 잡는데 소 잡는 칼을 쓰는 격. **이 문제를 해결하려면 sender가 무작정 기다리고 있는 것이 아니라 3.7/3.8(b)에서 처럼 packet을 보내고 있어야 한다.** 많은 전송중인 packet이 pipeline을 채우는 것 처럼 시각화 될 수 있기 때문에 위와 같은 방법을 **pipelining이라고 부른다.**  pipelining을 도입하게 되면 아래와 같은 일들이 생긴다:
- sequence number의 범위를 늘려야 한다. 더 이상 0/1로는 구별되지 않음
- sender, receiver가 여러 개의 packet을 buffering 해야 한다.
- 새로운 error recovery 방법이 필요함: Go-Back-N, selective repeat.

### 3.4.3 Go-Back-N(GBN)
**GBN protocol에서 sender는 여러 개의 unACKed packet을 전송할 수 있다, 단 pipeline 내에서 허용가능한 최대 unacknowledged packet의 개수가 정해져 있다.**
[Animation](https://computerscience.unicam.it/marcantoni/reti/applet/GoBackProtocol/goback.html)
![](https://i.imgur.com/5o3Rnsp.png)
`base` 를 가장 오래된 unacknowledged packet의 sequence number로 정의한다.
그리고 `nextseqnum` 을 가장 작은 unused sequence number로 정의한다. 즉 다음에 할당 될 seqnum.
위 그림을 보면 sequence number는 4개의 구간으로 나눠진다.
1. \[0, base - 1]: 이미 전송되고 ack된 packet
2. \[base, nextseqnum - 1]: 전송되었지만 ack되지 않은 packet
3. \[nextseqnum, base + N - 1]: 즉시 전송될 가능성이 있는 packet
4. \[base + N, inf): packet that cannot be used.
Transmitted but not ack이 될 수 있는 범위의 sequence number를 window of size N으로 볼 수 있다.
window가 slide하는 느낌으로 protocol이 작동하고 이러한 이유 때문에 N을 window size라고 부르며 GBN을 **sliding-window protocol**이라고도 부른다.

![](https://i.imgur.com/MLMRzLi.png)

ACK-based, NAK-free, GBN protocol이 적용된 FSM이다. 이를 extended FSM이라고 부른다.
#### Sending side
GBN sender는 다음과 같은 3가지 type의 event에 대응해야 한다.
1. Invocation from above.
   `rdt_send` 가 호출되면 sender가 window가 가득 찼는지, 즉 N개의 outstanding unack packet이 있는지 확인해야 한다. 비어있는 공간이 있으면 packet을 만들고 variable을 업데이트 하고 가득 차있다면packet을 도로 올려보냄으로써 가득 차있다고 간접적으로 말해준다.
2. Receipt of an ACK.
   seqnum n을 갖는 ACK는 **cumulative acknowledgement**로 받는다, 즉 n까지 packet은 모두 receiver에게 잘 전달되었다는 것을 알려준다.
3. Timeout.
   timeout이 발생하면 sender는 보냈는데 ack가 돌아오지 않은 모든 packet에 대하여 재전송을 수행한다. ACK를 받았는데 더 받을게 남아있다면 timer가 restart되고, 더 받을 packet(=outstanding, unack)이 없다면 timer를 멈춘다.
#### Receiving side
seqnum n인 packet을 받고 그게 순서에 맞게 잘 온거라면 ACK를 보낸다. GBN protocol에서 receiver는 out-of-order packet을 무시해버린다. 순서만 틀리고 올바른 데이터를 왜 무시하냐고? 만약 n이 와야할 자리에 n+1이 오면 어떻게 해야할까? upper layer에 순서대로 보내야 하니까 n+1을 buffering 해놓고 n을 기다려야 할 것이다. 근데 n이 안오면? n과 n+1 둘 다 잃는 것이다. **그니까 아예 buffering을 하지 말고 out-of-order packet을 무시하자.** 
즉 receiver는 window의 upper/lower bound와 함께 **다음 도착 예상 seqnum(=`expectedseqnum` 을 들고 있어야 한다.** 

![](https://i.imgur.com/rVYIJ2H.png)

### 3.4.4 Selective Repeat(SR)
[Selective Repeat](https://computerscience.unicam.it/marcantoni/reti/applet/SelectiveRepeatProtocol/selRepProt.html)
GBN protocol의 문제점을 생각해보자. window size와 bandwidth-delay product가 모두 큰 상황에서 packet은 pipeline에 많이 채워질 수 있다. 근데 만약 하나라도 중간에 drop되서 receiver가 재전송을 요청한다면 많은 packet들을 재전송 해야하는 문제가 생긴다. **즉 single packet error가 large number of packet을 재전송하게 할 수 있는 것이다.**
그렇다면 자연스럽게 drop된 packet만 다시 보내게 하는 protocol을 개발해야 겠다는 생각이 든다. GBN과 마찬가지로 window size of N을 유지하되 **sender가 수신한 ACK를 window에 저장(buffer)해놓는다!**
SR receiver는 order에 관계 없이 correct하게 전송된 packet이라면 모두 ack하고 buffer해놓는다.

![](https://i.imgur.com/oNNKCjX.png)

![](https://i.imgur.com/HLLl4pf.png)

sender에서 주목할 점은 GBN과 다르게 모든 packet이 각자의 timer를 가진다는 것이다. 그게 individual ACK의 철학이다.
SR receiver의 2번째 step에 주목할 필요가 있다. receiver는 이미 ACK한 packet이더라도 reACK할 수 있다, window base보다 낮은 seqnum에 있더라도! 이게 꼭 필요한 일이라는 걸 이해해야 한다. sender와 receiver의 window가 항상 일치하지는 않을 거기 때문에(GBN은 back N 함으로써 강제 일치) ACK를 다시 보내줘야 sender에서 window를 옮길 수 있기 때문이다.

![](https://i.imgur.com/UfJbjw0.png)

구체적인 흐름은 위와 같다.

앞서 언급한 **lack of synchronization between sender and receiver windows는 finite seqnum에서 중요하다.** 예를 들어 보자.

![](https://i.imgur.com/xUT06kR.png)

seqnum이 2비트, 즉 4까지 있다고 가정해보자. 0, 1, 2를 잘 보내서 receiver의 window가 3, 0, 1로 이동했다. 그 뒤로 2가지 시나리오가 있는데, a에서는 ACK가 다 튕기고 timeout이 되어서 0, 1, 2를 전부 순서대로 재전송할 것이다. 즉 receiver는 0을 먼저 받게 될 것이다.
b에서는 ACK가 잘 보내졌고 sender의 window도 옮겨져서 다시 3, 0, 1을 보낼 것이다. pkt3이 drop되지만 새로운 pkt0은 잘 전송된다. 

**receiver의 관점에서 나에게 도착한 이 pkt0이 timeout되어 재전송된건지 새로운 건지 어떻게 알까?** sender와 receiver 사이에는 일종의 커튼이 쳐져 있어서 절대 sender가 어떤 입장인지 알 수 없다. 즉 receiver 입장에서 a와 b는 완전히 같은 상황이다. 이러한 일을 막기 위해 window size가 seqnum 크기보다 1 작게 해서는 안될 것이다.

![](https://i.imgur.com/xXyGZDz.png)

위는 우리가 지금까지 살펴본 rdt에 대한 요약이다.

## 3.5 Connection-Oriented Transport: TCP
### 3.5.1 The TCP Connection
TCP는 하나의 process가 다른 process에게 본격적으로 데이터를 보내기 전에 handshake 과정을 거쳐야 하기 때문에 **connection-oriented**라고 불린다.
TCP는 **full-duplex service**를 제공하는데, 만약 process A가 process B에게 data를 전송한다면 동시에 B에서 A로도 전송가능하다는 이야기이다.
그리고 1:1 communication만 가능하기에 **point-to-point**라고도 불린다.

TCP connection이 어떻게 establish 되는지 한 번 살펴보자. process A가 process B와 connection을 initiate하고 싶어 한다. initiate하고 싶어 하는 주체가 client, 당하는 쪽이 server라고 이전 챕터에서 정의했었다. client process는 먼저 client의 transport layer에게 server와 connection을 만들거라고 얘기한다.

>` clientSocket.connect((serverName,serverPort))`

그러면 client의 TCP가 server의 TCP와 TCP connection을 형성한다. client가 first special TCP segment를 보내고 server가 second special TCP segment로 대답하며 마지막으로 clinet가 third special segment를 보낸다. 이를 **three-way handshake**라고 부른다. 처음 2개의 segment는 payload가 없다(즉 no application data). 세번째는 있을 수도 있다.

![](https://i.imgur.com/B2dhTlG.png)

data가 door(socket)을 통해 나오면 TCP의 손에 들어가고, TCP는 그 데이터를 **send buffer**에 저장해놓는다. 특정 시간마다 TCP는 data를 받아와서 buffer에 저장하고, network layer로 내보낸다.
흥미로운 점은 TCP spec에서 buffer를 내보내는 시점은 명시하지 않고 있다는 사실이다. 확실한건 바로바로 비우지는 않는다. 

>The maximum amount of data that can be grabbed and placed in a segment is limited by the maximum segment size (MSS).

MSS를 결정할때는 MTU(Maximum Transmission Unit, link layer 수준에서 결정)를 먼저 정한 후 정한다.
MSS는 segment에 들어있는 application layer data의 최대 amount를 의미하는 것이지, TCP segment의 최대 크기를 의미하는 것이 아니다.

receiver또한 receive buffer가 존재하고, network layer에서 segment data만 추출하여 저장해놓는다. 그리고 application이 buffer에서 stream을 읽는다.

### 3.5.3 TCP Segment Structure
TCP segment는 header와 data field로 나누어져 있다. data field는 application data를 포함한다. 위에서 언급했다시피 MSS가 segment의 data field size를 제한한다. 예를 들어 고용량의 image를 전송할 때 file을 여러 개의 chunk로 나누어서 전송하게 된다. 
![](https://i.imgur.com/kjkph82.png)

- seqnum, acknum: rdt를 위해 사용
- receive window: flow control에 사용
- header length: TCP header의 길이. 기본적으로 20바이트
- flag field: 6bit.
  ACK는 segment가 성공적으로 수신한 packet에 대한 ACK를 담고 있는지 알려준다.
  RST, SYN, FIN은 connection setup과 teardown에 대한 것이다. 후술
  CWR, ECE는 explicit congestion notification에 관한 것, 후술
  PSH는 receiver가 data를 upper layer로 즉시 보내야 한다는 것을 의미한다.
  URG는 urgent marking이다.

#### Sequence Numbers and Acknowledgement Numbers
##### Sequence number
TCP는 data를 unstructed, ordered stream of bytes로 본다. 그러한 관점에서 segment의 sequence number는 segment의 첫 바이트의 byte-stream number이다.
![](https://i.imgur.com/J44r7gK.png)

예를 들어 위와 같은 500,000 바이트의 파일을 보내려 하는데 MSS가 1,000 바이트라면 segment를 500개로 나누어 보낼 것이고, 첫 번째 segment에는 0, 두 번째는 1000, 세 번째는 2,000의 sequence number가 할당 될 것이다.
##### Acknowledgement number
Acknowledgement number에 대한 정의는 다음과 같다.

>The acknowledgment number that Host A puts in its segment is the sequence number of the next byte Host A is expecting from Host B.

예를 들어 보자. A가 0 ~ 535라고 적힌 byte를 받았고 B에게 segment를 보내기 직전이다. A는 536을 기다리고 있다. 그래서 A는 B에게 packet을 보낼 때 536을 acknum으로 적어서 보낸다. **즉 내가 다음에 받아야 되는 sequence number를 전달해 주는 것이다.**
만약 A가 536을 기대하고 있는데 900~1000을 전달한다고 해보자. out-of-order segment가 발생한거고, 여기에 대해 TCP는 programmer에게 문제를 맡긴다. 아마 out-of-order를 무시하거나, 받지 못한 걸 기다리거나 하는 두 가지 대응 방법이 존재하겠다.

#### Telnet: A Case Study for Sequence and Acknowledgement Numbers
host A가 client, host B가 server 역할을 한다.
![](https://i.imgur.com/LJHWxDU.png)

initial sequence number는 각각 42와 79라고 하자. host A가 telnet에 'C'를 입력하고 커피 마시러 갔다.
client는 42를 보내면서 79를 받기를 기대한다.
server는 79를 보내며 43을 받기를 기대한다. 이 때 server는 두 가지 목적이 있는데, 하나는 client의 request에 대한 ACK이다. ACK를 43으로 설정함으로써 '43까지 너가 보낸거 다 잘 받았어^^'라는 의도를 담아 낸다.
다른 하나는 'C'를 echo back 하는 것이다. 데이터 전송의 역할. **데이터 전송에 ACK가 업혀가는(=piggyback) 느낌인데, 이 acknowledgement를 piggyback되었다고 부른다.**
마지막으로 client가 다시 43을 보내면서 80을 기대한다. 그냥 '어 서버 너가 보낸거 잘 받았어' 하는 의미다. 그래서 데이터가 들어있지 않다. 그럼에도 sequence number가 부여되어 있음에 주목하자. **모든 TCP segment는 sequence number를 가져야 한다.**

### 3.5.3 Round-Trip Time Estimation and Timeout
TCP도 우리가 앞서 다루었던 rdt protocol처럼 timeout/retransmission mechanism이 존재한다. 가장 중요한 질문은 Timeout interval을 어떻게 제시할 거냐는 것이다. 당연히 RTT보다는 커야 할텐데, RTT는 어떻게 measure 할 것인가.

#### Estimating the Round-Trip Time
`sampleRTT` 는 segment가 보내진 시간과 ACK를 수신한 시간 사이 간격이라고 하자. 매 segment마다 RTT를 재는 것이 아니고, 대부분의 TCP에서는 특정 시간에 오직 하나의 RTT measure를 가진다. 그리고 재전송된 segment에 대해서는 RTT를 계산하지 않는다.
당연하게도 sampleRTT는 router에서의 congestion이나 end system의 load에 의해 fluctuate 될 것이다. 그러므로 평균을 내는 것이 자연스럽다. TCP는 평균값을 유지하는데 이를 `EstimatedRTT` 라고 부른다. 새로운 sampleRTT를 얻으면 TCP는 EstimatedRTT를 update한다.

![](https://i.imgur.com/KKrdbqw.png)

가중된것. 보통 $\alpha$ 값은 $1/8$을 사용한다.
즉 `EstimatedRTT` 는 `SampleRTT` 의 가중평균이다. 위와 같은 방식을 사용하면 최근의 value에 더 많은 가중치를 두게 된다. 통계학에서 위와 같은 평균을 Exponential Weighted Moving Average(EWMA)라고 부른다. exponential이라는 단어는 sampleRTT의 weight가 exponential하게 decay하기 때문이다.

![](https://i.imgur.com/O2hKBSO.png)

실제로 위와 같은 관계를 갖는다.
sampleRTT의 variance도 비슷한 방식으로 measure할 수 있다.
![](https://i.imgur.com/HlRL1H0.png)

이 경우 $\beta$ 값으로 보통 0.25를 사용한다.

#### Setting and Managing the Retransmission Timeout Interval
EstimatedRTT와 DevRTT 중에 무엇을 Timeout interval로 사용해야 할까?
timeout interval EstimatedRTT보다 너무 커서는 안된다. 너무 크면 delay가 길어진다. 즉 EstimatedRTT보다 조금 크게 설정하는 것이 바람직하다. 그럼 얼마나 조금? 그건 fluctuation에 달려 있다. fluctuation이 크면 margin을 크게 잡고, 작으면 작게 잡으면 된다. fluctuation은 우리가 DevRTT로 measure 했으므로,

$$TimeoutInterval = EstimatedRTT + 4 \cdot DevRTT$$

위와 같이 Timeout interval을 정하면 된다. initial 값은 1초로 정하는게 추천된다. 그리고 timeout이 발생하면 Interval을 2배로 늘려서 ACK를 잘 받을 수 있는 segment가 timeout되는 상황을 방지한다.
### 3.5.4 Reliable Data Transfer
언급했지만 IP service는 unreliable하다. IP는 그냥 UDP 마냥 무책임하다. datagram이 router buffer에서 overflow가 날 수도 있고, 순서가 뒤죽박죽이 될 수도 있으며, bit corruption이 발생할 수도 있다. 당연히 network layer와 맞닿아 있는 Transport layer에서도 그 문제점을 고스란히 안고 갈 수 밖에 없다.
그래서 TCP는 IP의 unreliable service 위에서 reliable data transfer service를 제공하며 고군분투한다.
앞선 논의에서는 segment마다 각각의 timer가 존재했지만 여기서는 overhead를 줄이기 위해 하나의 timer만 사용하도록 하자. 먼저 간략한 설명을 보고, 자세히 다루어보자.

![](https://i.imgur.com/5SdmByv.png)

위와 같이 TCP sender가 동작한다. sender에게 중요한 이벤트는 세 개인데, application에서 data 받기; timeout; ACK receive이다.
ACK y를 받았다는 건 acumulative ack에 의해 y까지 다 받았다는 의미인데 sendbase가 y보다 작다면 지금 받은게 unacknowledge라는 것이므로 sendbase를 y로 정하고 아직 못 받은게 있다면 타이머를 돌린다.

#### A Few Interesting Scenarios
##### 1.
![](https://i.imgur.com/A4E6Kqh.png)

host B 입장에서는 똑같은거 2번 받는게 되니까 내용물 byte는 무시하고 ACK 한 번 더 보낸다.

##### 2.
![](https://i.imgur.com/595f4q6.png)

92에 대한 ACK가 timeout이 지나도 오지 않아서 timer reset을 하고 가장 작은 seq num인 92를 다시 보냈는데, ACK가 와버려서 host A가 100을 재전송 할 수가 없다. 120 ACK을 받은 순간 '너 119까지 다 잘 가지고 있구나' 하기 때문에..

##### 3.
![](https://i.imgur.com/XZoipJS.png)

여기서는 92, 100 두 개를 보내는 데 첫 번째가 유실된다. host A는 ACK가 120이 뜬걸 보고 '아! 119까지 다 받았구나!' 라고 생각하게 된다. 그래서 어떤 재전송도 일어나지 않게 된다.

**이제 위와 같은 문제를 다루기 위해 TCP modification을 해보자.**
#### Doubling the Timeout Interval
timeout이 발생하면 smallest seqnum을 다시 보내되, 재전송을 할때 마다 next timeout interval을 이전 값의 2배로 늘린다. 위에 수식으로 구하는 것이 아니고!
위 modification은 limited form of congestion control을 제공한다. 대부분의 timer expiration은 congestion에 의해 발생하기 때문.

#### Fast Retransmit
또 다른 문제는 timeout period가 길 때 발생한다. timeout period가 길면 sender가 packet을 재전송하는데 걸리는 delay가 길어지고 이는 end2end delay로 이어진다. 다행히 sender는 packet loss를 **duplicate ACKs**로 감지할 수 있는데, 이는 이미 ACK된 segment를 또 ACK 하는 것이다. 
sender가 dupACK에 어떻게 대응하는지 알려면 receiver가 왜 dupACK를 보내는지 먼저 알아야 한다.

![](https://i.imgur.com/JVvwLIp.png)

TCP receiver가 next expected seqnum보다 큰 값을 받으면 gap을 인지해서 missing segment가 있음을 파악한다. TCP는 NAK을 사용하지 않기 때문에 explicit하게 거절 의사를 내비칠 수 없고, 대신에 마지막에 받은 segment에 대해 dupACK을 송신함으로써 NAK을 implicit하게 표현한다.

![](https://i.imgur.com/rVengiK.png)

three dupACK를 받으면 sender는 **fast retransmit**을 수행한다. missing segment에 대해 timer가 만료되기 전에 빠르게 보낸다는 의미에서 fast가 붙었다.
세 번이나 receiver가 '나 100없으니까 100줘' 하면 얼른 100을 주는게 맞다.

![](https://i.imgur.com/RLVPQSZ.png)
![](https://i.imgur.com/AuC6YPs.png)

위 코드는 fast retransmit을 반영한 ACK receive event의 modification이다.

#### Go-Back-N or Selective Repeat?
3.33을 보면 TCP는 smallest seqnum에 대해서 sendbase와 nextseqnum을 유지한다. 이러한 관점에서 보면 GBN style로 보인다. 그러나 TCP는 out-of-ordered segment를 찝어서 잘 받는다(selective acknowledgement). GBN은 다 보내야 하는데!
그래서 GBN과 SR의 hybrid protocol이라고 보는게 정확하다.

### 3.5.5 Flow Control
receiving side에서 segment의 data가 buffer에 저장되는데, application이 항상 즉시 읽어오는 것이 아니다. 다른 일을 하느라 busy할 수 있기 때문에, 심지어 오랫동안 방치시켜 놓을 수도 있다. 만약 receiver가 느리게 buffer에서 data를 읽는데 sender가 너무 빨리 보낸다면 overflow가 발생할 것이다.
이러한 점을 방지하기 위해 TCP는 **flow-control service**를 제공하여 overflow 가능성을 제거한다. Flow control은 즉 sending rate와 receiving rate를 matching 시켜주는 service라고 보면 된다!
**앞서도 말했지만 IP network의 congestion 때문에 TCP sender가 throttle(속도 제한)이 걸릴 수 있는데, 이거는 congestion control이다. flow control이랑 비슷하지만 엄연히 다른 개념이기 때문에 구분해야 한다!**
TCP는 **sender가 receive window라는 변수를 유지**함으로써 receiver의 buffer 상태에 대한 정보를 파악하고 flow control을 수행한다. TCP는 full-duplex기 때문에 양 방향 sender 모두 receive window 정보를 유지하고 있다. 
A가 B에게 큰 파일을 보내는 과정을 생각해보자. B의 buffer size는 RcvBuffer이고, LastByteRead, LastByteRcvd 변수를 정의한다. 각각 process B가 마지막으로 읽는 데이터의 byte수, receive buffer B가 network로 부터 수신한 마지막 데이터의 byte수를 의미한다.
TCP는 buffer overflow를 허용하지 않기 때문에

![](https://i.imgur.com/y6DO7Jy.png)

위와 같은 관계식을 만족한다. 즉 receive window는 buffer의 여유공간을 의미한다. rwnd는 그리고 시간에 따라 dynamic하게 바뀌는 값이다.

![](https://i.imgur.com/vgGy9nE.png)

만약 B의 rwnd가 0이라면? B는 A에게 아무것도 보내지 않는다. 왜냐면 보내야 할 데이터도, ACK도 없기 때문. 그래서 buffer가 비워져도 A는 아무것도 모른다. 그래서 A는 1바이트짜리 data를 담은 작은 segment를 보내서 buffer가 비워졌는지 확인한다. 만약 ACK를 받으면 이제 nonzero rwnd 값을 다시 알게 된다.

[Flow Control](https://computerscience.unicam.it/marcantoni/reti/applet/FlowControl/flow.html)

UDP는 flow control이 없기 때문에 buffer overflow가 발생할 수 있다.

### 3.5.6 TCP Connection Management
여기서는 TCP connection이 establish/tear down 되는 과정을 자세히 살펴보자.
TCP connection estabilishment는 사용자가 느끼는 delay와 network attack의 원인이 될 수 있기 때문에 이해하는 것이 중요하다. 먼저 client가 server에 connection을 형성하고자 하는 상황을 생각해보자.

![](https://i.imgur.com/cXbvR1i.png)
#### Step 1
client side TCP에서 server side TCP에 special TCP segment를 보낸다. 여기에는 application-layer data가 존재하지 않는다. 그러나 SYN bit가 1로 set되어 있다. 그래서 이 segment는 **SYN segment**라고 불린다. 그리고 client는 random하게 initial seqnum(`client_isn`)을 뽑아서 sequence number field에 담아서 보낸다.

#### Step 2
Step 1에서 만든 datagram이 network layer를 통해 server에 도착하면 TCP SYN segment를 뽑아내고 TCP buffer와 variable을 할당하고, connection-granted segment를 client TCP에 보낸다. 이를 **SYNACK segment**라고 부른다. 얘도 마찬가지로 application-layer data가 없지만, 3가지 중요한 정보를 header에 담고 있다.  
- SYN bit is set to 1
- Acknowledgement field is `client_isn` + 1
- Server chooses its own `server_isn`
즉 '나 너가 보내준 `client_isn` 잘 받았어. 나 너랑 잘 해보고 싶어. 나는 `server_isn` 이라고 해!' 라고 말하는 것이다.

#### Step 3
client는 SYNACK을 받아서 buffer와 variable을 할당한다. 그리고 또다른 segment를 보내는데, ACK는 `server_isn` + 1로 설정하고 SYN은 0으로 설정해서 server에게 마지막으로 보내게 된다. 여기서는 이전과 다르게 application-layer data가 payload에 들어갈 수 있다.

위 3가지 Step 때문에 Connection-estabilishment procedure를 **three-way handshake**라고 부른다. 
등산가 비유를 기억하면 쉽다.

Connection을 끝내는 것도 마찬가지이다. connection이 끝나면 모든 resource들은 deallocate된다. 아래의 그림을 보자.
![](https://i.imgur.com/UrRZHnj.png)

client가 server에게 FIN bit가 설정된 special segment를 먼저 보낸다. server는 이걸 받고 ACK segment를 보내고, 이번에는 server가 FIN segment를 보낸다. 그리고 client가 ACK를 보내면 연결이 끊어진다.

TCP connection의 life에서 각 host는 TCP state들을 transition하게 된다. 아래에 그림을 보면 명확하다.

![](https://i.imgur.com/zUIrz3M.png)

client 관점이라는 것에 주의하자. 먼저 `CLOSED` 상태에서 TCP가 시작된다. SYN segment를 보내면 `SYN_SENT` 상태로 바뀌고, SYNACK를 받고 ACK를 보내면 `ESTABILISHED` 상태가 되며 handshaking이 끝난다. 여기서 이제 application data를 막 주고받고 우당탕탕. 그러다가 client가 이제 헤어지고 싶어지면 FIN segment를 보내서 `FIN_WAIT_1` 상태가 된다. 서버도 헤어지는데 동의한다는 사실, 즉 ACK를 기다린다. 서버에게 동의를 받으면 `FIN_WAIT_2` 상태가 된다. 이제 서버에게 이별 통보를 받기를 기다린다. 그게 `FIN_WAIT_2` 상태이다. 서버에게 FIN을 받으면 ACK를 보내고 `TIME_WAIT` 상태로 옮겨간다. 일종의 ACK 단말마를 보내는데 server에게 닿지 않고 유실된다. 미리 정해진 시간이 지나면 `CLOSED` 상태로 옮겨가며 다시 솔로가 된다. 이게 곧 TCP client의 수명이 된다.

![](https://i.imgur.com/U59kStf.png)

위 그림은 server의 관점이다.

만약 client가 포트 80으로 꼬셨는데 server가 웹 application이 돌아가고 있지 않은 상태라면? server가 RST flag가 담긴 segment를 보내서 '나 그거 못해, 더 보내지마' 라고 알려준다.

## 3.6 Principles of Congestion Control
전에도 말했다시피 network layer가 congest됨에 따라 router buffer에서 overflow가 발생하기 때문에 그걸 관리해주는 congestion control이 필요하다. 여기서는 general한 맥락에서 congestion control 문제를 다뤄보자. 왜 문제가 되는지, 어떻게 상위 레이어로 문제가 전염되는지. rdt와 마찬가지로 networking에서 top-ten 문제에 들어간다.
### 3.6.1 The Causes and the Costs of Congestion
#### Scenario 1: Two senders, a Router with Infinite Buffers
가장 간단한 경우부터 보자. 두 개의 host는 single hop으로 연결되어 있다.
![](https://i.imgur.com/Yj8CwnO.png)
host A가 $\lambda_{in}$ 의 평균 rate로 data를 보낸다고 하자. 이 데이터들은 socket으로 딱 한 번만 보내진다는 점에서 original이다. error recovery, flow control, congestion control 등 모든게 없는 transport layer를 통해 encapsulate 되어 보내진다. header overhead를 무시하면 라우터에서 host A가 느끼는 traffic은  $\lambda_{in}$ 이다. host B도 마찬가지로 $\lambda_{in}$으로 보낸다. host A와 B가 router를 공유하고 outgoing link capacity는 R이다. packet이 router에 도착하는 속도가 link capacity R을 넘기면 buffer에 저장이 되는데, 이 때 buffer size는 무한하다고 가정하자.
이 때 host A의 performance는 다음과 같다.
![](https://i.imgur.com/pBOo1Pj.png)
왼쪽은 per-connection throughput을 나타낸다. 
R/2가 되기 전까지는 input rate와 output rate가 같다. 그러나 R/2를 넘어가면 throughput은 R/2를 넘어갈 수 없다. 당연하다. 왜냐면 link를 2개의 host가 같이 쓰고 있기 때문. 아무리 빨리 보내도 R/2를 넘길 수 없다. 그리고 input rate가 R/2에 가까워질수록 Buffer에 쌓이는 packet의 수가 늘어남에 따라 delay가 기하급수적으로 늘어나게 된다. **즉 link capacity에 가까운 속도로 보내면 delay 관점에서 큰 손해가 발생한다.**

>Even in this (extremely) idealized scenario, we’ve already found one cost of a congested network—large queuing delays are experienced as the packet-arrival rate nears the link capacity.

#### Scenario 2: Two Senders and a Router with Finite Buffers
앞선 시나리오에서 몇 가지 수정을 해보자.
1. buffer size를 유한하게 제한한다 -> potential packet drop
2. connection is reliable -> restore dropped packet by retransmitting
![](https://i.imgur.com/0A6uuZ9.png)

retransmission을 고려한 rate는 $\lambda_{in}'$ 이라고 하자. **offered load**라고 불린다.
먼저 host A가 마법을 부려서 router buffer가 어떤 상태인지 안다고 가정하자. 이러면 packet loss가 절대 발생하지 않을 것이다. 그래서 $\lambda_{in}'$ = $\lambda_{in}$ 일 것이다.
![](https://i.imgur.com/FvEpEVu.png)
그게 a의 경우이다. 여기서도 R/2를 넘을 수 없다.
이제 좀 더 현실적인 케이스를 보자. sender가 packet이 손실된 것이 확실할 때만 재전송한다고 하자. 이 경우는 b에 해당된다.  $\lambda_{in}'$ 가 R/2 일 때 output rate는 R/3이 된다. 이게 뭔소리냐?
0.5R 만큼의 data가 전송된다면 0.333R 만큼이 original이고 0.166R 만큼이 retransmitted data라는 소리이다. 

>We see here another cost of a congested network; the sender must perform retransmissions in order to compensate for dropped (lost) packets due to buffer overflow.

마지막으로 router에서 queuing되고 있는데 timeout이 되어서 재송신하는 경우를 생각해보자. 이 경우에는 original data와 retransmission data가 모두 receiver에게 전달된다. 물론 receiver는 하나만 받고 나머지 하나는 무시해버리면 될 문제이다. 그러나 중복된 packet을 보내는 문제점이 발생한다. 

>Here then is yet another cost of a congested network—unneeded retransmissions by the sender in the face of large delays may cause a router to use its link bandwidth to forward unneeded copies of a packet.

3.46c에서 각 packet이 두 번 forward 되었을 때의 경우를 보여준다. 당연히 반으로 준다.

#### Scenario 3: Four Senders, Routers with Finite Buffers, and Multihop Paths
![](https://i.imgur.com/sk7BNdR.png)

위와 같은 경우를 보자. 먼저 A에서 C로 연결된 connection에 대해 살펴보자. R1을 D-B connection과 공유하고 R2를 B-D connection과 공유한다.
![](https://i.imgur.com/nmGx6T9.png)

(와 이건 나중에 복습 좀 하자... 진짜 모르겠네 이건)

>So here we see yet another cost of dropping a packet due to congestion—when a packet is dropped along a path, the transmission capacity that was used at each of the upstream links to forward that packet to the point at which it is dropped ends up having been wasted.

### 3.6.2 Approaches to Congestion Control
크게 보자면 network layer가 explicit assistance를 주는 지에 따라 나뉜다.
- End2End congestion control: no explicit assistance of network layer
- Network assisted congestion control

## 3.7 TCP Congestion Control
### 3.7.1 Classic TCP Congestion Control
TCP는 각 송신자가 throttle하게 함으로써 congestion control을 수행한다. 즉 congestion을 감지하면 transmission rate를 내리고 아니면 올리는 식으로. 근데 이 방법은 의문점이 있다.
1. 어떻게 throttle하고
2. 어떻게 congestion을 detect하며
3. Transmission rate를 조절하는 어떤 algorithm을 사용할 것인지

#### 1
sender측에서 동작하는 TCP congestion control mechanism은 **congestion window**라는 변수를 통해 관리한다.
![](https://i.imgur.com/aH2shGx.png)
rwnd에 의한 제약이 걸리지 않는다고 가정하면 unACKed data의 양은 cwnd에 의해 bound된다. 즉 매 RTT의 시작 때에 cwnd 만큼의 데이터를 전송할 수 있다.
#### 2
이제 어떻게 congestion을 감지하는지 보자. sender에서 loss event가 발생하였다고 가정하자. router에서 congestion이 발생하여 overflow가 발생하고 datagram drop이 발생한다. 이 drop event가 sender 측에서 timeout/three dupACK을 발생시켜서 감지하게 된다.

![](https://i.imgur.com/fnaohcD.png)
ACK를 통해 cwnd를 조절한다!

#### 3
TCP congestion control은 어떻게 transmission rate를 결정할까?
![](https://i.imgur.com/aOIF2tx.png)

위와 같은 principle을 바탕으로 
1. slow start 
2. congestion avoidance
3. fast recovery
총 3가지 요소를 갖는 **TCP Congestion Control Algorithm**을 알아보자.

#### Slow Start
일반적으로 cwnd는 1 MSS로 intialize된다. 그래서 초기 전송률은 MSS/RTT가 된다. 
ACK가 잘 오면 1, 2, 4, 8, ...으로 지수적으로 증가한다.
각 ACK가 올 때 마다 cwnd++를 해주는 방식.
![](https://i.imgur.com/DuBiZqU.png)

loss event by timeout이 있을 경우 cwnd를 초기화 하고 다시 시작한다.
그리고 `ssthresh` 라는 값을 정의한다. slow start threshold. cwnd/2(congestion 발생 시 값)로 정의한다.
그리고 `ssthresh` 보다 cwnd가 커지면 slow start mode를 종료하고 congestion avoidance mode로 전환된다.

#### Congestion Avoidance
이 단계에서 cwnd의 값은 congestion 문턱에서의 값의 절반이다. 그래서 다시 2배를 하는건 바보 짓이고 여기서는 1 MSS만큼 increment한다. 
이 경우도 timeout이 발생하면 cwnd를 1로 설정하고 ssthresh를 설정하며 slow start로 돌아간다.
그리고 3dupACK를 수신하면 ssthresh를 반으로 하고 fast recovery phase로 돌입한다.

#### Fast Recovery
cwnd 값은 lost segment에 의한 dupACK을 수신할 때 마다 1MSS씩 증가한다. 
마찬가지로 timeout이 발생하면 slow start mode로 돌아간다. 
이 단계는 필수는 아닌 것이 TCP Taeho는 3dupACK에서 slow start로 돌아간다.
TCP Reno는 fast Recovery를 채택한다.
![](https://i.imgur.com/YbQjgLf.png)

위 FSM을 보면 3개의 phase와 transition에 대해 잘 이해할 수 있을 것이다.

![](https://i.imgur.com/Umg64ZM.png)

위 그림에서 초기 thresh는 8MSS로 Tahoe와 Reno에서 동일하다. 8round 까지 동작이 동일하다.
4라운드에서 threshold를 넘기고 congestion avoidance로 넘어간다. 그리고 3dupACK을 수신할 때 까지 linear하게 올라간다. 8round에서 cwnd는 12이고, sthresh는 12 \*0.5 = 6 MSS 로 정해진다.
Reno는 fast recovery mode로 전환되고 Tahoe는 slow start로 전환된다.

#### TCP Congestion Control: Retrospective
TCP congestion control에서 초기 slow start 부분을 제외하면 1MSS씩 올리는 부분과 cwnd를 반으로 떨구는 부분이 반복적으로 나타나는 과정을 통해 congestion control을 수행한다.
![](https://i.imgur.com/MP3X1qr.png)

이러한 특성을 **additive-increase, multiplicative-decrease(AIMD)** form of congestion control이라고 부른다.

#### TCP Cubic
Reno의 AIMD probing(탐색)은 어떻게 보면 엄청 조심하는 방법이다. congestion avoidance 단계에서 pre-loss sending rate를 빠르게 복구하는 방법이 나을 수 있다(congested link state가 이전과 크게 변하지 않았다는 가정 하에). 이러한 최적화를 수행한 방식이 바로 TCP Cubic이다.
![](https://i.imgur.com/HlHMxZ3.png)
![](https://i.imgur.com/G083dZv.png)

#### Macroscopic Description of TCP Reno Throughput
톱니모양 behavior를 갖는 TCP Reno를 보면 평균 throughput이 얼마나 되는지 궁금해진다. 아래에 시행할 분석에서 slow start phase는 매우 짧기 때문에 무시하도록 하자. RTT 동안 TCP의 data sending rate는 congestion window와 current RTT의 함수이다. window size가 w일때 TCP transmission rate는
$w/RTT$ 가 된다. W를 loss event가 발생할 때의 w라고 하자. 그렇다면 TCP Transmisson rate의 범위는 
$W/(2\cdot RTT)...W/RTT$ 가 된다.
이때 packet drop이 W/RTT에서 발생하고 rate는 반으로 떨어진다. 이 과정이 계속 반복되고, linear하게 이루어 지기 때문에 평균 값은 딱 그 중간인 3/4 지점, 즉$$average\space throughput\space of \space a \space connection = 0.75 \cdot W / RTT$$가 된다.


### 3.7.2 Network-Assisted Explicit Congestion Notification and Delayed-based Congestion Control
#### Explicit Congestion Notification, ECN
network-assisted congestion control performed by Internet.
IP datagram header에 적혀있는 2개의 bit를 이용한다.
하나는 router가 congestion을 겪고 있다는 것을 알려주는 bit이다. marking된 datagram이 destination host에 도착하고 sending host에게 다시 알려준다.
![](https://i.imgur.com/iK5qj0S.png)
두번째는 router에게 sender와 receiver가 ECN-capable하다는 것을 알려주는 역할을 한다.

### 3.7.3 Fairness
>A congestion control mechanism is said to be fair if the average transmission rate of each connection is approximately R/K; that is, each connection gets an equal share of the link bandwidth.

2개의 TCP connection이 transmission rate R인 link를 공유하고 있는 상황을 생각해보자. 두 connection은 같은 MSS, RTT를 가지며 slow-start phase는 무시하자.
![](https://i.imgur.com/PfI95PI.png)

만약 link bandwidth를 2개의 connection이 equally 반으로 나눠가진다면, throughput은 정확히 45도 아래로 떨어질 것이다. 그리고 2개의 throughput의 합은 정확히 R이 되어야 한다. 하나가 모두 점유하는 광경은 보기 좋지 않으므로, 두개가 경합하며 equal bandwidth share line과 full bandwidth utilization line의 intersection에서 와리가리 할 것이다.

![](https://i.imgur.com/ZJFvbUP.png)

A에서 link bandwidth jointly consumed amount가 R보다 작으므로 loss가 발생하지 않고, cwnd를 늘릴 것이다. Congestion avoidance phase니까. 그래서 A에서 45도 기울기로 양의 방향으로 throughput이 증가할 것이다. 같은 양만큼! 그러다 결국 bandwidth consumption이 R보다 커지게 되고 packet loss가 발생 할 수 있게 되면서 cwnd가 반으로 줄어든다. 즉 B와 원점을 잇는 선의 중점이 다음 위치 C가 된다.
**즉 bandwidth share line을 fluctuate하다가 결국 intersection으로 converge 할 것임을 알 수 있다.**
따라서 TCP는 fairness를 만족한다.

