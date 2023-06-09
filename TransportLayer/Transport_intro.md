# 트랜스포트 계층 서비스 및 개요

트랜스포트 계층 프로토콜은 각기 다른 호스트에서 동작하는 애플리케이션 프로세스간의 논리적 통신을 제공한다.  
논리적 통신은 애플리케이션 관점에서 보면 프로세스들이 동작하는 호스트들이 직접 연결된 것처럼 보인다.  

## 트랜스포트 계층과 네트워크 계층 사이의 관계

트랜스포트 계층 프로토콜은 각기 다른 호스트에서 동작하는 프로세스들 사이의 논리적 통신을 제공하지만, 네트워크 계층 프로토콜은 호스트들 사이의 **논리적 통신**을 제공한다.  

종단 시스템 안에서 트랜스포트 프로토콜은 애플리케이션 프로세스에서 네트워크 경계(즉, 네트워크 계층)까지 메시지를 운반하며, 또한 반대방향으로 네트워크 계층에서 애플리케이션 프로세스로 메시지를 운반한다.  

트랜스포트 계층이 제공할 수 있는 서비스는 하위 네트워크 계층 프로토콜의 서비스 모델에 의해 제약 받는다.  

만약 네트워크 프로토콜이 호스트 사이에서 전송되는 트랜스포트 계층 세그먼트에 대한 지연 보장이나 대역폭 보장을 제공할 수 없다면, 트랜스포트 계층 프로토콜은 프로세스끼리 전송하는 메시지에 대한 지연 보장이나 대역폭 보장을 제공할 수 없다.  

그럼에도 불구하고 하위 네트워크 프로토콜이 상응하는 서비스를 제공하지 못할 때도, 특정 서비스는 트랜스포트 프로토콜에 의해 제공될 수 있다.  

트랜스포트 계층 프로토콜은 네트워크 프로토콜이 제공하지 못하지만 애플리케이션에게 신뢰적인 데이터 전송 서비스와 세그먼트에 대한 기밀성을 제공할 수 있다.  

## 인터넷 트랜스포트 계층의 개요

인터넷의 네트워크 계층 프로토콜은 인터넷 프로토콜(Internet Protocol, IP)라는 이름을 갖는다.  

IP 서비스 모델은 호스트들 간에 논리적 통신을 제공하는 **최선형 전달** 서비스이다. 이것은 IP가 통신하는 호스트들 간에 세그먼트를 전달하기 위해 최대한 노력하지만, 어떤 보장도 하지 않는다는 것을의미한다.  

또한 무결성(세그먼트 전달 보장 및 순서대로 전달을 보장)을 보장하지 않는다. 따라서 IP는 **비신뢰적**인 서비스이다.  

UDP와 TCP(트랜스포트 계층 프로토콜)의 가장 기본적인 기능은 종단 시스템 사이의 IP전달 서비스를 종단 시스템에서 동작하는 두 프로세스간의 전달 서비스로 확장하는 것이다.  

호스트 대 호스트 전달을 프로세스 대 프로세스 전달로 확장하는 것을 **트랜스포트 계층 다중화와 역다중화**라고 부른다.  

UDP와 TCP는 헤더에 오류 검출 필드를 포함함으로써 **무결성 검사**를 제공한다.  

이러한 최소한의 두 가지 서비스가 UDP가 제공하는 유일한 두 가지 서비스이다.  


## 다중화와 역다중화

목적지 호스트에서의 트랜스포트 계층은 바로 아래의 네트워크 계층으로부터 세그먼트를 수신한다.  

**트랜스포트 계층은 한 호스트에서 도착하는 해당 애플리케이션 프로세스에게 이 세그먼트의 데이터를 전달하는 의무를 진다.**

트랜스포트 계층은 실제로 데이터를 직접 프로세스로 전달하지 않는다. 대신에 중간 매개자인 소켓에게 전달한다.  

수신측의 트랜스포트 계층은 수신 소켓을 식별하기 위해 헤더 필드를 검사한다. 그리고 이 세그먼트를 해당 소켓으로 보낸다. 트랜스포트 계층 세그먼트의 데이터를 올바른 소켓으로 전달하는 작업을 **역다중화(Demultiplexing)** 라고 한다.  

출발지 호스트에서 소켓으로부터 데이터를 모으고, 이에 대한 세그먼트를 생성하기 위해 각 데이터에 헤더정보(역다중화에 사용한다.)로 캡슐화하고, 그 세그먼트들을 네트워크 계층으로 전달하는 작업을 다중화라고 한다.  

비록 인터넷 트랜스포트 프로토콜의 내용에서 다중화와 역다중화를 설명했지만, 이들은 한 계층에서의 한 프로토콜이 그 상위 계층의 여러 프로토콜에 의해 사용될 때마다 관련된 것임을 알아두기 바란다.  

트랜스포트 계층 다중화에는 두 가지 요구사항이 있다.  
1. 출발지 포트 번호 필드
2. 목적지 포트 번호 필드

각가의 포트 번호는 0~65535까지의 16비트 정수이다. 그 중에서 0~1023까지의 포트번호를 잘 알려진 포트번호라고 하며 사용을 엄격하게 제한하고 있다.


## 비연결형 다중화와 역다중화

