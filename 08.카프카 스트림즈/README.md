# 카프카 스트림즈

## 카프카 스트림즈 소개
### 카프카 스트림즈

![image](https://user-images.githubusercontent.com/40031858/173007751-9845725f-58ab-48cf-9c98-127012edf81e.png)

카프카 스트림즈는 토픽에 적재된 데이터를 실시간으로 변환하여 다른 토픽에 적재하는 라이브러리이다. 스트림즈는 카프카에서 공식적으로 지원하는 라이브러리이다.

스트림즈 애플리케이션 또는 카프카 브로커의 장애가 발생하더라도 정확히 한번(exactly once)할 수 있도록 장애 허용 시스템(fault tolerant system)을

가지고 있어서 데이터 처리 안정성이 매우 뛰어나다. 카프카 클러스털르 운영하면서 실시간 스트림 처리를 해야하는 필요성이 있다면 카프카 스트림즈 애플리케이션을 개발하는 것을 1순위로 고려하는것이 좋다.

### 프로듀서와 컨슈머를 조합하지 않고 스트림즈를 사용해야 하는 이유

스트림 데이터 처리에 있어 필요한 다양한 기능을 스트림즈DSL로 제공하며 필요하다면 프로세서 API를 사용하여 기능을 확장할 수 있기 때문이다.

컨슈머와 프로듀서를 조합하여 스트림즈가 제공하는 기능과 유사하게 만들 수 있다. 그러나 스트림즈 라이브러리를 통해 제공하는 단 한번의 데이터 처리, 장애 허용 

시스템 등의 특징들은 컨슈머와 프로듀서의 조합만으로는 완벽하게 구현하기는 어렵다.

### 스트림즈 내부 구조

![image](https://user-images.githubusercontent.com/40031858/173008789-77f0ed40-206a-4ddc-8e61-9b1eb1eec926.png)

스트밀즈 애플리케이션은 내부적으로 스레드를 1개 이상 생성할 수 있으며, 스레드는 1개 이상의 태스크를 가진다. 스트림즈의 `태스크`는 스트림즈 애플리케이션을 실행하면

생기는 데이터 처리 최소 단위이다. 

### 스트림즈 애플리케이션 스케일 아웃

![image](https://user-images.githubusercontent.com/40031858/173008931-12ebc396-a1cb-47a9-8f92-ad961bae47be.png)

실제 운영환경에서는 장애가 발생하더라도 안정적으로 운영할 수 있도록 2개 이상의 서버로 구성하여 스트림즈 애플리케이션을 운영한다. 이를 통해 일부 스트림즈

애플리케이션 또는 애플리케이션이 실행되는 서버에 장애가 발생하더라도 안전하게 스트림 처리를 할 수 있다.

### 토폴리지

![image](https://user-images.githubusercontent.com/40031858/173009060-7c2b519a-d921-4a8f-9b4d-8bad57066100.png)

카프카 스트림즈의 구조와 사용 방법을 알기 위해서는 우선 `토폴리지`와 관련된 개념을 익혀야 한다. 토폴리지란 2개 이상의 노드들과 선으로 이루어진 집합을 뜻한다.

### 소스 프로세서, 스트림 프로세서, 싱크 프로세서

![image](https://user-images.githubusercontent.com/40031858/173009154-e6ba9f35-de82-481e-874d-4fbf65e66d14.png)

프로세서에는 소스 프로세서, 스트림 프로세서, 싱크 프로세서 3가지가 있다. 소스 프로세서는 데이터를 처리하기 위해 최초로 선언해야 하는 노드로, 하나 이상의 토픽에서

데이터를 가져오는 역할을 한다. 스트림 프로세서는 다른 프로세서가 반환한 데이터를 처리하는 역할을 한다. 변환, 분기처리와 같은 로직이 데이터 처리의 일종이라고 볼 수 있다.

마지막으로 싱크 프로세서는 데이터를 특정 카프카 토픽으로 저장하는 역할을 하며 스트림즈로 처리된 데이터의 최종 종착지다.

### 스트림즈DSL과 프로세서 API

`스트림즈DSL`과 프로세서 API 2가지 방법으로 개발 가능하다. 스트림즈 DSL은 스트림 프로세싱에 쓰일만한 다양한 기능들을 자체 API로 만들어 놓았기 때문에

대부분의 변환 로직을 어렵지 않게 개발할 수 있다. 만약 스트림즈 DSL에서 제공하지 않는 일부 기능들의 경우 프로세서 API를 사용하여 구현할 수 있다.

`스트림즈DSL로 구현하는 데이터 처리 예시`
- 메시지 값을 기반으로 토픽 분기 처리
- 지난 10분간 들어온 데이터의 개수 집계

`프로세서 API로 구현하는 데이터 처리 예시`
- 메시지 값의 종류에 따라 토픽을 가변적으로 전송
- 일정한 시간 간격으로 데이터 처리

## 스트림즈DSL

스트림즈DSL에는 레코드의 흐름을 추상화한 3가지 개념인 KStream,KTable,GlobalKTable이 있다. 이 3가지 개념은 컨슈머, 프로듀서,프로세서API에서는

사용되지 않고 스트림즈 DSL에서만 사용되는 개념이다.

----

## KStream, KTable, GlobalKTable

### KStream

![image](https://user-images.githubusercontent.com/40031858/173010065-e6275e79-9f5b-4f73-9980-1b904ec68520.png)

KStream은 레코드의 흐름을 표현한 것으로 메시지 키와 메시지 값으로 구성되어 있다. KStream으로 데이터를 조회하면 토픽에 존재하는 모든 레코드가 출력된다.

KStream은 컨슈머로 토픽을 구독하는 것과 동일한 선상에서 사용하는 것이라고 볼 수 있다.

### KTable

![image](https://user-images.githubusercontent.com/40031858/173010223-efe4c406-aec5-4366-a892-ac8edc424904.png)

KTable은 KStream과 다르게 미시지 키를 기준으로 묶어서 사용한다. KStream은 토픽의 모든 레코드를 조회할 수 있지만 KTable은 유니크한 메시지 키를 기준으로

가장 최신 레코드를 사용한다. 그러므로 KTabel로 데이터를 조회하면 메시지 키를 기준으로 가장 최신에 추가된 레코드의 데이터가 출력된다. 새로 데이터를 적재할 때

동일한 메시지 키가 있을 경우 데이터가 업데이트 되었다고 볼 수 있다.

### 코파티셔닝

![image](https://user-images.githubusercontent.com/40031858/173010449-e4ce244b-215b-415f-a006-08c92708fba6.png)

KStream과 KTable 데이터를 조인한다고 가정하자. KStream과 KTable을 조인하려면 반드시 코파티셔닝되어 있어야 한다. 코파티셔닝이란 조인을 하는 2개 데이터의 파티션

개수가 동일하고 파티셔닝전략을 동일하게 맞추는 작업이다. 파티션 개수가 동일하고 파티셔닝 전략이 같은 경우에는 동일한 메시지 키를 가진 데이터가 동일한 태스크에 들어가는 것을

보장한다. 이를 통해 각 태스크는 KStream의 레코드와 KTable의 메시지 키가 동일할 경우 조인을 수행할 수 있다.

### 코파티셔닝되지 않은 2개 토픽의 이슈

![image](https://user-images.githubusercontent.com/40031858/173010747-cb5b66ce-d297-46c8-838e-2f0845d18652.png)

문제는 조인을 수행하려는 토픽들이 코파티셔닝되어 있음을 보장할 수 없다는 것이다. KStream과 KTable로 사용하는 2개의 토픽이 파티션 개수가 다를 수도 있고 파티션

전략이 다를수 있다. 이런 경우에는 조인을 수행할 수 없다. 코파티셔닝이 되지 않은 2개의 토픽을 조인하는 로직이 담긴 스트림즈 애플리케이션을 실행하면 TopologyException이 발생한다.

### GlobalKTable

![image](https://user-images.githubusercontent.com/40031858/173010944-ff23dac5-4e66-400a-951c-a193377aac70.png)

코파티셔닝되지 않은 KStream과 KTable을 조인해서 사용하고 싶다면 KTable을 GlobalKTable로 선언하여 사용하면 된다. GlobalKTable은 코파티셔닝되지 않은 KStream과 데이터 

조인을 할 수 있다. 왜냐하면 KTable과 다르게 GlobalKTable로 정의된 데이터는 스트림즈 애플리케이션의 모든 태스크에 동일하게 공유되어 사용되기 때문이다.

## 스트림즈 주요 옵션 소개

### 스트림즈DSL 중요 옵션(필수 옵션)
- `bootstrap.servers`: 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커의 호스트이름:포트를 1개 이상 작성한다
    
    2개 이상 브로커 정볼르 입력하여 일부 브로커에 이슈가 발생하더라도 접속하는 데에 이슈가 없도록 설정 가능하다.
- `application.id` : 스트림즈 애플리케이션을 구분하기 위한 고유한 아이디를 설정한다. 다른 로직을 가진 스트림즈 애플리케이션들은 서로 다른 application.id 값을 가져야한다.

### 스트림즈DSL 중요 옵션(선택 옵션)
- `default.key.serde`: 레코드의 메시지 키를 직렬화, 역직렬화하는 클래스를 지정한다. 기본값은 바이트 직렬화, 역직렬화 클래스인 Serdes.ByteArray().getClass().getName()이다.
- `default.value.serde` : 레코드의 메시지 값을 직렬화, 역직렬화하는 클래스를 지정한다. 기본값은 바이트 직렬화, 역직렬화 클래스인 Serds.ByteArray().getClass().getName()이다.
- `num.stream.threads` : 스트림 프로세싱 실행 시 실행될 스레드 개술르 지정한다. 기본값은 11이다
- `state.dir` : 상태기반 데이터 처리를 할 때 데이터를 저장할 디렉토리를 지정한다. 기본값은 /tmp/kafka-streams이다.

---

## 스트림즈DSL의 윈도우 프로세싱

### 스트림즈DSL - window processing

스트림 데이털르 분석할 때 가장 많이 활용하는 프로세싱 중 하나는 윈도우 연산이다. 윈도우 연산은 특정 시간에 대응하여 취합 연산을 처리할 때 활용한다.

모든 프로세싱은 메시지 키를 기준으로 취합된다. 그러므로 해당 토픽에 동일한 파티션에는 동일한 메시지 키가 있는 레코드가 존재해야지만 정확한 취합이 가능하다.

만약 커스텀 파티셔너를 사용하여 동일 메시지 키가 동일한 파티션에 저장되는 것을 보장하지 못하거나 메시지 키를 넣지 않으면 관련 연산이 불가능하다.

카프카 스트림즈에서 제공하는 윈도우 연산 종류는 다음과 같다
- 텀블링 윈도우
- 호핑 윈도우
- 슬라이딩 윈도우
- 세션 윈도우

### 스트림즈DSL - 텀블링 윈도우

![image](https://user-images.githubusercontent.com/40031858/173055742-6b043faa-45fe-401e-8f3b-ad24c744b1e4.png)

텀블링 윈도우는 서로 겹치지 않은 윈도우를 특정 간격으로 지속적으로 처리할 때 사용한다. 윈도우 최대 사이즈에 도달하면 해당시점에 데이터를 취합하여 결과를 도출한다.

텀블링 윈도우는 단위 시간당 데이터가 필요할 경우 사용할 수 있다. 예를 들어 5분간 접속한 고객의 수를 측정하여 방문자 추이를 실시간 취합하는 경우 사용할 수 있다.

### 스트림즈DSL - 호핑 윈도우

![image](https://user-images.githubusercontent.com/40031858/173055919-0791fb95-ef1f-437b-b971-5d2bcca8963b.png)

호핑 윈도우는 일정 시간 간격으로 겹치는 윈도우가 존재하는 윈도우 연산을 처리할 경우 사용한다. 호핑 윈도우는 윈도우 사이즈와 윈도우 간격 2가지 변수를 가진다.

윈도우 사이즈는 연산을 수행할 최대 윈도우 사이즈를 뜻하고 윈도우 간격은 서로 다른 윈도우 간 간격을 뜻한다. 텀블링 윈도우와 다르게 동일한 키의 데이터는 서로 다른 윈도우에서 여려번 연산가능

### 스트림즈DSL - 슬라이딩 윈도우

![image](https://user-images.githubusercontent.com/40031858/173056103-3ab6fc74-06e0-472a-82e9-9b28f55ec97e.png)

슬라이딩 윈도우는 호핑 윈도우와 유사하지만 데이터의 정확한 시간을 바탕으로 윈도우 사이즈에 포함되는 데이터를 모두 연산에 포함시키는 특징이 있다.

### 스트림즈DSL- 세션 윈도우

![image](https://user-images.githubusercontent.com/40031858/173056192-07ada2ee-0469-4a47-9ca9-dbed6b52828a.png)

세션 윈도우는 동일 메시지 키의 데이터를 한 세션에 묶어 연산할 때 사용한다. 세션의 최대 만료시간에 따라 윈도우 사이즈가 달라진다.

세션 만료 시간이 자나게 되면 세션 윈도우가 종료되고 해당 윈도우의 모든 데이터를 취합하여 연산한다. 그렇기에 세션윈도우의 윈도우 사이즈는 가변적이다.

### 텀블링 윈도우 예제

텀블링 윈도우를 사용하기 위해서는 groupByKey와 windowedBy를 사용해야 한다. windowedBy의 파라미터는 텀블링 윈도우의 사이즈를 뜻한다.

이후 텀블링 연산으로 출력된 데이터는 KTable로 커밋 inerval마다 출력된다.

```java
StreamsBuilder builder = new StreamsBuilder();
KStream<String,String> stream = builder.stream(TEST_LOG);

KTable<Widowed<String>, Long> countTable = stream.groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofSeconds(5)))
    .count();

countTable.toStream().foreach(((key,value) -> {
    log.info(key.key() + " is [" + key.window().startTime() + "~" + key.window().endTime()+"] count:" +value)
;}));
```

## 스트림즈DSL의 Queryable store

카프카 스트림즈에서 KTable은 카프카 토픽의 데이터를 로컬의 rocksDB에 Materialized View로 만들어 두고 사용하기 떄문에 레코드의 메시지 키 , 메시지 값을 

기반으로 keyValueStore로 사용할 수 있다. 특정 토픽을 KTable로 사용하고 ReadOnlyKeyValueStore로 뷰를 가져오면 메시지 키를 기반으로

토픽 데이터를 조회할 수 있게 된다. 카프카를 사용해 로컬캐시를 구현한것과 유사하다고 볼 수 있다.


```java
ReadOnlyKeyValueStore<String,String> keyValueStore;
StreamBuilder builder = new StreamBuilder();
KTable<String,String> addressTable = builder.table(ADDRESS_TABLE, Materialized.as(ADDRESS_TABLE));

keyValueStore= streams.store(StoreQueryParameters.fromNameAndType(ADDRESS_TABLE,
    QueryableStoreTypes.keyValueStore()));

keyValueIterator<String,String> address = keyValueStore.all();
address.forEachRemaining(keyValue -> log.info(keyValue.toString()));
```

---

## 프로세서 API

프로세서 API는 스트림즈 DSL보다 투박한 코드를 가지지만 토폴리지를 기준으로 데이터를 처리한다는 관점에서는 동일한 역할을 한다. 스트림즈DSL은 데이터처리 ,분기, 조인을 위한

다양한 메서드들을 제공하지만 추가적인 상세 로직의 구현이 필요하다면 프로세서 API를 활용할 수 있다. 프로세서 API에서는 스트림즈DSL에서 사용했던

KStream, KTable, GlobalKTable 개념이 없다는 점을 주의해야 한다. 다만 스트림즈 DSL과 프로세서 API는 함께 구현하여 사용할 때는 활용할 수 있다.

프로세서API를 구현하기 위해서는 `Processor` 또는 `Transformer` 인터페이스로 구현한 클래스가 필요하다. Processor인터페이스는 일정 로직이 이루어진 뒤

다음 프로세서로 데이터가 넘어가지 않을 때 사용하고 Transformer인터페이스는 일정 로직이 이루어진 뒤 다음 프로세서로 데이터를 넘길때 사용한다

```java
public class FilterProcessor implements Processor<String,String>{
    private ProcessorContext context;

    @Override
    public void init(ProcessorContesxt context){this.context = context;}

    @Override
    public void process(String key, String value){
        if(value.length() > 5){
            context.forward(key,value);
        }
        context.commit();
    }
    @Override
    public void close(){
    }
}

```

```java
public class SimpleKafkaProcessor{
    ...
    public static void main(String[] args){
        ...
        
        Topology topology = new Topology();
        topology.addSource("Source", STREAM_LOG)
            .addProcessor("Process",
                ()-> new FIlterProcessor(),
                    "Source")
            .addSink("Sink", STREAM_LOG_FILTER, "Process");
        
        KafkaStreams streaming = new KafkaStreams(toplogy,props);
        streaming.start();
    }
}
```