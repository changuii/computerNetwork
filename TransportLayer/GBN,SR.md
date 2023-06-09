# GBN

파이프라인에서 확인응답이 안된 패킷의 최대 허용수 N보다 크지 말아야한다.  

확인응답이 안된 가장 오래된 패킷의 순서번호를 base 사용되지 않는 가장 작은 순서번호를 nextseqnum(전송될 다음 패킷의 순서번호)로 정의한다면 4개의 간격을 식별할 수 있다.  

- 간격 [0, base-1] 순서번호는 이미 전송되고 확인응답이 된 패킷
- 간격 [base, nextseqnum] 송신되었지만 아직 확인응답되지 않은 패킷
- 간격 [nextseqnum, base+N-1] 상위 계층으로부터 데이터가 도착하면 바로 전송될 수 있는 패킷을 위해 사용된다.
- 간격 [base+N 이상] 순서번호는 파이프라인에서 확인응답이 안된 패킷의 확인응답이 도착할 때 까지 사용될 수 없다.  

아직 확인응답 안된 패킷을 위해 허용할 수 있는 순서번호의 범위는 순서번호의 범위상에서 크기가 N인 윈도로 나타낸다.  
프로토콜이 동작할 때, 이 윈도는 순서번호 공간에서 오른쪽으로 이동(slide)된다. 이러한 이유로 N을 윈도 크기라 부르며 GBN은 슬라이딩 윈도 프로토콜이라 부른다.  

실제로 패킷의 순서 번호는 패킷의 헤더 안의 고정된 길이 필드에 포함된다. 만약 k가 패킷 순서번호 필드의 비트 수라면, 순서번호의 범위는 [0, $2^k$-1]이 된다.  

순서범위의 제한된 범위에서, 순서번호를 포함하는 모든 계산은 모듈로 $2^k$연산을 이용한다. rdt3.0의 경우 순서번호의 범위는 0~1이 된다.  

TCP가 32비트 순서번호 필드를 갖는다면 이 필드의 TCP 순서번호는 패킷 단위라기보다는 바이트 스트림에서 바이트를 세는 수이다.   

GBN 송신자는 다음과 같은 세 가지 타입의 이벤트에 반응해야 한다.  
1. 상위로부터의 호출: rdt_sen()가 위로부터 호출되면, 송신자는 우선 윈도가 가득 찼는지, 즉 N개의 확인응답되지 않은 패킷이 있는지 확인한다.
   - 만약 가득 차 있지 않다면 패킷이 생성되고 송신된다. 그리고 변수들을 적절하게 갱신된다.  
   - 만약 윈도가 가득 차있다면 송신자는 윈도가 가득 차 있음을 가르키는 함축적인 의미로 단지 데이터를 상위 계층으로 반환한다.  
   - 실제 구현에서 송신자는 데이터를 버퍼링하거나, 오직 윈도가 가득 차 있지 않을 때만 rdt_send()를 호출하는 동기화 매커니즘(세마포, 플래그)을 사용할 것이다.
2. ACK의 수신: 순서번호 n을 가진 패킷에 대한 확인응답은 누적 확인응답으로 인식된다. 이 누적 확인응답은 수신측에서 올바르게 수신된 n을 포함하여 n까지의 순서번호를 가진 모든 패킷에 대한 확인응답이다.  
3. 타임아웃 이벤트 : 만약 타임아웃이 발생하면 송신자는 ㅇ전에 전송되었지만 아직 확인응답되지 않은 모든 패킷을 다시 송신한다.
   - 송신자는 가장 오래된 '전송했지만 아직 확인응답 안된 패킷'에 대한 타이머로 생각될 수 있는 단일 타이머를 사용한다.
   - 만약 확인응답 안된 패킷이 없다면 타이머는 멈춘다.

GBN에서는 수신자의 행동도 단순하다.  

만약 순서 번호 n을 가진 패킷이 오류없이 그리고 순서대로 수신된다면 수신자는 패킷 n에 대한 ACK를 송신하고 상위 계층에 패킷의 데이터를 전달한다.  
그 외의 경우에는 수신자는 그 패킷을 버리고 가장 최근에 제대로 수신된 순서의 패킷에 대한 ACK를 재전송한다.  

GBN프로토콜에서 수신자는 순서가 잘못된 패킷들을 버린다.  
비록 정확하게 수신된 패킷을 버리는 것이 어리석어 보이고 낭비인 것 같지만, 그런 작업을 수행하는데는 몇가지 이유가 있다.  

