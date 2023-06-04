# 왕복 시간(RTT) 예측과 타임아웃

rdt 프로토콜 처럼 TCP가 손실 세그먼트를 처리하기 위해 타임아웃/재전송 매커니즘을 사용한다.  
이는 미묘한 상황이 많다. 아마 대부분 타임아웃 주기일 것이다.  

분명 타임아웃은 세그먼트가 전송된 시간부터 긍정확인응답될 때 까지의 시간인 RTT보다 좀 커야한다.  
그렇지 않다면 불필요한 재전송이 발생할 것이다.  

## 왕복시간 예측 

SampleRTT라고 표시되는 세그먼트에 대한 RTT 샘플은 세그먼트가 송신된 시간으로부터 그 세그먼트에 대한 긍정 확인응답이 도착한 시간까지의 길이이다.  

모든 전송된 세그먼트에 대해 SampleRTT를 측정하는 대신, 대부분의 TCP는 한번에 하나의 SampleRTT 측정만을 시행한다.  
즉, 어떤 시점에서 SampleRTT는 전송되었지만 현재까지 확인응답이 없는 세그먼트 중 하나에대해서만 측정되며 이는 대략 왕복시간 마다 SampleRTT의 새로운 값을 얻게한다.  
또한 재전송한 세그먼트에 대한 SampleRTT는 계산하지 않는다.  

SampleRTT의 값은 라우터에서의 혼잡과 종단 시스템에서의 부하변화 때문에 세그먼트마다 다르다.  
이러한 변동성 때문에 주어진 SampleRTT 값의 평균값을 채택한다. TCP는 SampleRTT의 값을 유지한다.  
긍정확인응답을 수신하고 새로운 SampleRTT를 획득하자마자 TCP는 다음 공식에 따라 EstimatedRTT를 계산한다.  

$EsimatedRTT=(1-a)EstimatedRTT+aSampleRTT$

EstimatedRTT의 새로운 값은 EstimatedRTT의 이런 값과 SampleRTT에 대한 새로운 값의 가중된 조합이다.  
권장되는 a의 값은 0.125이다.  

$EsimatedRTT=0.875EstimatedRTT+0.125SampleRTT$

EstimatedRTT는 SampleRTT값의 가중평균(weighted average)임을 유념하라.  
이 가중평균은 예전 샘플보다 최근 샘플에 더 높은 가중치를 준다.  

최근 샘플들이 네트워크 상의 현재 혼잡을 더 잘 반영한다.
통계에서 이런 평균은 지수적 가중 이동평균(Exponential Weighted Moving Average, EWMA)라고 부른다.  
SampleRTT의 가중치가 갱신절차가 진행됨에 따라 빠르게 지수적으로 감소하므로 EWMA에서 지수적이라는 용어가 쓰인다.  

RTT의 예측 외에 RTT의 변화율을 측정하는 것도 매우 유용하다. 변화율을 의미하는 DevRTT는 SampleRTT가 EstimatedRTT로부터 얼마나 많이 벗어나는지에 대한 예측으로 정한다.  

$DevRTT=(1-b)DevRTT+b|sampleRTT-EstimatedRTT|$

DevRTT는 SampleRTT와 EstimatedRTT 값 차이의 EWMA임을 유념하라.  
SampleRTT 값이 어떠한 변화도 없다면 DevRTT는 작을 것이며 그렇지 않다면 DevRTT는 클 것이다.  
b의 권장 값은 0.25이다.  

## 재전송 타임아웃 주기의 설정과 관리  

주이전 EstimatedRTT와 DevRTT의 값에서, TCP 타임아웃 주기에는 어떤 값이 사용되어야 하는가?  
분명히 타임아웃 주기는 EstimatedRTT보다 크거나 같아야 한다. 그렇지 않다면 불필요한 재전송이 보내질 것이다.  

그러나 타임아웃주기는 EstimatedRTT 값보다 너무 크면 안된다.  
너무크면 세그먼트를 잃었을 때 TCP는 세그먼트의 즉각적인 재전송을 하지 않게 된다.  
이 때문에 애플리케이션에서 확연한 데이터 전송지연이 나타난다.  

그러므로 타임아웃 주기는 EstimatedRTT에 약간의 여윳값을 더한 값으로 설정되어야 한다.  
SampleRTT의 값에 많은 변동값이 있을 때는 여윳값이 커야하며, 변동이 적을 때는 작아야  한다.  

따라서 DevRTT의 값이 역할을 하게 된다.

$TimeOutInterval=EstimatedRTT+4DevRTT$

초기 TimeOutInterval의 값을 1초로 권고하고 타임아웃이 발생하면 두배로 하며, 조기 타임아웃을 피하도록한다.  
그러나 세그먼트가 수신되고 EstimatedRTT가 수정되면 TimeOutInterval은 위의 공식에 따라 계산된다.  


## 정리 및 요약

TCP의 타임아웃에 대한 내용을 요약하였다.  

타임아웃의 주기는 패킷이 상대방에게 갔다 확인응답이 돌아오기 까지 걸리는 시간인 RTT보다 조금 더 커야할 것이다.  

이러한 수치를 어떻게 계산할 것인가?  
TCP는 SampleRTT와 EstimatedRTT, DevRTT를 이용하여 계산하였다.  
이 계산에는 이전의 수치값보다 최근의 수치에 더 많은 가중치를 두는 지수적 가중 이동평균(EWMA)를 사용하였다.  


계산에서 나온 TimeOutInterval을 타임아웃 주기로 사용하고 초기에는 1초를 사용하며 타임아웃이 발생할 때 두배로 한다.  

세그먼트가 수신되면 EstimatedRTT가 수정되고 TimeOutInterval의 계산에따라 갱신된다.  



