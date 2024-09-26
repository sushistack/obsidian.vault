> log(로그를) stash(저장한다)
> 플로그인 기반의 오픈소스 데이터 처리 파이프라인 도구
> 데이터를 저장하기 전에 원하는 형태로 가공하는 역할


## 특징

### 1. 플러그인 기반

: 파이프라인을 구성하는 각 요소들은 전부 플러그인 형태로 만들어져 있다.  
(플러그인 생성 방법 [contributing-to-logstash](https://www.elastic.co/guide/en/logstash/current/contributing-to-logstash.html))

### 2. 모든 형태의 데이터 처리

JSON, XML 등의 다양한 형태의 데이터를 입력받아 가공한 다음 저장할 수 있다.

### 3. 성능

자체적으로 내장되어 있는 메모리와 파일 기반의 큐를 사용하므로 처리 속도와 안정성이 높다.

### 4. 안정성

장애 상황에 대응하기 위한 재시도 로직이나 오류가 발생한 도큐먼트를 따로 보관하는 [dead-letter-queue](https://www.elastic.co/guide/en/logstash/current/dead-letter-queues.html)를 내장하고 있다.


## 파이프라인

![](Pasted%20image%2020240926152500.png)

```
input {
    { 입력 플러그인 }
}

filter {
    { 필터 플러그인(optional) }
}

output {
    { 출력 플러그인 }
}
```

### 입력 (input)

![](Pasted%20image%2020240926152527.png)

파이프라인 가장 앞부분에 위치하여 소스 원본으로부터 데이터를 입력받는 단계이다.

[input-plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)

#### file input - logstash.conf

```
input {
  file {
    path => "/Users/nhn/logs/study/test.log"
  }
}

... (생략)
```

| setting        | desc           | ex.                                                                                   |
| -------------- | -------------- | ------------------------------------------------------------------------------------- |
| path           | 대상 경로          | (ex) "/Users/user/logs/study/multiple/*.log"  <br>(ex) ["target.log", "target-2.log"] |
| exclude        | 제외할 파일명        | (ex) ["*.csv", "file-exclude.log"]                                                    |
| start_position | 파일 읽기를 시작하는 위치 | beginning  <br>end (default)                                                          |

#### jdbc input - logstash.conf

```
input {
    jdbc {
        # Loading class `com.mysql.jdbc.Driver'. This is deprecated.
        jdbc_driver_class => "com.mysql.cj.jdbc.Driver"

        # JDBC driver 경로 
        jdbc_driver_library => "mysql-connector-java-8.0.11.jar"

        # DB 연결 정보
        jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
        jdbc_user => "user"
        jdbc_password => "password"

        # SQL문
        statement => "SELECT * FROM table WHERE reg_ymdt > :sql_last_value"

        # sql_last_value 값 저장 경로. 기본값은 "~/.logstash_jdbc_last_run"
         last_run_metadata_path => “{last_run_path}"

        # 실행 빈도(cron 형식)
        schedule => "*/1 * * * * *" 
    }
}
```

#### Sincedb

Elasticsearch의 `_sincedb`는 로그 파일이나 데이터 파일을 읽는 위치를 추적하는 데 사용되는 내부 데이터베이스

##### `sincedb_path`

- Value type is [string](https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html#string "String")
- There is no default value for this setting.

Path of the sincedb database file (keeps track of the current position of monitored log files) that will be written to disk. The default will write sincedb files to `<path.data>/plugins/inputs/file` NOTE: it must be a file path and not a directory path



### 필터 (filter)

데이터를 정형화하고 사용자가 사용한 데이터 형태로 가공하는 역할을 한다.

#### mutate

- 일반적인 가공 함수들을 제공 (필드명을 변경, 문자열 처리 등)
- [Mutate Filter Configuration Options](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html#plugins-filters-mutate-options)
- mutate는 많은 옵션이 있어서 [처리 순서](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html#plugins-filters-mutate-proc_order) 주의

```
filter {
    mutate {
        split => { "message" => "&" }
        gsub => [ "message", "(log_code|log_type|inflow|event_id)=", "" ]
        add_field => { 
            log_code => "%{[message][0]}"
            log_type => "%{[message][1]}"
            inflow => "%{[message][2]}"
            event_id => "%{[message][3]}"
        }
        remove_field => ["message"]
    }
}
```

#### dissect

- 간단한 패턴을 사용해 메시지를 구조화된 형태로 분석
- grok에 비해 정규식을 사용하지 않아 자유도는 조금 떨어지지만 더 빠른 처리가 가능
- [Test Tokenizer Patterns for the Dissect filter](https://dissect-tester.jorgelbg.me/)
- 다양한 표기법 및 구분 기호를 사용하여 메시지를 분석

|method|desc|
|---|---|
|normal field notation|%{field_name}|
|skip field notation|`%{}` , `%{?foo}`   <br>결과에서 제외시킬 때 사용되며, 키가 주어질 땐 `?`가 접두사로 붙음|
|append field notation|`%{+field\_name}` `%{+field\_name/order}`  <br>특정 필드에 값을 추가할 때 사용  <br>(ex)  <br>text: `1 2 3 go`  <br>notation: `%{+a/2} %{+a/1} %{+a/4} %{+a/3}`   <br>output: `{ a => 2 1 go 3 }`|
|indirect field notation|`%{&field\_name}`  <br>키/값을 생성할 때 사용하며, 키 앞에 `&`가 붙음.  <br>(ex)  <br>text: `ERROR: error_code, error_description`   <br>notation: `ERROR: %{?err}, %{&err}`  <br>output: `{ error_code => error_description }`|
|공백 처리|공백을 무시하는 기호로 `->`를 사용.  <br>(ex)  <br>text: f3000a3b Calc       machine-123  <br>notation: `%{id} %{function->} %{server}`  <br>output:   <br>`{<br>"id": "00000043", "function": "ViewReceive","server": "machine-123"`|

```
// 예제 로그 file
[2022-01-27 21:45:45,875]    [ID2] 218.25.32.70 1070 [warn] - busy server.
```

```
filter {
    dissect {
        mapping => { "message" => "[%{timestamp}]%{?->}[%{id}] %{ip} %{+ip} [%{?level}] - %{}." },
        remove_field => ["message"]
    }
}
```

#### grok

```
// 예제 file
55.3.244.1 GET /index.html 15824 0.043 e1001
55.3.244.1 GET /index.html 15824 0.043 123456
```

```
filter {
    grok {
        pattern_definitions => { "EVENT_PATTERN" => "e[0-9]{4}" }
        match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration} %{EVENT_PATTERN:event}" }
    }
}
```

#### date

- 날짜/시간 포맷을 기본 날짜.시간 포맷으로 인덱싱할 수 있도록 지원
- [Joda Time 라이브러리](http://joda-time.sourceforge.net/timezones.html)
- elasticsearch => ISO8601 표준 포맷

```
// 예제 file
[2022-01-27 21:45:45,875]
```

```
// logstash.conf
filter {
    dissect {
        mapping => { "message" => "[%{timestamp}]" }
    }
    date {
        match => ["timestamp", "YYYY-MM-DD HH:mm:ss,SSS"]
        target => "new_timestamp"
        timezone => "UTC"
    }
}
```

### 출력 (output)

파이프라인의 입력과 필터를 거쳐 가공된 데이터를 지정한 대상으로 내보내는 단계이다.
[output-plugins](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)


```
output {
    elasticsearch {
        hosts => ["localhost:9200"]
        user => "user" 
        password => "password" 

        # 지정하지 않으면 logstash-%{+YYYY.MM.DD}
        index => "traffic-api"
    }
    stdout {}
}
```

```
output {
    csv {
        fields => ["column01", "column02"]
        path => "test.csv"
        csv_options => {
            write_headers => true
            headers =>["header01", "header02"]
        }
    }
}
```

### 코덱

- 데이터 표현을 변경할 수 있도록 다양한 [코덱 플러그인](https://www.elastic.co/guide/en/logstash/current/codec-plugins.html)을 제공하며, 입력과 출력에서 사용 가능
- `/bin/logstash-plugin list`로 설치여부를 확인하고 `bin/logstash-plugin install <codec-plugin-name>`를 통해 설치

```
// sample.csv
value-1,value-2,value-3,value-4

// logstash.conf
{
input {
  file {
    path => "/Users/nhn/logs/study/test-codec.log"
    start_position => "beginning"
    codec => "csv"
  }
}
```

```
input {
    tcp {
        port => 8080
    }
}
output {
    file {
        path => "/Users/nhn/logs/study/output.log"
        codec => "line"
    }
}
```

### 다중 파이프라인

로그 입수가 다양해질 때
1. 로그스태시를 별도로 N개 실행 => 독립적인 JVM 인스턴스이기 때문에 모니터링이 어렵고 전체적인 관리가 어려워짐
2. 로그스태시 설정 파일에 조건문을 두어 분리 => 하나의 파이프라인으로 처리할 수 있는 장점은 있지만 소스 로직이 복잡해짐.

=> **유지관리** 및 **재사용이 가능**한 모둘형 파이프라인 구성을 통해 위 단점을 해결한다.

#### 모듈형 파이프라인 구성 (pipelines.yml)

- 하나의 로그스태시에서 여러 개의 파이프라인을 독립적으로 실행할 수 있게 함.
- 로그스태시 실행 시에 설정 파일을 지정하지 않으면 pipelines.yml 파일이 실행됨
- [glob 표현식](https://www.elastic.co/guide/en/logstash/current/glob-support.html)을 사용하여 pipelines.yml 설정 파일을 정의


```
// pipelines.yml
- pipeline.id: my-pipeline_1
  path.config: "설정파일경로-1"
- pipeline.id: my-pipeline_2
  path.config: "설정파일경로-1"
```

## 모니터링

로그스태시에서 제공하는 API를 사용하면 간단하게 설정 정보를 확인할 수 있다.

| API              | 사용법                                                                                                                                                                                           |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Node Info API    | curl -XGET 'localhost:9600/_node?pretty'  <br>curl -XGET 'localhost:9600/_node/<types>?pretty'  <br>ㄴ types: pipeline, os, jvm                                                                |
| Plugins Info API | curl -XGET 'localhost:9600/_node/plugins?pretty'                                                                                                                                              |
| Node Stats API   | curl -XGET 'localhost:9600/_node/stats?pretty'  <br>curl -XGET 'localhost:9600/_node/stats/<types>?pretty'  <br>ㄴ types: jvm, process, events, pipelines, reloads, os, geoip_download_manager |
| Hot Threads API  | curl -XGET 'localhost:9600/_node/hot_threads?pretty'  <br>curl -XGET 'localhost:9600/_node/hot_threads?human=true'  <br>ㄴ logstash8.0.0 부터 `human=true` 옵션이 제공되어 JSON 형식을 평문으로 조회할 수 있다.      |

### 모니터링 기능 활성화

- elasticsearch x-pack 기능 사용
- x-pack 기능은 Elastic 라이센스를 갖고 있기에 oss 버전에서는 사용x
- logstash.yml의 xpack 설정을 true로 변경하여 키바나>Stack Monitoring 메뉴에서 확인

```
// logstash.yml
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.hosts: ["https://localhost:9200"]
```
