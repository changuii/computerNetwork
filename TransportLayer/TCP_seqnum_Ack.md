## 순서번호와 확인응답 번호

TCP는 데이터를 구조화되어 있지 않고 단지 순서대로 정렬되어 있는 바이트 스트림으로 본다.  
TCP의 순서번호 사용은 순서번호가 일련의 전송된 세그먼트에 대해서가 아니라 전송된 바이트의 스트림에 대해서라는 관점을 반영한 것이다.  

세그먼트에 대한 순서번호는 세그먼트에 있는 첫 번째 바이트 스트림 번호이다.  

호스트 A에서의 프로세스는 TCP 연결상에서 호스트 B의 프로세스로 데이터 스트림의 전송을 원한다고 가정하자.  
호스트 A의 TCP는 데이터 스트림의 각 바이트에 암시적으로 순서번호를 지정한다.  
데이터 스트림은 500,000바이트로 구성된 파일이라하자. 또한 MSS는 1000바이트이고, 데이터 스트림의 첫 번째 바이트는 0으로 설정했다고 가정하자.  

TCP는 데이터 스트림으로부터 500개의 세그먼트들을 구성한다.  
첫 번째 세그먼트는 순서번호 0, 두 번째 세그먼트는 순서번호 1000, 세 번째 세그먼트는 순서번호 2000과 같은 식으로 할당된다.  

이제 확인응답을 생각해보자. TCP는 전이중 방식임을 상기하자  
호스트 B로부터 도착한 각 세그먼트는 B로부터 서로 들어온 데이터에 대한 순서번호를 갖는다.  
호스트 A가 자신의 세그먼트에 삽입하는 확인응답 번호는 호스트 A가 B로부터 기대하는 다음 바이트의 순서번호이다.  

호스트 A가 B로부터 0~535까지의 번호가 붙은 모든 바이트를 수신했다고 가정하자. 그리고 호스트 B로 세그먼트를 송신하려고 한다. 호스트 A는 호스트 B의 데이터 스트림에서 536번째 바이트와 그 다음에 오는 모든 바이트를 기다린다.  
그래서 호스트 A는 세그먼트의 확인응답 번호 필드에 536을 삽입하고 송신한다.  

호스트 A가 호스트 B로부터 0~535의 바이트를 포함하는 세그먼트와 900~1000의 바이트를 포함하는 세그먼트를 수신했다고 가정하자.  
어떤 이유 때문인지 536~899의 바이트를 아직 수신하지 않았다.  
호스트 A는 호스트 B의 데이터 스트림을 재생성하기 위해 536번째 바이트를 기다리고 있다.  
그러므로 B에 대한 A의 다음 세그먼트는 확인응답 번호 필드에 536을 가질 것이다.  
TCP는 첫 번째 읽어드린 바이트까지의 바이트들까지만 확인응답을 하기 때문에 TCP는 누적 확인응답이다.  

호스트 A는 세 번째 세그먼트를 두 번째 세그먼트가 수신되기 전에 수신했다.  
즉, 세 번째 세그먼트는 순사가 틀리게 도착했다.  
TCP는 2가지 선택지가 있다.  
1. 수신자가 순서가 바뀐 세그먼트를 즉시 버린다.  
2. 순서가 바뀐 데이터를 보유하고 빈 공간에 잃어버린 데이터를 채우기 위해 기다린다.  

확실히 후자가 네트워크 대역폭 관점에서 효율적이며, 실제로도 취하는 방법이다.  

실제로는 TCP 연결 양쪽 모두 시작범위를 임의로 선택한다.  
이것은 두 호스트 사이에 이미 종료된 연결로부터 아직 네트워크에 남아있던 세그먼트가 같은 두 호스트 간의 다중 연결에서 유효한 세그먼트로 오인될 확률을 최소화한다.  

## 텔넷: 순서번호와 확인응답 번호 사례연구