지금 패킷 7이 수신되어야 하지만 그 사람 다음의 패킷 n+1이 먼저 도착했다고 가정하자 데이터가 순서대로 전달되어야 하므로 수신자는 패킷 n+1을 저장하고 나중에 패킷 n이 수신되고 전달된 후에 상위계층으로 전달한다.  

그러나 패킷 n이 손실된다면 패킷 n과 n+1 모두 재전송 될 것이다. 따라서 수신자는 어떤 순서가 잘못된 패킷에 대해 버퍼링을 할 필요가 없다.  

그러므로 송신자는 윈도 상위와 하위 경계 그리고 이 윈도 안에 있는 nextseqnum 위치를 유지해야 하지만 수신자가 유지해야 하는 것은 단지 다음 순서 패킷의 순서번호이다. 이 값은 expectedseqnum에 저장된다.  

물론 올바르게 수신된 패킷을 버리는 것의 단점은 그 패킷의 재전송이 손실되거나 왜곡될 수 있으므로 많은 재전송이 필요할 수도 있다는 것이다.  

따라서 GBN은 재전송 횟수가 많아지지만 타이머가 1개이고 버퍼가 없기 때문에 컴퓨팅자원 측면에서 유리하다.  


# SR 

GBN 자체에도 성능 문제를 겪는 시나리오들이 존재한다. 특히 윈도 크기와 대역폭 지연 곱의 결과가 모두 클 때 많은 패킷들이 파이프라인에 있을 수 있다.  
그러나 GBN은 하나의 오류 때문에 많은 패킷을 재전송하므로 많은 패킷을 불필요하게 재전송하는 경우가 발생한다.  

SR 프로토콜은 수신자에게 오류가 발생한 패킷을 수신했다고 의심되는 패킷만을 송신자가 다시 전송하므로 불필요한 재전송을 피한다.  
필요에 따라 각각의 개별적인 재전송을 수신자가 올바르게 수신된 패킷에 대한 개별적인 확인응답을 요구할 것이다.  

윈도크기 N은 아직 확인응답이 안된 패킷 수를 제한하는데 사용된다.  
그러나 GBN과는 달리 송신자는 윈도에 있는 몇몇 패킷에 대한 ACK를 이미 수신했을 것이다.  

SR 수신자는 패킷의 순서와 무관하게 손상없이 수신된 패킷에 대한 확인응답을 할 것이다.  
순서가 바뀐 패킷은 빠진 패킷이 수신될 때 까지 버퍼에 저장하고 빠진 패킷이 수신된 시점에서 일련의 패킷을 순서대로 상위 계층에 전달할 수 있다.  

SR 송신자 이벤트와 행동
1. 상위로부터의 수신 : SR 송신자는 패킷의 다음 순서번호를 검사한다.  
   - 순서번호가 송신자 윈도 내에 있으면 데이터는 패킷으로 송신된다.
   - 그렇지 않으면 GBN처럼 버퍼에 나중에 전송하기 위해 되돌려진다.  
2. 타임아웃 : 타임 아웃시 오직 한 패킷만 전송되기 때문에, 각 패킷은 자신의 논리 타이머가 있어야 한다. 하나의 하드웨어 타이머가 여러개의 논리 타이머 흉내를 낸다.  
3. ACK 수신 : ACK가 수신되었을 때, SR 송신자는 그 ACK가 윈도 내에 있다면 그 패킷을 수신된 것으로 표기한다. 
   - 만약 패킷 순서번호가 send_base와 같다면, 윈도 베이스는 가장 작은 순서번호를 가진 아직 확인응답 않은 패킷으로 옮겨진다. 만약 윈도가 이동하고 윈도 내의 순서번호를 가진 미전송 패킷이 있다면 이 패킷들은 전송된다.  

SR 수신자 이벤트와 행동
1. [rcv_base, rcv_base+N-1] 내의 순서번호를 가진 패킷이 손상없이 수신된다.
   - 이 패킷이 수신윈도의 base와 같은 순서번호를 가졌다면 이 패킷과 이전에 버퍼에 저장되어 연속적인 번호를 가진 패킷들은 상위 계층으로 전달된다.  
2. [rcv_base-N, rcv_base-1] 내의 순서번호를 가진 패킷이 수신된다.  
   - 이 경우에 패킷이 수신자가 이전에 확인응답한 것이라도 ACK가 생성되어야 한다.
3. 그 외의 경우 패킷을 무시한다.  

송신자와 수신자는 올바른 수신과 그렇지 않은 수신에 대해 항상 같은 관점을 갖지는 않을 것이다. 이는 SR 프로토콜에서 송신자와 수신자의 윈도가 항상 같지는 않다는 뜻이다.  

