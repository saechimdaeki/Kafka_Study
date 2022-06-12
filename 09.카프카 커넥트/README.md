# 카프카 커넥트

## 카프카 커넥트소개

![image](https://user-images.githubusercontent.com/40031858/173169941-b2bcd099-9e15-43e8-8af3-b26ad35874c2.png)


카프카 커넥트는 카프카 오픈소스에 포함된 툴 중 하나로 데이터 파이프라인 생성 시 반복 작업을 줄이고 효율적인 전송을 이루기 위한 애플리케이션이다.

커넥트는 특정한 작업 형태를 템플릿으로 만들어놓은 커넥터를 실행함으로써 반복 작업을 줄일 수 있다.

### 커넥트 내부 구조

![image](https://user-images.githubusercontent.com/40031858/173169977-1b86c410-9281-4c82-91de-38ab16e14e65.png)

사용자가 커넥트에 커넥터 생성 명령을 내리면 커넥트는 내부에 커넥터와 태스크를 생성한다. 커넥터는 태스크들을 관리한다.

태스크는 커넥터에 종속되는 개념으로 실질적인 데이터 처리를 한다. 그렇기 때문에 데이터 처리를 정상적으로 하는지 확인하기 위해서는 각 태스크의 상태를 확인해야함.

## 커넥터

### 소스 커넥터, 싱크 커넥터

커넥터는 프로듀서 역할을 하는 `소스 커넥터`와 컨슈머 역할을 하는 `싱크 커넥터` 2가지로 나뉜다. 예를 들어 파일을 주고받는 용도로 파일 소스 커넥터와 파일 싱크 커넥터가

있다고 가정하자. 파일 소스 커넥터는 파일의 데이터를 카프카 토픽으로 전송하는 프로듀서 역할을 한다. 그리고 파일 싱크 커넥터는 토픽의 데이터를 파일로 저장하는 컨슈머 역할을 한다. 

파일 외에도 일정한 프로토콜을 가진 소스 애플리케이션이나 싱크 애플리케이션이 있다면 커넥터를 통해 카프카로 데이터를 보내거나 카프카에서 데이터를 가져올 수 있다.

MySQL, S3, MongoDB 등과 같은 저장소를 대표적인 싱크 애플리케이션, 소스 애플리케이션이라 볼 수 있다. 즉 , MySQL에서 카프카로 데이터를 보낼 때 그리고 카프카에서 데이터를

MySQL로 저장할 때 JDBC 커넥터를 사용하여 파이프라인을 생성할 수 있다.

### 커넥터 플러그인

![image](https://user-images.githubusercontent.com/40031858/173170093-4e6ccda8-d05b-4c40-8018-1ca1b82a6a48.png)

카프카 2.6에 포함된 커넥트를 실행할 경우 클러스터간 토픽 미러링을 지원하는 미러메이커2 커넥터와 파일 싱크 커넥터, 파일 소스 커넥터를 기본 플러그인으로 제공한다. 이외에 추가적인

커넥터를 사용하고 싶다면 플러그인 형태로 커넥터 jar파일을 추가하여 사용할 수 있다. 커넥터 jar파일에는 커넥터를 구현하는 클래스를 빌드한 클래스 파일이 포함되어 있다.

커넥터 플러그인을 추가하고 싶다면 직접 커넥터 플러그인을 만들거나 이미 인터넷상에 존재하는 커넥터 플러그인을 가져다 쓸 수도 있다.

### 컨버터, 트랜스폼

사용자가 커넥터를 사용하여 파이프라인을 생성할 때 `컨버터`와 `트랜스폼`기능을 옵션으로 추가할 수 있다. 커넥터를 운영할 때 반드시 필요한 설정은 아니지만 데이터 처리를 더욱 풍부하게

도와주는 역할을 한다. 컨버터는 데이터 처릴르 하기 전에 스키마를 변경하도록 도와준다. 

JsonConverter, StringConverter, ByteArrayConverter를 지원하고 필요하다면 커스텀 컨버터를 작성하여 사용할 수도 있다.

트랜스폼은 데이터 처리 시 각 메시지 단위로 데이터를 간단하게 변환하기 위한 용도로 사용된다.

예를 들어 JSON 데이터를 커넥터에서 사용할 때 트랜스폼을 사용하면 특정 키를 삭제하거나 추가할 수 있다. 기본 제공 트랜스폼으로 Cast,Drop,ExtractField등이 있다.


---

## 커넥트 배포 및 운영

### 커넥트를 실행하는 방법

커넥트를 실행하는 방법은 크게 두가지가 있다. 첫 번째는 `단일 모드 커넥트`이고 두 번째는 `분산 모드 커넥트`이다. 단일 모드 커넥트는 단일 애플리케이션으로 실행된다.

커넥터를 정의하는 파일을 작성하고 해당 파일을 참조하는 단일 모드 커넥트를 실행함으로써 파이프라인을 생성할 수 있다.

### 단일 모드 커넥트

![image](https://user-images.githubusercontent.com/40031858/173177019-7e5c7aa8-b778-4ecc-b687-e5b712734efa.png)

단일 모드 커넥트는 1개 프로세스만 실행되는 점이 특징인데, 단일 프로세스로 실행되기 때문에 고가용성 구성이 되지 않아서 단일 장애점이 될 수 있다.

그러므로 단일 모드 커넥트 파이프라인은 주로 개발환경이나 중요도가 낮은 파이프라인을 운영할 때 사용한다.

### 분산 모드 커넥트

![image](https://user-images.githubusercontent.com/40031858/173177041-3a26456b-417d-460e-afee-2a083725c05d.png)

분산 모드 커넥트는 2대 이상의 서버에서 클러스터 형태로 운영함으로써 단일 모드 커넥트 대비 안전하게 운영할 수 있다는 장점이있다.

2개 이상의 커넥트가 클러스터로 묶이면 1개의 커넥트가 이슈 발생으로 중단되더라도 남은 1개의 커넥트가 파이프라인을 지속적으로 처리할 수 있기 때문이다.

분산 모드 커넥트는 데이터 처리량의 변화에도 유연하게 대응할 수 있다. 커넥트가 실행되는 서버 개수를 늘림으로써 무중단으로 스케일 아웃하여

처리량을 늘릴 수 있기 때문이다. 이러한 장점이 있기 때문에 상용환경에서 커넥트를 운영한다면 분산 모드 커넥트를 2대 이상으로 구성하고 설정하는 것이 좋다.

---

## 단일모드 커넥트

### 단일 모드 커넥트 설정

단일 모드 커넥트를 실행하기 위해서는 단일 모드 커넥트를 참조하는 설정 파일인 connect-standalone.properties파일을 수정해야 한다.

```properties
bootstrap.servers=my-kafka:9092
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false
offset.storage.file.filename=/tmp/connect.offsets
offset.flush.interval.ms=10000
plugin.path=/usr/local/share/java,/usr/local/share/kafka/plugins
```

### 단일 모드 커넥트의 실행

단일 모드 커넥트를 실행시에 파라미터로 커ㅏ넥트 설정 파일을 차례로 넣어 실행하면 된다

```markdown
name=local-file-source
connector.class=FileStreamSource
tasks.max=1
file=/tmp/test.txt
topic=connect-test
```

```console
$ bin/connect-standalone.sh config/connect-standalone.properties \
 config/connect-file-source.properties
```

## 분산모드 커넥트

![image](https://user-images.githubusercontent.com/40031858/173209723-d8bb3d67-07fc-42c2-9765-9170c2e64b96.png)


분산 모드 커넥트는 단일 모드 커넥트와 다르게 2개 이상의 프로세스가 1개의 그룹으로 묶여서 운영된다. 이를 통해 1개의 커넥트 프로세스에 이슈가 발생하여

종료되더라도 살아있는 나머지 1개 커넥트 프로세스가 커넥터를 이어받아서 파이프라인을 지속적으로 실행할 수 있다는 특징이있다.

이제 분산 모드 커넥트를 묶어서 운영하기위해 어떤 설정을 해야하는지 분산 모드 설정 파일인 connect-distributed.properties를보자.

```properties
bootstrap.servers=my-kafka:9092
group.id=connect-cluster
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false
offset.storage.topic=connect-offsets
offset.storage.replication.factor=1
config.storage.topic=connect-configs
config.storage.replication.factor=1
status.storage.topic=connect-status
status.storage.replication.factor=1
offset.flush.interval.ms=10000
plugin.path=/usr/local/share/java,/usr/local/share/kafka/plugins

```

## 커스텀 소스 커넥터

![image](https://user-images.githubusercontent.com/40031858/173209963-d0e8b5b2-b765-4c3a-8b72-18db1e8525d9.png)

소스 커넥터는 소스 애플리케이션 또는 소스 파일로부터 데이털르 가져와 토픽으로 넣는 역할을 한다. 오픈소스 소스 커넥터를 사용해도 되지만

라이선스 문제나 로직이 원하는 요구사항과 맞지 않아서 직접 개발해야 하는 경우도 있는데 이때는 카프카 커넥트 라이브러리에서 제공하는 SourceConnector와

SourceTask 클래스를 사용하여 직접 소스 커넥터를 구현하면 된다. 직접 구현한 소스 커넥터를 빌드하여 jar파일로 만들고 커넥트를 실행 시 플러그인으로 추가하여 사용가능

### SourceConnector , SourceTask

#### `SourceConnector`

태스크를 실행하기 전 커넥터 설정파일을 초기화하고 어떤 태스크 클래스를 사용할 것인지 정의하는 데에 사용한다. 그렇기 때문에

SourceConnector에는 실질적인 데이터를 다루는 부분이 들어가지 않는다. SourceTask가 실제로 데이터를 다루는 클래스라고 볼 수 있다.

#### `SourceTask`

소스 애플리케이션 또는 소스 파일로부터 데이터를 가져와서 토픽으로 데이터를 보내는 역할을 수행한다. SourceTask만의 특징은

토픽에서 사용하는 오프셋이 아닌 자체적으로 사용하는 오프셋을 사용한다는 점이다. SourceTask에서 사용하는 오프셋은 소스 애플리케이션 또는 소스 파일을

어디까지 읽었는지 저장하는 역할을 한다. 이 오프셋을 통해 데이터를 중복해서 토픽으로 보내는 것을 방지할 수 있다.

예를 들어, 파일의 데이터를 한 줄씩 읽어서 토픽으로 데이털르 보낸다면 토픽으로 데이터를 보낸 줄 번호를 오프셋에 저장할 수 있다.

```java
public class SingleFileSourceConnectorConfig extends AbstractConfig {
 public static final String DIR_FILE_NAME = "file";
 private static final String DIR_FILE_NAME_DEFAULT_VALUE = "/tmp/kafka.txt";
 private static final String DIR_FILE_NAME_DOC ="읽을 파일 경로와 이름";
 public static final String TOPIC_NAME = "topic";
 private static final String TOPIC_DEFAULT_VALUE = "test";
 private static final String TOPIC_DOC ="보낼 토픽명";
 
 public static ConfigDef CONFIG = new ConfigDef().define(DIR_FILE_NAME,Type.STRING,
                            DIR_FILE_NAME_DEFAULT_VALUE,
                            Importance.HIGH,DIR_FILE_NAME_DOC)
                            .define(TOPIC_NAME,Type.STRING,
                            TOPIC_DEFAULT_VALUE,
                            Importance.HIGH, TOPIC_DOC);
 
 public SingleFileSourceConnectorConfig(Map<String, String> props) {
    super(CONFIG, props);
 }
}

```

```java
public class SingleFileSourceConnector extends SourceConnector {
 
 private Map<String, String> configProperties;
 
 @Override
 public String version() { return "1.0"; }
 @Override
 public void start(Map<String, String> props) {
    this.configProperties = props;
 try {
    new SingleFileSourceConnectorConfig(props);
 } catch (ConfigException e) {
    throw new ConnectException(e.getMessage(), e);
  }
 }
  @Override
  public Class<? extends Task> taskClass() { 
    return SingleFileSourceTask.class; 
}
 ...
```

```java
public class SingleFileSourceConnector extends SourceConnector {
 ...
 @Override
 public List<Map<String, String>> taskConfigs(int maxTasks) {
 List<Map<String, String>> taskConfigs = new ArrayList<>();
 Map<String, String> taskProps = new HashMap<>();
 taskProps.putAll(configProperties);
 for (int i = 0; i < maxTasks; i++) {
 taskConfigs.add(taskProps);
 }
 return taskConfigs;
 }
 @Override
 public ConfigDef config() { return SingleFileSourceConnectorConfig.CONFIG; }
 @Override
 public void stop() {}
}
```

```java
public class SingleFileSourceTask extends SourceTask {
 ...
    @Override
    public void start(Map<String, String> props) {
     try {
    // Init variables
    SingleFileSourceConnectorConfig config = new SingleFileSourceConnectorConfig(props);
    topic = config.getString(SingleFileSourceConnectorConfig.TOPIC_NAME);
    file = config.getString(SingleFileSourceConnectorConfig.DIR_FILE_NAME);
    fileNamePartition = Collections.singletonMap(FILENAME_FIELD, file);
    offset = context.offsetStorageReader().offset(fileNamePartition); //소스 커넥터에서 관리하는 내부 번호를 기록하는 용도
     // Get file offset from offsetStorageReader
     if (offset != null) {
     Object lastReadFileOffset = offset.get(POSITION_FIELD);
     if (lastReadFileOffset != null) {
        position = (Long) lastReadFileOffset; //만약 기존에 저장된 내부 번호가 있다면 해당 번호부터 시작
     }
    } else { //만약 기존에 저장된 내부 번호가 없다면 0부터 시작
        position = 0;
 }
```


```java
public class SingleFileSourceTask extends SourceTask {
    ...
    @Override
    public List<SourceRecord> poll() {
        List<SourceRecord> results = new ArrayList<>();
    try {
        Thread.sleep(1000);
        List<String> lines = getLines(position); //가져가고 싶은 position(내부 번호)부터 데이터를 읽어감
        if (lines.size() > 0) {
        lines.forEach(line -> {
        Map<String, Long> sourceOffset = Collections.singletonMap(POSITION_FIELD, ++position);
        SourceRecord sourceRecord = new SourceRecord(fileNamePartition,
        sourceOffset, topic, Schema.STRING_SCHEMA, line);
        results.add(sourceRecord); //토픽으로 보내고 싶은 데이터는 List<SourceRecord>에 element 추가
             });
        }
        return results; //최종적으로 토픽으로 전송되는 List
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
        throw new ConnectException(e.getMessage(), e);
    }
 }
```

--- 

## 커넥터 옵션값 설정시 중요도 지정 기준

커넥터를 개발할 때 옵션값의 중요도를 Importance enum 클래스로 지정할 수 있다. Importance enum 클래스는 HIGH, MEDIUM, LOW 3가지 종류로 나뉘어 있다.

결론부터 말하자면 옵션의 Importance를 HIGH,MEDIUM,LOW로 정하는 명확한 기준은 없다. 단지 사용자로 하여금 이옵션이 중요하다는 것을 명시적으로 표시하기 위한

문서로 사용할 뿐이다. 그러므로 커넥터에서 반드시 사용자가 입력한 설정이 필요한 값은 HIGH, 사용자의 입력값이 없더라도 상관없고 기본값이 있는 옵션을 MEDIUM,

사용자의 입력값이 없어도 되는 옵션을 LOW 정도로 구분하여 지정하면 된다.