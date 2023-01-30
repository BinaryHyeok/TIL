# 1 : N

로운 게스트가 올때마다 Offer&Answer로 연결하도록.. 그리고 Candidate는 발생한 이벤트의 ice-ufrag를 기준으로 식별해서 한명

호스트는 한명의 게스트마다 RTCPeerConnection을 새로 생성해서 각자 따로 연결해줘야 한다

sender와 receiver는 통신하기 위해 웹에 들어와 시그널링 서버와 연결한다.

 

sender는 만들고자 하는 방의 제목(이름)을 시그널링 서버에 넘겨준다. ( 방 생성 )
시그널링 서버는 넘겨진 방의 이름과 sender와 연결된 커넥션을 저장한다.
방을 생성한 sender는 통화를 시작한다. ( RTCPeerConnection 생성 )
이때 생성된 RTCPeerConnection에 관한 candidate와 SDP(offer)를 생성한 후 서버에 넘겨줘 저장시킨다.
receiver도 통신하기 원하면 RTCPeerConnection을 생성하고 candidate를 서버에 넘겨준다.
receiver의 candidate를 받은 서버는 sender에게 candidate를 넘겨주면 sender는 이를 활용해 ICE Candidate를 추가한다.
그리고 receiver에게는 sender의 SDP(offer)와 candidate를 넘겨준다.
receiver는 서버를 통해 받은 sender의 SDP(offer)와 candidate를 이용해 RemoteDescription을 셋팅하고, ICE Candidate를 추가한다.
그리고 receiver는 서버에게 answer를 주고, 서버는 이를 받고 sender에게 receiver의 answer를 넘겨 준다.
sender는 receiver의 answer를 받은 후 RemoteDescription을 셋팅한다.
이로써 sender와 receiver 둘 다 Real Time Communication을 할 준비를 끝냈다.
무엇을 offer하지?

SDP = 스트리밍 미디어의 초기화 인수를 기술하기 위한 포맷입니다.

 

candidate란? 

피어 투 피어 통신을 하려는 양쪽은 네트워크 연결에 관한 정보도 교환해야 하는데, 이때의 네트워크 정보.

 

글로 설명하려니 어려운 것 같다. 

 

그림을 쉽게 표현을 해보자.