# Webflux, Reactor란?

## 사용하는 이유

## 1. 용어 정리
Publisher : 발행자, 게시자, 생산자, 방출자(Emitter)
Subscriber : 구독자, 소비자
Emit : Publisher가 데이터를 내보내는 것(방출하다. 내보내다. 통지하다.)
Sequence : Publisher가 emit하는 데이터의 연속적인 흐름. 스트림과 같은 의미라고 보면 됨
Subscribe : Subscriber가 Sequence를 구독하는 것
Dispose : Suscriber가 Sequence 구독을 해지 하는 것
Downstream : 현재 Operator 체인의 위치에서 봤을때 데이터가 전달 되는 하위 Operator 및 method 체인
Upstream : 현재 Operator 체인의 위치에서 봤을때 상위 Operator 및 method 체인


## 2. Operators UpStream, DownStream 방향 정리
Reactive Streams의 핵심 개념은 Publisher -> Data -> Subscriber의 흐름으로 데이터가 전달된다는 것이다.
아래에서 <- 방향으로의 흐름을 업스트림(Upstream)이라 하고 -> 방향으로의 흐름을 다운스트림(Downstream)이라 한다.
그리고 데이터는 업스트림에서 다운스트림 방향으로 (->) 흘러간다.
Publisher -> Data -> Subscriber
<- subscribe(Subscriber)
-> onSubscribe(Subscription)
-> onNext
-> onNext
-> ...
-> onComplete                  
Reactive Streams에서는 이 과정에서 Publisher -> [Data1] -> Operator -> [Data2] -> Operator2 -> [Data3] -> Subscriber 이런식으로 데이터를 가공하는 Operator를 적용할 수 있다.
아래와 같은 Publisher와 Subscriber가 있다고 해보자. 1부터 10까지 정수 데이터가 발생하고, Subscriber는 화면에 출력하고 프로그램은 종료된다.


## 3. Backpressure
upstream에 신호를 전파하는 것은 backpressure를 개발하여 사용할 수 있는데, backpressure는 조립라인에서 워크스테이션이 upstream 워크스테이션보다 더 느리게 처리될 때 전송되는 피드백 신호와 비슷하다.

Subscriber가 unbound mode에서 작업하고 source가 도달 가능한 가장 빠른 속도로 푸쉬하도록 하거나 요청 메커니즘을 사용하여 약 n개의 요소를 처리할 준비가 되어있다는 신호를 source에게 보낼 수 있는데, Reactive Stream 스펙에서 정의되는 실제 메커니즘은 이와 유사하다.

중간 operator는 전송 중인 요청을 변경할 수도 있다. 예를 들어, 데이터를 10개 묶음으로 그룹화하는 buffer operator를 생각해보자. subscriber가 하나의 버퍼를 요청한다면, source가 10개의 데이터를 생산하는 것이 허용된다. 일부 oprator는 request(1) round-trip을 방지하고 요청되기 전에 요소를 생산하는 것이 비용이 많이 들지 않으면 prefetching 전략을 구현한다.

이는 push 모델을 push-pull 하이브리드 모델로 변형하는데, 요소가 이미 사용 가능하다면 downstream은 n개의 요소를 upstream으로부터 받아올 수 있다. 하지만, 요소가 당장 사용가능하지 않다면 생산될 때 마다 upstream에서 요소들을 푸쉬 해준다.



## Hot vs Cold
- Reactive 라이브러리는 Hot 과 Cold 라는 두 가지의 리액티브 시퀀스를 구별한다. 이러한 구분은 주로 reactive stream이 subscriber에게 어떻게 반응하는지와 관련된다.

- Cold sequence는 각 subscriber에게 데이터 소스를 포함하여 시퀀스가 새로 시작된다. 예를 들어, source가 HTTP 통신을 사용한다면, 새로운 HTTP 요청이 각 구독마다 새로 만들어진다.
Hot sequence는 각 subscriber에 대해 시퀀스가 처음부터 시작되지 않는다. 대신에, 나중에 들어온 subscriber는 구독 후 방출되는 신호를 수신한다. 그러나, Hot reactive stream은 전체 또는 일부 방출된 이력을 캐싱하거나 재현할 수 있다. Hot sequence는 아무도 구독하지 않은 경우에도 Hot sequence가 방출될 수 있다.


source가 생산해내는 데이터의 특징에 따라 Hot과 Cold 중 어떤걸 사용해야 할지 생각을 하면 좋을 것 같다. 당장 떠오르는 것은 시간에 따라 민감하게 바뀌는 데이터라면 Cold를, 그렇지 않은 경우에는 Hot을 선택하면 좋지 않을까 싶다.
