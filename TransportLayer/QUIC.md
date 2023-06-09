# 트랜스포트 계층의 발전 

## QUIC: 빠른 UDP 인터넷 연결  

애플리케이션에서 필요로 하는 트랜스포트 서비스가 UDP또는 TCP 서비스 모델에 적합하지 않은 경우, 애플리케이션 설계자는 애플리케이션 계층에 항상 '자신의 프로토콜 확장'을 할 수 있다.  

이것은 QUIC(Quick UDP Internet Connections)에서 취한 방식이다.  

QUIC는 UDP를 하위 트랜스포트 계층 프로토콜로 사용하는 애플리케이션 계층 프로토콜이며, 특히 단순하지만 발전된 HTTP/2 버전 위에서 인터페이스 되도록 설계되었다.  

QUIC의 주요 기능은 다음과 같다.  

1. 연결지향형이고 안전함 : TCP와 마찬가지로 QUIC은 두 종단간의 연결지향형 프로콜이다. 이를 위해서 QUIC 연결 상태를 설정하기 위해 종단간 핸드 셰이크가 필요하다. 연결 상태의 두 부분은 출발지와 목적지 연결ID이다.
   - 모든 QUIC 패킷들은 암호화되며, QUIC은 연결 상태를 설정하는데 필요한 핸드셰이크와 인증 및 암호화에 필요한 핸드셰이크를 결합하며, 먼저 TCP 연결을 설정한 다음 TCP 연결을 통해 TLS 연결을 설정하여 여러 RTT가 필요한 프로토콜 스택보다 더 빠른 설정을 제공한다.  
2. 스트림 : QUIC을 사용하면 단일 QUIC연결을 통해 여러 애플리케이션 레벨의 '스트림'들을 다중화할 수 있으며 QUIC연결이 설정되면 새 스트림을 빠르게 추가할 수 있다.  
   - HTTP/3의 컨텍스트에는 웹 페이지의 각 객체에 대해 다른 스트림이 있다. 각 연결에는 연결 ID가 있고 연결내의 각 스트림에는 스트림 ID가 있다.
   - 이 두 ID 모두 QUIC 패킷 헤더에 포함된다. 여러 스트림의 데이터는 UDP를 통해 단일 QUIC 세그먼트 내에 포함될 수 있다.  
   - 스트림 제어 전송 프로토콜(SCTP)은 단일 SCTP연결을 통해 여러 애플리케이션 레벨의 '스트림'을 다중화하는 개념을 개척한 초기의 안정적인 메시지 지향 프로토콜이다.  
3. 신뢰적이고 TCP 친화적인 혼잡제어 데이터 전송: QUIC은 각 QUIC 스트림에 대해 독립적으로 신뢰적인 데이터 전송을 제공한다.  
   - 따라서 한 HTTP 요청의 바이트가 손실되면 나머지 HTTP 요청들은 손실된 바이트가 재전송되어 HTTP 서버에서 TCP가 올바르게 수신할 때 까지 전달될 수 없다.
   - 이 문제는 HOL 차단 문제이다.
   - QUIC은 스트림 별로 신뢰적이고 순서대로 전달하기 때문에 손실된 UDP 세그먼트는 해당 세그먼트에서 데이터가 전달된 스트림에만 영향을 준다.
   - 다른 스트림의 HTTP 메시지는 계속 수신되어 애프리케이션에게 전달될 수 있다.

QUIC은 TCP와 유사한 확인응답 매커니즘을 사용하여 신뢰적인 데이터 전송 서비스를 제공한다.  
QUIC의 혼잡제어는 TCP 리노 프로토콜을 약간 수정한 TCP 뉴리노를 기반으로 한다.  

끝으로 QUIC은 두 종단 사이에 신뢰적이고 혼잡제어된 데이터 전송을 제공하는 애플리케이션 계층 프로토콜이라는 점을 다시한번 강조한다.  
QUIC의 저자들은 이는'애플리케이션 프로그램 업데이트 시간 척도' 면에서 QUIC으로 변경될 수 있음을 의미하는 것이며 이는 TCP 또는 UDP 업데이트 시간 척도보다 훨씬 빠르다는 뜻이라고 강조한다.  


