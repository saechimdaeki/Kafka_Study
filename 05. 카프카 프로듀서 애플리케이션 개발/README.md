# 카프카 프로듀서 애플리케이션 개발

## 프로듀서 소개

![image](https://user-images.githubusercontent.com/40031858/170871494-ba5bae12-29d7-49b7-be66-843cfd9d106a.png)

카프카에서 데이터의 시작점은 프로듀서이다. 프로듀서 애플리케이션은 카프카에 필요한 데이터를 선언하고 브로커의 특정 토픽의 파티션에 전송한다.

프로듀서는 데이터를 전송할 때 리더 파티션을 가지고있는 카프카 브로커와 직접 통신한다. 프로듀서는 카프카 브로커로 데이터를 전송할 때 내부적으로 파티셔너, 배치 생성단꼐를 거친다.

![image](https://user-images.githubusercontent.com/40031858/170871555-fd72fc5f-ac7b-456b-92cc-d44a0ad7ad32.png)

- ProducerRecord : 프로듀서에서 생성하는 레코드. 오프셋은 미포함
- send() : 레코드를 전송 요청 메소드
- Partitioner : 어느 파티션으로 전송할지 지정하는 파티셔너. 기본값으로 DefaultPartitioner로 설정됨
- Accumulator : 배치로 묶어 전송할 데이터를 모으는 버퍼

---

## 파티셔너
### 프로듀서의 기본 파티셔너
프로듀서 API를 사용하면 'UniformStickyPartitioner'와 'RoundRobinPartitioner' 2개 파티셔너를 제공한다. 2.5.0버전에서 파티셔너를 지정하지 않은경우 UniformStickyPartitioner가 기본설정된다

`메시지 키가 있을 경우 동작`
- UniformStickyPartitioner와 RoundRobinPartitioner 둘 다 메시지 키가 있을때는 메시지 키의 해시값과 파티션을 매칭하여 레코드를 전송
- 동일한 메시지 키가 존재하는 레코드는 동일한 파티션 번호에 전달됨
- 만약 파티션 개수가 변경될 경우 메시지 키와 파티션 번호 매칭은 깨지게 됨

`메시지 키가 없을 때 동작`
- 메시지 키가 없을 때는 파티션에 최대한 동일하게 분배하는 로직이 들어있는데 UniformStickyPartitioner는 RoundRobinPartitioner의 단점을 개선하였다는 점이 다르다

`RoundRobinPartitioner`
- ProducerRecord가 들어오는 대로 파티션을 순회하면서 전송
- 어큐뮤레이터에서 묶이는 정도가 적기 때문에 전송 성능이 낮음

`UniformStickyPartitioner`
- 어큐뮤레이터에서 레코드들이 배치로 묶일 때까지 기다렸다가 전송
- 배치로 묶일 뿐 결국 파티션을 순회하면서 보내기 때문에 모든 파티션에 분배되어 전송됨
- RoundRobinPartitioner에 비해 향상된 성능을 가짐

### 프로듀서의 커스텀 파티셔너
카프카 클라이언트 라이브러리에서는 사용자 지정 파티셔너를 생성하기 위한 Partitioner 인터페이스를 제공한다.

Partitioner 인터페이스를 상속받은 사용자 정의 클래스에서 메시지 키 또는 메시지 값에 따른 파티션 지정 로직을 적용할수도 있다.

파티셔너를 통해 파티션이 지정된 데이터는 어큐뮬레이터에 버퍼로 쌓인다. 센더 스레드는 어큐뮬레이터에 쌓인 데이터를 가져가 카프카 브로커로 전송한다.