프로세스들은 각각의 UDP소켓과 그와 연관된 포트번호를 갖는다.  

네트워크로부터 UDP 세그먼트들이 도착하면 호스트 B는 세그먼트의 목적지 포트번호를 검사하여 세그먼트를 적절한 소켓으로 보낸다. (역다중화)  

UDP 소켓이 목적지 IP주소와 목적지 포트번호로 구성된 두 요소 집합에 의해 식별된다는 것을 이해하자.  

출발지 IP주소와 출발지 포트번호가 모두 다르거나 어느 하나만 다를지라도 같은 목적지 IP주소와 목적지 포트번호를 가지면 2개의 세그먼트는 같은 목적지 소켓을 통해 같은 프로세스로 향할 것이다.  

출발지 포트번호는 무슨 목적으로 사용되는 것일까?  

A에서 B로가는 세그먼트에서 출발지 포트번호는 '회신주소'의 한 부분으로 사용된다.  
즉, B가 세그먼트를 다시 A에게 보내기를 원할 때, B에서 A로 가는 세그먼트의 목적지 포트번호는 A로부터 B로 가는 세그먼트의 출발지 포트번호로부터 가져온다.  

## 연결지향형 다중화와 역다중화

TCP 소켓과 UDP소켓의 다른 점은 TCP 소켓은 4개 요소의 집합, 즉 출발지 IP, 출발지 포트번호, 목적지 IP, 목적지 포트번호에 의해 식별된다는 것이다.  

특히 UDP와는 다르게 다른 출발지 주소 또는 다른 출발지 포트번호를 가지고 도착하는 2개의 TCP 세그먼트(초기 연결 설정 세그먼트는 제외)는 2개의 다른 소켓으로 향하게 된다.  

TCP 서버 애플리케이션은 '환영소켓'을 가지고 있다. 이 소켓은 포트번호 12000을 가진 TCP 클라이언트로부터 연결 설정 요청을 기다린다.  

연결 설정 요청은 목적지 포트번호 12000(서버의 환영소켓)과 TCP 헤더에 설정된 특별한 연결설정 비트(SYN)를 가진 TCP 세그먼트이다.  

서버는 연결 요청 세그먼트의 다음과 같은 네 가지 값에 주목한다.  
1. 출발지 포트번호
2. 출발지 IP주소
3. 목적지 포트번호
4. 목적지 IP주소

새롭게 생성된 연결 소켓은 이 네가지 값에 의해 식별된다.  

그 다음에 도착하는 모든 세그먼트의 출발지 포트번호, 출발지 IP주소, 목적지 포트번호, 목적지 IP주소의 값들이 일치하면 세그먼트는 이 소켓으로 역다중화될 것이다.  

## 웹 서버와 TCP

클라이언트가 서버로 세그먼트를 보내면, 도착하는 세그먼트는 목적지 포트번호 80을 갖고있을 것이다. 서버는 각기 다른 클라이언트가 보낸 세그먼트를 출발지 IP주소와 출발지 포트 번호로 구별한다.  

그러나 연결 소켓과 프로세스 사이에 항상 일대일 대응이 이루어지는 것은 아니다.  
실제로 오늘날의 많은 고성능 웹 서버는 하나의 프로세스만을 사용한다.  
그러면서 각각의 새로운 클라이언트 연결을 위해 새로운 연결 소켓과 함께 새로운 스레드(가벼운 서브 프로세스)를 생성한다.  

하나의 같은 프로세스에 붙어있는 많은 연결 소켓들이(다른 식별자를 가진) 동시에 존재할 수 있다.  

만약 클라이언트와 서버가 지속적인 HTTP를 사용한다면, 지속적인 연결 종속 기간에 클라이언트와 서버는 같은 서버 소켓을 통해 HTTP 메시지를 교환할 것이다.  

그러나 만약 클라이언트와 서버가 비지속적인 HTTP를 사용한다면, 모든 요청 및 응답마다 새로운 TCP연결이 생성되고 종료될 것이다.  

이 빈번하게 발생하는 소켓 생성과 종료는 바쁘게 일하는 웹 서버 성능에 심각한 부담을 준다.  


## 정리 및 요약

트랜스포트 계층은 네트워크 계층에서 제공해주는 호스트와 호스트 즉, 컴퓨터 사이의 논리적 통신을 프로세스와 프로세스 즉, 컴퓨터 내부에서 동작하는 애플리케이션 계층과의 논리적 통신으로 확장시켜준다.  

따라서 트랜스포트 계층이 기본(즉, 최소한의 기능)으로 제공하는 것은 다중화와 역다중화 그리고 오류 검출이다.  

UDP는 이러한 기본적인 기능들을 제공하고 TCP는 부가적으로 흐름 제어, 혼잡 제어, 신뢰적 데이터전송을 제공한다.  

다중화는 애플리케이션 계층의 메시지를 모아서 캡슐화하여 세그먼트로 만들어 상대방이 세그먼트를 역다중화할 수 있도록 헤더필드에 포트번호를 삽입하는 것이다.  

역다중화는 네트워크 계층으로부터 온 세그먼트의 헤더필드를 검사하여 자신의 컴퓨터, 호스트에서 작동하고 있는 프로세스의 소켓으로 메시지를 추출하여 전달한다.  






