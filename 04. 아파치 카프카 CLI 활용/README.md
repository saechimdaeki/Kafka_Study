# 아파치 카프카 CLI 활용

## [카프카 바이너리 파일 다운로드](https://kafka.apache.org/downloads)

## 카프카 커맨드 라인 툴
카프카에서 제공하는 카프카 커맨드 라인 툴들은 카프카를 운영할 때 가장 많이 접하는 도구다. 커맨드 라인 툴을 통해 카프카 브로커 운영에 필요한

다양한 명령을 내릴 수 있다. 카프카 클라이언트 애플리케이션을 운영할 떄는 카프카 클러스터와 연동하여 데이터를 주고받는 것도 중요하지만

토픽이나 파티션 개수 변경과 같은 명령을 실행해야 하는 경우도 자주 발생한다.

커맨드 라인 툴을 통해 토픽 관련 명령을 실행할 때 필수 옵션과 선택 옵션이 있다. 선택 옵션은 지정하지 않을시 브로커에 설정된 기본 설정값 또는 커맨드 라인 툴의 기본값으로 대체되어 설정된다.

 그러므로 커맨드 라인 툴을 사용하기 전에 현재 브로커에 옵션이 어떻게 설정되어 있는지 확인한 후에 사용하면 실수할 확룔이 줄어든다.
 
### server.properties를 수정하자..! (아래는 `김준성` 개인 설정)
```properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://localhost:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
advertised.listeners=PLAINTEXT://localhost:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/Users/a1101783/Desktop/kafka_2.12-2.5.0/data/

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=3

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=18000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0

```

 ### 주키퍼 실행
 ```console
$ bin/zookeeper-server-start.sh config/zookeeper.properties
 ```

 ### 카프카 브로커 실행
 ```console
$ bin/kafka-server-start.sh config/server.properties
 ```

 ### 카프카 정상 여부 확인
 ```console
$ bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9092
 ```

 <img width="855" alt="image" src="https://user-images.githubusercontent.com/40031858/170609929-67681230-575d-406d-88c0-f85ee4967081.png">