호스트 A가 호스트 B와 텔넷 세션을 시작한다고 가정하자. 호스트 A가 세션을 시작하므로 클라이언트이고 호스트 B는 서버가 된다.  
사용자가 입력한 각 문자는 원격 호스트에게 송신될 것이다.  
원격 호스트는 각 문자의 복사본을 송신자에게 반송하여 텔넷 사용자의 화면에 표시하게될 것이다.  

이 에코백(echo back)은 텔넷 사용자가 보는 문자가 이미 원격 사이트에 수신되고 처리되었음을 나타낸다.  
그러므로 각 문자는 사용자가 키를 누르는 시간과 문자가 사용자의 모니터에 표시되는 시간 사이에서 네트워크를 두 번 횡단한다.  

초기 순서번호가 클라이언트와 서버가 각각 42와 79라고 가정한다.  
세그먼트의 순서번호는 데이터필드안에 있는 첫 번째 바이트의 순서임을 상기하라. 

그러므로 클라이언트에서 송신된 첫 번째 세그먼트는 순서번호 42를 가질 것이다.  
확인응답 번호는 호스트가 기다리는 데이터의 다음 바이트의 순서번호임을 상기하자.  
TCP 연결이 설정된 후에 어떤 데이터로 송신되기 전에 클라이언트는 바이트 79를 기다리고 서버는 바이트 42를 기다리고 있다.  

첫 번째 세그먼트는 클라이언트에서 서버로 송신된다.  
세그먼트는 데이터 필드 안에 문자 'C'를 포함한다. 순서번호 필드는 42를 갖는다.  

두 번째 세그먼트는 서버에서 클라이언트로 송신된다. 이는 두 가지 목적을 갖는다.  
첫째 수신하는 서버에게 데이터에 대한 확인응답을 제공한다. 확인응답 필드 안에 43을 넣음으로써 서버는 클라이언트에게 바이트 42를 성공적으로 수신했고 앞으로 바이트 43을 기다린다.  
두번째는 문자 'C'에 대한 반향이다.  

그리고 이것은 서버가 보내는 데이터의 맨 첫 번째 바이트이며 서버와 클라이언트 간에서 데이터를 운반하는 세그먼트안에서 전달된다.  
이러한 확인응답은 서버-클라이언트 데이터 세그먼트상에서 피기백(piggyback)된다. 라고 한다.  

세 번째 세그먼트는 클라이언트에서 서버로 송신되는데, 그 목적은 서버로부터 수신한 데이터에 대한 확인응답을 하는 것이다.  
이 세그먼트는 빈 데이터 필드를 갖는다. 즉, 확인응답은 어떤 클라이언트-서버 데이터와 함께 피기백되지 않는다.  
세그먼트는 확인응답필드 80을 갖는다. 왜냐하면 클라이언트가 순서번호 79의 바이트를 통해 바이트 스트림을 수신했기 때문이다.  
이제 앞으로 80으로 시작하는 바이트를 기다린다.  


## 정리 및 요약


순서번호 필드와 확인응답 필드에 대한 내용을 정리하였다.  
TCP에서는 순서번호가 호스트의 데이터에 대한 스트림 번호를 통하여 정해진다.  

그리고 전이중 연결이기 때문에 데이터를 주고 받을 때는 서로의 데이터와 더불어 상대방의 데이터스트림에서 받기를 기대하는 바이트스트림 번호를 확인응답 필드에 넣고 자신의 순서번호 필드에는 데이터에 대한 스트림 번호를 넣는다.  

이때 데이터와 함께 확인응답이 전달되기 때문에 이런 경우를 피기백(piggyback, 업다.)된다 라고한다.  

또한 데이터스트림의 번호라고 하였지만 이 번호는 매 연결시마다 임의로 선택된다.  
왜냐하면 이전의 연결에서 주고받았던 세그먼트들이 네트워크상에서 있을 수 있기 때문에 연결시 마다 새로운 임의의 스트림 번호를 사용한다.  



