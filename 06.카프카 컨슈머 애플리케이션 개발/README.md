# 카프카 컨슈머 애플리케이션 개발

## 컨슈머 소개

![image](https://user-images.githubusercontent.com/40031858/172329774-68bfc7b4-b1e7-4ece-afcf-de70a39ef2ca.png)

프로듀서가 전송한 데이터는 카프카 브로커에 적재된다. 컨슈머는 적재된 데이터를 사용하기 위해 브로커로부터 데이터를 가져와서 필요한 처리를 한다.

![image](https://user-images.githubusercontent.com/40031858/172329903-da87473f-598b-4e48-8d09-8d1ce765df74.png)

- Fetcher : 리더 파티션으로부터 레코드들을 미리 가져와서 대기
- poll() : Fetcher에 있는 레코드들을 리턴하는 레코드
- ConsumerRecords : 처리하고자 하는 레코드들의 모음. 오프셋이 포함되어 있음.

## 컨슈머 그룹

![image](https://user-images.githubusercontent.com/40031858/172273441-0910aac1-485e-4ddf-a852-3e65906d4408.png)

컨슈머 그룹으로 운영하는 방법은 컨슈머를 각 컨슈머 그룹으로부터 격리된 환경에서 안전하게 운영할수 있도록 도와주는 카프카의 독특한 방식이다.

컨슈머 그룹으로 묶인 컨슈머들은 토픽의 1개 이상 파티션들에 할당되어 데이터를 가져갈 수 있다. 컨슈머 그룹으로 묶인 컨슈머가 토픽을 구독해서 데이터를 가져갈 때, 1개의 파티션은

최대 1개의 컨슈머에 할당 가능하다. 그리고 1개 컨슈머는 여러개의 파티션에 할당될 수 있다. 이러한 특징으로 컨슈머 그룹의 컨슈머 개수는

가져가고자 하는 토픽의 파티션 개수보다 같거나 작아야 함.

### 컨슈머 그룹의 컨슈머가 파티션 개수보다 많을 경우 

![image](https://user-images.githubusercontent.com/40031858/172273611-5036f08e-879f-4bff-ae49-bdf6892739bb.png)

만약 4개의 컨슈머로 이루어진 컨슈머 그룹으로 3개의 파티션을 가진 토픽에서 데이터를 가져가기 위해 할당하면 1개의 컨슈머는 파티션을 할당받지 못하고 유휴상태로 남게된다.

즉, 스레드만 차지하고 데이터처리를 못하므로 불필요한 스레드이다.

### 컨슈머 그룹의 컨슈머가 파티션 개수보다 많을 경우

![image](https://user-images.githubusercontent.com/40031858/172273767-f4f96a4c-8d56-4a3c-9cac-cf2eb38530c6.png)

카프카는 이러한 파이프라인을 운영함에 있어 최종 적재되는 저장소의 장애에 유연하게 대응할 수 있도록 각기 다른 저장소에 저장하는 컨슈머를 다른 컨슈머 그룹으로

묶음으로써 각 저장소의 장애에 격리되어 운영할 수 있다. 따라서 엘라스틱서치의 장애로 인해 더는 적재가 되지 못하더라도 하둡으로 데이터를 적재

하는데에는 문제가 없다. 엘라스틱서치의 장애가 해소되면 엘라스틱 서치로 적재하는 컨슈머의 컨슈머 그룹은 마지막으로 적재 완료한 데이터 이후부터 다시 적재를 수행하여 최종적으로 모두 정상화될 것이다.


## 리밸런싱

![image](https://user-images.githubusercontent.com/40031858/172331433-6dd11cef-9b76-4f02-8796-1c92956042d8.png)

컨슈머 그룹으로 이루어진 컨슈머들 중 일부 컨슈머에 장애가 발생하면, 장애가 발생한 컨슈머에 할당된 파티션은 장애가 발생하지 않은 컨슈머에게 소유권이 넘어간다.

이러한 과정을 `리밸런싱`이 라고 부른다. 리밸런싱은 크게 두가지 상황에서 일어나는데, 첫 번째는 컨슈머가 추가되는 상황이고 두번 째는 컨슈머가 제외되는 상황이다. 

이슈가 발생한 컨슈머를 컨슈머 그룹에서 제외하여 모든 파티션이 지속적으로 데이터를 처리할 수 있도록 가용성을 높여준다.

## 커밋

![image](https://user-images.githubusercontent.com/40031858/172333118-6d36ee46-9a0d-4a0e-9b22-d35a612f611d.png)


컨슈머는 카프카 브로커로부터 데이터를 어디까지 가져갔는지 커밋을 통해 기록한다. 특정 토픽의 파티션을 어떤 컨슈머 그룹이 몇 번째 가져갔는지 카프카 브로커 내부에서 사용되는 내부 토픽에 기록된다.

컨슈머 동작 이슈가 발생하여 __consumer_offsets 토픽에 어느 레코드까지 읽어갔는지 오프셋 커밋이 기록되지 못했다면 데이터 처리의 중복이 발생할 수 있다.  

그러므로 데이터 처리의 중복이 발생하지 않게 하기 위해서는 컨슈머 애플리케이션이 오프셋 커밋을 정상적으로 처리했는지 검증해야함.

## 어사이너

컨슈머와 파티션 할당 정책은 컨슈머의 Assignor에 의해 결정된다. 카프카에서는 RangeAssignor, RoundRobinAssignor, StickyAssignor를 제공한다. 

카프카 2.5.0는 RangeAssignor가 기본값으로 설정된다

- RangeAssignor : 각 토픽에서 파티션을 숫자로 정렬, 컨슈머를 사전 순서로 정렬하여 할당
- RoundRobinAssignor : 모든 파티션을 컨슈머에서 번갈아가면서 할당
- StickyAssignor : 최대한 파티션을 균등하게 배분하면서 할당

## 컨슈머 주요 옵션

### 필수 옵션
- boostrap.servers : 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커의 호스트 이름:포트를 1개 이상 작성한다. 2개 이상 브로커 정보를 입력해 일부 브로커에 

    이슈가 발생하더라도 접속하는 데에 이슈가 없도록 설정 가능하다
- key.deserializer : 레코드의 메시지 키를 역직렬화하는 클래스를 지정한다
- value.deserializer : 레코드의 메시지 값을 역직렬화하는 클래스를 지정한다

### 선택 옵션
- group.id : 컨슈머 그룹 아이디를 지정한다. subscribe()메소드로 토픽을 구독하여 사용할 때는 이옵션을 필수로 넣어야 한다. 기본값은 null이다
- auto.offset.reset : 컨슈머 그룹이 특정 파티션을 읽을 때 저장된 컨슈머 오프셋이 없는 경우 어느 오프셋부터 읽을지 선택하는 옵션이다.

    이미 컨슈머 오프셋이 있다면 이 옵션값은 무시된다. 기본값은 latest이다.
- enable.auto.commit : 자동 커밋으로 할지 수동 커밋으로 할지 선택한다. 기본값은 true다.
- auto.commit.interval.ms : 자동 커밋일 경우 오프셋 커밋 간격을 지정한다. 기본값은 5000(5초)다.
- max.poll.records: poll() 메서드를 통해 반환되는 레코드 개수를 지정한다. 기본값은 500이다.
- session.timeout.ms : 컨슈머가 브로커와 연결이 끊기는 최대 시간이다. 기본값은 10000(10초)이다.
- hearbeat.interval.ms : 하트비트를 전송하는 시간 간격이다. 기본값은 3000(3초)이다.
- max.poll.interval.ms : poll()메서드를 호출하는 간격의 최대 시간. 기본값은 300000(5분)이다.
- isolation.level : 트랜잭션 프로듀서가 레코드를 트랜잭션 단위로 보낼 경우 사용한다.

### auto.offset.reset
컨슈머 그룹이 특정 파티션을 읽을 때 저장된 컨슈머 오프셋이 없는 경우 어느 오프셋부터 읽을지 선택하는옵션이다.

이미 컨슈머 오프셋이 있다면 이 옵션값은 무시된다. 이 옵션은 latest, earliest, none 중 1개를 설정할 수 있다.
- latest : 설정하면 가장 높은(가장 최근에 넣은) 오프셋부터 읽기 시작한다
- earliest : 설정하면 가장 낮은 (가장 오래전에 넣은) 오프셋부터 읽기 시작한다
- none: 설정하면 컨슈머 그룹이 커밋한 기록이 있는지 찾아본다. 만약 커밋 기록이 없으면 오류를 반환하고, 커밋 기록이 있다면 기존 커밋 기록 이후 오프셋부터 읽기 시작한다. 기본값은 latest다.

---

### 동기 오프셋 커밋 컨슈머
poll() 메서드가 호출된 이후에 commitSync() 메서드를 호출하여 오프셋 커밋을 명시적으로 수행할 수 있다.

commitSync()는 poll()메서드로 받은 가장 마지막 레코드의 오프셋을 기준으로 커밋한다. 동기 오프셋 커밋을 사용할 경우에는

poll() 메서드로 받은 모든 레코드의 처리가 끝난 이후 commitSync() 메서드를 호출해야한다

```java
KafkaConsumer<String,String> consumer = new KafkaConsumer<>(configs);
consumer.subscribe(Arrays.asList(TOPIC_NAME));

while(true){
    ConsumerRecords<String,String> records = consumer.poll(Duration.ofSeconds(1));
    for(ConsumerRecord<String,String> record : records){
        logger.info("record:{}" , record);
    }
    consumer.commitSync();
}
```

### 동기 오프셋 커밋(레코드 단위) 컨슈머
```java
KafkaConsumer<String,String> consumer = new KafkaConsumer<>(configs);
consumer.subscribe(Arrays.asList(TOPIC_NAME));
while(true){
    ConsumerRecords<String,String> records = consumer.poll(Duration.ofSeconds(1));
    Map<TopicPartition, OffsetAndMetadata> currentOffset = new HashMap<>();
    
    for(ConsumerRecord<String,String> record : records){
        logger.info("record:{}" , record);

        currentOffset.put(
            new TopicPartition(record.topic(), record.partition()),
            new OffsetAndMetadata(record.offset() + 1, null));
        consumer.commitSync(currentOffset);
    }
}
```

### 비동기 오프셋 커밋 컨슈머
동기 오프셋 커밋을 사용할 경우 커밋 응답을 기다리는 동안 데이터 처리가 일시적으로 중단 되기 때문에 더 많은 데이터를 처리하기 위해서 비동기 오프셋 커밋을 사용할 수 있다.

비동기 오프셋 커밋은 commitAsync()메서드를 호출하여 사용할 수 있다.
```java
KafkaConsumer<String,String> consumer = new KafkaConsumer<>(configs);
consumer.subscribe(Arrays.asList(TOPIC_NAME));

while(true){
    ConsumerRecords<String,String> records = consumer.poll(Duration.ofSeconds(1));
    for(ConsumerRecord<String,String> record : records){
        logger.info("record:{}", record);
    }
    consumer.commitAsync();
}
```

### 비동기 오프셋 커밋 롤백
```java
while(true){
    ConsumerRecords<String,String> records = consumer.poll(Duration.ofSeconds(1));
    for(ConsumerRecord<String,String> record : records){
        logger.info("record:{}" , record);
    }
    consumer.commitAsync(new OffsetCommitCallback(){
        public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception e{
            if( e != null)
                System.err.println("Commit failed");
            else
                System.err.println("Commit succeeded");
            if( e != null)
                logger.error("Commit failed for offsets {}", offsets, e);
        }
    });
}
```

----
### 리밸런스 리스너를 가진 컨슈머

리밸런스 발생을 감지하기 위해 카프카 라이브러리는 ConsumerRebalanceListener 인터페이스를 지원한다. 

ConsumerRebalanceListener 인터페이스로 구현된 클래스는 onPartitionAssigned() 메서드와 onPartitionRevoked() 메서드로 이루어져 있다.

- onPartitionAssgined() : 리밸런스가 끝난 뒤에 파티션이 할당 완료되면 호출되는 메서드이다.
- onPartitionRevoked() : 리밸런스가 시작되기 직전에 호출되는 메서드이다. 마짐가으로 처리한 레코드를 기준으로 커밋을 하기 위해서는 
    
    리밸런스가 시작하기 직전에 커밋을 하면 되므로 onPartitionRevoked() 메서드에 커밋을 구현하여 처리할 수 있다.


----

### 파티션 할당 컨슈머

```java
private final static int PARTITION_NUMBER = 0;
private final static String BOOTSTRAP_SERVERS = "my-kafka:9092";

public static void main(String[] args){
    ...
    KafkaConsumer<String,String> consumer = new KafkaConsumer<>(configs);
    consumer.assign(Collections.singleton(new TopicPartition(TOPIC_NAME,PARTITION_NUMBER)));
    while(true){
        ConsumerRecords<String,String> records = consumer.poll(Duration.ofSeconds(1));
        for(ConsumerRecord<String,String> record : records){
            logger.info("record:{}" , record);
        }
    }
}

```

---

### 컨슈머의 안전한 종료

컨슈머 애플리케이션은 안전하게 종료되어야 한다. 정상적으로 종료되지 않은 컨슈머는 세션 타임아웃이 발생할때까지 컨슈머 그룹에 남게 된다.

컨슈머를 안전하게 종료하기 위해 KafkaConsumer 클래스는 wakeup() 메서드를 지원한다. wakeup()메서드를 실행하여 KafkaConsumer 인스턴스를 안전하게 종료할 수 있다.

wakeup()메서드가 실행된 이후 poll()메서드가 호출되면 WakeupException 예외가 발생한다. Wakeup Exception 예외를 받은 뒤에는 데이터 처리를 위해 사용한 자원들을 해제하면 된다.

```java
static class ShutdownThread extends Thread{
    public void run(){
        logger.info("Shutdown hook");
        consumer.wakeup();
    }
}
```

---

## 멀티스레드 컨슈머

![image](https://user-images.githubusercontent.com/40031858/172381410-c9a2a8d0-6013-4e06-8d39-b1eb3351f70d.png)

카프카는 처리량을 늘리기 위해 파티션과 컨슈머 개수를 늘려서 운영할 수 있다. 파티션을 여러 개로 운영하는 경우 데이터를 병렬처리하기 위해서 파티션 개수와 컨슈머 개수를

동일하게 맞추는 것이 가장 좋은 방법이다. 토픽의 파티션은 1개 이상으로 이루어져 있으며 1개의 파티션은 1개 컨슈머가 할당되어 데이터를 처리할 수 있다.

파티션 개수가 n개라면 동일 컨슈머 그룹으로 묶인 컨슈머 스레드를 최대 n개 운영할 수 있다. 그러므로 n개의 스레드를 가진 1개의 프로세스를 운영하거나 1개의 스레드를 가진 프로세스를 n개 운영하는 방법도 있다.

---

## 컨슈머 랙

![image](https://user-images.githubusercontent.com/40031858/172381961-a9684eaf-3679-4011-b401-b720a20f0bea.png)

컨슈머 랙(LAG)은 파티션의 최신 오프셋(LOG-END-OFFSET)과 컨슈머 오프셋(CURRENT-OFFSET)간의 차이다.

프로듀서는 계속해서 새로운 데이터를 파티션에 저장하고 컨슈머는 자신이 처리할 수 있는 만큼 데이터를 가져간다.

컨슈머 랙은 컨슈머가 정상 동작하는지 여부를 확인할 수 있기 때문에 컨슈머 애플리케이션을 운영한다면 필수적으로 모니터링해야 하는 지표다.

![image](https://user-images.githubusercontent.com/40031858/172382160-49eae361-fbae-4da1-8887-75d803fbfe88.png)

컨슈머 랙은 컨슈머 그룹과 토픽, 파티션별로 생성된다. 1개의 토픽에 3개의 파티션이 있고 1개의 컨슈머 그룹이 토픽을 구독하여 데이터를 가져가면 컨슈머 랙은 총 3개가 된다.

### 컨슈머 랙 - 프로듀서와 컨슈머의 데이터 처리량

![image](https://user-images.githubusercontent.com/40031858/172382314-7015f06d-5747-4a48-883e-ea4a6f657bf9.png)

프로듀서가 보내는 데이터양이 컨슈머의 데이터 처리량보다 크다면 컨슈머 랙은 늘어난다. 반대로 프로듀서가 보내는 데이터양이 컨슈머의 

데이터 처리량보다 적으면 컨슈머 랙은 줄어들고 최솟값은 0으로 지연이 없음을 뜻한다.

### 컨슈머 랙 모니터링
컨슈머 랙을 모니터링하는 것은 카프카를 통한 데이터 파이프라인을 운영하는데에 핵심적인 역할을 한다. 컨슈머 랙을 모니터링함으로써 컨슈머의 장애를 확인할 수 있고 파티션 개수를 정하는 데에 참고할 수 있기때문

### 컨슈머 랙 모니터링 - 처리량 이슈

![image](https://user-images.githubusercontent.com/40031858/172382739-33ac99da-aac4-4e85-b275-c67b7acfc406.png)

프로듀서의 데이터양이 늘어날 경우에는 컨슈머 랙이 늘어날 수 있다. 이경우 파티션 개수와 컨슈머 개수를 늘려 병렬 처리량을 늘려 컨슈머 랙을 줄일 수 있다. 컨슈머의 개수를 2개로 늘림으로써 컨슈머의 데이터 처리량을 2배로 늘린다.

### 컨슈머 랙 모니터링 - 파티션 이슈

![image](https://user-images.githubusercontent.com/40031858/172382980-0e7a5340-20c7-4bb7-bf70-8f4defd076e1.png)

프로듀서의 데이터양이 일정함에도 불구하고 컨슈머의 장애로 인해 컨슈머 랙이 증가할 수도 있다. 컨슈머는 파티션 개수만큼 늘려서 병렬처리하며 파티션마다 컨슈머가 할당되어 데이터를 처리한다.

예를 들어 2개의 파티션으로 구성된 토픽에 2개의 컨슈머가 각각 할당되어 데이터를 처리한다고 가정해보자. 프로듀서가 보내는 데이터 양은 

동일한데 파티션 1번의 컨슈머 랙이 늘어나는 상황이 발생한다면 1번 파티션에 할당된 컨슈머에 이슈가 발생했음을 유추할 수 있다.

---

## 컨슈머 랙을 모니터링하는 방법 세가지

### 1. 카프카 명령어 사용

kafka-consumer-groups.sh 명령어를 사용하면 컨슈머 랙을 포함한 특정 컨슈머 그룹의 상태를 확인할 수 있다. 컨슈머랙을 확인하기 위해 가장 기초적인 방법으로 다음과 같이 명령어를 사용하면 된다.

카프카 명령어를 통해 컨슈머랙을 확인하는 방법은 일회성에 그치고 지표를 지속적으로 기록하고 모니터링하기에는 부족하다.

그렇기 때문에 kafka-consumer-groups.sh 를 통해 컨슈머 랙을 확인하는 것은 테스트용 카프카에서 사용한다.

![image](https://user-images.githubusercontent.com/40031858/172384519-113b515c-00c1-4f69-9a10-6b4322f72bdf.png)

### 2. metrics() 메서드 사용

컨슈머 애플리케이션에서 KafkaConsumer 인스턴스의 metrics() 메서드를 활용하면 컨슈머 랙 지표를 확인할 수 있다. 컨슈머 인스턴스가 제공하는 컨슈머 랙 관련 모니터링 지표는

3가지로 records-lag-max, records-lag, records-lag-avg이다.

```java
for(Map.Entry<MetricName,? extends Metric> entry: KafkaConsumer.metrics().entrySet() ){
    if("records-lag-max".equals(entry.getKey().name()) ||
            "records-lag".equals(entry.getKey().name()) ||
            "records-lag-avg".equals(entry.getKey().name())){
                Metric metric = entry.getValue();
                logger.info("{} : {}",entry.getKey().name(), metric.metricValue());
            }
}
```

#### metrics() 메서드 사용 이슈
- 컨슈머가 정상 동작할 경우에만 확인할 수 있다. metrics()메서드는 컨슈머가 정상적으로 실행될 경우에만 호출된다. 그렇기 때문에 만약 컨슈머 애플리케이션이
    
    비정상적으로 종료되면 더는 컨슈머 랙을 모니터링할 수 없다.
- 모든 컨슈머 애플리케이션에 컨슈머 랙 모니터링 코드를 중복해서 작성해야 한다. 컨슈머 애플리케이션을 여러 종류로 운영할 경우 각기 다른 컨슈머 애플리케이션에

    metrics()메서드를 호출하여 컨슈머 랙을 수집하는 로직을 중복해서 넣어야 한다. 왜냐하면 특정 컨슈머 그룹에 해당하는 애플리케이션이 수집하는 컨슈머 랙은 자기 자신 컨슈머 그룹에 대한 컨슈머 랙만 한정되기 때문
- 컨슈머 랙을 모니터링하는 코드를 추가할 수 없는 카프카 서드파티(third party)애플리케이션의 컨슈머 랙 모니터링이 불가능하다.

### 3. 외부 모니터링 툴 사용
컨슈머 랙을 모니터링하는 가장 최선의 방법은 외부 모니터링 툴을 사용하는 것이다. 데이터 독(Datadog), 컨플루언트 컨트롤 센터(Confluent Control Center)와 같은

카프카 클러스터 종합 모니터링 툴을 사용하면 카프카 운영에 필요한 다양한 지표를 모니터링할 수 있다.

모니터링 지표에는 컨슈머 랙도 포함되어 있기 때문에 클러스터 모니터링과 컨슈머 랙을 함께 모니터링하기에 적합하다. 컨슈머 랙 모니터링만을 위한 오픈소스로 버로우(Burrow)도 있다.


---

## section06 정리
- 컨슈머 랙은 파티션의 최신 오프셋과 컨슈머 오프셋간의 차이다.
- 컨슈머 랙은 컨슈머 애플리케이션을 운영할 때 반드시 모니터링 해야 한다
- 컨슈머 랙은 파티션 별로 컨슈머 그룹별로 생성된다
- 컨슈머 처리량보다 프로듀서의 전송량이 많을 경우 컨슈머 랙이 반드시 발생한다
- 카프카 명령어 또는 metrics()메서드를 사용하면 컨슈머 랙을 확인할 수 있다.
- 컨슈머 랙을 확인하는 방법 중 하나는 외부 모니터링 툴인 버로우(burrow)를 확인하는 것이다
- 카프카 버로우는 시간 위도우를 통해 컨슈머 랙을 평가한다