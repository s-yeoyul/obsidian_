# Ch 4. The Network Layer: Data Plane
## 4.1 Introduction
![](https://i.imgur.com/0s0dBO2.png)

H1과 H2를 연결하는 path 상에는 X표시된 router들이 많이 존재한다. H1이 H2에게 정보를 전달할 때, Network layer는 정확히 어떤 일을 하는 것일까?
먼저 H1의 transport layer가 network layer에게 segment를 전달하고, datagram을 근처에 있는 router R1에게 전달한다. H2는 근처의 R2에게 datagram을 받고 transport layer segment를 extract한다.
router의 역할은 input link에서 output link로 forwarding 하는 것이다.
### 4.1.1 Forwarding and Routing
- Forwarding: packet이 router의 input link에 도착하면 적절한 output link로 내보내준다. router-local action.
- Routing: network layer는 packet이 sender to receiver로 잘 흘러갈 수 있는 route/path를 정해줘야 한다. 이를 결정하는 알고리즘을 routing algorithm이라고 부른다. Network-wide process.
예를 들어 문경에서 서울까지 간다고 해보자. forwarding은 교차로를 지나는 것이다. routing은 문경에서 서울로 갈때 국도를 탈건지 고속도로를 탈건지, 어떤 고속도로를 어디서 갈아 탈건지 등등 거시적인 관점에서 경로를 정하는 것이다.

모든 router는 forwarding table이 존재한다. router는 packet header를 확인하고 그 값을 table에서 참조하여 outgoing link를 정한다.

![](https://i.imgur.com/yI1ezzl.png)

>In this example, a routing algorithm runs in each and every router and both forwarding and routing functions are contained within a router.

근데, forwarding table은 어떻게 configuration이 되는 것일까? 위 그림을 보면 알겠지만 routing algorithm이 forwarding table에 들어오는 값을 정한다. routing algorithm은 centralized or decentralized 되어 있다. 어떤 경우든 간에 router가 routing protocol message를 받아서 configuration을 진행한다.
수리공들이 라우터를 뜯어서 하나하나 설정해주는 상황을 생각해보자. 말도 안되지만 forwarding and routing function의 distinct / different purpose를 설명하기에는 적합하다. 그정도로 별개라는 뜻.
#### Control Plane: The SDN Approach

![](https://i.imgur.com/3sAEmXf.png)

원래는 router가 communicate하는 다른 router component가 정해져 있었는데, 최근에는 수동적으로 사람이 forwarding table을 configure할 수 있게 되었다. 즉 control-plane에서 data-plane을 결정하는 기능이 생겼다.
4.2와 비교했을 때 4.3의 특징은 Remote Controller의 존재이다. routring device는 forwarding만 수행하지만 remote controller는 forwarding table을 계산한다. 4.3에서의 control plane의 approach는 SDN, 즉 Software-defined networking의 정수이다.

### 4.1.2 Network Service Model
Transport layer를 다룰 때 의문이 들었던 부분들, 예를 들면 network layer에서 congestion에 대한 feedback을 제공하는가? 이런 문제들은 network layer에서 제공하는 service model에 따라 다르다.
- Guaranteed delivery: Destination host에 packet 도착을 guarantee
- Guaranteed delivery with bounded delay: 특정 시간 내에 도착을 guarantee
- In-order packet delivery: packet이 dest에 송신 순서대로 수신됨
- Guaranteed minimal bandwidth: 최소 transmission bit rate를 보장
- Security: Encrypt all datagram, and decrypt at the dest.

Internet의 network layer는 딱 하나의 서비스만 제공하고, 이는 best-effort service로 알려져 있다.

![](https://i.imgur.com/BWmW7qF.png)

사실 no service at all인데 보기 좋게 포장한 것일 뿐이다.

![](https://i.imgur.com/kvIdzYN.png)

## 4.2 What's Inside a Router?
generic router architecture에 대한 high-level view는 다음과 같다:

![](https://i.imgur.com/wKJf8Js.png)

- input ports: physical layer function(incoming physical link를 terminate)과 link layer function(다른 incoming link와 interpolate)을 수행. lookup function을 통해 forwarding table을 보고 어디로 나갈지 switching fabric으로 결정. 각 3개의 기능이 순서대로 left, middle, right box에 해당됨.
- switching fabric: router의 input port와 output port를 연결하는 역할을 수행. network inside a network
- output port: switching fabric을 지나온 packet을 저장해 놓고, 적절한 link-layer 및 physical layer function을 통해 transmit. link가 bidirectional 하다면 input port와 pair를 이룰 것이다.
- Routing processor: control-plane function을 수행한다. 이후 챕터에서 후술.

input/output port, switching fabric은 hardware로 implement되어 있다. 그래서 nanosecond scale로 도는데 control plane function은 software로 implement되어 있고 그래서 (milli)second scale로 돈다.
**정확히 말하면 nanosecond 단위에 scaling이 필요해서 hardware로 구현한 것이고, control plane은 그렇지 않으니까 software로 구현한 것이다.**

위에서 했던 비유에서, 교차로를 로터리로 바꿔서 생각해보자. 그렇게 될 경우 추가적으로 무엇을 고려해야 할까?
- Destionation-Based forwarding: 진입로에 서있는 차는 final destination에 기반하여 어떤 exit으로 나가야 할지 정해야 한다.
- Generalized forwarding: destination 외에도 다른 정보를 고려해야 한다. 이를테면 번호판에 따라 짝수로 끝나면 특정 exit으로 가야하고 홀수면 다른 exit으로 뭐 이런식...
 ![](https://i.imgur.com/30wmlwC.png)
![](https://i.imgur.com/WXpDek9.png)
![](https://i.imgur.com/YT2SN6t.png)

4.4를 이제 다시 보면 entry는 input port(with a lookup fn)에 대응된다;
roundabout은 switch fabric에 대응된다;
exit은 output port에 대응된다.
이 비유를 통해 알 수 있는 사실은 potential bottleneck이 roundabout에 존재한다는 것이다. 엄청 빨리 차들이 몰려와서 entry에 대기열이 길어지는데 로터리는 느리게 돈다면? 대부분의 차들이 같은 exit으로 나가려 한다면? 이러한 문제는 router에서도 똑같이 발생한다. 그래서 우리는 대응 방법을 배우게 된다.

### 4.2.1 Input Port Processing and Destination-Based Forwarding

![](https://i.imgur.com/flin69S.png)

Input port에서 일어나는 Lookup은 router의 핵심이다. forwarding table은 routing processor에 의해 compute되거나 remote SDN controler에게 제공받는다. forwarding table은 routing processsor가 separate bus에 line card를 실어 보내는 것에서 복사되어 온다. (4.4에서 점선)
어떻게 forwarding이 일어날까? 당연히 IP를 brute-force로 matching하지는 않을 것이다.
![](https://i.imgur.com/vSY5n0Y.png)

대신 이런 식으로 앞부분(prefix)이 matching되는지 확인함으로써 분류할 수 있다.
근데 2번째랑 3번째가 겹친다는걸 알 수 있고, 이럴 때는 router가 longest prefix matching rule을 적용하여 젤 긴거를 기준으로 삼는다.
간단해보이지만 lookup은 엄청 빨리, ns 단위로 일어나야 한다. 그래서 단순히 hardware 구현으로 될 게 아니라 추가적인 최적화가 더 필요하다. 
![](https://i.imgur.com/henOV4g.png)

그래서 TCAM 방법을 이용하는데, forwarding table에서 값을 상수시간에 찾아낼 수 있다.
![](https://i.imgur.com/rMWazLG.png)

### 4.2.2 Switching
Switching fabric이 바로 router의 heart이다. (심장이 대체 몇 개야 ㅜㅠ)
다양한 방법으로 switching이 이루어질 수 있는데, 다음과 같다:

![](https://i.imgur.com/mmAdC3Z.png)

1. switch by memory:
   초기에 간단한 router는 traditional computer 그 자체였다. 즉 switch도 CPU가 하던 일이었다. 즉 OS에서 I/O를 다루는 방식. packet이 들어오면 interrupt가 날아가고, packet이 메모리에 캐싱 된 뒤 processor가 dest를 찾아서 forward. 내보낼 때는 메모리에서 output buffer로 카피한다. 즉 읽고 쓰는 과정에 모두 bandwidth를 사용하기 때문에 최대 B개의 packet이 들어오면 throughput은 B/2보다 작다.
2. switch by bus:
   packet을 shared bus를 통해 direct로 output port로 forwarding 한다. 이는 routing processor가 개입하지 않는다. packet은 exit에 대한 정보를 담고 있는 label을 달고 bus에 탄다. bus에는 한 번에 한 packet만 이용할 수 있고, output port와 일치하는 label만 하차할 수 있다. 하차하면 label은 제거된다. bus에 한 packet만 탈 수 있기 때문에 bottleneck이 되어 다른 packet이 waiting을 해야 한다... 로터리에 한 자동차만 이용할 수 있는 경우와 같다. **bus bandwidth에 의해 switching speed limitation이 발생한다.**
3. switch by interconnection network:
   single shared bus의 문제점을 극복하려면 당연히 여러 개를 효율적으로 어떻게 보낼 지 생각해야 한다. multiple bus와 맥락이 같다. 2N개의 bus가 존재하여 N개의 input과 N개의 output을 연결한다. horizontal bus와 vertical bus는 서로 intersect하며, 교차점은 fabric controller에 의해 열고 닫힌다. A-Y와 B-X는 동시에 일어날 수 있다. 다른 bus를 사용하기 때문에. 이러한 성질을 **non-blocking**이라고 부른다. 다만 다른 input port에서 같은 output으로 가야하는 경우는 또 대기가 생긴다.

### 4.2.3 Output Port Processing
![](https://i.imgur.com/XNOGVdr.png)

output port의 memory에 저장되어있는 packet을 받아서 outgoing link로 보낸다. scheduling과 de-queuing, 그리고 link/physical layer function을 수행한다.
### 4.2.4 Where Does Queuing Occur?
4.6을 보면 packet queue는 input/output port에서 모두 발생할 수 있음을 알 수 있다. Queuing이 발생하는 위치와 정도는 traffic load, switching fabric의 relative speed, line speed 등에 영향을 받는다. queue가 grow하면서 router의 memory가 exhaust 되고, 결국 packet loss가 발생한다. **앞에서 "packets were lost within the network, dropped at a router" 라는 표현을 많이 사용했는데 여기가 바로 그 부분에 대한 설명이다.** 

>It is here, at these queues within a router, where such packets are actually dropped and lost.

만약 input/output line이 N개 있고 각각의 transmission rate가 $R_{line}$으로 같고, 모든 packet이 input port에 synchronously arrive한다고 가정하자. switching fabric rate가 $R_{switch}$ 이고 이게 $R_{line}$의 N배면, input port에는 queuing이 거의 발생하지 않는다. 왜냐면 worst case가 N개의 모든 input port가 packet을 수신하고, 모두 같은 output link로 나가는 경우인데 switch fabric이 다음 batch가 오기 전에 처리해버리기 때문이다.
#### Input Queuing
그러나 switch fabric이 그렇게 빠르지 않은 경우엔 어떨까? 그러면 packet이 switching fabric으로 transfer되기를 기다려야 하는 상황이 생긴다. crossbar switching fabric에서 2개의 input queue에서 하나의 output port로 forwarding해야 하는 상황을 생각해보자. 하나는 forward되어야 하고 하나는 block되어야 한다.
![](https://i.imgur.com/ho3D3Mk.png)

위 그림과 같은 상황에서 switch fabric이 왼쪽 위 queue를 먼저 처리하면 왼쪽 아래의 packet은 기다려야 한다. 그러면 왼쪽 아래 queue에서 대기중인 packet의 목적지는 contention이 없음에도 불구하고 덩달아 기다려야 하는 불합리한 상황이 발생하게 되는데, 이를 **head-of-the-line(HOL) blocking**이라고 부른다.
즉 대가리 때문에 기다려야 하는 것...!

#### Output Queuing
outgoing link에서 single packet을 보낼 때 N개의 새로운 packet들이 output port에 도착할 수도 있다. 그런데 한 link에는 하나의 packet만 처리될 수 있으므로 N개가 기다려야 한다. N개를 처리하는 동안 또 N개가 오고... 즉 switching fabric이 N배 빨라봐야 어차피 output port에서 queuing이 생긴다.

만약 memory가 추가적인 buffer를 하기에 모자르다면 새로 도착하는 packet을 버리거나 아니면 박힌 packet을 빼내야 한다. 사실 buffer가 가득 차기 전에 미리미리 drop해놓는 방식이 더 효율적이다. sender에게 congestion signal을 보낼 수 있기 때문. marking을 이용한 ECN 방식을 3.7.2에서 다뤘었다. AQM algorithm(packet drop, mark policy에 대한 알고리즘)의 일종. 가장 많이 연구되고 사용되는 AQM algorithm은 Random Early Detection(RED) algorithm이다.
![](https://i.imgur.com/qkHMlZE.png)

packet이 output port에서 queuing이 되고 있으면 queue에서 하나를 내보내게 된다. 매 packet time마다. 이는 output port의 packet scheduler가 정한다.

#### How Much Buffering is "Enough"?
buffering의 양이 RTT times link capacity여야 한다는 rule of thumbs가 있었다.
$$B = RTT \times C$$
가장 최근의 이론적, 실험적인 노력에 의하면 필요한 buffering의 양은 다음과 같다:
$$B = RTT \times C / \sqrt{N}$$ 여기서 N은 independent TCP flow를 의미한다. core network에서는 router의 속도가 매우 빠르고, 그래서 TCP flow가 엄청 많이 존재할 수 있는데, 이는 buffer size가 덜 중요하다는 의미가 된다.

buffer가 크면 클수록 더 좋다고 생각하는게 당연할 것 처럼 보인다. 그러나 buffer가 커지면 queuing delay 또한 커진다는 것을 의미한다.
buffering은 short-term fluctuations in traffic을 흡수 할 수 있지만, 동시에 delay increasing에 기여할 수 있는 양날의 검이다. 소금 같은 거다. 적당해야 안 짜고 맛있다.

### 4.2.5 Packet Scheduling
#### First-in-First-Out (FIFO)
![](https://i.imgur.com/vt7nd8U.png)

link가 다른 packet을 transmit하느라 busy한 경우 packet은 output queue에서 waiting을 시작한다. buffer 공간이 여유롭지 않은 경우 packet-discarding policy를 적용하여 어떤 packet을 drop할 지 정한다.
FIFO scheduling discipline에서는 link transmission을 시킬 packet을 도착 순서 기준으로 정한다. 
![](https://i.imgur.com/UoN1mZK.png)

#### Priority Queuing
output link에 도착하는 packet은 priority에 따라 분류된다. 
![](https://i.imgur.com/EE3woB3.png)
network operator가 priority를 configure 할 수 있다. 구현은 priority queue를 이용하는게 아니라 queue 여러 개를 이용하는 방식으로 이루어져 있다. priority가 높은 queue부터 먼저 비운다.
![](https://i.imgur.com/MnppDNn.png)
**non-preemptive priority queuing discipline**에서 packet이 일단 transmit되기 시작하면 절대 interrupt되지 않는다!
![](https://i.imgur.com/5GlstCb.png)

#### Round Robin and Weighted Fair Queuing (WFQ)
![](https://i.imgur.com/r0TPv7s.png)
round robin에서도 priority queuing과 마찬가지로 packet이 여러 class로 분류된다. 그러나 strict한 우선순위가 존재한다기 보다는 class들을 scheduler가 alternate한다고 보는게 맞다. 예를 들면 class 1 보내고 class 2보내고 다시 class 1 보내고... 뭐가 떠오르지 않나? context switching!!
**work-conserving queuing discipline**이라고도 불리는데, 여기서는 link를 절대 idle하게 내버려두지 않는다. (idle: 공회전) given class에 packet이 없으면 곧바로 다른 class를 찾아서 손해를 없앤다.
![](https://i.imgur.com/MW8dkgu.png)
한 class에서 packet을 보내면 다른 class로 alternate된다. 1 -> 3을 보면 알 수 있다.
그리고 class가 비어있으면 alternate하지 않는다. 3 -> 4를 보면 알 수 있다. 뭐 당연한 이야기.
이를 조금 더 일반화 한게 **Weighted fair queuing, WFQ discipline**이다.
![](https://i.imgur.com/AarN1Rr.png)
도착하는 packet는 class별로 분류되고 queuing되는데 round robin과 마찬가지로 class를 circulate한다. 
그러나 round robin과의 차이점은 각 class가 단위 시간당 부여되는 differential amount가 다르다는 것이다.
즉 어떤 class의 가중치가 $w_i$이면 그 class는 $w_i / (\sum w_j)$ 만큼 packet을 전송하는 것이 보장된다는 의미. 즉 link의 transmission rate가 R이면 throughput 또한 $R \times w_i / (\sum w_j)$ 이 최소한 보장된다.

## 4.3 The Internet Protocol (IP): IPv4, Addressing, IPv6, and More
현대에 사용되는 IP의 version은 크게 2가지이다. 하나는 IP protocol version 4, 주로 IPv4라고 불린다.
그리고 IPv4를 대체하기 위해 제안된 IPv6에 대해 다뤄볼 것 이다. 그리고 그 사이에 IP Addressing 에 대해 다룰 것이다.
### 4.3.1 IPv4 Datagram Format
IPv4에서 핵심 field는 다음과 같다.
![](https://i.imgur.com/exE4Lsj.png)
- Version number(4bit): IP protocol의 version을 명시한다. version을 보고 router는 나머지 datagram을 어떻게 해석할 지 판단한다.
- Header length(4bit): datagram의 header의 길이가 가변적이므로 payload의 정확한 범위를 계산하기 위해 사용된다.
- Type of service(TOS): datagram의 type을 명시한다. 예를 들면 real-time인지 아닌지...
- Datagram length: header와 payload를 합산한 총 길이.
- Identifier, flags, fragmentation offset: IP fragmentation과 관련이 있다. IPv6에서는 fragmentation을 지원하지 않는다.
- Time-to-live(TTL): datagram이 circulate forever하는 문제를 방지하기 위해 만료 시간을 설정. 매 라우터를 지날 때 마다 1씩 감소.
- Protocol: datagram이 destination에 도착했을 때, 사용될 transport-layer protocol을 알려준다. 
- Header checksum: detecting bit errors. 3.3에서와 같이 1's complement를 이용한 검증. router가 검증을 수행한다. 조심해야 할 점은 TTL field같이 가변 데이터가 존재하기 때문에 매 router에서 checksum 값을 update 해주어야 한다.
- Options: 잘 사용되는 field는 아니지만, header를 연장하게 해준다. 여러모로 복잡하기 때문에 IPv6에서는 빠졌다.
- Data(=payload)
**IP datagram은 option이 없다는 가정 하에 20바이트이다.** TCP의 header가 20바이트기 때문에 application-layer message의 관점에서는 40바이트의 header size라고 볼 수 있다.
### 4.3.2 IPv4 Addressing
먼저 용어정리부터 해보자. Host는 거의 대부분 network와 single link로 연결되어 있다. 만약 host에서 datagram을 전송하고 싶으면 '모든 길을 link를 통한다.' 그 host와 link 사이의 boundary를 우리는 **interface**라고 부르기로 하자. router는 forwarding을 수행하기 때문에 여러 개의 link와 연결되어 있을 수 밖에 없는데, 여기서도 router와 link 사이의 boundary를 interface라고 부른다. 그러므로 router는 multiple interface를 갖는다. host와 router 모두 IP datagram을 주고 받을 수 있기 때문에 **각 router와 host 모두 고유한 IP Address를 가지고 있어야 한다. 즉 IP Address는 interface를 갖는 host, router에 대한 것이라기 보다는 interface 그 자체와 더 관련이 있다고 이해하는 것이 옳다.**

알다시피 IP는 32 비트로 구성되고, **dotted-decimal notation**으로 표기한다. 192.168.0.1 같이 말이다.
각 interface는 global하게 unique한 IP Address를 가져야 한다. 근데 그냥 남는거를 닥치는 대로(wily-nilly) 고르는 것이 아니라 **subnet이라는 개념에 의해 정해진다.**
![](https://i.imgur.com/uDcQCYg.png)

- 왼쪽 3개의 host는 router의 interface와 모두 연결되어 있고, 223.1.1.xxx 형태의 IP 주소를 갖는다.
- **즉 leftmost 24bit가 동일하다.** 
- 그리고 3개의 host interface와 router interface가 연결되어 있는데, 이는 Ethernet, Wireless Access Point 등으로 interconnect된다. (ch6, 7에서 후술)
IP 용어에서 왼쪽 3개의 interface와 router의 interface는 **subnet을 형성한다**고 말한다. 이 subnet은 IP network, 혹은 그냥 network라고도 불린다. 이 subnet은 "223.1.1.0/24"로 표기한다. '/24' notation은 subnet mask라고도 불리는데, **leftmost 24bit가 같다는 뜻이다.**  여기에 새로 attach되는 interface는 무조건 저 형태를 만족해야 한다.
![](https://i.imgur.com/BYWfG2z.png)

>To determine the subnets, detach each interface from its host or router, creating islands of isolated networks, with interfaces terminating the end points of the isolated networks. Each of these isolated networks is called a subnet.

위 규칙을 아래 그림에 적용하면

![](https://i.imgur.com/fG5BhbI.png)
6개의 subnet을 얻는다. 

서로 다른 subnet은 이론상 address가 꽤나 달라야하지만, 현실은 그렇지 않다. 그 이유가 CIDR(Classless InterDomain Routing)과 관련이 있다. CIDR은 internet의 Address assignment strategy이다. subnet addressing을 수행할 때 a.b.c.d/x 형태로 나타낸다. x개의 most significant bit를 **prefix of the address라고 부른다.**  각 Organization마다 prefix를 부여 받는데, 그 안의 모든 device들은 common prefix를 갖는다. 즉 organization에서 나가는 경우에는 앞의 x bit만 보면 되고, 내부에서는 뒷부분 32 - x bit 만으로도 구별하는데 충분하다. 
만약 host가 255.255.255.255로 datagram을 보내면 message는 같은 subnet 내의 모든 host에게 전달된다. 그래서 IP broadcast address라고 불린다. 이제 host와 subnet이 자기들의 address를 어떻게 알아내는지 보자.
#### Obtaining a Block of Address
어떤 한 organization의 subnet에서 IP Address를 알아내기 위해서는 먼저 ISP에 연락해야 한다. ISP는 자신에게 할당되어 있는 larger block의 address로 부터 address를 제공해줄 것이다. 뭔 소리인지 하나도 모르겠지? 예시를 통해 살펴보자. ISP 스스로가 200.23.16.0/20으로 할당되어 있다고 하자. ISP는 이 address block을 8개의 equal sized contiguous address block으로 쪼개서 각 organization에 할당해준다... 뭔소리일까.
**아마도 ISP 자신의 block에서 뒷부분을 떼준다는 의미인 것 같다.**
![](https://i.imgur.com/9UbYdvV.png)
밑줄 친 부분이 subnet이라고 생각하면 된다.

#### Obtaining a Host Address: The Dynamic Host Configuration Protocol
organization이 block of address를 얻으면 interface에 IP address를 할당해줄 수 있다. System administrator가 수동으로 router의 IP Address를 설정하는 반면 Host의 address도 수동적으로 configure되지만 **Dynamic Host Configuration Protocol**을 이용하여 진행된다. DHCP는 host가 IP Address를 자동으로 할당되게 해주는 protocol이다. network admin은 DHCP를 설정하여 host가 network에 연결될 때 마다 **같은 IP 주소를 할당받게 하거나 혹은 temporary IP Address를 할당받게 설정할 수 있다.** 

DHCP의 자동화된 특성 때문에, 주로 **plug-and-play 혹은 zeroconf protocol**이라고 불린다.
- plug-and-play: 꽂으면 바로 연결이 이루어짐
- zero-configuration: 사용자가 추가적인 설정을 할 필요가 없음.
학생이 기숙사에서 도서관 갔다가 교실 갔다가 하는 상황을 생각해보자. 각 경우마다 새로운 subnet에 연결될 것이며 그 때마다 새로운 IP address를 할당받을 것인데, DHCP는 자동으로 처리해주기 때문에 매우 유용하다. 각 laptop 사용자가 수동적으로 configuration을 수행해줘야 한다면 얼마나 번거로웠을까?

DHCP는 그리고 client-server protocol이기도 하다. Client는 전형적으로 새로 도착하는 host를 의미한다. 가장 단순한 경우, 각 subnet이 DHCP server를 갖고 있을 것이다. subnet에 server가 존재하지 않는다면 router가 DHCP server의 주소를 알고 있을 것이다. 
![](https://i.imgur.com/972WiTa.png)
223.1.2/24 subnet에 DHCP server가 존재하고, router는 223.1.1/24, 223.1.3/24의 relay agent 역할을 한다.
![](https://i.imgur.com/9hUyHj2.png)
새롭게 도착하는 host에게 DHCP는 4단계 process를 제공한다. `yiaddr` (your internet address)는 client에 할당될 주소를 가리킨다.
![](https://i.imgur.com/OFKk0SP.png)

1. DHCP server discovery: 뉴비 host가 해야할 일은 DHCP 서버를 찾아가는 것이다. **DHCP discover message**라는 것을 통해 찾게 되는데, client는 UDP packet에 메시지를 담아서 67번 포트로 보낸다. 근데 목적지를 모르는데 어떻게 찾느냐? 소리를 크게 지르면 된다. 즉 Broadcast destination IP address에게 보내면 된다. 그리고 내 주소는 모르니까 0.0.0.0으로 적어놓고 보낸다. 그러면 IP datagram은 Link layer를 통해 subnet에 연결된 모든 node에게 메시지가 전달되게 된다.
2. DHCP server offer: 모든 subnet에 뿌렸으니까 server도 받게 된다. 그러면 server는 응당 그 요구에 대해 대답을 해줘야 하는데 이를 **DHCP offer message**라고 부른다. 이 메시지도 broadcast된다. 왜냐면 뉴비 host는 아직 할당받은 주소가 없기 때문에 주소를 콕 찍어서 전달할 수 없기 때문이다. 그리고 subnet에 여러개의 서버가 존재할 수 있기 때문에 뉴비 host는 여러군데에서 러브콜을 받을 수 있게 된다. 메시지에는 proposed IP address와 network mask, IP address lease time(사용 가능 시간)을 담아 보낸다. 몇 시간 ~ 일 단위로 보내는 것이 일반적.
3. DHCP request: 뉴비가 offer중에 하나를 골라서 **DHCP request message**를 전달한다. configuration param을 echo한다고 생각하면 된다.
4. DHCP ACK: 서버는 DHCP request message를 보고 parameter를 confirm한 뒤 **DHCP ACK message**를 전달한다.

이제 뉴비는 ACK까지 받게 되고 interaction이 끝난다. 그 뒤로는 제공받은 IP Address를 사용하여 subnet의 일원으로 활동하면 된다. lease 기간이 끝나면 연장해주는 메커니즘도 당연히 존재한다.

mobility 관점에서 DHCP는 단점이 존재하는데, 움직일 때 마다 새로운 subnet에 연결되어야 하기 때문에 TCP connection이 유지될 수 없다.

### 4.3.3 Network Address Translation (NAT)
새로운 장비를 연결하면 ISP가 모든 기존의 장치의 IP Address를 커버할 수 있는 새로운 address block을 제공해야 할 것 처럼 보인다. 그러냐 만약 ISP가 이미 연속적인 address range를 제공해 놓았다면?
NAT를 이용하면 된다!
![](https://i.imgur.com/nzBLwzb.png)
집에 있는 NAT-enabled router는 interface를 갖고 있는데 이는 home network의 일부분이다. 그림 기준 오른쪽. home network interface는 모두 10.0.0.0/24 subnet에 속해 있다. 그리고 address space 10.0.0.0/8은 **private network, 혹은 realm with private addresses**라고 불린다. 이는 주소가 그 네트워크 내의 속하는 device에게만 의미가 있는 경우를 의미한다. 그래서 이게 왜 중요한데...?
fact는 home network가 엄~청나게 많다는 것이고, 많은 곳에서 같은 주소 10.0.0.0/24를 사용한다. 같은 네트워크 내에 속해있는 device들은 10.0.0.0/24로 packet을 전송할 수 있다. 그러나 home network를 넘어서 더 global한 network로 나아가고 싶어하는 경우에는 이러한 address를 사용할 수 없는데 왜냐면 10.0.0.0/24 block을 사용하는 수많은 network가 존재하기 때문이다.
![](https://i.imgur.com/EW6uAm7.png)
즉 위와 같은 ip는 home network에서만 의미를 갖는다. 그렇다면 이 ip address들이 외부로 보내져야 하는 경우에는 어떻게 처리되는 것일까? **그 답은 NAT에 있다.**
NAT-enabled router는 외부의 router와 다르게 생겨먹었다. NAT router는 ip address를 가진 single device로서 outside world에 존재한다. 4.25를 보면 home router를 떠나는 모든 traffic은 138.76.29.7의 ip 주소를 갖고 있고, home router로 들어오는 모든 traffic 또한 그 주소를 갖고 있다. 즉 **본질적으로 NAT-enabled router는 home network의 디테일을 outside world로부터 숨기고 있는 것이다.**
![](https://i.imgur.com/EWufN6u.png)
NAT router가 WAN으로부터 같은 destination이 적힌 datagram을 받으면, 어떻게 그걸 internal host로 forwarding을 할까? **NAT translation table**을 이용하고, port number를  포함시키면 된다. 
4.25에서 10.0.0.1 host가 Web page를 Web server(128.119.40.186, 80)에 요청한다고 해보자. 10.0.0.1은 임의의 portnum 3345를 할당받고 datagram을 LAN으로 전달한다. NAT router에서 그 datagram을 받고, 새로운 portnum 5001을 할당해주고 source IP address를 WAN-side IP address인 138.76.29.7로 바꿔준다. 이때 새로운 portnum은 현재 NAT translation table에 없는 값을 할당해준다. 그리고 이 connection을 NAT translation table에 추가한다. server는 NAT router를 목적지로 하여 datagram을 송신한다. NAT에 도착하면 translation table을 통해 원래 host로 forwarding을 수행하고 table에서 제거한다.

*(참고: NAT은 최근 몇 년간 널리 사용되었지만, NAT에 반대하는 사람들도 있다. 첫 번째로, 포트 번호는 프로세스를 주소 지정하기 위해 사용되도록 설계된 것이지, 호스트를 주소 지정하기 위한 것이 아니라는 주장이 있다. 이는 실제로 홈 네트워크에서 서버를 실행하는 경우 문제를 일으킬 수 있다. 왜냐하면 2장에서 본 것처럼, 서버 프로세스는 잘 알려진 포트 번호에서 들어오는 요청을 대기하고 있으며, P2P 프로토콜의 피어는 서버 역할을 할 때 들어오는 연결을 수락해야 하기 때문이다. NAT 서버 뒤에 있는 DHCP가 제공한 NAT 주소를 가진 한 피어가 어떻게 다른 피어와 연결할 수 있을까? 이러한 문제에 대한 기술적 해결책으로는 NAT traversal 도구 [RFC 5389, RFC 5128, Ford 2005]가 포함된다.
보다 “철학적인” 논쟁도 NAT에 대해 제기되었다. 여기서 문제는 라우터는 계층 3(즉, 네트워크 계층) 장치로 설계되었으며, 네트워크 계층까지만 패킷을 처리해야 한다는 점이다. NAT는 호스트가 서로 직접 통신해야 하며, 중간 노드가 IP 주소를 수정하거나 포트 번호를 수정하지 않아야 한다는 원칙을 위반한다. 이러한 논쟁은 4.5절에서 중간 장치(middleboxes)를 다룰 때 다시 논의될 것이다.)*
### 4.3.4 IPv6
1990년대 후반에 이르러 IPv4를 더 발전시키려는 연구가 시작되었는데, 왜냐면 32-bit IPv4 주소가 고갈되어가고 있었기 때문이다. 새로운 subnet이 말도안되는 속도로 다닥다닥 붙기 시작하면서, large IP address space에 대한 필요가 증가해왔고, 그에 발 맞춰 IPv6이 개발되었다.
IPv4 주소 공간이 고갈되는 시점에 대해서는 논쟁이 있었다. 2008년, 2018년이라고 예상한 두 주장이 경합을 이루었는데, 2011년에 이르러 마지막 남은 공간을 할당하면서 고갈되었다. 즉 central pool에 더 이상 남은 address block이 없는 상황이었다. 1990년대에 고갈 예상 시간을 계산하면서, 지금부터 개발해야 고갈 시간에 맞춰 새로운 공간을 만들어 낼 수 있을 것이라 생각해 프로젝트가 시작되었다.

#### IPv6 Datagram Format
![](https://i.imgur.com/LQp2fDe.png)

중요한 변화들은 datagram format을 보면 알 수 있다.
 - Expanded addressing capabilities
   IPv6은 address size를 32Bit에서 128bit로 늘렸다. 절대로 IP address가 고갈될 일이 없다는 것을 보장해줄 만큼 크다. unicast와 multicast address 이외에도 **anycast address**라는 새로운 타입의 주소를 추가했다. 이는 datagram이 특정 그룹에 속한 하나의 host로 전달되게 한다.
- A streamlined 40-byte header
  많은 IPv4의 field가 없어지고, 40byte로 fixed된 길이의 header가 되었다. 이는 datagram을 router에서 더 빠르게 처리할 수 있게 만든다.
- Flow labeling
  IPv6은 flow라는 개념이 존재한다. 특정한 방법으로 handling이 필요한 flow에 속하는 packet에는 label을 지정해줘야 한다. 예를 들면 audio/video transmission의 경우 flow로 볼 수 있다. 반면 더 고전적인 application의 경우 flow로 여겨지지 않을 수 있다. File transfer나 e-mail 같은 경우. 그리고 high-priority를 가진 user(프리미엄 결제자 같은 경우..)에 의해 발생한 traffic의 경우도 flow로 볼 수 있다.
  ![](https://i.imgur.com/ZM4HWTg.png)
그리고 IPv6에 정의된 field는 다음과 같다:
- Version: IP version을 명시. IPv6의 경우 이 필드에 6을 저장하는 것은 놀랍진 않다. 하지만 여기 4를 저장한다고 IPv4가 되는 것은 아니다...
- Traffic class: IPv4에서의 TOS와 유사하게, flow 내에서의 우선순위를 정하거나 특정 application에서 송수신하는 datagram의 우선순위를 다른 datagram에 비해 높게 설정할 수 있다. 예를 들면 voIP over SMTP email...
- Flow label: datagram의 flow를 명시하는 20bit field.
- Payload length: IPv6에서의 byte 수를 명시해놓은 16bit field.
- Next header: datagram이 어떤 protocol로 전달되는지를 명시해놓는다. TCP / UDP. IPv4에서 protocol field와 완전히 같은 값을 사용
- Hop limit: router를 지날때마다 decrement된다. hop limit이 0이 되면 router는 그 packet을 무시한다.
- Source and destination address
- Data
그리고 IPv4와 비교했을 때 없어진 부분은 다음과 같다:
- Fragmentation/reassembly: IPv6는 fragmentation을 허용하지 않는다. 그럼 너무 큰 datagram이 router에 도착하면 어떻게 되는가? 그러면 router에서는 그냥 datagram을 drop하고 "Packet Too Big" IMCP error를 sender에게 보낸다. 그러면 sender는 작은 datagram을 다시 보낼 수 있다. fragmentation은 시간을 많이 잡아먹는 operation이기 때문에 과감히 없애고 IP forwarding의 speed-up에 더 초점을 맞추었다.
- Header checksum: transport-layer와 link-layer에서 checksum을 수행하기 때문에 network layer에서는 필요하지 않다고 생각하여 없앤 듯 하다. **다시 말하지만 fast processing of IP packet이 중요한 사안이었다.** IPv4에서 TTL을 매 router마다 계산해야 했다는 것을 기억할 것이다. 그래서 checksum도 매 router마다 새로 계산해야 했고, 너무 time-consuming하기 때문에, 자연스럽게 drop되었다.
- Options: option field는 더이상 standard IP header의 part가 아니다. 그래서 IP header가 40바이트로 고정되었다. next header에 포함될 수도 있다..

#### Transitioning from IPv4 to IPv6
근데 이미 깔려있는 IPv4를 어떻게 IPv6로 변환하였을까? 새로 만들어진 시스템이 IPv4를 처리하게 할 순 있어도, 이미 존재하고 있는 시스템이 IPv6를 지원하도록 하는건 어려워 보이는데... 몇 가지 방법을 생각해보자.
Flag day: 모든 인터넷 기기를 정해진 시간동안 꺼버리고 IPv6으로 올리자! 40년 전에 NCP에서 TCP로 올릴 때고 flag day로 모든 internet을 셧다운 하는 것은 말도 안되는 이야기였다... 하물며 지금은!
![](https://i.imgur.com/o4VaboY.png)
현재 통용되는 방법은 **tunneling**이다. tunneling의 기본적인 아이디어는 다음과 같다:
![](https://i.imgur.com/qpnvctE.png)

2개의 IPv6 node가 있다고 하자. (그림에서는 B, E) 그 2개의 node가 IPv6 datagram을 전달하려고 하지만 연결된 router는 IPv4이기 때문에 intervene된다. 여기서 2개의 IPv6 router 사이에 IPv4 router를 tunnel이라고 부른다. B에서 IPv6 datagram 전체를 IPv4 format에 우겨넣고 그걸 C로 보낸다. C는 바보같이 자기한테 온 datagram이 사실 IPv6이라는 것도 모르는 채로 그대로 D에게 전달한다. 그리고 IPv6 node에 IPv4 packet이 도착하면 이게 사실 IPv6이라는 것을 눈치채고 IPv6 datagram만 추출한다. (IPv4 protocol number field에 41이 적혀있으면 IPv4의 payload가 IPv6이라는 의미!!!). 그리고 원래 그랬던 것 처럼 IPv6으로 다시 전달하면 끝.
여기서 얻을 수 있는 교훈은, network-layer protocol을 바꾸는 것은 정~말 어렵다는 것이다. 1990년대 초반부터 새로운 network layer protocol이 우후죽순 등장하였지만 대부분은 제한된 penetration(침투력; 기술이 사용되는 범위를 의미)을 가지고 있었다. 새로운 network layer protocol을 도입하려 하는 것은 집의 기반을 바꾸는 것과 같다. 기반을 바꾸려면 전체 집을 다 들어내고 주민들을 일시적으로 내보내는 과정이 필수적이다. 그러나 Internet은 application layer에서는 protocol이 슉슉 바뀐다. SNS 플랫폼 같은. 먼 미래에는 network protocol이 바뀌는 것을 보게 될 지 몰라도, 그건 application layer에서의 변화에 비하면 엄청 느릴 것이다.
## 4.4 Generalized Forwarding and SDN
4.2.1에서 봤던 destination-based forwarding은 match-action의 두 단계로 나눠서 진행된다. 처음에 forwarding table을 보고 IP address를 match하고, switching fabric을 통해 output port로 내보내는 action을 취하는 방식. **여기서는 match-plus-action paradigm을 도입해보자. 이 paradigm은 match가 multiple header field associated with different protocols at different layers in the protocol stack에서 만들어질 수 있다.** 
Generalized forwarding에서, match-plus-action table은 destination-based forwarding table을 일반화 한 것이다. 
![](https://i.imgur.com/bxG0ZjV.png)
위 그림은 각 packet switch에서 table이 remote controller에 의해 compute/update되는 모습을 보여준다. 
![](https://i.imgur.com/yI1ezzl.png)![](https://i.imgur.com/3sAEmXf.png)
자.. 4.2/4.3/4.28의 차이점이 뭘까? 고민해볼 필요가 있다.
우리의 generalized forwarding에 대한 논의는 OpenFlow에 기반하고 있다. 
![](https://i.imgur.com/wQ0V3Yr.png)
**flow table**은 다음과 같은 요소를 포함한다:
- A set of header field values: destination-based forwarding처럼, hardware-based matching은 가장 빠르게 수행될 수 있다.(TCAM) packet이 어떤 flow table과도 matching되지 않으면 drop되거나 remote controller로 보내져서 추가적인 처리를 받는다.
- A set of counters: packet이 flow table entry와 matching될 때 마다 update된다.
- A set of actions to be taken: packet이 특정 flow table entry와 matching이 될 경우, action들은 packet을 forward하거나 drop하거나 copy를 만들거나 multiple output을 보내거나 selected header field를 재작성하거나 한다.
### 4.4.1 Match
![](https://i.imgur.com/mvNU4vA.png)
OpenFlow의 Match abstraction은 match가 3개 layer의 protocol header에서 이루어질 수 있도록 해준다. 우리가 아직 link layer는 잘 모르지만 MAC이 frame의 전송과 수신 interface와 관련 있다는 것만 알아도 충분하다. IP Address가 아닌 Ethernet address를 기반으로 forwarding을 할 수 있기 때문에, OpenFlow-enabled device는 router의 역할과 switch(in link layer)의 역할을 모두 수행할 수 있다. 
Ingress port는 packet이 수신되는 input port를 의미한다. Transport layer의 portnum 또한 matching에 사용될 수 있다.
Flow table은 wildcard도 가지고 있다. 예를 들어서 128.119.\*.\*같은 경우 처음 16bit만 맞으면 match되었다고 본다. 그리고 각 flow table의 entry는 우선순위가 존재해서 여러개가 matching이 되는 경우 가장 우선순위가 높은 entry에 대응되는 action을 취한다.
그리고 모든 header field가 사용되지 않는 다는 점에 주목해보자. 예를 들어 TTL은 matching에 사용되지 않는 field이다. 왜 몇 개는 matching에 사용되고, 다른 것들은 그렇지 않게 설계되었을까? **그 답은 아마 functionality와 complexity 사이의 trade-off 때문일 것이다.** 
>Do one thing at a time, and do it well. An interface should capture the minimum essentials of an abstraction. Don’t generalize; generalizations are generally wrong.

**IP뿐만 아니라 다양한 header field를 이용하여 matching을 구할 수 있다는 것이 OpenFlow의 철학**
### 4.4.2 Action

4.28에 나와있듯이 각 entry에는 여러 개의 action이 존재하여 matching packet을 어떻게 처리할지 정해준다. multiple action이 존재한다면 specified order로 수행한다. 대표적인 예시 몇 개만 보면:
- Forwarding: 특정 output port로 forwarding 되거나, 모든 port에 Broadcasting 하거나, 여러개의 port에 multicast하거나.
- Dropping: flow table entry가 matching 되는 action이 없다면 그냥 dropped.
- Modify-field: output port로 forwarding 되기 전에 4.29에 나와있는 field를 수정할 수 있다(단, IP protocol field는 불가능)
### 4.4.3 OpenFlow Examples of Match-plus-action in Action
OpenFlow의 matching, action에 대해서 배웠으니 예시를 통해 이해해보자.
![](https://i.imgur.com/luFGk56.png)
network는 6개의 host로 이루어져 있고, 3개의 Packet switch(각각 4개의 local interface)로 이루어져 있다. 우리는 여러 개의 구현 목적을 달성하기 위해 s1, s2, s3의 flow table을 어떻게 구현해야 할 지 살펴볼 것이다.
#### A First Example: Simple Forwarding
목표: h5/h6 host -> s3 -> s1 -> s2 -> h3/h4 host
![](https://i.imgur.com/rJrcaBP.png)
![](https://i.imgur.com/7F9E0nw.png)
![](https://i.imgur.com/YK6Xyrm.png)

#### A Second Example: Load Balancing
목표: h3에서 10.1.\*.\*로 가는 datagram이 s2 -> s1의 direct link를 지나고, h4에서 10.1.\*.\*로 가는 datagram은 s3을 경유해서 s1으로 가게 하는 것
![](https://i.imgur.com/fk62yo1.png)
나머지 flow table도 채울 수 있어야 한다.
#### A Third Example: Firewalling
Firewall: 방화벽
목표: s2가 s3에 attach된 host에게서만 traffic을 수신하고자 함
![](https://i.imgur.com/8z1SgVO.png)
만약 다른 entry가 존재하지 않는다면 10.3.\*.\*에서 오는 traffic만 forward가 이루어진다.

위와 같은 예시를 통해 우리는 match-plus-action flow table이 제한된 형태의 programability를 가진다는 사실을 알았다. 더 high level로 수행할 수도 있을 것 같지 않는가? 
![](https://i.imgur.com/A5dxtip.png)

그게 P4(Programming Protocol-independent Packet Processors).
## 4.5 Middleboxes