```console
$ bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

### 테스트 편의를 위한 hosts 설정

![image](https://user-images.githubusercontent.com/40031858/170610167-3a48cf20-dcd9-4f17-aba5-511efb8a6785.png)

### kafka-topics.sh

```console
a1101783@1101783M01 kafka_2.12-2.5.0 % bin/kafka-topics.sh --create --bootstrap-server my-kafka:9092 --topic hello.kafka
```

<img width="880" alt="image" src="https://user-images.githubusercontent.com/40031858/170709018-4c1217fd-75d1-4cfc-84df-e2494b38caab.png">

hello.kafka 토픽처럼 카프카 클러스터 정보와 토픽 이름만으로 토픽을 생성할 수 있다. 클러스터 정보와 토픽 이름은 토픽을 만들기 위한 필수 값이다.

이렇게 만들어진 토픽은 파티션 개수, 복제 개수 등과 같이 다양한 옵션이 포함되어 있지만 모두 브로커에 설정된 기본값으로 생성되었다.


파티션 개수, 복제개수, 토픽 데이터 유지 기간 옵션들을 지정하여 토픽을 생성하고 싶다면 다음과 같이 명령을 실행하면 된다. 생성된 토픽들의 이름을 조회하려면 --list옵션을 사용한다

```console
a1101783@1101783M01 kafka_2.12-2.5.0 % bin/kafka-topics.sh --create --bootstrap-server my-kafka:9092 --partitions 10 --replication-factor 1 --topic hello.kafka2 --config retention.ms=172800000
```

<img width="1370" alt="image" src="https://user-images.githubusercontent.com/40031858/170709846-2d65839a-f585-4441-8d83-0ebd48c44953.png">

파티션 개수를 늘리기 위해서 --alter 옵션을 사용하면 된다.

<img width="1025" alt="image" src="https://user-images.githubusercontent.com/40031858/170710684-89532b86-6ff2-4900-9f82-a404956b6688.png">

파티션 개수를 늘릴 수 있지만 줄일 수는 없다. 다시 줄이는 명령을 내리면 InvalidPartitionsException이 발생한다.

분산 시스템에서 이미 분산된 데이터를 줄이는 방법은 매우 복잡하다. 삭제 대상 파티션을 지정해야할 뿐만 아니라 기존에 저장되어 있던

레코드를 분산하여 저장하는 로직이 필요하기 때문이다.

이때문에 카프카에서는 파티션을 줄이는 로직은 제공하지 않는다. 만약 피치못할 사정으로 파티션 개수를 줄여야 할때는 토픽을 새로만드는 편이 좋다.

<img width="1467" alt="image" src="https://user-images.githubusercontent.com/40031858/170710984-2c1789e7-53ec-4c77-bd2f-3f6ed8994475.png">

## kafka-configs.sh

토픽의 일부 옵션을 설정하기 위해서는 kafka-configs.sh 명령어를 사용해야 한다. --alter과 --add-config옵션을 사용해서 min.insync.replicas 옵션을토픽별로 설정할 수 있다.

<img width="1098" alt="image" src="https://user-images.githubusercontent.com/40031858/170721226-eab9d250-a7ce-4234-9198-3b0e9b30024c.png">

브로커에 설정된 각종 기본값은 --borker, --all, --describe 옵션을 사용하여 조회할 수 있다.

<img width="1712" alt="image" src="https://user-images.githubusercontent.com/40031858/170721481-9884c1d0-aff2-4e49-b400-277c26ed53ba.png">

## kafka-console-producer.sh

hello.kafka 토픽에 데이터를 넣을 수 있는 kafka-console-producer.sh 명령어를 실행해 보자. 키보드로 문자를 작성하고 엔터키를 누르면 별 다른 응답없이 메시지 값이 전송된다.

메시지 키를 가지는 레코드를 전송해 보자. 메시지 키를 가지는 레코드를 전송하기 위해서는 몇가지 추가 옵션을 작성해야 한다.

`key.separator`를 선언하지 않으면 기본 설정은 Tab delimiter(\t)이므로 key.separator를 선언하지 않고

메시지를 보내려면 메시지 키를 작성하고 탭키를 누른뒤 메시지 값을 작성하고 엔터를 누른다. 여기선 명시적으로 확인하기 위해 콜론을 구분자로 선언했다.

<img width="1320" alt="image" src="https://user-images.githubusercontent.com/40031858/170818805-fc7c1e33-e1c1-4008-b142-affaca6fed75.png">


### 메시지 키와 메시지 값이 포함된 레코드가 파티션에 전송됨.

![image](https://user-images.githubusercontent.com/40031858/170818841-89dcd3a4-a68a-42d0-a0fc-09a9fb989ced.png)

메시지 키와 메시지 값을 함께 전송한 레코드는 토픽의 파티션에 저장된다. 메시지 키가 null인 경우에는 프로듀서가 파티션으로 전송할 때 

레코드 배치 단위(레코드 전송 묶음)로 라운드 로빈으로 전송한다. 메시지 키가 존재하는 경우에는 키의 해시값을 작성하여

존재하는 파티션 중 한개에 할당된다. 이로인해 메시지키가 동일한 경우에는 동일한 파티션으로 전송된다.

## kafka-console-consumer.sh

hello.kafka 토픽으로 전송한 데이터는 kafka-console-consumer.sh 명령어로 확인할 수 있다. 이때 필수 옵션으로 --bootstrap-server

에 카프카 클러스터 정보, --topic에 토픽 이름이 필요하다. 추가로 from-beginning 옵션을 주면 토픽에 저장된 가장 처음 데이터부터 출력한다.

```console
bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning
```

<img width="1711" alt="image" src="https://user-images.githubusercontent.com/40031858/170819156-1a47d289-2de0-4134-8e4b-563dd4e380fa.png">

kafka-console-producer.sh로 보낸 메시지 값이 출력된 것을 확인할 수 있다. 만약 레크드의 메시지 키와 메시지 값을 확인하고 싶다면 --property 옵션을 사용하면 된다.

```console
bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --property print.key=true --property key.separator="-" --from-beginning
```

<img width="937" alt="image" src="https://user-images.githubusercontent.com/40031858/170819285-2adb9be8-14de-44d8-8bd0-8c642b66c948.png">

--max-messages 옵션을 사용하면 최대 컨슘 메시지 개수를 설정할 수 있다.

```console
bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning --max-messages 1
```

<img width="1116" alt="image" src="https://user-images.githubusercontent.com/40031858/170819366-553b999a-0a3a-4258-af57-761126d31e6f.png">


-- partition 옵션을 사용하면 특정 파티션만 컨슘할 수 있다.

```console
 bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --partition 2 --from-beginning
