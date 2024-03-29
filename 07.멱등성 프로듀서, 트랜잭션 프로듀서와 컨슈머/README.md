# 멱득성 프로듀서, 트랜잭션 프로듀서와 컨슈머

## 멱득성 프로듀서

### 전달 신뢰성

멱등성이란 여러 번 연산을 수행하더라도 동일한 결괄르 나타내는 것을 뜻한다. 이러한 의미에서 멱등성 프로듀서는 동일한 데이터를 여러번 전송하더라도 카프카 클러스터에 단 한번만 저장됨을 의미한다.

기본 프로듀서의 동작 방식은 적어도 한번 전달을 지원한다. 적어도 한번 전달이란 프로듀서가 클러스터에 데이터를 전송하여 저장 할때 적어도 한번 이상 데이터를 적재할 수 있고 데이터가 유실되지 않음을 뜻한다.

다만, 두 번 이상 적재할 가능성이 있으므로 데이터의 중복이 발생할 수 있다.
- At least onece : 적어도 한번 이상 전달
- At most once: 최대 한번 전달
- Exactly once: 정확히 한번 전달

프로듀서가 보내는 데이터 중복 적재를 막기 위해 0.11.0 이후 버전부터는 프로듀서에서 enable.idempotence옵션을 사용하여 정확히 한번 전달을 지원한다.

enable.idempotence옵션의 기본값은 false이며 정확히 한번 전달을 위해서는 true로 옵션값을 설정해서 멱등성 프로듀서로 동작하도록 만들면 된다.

```java
Proeprties configs = new Properties();
configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFG, BOOSTRAP_SERVERS);
configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
configs.put(ProducerCOnfig.ENABLE_IDEMPOTENCE_CONFIG, true);

KafkaProducer<String,String> producer = new KafkaProducer<>(configs);
```

### 멱등성 프로듀서의 동작

멱등성 프로듀서는 기본 프로듀서와 달리 데이터를 브로커로 전달할때 프로듀서 PID와 시퀀스 넘버를 함께 전달한다. 그러면 브로커는 프로듀서의 PID와 시퀀스 넘버를 확인하여

동일한 메시지의 적재 요청이 오더라도 단 한 번만 데이터를 적재함으로써 프로듀서의 데이터는 정확히 한번 브로커에 적재되도록 동작한다.

- PID(Producer unique ID) : 프로듀서의 고유한 ID
- SID(Sequence ID) : 레코드의 전달 번호 ID

### 멱등성 프로듀서가 아닌 경우

![image](https://user-images.githubusercontent.com/40031858/172809709-f0fa60f9-340b-4e91-bca3-189a797843ca.png)

### 멱등성 프로듀서인 경우

![image](https://user-images.githubusercontent.com/40031858/172809792-9a15affd-cded-4a87-b191-48c2988bb821.png)

### 멱등성 프로듀서로 설정할 경우 옵션

멱등성 프로듀서를 사용하기 위해 enable.idempotence를 true로 설정하면 정확히 한번 적재하는 로직이 성립되기 위해 프로듀서의 일부 옵션들이 강제로 설정된다.

프로듀서의 데이터 재전송 횟수를 정하는 retries는 기본값으로 Integer.MAX_VALUE로 설정되고 acks옵션은 all로 설정된다. 이렇게 설정되는 이유는 프로듀서가 적어도 한 번 이상 브로커에

데이터를 보냄으로써 브로커에 단 한번만 데이터가 적재되는 것을 보장하기 위해서다. 멱등성 프로듀서는 정확히 한번 브로커에 데이터를 적재하기 위해 정말로 한번 전송하는 것이 아니다.

상황에 따라 프로듀서가 여러번 전송하되 브로커가 여러번 전송된 데이터를 확인하고 중복된 데이터는 적재하지 않는 것이다.

### 멱등성 프로듀서 사용시 오류 확인

![image](https://user-images.githubusercontent.com/40031858/172810259-daec0f37-f9a3-418e-b19f-4efee7d84dd4.png)

멱등성 프로듀서의 시퀀스 넘버는 0부터 시작하여 숫자를 1씩 더한 값이 전달된다. 브로커에서 멱등성 프로듀서가 전송한 데이터의 PID와 시퀀스 넘버를 확인하는 과정에서 시퀀스 넘버가

일정하지 않은 경우에는 OutofOrderSequenceException이 발생할 수 있다. 이 오류는 브로커가 예상한 시퀀스 넘버와 다른번호의 데이터의 적재 요청이 왔을 때 발생한다.

OutofOrderSequenceException이 발생했을 경우에는 시퀀스 넘버의 역전현상이 발생할 수 있기 때문에 순서가 중요한 데이터를 전송하는 프로듀서는 해당 Exception이 발생했을경우 대응하는 방안을 고려해야 함

## 트랜잭션 프로듀서, 컨슈머

### 트랜잭션 프로듀서의 동작

![image](https://user-images.githubusercontent.com/40031858/172845208-43da486b-9034-445d-9139-15dcc7c70b92.png)

카프카에서 트랜잭션은 다수의 파티션에 데이터를 저장할 경우 모든 데이터에 대해 동일한 원자성을 만족시키기 위해 사용된다. 원자성을 만족시킨다는 의미는 다수의 

데이터를 동일 트랜잭션으로 묶음으로써 전체 데이터를 처리하거나 전체 데이터를 처리하지 않도록 하는것을 의미한다. 트랜잭션 프로듀서는 사용자가 보낸 데이터를 레코드로

파티션에 저장할 뿐만 아니라 트랜잭션의 시작과 끝을 표현하기 위해 트랜잭션 레코드를 한 개 더 보낸다.

### 트랜잭션 컨슈머의 동작

![image](https://user-images.githubusercontent.com/40031858/172845517-6356c4d2-8ea0-4224-a1b5-abcde597c433.png)

트랜잭션 컨슈머는 파티션에 저장된 트랜잭션 레코드를 보고 트랜잭션이 완료되었음을 확인하고 데이터를 가져간다. 트랜잭션 레코드는 실질적인 데이터는

가지고 있지 않으며 트랜잭션이 끝난 상태를 표시하는 정보만 가지고 있다.

### 트랜잭션 프로듀서 설정

트랜잭션 프로듀서로 동작하기 위해 transactional.id를 설정해야 한다. 프로듀서별로 고유한 ID 값을 사용해야 한다. init, begin, commit순서대로 수행되어야 한다

```java
configs.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, UUID.randomUUID());

Producer<String,String> producer = new KafkaProducer<>(configs);

producer.initTransactions();

producer.beginTransaction();
producer.send(new ProducerRecord<>(TOPIC,"전달하는 메시지 값"));
producer.commitTransaciton();

producer.close();
```

### 트랜잭션 컨슈머 설정

트랜잭션 컨슈머는 커밋이 완료된 레코드들만 읽기 위해 isolation.level옵션을 read_committed로 설정해야한다.

기본 값은 read_uncommitted로서 트랜잭션 프로듀서가 레코드를 보낸 후 커밋여부와 상관없이 모두 읽는다. read_committed로 설정한 컨슈머는 커밋이 완료된 레코드들만 읽어 처리한다

```java
configs.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");
KafkaConsumer<String,String> consumer = new KafkaConsumer<>(configs);
```