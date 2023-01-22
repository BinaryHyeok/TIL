# Web RTC

WebRTC(Web Real-Time Communications)
  
![](https://velog.velcdn.com/images/jsb100800/post/eb72ec7d-c0b9-4038-9851-df1b27524b7e/image.webp)

WebRTC는 별도의 플러그인 설치 없이 실시간으로 미디어(오디오, 비디오, 텍스트, 파일 등)를 최대한 서버를 거치치 않고 Peer간 전송할 수 있는 오픈소스 웹 기반 기술이다.

## 서버의 종류
### 1. Signaling Server 

Signaling Server는 WebRTC에서 가장 기본이 되는 서버라고 할 수 있다.  
서로 다른 네트워크에 있는 Peer들을 연결시키기 위해서는 Session Control Message, Error Message, Codec, Bandwith 등 다양한 정보가 필요하다.  

결국 WebRTC 기수을 활용해서 Peer끼리 연결하기 위해서는 위의 정보들이 먼저 각각의 Peer들에게 전달돼야 한다는 것이며 이 프로세스를 Signaling 이라고 한다.  

이러한 정보들을 중계해주는 역할을 하는 서버가 바로 Signaling Server이다.  
또한 위의 정보들은 SDP(Session Description Protocol)을 사용한다.  

WebRTC와는 별개로 Signaling Server는 직접 구축해야하고, 보통 클라이언트 사이드와 WebSocket을 사용하여 통신한다.  

Signaling Server를 중심으로 WebRTC의 동작 과정을 설명해보면 다음과 같다.

> Client와 Server 사이 통신은 WebSocket을 사용한다.
> ![](https://velog.velcdn.com/images/jsb100800/post/530ee447-7034-4542-878e-19baba3b116f/image.png)
> 1. Client Side의 A Client(Peer)에서 Signaling Server로 연결에 필요한 A Client의 데이터를 보낸다.  
-> Signling Offer
> 2. Server Side에서, Signaling Server에 연결된 모든 세션들에게 A Client의 데이터를 전달한다.
> 3. Client Side의 B Client(Peer)에서 A Client의 데이터를 활용해서 연결에 필요한 일련의 작업을 한 후, B Client의 데이터를 Signaling Server로 보낸다.  
> -> Signaling Answer
> 4. Server Side에서, A Client의 세션에게 B Client의 데이터를 전달한다.
> 5. 각각의 데이터를 활용하여 WebRTC가 A Client와 B Client가 연결한다. 

<br>

### 2. STUN Server(Session Traversal Utilities for NAT)
위의 Signaling Server을 이용해서 Peer간의 통신이 가능하다.  
하지만 통신 중간에 방화벽, NAT 환경에 놓여 있는 Peer에 대해서는 Signaling이 불가능하다.  
이때 이어 설명할 STUN Server와 TURN Server가 필요하다.  

STUN Server는 클라이언트 자신의 공인 IP(Public Address)를 알려주는 서버이다.
Client에서 STUN Server로 요청을 보내서 자신의 공인 IP를 확인한 후, 해당 IP를 활용하여 Signaling하게 된다.  

> STUN Server같은 경우는 단순히 정보 제공을 위한 서버라 트래픽 발생이 현저히 낮기 때문에 웬만하면 무료 STUN Server를 사용해도 문제가 크게 없다고 한다.

<br>

### 3. TURN(Traversal Using Relays around NAT) Server
STUN Server를 활용하면 80% 정도는 Signaling을 통한 연결이 가능하다고 한다.  
하지만, 그렇지 못한 20%의 경우가 존재하는데, 보호정책이 강한 NAT나 라우터, 보통 Symmetric NAT환경에서 나타난다고 한다.  

TURN Server는 Symmetric NAT 제한을 우회할 수 있게끔 해주는 기능을 한다.  
**결국 TURN Server가 Peer간의 통신 채널을 중계해주는 역할을 하며, WebRTC의 가장 큰 특징인 P2P 방식에서 벗어나게 된다.**

때문에 Peer간 모든 트래픽을 중계해주기 때문에 상당한 부하를 감당해야하며, 비용 또한 크게 발생할 것이다.  

**즉, Local IP와 Public IP 둘다로도 연결할 수 없는 경우 TURN Server을 최후의 수단으로 사용하게 된다.**  

<br>

### 4. Media Server
WebRTC의 구현 방식에는 크게 3가지로 Mesh, SFU, MCU 방식이 있다.  
지금까지 말한 방식은 Mesh 방식으로 서버의 자원이 적게 들지만 Peer 수가 늘어날 수록 Client 사이드의 과부하게 급격하게 증가하는 방식이다. 따라서 소규모 연결에 적합할 것이다.  

Mesh 방식에서는 Media Server가 필요하지 않다.
Media Server는 SFU, MCU 방식의 WebRTC에서 필요한 서버이다.  

각각의 Peer들은 Media Server에게 미디어 스트림들을 쏴주고 Media Server에서 미디어 트래픽을 관리하여 각각의 Peer에게 다시 배포해주는 멀티미디어 미들웨어이다.  

즉, WebRTC의 특징은 P2P 통신이 아니게 되는 것은 TURN Server와 유사하다고 할 수 있고, 클라이언트에 부하가 현저히 줄어드는 대신 서버의 부하가 커지며 구현의 난이도가 높다.  

<br>

## WebRTC 구현 방식 종류
![캡처](https://user-images.githubusercontent.com/94176133/211991461-bf472326-edc9-4ca3-92a0-bbef7c7ce04f.PNG)

### 1. Mesh
특징  
- Signaling Server, STUN Server, TRUN Server를 사용하는 전형적인 P2P WebRTC 구현 방식이다.
- 1:1 연결 혹은 소규모 연결에 적합하다.  

장점
- Peer간의 Signaling 과정만 서버가 중계하기 떄문에 서버의 부하가 적다.
- 직접적으로 Peer간 연결되기 때문에 실시간 성이 보장된다.
  
단점
- 연결된 Client의 수가 늘어날 수록 Client의 과부하가 급격하게 증가한다.
- 간단하게 생각해봐도 N명이 접속한 화상회의라면, 클라이언트 각각에서 N-1개의 연결을 유지해야 하기 떄문이다.


<br>

### 2. MCU(Multi-point Control Unit) 방식
특징
- 다수의 송출 미디어 데이터를 Media Server에서 혼합(Multiplexing) 또는 가공(Transcoding)하여 수신측으로 전달하는 방식
- P2P방식 X, Server와 Client간의 peer을 연결한다.
- Media Server의 매우 높은 컴퓨팅 파워가 요구된다.
  
장점
- Client의 부하가 크게 줄어든다.
- N : M 구조에서 사용 가능하다.

단점
- 실시간성이 저해된다.
- 구현 난이도가 상당하게 어렵고, 비디오와 오디오를 혼합 및 가공하는 과정에서 고난도 기술과 서버의 큰 자원이 필요하다.

<br>

### 3. SFU(Selective Forwarding Unit) 방식
특징
- 각각의 Client간 미디어 트래픽을 중계하는 Media Server을 두는 방식
- P2P방식 X, Server와 Client간의 peer를 연결한다.
- Server에게 자신의 영상 데이터를 보내고, 상대방의 수만큼 데이터를 받는 형식
- 1:N 혹은 소규모 N:M 형식의 실시간 스트리밍에 적합
  
장점
- Mesh 방식보다 느린 것은 어쩔수 없다. 하지만 비슷한 수준의 실시간성을 유지할 수 있다.
- Mesh 방식보다는 Client의 부하가 줄어든다.

단점
- Mesh 방식보다는 서버의 부하가 늘어난다.
- 대규모 N:M 구조에서는 여전히 Client의 부하가 크다.

<br>



[참고자료]  
https://velog.io/@jsb100800/%EA%B0%9C%EB%B0%9C-WebRTC-SpringBoot-Vue.js%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Group-Video-Call