```

<img width="1140" alt="image" src="https://user-images.githubusercontent.com/40031858/170819412-f2a84dfd-ee46-4bf1-a352-b8460a75d62c.png">

--group 옵션을 사용하면 컨슈머 그룹을 기반으로 kafka-console-consumer가 동작한다. 

`컨슈머 그룹`이란 특정 목적을 가진 컨슈머들을 묶음으로 사용하는 것을 뜻한다. 컨슈머 그룹으로 토픽의 레코드를 가져갈 경우 어느 레코드까지 읽었는지에 대한 데이터가 브로커에 저장된다.

```console
bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --group hello-group --from-beginning
```

<img width="1156" alt="image" src="https://user-images.githubusercontent.com/40031858/170819503-0ef496fa-dfd9-4001-9850-dccd26a2fa4d.png">

## kafka-consumer-groups.sh

hello-group이름의 컨슈머 그룹으로 생성된 컨슈머로 hello.kafka토픽의 데이터를 가져갔다. 컨슈머 그룹은 따로 생성하는

명령을 날리지 않고 컨슈머를 동작할 때 컨슈머 그룹 이름을 지정하면 새로 생성된다. 생성된 컨슈머 리스트는 kafka-consumer-groups.sh 명령어로 확인할 수 있다.

```console
bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 --list
```

<img width="787" alt="image" src="https://user-images.githubusercontent.com/40031858/170819955-c5494bef-6463-4525-983e-83c595d06abb.png">


--dscribe 옵션을 사용하면 해당 컨슈머 그룹이 어떤 토픽을 대상으로 레코드를 가져갔는지 상태를 확인 할 수 있다.

파티션 번호, 현재까지 가져간 레코드의 오프셋, 파티션 마지막 레코드의 오프셋, 컨슈머 랙, 컨슈머 ID, 호스트를 알 수 있기 때문에 컨슈머의 상태를 조회할 때 유용하다

```console
bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 --group hello-group --describe
```

<img width="983" alt="image" src="https://user-images.githubusercontent.com/40031858/170820023-1e6dfe44-c3ea-4449-bed7-8c2f6d5c7971.png">

### kafka-consumer-groups.sh 오프셋 리셋

```console
bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 --group hello-group --topic hello.kafka --reset-offsets --to-earliest --execute
```

<img width="1297" alt="image" src="https://user-images.githubusercontent.com/40031858/170820179-5c7c7a1d-21dc-408f-86f4-1811c0640c16.png">

### kafka-consumer-groups.sh 오프셋 리셋 종류

- `--to-earliest` : 가장 처음 오프셋(작은번호)으로 리셋
- `--to-latest` : 가장 마지막 오프셋(큰번호)으로 리셋
- `--to-current` : 현 시점 기준 오프셋으로 리셋
- `--to-datetime {YYYY-MM-DDTHH:mmSS.sss}` : 특정 일시로 오프셋 리셋(레코드 타임스탬프 기준)
- `--to-offset{long}` : 특정 오프셋으로 리셋
- `--shift-by {+/- long}` : 현재 컨슈머 오프셋에서 앞뒤로 옮겨서 리셋


---

# 그 외 커맨드 라인 툴

## kafka-producer-perf-test.sh

kafka-producer-perf-test.sh 는 카프카 프로듀서로 퍼포먼스를 측정할 때 사용된다.

```console
bin/kafka-producer-perf-test.sh --producer-props bootstrap.servers=my-kafka:9092 --topic hello.kafka --num-records 10 --throughput 1 --record-size 100 --print-metric
```

<img width="1546" alt="image" src="https://user-images.githubusercontent.com/40031858/170820648-f16e390f-dd73-4f10-9213-312c2884fa7f.png">

### kafka-reassign-partitions.sh

![image](https://user-images.githubusercontent.com/40031858/170820714-1fa0a626-9647-457c-a941-e0013e5a60ba.png)

kafka-reassign-partitions.sh를 사용하면 리더 파티션과 팔로워 파티션이 위치를 변경할 수 있다. 카프카 브로커에는 

auto.leader.rebalance.enable 옵션이 있는데 이 옵션의 기본값은 true로써 클러스터 단위에서 리더 파티션을

자동 리밸런싱하도록 도와준다. 브로커의 백그라운드 스레드가 일정한 간격으로 리더의 위치를 파악하고 필요시 리더 리밸런싱을 통해 리더의 위치가 알맞게 배분된다.

```console
$ cat partitions.json
{
 "partitions":
 [ { "topic": "hello.kafka", "partition": 0, "replicas": [ 0 ] } ]
 ,"version": 1
}
$ bin/kafka-reassign-partitions.sh --zookeeper my-kafka:2181 \
 --reassignment-json-file partitions.json --execute