송신자와 수신자 윈도 사이의 동기화 부족은 순서버호의 한정된 범위에 직면했을 때 중대한 결과를 가져온다.  
예를 들어 한정된 범위의 네 패킷 순서번호 0, 1, 2, 3과 윈도크기 3에서 어떤 일이 일어날 수 있는지 생각해보자  

0부터 2까지의 패킷이 전송되어 올바르게 수신되고 수신자에서 확인이 되었다고 가정하자.  
그 순간에 수신자 윈도는 각각의 순서번호가 3, 0, 1,인 4, 5, 6번째 패킷에 있다.  
여기서 두 가지 시나리오를 고려한다.  

1. 첫 번째 시나리오에서는 처음 3개의 ACK 신호가 손실되고 송신자는 이 패킷을 재전송한다. 그 다음 수신자는 순서번호 0번 패킷을 수신한다.
2. 두 번째 시나리오에서 처음 3개의 패킷에 대한 ACK가 모두 올바르게 전달 되었다. 그러면 송신자는 자신의 윈도를 앞으로 이동시켜 각각의 순서번호가 3, 0, 1인 4, 5, 6번째 패킷을 보낸다. 순서번호 3을 가진 패킷이 손실되고 순서번호 0을 가진 패킷은 도착한다.  

이 두가지 시나리오에서 다섯 번째 패킷의 원래 전송과 첫 번째 패킷의 재전송을 구별할 방법은 없다.  
의심할 것도 없이 순서번호 공간의 크기보다 1이 작은 윈도 크기에서 작동하지 않는다.  

SR 프로토콜의 윈도 크기는 순서번호 공간의 크기의 절반보다 작거나 같아야 한다.  

패킷의 순서번호 바뀜 현상으로 송신자와 수신자간의 윈도가 x를 포함하지 않더라도 순서번호 또는 확인응답 번호 x를 가진 오래된 패킷의 복사본이 생길 수 있다.  
순서번호가 재사용될 수 있으므로 그런 중복 패킷들을 막을 수 있는 조치가 있어야 한다.  
실제 방식은 송신자가 이전에 송신된 순서번호가 재사용되지 않음을 확실히 하는 것이다.  
이는 패킷이 일정 시간 이상으로 네트워크에 존재할 수 없다는 가정에 의해 이루어진다.  

대략 3분의 최대 패킷 수명이 고속 네트워크에 대한 TCP 확장에 가정되어있다.  

따라서 SR 프로토콜은 재전송 개수는 적지만 타이머가 많고 버퍼가 있기 때문에 컴퓨팅 자원 측면에서 불리하다.  


# 정리 및 요약

GBN 프로토콜은 수신윈도가 없고 오직 송신자에게 윈도가 있다. 수신자는 오직 최근에 수신된 패킷에 대한 확인응답 번호만 가지고 있으면 된다. (expectedseqnum)  

따라서 GBN프로토콜의 동작은 base가 ACK되면 윈도를 이동시키고 타임아웃이 발생하면 확인응답되지 않은 모든 패킷을 재전송한다.  
따라서 수신자는 패킷을 버퍼링할 필요 없이 버리고 가장 최근에 ACK된 패킷을 재전송하면 된다.  

또한 상위 계층으로부터 데이터가 들어왔을 때 윈도 크기를 체크하고 가득차지 않았다면 패킷을 전송한다.  
만약 가득 찼다면 상위 계층으로 데이터를 반환하거나 버퍼링한다.  
실제로는 세마포나 플래그를 이용해 동기화 매커니즘을 적용하기도 한다.  


SR 프로토콜은 GBN과 달리 수신자에게도 윈도가 있어 순서대로 오지 않은 패킷에 대한 확인응답 및 버퍼링을 제공한다.  

송신자와 수신자의 윈도는 다르게 움직이며 서로 base에 대한 확인응답, 패킷이 도착한다면 이동하게 된다.  

GBN과 달리 SR은 타임아웃된 패킷에 대해서만 재전송하기 때문에 모든 순서번호에 타이머를 작동시켜야 한다.  

송신자와 수신자의 윈도가 다르게 움직이기 때문에 SR 프로토콜에서는 윈도 크기가 순서번호의 크기의 절반보다 작거나 같아야만 동작한다.  

GBN프로토콜은 재전송 갯수가 많지만 타이머가 한개이고 수신자의 버퍼도 존재하지 않는다. 
따라서 컴퓨팅 자원 측면에서 봤을 때 유리하다.  
SR 프로토콜은 재전송 갯수는 적지만 타이머가 여러개이고 수신자의 버퍼도 존재한다.  
따라서 컴퓨팅 자원 측면에서 봤을 때 불리하다.  




