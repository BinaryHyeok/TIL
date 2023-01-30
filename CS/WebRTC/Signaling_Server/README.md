# Signaling Server

## 기본 흐름

## 1. SDP 교환 - Offer, Answer
<img width="697" alt="image" src="https://user-images.githubusercontent.com/94176133/215475713-12c97e51-36bf-4126-a31f-63f0f765f6c9.png">

1~4는 offer-Answer에 관한 내용이며 WebRTC가 아닌 SDP(Session Description Protocol)을 생성하여 서로 주고받으며 시그널링을 수행한다.

<br>

<img width="666" alt="image" src="https://user-images.githubusercontent.com/94176133/215480523-de9d978d-f70e-4756-8514-de11a8a231ea.png">

Alice와 Bob 두명의 Peer가 있다. Alice와 Bob은 서로 SDP 기반의 Offer와 Answer 메시지를 주고 받는다.

1. Alice가 SDP 형태의 Offer 메시지를 생성한다.
2. Alice가 생성된 Offer 메시지를 본인의 LocalDescription으로 등록한다.
3. Alice가 Offer메시지를 시그널링 서버에게 전달한다.
4. 시그널링 서버는 상대방 Bob을 찾아서 SDP 정보를 전달한다.
5. Bob은 전달받은 Offer 메시지를 본인의 RemoteDescription에 등록한다.
6. Bob은 Answer 메시지를 생성한다.
7. 생성된 Answer 메시지를 본인의 LocalDescription으로 등록한다.
8. Bob은 Answer 메시지를 시그널링 서버에게 전달한다.
9. 시그널링 서버는 상대방 Alice를 찾아서 Answer 메시지를 전달한다.
10. Alice는 전달받은 Answer 메시지를 본인의 RemoteDescription에 등록한다.

## 2. ICE 협상(ICE Negotiation)
SDP를 서로 교환한 후, 각 Peer Alice와 Bob은 서로의 주소 값을 알기 위해 ICE Candidate(ICE 후보)를 교환한다. 이때 사용되는 기술이 NAT Traversal이다.

> ICE(Interactive Connectivity Establishment)는 p2p 네트워킹에서 두 컴퓨터가 가능한 한 직접 서로 통신하는 방법을 찾기 위해 컴퓨터 네트워킹에 사용되는 기술

<br>

> 각 ICE 메세지들은 두 개의 컴퓨터를 서로 연결하기 위한 정보들에 덧붙여 프로토콜(TCP or UDP), IP주소, 포트넘버, 커넥션 타입 등을 제안한다.


### WebRTC에서 시그널링 서버는 다음과 같은 일을 한다.
- 각 Peer간 SDP 메시지 Offer, Answer를 전달해주고, ICE 후보를 주고 받을 수 있도록 돕는 서버
- Peer 관리를 해주는 포워딩 서버