```

### kafka-delete-record.sh

```console
$ cat delete.json
{
 "partitions": [
 {
 "topic": "hello.kafka", "partition": 0, "offset": 5
 }
 ], "version": 1
}
$ bin/kafka-delete-records.sh --bootstrap-server my-kafka:9092 \
 --offset-json-file delete.json
Executing records delete operation
Records delete operation completed:
partition: hello.kafka-0 low_watermark: 5
```

### kafka-dump-log.sh

```console
$ ls data/hello.kafka-0
00000000000000000000.index 00000000000000000000.timeindex
00000000000000000000.log leader-epoch-checkpoint
$ bin/kafka-dump-log.sh \
 --files data/hello.kafka-0/00000000000000000000.log \
 --deep-iteration
Dumping data/hello.kafka-0/00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 2 count: 3 baseSequence: -1 lastSequence: -1
producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false
isControl: false position: 0 CreateTime: 1642337213446 size: 87 magic: 2
compresscodec: NONE crc: 23690631 isvalid: true
...
```

---

# 토픽을 생성하는 두가지 방법
토픽을 생성하는 상황은 크게 2가지가 있다. `첫 번째는 카프카 컨슈머 또는 프로듀서가 카프카 브로커에 생성되지 않은 토픽에 대해 데이터를 요청할 때`

그리고 `두 번째는 커맨드 라인 툴로 명시적으로 토픽을 생성하는 것이다.` 토픽을 효과적으로 유지보수하기 위해서는 토픽을 명시적으로 생성하는 것을 추천한다.

토픽마다 처리되어야 하는 데이터의 특성이 다르기 때문이다. 토픽을 생성할 때는 데이터 특성에 따라 옵션을 다르게 설정할 수 있다. 예를 들어, 동시 데이터 처리량이 많아야 하는 토픽의 경우 파티션의 개수를 100으로 설정할 수 있다.

단기간 데이터 처리만 필요한 경우에는 토픽에 들어온 데이터의 보관기간 옵션을 짧게 설정할 수도 있다.

